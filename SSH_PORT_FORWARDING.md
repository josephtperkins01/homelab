# VirtualBox Port Forwarding & SSH Test

I chose a localhost-only forward so Windows could reach SSH without exposing the VM directly to the LAN. This is the connection path I tested from PowerShell.


This document shows the recommended VirtualBox NAT port-forwarding rule and how to test SSH access from Windows.

Important: Use 127.0.0.1 (localhost) for the Host IP — do not use your Windows LAN IP.

Recommended Rule (VirtualBox → VM using NAT):

| Setting   | What to enter                                    |
|-----------|--------------------------------------------------|
| Name      | SSH                                              |
| Protocol  | TCP                                              |
| Host IP   | 127.0.0.1                                        |
| Host Port | 2222                                             |
| Guest IP  | (leave blank)                                    |
| Guest Port| 22                                               |

Why this works
- Host = your Windows PC (127.0.0.1 is the loopback for the host).
- Guest = the Ubuntu VM (VirtualBox NAT routes to the VM; leaving Guest IP blank is robust across reboots).
- Windows connects to localhost:2222 which VirtualBox forwards to the VM's port 22.

Topology
```
Windows Host (127.0.0.1:2222)
   ↓ (VirtualBox NAT port forward)
Ubuntu VM (10.0.2.x:22)
```

Step-by-step: Add the rule
1. Shutdown VM: In Ubuntu run: `sudo shutdown now` or use VirtualBox manager to power off.
2. In VirtualBox Manager: Right-click the VM → Settings → Network → Adapter 1 → Advanced → Port Forwarding
3. Click + to add a rule and enter the values from the table above. Click OK/save.
4. Start the VM and log in locally or proceed to test from the host.

Test SSH from Windows PowerShell
1. Open PowerShell on Windows
2. Run:

```powershell
ssh vboxuser@127.0.0.1 -p 2222
```

3. On first connect, type `yes` to accept the host key fingerprint.
4. Enter the `vboxuser` password (nothing will echo as you type).
5. You should get a shell prompt: `vboxuser@homelab-server:~$`

Troubleshooting checklist
- Connection refused:
  - Ensure VM is running.
  - Verify port-forward rule is present and correct.
  - From Windows run: `Test-NetConnection -ComputerName 127.0.0.1 -Port 2222`
- Authentication failure:
  - Verify the username/password are correct.
  - Consider setting up SSH key auth (see SETUP.md).
- Host key warning:
  - If you reinstalled the VM or changed keys, remove the old host key entry in `~/.ssh/known_hosts` on Windows.
- SSH not running on VM:
  - From VM: `sudo systemctl status ssh` and `sudo systemctl start ssh`.
- Firewall issues (rare with NAT):
  - If using a host firewall, ensure it allows localhost:2222 (usually allowed by default).

Notes
- Leaving Guest IP blank avoids issues if the VM's NAT-assigned IP changes.
- For additional services (HTTP/HTTPS), add similar port-forwarding rules (e.g., Host 8080→Guest 80, Host 8443→Guest 443).

Last updated: August 27, 2026
Author: josephtperkins01
