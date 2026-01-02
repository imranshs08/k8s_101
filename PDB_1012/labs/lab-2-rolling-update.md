# Lab 2: Rolling Update with PDB

## Objective
Observe how PDB throttles a Deployment's rolling update speed to ensure stability.

## Scenario
We will use `maxUnavailable: 1`. This ensures that during an update, we never have more than 1 pod down, keeping 3/4 replicas active at all times.

## Step-by-Step Instructions

### 1. Clean Setup
Remove previous PDB and apply the `maxUnavailable` one.
```bash
kubectl delete pdb payment-pdb-minavailable
kubectl apply -f ../manifests/pdb-maxUnavailable.yaml
```

### 2. Trigger a Rolling Update
Update the image to `nginx:1.26` to trigger a rollout.
```bash
kubectl set image deployment/payment-service payment-api=nginx:1.26
```

### 3. Watch the Rollout
Immediately watch the pods:
```bash
kubectl get pods -w
```

### 4. Analysis
You should see:
1.  Kubernetes terminates **ONLY ONE** old pod.
2.  It creates one new pod.
3.  It **waits** for the new pod to be `Ready`.
4.  Only **Match** the PDB budget (3 ready) before terminating next old pod.

**Without PDB**, if your Deployment `maxUnavailable` strategy (in .spec.strategy) was set loosely (e.g., 50%), K8s might kill 2 pods at once. The PDB acts as a **safety net** enforcing the "SLA".

### 5. Break the PDB (Optional)
Try setting `maxUnavailable: 0`.
Then trigger another rollout.
**Result**: The rollout will likely hang or be extremely slow/deadlocked depending on controller behavior, because "0 unavailable" means you can't kill anything.

### 6. Cleanup
```bash
kubectl rollout undo deployment/payment-service
```
