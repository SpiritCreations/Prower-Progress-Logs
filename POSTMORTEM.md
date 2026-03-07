# Postmortems

A log of significant incidents and full or partial system outages on Prower. For smaller incidents and unexpected behaviors, see Incident-Reports.md.

---

## Postmortem Template
```
## PM-XXX | [Title]
**Date:**
**Affected Systems:**
**Severity:** High / Medium / Low
**Outage Start:**
**Outage End:**
**Total Downtime:**

### Summary

### Timeline

### Root Cause

### Resolution

### Lessons Learned

### Follow-up Actions
```

---

## PM-001 | Planned Maintenance Window - GPU Passthrough & Full Service Restart

**Date:** 01-19-2026
**Affected Systems:** All Prower Services (CEJ, Infrastructure-VM-01, Monitoring-VM-01, Metal-01)
**Severity:** Low - planned maintenance, no unplanned data loss
**Outage Start:** 1030 JST
**Outage End:** 1052 JST
**Total Downtime:** ~22 minutes

### Summary
Planned maintenance window to add GPU passthrough to Metal-01 via Proxmox raw device passthrough. Required a full node shutdown and restart. All services were brought back online successfully, however auto-start did not function as expected and a manual launch was required for the CEJ Minecraft server.

### Timeline

| Time (JST) | Event |
|---|---|
| 1030 | Initial node shutdown |
| 1034 | Node powered on |
| 1036 | Second shutdown triggered - Proxmox pushing updates to GRUB and initramfs |
| 1038 | Node powered on |
| 1042 | GPU detected and manually added to Metal-01 VM via Proxmox raw device passthrough |
| 1042 | Metal-01 VM started to verify GPU passthrough |
| 1044 | NVIDIA drivers installed to verify GPU detection via `nvidia-smi` | 
| 1046 | NVIDIA drivers installed, Metal-01 rebooted |
| 1047 | CEJ, Infrastructure-VM-01, Monitoring-VM-01 services rebooting |
| 1049 | Monitoring-VM-01 restored full functionality |
| 1049 | Infrastructure-VM-01 restored full functionality |
| 1052 | CEJ restored full functionality - required manual launch |

### Root Cause
Planned maintenance - not an unplanned incident. The only unexpected behavior was the auto-start script for the CEJ Minecraft server failing to launch the server automatically on boot.

### Resolution
- GPU passthrough to Metal-01 completed successfully and verified via `nvidia-smi`
- All services brought back online manually after auto-start failure
- CEJ server launched manually via:
  ```bash
  java -Xmx3G -jar fabric-server-launch.jar nogui
  ```

  ### Lessons Learned
  - Auto-start for CEJ did not function as expected - this should not be assumed reliable for future maintenance windows
  - A second unplanned shutdown occurred due to Proxmox pushing GRUB and initramfs updates, which extended the maintenance window slightly. Worth checking for pending Proxmox updates before future maintenance to avoid surprises or outage extensions.
 
  ### Follow-up Actions
  - [ ] Fix CEJ auto-start script - migrate from current setup to `screen` or `tmux` for reliable background process management
