# CoreDNS Plugins Deep Dive

CoreDNS is a **Plugin Chain**. The order in `Corefile` matters.

## The Chain of Command
Config:
```
.:53 {
    errors
    health
    kubernetes cluster.local ...
    forward . /etc/resolv.conf
    cache 30
}
```

### 1. `errors` & `health`
*   **errors**: Logs errors to stdout. Essential for `kubectl logs`.
*   **health**: Exposes `/health` endpoint for Kubelet liveness probes.

### 2. `kubernetes` (The Brain)
Accesses the Kubernetes API (via ServiceAccount) to watch Services and Endpoints.
*   **fallthrough**: If the name ends in `cluster.local` but isn't found (e.g., `typo.cluster.local`), pass it to the next plugin. Default is to return NXDOMAIN immediate.

### 3. `forward` (The Gateway)
Any query NOT handled by `kubernetes` (e.g., `google.com`) hits this.
*   **policy**: Load balancing strategy (random, round_robin).
*   **`/etc/resolv.conf`**: Uses the Node's DNS settings.

### 4. `cache` (The Speed)
Caches answers for `TTL` seconds.
*   **Success**: Caches valid answers (A records).
*   **Denial**: Caches NXDOMAIN.
*   **Prefetch**: Automatically refreshes popular items before they expire.

## Enterprise Tuning
*   **Use `autopath`**: Optimizes the "5 search domain" problem server-side.
*   **Use `reload`**: Allows config updates without restarting.
