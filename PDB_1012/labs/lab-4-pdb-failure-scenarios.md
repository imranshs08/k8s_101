# Lab 4: Failure & Deadlock Scenarios

## Objective
Create a self-inflicted outage/deadlock using PDBs and learn how to fix it.

## The Deadlock Scenario

### 1. Deploy the "Impossible" PDB
We will require 100% availability, meaning no voluntary disruptions are allowed.
```bash
kubectl apply -f ../manifests/deployment.yaml
# Manually create a strict PDB
kubectl create pdb strict-pdb --selector=app=payment-service --min-available=100%
```

### 2. Attempt Maintenance
Try to drain the node where these pods run.
```bash
kubectl drain <node-name> --ignore-daemonsets
```

### 3. Observe the Deadlock
The command will hang forever.
```
Cannot evict pod ... violates disruption budget
```

### 4. Recovery (The Fix)
You are the SRE on call. The upgrade is stuck. What do you do?

**Option A: Scale Up**
If you have capacity, scale the deployment to 5. Now 4/5 are required (if percentage) or if absolute number was used.
*Note: This doesn't work if minAvailable is 100%.*

**Option B: Edit/Delete PDB**
The fastest fix during an incident.
```bash
kubectl delete pdb strict-pdb
```
The drain proceeds immediately.

**Option C: Allow Data Loss (Emergency)**
Force delete the pod (bypasses PDB).
```bash
kubectl delete pod <pod-name> --force --grace-period=0
```
*Warning*: This is a "Involuntary Disruption" from the PDB's perspective. It skips the safety check.

### 5. Conclusion
Always alert on `DisruptionBudgetAtLimit` or check for PDBs with `AllowedDisruptions: 0`.
