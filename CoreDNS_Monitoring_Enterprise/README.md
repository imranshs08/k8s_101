# CoreDNS in Kubernetes: Enterprise Observability & Troubleshooting
> **A complete SRE guide to monitoring, debugging, and simulating DNS failures in Production.**

---

## 📚 Introduction

### What is Kubernetes DNS?
In Kubernetes, **DNS (Domain Name System)** is the nervous system of cluster networking. Every time a pod tries to connect to a service (`payment-service`), a database, or an external API (`api.stripe.com`), it relies on DNS to resolve the hostname to an IP address.

If DNS fails, **everything fails**. Your applications cannot talk to each other, even if the network and pods are perfectly healthy.

### Why is DNS a top cause of production outages?
> *"It's always DNS."* — System Administrator Adage

1.  **Invisible Dependency**: Developers assume DNS "just works" and don't build retry logic for resolution failures.
2.  **Concurrency Limits**: CoreDNS is often CPU-bound. A sudden spike in pod sealing (HPA) or a misconfigured app loop can DDOS the DNS server.
3.  **Latency Sensitive**: A 500ms DNS delay adds 500ms to users' page load time.
4.  **Black Box**: Without observability, "Application Timeout" errors masquerade as network issues, when they are actually DNS timeouts.

### Why CoreDNS?
**CoreDNS** is the default DNS server for Kubernetes. It is flexible, plugin-based, and written in Go. However, in large enterprises, standard configurations often fail under load.

**This project is designed to turn you into a DNS expert who can:**
-   **Visualize** invisible DNS traffic with Dynatrace.
-   **Simulate** real-production disasters (Latency, Packet Loss, Panic).
-   **Troubleshoot** like a Senior SRE.

---

## 🏗️ Enterprise DNS Architecture in Kubernetes

Understanding how packets flow is critical for debugging.

### The DNS Resolution Flow

```ascii
+-------------------+       +-----------------------+       +-------------------------+
|    Application    |       |      CoreDNS Pod      |       |      Upstream DNS       |
|       Pod         | ----> |    (Service IP)       | ----> |     (Cloud/Corp DNS)    |
+-------------------+       +-----------------------+       +-------------------------+
       |                           ^    |                             ^
       | /etc/resolv.conf          |    | Cache                       |
       +---------------------------+    +-----------------------------+
             UDP :53 (mostly)
```

1.  **Pod Level**: The `kubelet` injects `/etc/resolv.conf` into every pod. It points `nameserver` to the ClusterIP of `kube-dns`.
2.  **Service Discovery**:
    *   **Internal**: `payment-service.default.svc.cluster.local` -> Resolves to Service IP.
    *   **Headless**: `kafka-0.broker.default.svc.cluster.local` -> Resolves to Pod IP.
3.  **Forwarding**: If CoreDNS doesn't know the domain (e.g., `google.com`), it forwards the query to the Upstream DNS (the node's DNS).

### Common Failure Points
*   **Conntrack Exhaustion**: Too many UDP packets from a single node can hit Linux kernel limits.
*   **CoreDNS CPU Throttling**: High QPS (Queries Per Second) forces CoreDNS to drop packets.
*   **NDOTS Issues**: Default `ndots:5` settings cause 5x more queries than necessary for external domains.

---

## 🧠 CoreDNS Deep Dive

CoreDNS is a **Chain of Plugins**. Every query passes through a pipeline of middleware.

### Core Architecture
A request enters CoreDNS and is processed by plugins defined in `Corefile`.

**Simplified Request Lifecycle:**
1.  **Metric Plugin**: Records that a query arrived.
2.  **Cache Plugin**: Checks if the answer is already known (TTL).
3.  **Kubernetes Plugin**: Checks if it matches a Service or Pod in the cluster.
4.  **Forward Plugin**: If no match, sends it to the internet (8.8.8.8 etc).
5.  **Log/Errors**: Logs failures.

### Key Plugins for SREs
| Plugin | Role | SRE Relevance |
| :--- | :--- | :--- |
| `kubernetes` | Answers cluster queries | The source of truth for Service Discovery. |
| `prometheus` | Exposes metrics on :9153 | **CRITICAL**. This enables Dynatrace monitoring. |
| `forward` | Sends traffic upstream | Latency here means your Node/ISP is slow. |
| `cache` | Caches responses | High cache hit rate = Healthy cluster. |
| `errors` | Logs failed queries | Shows up in Dynatrace Logs. |

---

## 📊 CoreDNS Metrics (Mission Critical)

You cannot manage what you don't measure. In Dynatrace, we track these distinct signals:

### 1. Request Count (Throughput)
*   **Metric**: `coredns_dns_request_count_total`
*   **Why**: Sudden spikes indicate a misconfigured app (infinite loop) or a DDoS.
*   **Labels**: `type` (A, AAAA, SRV), `proto` (udp/tcp).

### 2. Request Duration (Latency)
*   **Metric**: `coredns_dns_request_duration_seconds`
*   **Why**: **The Golden Signal**. High latency kills app performance.
*   **Threshold**: > 10ms is concerning. > 100ms is an outage.

### 3. Response Codes (Errors)
*   **Metric**: `coredns_dns_response_rcode_count_total`
*   **Focus**:
    *   `NXDOMAIN`: Domain not found. (Apps calling wrong URL?)
    *   `SERVFAIL`: DNS server failed to answer. (CoreDNS is broken/overloaded).
    *   `REFUSED`: Policy blocked the request.

### 4. Cache Efficiency
*   **Metric**: `coredns_cache_hits_total` / `coredns_cache_misses_total`
*   **Why**: Low cache hit ratio means higher load on upstream DNS and higher latency.

---

## 🦅 Dynatrace Integration Strategy

To monitor this in production, we deploy the **Dynatrace Operator**, which automatically scrapes the CoreDNS Prometheus endpoint.

### Dynakube Configuration
We will use a standard `Dynakube` custom resource that enables:
*   **ActiveGate**: For routing and API monitoring.
*   **Data Ingest**: Generic metric ingestion from Prometheus.

### SRE Dashboards
Our project includes JSON definitions for a **"Kubernetes DNS Overview"** dashboard:
1.  **Cluster Health**: Overall DNS success rate (Target: 99.99%).
2.  **Latency Heatmap**: Which pods are experiencing slow resolution?
3.  **Usage by Namespace**: Who is spamming DNS?
4.  **Error Breakdown**: Top NXDOMAIN targets (e.g., `db-prod` vs `db-prod.svc`).

---

## 🧪 Simulation Labs Overview

This project builds a **Lab Environment** on your machine to practice disaster scenarios.

| Lab | Scenario | Goal |
| :--- | :--- | :--- |
| **Lab 1** | **Normal Traffic** | Establish a baseline. Understand "good". |
| **Lab 2** | **Latency Injection** | use `tc` or `toxiproxy` to delay UDP packets. Watch apps slow down. |
| **Lab 3** | **NXDOMAIN Storm** | Deploy a buggy app that queries random domains. Trigger alerts. |
| **Lab 4** | **Pod Failure** | `kill` CoreDNS pods. Verify High Availability (HA) configuration. |
| **Lab 5** | **Load Spike** | Use a load generator to hammer DNS. Observe CPU throttling. |
| **Lab 6** | **Misconfiguration** | Break the `Corefile`. Observe partial outages. |

---

## 📂 Project Structure

```text
CoreDNS_Monitoring_Enterprise/
├── 00-prerequisites/       # Concepts you MUST know before starting
├── 01-installation/        # Setup Kind, Docker, and Dynatrace
├── 02-cluster-setup/       # Multi-node Kind cluster & CoreDNS tuning
├── 03-coredns-internals/   # Deep dives into plugins and architecture
├── 04-dynatrace-observability/ # Dashboarding, Alerting, & Metrics
├── 05-simulations/         # The 6 Hands-on Labs
├── manifests/              # YAMLs for simulation apps & generators
└── troubleshooting/        # Production playbooks & Interview prep
```

## ⚠️ DO's and DON'Ts

### ✅ DO
*   **Monitor DNS as a Tier-0 Service**. It is as important as your database.
*   **Set Resource Requests/Limits**. CoreDNS needs CPU.
*   **Use NodeLocal DNSCache** in large clusters to offload traffic.

### ❌ DON'T
*   **Use `ndots:5` blindly**. It kills performance.
*   **Ignore `SERVFAIL`**. It usually means upstream network issues.
*   **Run single-replica CoreDNS**. Always run at least 2 replicas (anti-affinity favored).

---
*Built for the Enterprise SRE - 2026 Edition*
