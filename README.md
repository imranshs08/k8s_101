# Kubernetes 101: Enterprise Concepts & Labs

Welcome to the **Kubernetes 101** repository. This project is a curated collection of deep-dive learning modules, hands-on labs, and real-world enterprise scenarios for Kubernetes professionals.

## 🎯 Repository Goal
To bridge the gap between "Hello World" tutorials and "Production Reality". Each module focuses on a specific Kubernetes concept, explaining **WHY** it exists, **HOW** it works internally, and **WHEN** to use it in an enterprise environment (Fintech, Banking, High-Scale Tech).

## 📚 Modules

### [Module 1: PDB_1012 (Pod Disruption Budget)](PDB_1012/README.md)
**Status**: ✅ Completed
**Focus**: Availability, Upgrades, Node Draining.
**Content**:
-   **Concept**: Voluntary vs Involuntary Disruptions.
-   **Deep Dive**: How `Eviction API` and PDB Controller work.
-   **Manifests**: Best practice YAMLs for Deployments and StatefulSets.
-   **Labs**:
    -   [Lab 1: Node Drain Simulation](PDB_1012/labs/lab-1-node-drain.md)
    -   [Lab 2: Rolling Update Safety](PDB_1012/labs/lab-2-rolling-update.md)
    -   [Lab 3: Cluster Upgrade & Quorum](PDB_1012/labs/lab-3-cluster-upgrade.md)
    -   [Lab 4: Diagnosing Deadlocks](PDB_1012/labs/lab-4-pdb-failure-scenarios.md)
-   **Interview Prep**: Senior-level Q&A on PDBs.

## 🛠️ Prerequisites for Labs
-   A running Kubernetes Cluster (Kind, Minikube, EKS, AKS, GKE).
-   `kubectl` installed and configured.
-   Basic understanding of YAML and Kubernetes Resource definitions.

## 🤝 Contribution
Feel free to open PRs for typos, additional scenarios, or new modules.

---
*Maintained by Imran*