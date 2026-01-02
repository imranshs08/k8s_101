# Kind (Kubernetes in Docker) Setup

## 1. Why Kind?
For PDB labs, we need **Multi-Node Clusters**.
*   **Minikube**: Can do multi-node but is heavier (VM based usually).
*   **Kind**: Runs nodes as Docker Containers. It is lightweight, fast, and the standard for Kubernetes testing (Kubernetes CI uses Kind).

## 2. How Kind Works
Kind spins up a container for each "Node".
*   **Control Plane Node**: A container running API Server, Controller Manager, Scheduler, Etcd.
*   **Worker Node**: A container running Kubelet, Kube-proxy.

This allows us to simulate `kubectl drain node-1` by just stopping containers internally, without crashing your Windows machine.

## 3. Installation

**Chocolatey:**
```powershell
choco install kind -y
```

**Scoop:**
```powershell
scoop install kind
```

## 4. Verification
```powershell
kind --version
# kind v0.20.0 go1.20.5 windows/amd64
```

## 5. Version Pinning (Advanced)
If you need to simulate a specific K8s version (e.g., simulating an upgrade from 1.28 to 1.29), you specify the image version during creation.

```powershell
kind create cluster --image kindest/node:v1.29.0
```
