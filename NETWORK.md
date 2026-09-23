# Network Configuration - Linux Home Lab

I kept the VM behind VirtualBox NAT and used explicit localhost forwards so the lab stayed isolated from the home network while I learned SSH and service routing.


## Network Overview

```
┌─────────────────────────────────────────────────────────┐
│                  Windows PC (192.168.x.x)               │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │  VirtualBox NAT Gateway                          │  │
│  │  • Host Router: 10.0.2.2                         │  │
│  │  • DNS: 10.0.2.3                                 │  │
│  │  • DHCP Server: 10.0.2.4                         │  │
│  │  • Network: 10.0.2.0/24                          │  │
│  │                                                  │  │
│  │  ┌────────────────────────────────────────────┐ │  │
│  │  │  Ubuntu Server (10.0.2.15)                │ │  │
│  │  │  Network Interface: enp0s3 (Intel PRO)    │ │  │
│  │  │  MAC: auto-generated                      │ │  │
│  │  │  IP: DHCP assigned from VirtualBox        │ │  │
│  │  └────────────────────────────────────────────┘ │  │
│  │                                                  │  │
│  │  Port Forwarding:                               │  │
│  │  • Host 2222 → Guest 10.0.2.15:22 (SSH)        │  │
│  │  • Host 8080 → Guest 10.0.2.15:80 (HTTP)       │  │
│  │  • Host 8443 → Guest 10.0.2.15:443 (HTTPS)     │  │
│  └──────────────────────────────────────────────────┘  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

## Network Details

### VirtualBox NAT Configuration

The VM uses VirtualBox's built-in NAT (Network Address Translation):

| Setting | Value | Purpose |
|---------|-------|---------|
| **Mode** | NAT | Isolated, port-forwardable |
| **Network** | 10.0.2.0/24 | Private VirtualBox network |
| **VM IP** | 10.0.2.15 | Assigned via DHCP |
| **Gateway** | 10.0.2.2 | Routes to host |
| **DNS** | 10.0.2.3 | Resolves domain names |
| **DHCP Server** | 10.0.2.4 | Assigns IPs to VMs |

### Network Interface in VM

#### Primary NIC (enp0s3)

```bash
# View detailed configuration
ip addr show enp0s3

# Expected output:
# 2: enp0s3: <BROADCAST,RUNNING,MULTICAST> mtu 1500
#     link/ether 08:00:27:xx:xx:xx brd ff:ff:ff:ff:ff:ff
#     inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
#     inet6 fe80::a00:27ff:fexx:xxxx/64 scope link
```

#### Netplan Configuration (Ubuntu)
Located at: `/etc/netplan/00-installer-config.yaml`

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
```

This uses DHCP to automatically obtain an IP from VirtualBox.

---

## SSH Access

### Quick Connect from Windows

```powershell
# Basic SSH connection
ssh vboxuser@localhost -p 2222

# First time will prompt:
# The authenticity of host '[localhost]:2222 ([127.0.0.1]:2222)' can't be established.
# Are you sure you want to continue connecting (yes/no/[fingerprint])? yes

# Enter password: [vboxuser password]
```

### SSH Connection Diagram

```
┌──────────────────┐
│  Windows PC      │
│  SSH Client      │
│                  │
│ ssh user@        │
│     localhost    │
│     -p 2222      │
└────────┬─────────┘
         │
         │ Port 2222 (localhost)
         │
    ┌────▼──────────────────┐
    │ VirtualBox NAT        │
    │ Port Forwarding Rule  │
    │                       │
    │ 2222 → 10.0.2.15:22  │
    └────┬──────────────────┘
         │
         │ Port 22 (SSH Server)
         │
    ┌────▼────────────────────┐
    │ Ubuntu Server (VM)       │
    │ SSH Server (OpenSSH)     │
    │ Listening on :22         │
    │                          │
    │ Accepts connection       │
    │ Authenticates user       │
    │ Provides shell           │
    └──────────────────────────┘
```

### Verify SSH Server Status

From within Ubuntu:

```bash
# Check SSH service status
sudo systemctl status ssh

# Output should show:
# ssh.service - OpenBSD Secure Shell server
#    Loaded: loaded (...; enabled; vendor preset: enabled)
#    Active: active (running) since ...

# Check SSH is listening on port 22
sudo ss -tlnp | grep ssh
# or
sudo netstat -tlnp | grep ssh

# Output should show:
# tcp   0  0 0.0.0.0:22   0.0.0.0:*   LISTEN   [PID]/sshd
```

### SSH Configuration File

Located at: `/etc/ssh/sshd_config`

Current settings (after installation):
```bash
Port 22
Protocol 2
PermitRootLogin prohibit-password
PubkeyAuthentication yes
PasswordAuthentication yes
X11Forwarding yes
Subsystem sftp  /usr/lib/openssh/sftp-server
```

---

## Port Forwarding Setup

### Configure Port Forwarding in VirtualBox

#### Step 1: Access Network Settings
1. Right-click VM → Settings
2. Network → Adapter 1
3. Click "Port Forwarding" button

#### Step 2: Add SSH Rule (2222 → 22)

| Field | Value |
|-------|-------|
| Name | SSH |
| Protocol | TCP |
| Host IP | (leave blank or 127.0.0.1) |
| Host Port | 2222 |
| Guest IP | 10.0.2.15 |
| Guest Port | 22 |

#### Step 3: Test from Windows

```powershell
# Test connection (will timeout if nothing is listening)
Test-NetConnection localhost -p 2222

# Try SSH (after VM is started)
ssh -v vboxuser@localhost -p 2222
```

### Future Port Forwarding Rules

For web services (Phase 2):

```
Name: HTTP
Protocol: TCP
Host Port: 8080
Guest Port: 80

Name: HTTPS
Protocol: TCP
Host Port: 8443
Guest Port: 443
```

---

## Network Troubleshooting

### Problem: "Connection refused" on SSH

**Diagnostic Steps:**
```powershell
# 1. Check if SSH port is open on host
Test-NetConnection localhost -p 2222

# 2. If fails, verify VirtualBox port forward is set
# (Go to VirtualBox Settings → Network → Port Forwarding)

# 3. Verify VM is running
# (Check VirtualBox Manager window)

# 4. From inside VM, verify SSH is running
ssh vboxuser@localhost  # This should work from VM
sudo systemctl status ssh
```

**Solutions:**
- Restart VirtualBox port forwarding (shutdown → start VM)
- Ensure SSH is enabled: `sudo systemctl enable ssh && sudo systemctl restart ssh`
- Check firewall isn't blocking (unlikely in VirtualBox NAT)

### Problem: "Network is unreachable"

From inside Ubuntu:
```bash
# Check interface is up
ip link show enp0s3

# If DOWN, bring it up
sudo ip link set enp0s3 up

# Restart networking
sudo systemctl restart networking

# Verify DHCP got an IP
ip addr show
```

### Problem: No internet access in VM

```bash
# Check gateway
ip route show
# Should show: default via 10.0.2.2 dev enp0s3

# Check DNS
cat /etc/resolv.conf
# Should have nameserver 10.0.2.3

# Test connectivity to gateway
ping 10.0.2.2

# Test DNS
ping 8.8.8.8

# Check interface has IP
ip addr show enp0s3
```

---

## DNS Resolution

### Current Configuration

DNS in VirtualBox NAT automatically uses host's DNS:

```bash
# View current DNS settings
cat /etc/resolv.conf

# Example output:
# nameserver 10.0.2.3   (VirtualBox DNS)
```

### Test DNS Resolution

```bash
# Test DNS lookups
nslookup google.com
dig ubuntu.com
host github.com

# These should all resolve to IPs
```

### Network Connectivity Test

```bash
# Test full internet connectivity
curl https://www.google.com
wget https://www.github.com

# Check network stats
netstat -i
ip -s link
```

---

## Static IP Configuration (Optional)

By default, VM uses DHCP (dynamic IP). To use static IP:

### Edit Netplan Configuration

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Change to:
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [10.0.2.15/24]
      routes:
        - to: 0.0.0.0/0
          via: 10.0.2.2
      nameservers:
        addresses: [10.0.2.3, 8.8.8.8]
```

Apply changes:
```bash
sudo netplan apply
ip addr show  # Verify IP is now static
```

---

## Advanced: Multiple Network Adapters (Future)

For multi-VM labs, add Host-Only Adapter:

### Adapter 2 (Host-Only Network)

| Setting | Value |
|---------|-------|
| Attached to | VirtualBox Host-Only Network |
| Name | vboxnet0 |
| Promiscuous Mode | Deny |
| MAC | Auto-generated |

### Benefits
- Direct VM-to-VM communication
- Isolated from host network
- Useful for multi-tier architecture testing

---

## Network Security

### Current Posture
- ✅ SSH only exposed via localhost:2222
- ✅ VirtualBox NAT provides isolation
- ✅ No direct internet-facing services
- ⚠️ Default SSH password authentication (upgradeable)

### Future Enhancements
- SSH key-based authentication
- Disable SSH password authentication
- Configure firewall rules (ufw)
- Setup fail2ban for brute-force protection

### Enable Ubuntu Firewall

```bash
# Install UFW (Uncomplicated Firewall)
sudo apt install ufw

# Allow SSH (port 22)
sudo ufw allow 22/tcp

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status
```

---

## Quick Reference Commands

```bash
# Network information
ip addr              # Show all network interfaces
ip route            # Show routing table
ip -s link          # Show link statistics
netstat -an         # Show all connections
ss -tlnp            # Show listening ports

# Connectivity tests
ping 10.0.2.2       # Test gateway
ping 8.8.8.8        # Test external IP
curl https://google.com  # Test DNS + HTTP

# SSH service
systemctl status ssh
systemctl start ssh
systemctl restart ssh
journalctl -u ssh   # View SSH logs

# View SSH config
cat /etc/ssh/sshd_config
```

---

**Last Updated:** August 25, 2026
**Network Mode:** NAT with Port Forwarding
**VM IP:** 10.0.2.15/24
