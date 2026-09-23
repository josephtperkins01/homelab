# Monitoring and Admin Operations

I finished the third phase by adding the operational pieces I would want on a small server: live inspection, bounded logs, recoverable backups, and one health-check command.


This document records the Phase 3 work completed for the homelab project.

## System Monitoring

Installed monitoring tools to observe live server resource usage.

- Installed `htop` and `glances` via apt
- Verified both tools display accurate, real-time CPU, memory, network, and disk I/O statistics
- Confirmed no warnings or critical alerts were present under normal load

## Log Rotation

Reviewed and customized the default nginx log rotation policy rather than leaving it at default settings.

- Reviewed `/etc/logrotate.d/nginx`, understanding each directive (daily rotation, compression, delayed compression, log recreation with correct ownership/permissions, and postrotate hook to make nginx reopen its log handles)
- Customized retention from the default 14 days down to 7 days to conserve disk space on the lab VM
- Validated the updated rule using `logrotate -d` (dry-run/debug mode) to confirm correct behavior without waiting for a scheduled rotation
- Confirmed real rotation occurred correctly: old logs aged and renamed, oldest logs beyond the retention count were handled correctly, and a fresh log file was created with proper permissions

## Backup Strategy

Implemented an automated backup system for server configuration and site content.

- Wrote a backup script (`/usr/local/bin/backup-nginx.sh`) that archives `/etc/nginx` and `/var/www` into a timestamped, compressed `.tar.gz` file stored in `/var/backups/homelab`
- Made the script executable and tested it manually, confirming a valid backup archive was created
- Automated the script with a cron job (`0 2 * * * /usr/local/bin/backup-nginx.sh`) to run nightly at 2:00 AM
- Verified the cron job was correctly registered using `sudo crontab -l`

## Admin Scripts

Wrote a custom system health-check script to consolidate key server status checks into a single command.

- Created `/usr/local/bin/health-check.sh`, reporting: system uptime, disk usage, memory usage, nginx service status, SSH service status, firewall (`ufw`) status, and the filename of the most recent backup
- Made the script executable and tested it, confirming accurate output across all sections

This completes Phase 3: Monitoring & Admin of the project.
