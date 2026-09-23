# Setup Guide - Linux Home Lab

I used this guide while bringing the Ubuntu VM from a fresh install to a reachable, managed server. The commands below reflect the order I followed and the checks I used to avoid losing access.


## Table of Contents
1. [Initial System Setup](#initial-system-setup)
2. [Network Configuration](#network-configuration)
3. [SSH Setup](#ssh-setup)
4. [Admin Tools Installation](#admin-tools-installation)
5. [Verification Steps](#verification-steps)

## Initial System Setup

### Step 1: System Update and Upgrade
After initial Ubuntu installation, update all packages:

```bash
sudo apt update
sudo apt upgrade -y
sudo reboot
```

This ensures all system packages are current and secure.

### Step 2: Change Hostname
Instead of generic server names, give your server a meaningful name:

```bash
sudo hostnamectl set-hostname homelab-server
hostname  # Verify the change
```

### Step 3: Check System Health
After reboot, verify everything is working:

```bash
hostname          # Should show: homelab-server
ip addr           # Check network configuration
df -h             # Verify disk space
free -h           # Verify RAM
uptime            # Check uptime and load
```

## Network Configuration

### Current Setup
- **VM Network Mode:** NAT (Network Address Translation)
- **VM IP Address:** 10.0.2.15 (assigned by VirtualBox)
- **Access Method:** SSH with port forwarding

### VirtualBox Network Configuration

#### Setting Up Port Forwarding (NAT)

1. **Shutdown the VM:**
   ```bash
   sudo shutdown now
   ```

2. **Configure Port Forward in VirtualBox:**
   - Right-click VM → Settings
   - Network → Adapter 1
   - Attached to: NAT
   - Click "Port Forwarding" button
   - Add new rule:
     - Name: SSH
     - Protocol: TCP
     - Host Port: 2222
     - Guest Port: 22
     - Guest IP: 10.0.2.15

3. **Start VM and verify:**
   ```bash
   ip -br addr
   ```
   Should show: `enp0s3 UP 10.0.2.15/24`

## SSH Setup

### Enable SSH Server (usually enabled by default)

```bash
sudo systemctl status ssh
sudo systemctl start ssh
sudo systemctl enable ssh
```

### Connect from Windows

Once port forwarding is configured:

```powershell
ssh vboxuser@localhost -p 2222
```

**First time connection:**
- Type "yes" when asked about fingerprint
- Enter password when prompted

### SSH Key-Based Authentication (Optional)

For more secure access without passwords:

1. **Generate SSH key on Windows:**
   ```powershell
   ssh-keygen -t ed25519 -C "your-email@example.com"
   ```

2. **Copy public key to server:**
   ```powershell
   cat $env:USERPROFILE\.ssh\id_ed25519.pub | ssh -p 2222 vboxuser@localhost "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
   ```

3. **Test key-based login:**
   ```powershell
   ssh -i $env:USERPROFILE\.ssh\id_ed25519 -p 2222 vboxuser@localhost
   ```

4. **(Optional) Disable password authentication:**
   ```bash
   sudo nano /etc/ssh/sshd_config
   # Find and change:
   # PasswordAuthentication yes → PasswordAuthentication no
   sudo systemctl restart ssh
   ```

## Admin Tools Installation

### Install Essential Tools

```bash
sudo apt install -y \
  curl \
  wget \
  git \
  vim \
  htop \
  net-tools \
  tree \
  unzip \
  build-essential \
  software-properties-common
```

### What Each Tool Does

| Tool | Purpose |
|------|---------|
| curl | Download files and test APIs |
| wget | Download files (alternative to curl) |
| git | Version control |
| vim | Text editor |
| htop | Interactive process monitor |
| net-tools | Network diagnostics (ifconfig, netstat, etc.) |
| tree | Display directory structure |
| unzip | Extract zip files |
| build-essential | Compilers and dev tools |

### Test Installation

```bash
htop          # Press 'q' to exit
git --version
curl --version
vim --version
tree --version
```

## Sudo Access

Your user already has sudo access. To understand permissions:

```bash
sudo -v            # Check sudo access
sudo -l             # List what you can run
id                  # Show user/group info
```

## Verification Steps

### Complete System Check

Run these commands to verify everything is working:

```bash
# System identity
echo "=== System Identity ==="
hostname
whoami

# Network
echo "=== Network Status ==="
ip -br addr
ip route

# System resources
echo "=== System Resources ==="
df -h
free -h
lscpu

# Services
echo "=== Key Services ==="
systemctl status ssh
systemctl status cron

# Tools verification
echo "=== Tools Available ==="
git --version
curl --version
htop --version
```

### Checklist

- System updated and rebooted
- Hostname changed to homelab-server
- SSH is running and enabled
- Port forwarding configured in VirtualBox (2222 → 22)
- Can SSH from Windows: `ssh vboxuser@localhost -p 2222`
- Admin tools installed
- All verification commands run successfully

## Troubleshooting

### Can't connect via SSH

1. Verify SSH is running:
   ```bash
   sudo systemctl status ssh
   ```

2. Verify port forwarding in VirtualBox Settings

3. Test connectivity:
   ```powershell
   Test-NetConnection localhost -p 2222
   ```

### Network not working in VM

1. Check interface status:
   ```bash
   ip -br addr
   ```

2. Restart networking:
   ```bash
   sudo systemctl restart networking
   ```

3. Check VirtualBox Settings → Network → Adapter 1 is enabled

### Forgot password

- Restart VM and use GRUB recovery mode to reset password

## Next Steps

Once setup is complete:
1. Setup your preferred text editor (.vimrc or .nanorc)
2. Configure git on the server
3. Create SSH key pair for GitHub access
4. Begin Phase 2: Install and configure Nginx

## SSH Hardening (applied)

The following hardening steps were applied and verified on the VM:

- Created dedicated `admin` user and added to `sudo` group
- Disabled root login over SSH (`PermitRootLogin no`)
- Disabled password authentication for SSH (`PasswordAuthentication no`) — key-only access enforced

Verification commands (run on the VM):

```bash
# Restart SSH to apply changes
sudo systemctl restart ssh

# Verify sshd effective configuration
sudo sshd -T | grep passwordauthentication
# Expected: passwordauthentication no
```

From a fresh Windows PowerShell session, verify key-based login still works:

```powershell
ssh admin@127.0.0.1 -p 2222
# Should log in without prompting for password (keys must be installed)
```

**Important:** Keep at least one account with a valid key before disabling password auth.

## Firewall (ufw)

A basic firewall policy was enabled using `ufw` with default-deny incoming and SSH allowed:

```bash
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw enable
sudo ufw status verbose
# Expected: "Status: active" and "22/tcp ALLOW IN"
```

This provides a minimal secure baseline: no incoming connections except explicitly allowed services.

---

**Setup Date:** August 25, 2026
**Completed By:** josephtperkins01
