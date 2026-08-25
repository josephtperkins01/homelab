# Linux Home Lab Documentation

## Project Overview

A comprehensive Linux home lab environment built on VirtualBox for learning and experimenting with Linux system administration, networking, and infrastructure.

**Status:** 🟡 In Progress - Initial Setup Phase

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
   │
   └─ VirtualBox
      └─ Ubuntu Server (10.0.2.15)
         ├─ SSH (port 2222 → 22)
         ├─ Web Server (Nginx)
         └─ Monitoring & Admin Tools
```

## Documentation Structure

- **[SETUP.md](./SETUP.md)** - Installation and initial configuration steps
- **[ARCHITECTURE.md](./ARCHITECTURE.md)** - System design and architecture diagram
- **[HARDWARE.md](./HARDWARE.md)** - Server specifications and resource allocation
- **[NETWORK.md](./NETWORK.md)** - Network configuration, SSH setup, port forwarding
- **[SERVICES.md](./SERVICES.md)** - Services running and how to manage them
- **[TROUBLESHOOTING.md](./TROUBLESHOOTING.md)** - Common issues and solutions

## Project Goals

### Phase 1: Foundation (Current)
- [x] Install Ubuntu Server 24.04 LTS
- [x] Basic system updates and configuration
- [x] Install admin tools (curl, wget, git, vim, htop, net-tools, tree, unzip)
- [ ] SSH setup with port forwarding
- [ ] Hostname configuration

### Phase 2: Core Services
- [ ] Install and configure Nginx
- [ ] Setup basic web server
- [ ] Configure SSL/TLS
- [ ] Implement basic firewall rules

### Phase 3: Monitoring & Admin
- [ ] Install system monitoring (htop, glances)
- [ ] Setup log rotation
- [ ] Implement backup strategy
- [ ] Create admin scripts

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
2. **Configure SSH Access** - Follow NETWORK.md
3. **Install First Service** - Setup Nginx
4. **Document Progress** - Update this README

---

**Last Updated:** August 25, 2026
**Author:** josephtperkins01
**License:** MIT
