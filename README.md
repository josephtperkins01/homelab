# Linux Home Lab Documentation

## Project Overview

A comprehensive Linux home lab environment built on VirtualBox for learning and experimenting with Linux system administration, networking, and infrastructure.

**Status:** 🟢 Complete - Phase 1, 2, and 3 Done

## Quick Start

### Environment
- **Host OS:** Windows 11
- **Virtualization:** VirtualBox
- **Guest OS:** Ubuntu Server 24.04.4 LTS
- **Server Name:** homelab-server
- **Username:** vboxuser

### Current Setup
```
Windows PC
   â”‚
   â””â”€ VirtualBox
      â””â”€ Ubuntu Server (10.0.2.15)
         â”œâ”€ SSH (port 2222 â†’ 22)
         â”œâ”€ Web Server (Nginx)
         â””â”€ Monitoring & Admin Tools
```

## Documentation Structure

- **[SETUP.md](./SETUP.md)** - Installation and initial configuration steps
- **[ARCHITECTURE.md](./ARCHITECTURE.md)** - System design and architecture diagram
- **[HARDWARE.md](./HARDWARE.md)** - Server specifications and resource allocation
- **[NETWORK.md](./NETWORK.md)** - Network configuration, SSH setup, port forwarding
- **[SSH_PORT_FORWARDING.md](./SSH_PORT_FORWARDING.md)** - VirtualBox port forwarding and SSH test instructions
- **[SSH_HARDENING.md](./SSH_HARDENING.md)** - SSH hardening: admin user, key auth, root disabled
- **[UFW.md](./UFW.md)** - Firewall configuration and rules
- **[NGINX.md](./NGINX.md)** - Nginx web server installation and configuration
- **[VIRTUAL_HOSTS.md](./VIRTUAL_HOSTS.md)** - Multi-site Nginx server blocks and routing
- **[SSL_TLS.md](./SSL_TLS.md)** - Self-signed SSL/TLS certificate setup for HTTPS
- **[MONITORING_ADMIN.md](./MONITORING_ADMIN.md)** - Monitoring, logging, backups, and admin health checks
- **[SERVICES.md](./SERVICES.md)** - Services running and how to manage them
- **[TROUBLESHOOTING.md](./TROUBLESHOOTING.md)** - Common issues and solutions

## Project Goals

### Phase 1: Foundation (Current)
- [x] Install Ubuntu Server 24.04 LTS
- [x] Basic system updates and configuration
- [x] Install admin tools (curl, wget, git, vim, htop, net-tools, tree, unzip)
- [x] SSH setup with port forwarding (see [SSH_PORT_FORWARDING.md](./SSH_PORT_FORWARDING.md))
- [x] Hostname configuration
- [x] SSH hardening: created `admin` user, enabled key-based auth, disabled root login, disabled password authentication (see [SSH_HARDENING.md](./SSH_HARDENING.md))
- [x] Firewall (ufw) enabled with default-deny incoming, SSH allowed (see [UFW.md](./UFW.md))

### Phase 2: Core Services (Complete)
- [x] Install and configure Nginx (see [NGINX.md](./NGINX.md))
- [x] Setup custom homepage
- [x] Configure firewall rules for HTTP
- [x] Port forwarding for HTTP (8080 â†’ 80)
- [x] Configure SSL/TLS with self-signed certificate (see [SSL_TLS.md](./SSL_TLS.md))
- [x] Port forwarding for HTTPS (8443 â†’ 443)
- [x] Configure virtual hosts for `site1.local` and `site2.local` (see [VIRTUAL_HOSTS.md](./VIRTUAL_HOSTS.md))
- [x] Troubleshoot Nginx default-server selection and verify Host-header routing

### Phase 3: Monitoring & Admin (Complete)
- [x] Install system monitoring (htop, glances) (see [MONITORING_ADMIN.md](./MONITORING_ADMIN.md))
- [x] Setup log rotation
- [x] Implement backup strategy
- [x] Create admin scripts

### Phase 4: Advanced
- [ ] Docker containerization
- [ ] Python automation scripts
- [ ] CI/CD pipeline basics
- [ ] Cloud deployment basics

## Key Learning Outcomes

By completing this home lab, you'll gain practical experience with:
- Linux system administration
- User and permission management
- Networking and SSH
- Web server configuration
- System monitoring and logging
- Shell scripting and automation
- Containerization and deployment

## Quick Commands Reference

### System Information
```bash
hostname              # Server name
ip addr              # Network configuration
df -h                # Disk usage
free -h              # Memory usage
uptime               # Uptime and load
htop                 # Interactive process monitor
```

### Server Management
```bash
sudo systemctl status nginx     # Check service status
sudo systemctl restart nginx    # Restart service
sudo journalctl -xe             # View system logs
```

## Getting Started

1. Start Ubuntu Server VM in VirtualBox
2. Login with `vboxuser` credentials
3. Follow steps in [SETUP.md](./SETUP.md)
4. Configure SSH following [NETWORK.md](./NETWORK.md)
5. Begin Phase 1 setup tasks

## Contributing to This Lab

As you build the home lab:
1. Update relevant documentation files
2. Add new configuration files to appropriate directories
3. Keep notes on issues encountered and resolutions
4. Commit changes with descriptive messages

```bash
git add .
git commit -m "Feature: Add description of changes"
git push origin main
```

## Resources & References

- [Ubuntu Server Documentation](https://ubuntu.com/server/docs)
- [VirtualBox Manual](https://www.virtualbox.org/manual/)
- [Linux man pages](https://man7.org/)
- [Nginx Documentation](https://nginx.org/en/docs/)

## Next Steps

1. **Complete Basic Setup** - Follow SETUP.md
2. **Configure SSH Access** - Follow NETWORK.md (see [SSH_PORT_FORWARDING.md](./SSH_PORT_FORWARDING.md))
3. **Install First Service** - Setup Nginx
4. **Document Progress** - Update this README

## Security & Maintenance (recommended next tasks)
- [ ] Install and configure fail2ban to protect SSH
- [ ] Implement an automated backup strategy and document scripts/locations
- [ ] Configure monitoring (Prometheus / Node Exporter / Glances) and logging/rotation
- [ ] Rotate SSH keys periodically and document key management
- [ ] Harden services (Nginx/TLS, Docker security) and add related docs

---

**Last Updated:** September 14, 2026
**Author:** josephtperkins01
**License:** MIT

