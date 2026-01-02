# Kind Management Cheat Sheet

## Creation
```powershell
# Simple
kind create cluster --name lab1

# From Config (Multi-node)
kind create cluster --config kind-multi-node.yaml --name ha-cluster

# Specific Version
kind create cluster --image kindest/node:v1.28.0
```

## Deletion
```powershell
# Delete specific
kind delete cluster --name lab1

# Delete ALL (Cleanup)
kind delete clusters --all
```

## Interaction
```powershell
# Load local Docker image into Cluster (Avoids re-pulling)
kind load docker-image my-app:latest --name ha-cluster

# Export Kubeconfig
kind export kubeconfig --name ha-cluster
```
