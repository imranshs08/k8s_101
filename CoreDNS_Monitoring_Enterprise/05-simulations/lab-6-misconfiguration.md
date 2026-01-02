# Lab 6: CoreDNS Misconfiguration (The Disaster)

## Objective
Simulate a "Fat Finger" configuration error that takes down DNS.

## Steps

### 1. Apply Broken Configuration
We are applying a `ConfigMap` with a syntax error (`ready_fail` plugin isn't real).
```bash
kubectl apply -f manifests/broken-coredns.yaml
```

### 2. Restart CoreDNS
ConfigMap changes don't apply instantly. Restart pods to pick it up.
```bash
kubectl rollout restart deployment coredns -n kube-system
```

### 3. The Outage
Check the pod status.
```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```
**Status**: `CrashLoopBackOff` or `Error`.
**Logs**:
```bash
kubectl logs -n kube-system -l k8s-app=kube-dns
# "plugin/ready_fail: setup: not found"
```

### 4. Verify Impact
Your test app is now blind.
```bash
kubectl exec -it dns-test -- nslookup google.com
# ";; connection timed out; no servers could be reached"
```

### 5. Recovery
Apply the known-good config.
```bash
kubectl apply -f manifests/coredns-configmap.yaml
kubectl rollout restart deployment coredns -n kube-system
```

## Production Lesson
**Never** edit CoreDNS ConfigMaps directly in production.
1.  Use a linter / validater in CI/CD.
2.  Use a canary rollout (update 1 pod first).
3.  Have a `restore` script ready.
