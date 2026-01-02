# Lab 4: CoreDNS Pod Failure (HA Test)

## Objective
Verify that Kubernetes High Availability (HA) works. We will kill a CoreDNS pod and ensure DNS resolution continues without interruption.

## The HA Setup
By default, our `kind-multinode.yaml` cluster runs **2 replicas** of CoreDNS. This is the minimum for production.

## Steps

### 1. Check Replicas
```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```
You should see 2 pods running.

### 2. Start Continuous Traffic
Run a background loop in Terminal 1:
```bash
kubectl exec -it dns-test -- sh -c "while true; do nslookup google.com > /dev/null && echo 'OK' || echo 'FAIL'; sleep 0.5; done"
```

### 3. Kill a Pod
In Terminal 2, delete one of the CoreDNS pods.
```bash
kubectl delete pod -n kube-system -l k8s-app=kube-dns --wait=false
```
Note: We use `--wait=false` to return control immediately.

### 4. Observe Traffic
Watch Terminal 1.
*   **Ideal Result**: You see continuous "OK". No drops. The Service `kube-dns` load balances to the surviving pod.
*   **Potential Issue**: You might see 1-2 "FAIL" messages if the iptables rules update is slow (Client-side timeout).

### 5. Dynatrace View
See the `CoreDNS Pod Availability` chart drop from 100% (2 pods) to 50% (1 pod) and then recover.

## Production Lesson
Never run CoreDNS with `replicas: 1`. Even with 2 replicas, ensure `podAntiAffinity` is set so they don't land on the same node. If that node dies, you lose DNS entirely.
