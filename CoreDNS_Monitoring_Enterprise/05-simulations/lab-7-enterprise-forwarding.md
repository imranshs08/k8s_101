# Lab 7: Simulating Enterprise/Hybrid DNS Forwarding

## Objective
Simulate a real-world enterprise scenario where Kubernetes must resolve internal corporate domains (e.g., `hr.corp.local`) hosted on an on-premise Windows/Active Directory DNS server.

Instead of a real Windows Server, we will use a **Simulated DNS Pod**.

## Architecture
```
[ App Pod ] --(corp.local)--> [ Cluster CoreDNS ] --(Forward)--> [ "External" DNS Pod ]
```

## Steps

### 1. Deploy the "External" DNS Simulator
This pod acts as your "Corporate DNS". It knows about `*.corp.local`.
```bash
kubectl apply -f ../manifests/external-dns-sim.yaml
```

### 2. Get the Simulator's IP
We need the ClusterIP of this new service to tell CoreDNS where to send traffic.
```bash
EXTERNAL_DNS_IP=$(kubectl get svc external-dns-service -n default -o jsonpath='{.spec.clusterIP}')
echo "Corporate DNS is at: $EXTERNAL_DNS_IP"
```
*Make a note of this IP (e.g., 10.96.x.x).*

### 3. Reconfigure Cluster CoreDNS
We need to add a `forward` rule for `corp.local`.

1.  Edit the CoreDNS ConfigMap:
    ```bash
    kubectl edit configmap coredns -n kube-system
    ```
2.  Add the new block **before** the main `.:53` block (Order matters!):

    ```coredns
    corp.local:53 {
       errors
       cache 30
       forward . <YOUR_EXTERNAL_DNS_IP>
    }
    .:53 {
       ... existing config ...
    }
    ```
    *Replace `<YOUR_EXTERNAL_DNS_IP>` with the IP from Step 2.*

3.  Restart CoreDNS:
    ```bash
    kubectl rollout restart deployment coredns -n kube-system
    ```

### 4. Verify Resolution
Test if your pod can now resolve "corporate" records.
```bash
kubectl exec -it dns-test -- nslookup hr.corp.local
```

**Expected Output**:
```text
Server:     10.96.0.10
Name:       hr.corp.local
Address:    10.0.0.100
```

### 5. Troubleshooting (Dynatrace)
If it's slow:
1.  Check `coredns_forward_request_duration_seconds`.
2.  Filter by `to="<YOUR_EXTERNAL_DNS_IP>"`.
3.  Forwarding latency usually indicates network issues between K8s and the remote server.

## 6. Breaking It (Simulation)

Try these scenarios to see what "Upstream Failure" looks like.

### Scenario A: The "Cut Cable" (Upstream Outage)
Simulate a complete network failure to the Windows DNS server.
```bash
kubectl scale deployment external-dns-sim --replicas=0
```
Now try to resolve:
```bash
kubectl exec -it dns-test -- nslookup hr.corp.local
```
**Result**:
*   The command hangs for ~5-10 seconds.
*   Output: `;; connection timed out; no servers could be reached`
*   **Dynatrace**: You will see `SERVFAIL` errors spiking.

### Scenario B: The "Fat Finger" (Bad Config)
Simulate a typo in the CoreDNS config (Wrong IP).

1.  Edit the map: `kubectl edit configmap coredns -n kube-system`
2.  Change the forward IP to `1.2.3.4` (or any dead IP).
3.  Restart CoreDNS.
4.  Test resolution.
**Result**: Identical timeout, but **Dynatrace** will show the `forward` destination as `1.2.3.4`, proving it's a config error, not a network error to the *correct* server.

## Cleanup
Remove the simulator and revert the ConfigMap.
```bash
kubectl delete -f ../manifests/external-dns-sim.yaml
kubectl apply -f ../manifests/coredns-configmap.yaml
kubectl rollout restart deployment coredns -n kube-system
```
