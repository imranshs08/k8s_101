# Lab 3: Cluster Upgrade Scenario

## Objective
Simulate a managed Kubernetes (AKS/EKS/GKE) upgrade process where nodes are replaced one by one.

## Concept
In managed upgrades, the cloud provider creates a new node (Upgrade Surge), then `drains` an old node. This is identical to **Lab 1** but happens automatically for *every* node.

## Steps (Simulation)

### 1. Setup Postgres (Stateful Workload)
Stateful workloads are the most sensitive to upgrades.
```bash
kubectl apply -f ../manifests/statefulset.yaml
kubectl apply -f ../manifests/pdb-statefulset.yaml
```
Wait for 3 replicas.

### 2. Simulate Node Replacement
Pretend `node-02` is being upgraded. It has `postgres-db-1`.
```bash
kubectl drain node-02 --ignore-daemonsets
```

### 3. Verify Quorum Preservation
While the drain is running, check the other pods.
Since `minAvailable: 2`, the drain is allowed ONLY if `postgres-db-0` and `postgres-db-2` are Healthy.

If `postgres-db-0` was crashing (simulated failure), the drain of `node-02` would be **BLOCKED** to prevent dropping to only 1 replica (loss of Quorum).

### 4. The "Blocking" Risk
If you had set `minAvailable: 3` (requiring 3/3 pods), the cluster upgrade would **never complete**. The cloud provider would wait for minutes/hours and then fail the upgrade operation.

**Takeaway**: PDBs control the *speed* and *success* of cluster upgrades.
