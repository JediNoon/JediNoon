# SSH Hardening Checklist
- [ ] Root login disabled
- [ ] Key-based authentication enabled
- [ ] Password authentication disabled after keys confirmed
- [ ] AllowUsers restricted to VM accounts
- [ ] X11Forwarding disabled
- [ ] TCP forwarding disabled
- [ ] ClientAliveInterval and ClientAliveCountMax set
- [ ] MaxAuthTries reduced to 3
- [ ] Backup original sshd_config before changes
