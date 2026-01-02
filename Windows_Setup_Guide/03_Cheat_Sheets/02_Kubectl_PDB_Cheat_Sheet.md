# Kubectl & PDB Cheat Sheet

## General
```powershell
kubectl get nodes -o wide      # See node IPs
kubectl top nodes              # Check CPU/Memory load on simulated nodes
```

## Pod Disruption Budget (PDB)
```powershell
# List all PDBs
kubectl get pdb -A

# Detailed Status (Check AllowedDisruptions)
kubectl describe pdb <pdb-name>

# YAML Dump
kubectl get pdb <pdb-name> -o yaml
```

## Draining & Node Ops
```powershell
# Cordon (Mark unschedulable)
kubectl cordon <node-name>

# Drain (The Main Event)
# --ignore-daemonsets: Required because DaemonSets ignore PDBs and can't move
# --delete-emptydir-data: Required if pods use local emptyDir volumes
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

## Debugging Sticks
If drain hangs:
1. `Ctrl+C` out of drain command.
2. `kubectl get pdb` (Is allowed disruptions 0?).
3. `kubectl get pods -A -o wide` (Are pods running on the node stuck terminating?).
