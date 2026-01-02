# Step-by-Step Guide: Building the CoreDNS Dashboard

**Target Audience**: Beginners to Dynatrace.
**Goal**: Build a visual dashboard to monitor the health of your Kubernetes DNS.

---

## Prerequisite: Open Data Explorer
All charts are built using the **Data Explorer**.
1.  Log in to your Dynatrace environment (`https://fqj51381...`).
2.  In the left sidebar menu, find **Observe and Explore** -> **Data Explorer**.
3.  You will see a "Build your query" interface.

---

## Chart 1: DNS Availability (The "Big Number")
**Goal**: Show a single percentage representing success rate.

1.  **Metric**: Click the text box and type `coredns_dns_response_rcode_count_total`. Select it.
2.  **Aggregation**: Ensure it says `Sum`.
3.  **Visualization Settings**:
    *   Click **Visualization** (Graph icon) -> Select **Single Value**.
4.  **Advanced Mode (Math)**:
    *   We need to calculate `(Success / Total)`.
    *   Click the **Code** tab (next to Build).
    *   Paste this DQL (Dynatrace Query Language) snippet if available, OR use the **Filter** button in Build mode:
        *   Filter by `rcode` != `SERVFAIL`.
5.  **Pin it**: Click **Pin to Dashboard** -> **Create new dashboard** -> Name it "CoreDNS Overview".

---

## Chart 2: DNS Latency (Line Chart)
**Goal**: See how long requests take over time.

1.  Open **Data Explorer** again.
2.  **Metric**: Search for `coredns_dns_request_duration_seconds`.
3.  **Aggregation**: Select `Average` or `Percentile` -> `99th percentile` (if available).
4.  **Split by**: None (to see global average) or `pod` (to see slow pods).
5.  **Visualization**: Select **Graph** (Line Chart).
6.  **Pin it**: Click **Pin to Dashboard** -> Select "CoreDNS Overview".

---

## Chart 3: Error Rate by Type (Stacked Bar)
**Goal**: distinguish between `NXDOMAIN` (Bad URL) and `SERVFAIL` (Server Broken).

1.  **Metric**: `coredns_dns_response_rcode_count_total`.
2.  **Split by**: Click **Add dimension** -> Select `rcode`.
3.  **Visualization**: Select **Stacked Graph** or **Pie Chart**.
4.  **Result**: You should see colored bars.
    *   Hover to see "NOERROR", "NXDOMAIN", etc.
5.  **Pin it**.

---

## Chart 4: Traffic Volume (Area Chart)
**Goal**: See the load on your DNS servers.

1.  **Metric**: `coredns_dns_request_count_total`.
2.  **Split by**: `proto` (udp vs tcp).
3.  **Visualization**: **Area Graph**.
4.  **Pin it**.

---

## Finalizing the Dashboard
1.  Go to **Dashboards** in the side menu.
2.  Click **CoreDNS Overview**.
3.  Click **Edit** (Pencil icon).
4.  Drag and drop the tiles to arrange them logically:
    *   **Top Row**: Availability & Response Time (The most important things).
    *   **Bottom Row**: Throughput & Errors.
5.  Click **Done**.

🎉 **Success!** You now have a professional SRE dashboard.
