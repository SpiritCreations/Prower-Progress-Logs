# Incident Reports
A log of incidents, unexpected behaviors, and service degradations on Prower. For full system outages, refer to POSTMORTEM.md.

---

## Incident Template
```
## INC-XXX | [Title]
**Date:**
**Affected System:**
**Severity:**
**Detected:**
**Resolved:**

### Summary

### Timeline

### Root Cause

### Resolution

### Follow-up Actions
```

---

## INC-001 | Disk Space Alert & Silent Stack Degradation on Monitoring-VM-01

**Date:** 03-07-2026
**Affected System:** Monitoring-VM-01
**Severity:** Low - no visible service impact
**Detected:** ~2130 JST
**Resolved:** 2200 JST

### Summary
A low disk space alert fired on Monitoring-VM-01 at ~91% root partition usage. Investigation revealed three different issues that compounded into a larger problem: accumulated journal logs, orphaned Docker containers preventing stack management, and a bare metal Node Exporter service conflicting with the Dockerized version on port 9100. Alertmanager was also found to have been silently exited for approximately 3 weeks. All issues were resolved and the full stack was confirmed running as of 2200 JST.

### Timeline

| Time (JST) | Event | 
|---|---|
| ~2130 | Low disk space alert received via email |
| ~2135 | Began investigation - ran `df -h`, `du -sh`, `docker system df` |
| ~2140 | Identified journal logs, orphaned containers, and unused Docker images as primary causes |
| ~2145 | Vacuumed journal logs, freed 67.5MB |
| ~2150 | Attempted `docker compose up -d`, blocked by orphaned Alertmanager container |
| ~2153 | Removed orphaned Alertmanager container, re-attempted compose - Now blocked by Node Exporter on port 9100 |
| ~2156 | Ran `sudo ss -tlnp` - discovered bare metal `prometheus-node-exporter` holding port 9100 |
| ~2158 | Stopped and disabled bare metal Node Exporter service |
| 2200 | Full stack confirmed running: Prometheus, Grafana, Node Exporter, Alertmanager |

### Root Cause
Three compounding issues:
1. Systemd journal logs had grown unchecked with no retention cap configured
2. Manually stopped containers left orphaned, preventing Docker Compose from managing them cleanly despite `restart: unless-stopped`
3. A bare metal `prometheus-node-exporter` system service was installed at an unknown point (likely when attempted to set up email notifications) and was conflicting with the Dockerized version on port 9100

### Resolution
- Capped journal log retention at 100MB via `journalctl --vacuum-size=100M`
- Removed orphaned Alertmanager and Node Exporter containers via `docker rm`
- Stopped and disabled bare metal Node Exporter via `systemctl`
- Brought full stack back up via `docker compose up -d`
- Confirmed email alerting functioning as expected via test

### Follow-up Actions
- [ ] Configure Node Exporter down alert rule in Prometheus
- [ ] Add journal log retention cap to standard VM build checklist
- [ ] Investigate when and how bare metal Node Exporter was installed
