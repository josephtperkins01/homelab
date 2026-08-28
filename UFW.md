# UFW Firewall Configuration

This document records the basic firewall setup used in the homelab VM.

Goals
- Restrict incoming connections by default
- Allow necessary services explicitly (SSH initially)
- Provide simple commands for verification and future changes

Commands used

```bash
# Install ufw if not present
sudo apt update
sudo apt install -y ufw

# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (adjust if you change SSH port)
sudo ufw allow 22/tcp

# Enable the firewall (will prompt to confirm)
sudo ufw enable

# Check status
sudo ufw status verbose
```

Expected output

```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                    ALLOW       Anywhere
22/tcp (v6)               ALLOW       Anywhere (v6)
```

Adding rules for services

```bash
# Allow HTTP
sudo ufw allow 80/tcp
# Allow HTTPS
sudo ufw allow 443/tcp
# Remove a rule
sudo ufw delete allow 80/tcp

# Allow a specific IP
sudo ufw allow from 192.168.1.100 to any port 22 proto tcp
```

Notes
- When changing SSH-related rules, keep an active session open so you can roll back if needed.
- If you lock yourself out, use the VirtualBox GUI/console to access the VM and fix ufw rules.

Last updated: August 28, 2026
Author: josephtperkins01
