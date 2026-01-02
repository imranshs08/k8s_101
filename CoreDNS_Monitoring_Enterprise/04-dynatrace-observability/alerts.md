# Step-by-Step Guide: Configuring CoreDNS Alerts

**Target Audience**: Beginners to Dynatrace.
**Goal**: Get notified via Email or Slack when DNS breaks.

---

## The Concept
Dynatrace alerts are called **Anomaly Detection** rules. There are two ways to do this:
1.  **Metric Events**: Simple Thresholds (e.g., "Limit exceeded").
2.  **Davis AI**: Automatic baselining (Advanced).

**We will use Metric Events (Method 1)** because they are predictable for labs.

---

## Alert 1: High Error Rate (The "Recall" Alert)
**Scenario**: If `SERVFAIL` happens more than 0 times, tell me.

1.  **Go to Settings**:
    *   Left Menu -> **Settings**.
    *   Scroll down to **Anomaly Detection** -> **Metric Events**.
2.  **Add New Rule**:
    *   Click **Add metric event**.
3.  **Step 1: Metric Definition**:
    *   **Summary**: "CoreDNS High Failure Rate".
    *   **Metric key**: Type `coredns_dns_response_rcode_count_total`.
    *   **Dimension filter**: Click **Add filter** -> `rcode` -> `SERVFAIL`.
        *   *Why?* We don't care about `NXDOMAIN` (User error) for this alert, only Server Failures.
4.  **Step 2: Monitoring Strategy**:
    *   **Model type**: Select **Static threshold**.
    *   **Threshold**: `> 0` (Strict) or `> 5` (Lenient/Production).
    *   **Observation period**: "any 3 minutes out of 5 minutes".
5.  **Step 3: Event Description**:
    *   **Event title**: "CoreDNS is failing to resolve names".
    *   **Severity**: **Error** or **Availability**.
6.  **Save**: Click **Save changes**.

---

## Alert 2: High Latency (The "Slow" Alert)
**Scenario**: DNS is working, but it's too slow (>50ms).

1.  **Add New Rule** (Metric Events page).
2.  **Step 1: Metric Definition**:
    *   **Summary**: "CoreDNS Latency Spike".
    *   **Metric key**: `coredns_dns_request_duration_seconds`.
    *   **Aggregation**: `Average` (or `90th percentile` if allowed).
3.  **Step 2: Monitoring Strategy**:
    *   **Model type**: **Static threshold**.
    *   **Threshold**: `> 0.05` (Which means 50ms. Remember metrics are in seconds!).
    *   **Alert if**: "Metric is above threshold".
4.  **Step 3: Event Description**:
    *   **Event title**: "DNS Resolution is Slow (>50ms)".
    *   **Severity**: **Performance**.
5.  **Save**.

---

## How to Test It?
1.  Go to **Lab 6 (Misconfiguration)** or **Lab 7 (Cut Cable)**.
2.  Trigger the failure.
3.  Wait ~5 minutes.
4.  Look at the **Problems** page in the left menu (Red warning icon).
5.  You should see your new alert active!
