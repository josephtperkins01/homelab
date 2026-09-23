# SSH Hardening and Admin Access

I created a dedicated `admin` account so I could manage the server without using the default `vboxuser` account for every task. The account belongs to the `sudo` group, which lets me escalate only when an administrative command requires it.

## What I changed

- Created `admin` and verified its groups with `groups admin`.
- Added an Ed25519 public key to `~admin/.ssh/authorized_keys`.
- Set `PermitRootLogin no` because root should never be a named SSH login target; privileged work should be traceable to a real user and escalated with `sudo`.
- Set `PasswordAuthentication no` after confirming key-based login worked.

## Verification

```bash
groups admin
# Expected: admin : admin sudo users

sudo systemctl restart ssh
sudo sshd -T | grep passwordauthentication
# Expected: passwordauthentication no

sudo whoami
# Expected: root
```

From a new Windows PowerShell session, I verified the key-only path with:

```powershell
ssh admin@127.0.0.1 -p 2222
```

The connection opened without asking for a password.

## The edit that made this feel real

While I was editing `/etc/ssh/sshd_config`, my PC shut down mid-edit. I used the VirtualBox console to regain access, checked the file rather than assuming the change was lost, and confirmed that `PermitRootLogin no` was still present. I then searched for `PasswordAuthentication` directly in `nano`, changed it to `no`, restarted SSH, and tested from a completely fresh PowerShell window.

That sequence matters: I kept a console path available, verified the effective configuration with `sshd -T`, and tested the hardened login before closing the original access path.

## Recovery notes

If SSH access breaks during future changes, I can use the VirtualBox console to repair `/etc/ssh/sshd_config` or restore a snapshot. If Windows reports a stale host key after rebuilding the VM, I can remove it with:

```powershell
ssh-keygen -R [127.0.0.1]:2222
```

Last updated: September 23, 2026
