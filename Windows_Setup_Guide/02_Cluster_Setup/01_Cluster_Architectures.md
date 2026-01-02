# Cluster Architecture Setup

This section covers creating the Kubernetes environment required for PDB labs.

## 1. Single Node Cluster (The Limitation)
A default Kind cluster is single-node.
```powershell
kind create cluster --name single-node-lab
```

**Why it's bad for PDB Labs:**
If you have a Deployment with `replicas: 2` and `minAvailable: 1`, both pods run on `Node-A`.
If you `drain Node-A`, you evict **BOTH** pods. The PDB blocks eviction of the last pod, but since the node is draining, the pod has nowhere else to go. The drain is stuck effectively, or fails immediately.
*You cannot safely test "Rescheduling" on a single node.*

## 2. Multi-Node Cluster (Production Simulation) ✅
We need at least 1 Worker separate from Control Plane, ideally 2 Workers to show movement.

### Config File (`kind-multi-node.yaml`)
Create this file in your workspace:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

### Create Command
```powershell
kind create cluster --name pdb-cluster --config kind-multi-node.yaml
```

### Verification
```powershell
kubectl get nodes
```
You should see 3 nodes listed.

## 3. Multiple Clusters (Advanced Simulation)
To simulate a "Blue-Green" cluster upgrade or multi-region failover, you can run two kind clusters side-by-side.

```powershell
kind create cluster --name region-east
kind create cluster --name region-west
```

Switch contexts:
```powershell
kubectl config use-context kind-region-east
kubectl config use-context kind-region-west
```
