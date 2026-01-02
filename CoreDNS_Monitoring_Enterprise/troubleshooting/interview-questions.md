# SRE Interview Questions: Kubernetes DNS & CoreDNS

## Beginner
1.  **What is the role of CoreDNS in Kubernetes?**
    *   *Ans*: Service Discovery. Resolving Service names to ClusterIPs.
2.  **How do Pods know where to send DNS queries?**
    *   *Ans*: Kubelet configures `/etc/resolv.conf` with the `nameserver` set to the `kube-dns` Service IP (usually 10.96.0.10).
3.  **What is the difference between ClusterIP and Headless Service resolution?**
    *   *Ans*: ClusterIP returns a single Virtual IP. Headless returns the list of Pod IPs directly.

## Intermediate
4.  **How would you debug intermittent DNS failures that only happen on one Node?**
    *   *Ans*: Check node-local logs, conntrack table (`dmesg | grep conntrack`), and CNI status. It's likely a networking issue on that specific host.
5.  **Explain the `ndots:5` problem.**
    *   *Ans*: Default setting causes every query (e.g., `google.com`) to be suffixed with 5 search domains first (`google.com.default.svc...`, `google.com.svc...`), amplifying traffic by 5-6x.
6.  **What metric tells you CoreDNS is overwhelmed?**
    *   *Ans*: `coredns_dns_request_duration_seconds` (Latency) and CPU throttling stats.

## Advanced (Senior SRE)
7.  **We are seeing high `SERVFAIL` responses. What is your investigation process?**
    *   *Ans*: `SERVFAIL` means CoreDNS "gave up".
        1.  Check Upstream connectivity (can CoreDNS reach 8.8.8.8?).
        2.  Check recursion depth.
        3.  Check for loops (CoreDNS forwarding to itself).
8.  **Design a High Availability DNS architecture for a 1000-node cluster.**
    *   *Ans*:
        *   Scale CoreDNS (HPA based on CPU/QPS).
        *   Use `NodeLocal DNSCache` (DaemonSet) to offload UDP connection tracking from the Node and reduce latency.
        *   Use strict `podAntiAffinity` to spread CoreDNS pods across Availability Zones.
9.  **A developer claims "DNS is slow". How do you prove it's the Network vs the App?**
    *   *Ans*: Use distributed tracing (Dynatrace Service Flow) to see the "DNS Lookup" time in the span. Correlate with CoreDNS p99 metrics. If CoreDNS p99 is 2ms but App sees 2s, it's the Node Network or Client Library timeout.

## Scenario
> **"During a deployment, 50% of requests to `payment-service` fail with 'Unknown Host'. What do you check?"**
> *   Did the Service object get deleted?
> *   Are the Endpoints populated? (`kubectl get endpoints`)
> *   Is CoreDNS crashing?
> *   Is there a network partition?
