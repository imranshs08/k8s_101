# Quick Start Installation Guide

## 1. Install Tools
Ensure you have Docker Desktop (WSL2), `kind`, and `kubectl` installed.

## 2. Create the Cluster
Use the multi-node config we prepared.
```bash
kind create cluster --config ../02-cluster-setup/kind-multinode.yaml --name dns-lab
```
*Verify*:
```bash
kubectl get nodes
# Should see 1 control-plane, 2 workers.
```

## 3. Install Dynatrace Operator
(Requires your Tenant credentials)
1.  Create the `dynatrace` namespace:
    ```bash
    kubectl create namespace dynatrace
    ```
2.  Install the Operator:
    ```bash
    kubectl apply -f https://github.com/Dynatrace/dynatrace-operator/releases/latest/download/kubernetes.yaml
    ```
3.  Apply your Secret (Token) and DynaKube:
    ```bash
    kubectl apply -f ../04-dynatrace-observability/dynakube.yaml
    ```

## 4. Verification
Check if CoreDNS is running:
```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

You are now ready to start **Lab 1**.
