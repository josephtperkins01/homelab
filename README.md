# Linux Home Lab

## Portfolio Overview

I built and documented a four-phase Linux home lab to demonstrate practical systems administration, infrastructure, security, networking, monitoring, containerization, and CI/CD skills.

**Status:** Complete ? Phases 1?4 implemented and documented

## Environment

- **Host:** Windows 11
- **Virtualization:** VirtualBox
- **Guest:** Ubuntu Server 24.04.4 LTS
- **Server:** `homelab-server`
- **Network model:** VirtualBox NAT with localhost port forwarding
- **Access path:** Windows `127.0.0.1:2222` to Ubuntu SSH port 22

```text
Windows PC
   |
   +-- VirtualBox NAT
       +-- Ubuntu Server (homelab-server)
           +-- SSH: host 2222 -> guest 22
           +-- HTTP: host 8080 -> guest 80
           +-- HTTPS: host 8443 -> guest 443
           +-- Nginx, UFW, monitoring, backups, Docker
```

## Skills Demonstrated

### Linux administration and security

- Built and maintained an Ubuntu Server VM.
- Created a dedicated sudo administrator and used key-based SSH access.
- Disabled root SSH login and password-based SSH authentication after validating key access.
- Applied a default-deny inbound UFW policy and opened only required service ports.

### Networking and web infrastructure

- Configured VirtualBox NAT port forwarding for SSH, HTTP, and HTTPS.
- Deployed Nginx with a custom homepage and self-signed TLS.
- Hosted two independent virtual sites with Nginx server blocks.
- Diagnosed a default-server selection issue and verified host-based routing with custom `Host` headers.

### Operations and reliability

- Used `htop` and `glances` for live resource inspection.
- Customized Nginx log rotation and validated it with a dry run and a real rotation.
- Automated nightly backups of `/etc/nginx` and `/var/www` with cron.
- Created a health-check script covering uptime, resources, services, firewall status, and backup state.

### Containers and CI/CD

- Built a containerized Nginx site with Docker.
- Added GitHub Actions to validate Docker builds on pushes and pull requests.
- Used the initial failed workflow to identify missing root-level build files, corrected the repository structure, and verified a passing run.

## Evidence and Technical Records

These files document the implementation and the troubleshooting behind it. They are project evidence, not a step-by-step tutorial.

- **[ARCHITECTURE.md](./ARCHITECTURE.md)** ? Built system design and component relationships
- **[HARDWARE.md](./HARDWARE.md)** ? Virtual machine resource assumptions
- **[NETWORK.md](./NETWORK.md)** ? Network model and connectivity decisions
- **[SSH_PORT_FORWARDING.md](./SSH_PORT_FORWARDING.md)** ? Tested SSH access path
- **[SSH_HARDENING.md](./SSH_HARDENING.md)** ? Admin access and SSH hardening, including recovery from an interrupted edit
- **[UFW.md](./UFW.md)** ? Firewall policy and service rules
- **[NGINX.md](./NGINX.md)** ? Nginx deployment and custom homepage
- **[VIRTUAL_HOSTS.md](./VIRTUAL_HOSTS.md)** ? Multi-site hosting and default-server troubleshooting
- **[SSL_TLS.md](./SSL_TLS.md)** ? Local self-signed HTTPS implementation
- **[MONITORING_ADMIN.md](./MONITORING_ADMIN.md)** ? Monitoring, log rotation, backups, and health checks
- **[DOCKER_CI.md](./DOCKER_CI.md)** ? Docker image and GitHub Actions build validation
- **[SERVICES.md](./SERVICES.md)** ? Services represented in the completed lab
- **[TROUBLESHOOTING.md](./TROUBLESHOOTING.md)** ? Problems encountered and how I resolved them
- **[screenshots/](./screenshots/)** ? Evidence of the Nginx site running

## Project Outcome

This repository captures a completed, working lab rather than a list of commands. It shows how I designed the environment, made security tradeoffs, validated changes, investigated failures, and documented the final state.

The project is closed at Phase 4. Any future cloud migration or additional automation will be treated as a separate project.

## References

- [Ubuntu Server Documentation](https://ubuntu.com/server/docs)
- [VirtualBox Manual](https://www.virtualbox.org/manual/)
- [Linux man pages](https://man7.org/)
- [Nginx Documentation](https://nginx.org/en/docs/)

---

**Last updated:** September 23, 2026
**Author:** josephtperkins01
**License:** MIT
