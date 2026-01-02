# Quick Start Installation Guide

## 1. Install Tools
Ensure you have Docker Desktop (WSL2), `kind`, and `kubectl` installed.

## 2. Create the Cluster
Run these commands from the **Project Root** (`CoreDNS_Monitoring_Enterprise` folder).

```bash
kind create cluster --config 02-cluster-setup/kind-multinode.yaml --name dns-lab
```
*Verify*:
```bash
kubectl get nodes
# Should see 1 control-plane, 2 workers.
```

## 3. Install Dynatrace Operator
(Requires your Tenant credentials)
> **Your Lab Credentials**:
> *   **URL**: `https://fqj51381.live.dynatrace.com`
> *   **Login**: `gevohik943@nctime.com`
1.  Create the `dynatrace` namespace:
    ```bash
    kubectl create namespace dynatrace
    ```
2.  Install the Operator:
    ```bash
    kubectl apply -f https://github.com/Dynatrace/dynatrace-operator/releases/latest/download/kubernetes.yaml
    ```
3.  **Generate Tokens**:
    *   **API Token** (Scopes needed):
        *   `Read metrics`
        *   `Read entities`
        *   `Read settings`
        *   `Write settings`
        *   `Access problem and event feed, metrics, and topology`
        *   `PaaS integration - Installer download`
    *   **Data Ingest Token** (Scopes needed):
        *   `Ingest metrics`
        *   `Ingest logs`
        *   `Ingest events`

4.  Apply your Secret (Token) and DynaKube:
    
    **Option A: Using the Secrets File (Recommended)**
    ```bash
    kubectl apply -f 04-dynatrace-observability/dynatrace-secrets.yaml
    ```

    **Option B: Manual Creation**
    ```bash
    kubectl create secret generic dynatrace-tokens -n dynatrace \
      --from-literal="apiToken=<API_TOKEN>" \
      --from-literal="dataIngestToken=<DATA_INGEST_TOKEN>"
    ```

    **Deploy the Operator config:**
    ```bash
    kubectl apply -f 04-dynatrace-observability/dynakube.yaml
    ```

## 4. Verification
Check if CoreDNS is running:
```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

You are now ready to start **Lab 1**.
