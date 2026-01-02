# Windows Kubernetes Simulation Cheat Sheet

This guide is designed for **Windows Users** to simulate a real Enterprise Kubernetes environment locally.
To practice **Pod Disruption Budgets (PDB)** and **Node Draining**, you CANNOT use standard Docker Desktop (single node). You need a **Multi-Node Cluster**.

We will use **Kind (Kubernetes in Docker)** to create a 3-node cluster on your machine.

---

## 🛑 Dos and Don'ts (Critical for Windows)

| Category | Do ✅ | Don't ❌ |
| :--- | :--- | :--- |
| **Terminal** | Use **PowerShell** or **Git Bash**. | Do NOT use standard Command Prompt (cmd). |
| **Paths** | Use forward slashes `/` or double backslashes `\\` in configs. | Do NOT assume Linux paths (`/etc/...`) work on Host. |
| **Line Endings** | Configure Git: `git config --global core.autocrlf true` | Do NOT save YAML files with Windows CRLF if mapping volumes. |
| **Cluster Tool** | Use **Kind** or **Minikube (Multi-node)**. | Do NOT use **Docker Desktop K8s** (It's 1 node only). |
| **Resources** | Allocate at least **6GB RAM** to Docker Desktop. | Do NOT try this with < 4GB RAM. |

---

## 🛠️ Step 1: Pre-requisites Installation

### 1. Install Docker Desktop
Ensure Docker Desktop is installed and running correctly.
-   Enable **WSL 2** backend in Docker Settings for best performance.

### 2. Install Tools (Using Scoop or Choco)
Open PowerShell as Administrator.

**Using Chocolatey:**
```powershell
choco install kubernetes-cli -y
choco install kind -y
```

**Using Scoop:**
```powershell
scoop install kubectl
scoop install kind
```

**Verify Installation:**
```powershell
kubectl version --client
kind --version
```

---

## 🏗️ Step 2: Create a Multi-Node Cluster

For PDB labs, we need to drain nodes. You cannot drain a 1-node cluster (everything dies). We will create a 3-node cluster (1 Control Plane, 2 Workers).

### 1. Create Config
I have created the config file for you at: `kind-multi-node.yaml`.

### 2. Spin up the Cluster
Run this command in this directory:
```powershell
kind create cluster --name pdb-lab --config ./kind-multi-node.yaml
```

*This will take 1-2 minutes.*

### 3. Verify Nodes
```powershell
kubectl get nodes
```
**Expected Output:**
```
NAME                    STATUS   ROLES           AGE   VERSION
pdb-lab-control-plane   Ready    control-plane   1m    v1.30.0
pdb-lab-worker          Ready    <none>          1m    v1.30.0
pdb-lab-worker2         Ready    <none>          1m    v1.30.0
```

---

## 🧪 Step 3: Running the Labs (Simulation)

Now you are ready to simulate the Enterprise Labs.

### Lab 1: Node Drain
1.  **Navigate to Labs:**
    ```powershell
    cd ../Pod_Disruption_Budget/manifests
    ```
2.  **Apply Workload:**
    ```powershell
    kubectl apply -f deployment.yaml
    kubectl apply -f pdb-minAvailable.yaml
    ```
3.  **Simulate Drain:**
    Pick a worker node name from `kubectl get nodes`.
    ```powershell
    kubectl drain pdb-lab-worker --ignore-daemonsets
    ```
    *Result*: You will see the PDB logic blocking/allowing eviction locally on your Windows machine!

4.  **Recover:**
    ```powershell
    kubectl uncordon pdb-lab-worker
    ```

---

## 🧹 Cleanup
When you are done with the labs, destroy the simulation to save RAM.

```powershell
kind delete cluster --name pdb-lab
```
