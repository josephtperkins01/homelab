# Architecture - Linux Home Lab

## System Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      Windows PC                             │
│                  (Windows 11 Pro)                           │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │         VirtualBox Hypervisor                        │  │
│  │                                                      │  │
│  │  ┌────────────────────────────────────────────────┐ │  │
│  │  │   Ubuntu Server 24.04.4 LTS VM                │ │  │
│  │  │   "homelab-server"                            │ │  │
│  │  │                                                │ │  │
│  │  │  Resources:                                   │ │  │
│  │  │  • vCPU: 2-4 cores                            │ │  │
│  │  │  • RAM: 4-8 GB                                │ │  │
│  │  │  • Storage: 40-60 GB                          │ │  │
│  │  │                                                │ │  │
│  │  │  Network: NAT 10.0.2.15                       │ │  │
│  │  │  SSH Port: 2222 (forwarded to 22)            │ │  │
│  │  │                                                │ │  │
│  │  │  ┌─────────────────────────────────────┐    │ │  │
│  │  │  │ Services Layer                      │    │ │  │
│  │  │  │ • Nginx Web Server                  │    │ │  │
│  │  │  │ • SSH Server                        │    │ │  │
│  │  │  │ • Cron Daemon                       │    │ │  │
│  │  │  │ • System Logging                    │    │ │  │
│  │  │  │ (Future: Docker, Python scripts)    │    │ │  │
│  │  │  └─────────────────────────────────────┘    │ │  │
│  │  │                                                │ │  │
│  │  │  ┌─────────────────────────────────────┐    │ │  │
│  │  │  │ Admin Tools                         │    │ │  │
│  │  │  │ • htop - Process monitoring         │    │ │  │
│  │  │  │ • git - Version control             │    │ │  │
│  │  │  │ • vim/nano - Text editors           │    │ │  │
│  │  │  │ • curl/wget - Download tools        │    │ │  │
│  │  │  │ • net-tools - Network diagnostics   │    │ │  │
│  │  │  └─────────────────────────────────────┘    │ │  │
│  │  │                                                │ │  │
│  │  │  ┌─────────────────────────────────────┐    │ │  │
│  │  │  │ Linux Kernel                        │    │ │  │
│  │  │  │ Ubuntu 24.04 LTS                    │    │ │  │
│  │  │  └─────────────────────────────────────┘    │ │  │
│  │  └────────────────────────────────────────────────┘ │  │
│  │                                                      │  │
│  │  Network: NAT Bridge                                │  │
│  │  Port Forwarding: 2222 → 22                         │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Host Network Access                                 │  │
│  │ • SSH: localhost:2222 → Server:22                   │  │
│  │ • HTTP (future): localhost:8080 → Server:80         │  │
│  │ • HTTPS (future): localhost:8443 → Server:443       │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Network Topology

### Current Phase (Phase 1)
```
Windows PC (Host)
    │
    ├─ SSH Client
    │   └─ Connects to: localhost:2222
    │
    └─ VirtualBox NAT
        └─ Port Forward: 2222 → 10.0.2.15:22
            │
            └─ Ubuntu Server
                └─ SSH Server (port 22)
                   └─ Login: vboxuser
```

### Phase 2+ (With Web Services)
```
Windows PC (Host)
    │
    ├─ SSH Client → localhost:2222 (Server port 22)
    ├─ Web Browser → localhost:8080 (Server port 80)
    └─ Web Browser → localhost:8443 (Server port 443)
        │
        └─ VirtualBox NAT
            └─ Ubuntu Server (10.0.2.15)
                ├─ SSH Server :22
                ├─ Nginx :80 → :8080 (HTTP)
                └─ Nginx :443 → :8443 (HTTPS)
```

## Component Breakdown

### 1. Virtualization Layer (VirtualBox)
- **Role:** Host the Ubuntu VM
- **Key Settings:**
  - Type: Linux
  - Version: Ubuntu (64-bit)
  - Memory: 4-8 GB
  - CPU: 2-4 cores
  - Storage: 40-60 GB
  - Network: NAT with port forwarding
  - Display: Headless (can be accessed via SSH)

### 2. Operating System (Ubuntu Server)
- **Distribution:** Ubuntu Server 24.04.4 LTS
- **Hostname:** homelab-server
- **User:** vboxuser (with sudo access)
- **IP:** 10.0.2.15 (VirtualBox NAT)
- **Update Cadence:** LTS (Long Term Support until 2029)

### 3. Service Layer
Currently installed/running:
- **SSH Server** - Remote access and automation
- **Cron Daemon** - Scheduled tasks
- **Systemd** - Service management
- **Journald** - Logging

Future services:
- **Nginx** - Web server
- **Docker** - Container runtime
- **Python** - Automation scripts

### 4. Administrative Tools

| Tool | Purpose | Phase |
|------|---------|-------|
| htop | Process monitoring | 1 |
| git | Version control | 1 |
| vim/nano | Text editing | 1 |
| curl/wget | HTTP/file download | 1 |
| net-tools | Network diagnostics | 1 |
| tree | Directory browsing | 1 |
| unzip | Archive extraction | 1 |
| nginx | Web server | 2 |
| docker | Containerization | 4 |
| python | Automation scripts | 3 |

## Communication Flow

### SSH Connection Flow
```
1. User initiates SSH on Windows
   ssh vboxuser@localhost -p 2222

2. SSH client connects to localhost:2222

3. VirtualBox NAT port forwarder intercepts
   Forwards to 10.0.2.15:22

4. Ubuntu SSH server responds
   Authenticates user

5. SSH session established
   User gets shell access to homelab-server
```

### File Structure

```
~/ (vboxuser home directory)
├── .ssh/
│   ├── authorized_keys      (Public key authentication)
│   └── config              (SSH config, optional)
├── .bashrc                  (Bash configuration)
├── .vimrc                   (Vim configuration, optional)
└── projects/                (User projects, optional)
    └── homelab-docs/        (This documentation repo)
```

## Security Architecture

### Current Security Posture

```
┌─────────────────────────────────┐
│     Security Layers             │
├─────────────────────────────────┤
│ 1. Host Machine Security        │
│    - Windows Firewall           │
│    - Windows Defender           │
├─────────────────────────────────┤
│ 2. VirtualBox Isolation         │
│    - VM sandboxed from host     │
│    - NAT prevents direct access │
├─────────────────────────────────┤
│ 3. SSH Encryption               │
│    - SSH encryption (TLS)       │
│    - Password or key auth       │
├─────────────────────────────────┤
│ 4. Linux User Permissions       │
│    - User: vboxuser             │
│    - Root operations via sudo   │
└─────────────────────────────────┘
```

### Security Enhancements (Future)

- [ ] SSH key-based authentication (no passwords)
- [ ] Disable SSH password authentication
- [ ] Configure firewall rules
- [ ] Setup fail2ban for brute-force protection
- [ ] Enable SELinux or AppArmor
- [ ] Regular security updates via unattended-upgrades
- [ ] Container security (Docker best practices)

## Resource Allocation

### Recommended VM Specifications

| Resource | Minimum | Recommended | Comfortable |
|----------|---------|-------------|------------|
| vCPU | 1 | 2 | 4 |
| RAM | 2 GB | 4 GB | 8 GB |
| Storage | 20 GB | 40 GB | 60+ GB |
| Network | 1 NIC | 1 NIC | 1 NIC |

### Performance Considerations

- Ubuntu Server is lightweight (vs desktop)
- LTS release = stable performance
- Nginx and Docker are efficient
- htop allows real-time monitoring

## Phase Progression

### Phase 1: Foundation ✓ In Progress
- System setup
- SSH access
- Admin tools

### Phase 2: Web Services
- Nginx installation
- SSL/TLS setup
- Virtual hosts

### Phase 3: Monitoring
- System monitoring
- Log aggregation
- Backup scripts

### Phase 4: Advanced
- Containerization
- Automation
- Cloud deployment

---

**Last Updated:** August 25, 2026
**Version:** 1.0
