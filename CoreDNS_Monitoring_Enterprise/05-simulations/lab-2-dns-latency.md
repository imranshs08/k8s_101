# Lab 2: Simulating DNS Latency

## Objective
Inject artificial network latency into the node and observe how it impacts DNS resolution time and application performance.

## The Tool: `tc` (Traffic Control)
We will use a DaemonSet that runs `tc qdisc` (Queue Discipline) to hold packets in a buffer before releasing them.

## Steps

### 1. Start a Continuous Ping
Open Terminal 1. Watch the latency increase in real-time.
```bash
# This uses the debug container to ping google via CoreDNS
kubectl exec -it dns-test -- sh -c "while true; do dig google.com | grep 'Query time'; sleep 1; done"
```
**Baseline**: `;; Query time: 4 msec`

### 2. Inject the Failure
Open Terminal 2. Apply the "Network Delay" DaemonSet.
```bash
kubectl apply -f manifests/network-delay.yaml
```

### 3. Observe the Impact
Look at Terminal 1.
**New Output**: `;; Query time: 204 msec`

### 4. Dynatrace Verification
1.  Go to **Dashboards > Kubernetes DNS**.
2.  Look at the **CoreDNS Latency** chart.
3.  You should see a massive vertical spike from <5ms to >200ms.
4.  **Alerting**: In production, this would trigger a `High DNS Latency > 100ms` alert.

### 5. Cleanup
Remove the injector to restore sanity.
```bash
kubectl delete -f manifests/network-delay.yaml
```
*Note: It may take 30s for the latency to disappear as the DaemonSet cleans up.*

## Production Lesson
High DNS latency is often invisible to CPU/Memory metrics. The CPU is fine. The Memory is fine. But the *Network* is congested. This is why **Service Level Objectives (SLOs)** must be based on Latency, not just Errors.
