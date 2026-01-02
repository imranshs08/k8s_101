# Pod Disruption Budget (PDB) - The Complete Enterprise Guide

## 1. Introduction

### What is Pod Disruption Budget (PDB)?
A **Pod Disruption Budget (PDB)** is a Kubernetes resource that limits the number of pods of a replicated application that can be down simultaneously from **voluntary disruptions**. It ensures that your application maintains a minimum number of available replicas or a maximum number of unavailable replicas during cluster maintenance operations.

Think of PDB as an **SLA (Service Level Agreement)** between the **Application Owner** (Developer) and the **Cluster Administrator** (Platform/SRE Team).

### Why Kubernetes Introduced PDB
In the early days of Kubernetes, cluster administrators performing node maintenance (like upgrades or scaling down) had no easy way to know if draining a node would cause an outage for a specific application.  
PDB was introduced to give app owners control over their application's availability during these "voluntary" administrative actions.

### Voluntary vs. Involuntary Disruptions

| Disruption Type | Description | Examples | Does PDB Protect? |
| :--- | :--- | :--- | :--- |
| **Voluntary** | Actions initiated by an administrator or automation system. | `kubectl drain`, Cluster Autoscaler scaling down, Cluster Upgrades, Kernel patching. | **YES** ✅ |
| **Involuntary** | Unavoidable hardware or software failures. | Kernel panic, Node hardware failure, OOM kills, Network partition, Node not ready. | **NO** ❌ |

### Real Enterprise Problems PDB Solves
1.  **Zero-Downtime Cluster Upgrades**: Ensures your banking API doesn't drop below 3 replicas while the platform team rotates nodes.
2.  **Safe Auto-Scaling**: Prevents the Cluster Autoscaler from removing a node that holds the *last* healthy replica of a critical payment service.
3.  **Cost Optimization**: Allows aggressive use of Spot Instances (with graceful draining) or aggressive bin-packing without risking availability.

---

## 2. Why PDB is CRITICAL in Enterprise Kubernetes

In a large-scale enterprise environment (Fintech, Insurance, Banking), missing PDBs are a primary cause of **self-inflicted outages**.

### Real-World Scenarios

#### 1. OS Patching (Node Draining)
**Scenario**: The SecOps team mandates a critical kernel patch. The Platform team runs a script to `drain` and `reboot` 100 nodes.
-   **Without PDB**: The script might drain 5 nodes hosting ALL replicas of your "Transaction Service" simultaneously. Service goes DOWN.
-   **With PDB**: The Eviction API checks the budget. It allows draining 1 node, waits for a new pod to come up on a different node, and *blocks* draining of other nodes until the PDB is satisfied.

#### 2. Kubernetes Version Upgrades
**Scenario**: Upgrading EKS/AKS from v1.29 to v1.30. This involves replacing every node in the cluster.
-   **Without PDB**: The upgrade process is blind. It creates a "rolling outage" across random services.
-   **With PDB**: The upgrade process respects the `minAvailable` of every service, pausing the node rotation if a service is at risk.

#### 3. Cluster Autoscaling
**Scenario**: Traffic drops at night. Cluster Autoscaler decides to remove a "underutilized" node.
-   **Without PDB**: That node might contain the application's Leader or the last serving replica of a cache.
-   **With PDB**: Autoscaler simulates the eviction. If PDB says "No", the node is **not** scaled down, preserving availability over cost.

---

## 3. How PDB Works Internally (Deep Dive)

### The enforcement mechanism: `Eviction API`
PDBs are NOT enforced by the kubelet or the scheduler directly. They are enforced by the **Eviction API**.
Tools like `kubectl drain` do not just kill pods; they call the Eviction API.

1.  **Request**: Administrator runs `kubectl drain node-01`.
2.  **Simulation**: The API server receives an eviction request for `pod-A` on `node-01`.
3.  **Check**: The **Disruption Controller** (part of `kube-controller-manager`) checks all PDBs matching `pod-A`.
    -   It calculates `Current Healthy Replicas`.
    -   It compares against `minAvailable` or `maxUnavailable`.
4.  **Decision**:
    -   **Approved (200 OK)**: The pod is deleted gracefully.
    -   **Denied (429 Too Many Requests)**: The eviction is blocked. `kubectl drain` will retry (wait) until it is allowed.

### Interaction with Managed Services
-   **GKE / EKS / AKS Upgrade Processes**: These managed control planes effectively run a managed `drain` process. They **strictly respect** PDBs.
-   **Danger**: A generic PDB like `minAvailable: 100%` (or `maxUnavailable: 0`) will **BLOCK** a cluster upgrade indefinitely. The upgrade process will hang, waiting for a permission that never comes.

---

## 4. PDB Core Concepts & Configuration

### Key Fields

| Field | Description | Example | Use Case |
| :--- | :--- | :--- | :--- |
| `.spec.minAvailable` | The minimum number of pods that must remain available after eviction. | `2`, `80%` | Good for stateful apps (Quorum requirements). |
| `.spec.maxUnavailable` | The maximum number of pods that can be unavailable (down) after eviction. | `1`, `25%` | Better for stateless web apps (allows faster rolling updates). |
| `.spec.selector` | Label selector to find the pods this PDB protects. | `app: payment` | **MUST** match the Deployment/StatefulSet labels EXACTLY. |

### minAvailable vs maxUnavailable

| Feature | `minAvailable: 1` | `maxUnavailable: 1` |
| :--- | :--- | :--- |
| **Logic** | "Keep at least 1 alive" | "Don't kill more than 1 at a time" |
| **With `replicas: 1`** | **BLOCKS EVICTIONS** (if already 1). Dangerous. | Allows eviction (1 down is allowed). |
| **During Rolling Update** | Ensures absolute capacity. | Allows faster turnover. |

> [!TIP]
> **Best Practice**: Use `maxUnavailable` for Stateless applications. It handles scale-changes better. Use `minAvailable` for Quorum-based Stateful applications (e.g., ElasticSearch, ZooKeeper, Databases).

---

## 5. Working Examples & Explanations

See the `manifests/` directory for full YAML files.

### 1. Deployment + PDB (minAvailable) [Stateless]
**File**: `manifests/pdb-minAvailable.yaml`
-   **Configuration**: `replicas: 3`, `minAvailable: 2`.
-   **Behavior**: You can lose 1 pod. If you try to drain 2 nodes simultaneously, the second drain will wait.
-   **Use Case**: Critical web servers where you need to guarantee 66% capacity at all times.

### 2. Deployment + PDB (maxUnavailable) [Stateless]
**File**: `manifests/pdb-maxUnavailable.yaml`
-   **Configuration**: `replicas: 4`, `maxUnavailable: 25%` (equivalent to 1 pod).
-   **Behavior**: At least 3 pods must always be up.
-   **Use Case**: High-scale microservices. Allows the platform to clear nodes, provided it doesn't take down too many replicas at once.

### 3. StatefulSet + PDB [Stateful]
**File**: `manifests/pdb-statefulset.yaml`
-   **Configuration**: `replicas: 3`, `minAvailable: 2`.
-   **Criticality**: Databases like **PostgreSQL** or **Etcd** often rely on a consensus algorithm (Raft/Paxos). If `replicas` is 3, you need `2` for a quorum.
-   **Safety**: This PDB ensures that you never lose quorum during a cluster upgrade.

### 4. Invalid Scenario: DaemonSet
**File**: `manifests/daemonset.yaml`
-   **Why it fails**: DaemonSets are managed by the DaemonSet controller, which ensures one pod per node. PDBs are **NOT** supported for DaemonSets because a DaemonSet pod *must* exist on that specific node. You cannot "reschedule" it effectively to respect a budget in the same way.
-   **Note**: While the spec technically allows it in newer versions for specific edge cases, generally, **PDBs are irrelevant for DaemonSets**.

---

## 6. Labs Overview

We have prepared 4 hands-on labs in the `labs/` directory:

-   [**Lab 1: Node Drain with PDB**](labs/lab-1-node-drain.md) - The "Hello World" of PDBs.
-   [**Lab 2: Rolling Update & PDB**](labs/lab-2-rolling-update.md) - Interaction with Deployment rolling updates.
-   [**Lab 3: Cluster Upgrade Simulation**](labs/lab-3-cluster-upgrade.md) - Testing resilience.
-   [**Lab 4: Failure & Deadlocks**](labs/lab-4-pdb-failure-scenarios.md) - Debugging "Cannot evict pod".

---

## 7. Troubleshooting & Common Mistakes

### Stuck Drains
If `kubectl drain` hangs with `Cannot evict pod...`, check:
1.  **Is PDB blocked?** `kubectl get pdb` -> Check `ALLOWED DISRUPTIONS`. If 0, drain will wait.
2.  **Is `minAvailable` too high?** If `replicas: 3` and `minAvailable: 3` (or 100%), no voluntary disruption is possible. You have created a **Deadlock**.
3.  **Are pods healthy?** If 1 pod is crashing, PDB sees only 2/3 healthy. It counts the crashing pod against the budget, potentially blocking eviction of the healthy ones.

### Debugging Commands
```bash
kubectl get pdb -A
kubectl describe pdb <pdb-name>
kubectl get events --sort-by='.lastTimestamp'
```

---

## 8. Best Practices Checklist

-   [ ] **One PDB per Deployment/StatefulSet**: Don't share PDBs across apps.
-   [ ] **Prefer `maxUnavailable` for Stateless**: It's safer and prevents deadlocks if replicas scale down to 1.
-   [ ] **Avoid `minAvailable: 100%`**: This prevents any node maintenance.
-   [ ] **Define `minAvailable` for Quorum Apps**: Ensure (N/2)+1 availability.
-   [ ] **Test in Staging**: Run `kubectl drain` in staging before production to verify PDB behavior.
-   [ ] **Monitor PDBs**: Alert if `ALLOWED DISRUPTIONS` is 0 for an extended period.

---

*Created for PDB Learning Project (PDB_1012)*
