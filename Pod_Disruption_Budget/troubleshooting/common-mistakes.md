# Troubleshooting & Common PDB Mistakes

## 🚨 Top 5 Common PDB Mistakes

### 1. `minAvailable: 100%` (The Upgrade Blocker)
**The Mistake**: Setting `minAvailable: 3` when you only have `replicas: 3`.
**The Impact**: You have created an "Immortal Deployment". Nodes hosting these pods CANNOT be drained. Cluster upgrades will hang indefinitely.
**The Fix**: always leave room for at least 1 disruption. Use `minAvailable: N-1` or `maxUnavailable: 1`.

### 2. Using PDBs with DaemonSets
**The Mistake**: Creating a PDB for a DaemonSet (e.g., Fluentd, Datadog).
**The Impact**: The PDB Controller ignores it or behaves unpredictably because DaemonSet pods are tied to specific nodes. If a node drains, the pod *must* die. It cannot move.
**The Fix**: Do not use PDBs for DaemonSets.

### 3. Mismatched Label Selectors
**The Mistake**: Deployment has label `app: my-app`, but PDB selects `app: myapp`.
**The Impact**: The PDB protects *nothing*. The controller sees 0 pods matching the PDB.
**The Fix**: Always copy-paste labels or use `kubectl get pods -l <pdb-selector>` to verify coverage.

### 4. Zero Allowed Disruptions (Deadlock)
**The Symptom**: `kubectl drain node-01` hangs forever with `Cannot evict pod`.
**The Check**:
```bash
kubectl get pdb
# NAME      MIN AVAILABLE   MAX UNAVAILABLE   ALLOWED DISRUPTIONS   AGE
# my-pdb    3               N/A               0                     10d
```
If `ALLOWED DISRUPTIONS` is `0`, the drain will block.
**The Reason**:
- You might have unhealthy pods (CrashLoopBackOff).
- PDB counts "Healthy" pods. If 1 is crashing, and your budget allows only 1 unavailable, the budget is already spent.

### 5. Single Replica Deployments + PDB
**The Mistake**: `replicas: 1` and `minAvailable: 1`.
**The Impact**: Same as #1. You cannot drain the node. Note that if you use `maxUnavailable: 1`, it works (because 1 unavailable is allowed).

---

## 🛠️ How to Debug "Stuck Draining"

### Step 1: Identify the Blocking Pod
The drain output usually tells you:
```
evicting pod default/frontend-7b87c5d7d-xyz...
error when evicting pools: ... Cannot evict pod as it would violate the pod's disruption budget.
```

### Step 2: Inspect the PDB
```bash
kubectl get pdb -A | grep frontend
```
Check `ALLOWED DISRUPTIONS`.

### Step 3: Check Pod Health
Are there other replicas of "frontend" that are currently dead?
```bash
kubectl get pods -l app=frontend
```
If 2/3 are Running and 1 is Error, and PDB requires `minAvailable: 2`, then you have 0 disruptions allowed. The draining pod cannot be killed because you are already at the limit.

### Step 4: Emergency Bypass
If you MUST drain the node (e.g., hardware fire), delete the pod directly:
```bash
kubectl delete pod frontend-7b87c5d7d-xyz --force --grace-period=0
```
Or delete the PDB temporarily:
```bash
kubectl delete pdb frontend-pdb
```
