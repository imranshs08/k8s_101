# Lab 1: Normal DNS Traffic Baseline

## Objective
Establish a "Golden Signal" baseline for DNS performance in a healthy cluster. Before you can debug "slow", you must know what "fast" looks like.

## Architecture
```
[ Test Pod ] --UDP:53--> [ Service: kube-dns ] --> [ Pod: coredns ]
```

## Steps

### 1. Deploy the Test App
*Run from Project Root*:
```bash
kubectl apply -f manifests/test-app.yaml
```

### 2. Generate "Good" Traffic
Exec into the pod and run manual queries.
```bash
kubectl exec -it dns-test -- nslookup google.com
kubectl exec -it dns-test -- nslookup kubernetes.default.svc.cluster.local
```
**Expected Output:**
```text
Server:     10.96.0.10
Address:    10.96.0.10#53

Name:       google.com
Address:    142.250.183.46
```

### 3. Observe in Dynatrace
1.  Go to **Kubernetes** Application in Dynatrace.
2.  Select your Cluster.
3.  Navigate to **Analyzation > DNS**.
4.  Look for:
    *   **Throughput**: Should see a small blip for your queries.
    *   **Response Time**: Should be < 2ms for cached queries.
    *   **Errors**: 0.

## Production Lesson
In a real environment, "Normal" is a steady hum of internal service discovery traffic. If you see sudden drops in traffic, it might not mean less load—it might mean the network is down and requests aren't even reaching CoreDNS.
