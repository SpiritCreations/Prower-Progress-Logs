# Monitoring-VM-01

## Goal 
Create a dedicated VM responsible for monitoring all VMs and containers on prower, And consolidating all metrics into a one-glance dashboard

---

## System Specs

| Property | Value | 
|---|---|
|**Type** | Proxmox VM |
|**OS** | Ubuntu Server 24.04.3 LTS |
|**CPU** | 1 Core (1 Socket) |
|**Memory**| 2 GiB |
|**Disk** | 20GB |

---

## Changes

**12-10-2025 - Initial Build**
- Created VM running Ubuntu Server 24.04.3
- Installed Docker Engine
- Installed Docker Compose (Ubuntu repo version did not include this by default, had to be manually installed)
- Generated and Configured SSH key pair
- Configured SSH key-only authentication by editing:
  -`/etc/ssh/sshd_config`
  -`/etc/ssh/sshd_config.d/50-cloud-init.conf`
- Added weekly maintenance reboot via systemd timer, offset 15 minutes after Infrastructure-VM-01 to ensure it restarts after the control node is back up.
- Created monitoring stack directory structure:
  - `/opt/monitoring/{prometheus,grafana}`
  - `/opt/monitoring/grafana/data`
  - `/opt/monitoring/prometheus/rules`
  - `/opt/monitoring/prometheus/alertmanager/templates`
- Created stack config files via heredoc (see Notes):
  -`prometheus.yml`
  -`compose.yaml`
  -`alertmanager.yml`
- Stack includes: Prometheus, Grafana, Node Exporter, Alertmanager

**12-13-2025 - Security Hardening**
- Installed and configured `unattended-upgrades`
  - Enabled daily security updates
  - Disabled automatic reboot (weekly reboot handled by systemd timer)
  - Verified via dry run

 **12-16-2025 - Firewall & Intrusion Prevention**
 - Installed UFW
  - Default deny inbound, allow outbound
  - Explicitly allowed:
    - SSH (22)
    - Grafana (3000)
    - Prometheus (9090)
    - Node Exporter (9100)
  - Enabled and verified firewall over existing SSH session
- Installed and configured Fail2ban
  - SSH jail only
  - Backend: systemd
  - 5 failed attempts within a 10 minute window triggers a 1 hour ban
  - LAN IP ranges whitelisted to prevent admin lockout
  - Verified service startup and active SSH jail

 **03-07-2026 - Disk Space Cleanup & Stack Recovery**
 - - Disk usage was at 91% on root partition, triggering a low disk space alert
 - - Vacuumed systemd journal logs, capping retention at 100MB - freed 67.5MB
 - - Removed orphaned Alertmanager and Node Exporter containers that were blocking `docker compose up -d`
 - - Discovered and disabled a bare metal `prometheus-node-exporter` system service conflicting with the Dockerized Node Exporter on port 9100:
 - ```bash
   sudo systemctl stop prometheus-node-exporter
   sudo systemctl disable prometheus-node-exporter
   ```
   - Brought full stack back up cleanly via `docker compose up -d`
   - Confirmed Alertmanager email alerting is fully functional
   - Full stack confirmed running: Prometheus, Grafana, Node Exporter, Alertmanager

---

## Problems & Resolutions

**YAML corruption when editing via SSH**
- Pasting config content into Vim over SSH caused formatting corruption
- Fixed by writing all config files using heredoc (`<<EOF`) instead

**Docker Compose not found**
- The Ubuntu repo version of Docker did not include Compose by default
- Fixed by manually installing standalone Docker Compose

**`-d` flag not recognized on `docker compose up`**
- Was caused by using the wrong Compose installation
- Resolved once standalone Docker Compose was properly installed

**SSH key-only auth not enforcing**
- `cloud-init.conf` was overriding the `PasswordAuthentication no` policy set in `sshd_config`
- Fixed by applying the policy change in both config files

**Disk space alert firing at 91% usage**
- Root cause was a combination of accumulated journal logs, orphaned Docker containers, and a conflicting bare metal Node Exporter service
- Resolved via journal vacuum, container cleanup, and disabling the conflicting system service
- See 03-07-2026 changes entry for full details

**Alertmanager silently exited and not recovering**
- Alertmanager had been in an exited state for ~3 weeks despite `restart: unless-stopped` being set
- Root cause was an orphaned container blocking Compose from managing it
- Resolved by removing the dead container and running `docker compose up -d`

---

## Current Status
VM is stable and running. Hardening complete (UFW, Fail2ban, SSH key-only, unattended upgrades). Full monitoring stack running cleanly: Prometheus, Grafana, Node Exporter, and Alertmanager. Email alerting confirmed functional. Node Exporter down alerting rule not yet configured - tracked in ToDo

---

## Notes

**Unattended Upgrades - useful log locations for troubleshooting:**
```
/var/log/unattended-upgrades/unattended-upgrades.log
/var/log/unattended-upgrades/unattended-upgrades-dpkg.log
```

**Heredoc was used to create all config files** due to SSH paste corruption when using Vim. if you need to recreate or modify config files over SSH, use heredoc to avoid formatting issues:
```bash
cat <<EOF > /path/to/file.yml
# your config here
EOF
```

**Bare metal Node Exporter conflicts** - A `prometheus-node-exporter` system service was found running natively on the VM, conflicting with the Dockerized version on port 9100. It has been stopped and disabled. If Node Exporter ever fails to bind to 9100 in the future, check for this service:
```bash
sudo ss -tlnp | grep 9100
sudo systemctl status prometheus-node-exporter
```
