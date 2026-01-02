# Building the CoreDNS SRE Dashboard in Dynatrace

## Dashboard Layout Strategy

### Row 1: The Executive Summary (SLOs)
*   **Availability**: `(Total - SERVFAIL) / Total * 100` (Big Number). Target: >99.99%.
*   **Latency p99**: `coredns_dns_request_duration_seconds` (Line Chart). Target: <50ms.
*   **Total Throughput**: Requests/sec (Area Chart).

### Row 2: Reliability Details
*   **Error Rate Split**: `coredns_dns_response_rcode_count_total` split by `rcode` (Stacked Bar).
    *   *Green*: NOERROR
    *   *Yellow*: NXDOMAIN (Actionable)
    *   *Red*: SERVFAIL (Critical)
    *   *Orange*: REFUSED

### Row 3: Performance Internals
*   **Cache Hit Ratio**: `hits / (hits + misses)`. (Gauge). Target: >80% for external.
*   **Upstream Latency**: `coredns_forward_request_duration_seconds`. (Line Chart).
    *   *Insight*: If Upstream is slow but CoreDNS p99 is fast, it means Caching is saving you.

### Row 4: Top Talkers (Data Explorer)
*   **Top Source IPs**: Who is querying?
*   **Top Requested Domains** (Requires CoreDNS Log Plugin + Dynatrace Log V2).
