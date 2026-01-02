# Production Playbook: Troubleshooting Kubernetes DNS

> **Severity 1: DNS Outage**
> *Symptoms*: "App can't connect to DB", "502 Bad Gateway everywhere", "Connection Timed Out".

## 🚨 Phase 1: Verify the Scope (Is it DNS?)
Don't assume. Verify.

1.  **Check CoreDNS Pods**:
    ```bash
    kubectl get pods -n kube-system -l k8s-app=kube-dns
    ```
    *   *If CrashLoopBackOff*: Config error or OOM. -> **Goto Phase 3**
    *   *If Running*: It might be network or load. -> **Goto Step 2**

2.  **Test Resolution from Inside**:
    Exec into a debug pod.
    ```bash
    kubectl exec -it dns-utils -- nslookup google.com
    kubectl exec -it dns-utils -- nslookup my-service.default.svc.cluster.local
    ```
    *   *If External fails but Internal works*: Upstream DNS issue (Node/Cloud Provider).
    *   *If Both fail*: CoreDNS service is down or unreachable.

## 🕵️ Phase 2: Metrics Analysis (Dynatrace)

1.  **Check `coredns_dns_request_duration_seconds`**:
    *   Is it > 100ms? Network congestion or Node saturation.
2.  **Check `coredns_dns_response_rcode_count_total`**:
    *   High `SERVFAIL`? CoreDNS can't talk to APIServer or Upstream.
    *   High `NXDOMAIN`? App misconfiguration.

## 🛠️ Phase 3: Mitigation

### Scenario A: Pods are Crashing
1.  **Get Logs**: `kubectl logs -l k8s-app=kube-dns -n kube-system`
2.  **Revert Changes**: Did someone edit the ConfigMap?
    ```bash
    kubectl rollout undo deployment coredns -n kube-system
    ```

### Scenario B: High Latency / Load
1.  **Scale Up**: Add replicas.
    ```bash
    kubectl scale deployment coredns --replicas=5 -n kube-system
    ```
2.  **Enable Autopath** (Advanced): Reduces query amplification.

### Scenario C: Node Networking broken
If only *some* pods fail, it might be a specific Node.
*   **Drain the Node**: Move pods elsewhere.
    ```bash
    kubectl drain <node-name> --ignore-daemonsets
    ```

## 📜 Root Cause Analysis (RCA)
Once resolved, answer:
1.  Was the ConfigMap validated?
2.  Did we hit Conntrack limits on the Node?
3.  Was it a noisy neighbor pod?
