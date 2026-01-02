# Lab Architecture Overview

This project simulates a complete Enterprise Kubernetes Monitoring stack on your local laptop.

```mermaid
graph TD
    subgraph "Windows Desktop"
        A[WSL2 Kernel] --> B[Docker Desktop Engine]
        B --> C[Kind Cluster]
    end

    subgraph "Kind Cluster (k8s)"
        D[Node: Control Plane]
        E[Node: Worker 1]
        F[Node: Worker 2]
    end

    subgraph "Observability"
        G[CoreDNS Pods] -- Metrics --> H[Dynatrace OneAgent]
        H -- Ingest --> I[Dynatrace SaaS]
    end

    C --> D
    C --> E
    C --> F
    E --> G
    F --> G
```

## Components
1.  **Windows Host**: The physical machine running the simulation.
2.  **Kind**: "Kubernetes in Docker". Creates containerized "Nodes".
3.  **ActiveGate / OneAgent**: The Dynatrace components that capture data.
4.  **CoreDNS**: The target application we are monitoring.
