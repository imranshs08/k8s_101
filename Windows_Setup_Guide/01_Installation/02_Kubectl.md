# Kubectl Installation on Windows

## 1. Installation Methods

### Option A: Chocolatey (Recommended)
If you have Chocolatey installed:
```powershell
choco install kubernetes-cli -y
```

### Option B: Scoop
```powershell
scoop install kubectl
```

### Option C: Direct Binary
1.  Download `kubectl.exe` from official Kubernetes release page.
2.  Place it in a folder e.g., `C:\k8s\bin`.
3.  Add `C:\k8s\bin` to your System `PATH` environment variable.

## 2. Version Compatibility
*   PDB API `policy/v1` became stable in **v1.21**.
*   Ensure your client (`kubectl`) is at least **v1.25+** to avoid warning messages and support newer debug features (`kubectl debug`).

## 3. Verification
```powershell
kubectl version --client --output=yaml
```

**Verify PATH:**
```powershell
Get-Command kubectl
# Should return the path, e.g., C:\ProgramData\chocolatey\bin\kubectl.exe
```

## 4. Shell Autocompletion (PowerShell)
(Optional but highly recommended)

```powershell
kubectl completion powershell | Out-String | Invoke-Expression
```
To make it permanent, add the above line to your PowerShell `$PROFILE`.
