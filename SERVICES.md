# Services in the Homelab

I kept the service set intentionally small so I could understand each layer before adding another one.

## Running or configured services

| Service | Purpose | Access |
|---|---|---|
| OpenSSH | Remote administration | VirtualBox host port 2222 to guest port 22 |
| Nginx | Web serving and virtual hosts | HTTP 80, HTTPS 443 |
| UFW | Default-deny host firewall | Allows only explicitly opened ports |
| Cron | Nightly backup scheduling | `0 2 * * *` |
| Docker | Container build exercise | Local image build and CI validation |

I manage system services with `systemctl`, inspect logs with `journalctl`, and verify listening ports with `ss -tlnp`.

## Useful checks

```bash
sudo systemctl status ssh
sudo systemctl status nginx
sudo ufw status verbose
sudo ss -tlnp
sudo journalctl -u nginx --since today
```

The project is closed at Phase 4. Future services belong in a separate project rather than being added here without a new design and security review.
