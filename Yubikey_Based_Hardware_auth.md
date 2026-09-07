# Hardware Authentication  YubiKey 5 Rollout

> Tiered, hardware-backed authentication across Prower using 3x YubiKey 5 devices.

*Related: [SSH Key Only Setup steps.md](<SSH Key Only Setup steps.md>) for the underlying SSH key-only baseline this builds on top of.*

---

## Goal

Replace password and software-only key authentication across the homelab with hardware-backed FIDO2 credentials, modeled on real privileged access management (PAM/PIM) patterns, separating standard workload access from privileged infrastructure access, with a documented recovery path if a key is lost.

---

## System Specs

- **3x Yubico YubiKey 5 Series** (USB-A/C, FIDO2/WebAuthn, PIV, OpenPGP, OATH capable)
- **Client:** Arch Linux (main PC), OpenSSH 10.5p1, `libfido2`, `pam-u2f`, `yubikey-manager`
- **Targets:** Proxmox VE host (Prower), 4 active VMs/CTs, GitHub, Google
- **Display manager:** SDDM (HyDE desktop)

---

## Key Roles

| Key | Nickname | Scope | Physical location |
|---|---|---|---|
| Key 1 | Daily driver | Arch PC login/sudo, git commit signing, GitHub, Google, SSH to all workload VMs/CTs | Daily carry |
| Key 2 | `prower` (infra) | Proxmox host root SSH only | Reserved for privileged/homelab admin work |
| Key 3 | Backup | Registered as a secondary credential everywhere Key 1 and Key 2 are used | Cold storage |

**Design rationale:** VMs/CTs are treated as standard-tier (a compromised workload has limited blast radius). The Proxmox host itself is treated as privileged-tier, since compromising the hypervisor compromises every guest on it.

---

## Changes (dated)

### 2026-08-17  Initial FIDO2 SSH key generation (Key 1)
- Installed `libfido2` (missing dependency caused first `ssh-keygen` attempt to fail with `ssh-sk-helper` shared library error).
- Generated resident, touch+PIN-required SSH credential:
  ```
  ssh-keygen -t ed25519-sk -O resident -O verify-required
  ```
- Set a FIDO2 PIN via `ykman fido access change-pin`.

### 2026-08-18  Arch PC login/sudo hardware auth
- Installed `pam-u2f`.
- Registered Key 1 (and later Key 3) via `pamu2fcfg`.
- Added `auth required pam_u2f.so` to `/etc/pam.d/sudo`.
- Confirmed `/etc/pam.d/sddm` already had the same line present  SDDM login now also requires touch on top of password.
- Validated the mechanism safely first via `hyprlock`'s own PAM file before trusting the full login screen (session stays alive on failure, unlike a full SDDM logout).

### 2026-08-18  Git commit signing (SSH-based, not GPG)
- Configured SSH-based git signing (Git 2.34+) rather than traditional GPG, reusing the existing FIDO2 SSH key:
  ```
  git config --global gpg.format ssh
  git config --global user.signingkey ~/.ssh/id_ed25519_sk.pub
  git config --global commit.gpgsign true
  ```
- Added an `allowed_signers` file for local signature verification.
- Registered the same public key on GitHub twice  once as an **Authentication Key**, once as a **Signing Key** (these are separate registrations under GitHub's SSH key settings, easy to miss).

### 2026-08-18  Proxmox host (Key 2 / `prower`) SSH access
- Generated a separate resident SSH credential dedicated to the Proxmox host:
  ```
  ssh-keygen -t ed25519-sk -O resident -O verify-required -f ~/.ssh/id_ed25519_sk_prower
  ```
- Pushed the public key to the Proxmox host's `authorized_keys` using existing (password) access.
- Hardened `sshd_config` on the Proxmox host:
  ```
  PasswordAuthentication no
  PubkeyAuthentication yes
  PermitRootLogin prohibit-password
  ```
- Verified in a **second, separate session** before closing the original  standard safe-change practice for anything touching the only way in.

### 2026-08-18  Backup credential (Key 3) rollout
- Set a PIN on the third, previously unregistered key.
- Registered Key 3 as an additional PAM credential (Arch PC login/sudo).
- Generated a dedicated backup SSH credential:
  ```
  ssh-keygen -t ed25519-sk -O resident -O verify-required -f ~/.ssh/id_ed25519_sk_backup
  ```
- Pushed the backup public key to: Proxmox host, all 4 active VMs/CTs, GitHub (Authentication + Signing key), Google (2-Step Verification security key).
- Key 3 is now in cold storage  not used day to day, exists solely as a recovery path.

### 2026-09-07  VM/CT hardening via Ansible
- Rolled Key 1's public key out to all active VMs/CTs via a loop over `ssh-copy-id` (different usernames per host required a `user@ip` list rather than a flat IP list).
- Wrote a small Ansible playbook (`site.yml`) to enforce the same `sshd_config` hardening applied to Proxmox, across all VMs in one run:
  - `PasswordAuthentication no`
  - `PubkeyAuthentication yes`
  - `PermitRootLogin prohibit-password` (where root SSH is used)
  - Handler to restart `sshd` only on actual change.
- Ran serially (`-f 1`)  deliberate, not a workaround: each host requires a physical touch, so parallel connections would produce overlapping, unmanageable touch prompts.

### 2026-08-18  Android (Z Fold 6) NFC usage
- Registered Key 1 as a WebAuthn security key on Google.
- Installed Yubico Authenticator (mobile) for NFC-based TOTP on services that don't yet support FIDO2/passkeys  seeds live on the physical key, not in an app database on the phone.

---

## Problems and Resolutions

| Problem | Cause | Resolution |
|---|---|---|
| `ssh-sk-helper: error while loading shared libraries: libfido2.so.1` | `libfido2` not installed | `sudo pacman -S libfido2` |
| `yubikey-manager` install 404s from every mirror | Stale local mirror database | `sudo pacman -Syyu` (force full refresh) before retrying |
| `sign_and_send_pubkey: signing failed ... invalid format` | Wrong physical key plugged in for the credential being used, or PIN typed into the wrong prompt | Confirm which key is physically connected with `ykman fido info` before authenticating; keep only one key plugged in at a time unless bootstrapping a new credential |
| Confused the FIDO2 PIN with the local SSH keyfile passphrase set during `ssh-keygen` | These are two separate secrets that both show up as "Enter passphrase" / PIN prompts | Cleared the local file passphrase with `ssh-keygen -p -f <key>` (set to blank); PIN lives on the hardware key itself via `ykman fido access change-pin` |
| `FIDO_ERR_PIN_NOT_SET` during SSH sign | PIN had actually been set on a *different*, still-boxed key  easy to mix up with 3 visually identical devices | Physically labeled all 3 keys after this; recommend doing this **before** starting registration, not after |
| Adding a new key (Key 3) to a host that already had password auth disabled | `ssh-copy-id` needs a valid existing credential to log in before it can add a new one  no bootstrap path once passwords are off | Plugged in both the already-authorized key and the new key simultaneously; authenticated with the old, installed the new |
| `no such identity: ...: No such file or directory` | Tilde (`~`) does not expand inside `-o IdentityFile=...` the way it does elsewhere in the shell | Used the full absolute path instead of `~` |
| Random garbled string typed into the terminal after a key touch (e.g. `cccccdftrhkv...`) | Cosmetic  some YubiKey configurations emit the touch event as literal keystrokes (legacy Yubico OTP slot behavior bleeding through) | Harmless; ignore, don't attempt to run it as a command |
| Ansible: `ssh_askpass: exec(/usr/lib/ssh/ssh-askpass): No such file or directory`, `incorrect passphrase supplied to decrypt private key` | Ansible's SSH subprocess has no attached interactive terminal, so PIN/touch prompts fall through to a GUI askpass helper that wasn't installed | Installed `x11-ssh-askpass`; set `SSH_ASKPASS` and `SSH_ASKPASS_REQUIRE=force`; loaded the key into a running `ssh-agent` beforehand (`ssh-add`) so PIN entry happens once, interactively, before Ansible ever runs |
| Ansible connections to multiple hosts in parallel each needing a physical touch | Default Ansible forking runs hosts concurrently, but only one key can be touched at a time | Forced serial execution with `-f 1` for any playbook run against FIDO2-authenticated hosts |

---

## Current Status

- [x] Key 1  Arch PC login/sudo, git signing, GitHub, Google, SSH to all 4 active VMs/CTs
- [x] Key 2 (`prower`)  Proxmox host root SSH, password auth fully disabled
- [x] Key 3  Backup credential registered everywhere Key 1/Key 2 are used; stored offline
- [x] All 4 active VMs/CTs  `PasswordAuthentication no`, key-only SSH, hardened via Ansible
- [ ]Physical `~/.ssh/config` aliases not yet set up (currently using explicit `-i` / `-o IdentityFile` per host)
- [ ]Vault / Keycloak not yet deployed  WebAuthn registration for those is a future phase once they exist

---

## Notes  Day-to-Day Usability, Read Before You Forget This

Since Prower isn't touched daily, future-me: **read this before assuming something is broken.**

- **There is no password fallback anywhere this was applied.** Proxmox root and all 4 VMs are key-only. If all 3 YubiKeys are ever lost simultaneously, there is no software recovery path  only re-provisioning access via the Proxmox console (noVNC) directly.
- **Every SSH connection requires a fresh physical touch**, even to the same host repeatedly. This is intentional (FIDO2 `verify-required`), not a bug, and not something a session cache can bypass.
- **Wrong key plugged in produces a confusing error, not a clear one**  typically `sign_and_send_pubkey: signing failed ... invalid format`. First troubleshooting step is always `ykman fido info` to confirm which physical key is actually connected.
- **Any unattended automation (cron, scripts, CI) against these hosts will hang forever** waiting for a touch that will never come. FIDO2-backed hosts are not compatible with unattended access by design  this includes future Ansible runs, which need the `ssh-agent` + `SSH_ASKPASS` setup documented above, run interactively.
- **Key 2 (`prower`) is intentionally not used for VM access, and Key 1 is intentionally not authorized on the Proxmox host.** This is the whole point of the tiering  don't "fix" this by cross-registering keys for convenience.
- If a key is ever lost: revoke its public key from `authorized_keys` on every host it had access to, remove it from GitHub/Google, and re-provision a replacement key into the same role using Key 3 (or whichever key remains) to bootstrap the new one in, following the same pattern documented above under "Adding a new key to a host that already had password auth disabled."

---
