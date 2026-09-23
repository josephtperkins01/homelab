# Troubleshooting Notes

I kept this page focused on problems I actually encountered while building the lab, rather than listing generic Linux failures.

## SSH access after hardening

When my PC shut down during an SSH configuration edit, I used the VirtualBox console to recover access, verified `/etc/ssh/sshd_config`, checked the effective settings with `sudo sshd -T`, and tested key-based login from a fresh PowerShell window before closing the original session.

```bash
sudo sshd -T | grep -E 'permitrootlogin|passwordauthentication'
```

## Nginx served the wrong site

The first request for `site1.local` returned the old default page. Multiple port-80 server blocks existed, and Nginx selected the first loaded block when no explicit default server matched. I removed the old enabled default site, ran `nginx -t`, reloaded Nginx, and verified the result with a custom `Host` header.

```bash
curl -H "Host: site1.local" http://localhost/
```

## Docker workflow failed

The first GitHub Actions build failed because the root of the repository did not contain `Dockerfile` and `index.html`. I added both files to the build context and pushed again; the second run passed. This is documented in `DOCKER_CI.md`.

## Safe recovery habits

- Keep a working SSH or VirtualBox console session open while changing access controls.
- Run `nginx -t` before every reload.
- Use firewall status and listening-port checks before debugging a browser connection.
- Treat a failed CI run as evidence about the repository, not as something to bypass.
