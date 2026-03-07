# Prower Progress Logs
> A living record of building, breaking, and learning on my homelab

---

## What Is This?

This repo is meant to track everything happening on **Prower**, which is my personal homelab server that runs Proxmox, where I build VMs and containers. The hope is that every change, problem, and fix gets logged here.

The goal isn't just a working homelab, rather I use it as a platform to learn **Ansible**, **Terraform**, and proper DevOps documentation practices before I do it professionally.

Not to say everything is always professional, I generally only build things on this homelab that gets **Real** use out of it for me, and if I don't use it, it gets deprecated and kept as a learning experience

---

## Why Does This Repo Exist?

Many homelab ideas or progress is easily written on disorganized napkins strewn about. I wanted to practice documenting my infrastructure changes the way a real ops team would, that being dated, changelogs, problem/resolution tracking, and postmortems for record keeping that I can learn from.

If you're in DevOps, a recruiter, or a traveller that came across this, just know everything here is real. Real problems I've encountered, real fixes, real lessons, and real time lost, ideally if you encounter any of the issues I have, this repo can hopefully help you find a solution faster than I did.

## What's Running on Prower

| Name | Type | Role |
|---|---|---|
| Infrastructure-VM-01 | VM | Primary control node for automation |
| Monitoring-VM-01 | VM | Prometheus + Grafana + Node Exporter
| CEJ-CT-01 | Container | Minecraft Server (Self-Hosted) |
| MCTestBranch-CT-02 | Container | Sandbox Test Environment for CEJ (and any other future servers) |
| Metal-VM-01 | VM | Local LLM Project |

---

## Stack & Tools

- **Hypervisor:** Proxmox VE
- **OS:** Ubuntu Server 24.04 LTS (VMs), Debian 12 (Containers)
- **Containers:** Docker + Docker Compose
- **Monitoring:** Prometheus, Grafana, Node Exporter, Alertmanager
- **Security:** UFW, Fail2ban, SSH key-only auth
- **IaC (in progress):** Terraform, Ansible
- **CI/CD (in progress):** GitHub Actions

---

## How This Repo Is Organized

Each file corresponds to a VM, Container, or topic on prower. The structure is as follows:

```
## Goal
## System Specs
## Changes (dated)
## Problems and Resolutions
## Current Status
## Notes
```

Notable files:
- **`SSH-Key-Only-Setup-Steps.md`** - A step-by-step guide for setting up key-only SSH. I made this because it took a long time to find a file that was overriding the SSHD_config file (cloudinit)
- **`POSTMORTEM.md`** - Incident log from a full server maintenance window. With real timestamped ops record.
- **`Infrastructure-VM-01.md`** - The core automation node setup, including fail2ban, UFW, and unattended upgrades.
- **`Monitoring-VM-01.md`** - Full observability stack deployment and config.

---

## Current Focus
- Beginning Terraform and Ansible integration for automated VM/CT provisioning
- Standardizing postmortem and incident response format across all services

---

## Related Repo

**[Prower](https://github.com/SpiritCreations/Prower)** - The high-level overview of the project: goals, planned stack, and architecture direction.

---

*Named after Tails "Miles" Prower from Sonic the Hedgehog. Because he's awesome*
