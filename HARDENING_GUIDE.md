
# 🔐 Proxmox Lab Hardening Guide

This guide covers **system hardening for Ubuntu 24.04.2 VMs** and **Proxmox hypervisor-level security**.  
It is divided into three sections:

1. **Ubuntu VM Command Runbook** – copy/paste commands to harden a new Ubuntu VM.  
2. **Ubuntu VM Policy Checklist** – documentation-style checklist for auditing or GitHub publishing.  
3. **Proxmox VM Hardening Checklist** – security controls outside the guest OS (hypervisor, networking, snapshots).  

---

## 1. 🖥️ Ubuntu 24.04.2 Hardening – Command Runbook

Execute these commands in order on your VM (adjust usernames where needed).

```bash
# --- Update & Patch ---
sudo apt update && sudo apt upgrade -y
sudo apt autoremove --purge -y

# Enable unattended security updates
sudo apt install unattended-upgrades apt-listchanges -y
sudo dpkg-reconfigure --priority=low unattended-upgrades

# --- User & Auth Hardening ---
sudo adduser shaun
sudo usermod -aG sudo shaun
sudo passwd -l root

# Enforce strong password policies
sudo apt install libpam-pwquality -y
sudo bash -c 'cat > /etc/security/pwquality.conf <<EOF
minlen = 12
ucredit = -1
lcredit = -1
dcredit = -1
ocredit = -1
EOF'

# --- SSH Hardening ---
sudo apt install openssh-server -y
[ ! -s /etc/ssh/sshd_config ] && sudo cp /usr/share/openssh/sshd_config /etc/ssh/sshd_config

sudo bash -c 'cat > /etc/ssh/sshd_config <<EOF
Port 22
Protocol 2
PermitRootLogin no
PasswordAuthentication yes
PubkeyAuthentication yes
AllowUsers shaun
X11Forwarding no
AllowTcpForwarding no
ClientAliveInterval 300
ClientAliveCountMax 2
MaxAuthTries 3
EOF'

sudo sshd -t && sudo systemctl restart ssh

# --- Firewall ---
sudo apt install ufw -y
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw enable

# --- Service Hardening ---
systemctl list-unit-files --type=service --state=enabled
sudo systemctl disable --now avahi-daemon.service cups.service || true

# --- Filesystem Hardening ---
sudo bash -c 'echo "tmpfs /tmp tmpfs defaults,noexec,nosuid,nodev 0 0" >> /etc/fstab'
sudo mount -o remount /tmp
sudo chmod 700 /boot

# --- Kernel Hardening ---
sudo bash -c 'cat > /etc/sysctl.d/99-hardening.conf <<EOF
net.ipv4.ip_forward = 0
net.ipv6.conf.all.forwarding = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv4.icmp_ignore_bogus_error_responses = 1
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0
EOF'
sudo sysctl --system

# --- AppArmor ---
sudo aa-status || true
sudo aa-enforce /etc/apparmor.d/*

# --- Auditing & IDS ---
sudo apt install auditd audispd-plugins fail2ban -y
sudo bash -c 'cat > /etc/audit/rules.d/hardening.rules <<EOF
-w /etc/passwd -p wa -k passwd_changes
-w /etc/sudoers -p wa -k sudoers_changes
EOF'
sudo systemctl restart auditd
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# --- Security Auditing ---
sudo apt install lynis -y
sudo lynis audit system
```

---

## 2. ✅ Ubuntu VM Hardening – Policy Checklist

### 🔧 System Updates
- [ ] Apply security updates immediately after install.  
- [ ] Enable unattended-upgrades for automatic patches.  

### 👤 Users & Authentication
- [ ] Create separate admin account(s).  
- [ ] Lock the root account.  
- [ ] Enforce strong password policies via PAM.  

### 🔐 SSH Security
- [ ] Disable root SSH login.  
- [ ] Use key-based authentication (disable passwords once working).  
- [ ] Restrict login to specific user accounts (`AllowUsers`).  
- [ ] Disable X11 forwarding and TCP forwarding.  

### 🌐 Firewall / Networking
- [ ] Enable UFW, default deny incoming / allow outgoing.  
- [ ] Allow only required services.  
- [ ] Apply sysctl kernel hardening.  

### ⚙️ Services
- [ ] Audit running services (`systemctl`).  
- [ ] Disable or remove unnecessary services (Avahi, CUPS).  

### 📂 Filesystem
- [ ] Mount `/tmp` and `/var/tmp` with `noexec`, `nosuid`, `nodev`.  
- [ ] Restrict `/boot` permissions.  

### 🛡️ Mandatory Access Control
- [ ] Ensure AppArmor is enabled.  
- [ ] Set profiles to enforce mode.  

### 📜 Auditing & Monitoring
- [ ] Install and configure `auditd`.  
- [ ] Enable `fail2ban` for brute force protection.  
- [ ] Use `lynis` for periodic scans.  

### 🔄 Backup & Recovery
- [ ] Take Proxmox snapshots after hardening.  
- [ ] Document recovery steps for failed boot / lockouts.  

---

## 3. 🛡️ Proxmox VM Hardening Checklist

### 🔧 Proxmox Host Security
- [ ] Keep Proxmox updated.  
- [ ] Restrict WebGUI to LAN or Tailscale only.  
- [ ] Use HTTPS with valid TLS certs.  
- [ ] Disable root login; use admin group + 2FA.  

### 🔒 VM Networking & Isolation
- [ ] Use dedicated bridges for lab traffic.  
- [ ] Place VMs in isolated subnets.  
- [ ] Route via OPNsense/pfSense firewall VM.  
- [ ] Deny VM-to-host communication.  
- [ ] Apply VLAN tagging if available.  
- [ ] Use Proxmox firewall rules (default deny).  

### 🔑 Access Control
- [ ] Enable firewall at datacenter + VM level.  
- [ ] Restrict management access to trusted IPs/Tailscale.  
- [ ] Require SSH keys for host login.  
- [ ] No shared root accounts.  

### 🗄️ Storage & Snapshots
- [ ] Use LVM-thin or ZFS with snapshots.  
- [ ] Take baseline snapshots after hardening.  
- [ ] Automate backups with Proxmox Backup Server.  
- [ ] Verify restores.  

### 📜 Monitoring & Logging
- [ ] Forward logs to Wazuh/SIEM.  
- [ ] Enable Proxmox email alerts.  
- [ ] Monitor VM CPU/memory/disk usage.  
- [ ] Enable firewall log review.  

### 🛡️ VM Lifecycle & Security
- [ ] Apply OS hardening (Ubuntu/Windows).  
- [ ] Use hardened templates.  
- [ ] Remove unused VM hardware.  
- [ ] Use cloud-init templates.  

### 🔄 Tailscale / Remote Access
- [ ] Install Tailscale on Proxmox host.  
- [ ] Restrict WebGUI/SSH via Tailscale ACLs.  
- [ ] Install Tailscale inside VMs if needed.  
- [ ] No WAN exposure for VMs.  

### 🚨 Incident Readiness
- [ ] Maintain offline backups of configs.  
- [ ] Document recovery from host/VM failures.  
- [ ] Snapshot before major experiments.  

---

## 📌 Summary

With this setup:

- **Proxmox Host** → Hardened, WebGUI restricted, Tailscale-only access.  
- **VM Networking** → Isolated bridges, subnets, and firewall rules.  
- **VMs** → Individually hardened with Linux/Windows baselines.  
- **Storage** → Snapshots + backups, with tested restores.  
- **Monitoring** → Logs, alerts, and SIEM integration.  
- **Remote Access** → Tailscale only, zero public exposure.  

---

📂 Save this file as **`HARDENING_GUIDE.md`** in your Proxmox Lab GitHub repo.
