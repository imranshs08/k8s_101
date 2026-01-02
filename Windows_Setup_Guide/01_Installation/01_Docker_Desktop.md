# Docker Desktop for Windows Setup Guide

## 1. Why Docker Desktop?
For local Kubernetes learning on Windows, Docker Desktop is the industry standard runtime. It manages the Linux VM (WSL2) transparently and provides the container runtime needed by **Kind**.

## 2. WSL2 vs Hyper-V
You **MUST** use WSL2 backend for performance and compatibility.

| Feature | WSL 2 (Recommended) ✅ | Hyper-V Backend ❌ |
| :--- | :--- | :--- |
| **Boot Time** | Instant (< 2s) | Slow (VM boot) |
| **Memory Usage** | Dynamic (Grows/Shrinks) | Fixed Allocation |
| **IO Performance** | Native Speed | Slow |
| **Kind Compatibility** | Excellent | Good |

## 3. Recommended Settings for PDB Labs
Since we are running **3 Kubernetes Nodes** (Simulated as Docker containers), resource allocation is critical.

1.  Open Docker Dashboard -> **Settings** -> **Resources**.
2.  **CPU**: Assign at least **4 Cores** (K8s control plane is CPU hungry).
3.  **Memory**: Assign at least **6 GB** (Each node needs ~1GB + Overhead).
    *   *Note*: In `.wslconfig` (User Profile), ensure `memory=6GB` or more if you limit WSL globally.
4.  **Disk Image Size**: Default (64GB) is usually fine.
5.  **Swap**: Keep default (Docker manages this).

## 4. Verification Commands
Open PowerShell and run:

```powershell
# Check Client/Server version and verify WSL2 backend usage
docker version

# Expected Output in 'Server' section:
#  OS/Arch:           linux/amd64
#  Context:           default
#  Experimental:      true (or false)
```

If it says `OS/Arch: windows/amd64` for Server, you are in Windows Containers mode. **Switch to Linux Containers**.

```powershell
# Verify resources available to Docker VM
docker run --rm alpine free -m
```
Ensure `Mem:` shows approx 6000MB or whatever you configured.
