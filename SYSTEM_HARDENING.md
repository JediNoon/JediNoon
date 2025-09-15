
# 🖥️ SYSTEM HARDENING GUIDE - Ubuntu 24.04.2 on Proxmox

This guide is intended for **fresh Ubuntu 24.04.2 VMs** running in a **Proxmox lab environment**.  
It contains **two sections**: 

1. Step-by-step command runbook (copy-paste ready)  
2. Policy-style checklist (audit/documentation)

It covers updates, user auth, SSH, firewall, filesystem, logging, services, monitoring, and Proxmox lab notes.

---

## 1. Step-by-Step Runbook

### 🔄 Updates & Packages
```bash
sudo apt update && sudo apt upgrade -y
sudo apt autoremove --purge -y
sudo apt install unattended-upgrades apt-listchanges -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

### 👤 Users & Authentication
```bash
sudo adduser shaun
sudo usermod -aG sudo shaun
sudo passwd -l root
sudo apt install libpam-pwquality -y
sudo bash -c 'cat > /etc/security/pwquality.conf <<EOF
minlen = 12
ucredit = -1
lcredit = -1
dcredit = -1
ocredit = -1
EOF'
```

### 🔐 SSH Configuration (summary)
- Refer to [`HARDENING_GUIDE.md`](./HARDENING_GUIDE.md) for detailed SSH hardening.  
- Recommended: `PasswordAuthentication no`, `PubkeyAuthentication yes`, `AllowUsers <vmuser>`.

### 🌐 Firewall & Networking
```bash
sudo apt install ufw -y
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw enable
```

### 📂 Filesystem & Permissions
```bash
sudo bash -c 'echo "tmpfs /tmp tmpfs defaults,noexec,nosuid,nodev 0 0" >> /etc/fstab'
sudo mount -o remount /tmp
sudo chmod 700 /boot
```

### 📜 Logging & Auditing
```bash
sudo apt install auditd audispd-plugins fail2ban -y
sudo bash -c 'cat > /etc/audit/rules.d/hardening.rules <<EOF
-w /etc/passwd -p wa -k passwd_changes
-w /etc/sudoers -p wa -k sudoers_changes
EOF'
sudo systemctl restart auditd
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo apt install lynis -y
sudo lynis audit system
```

### ⚙️ Services & Daemons
```bash
systemctl list-unit-files --type=service --state=enabled
sudo systemctl disable --now avahi-daemon.service cups.service || true
```

### 🛡️ Proxmox Lab Notes
- Take snapshot before any hardening.  
- Route VM traffic via firewall VM (OPNsense/pfSense).  
- Restrict Proxmox WebGUI & SSH to Tailscale only.  
- Maintain offline backups.  

---

## 2. Policy-Style Checklist

### Updates & Packages
- [ ] Apply all security updates.  
- [ ] Enable unattended-upgrades.

### User & Authentication
- [ ] Admin accounts separate from default users.  
- [ ] Root account locked.  
- [ ] Strong password policies enforced.  

### SSH
- [ ] Root login disabled.  
- [ ] Key-based authentication only.  
- [ ] Allow only specific VM users (`AllowUsers`).  

### Firewall & Networking
- [ ] Default deny incoming, allow outgoing.  
- [ ] Only allow necessary services.  
- [ ] Apply sysctl kernel hardening.  

### Filesystem & Permissions
- [ ] `/tmp` mounted `noexec,nosuid,nodev`.  
- [ ] `/boot` restricted.  

### Logging & Auditing
- [ ] Auditd installed with baseline rules.  
- [ ] Fail2ban enabled for SSH.  
- [ ] Regular audits with Lynis.  

### Services & Daemons
- [ ] Audit enabled services.  
- [ ] Disable unnecessary services (Avahi, CUPS).  

### Monitoring & IDS
- [ ] Optional: Wazuh or local monitoring.  
- [ ] Periodically review logs and alerts.  

### Proxmox Lab Practices
- [ ] Snapshots before changes.  
- [ ] Lab subnet isolation via virtual bridges.  
- [ ] Proxmox host access restricted to Tailscale/VPN.  
- [ ] Offline backups maintained.  
- [ ] Document recovery & rollback procedures.  

---
