# SSH Hardening and Admin User

This document records SSH and admin user hardening steps performed in the homelab.

Summary of changes applied:

- Created a dedicated admin user with sudo privileges (group: sudo)
- Enabled SSH key-based authentication for the admin user
- Disabled root login over SSH (PermitRootLogin no)
- Continued to allow password authentication for other users (not disabled here)

Details

1. Admin account

- Username: admin
- Groups: admin, sudo, users

Verification:
```
# On the VM:
groups admin
# Expected: admin : admin sudo users

# Validate sudo works as admin:
ssh admin@127.0.0.1 -p 2222
sudo whoami
# Expected output: root
```

2. SSH key-based authentication

- Admin's public key stored in `~admin/.ssh/authorized_keys`
- Key was generated on Windows with `ssh-keygen -t ed25519 -C "your-email@example.com"` and copied to the VM

3. SSHD configuration changes

File: `/etc/ssh/sshd_config`
- `PermitRootLogin no` (root login disabled)
- `PubkeyAuthentication yes` (allowed)
- `PasswordAuthentication yes` (currently still allowed for accounts without keys)

After changes, reload SSH:
```
sudo systemctl restart ssh
```

4. Recommended next step (optional)
- Disable password authentication entirely:
  - Edit `/etc/ssh/sshd_config`
  - Set `PasswordAuthentication no`
  - Restart SSH: `sudo systemctl restart ssh`
- Ensure at least one account (e.g., admin) has keys installed before disabling passwords

5. Troubleshooting
- If SSH stops accepting connections after edits, use VirtualBox console to revert changes or restore from snapshot.
- To remove a problematic host key from Windows: `ssh-keygen -R [127.0.0.1]:2222`

Last updated: August 27, 2026
Author: josephtperkins01
