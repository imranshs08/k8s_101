# Lab 5: DNS Load Spike

## Objective
Simulate a Distributed Denial of Service (DDoS) or a massive scale-up event. Observe CoreDNS CPU saturation and throttling.

## Steps

### 1. Deploy the Load Generator
This deployment runs a tight loop of DNS queries.
```bash
kubectl apply -f ../manifests/dns-load-generator.yaml
```

### 2. Verify Load
Check the logs of the generator.
```bash
kubectl logs -l app=dns-load-generator --tail=10 -f
```

### 3. Observe CPU Usage
```bash
kubectl top pod -n kube-system -l k8s-app=kube-dns
```
You should see CPU usage spike.

### 4. Dynatrace Analysis
1.  Go to **Kubernetes > Workloads**.
2.  Find `coredns`.
3.  Look at **CPU Throttling**. If the pod hits its limit (e.g. 200m), Kubernetes throttles it.
4.  **Effect**: Latency increases drastically as requests queue up in the kernel.

### 5. Scale the Attack (Stress Test)
Increase the load generator to 5 replicas.
```bash
kubectl scale deployment dns-load-generator --replicas=5
```
Watch the CoreDNS latency skyrocket.

### 6. Cleanup
```bash
kubectl delete -f ../manifests/dns-load-generator.yaml
```

## Production Lesson
CoreDNS is CPU-bound. In huge clusters, enabling the `autopath` plugin or reducing `ndots` (client-side) can reduce query volume by 50%+.
