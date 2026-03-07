# Infrastructure-VM-01

## Goal
Create a stable, hardened control VM to serve as the primary automation node for Prower. This VM is intended to be the first thing up and last thing down, all automation and provisioning scripts live here.

---

## System Specs

| Property | Value |
|---|---|
| **Type** | Proxmox VM |
| **OS** | Ubuntu Server 24.04.3 LTS |
| **CPU** | 2 Cores (1 Socket) |
| **Memory** | 2 GiB |
| **Disk** | 20GB |

---

## Changes

**12-10-2025 - Initial Build**
- Created VM running Ubuntu Server 24.04.3
- Installed Docker, Git, SSH
- Generated and configured SSH key pair
- Configured SSH key-only authentication by editing:
  - `/etc/ssh/sshd_config`
  - `/etc/ssh/sshd_config.d/50-cloud-init.conf`
- Added weekly maintenance reboot via systemd timer to ensure a clean stack recovery

**12-13-2025 - Security Hardening**
- Installed and configured `unattended-upgrades`
  - Enabled daily security updates
  - Disabled automatic reboot (weekly reboot handled by systemd timer)
  - Verified via dry run

**12-16-2025 - Firewall & Intrusion Prevention**
- Installed UFW
  - Default deny inbound, allow outbound
  - Explicitly allowed SSH (22)
  - Enabled and verified firewall over existing SSH session
- Installed and configured Fail2ban
  - SSH jail only
  - Backend: systemd
  - 5 failed attempts within a 10 minute window triggers a 1 hour ban
  - LAN IP ranges whitelisted to prevent admin lockout
  - Verified service startup and active SSH jail
 
---

## Problems & Resolutions

**SSH key-only auth not enforcing**
- `cloud-init.conf` was overriding the `PasswordAuthentication no` policy set in `sshd_config`
- Fixed by applying the policy change in both config files

**Fail2ban failed to start on initial setup**
- `jail.local` contained parameters outside of valid INI headers, causing parse error
- Fixed by recreating `jail.local` using heredoc to avoid copy/paste corruption

---

## Current Status
VM is stable and running. Core hardening complete (UFW, Fail2ban, SSH key-only, unattended upgrades). Currently serving as primary control node for Prower. Ansible and Terraform integration is next planned step.
