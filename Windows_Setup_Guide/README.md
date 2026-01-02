# Windows Setup Guide for Pod Disruption Budget (PDB) Labs

This guide provides the **exact infrastructure setup** required to execute the labs found in `../Pod_Disruption_Budget`.

## 📂 Quick Links
-   [**Installation**](01_Installation/): Docker, Kubectl, Kind.
-   [**Cluster Setup**](02_Cluster_Setup/): Creating the 3-node cluster.
-   [**Cheat Sheets**](03_Cheat_Sheets/): Rapid reference.

## 🚀 The "Happy Path": Get Started in 5 Minutes

### 1. Prerequisite Check
Ensure Docker Desktop is running (WSL2 mode).

### 2. Create the Infrastructure
We will create a 3-node HA cluster named `pdb-lab`.

```powershell
# Navigate to this folder
cd c:\devops-labs\Kubernetes_101\Windows_Setup_Guide

# Create Cluster
kind create cluster --config kind-multi-node.yaml --name pdb-lab
```

### 3. Verify
```powershell
kubectl cluster-info --context kind-pdb-lab
kubectl get nodes
```
*Expected Output: 3 Nodes (Ready).*

### 4. Run the Labs
Now navigate to the PDB module:
```powershell
cd ..\Pod_Disruption_Budget
type README.md
```
Follow the Lab instructions. When they say "kubectl drain", perform it against one of the `worker` nodes you just created.

## ⚠️ Known Windows Gotchas

### 1. `kubectl drain` Timeouts
Sometimes on Windows, `kubectl drain` might timeout if the terminal loses connection.
**Fix**: PDBs still work on the server side. Check `kubectl get pods` in a new terminal.

### 2. High Memory Usage
Running 3 nodes takes ~3-4GB RAM. If your PC slows down, delete the cluster immediately after use:
```powershell
kind delete cluster --name pdb-lab
```
