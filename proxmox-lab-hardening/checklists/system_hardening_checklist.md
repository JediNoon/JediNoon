# System Hardening Checklist
## Updates & Packages
- [ ] Apply security updates
- [ ] Enable unattended upgrades

## User & Authentication
- [ ] Separate admin accounts
- [ ] Lock root
- [ ] Strong password policies

## SSH
- [ ] Root login disabled
- [ ] Key-based only
- [ ] Restrict allowed users

## Firewall & Networking
- [ ] Default deny incoming
- [ ] Allow only required services
- [ ] Kernel hardening applied

## Filesystem
- [ ] /tmp mounted noexec,nosuid,nodev
- [ ] /boot permissions restricted

## Logging & Auditing
- [ ] auditd installed and configured
- [ ] fail2ban enabled for SSH
- [ ] Regular Lynis audits

## Services
- [ ] Audit enabled services
- [ ] Disable unnecessary services

## Proxmox Lab Practices
- [ ] Snapshots before changes
- [ ] Subnet isolation via bridges
- [ ] Tailscale/VPN access only
- [ ] Offline backups maintained
- [ ] Document recovery procedures
