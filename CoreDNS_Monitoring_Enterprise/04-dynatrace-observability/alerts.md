# Enterprise DNS Alerting Strategy

Alert on **Symptoms** (User Pain), not just **Causes** (CPU high).

## P1: Critical Alerts (Page the SRE)
### 1. High DNS Error Rate (Effect)
*   **Condition**: `SERVFAIL` count > 5% of total queries for 5 minutes.
*   **Meaning**: CoreDNS is broken, Upstream is down, or Network is partitioned.
*   **Action**: Restart CoreDNS, Check Upstream status.

### 2. High Resolution Latency (Effect)
*   **Condition**: `coredns_dns_request_duration_seconds` (p90) > 200ms for 5 minutes.
*   **Meaning**: Users perceive the app as "hanging".
*   **Action**: Check Node CPU/Network saturation.

## P3: Warning Alerts (Ticket/Slack)
### 3. CoreDNS Pod CrashLoop (Cause)
*   **Condition**: `k8s.pod.crash_loop_back_off` event detected on label `k8s-app=kube-dns`.
*   **Meaning**: Pods are unstable but Service might still be working via other replicas.

### 4. Zero Available Replicas (Cause)
*   **Condition**: `kube_deployment_status_replicas_available` == 0.
*   **Meaning**: Total DNS outage. (Usually covered by Error Rate alert, but good redundancy).

### 5. High NXDOMAIN Rate (Anomaly)
*   **Condition**: `NXDOMAIN` > 200% of baseline.
*   **Meaning**: New deployment is misconfigured or security scan in progress.
