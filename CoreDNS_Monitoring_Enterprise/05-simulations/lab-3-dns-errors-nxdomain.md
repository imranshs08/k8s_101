# Lab 3: Troubleshooting NXDOMAIN Storms

## Objective
Simulate a "noisy neighbor" application that spams the DNS server with invalid requests, causing a flood of `NXDOMAIN` errors.

## Scenario
A developer misspelled a database URL in their code config: `postgres-prod` instead of `postgres-prod.default.svc.cluster.local`. This forces CoreDNS to search *every* search domain (`default.svc.cluster.local`, `svc.cluster.local`, `cluster.local`, `ec2.internal`...) before failing.

## Steps

### 1. Trigger the Storm
We will manually spam bad requests.
```bash
kubectl exec -it dns-test -- sh -c "for i in \$(seq 1 100); do nslookup bad-domain.local; done"
```

### 2. Analyze in Dynatrace
1.  Navigate to **Metrics**.
2.  Search for `coredns_dns_response_rcode_count_total`.
3.  Filter by `rcode="NXDOMAIN"`.
4.  **Result**: You should see a spike of ~100 errors.

### 3. Investigation
In Dynatrace, how do you find *who* caused it?
1.  Drill down by `pod` or `client_ip`.
2.  You will see `dns-test` as the top contributor.

## Production Lesson
High NXDOMAIN counts are not harmless.
1.  **Performance**: They waste CPU processing failed lookups.
2.  **Cost**: If you use AWS Route53Resolver, you pay for every query forwarded upstream.
3.  **Security**: It could be an attacker ensuring they aren't noticed (Service Discovery recon).
