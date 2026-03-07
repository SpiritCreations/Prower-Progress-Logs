# SSH Key-Only Authentication Setup

## Overview
This guide walks through configuring SSH key-only authentication on Ubuntu Server, disabling password-based login entirely. Written based on experience with 24.04.3 LTS, but applicable to recent Ubuntu Server versions.

> **Why it matters:** A common gotcha on Ubuntu Server is that `cloud-init` ships with its own SSH config file that can silently override your `sshd_config` settings. This guide accounts for that.

---

## Prerequisites
- Ubuntu Server VM installed and running
- SSH client installed on your local machine
- SSH access to the VM (password login still enabled at this point)

---

## Steps

### Step 1 - Generate an SSH key pair locally
On your local machine, generate a key pair. `ed25519` is recommended over `rsa` for modern setups:
```bash
ssh-keygen -t ed25519 -C "your-key-label"
```
You'll be prompted to choose a save location and optionally set a passphrase. A passphrase adds an extra layer of security and is **HIGHLY** recommended.

### Step 2 - Copy your public key to the VM
```bash
ssh-copy-id <username>@<VM-IP>
```
This appends your public key to `~/.ssh/authorized_keys` on the VM. You'll be prompted for your password one last time

### Step 3 - Verify SSH server is running
```bash
ps -ef | grep sshd
```
Confirm the binary shown is `/usr/sbin/sshd`. You can also verify the package:
```bash
dpkg -S /usr/sbin/sshd
```
Expected output: `openssh-server`

### Step 4 - Edit the main SSH config
```bash
sudo vim /etc/ssh/sshd_config
```
Check that the following lines exist. If they don't, add them:
```
PubkeyAuthentication yes
PasswordAuthentication no
UsePAM no
```

### Step 5 - Check the cloud-init override config
Ubuntu Server ships with a cloud-init SSH config that can override `sshd_config`. Always check this file:
```bash
sudo vim /etc/ssh/sshd_config.d/50-cloud-init.conf
```
Ensure it contains:
```
PasswordAuthentication no
```
If it says `yes` change it to `no`. If the line doesn't exist, add it.

**Note:** Skipping this step is the most common reason key only auth appears to be configured but still allows password login.

### Step 6 - Validate the config syntax
```bash
sudo sshd -t
```
No output means the syntax is valid. Any errors will be printed here - fix them before restarting.

### Step 7 - Restart the SSH service
> **Important:** Keep your current SSH session open until you've verified key login works. If something goes wrong you'll still have access.
```bash
 sudo systemctl restart ssh
```

### Step 8 - Verify the running config
```bash
sudo sshd -T | grep -i passwordauthentication
sudo sshd -T | grep -i pubkeyauthentication
```
Expected output:
```
passwordauthentication no
pubkeyauthentication yes
```

### Step 9 - Test from a clean client
From a machine without your SSH key loaded, attempt to connect:
```bash
ssh <username>@<VM-IP>
```
This should fail with `Permission denied (publickey)` - that's the expected result and confirms password login is disabled.

Then connect from your machine with the keys loaded:
```bash
ssh <username>@<VM-IP>
```
This should succeed without prompting for a password. If you set a passphrase during key generation, you will be prompted for that.

---

## Notes

**cloud-init is the most common culprit** if key-only auth isn't enforcing after following this guide. Always verify `/etc/ssh/sshd_config.d/50-cloud-init.conf` has `PasswordAuthentication no` set.

**The `50-cloud-init.conf` file may not exist on all versions** - if it's not there, `sshd_config` alone should be sufficient.

**`UsePAM no` may cause issues in some environments** - if you experience unexpected login failures after following this guide, try setting `UsePAM yes` and retesting. PAM handles some authentication flows that may be required depending on your setup.
