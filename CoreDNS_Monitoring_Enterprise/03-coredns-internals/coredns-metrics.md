# CoreDNS Metrics Reference within Dynatrace

Dynatrace scrapes the `/metrics` endpoint (port 9153).

## Key Metrics Table

| Metric Name | Type | Description | SRE Interpretation |
| :--- | :--- | :--- | :--- |
| `coredns_dns_request_count_total` | Counter | Total queries received. | **Spike?** DDoS or loop. **Drop?** Network issue. |
| `coredns_dns_request_duration_seconds` | Histogram | Time to process (excluding network). | **High?** node CPU/Memory pressure. |
| `coredns_dns_response_rcode_count_total` | Counter | Result codes (NOERROR, NXDOMAIN). | **High SERVFAIL?** Critical outage. |
| `coredns_cache_hits_total` | Counter | Queries answered from RAM. | **Low?** Inefficient caching, high upstream latency. |
| `coredns_forward_request_duration_seconds`| Histogram | Time waiting for Upstream DNS. | **High?** The problem is OUTSIDE K8s (ISP/Cloud). |

## Dimensions (Filtering)
*   `server`: The server block (usually `dns://:53`).
*   `zone`: The domain zone (`cluster.local.` vs `.`).
*   `proto`: `udp` or `tcp`.
*   `type`: `A`, `AAAA`, `SRV`, `PTR`.
