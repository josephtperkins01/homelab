# Linux Home Lab

A hands-on Linux home lab I built to get more experience with Linux administration, networking, web servers, security, troubleshooting, and basic automation.

The lab runs on an Ubuntu Server VM in VirtualBox on my Windows 11 PC. I built it from the ground up and used it to practice the same types of tasks I would expect to encounter in a Linux or technical support environment: setting up remote access, securing the server, deploying Nginx, configuring networking, monitoring system health, creating backups, and running a small Docker deployment with GitHub Actions.

I also documented the problems I ran into along the way and how I fixed them rather than only documenting the final working configuration.

## Lab Environment

- **Host:** Windows 11
- **Virtualization:** VirtualBox
- **Guest OS:** Ubuntu Server 24.04.4 LTS
- **Server:** `homelab-server`
- **Network:** VirtualBox NAT with port forwarding
- **SSH:** Windows `127.0.0.1:2222` -> Ubuntu `22`
- **HTTP:** Windows `127.0.0.1:8080` -> Ubuntu `80`
- **HTTPS:** Windows `127.0.0.1:8443` -> Ubuntu `443`

### Basic Layout

```text
Windows 11 PC
   |
   +-- VirtualBox NAT
       |
       +-- Ubuntu Server (homelab-server)
           |
           +-- SSH
           +-- Nginx
           +-- UFW firewall
           +-- Monitoring
           +-- Automated backups
           +-- Docker
```

## What I Built

### Linux Administration & Security

- Installed and configured an Ubuntu Server VM.
- Created a dedicated administrative user with sudo access.
- Configured SSH key-based authentication.
- Disabled root SSH login and password authentication after verifying key-based access.
- Configured UFW with a default-deny inbound policy and only allowed the ports required by the lab.
- Used systemd and standard Linux tools to inspect and manage services.

### Networking & Web Server

- Configured VirtualBox NAT port forwarding for SSH, HTTP, and HTTPS.
- Installed and configured Nginx.
- Created a custom web page hosted by the server.
- Configured HTTPS using a local self-signed certificate.
- Set up multiple Nginx virtual hosts.
- Troubleshot an issue where Nginx was serving the default site instead of the intended virtual host.
- Used custom `Host` headers and Nginx configuration testing to confirm the correct site was being served.

### Monitoring, Logging & Backups

- Used `htop` and `glances` to monitor system resources.
- Reviewed Nginx and system logs with `journalctl`.
- Configured and tested Nginx log rotation.
- Created automated nightly backups of `/etc/nginx` and `/var/www`.
- Wrote a basic health-check script to report on:
  - System uptime
  - Resource usage
  - Running services
  - Firewall status
  - Backup status

### Docker & GitHub Actions

- Containerized an Nginx web site with Docker.
- Created a GitHub Actions workflow to validate Docker builds on pushes and pull requests.
- Troubleshot an initial failed workflow caused by the repository structure and missing build files.
- Corrected the repository layout and verified successful CI runs.

## What I Learned

This project gave me practical experience with Linux beyond simply running commands. A large part of the project was troubleshooting problems when something did not work as expected.

Some examples included:

- Fixing SSH configuration without locking myself out of the server.
- Tracking down why the wrong Nginx site was being served.
- Verifying network connectivity through VirtualBox port forwarding.
- Testing firewall rules and confirming which ports were actually listening.
- Diagnosing a failed GitHub Actions build and fixing the repository structure.
- Validating backups and log rotation instead of assuming they were working.

## Project Documentation

The repository contains notes and configuration details for the major parts of the lab:

- [**ARCHITECTURE.md**](./ARCHITECTURE.md) - Overall lab design
- [**HARDWARE.md**](./HARDWARE.md) - VM resources and assumptions
- [**NETWORK.md**](./NETWORK.md) - Networking and port forwarding
- [**SSH_PORT_FORWARDING.md**](./SSH_PORT_FORWARDING.md) - SSH connectivity
- [**SSH_HARDENING.md**](./SSH_HARDENING.md) - SSH security configuration and troubleshooting
- [**UFW.md**](./UFW.md) - Firewall configuration
- [**NGINX.md**](./NGINX.md) - Nginx installation and configuration
- [**VIRTUAL_HOSTS.md**](./VIRTUAL_HOSTS.md) - Multiple sites and virtual host troubleshooting
- [**SSL_TLS.md**](./SSL_TLS.md) - Local HTTPS setup
- [**MONITORING_ADMIN.md**](./MONITORING_ADMIN.md) - Monitoring, logging, backups, and health checks
- [**DOCKER_CI.md**](./DOCKER_CI.md) - Docker and GitHub Actions
- [**SERVICES.md**](./SERVICES.md) - Services running in the lab
- [**TROUBLESHOOTING.md**](./TROUBLESHOOTING.md) - Problems encountered and how I resolved them
- [**screenshots/**](./screenshots/) - Screenshots of the working environment

## Useful Commands

A few of the commands I used regularly while building and troubleshooting the lab:

```bash
hostname
ip -br addr
sudo systemctl status nginx
sudo nginx -t
sudo ufw status verbose
sudo ss -tlnp
sudo journalctl -u nginx --since today
```

## References

- [Ubuntu Server Documentation](https://ubuntu.com/server/docs)
- [VirtualBox Manual](https://www.virtualbox.org/manual/)
- [Linux man pages](https://man7.org/)
- [Nginx Documentation](https://nginx.org/en/docs/)

---

**Last updated:** September 23, 2026  
**Author:** josephtperkins01  
**License:** MIT
