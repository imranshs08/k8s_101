# Lab 1: Node Drain with PDB

## Objective
Understand how PDB prevents a node drain when it violates the availability budget.

## Prerequisites
- A Kubernetes cluster (minikube, kind, or EKS/AKS) with at least 2 nodes.
- `kubectl` configured.

## Scenario
You will deploy the `payment-service` with 4 replicas and a PDB requiring `minAvailable: 2`. You will then try to drain a node hosting some of these replicas.

## Step-by-Step Instructions

### 1. Deploy the Application and PDB
```bash
kubectl apply -f ../manifests/deployment.yaml
kubectl apply -f ../manifests/pdb-minAvailable.yaml
```

Wait for pods to be ready:
```bash
kubectl get pods -l app=payment-service -o wide
```
*Verification*: Ensure you see 4 Running pods.

### 2. Identify the Nodes
Find which node hosts your pods:
```bash
kubectl get pods -l app=payment-service -o wide
```
Let's say `node-01` has 2 replicas.

### 3. Attempt to Drain the Node
We will simulate a kernel patch maintenance.
```bash
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

### 4. Observe the Behavior
**Without PDB** (Hypothetical): All pods on the node would be killed instantly.
**With PDB**:
- The first pod on the node might be evicted immediately (since 3/4 > 2).
- The **second** pod eviction might be **blocked** or delayed until the first evicted pod is rescheduled and Running on another node.

Check the PDB status in another terminal:
```bash
kubectl get pdb payment-pdb-minavailable -w
```
Watch the `ALLOWED DISRUPTIONS` column. It should drop to 0.

### 5. Expected Output
If `kubectl drain` hangs with:
```
evicting pod default/payment-service-xyz...
error when evicting pools: ... Cannot evict pod as it would violate the pod's disruption budget.
```
**This is SUCCESS!** The PDB is doing its job protecting your application. PDB forces the drain process to wait until new pods are up.

### 6. Cleanup
Uncordon the node to allow pods back:
```bash
kubectl uncordon <node-name>
```
