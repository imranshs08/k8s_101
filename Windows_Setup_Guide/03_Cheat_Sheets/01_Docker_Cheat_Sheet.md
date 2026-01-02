# Docker Commands Cheat Sheet (for K8s Users)

## Container Management
| Action | Command | Purpose |
| :--- | :--- | :--- |
| **List Running** | `docker ps` | View running Kind nodes (containers). |
| **Resource Usage** | `docker stats` | View CPU/RAM usage of nodes. Critical if laptop lags. |
| **Stop Node** | `docker stop <container-id>` | Simulate a hard "Node Crash" (Involuntary disruption). |
| **Restart Node** | `docker start <container-id>` | Simulate node recovering from reboot. |
| **Nuke All** | `docker system prune -a` | **DANGER**: Cleans up all stopped containers/images. Use only if broken. |

## Network Debugging
| Action | Command |
| :--- | :--- |
| **Inspect Net** | `docker network inspect kind` | See IP addresses of nodes. |
