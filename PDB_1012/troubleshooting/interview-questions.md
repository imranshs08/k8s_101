# PDB Interview Questions (Senior Level)

## Conceptual & Architecture

**Q1: What is the difference between Voluntary and Involuntary disruption? Give examples.**
*Score Guide*:
- Basic: Mentioned "Maintenance" vs "Crash".
- Senior: Correctly categorized `kubectl drain` and Cluster Autoscaler as Voluntary. Categorized Kernel Panic, OOMKill, and Node Failure as Involuntary. Explicitly stated PDB **only** protects against Voluntary disruptions.

**Q2: I have a Deployment with `replicas: 3`. I set `minAvailable: 2`. One pod is currently CrashLoopBackOff. Can I drain a node hosting one of the healthy pods?**
*Answer*: **No**.
*Why*: The PDB controller sees 2/3 healthy pods. The budget requires 2 healthy pods. Current healthy = 2. If you evict one, healthy drops to 1, violating the budget. The drain is blocked.

**Q3: Why doesn't PDB work for DaemonSets?**
*Answer*: DaemonSets ensure one pod per node. If a node is drained/terminated, the DaemonSet pod *must* be terminated. It cannot be rescheduled to another node to "maintain availability" because it is strictly node-bound.

**Q4: How does Cluster Autoscaler interact with PDB?**
*Answer*: Before scaling down a node, CA checks if any pod on that node has a restrictive PDB. If evicting a pod would violate the PDB, CA considers the node "unremovable" and will not scale it down. This can prevent cost savings if PDBs are too strict.

## Scenario Based

**Q5: We are running a legacy app that is not cloud-native and takes 5 minutes to start. It relies on a static quorum of 3 nodes. How do you configure PDB?**
*Answer*: Set `minAvailable: 3` (or whatever the quorum is, if they need 3/3). However, warn the interviewer that this blocks automated upgrades. A better answer for "High Availability" is `replicas: 4` and `minAvailable: 3`, allowing 1 disruption.

**Q6: You ran `kubectl drain` and it has been stuck for 20 minutes. What command do you run to debug?**
*Answer*:
1. `kubectl get pods` (check for crashing pods reducing the budget).
2. `kubectl get pdb` (check Allowed Disruptions).
3. `kubectl get events` (look for FailedEviction events).

**Q7: Explain the difference between `minAvailable` and `maxUnavailable`. Which one is safer for a web application?**
*Answer*:
- `minAvailable` ensures a floor of capacity (good for Quorum/Stateful).
- `maxUnavailable` ensures a ceiling of disruption (good for Web/Stateless).
- **Recommendation**: `maxUnavailable: 1` or `10%` is usually safer for web apps because it handles scaling events better (e.g., if you scale from 10 to 100 replicas, `maxUnavailable: 10%` scales the budget automatically).

**Q8: Can a PDB prevent a Spot Instance interruption?**
*Answer*: **Mostly No**. Spot interruption is an "Involuntary" event from the Kubernetes perspective (the node just dies or receives a termination notice). K8s tries to gracefully terminate, but it cannot "block" AWS/Azure from taking the node back. The PDB cannot stop the node termination.

---

## "Red Flag" Answers (Fail the Candidate)
- "PDB makes sure my app never goes down if the node crashes." (Wrong - PDB implies voluntary action only).
- "I set minAvailable to 100% to be safe." (Shows lack of operational experience with upgrades).
