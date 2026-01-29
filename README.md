_This project has been created as part of the 42 curriculum by gumagni_

# Born2beRoot

## Description

Born2beRoot is a system administration project that teaches the fundamentals of Linux server setup and security hardening. The goal is to create a properly secured Debian virtual machine with correct partitioning, firewall rules, SSH configuration, password policies, and a monitoring script.

## Instructions

### Installation

1. Create a VirtualBox VM with 2GB RAM and 10GB disk
2. Install Debian with LVM partitioning:
   - `/boot`: 512MB
   - `/`: 8GB
   - `/home`: 6GB
   - `/var`: 4GB
   - `/var/log`: 4GB
   - `swap`: 2GB
3. During installation, create a non-root user and select SSH server

### Configuration

**SSH (Port 4242):**
```bash
sudo nano /etc/ssh/sshd_config
# Port 4242
# PermitRootLogin no
sudo systemctl restart ssh
```

**Firewall:**
```bash
sudo apt install ufw
sudo ufw enable
sudo ufw allow 4242
```

**Password Policy:**
```bash
sudo apt install libpam-pwquality
sudo nano /etc/security/pwquality.conf
# minlen=10, dcredit=-1, ucredit=-1, lcredit=-1, maxrepeat=3, reject_username, difok=7

sudo nano /etc/login.defs
# PASS_MAX_DAYS 30
# PASS_MIN_DAYS 2
# PASS_WARN_AGE 7
```

**Account Lockout:**
```bash
sudo nano /etc/pam.d/common-auth
# auth required pam_tally2.so onerr=fail audit silent maxfail=3 unlock_time=600
```

**Sudo Logging:**
```bash
sudo visudo
# Defaults logfile="/var/log/sudo/sudo.log"
# Defaults log_input, log_output
sudo mkdir -p /var/log/sudo && sudo touch /var/log/sudo/sudo.log
```

**AppArmor:**
```bash
sudo systemctl enable apparmor
sudo systemctl start apparmor
sudo aa-status
```

**Monitoring Script (Bonus):**
```bash
sudo nano /usr/local/bin/monitoring.sh
```
Add content and make executable:
```bash
#!/bin/bash
ARCH=$(uname -a)
CPUS=$(grep -c "^processor" /proc/cpuinfo)
VRAM=$(free --mega | awk '$1 == "Mem:" {print $2}')
VRAM_USE=$(free --mega | awk '$1 == "Mem:" {print $3}')
DISK=$(df -Bg | grep "^/dev" | grep -v boot | awk '{fd += $2} END {print fd}')
DISK_USE=$(df -Bg | grep "^/dev" | grep -v boot | awk '{ud += $3} END {print ud}')
CPU_LOAD=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{printf("%.1f", 100 - $1)}')
LAST_BOOT=$(who -b | awk '$1 == "system" {print $3 " " $4}')
LVM=$(if [ $(lsblk | grep lvm | wc -l) -gt 0 ]; then echo "yes"; else echo "no"; fi)
TCP=$(ss -t | grep ESTAB | wc -l)
USERS=$(users | wc -w)
IP=$(hostname -I)
MAC=$(ip link | grep "link/ether" | awk '{print $2}')

echo "#Architecture: $ARCH"
echo "#CPU physical: $CPUS"
echo "#vCPU: $CPUS"
echo "#Memory Usage: $VRAM_USE/${VRAM}MB"
echo "#Disk Usage: $DISK_USE/${DISK}Gb"
echo "#CPU load: ${CPU_LOAD}%"
echo "#Last boot: $LAST_BOOT"
echo "#LVM use: $LVM"
echo "#Connections TCP: $TCP ESTABLISHED"
echo "#User log: $USERS"
echo "#Network: IP $IP ($MAC)"
```

Make executable and add to login:
```bash
sudo chmod +x /usr/local/bin/monitoring.sh
echo "/usr/local/bin/monitoring.sh" >> ~/.bashrc
```

## Operating System & Design Choices

### Debian vs Rocky Linux

**Why Debian:**
- Large package repository (60,000+ packages)
- Intuitive APT package manager
- Extensive community documentation
- Simpler default security (AppArmor)
- Ideal for learning Linux fundamentals

**Rocky Linux drawbacks:**
- Steeper learning curve (DNF/YUM)
- Complex default security (SELinux)
- Smaller community for learning
- Enterprise-focused, less educational

### Disk Partitioning with LVM

**Advantages:**
- `/boot` separated prevents boot failures
- `/home` isolated protects user data
- `/var` and `/var/log` prevent log flooding
- LVM allows flexible resizing without rebooting
- Better security and system stability

### AppArmor vs SELinux

| Feature | AppArmor | SELinux |
|---------|----------|---------|
| Default on Debian | Yes | No |
| Learning curve | Moderate | Steep |
| Path-based | Yes | Type-based |
| Performance | Low overhead | Higher overhead |
| Best for | Learning | Enterprise |

AppArmor selected: simpler to understand and configure, native to Debian.

### UFW vs firewalld

| Feature | UFW | firewalld |
|---------|-----|-----------|
| Default on Debian | Yes | No (Rocky) |
| Simplicity | Very simple | Complex |
| Learning curve | Beginner-friendly | Steeper |
| Performance | Lightweight | Heavier |

UFW selected: easier to learn, sufficient for project needs.

### VirtualBox vs UTM

| Platform | VirtualBox | UTM |
|----------|-----------|-----|
| Windows/Linux/Mac | Yes | macOS only |
| x86 Performance | Excellent | Good |
| Apple Silicon | Poor (emulation) | Excellent (native) |
| Community | Large | Small |

VirtualBox recommended: standardized across 42 campuses, extensive resources.

## Resources

**Documentation:**
- [Debian Official](https://www.debian.org/doc/)
- [AppArmor Guide](https://gitlab.com/apparmor/apparmor/-/wikis/home)
- [UFW Docs](https://help.ubuntu.com/community/UFW)
- [SSH Config](https://man.openbsd.org/sshd_config)
- [PAM Documentation](http://www.linux-pam.org/)

**Learning:**
- [Linux Command Line](https://linuxcommand.org/)
- [DigitalOcean Tutorials](https://www.digitalocean.com/community/tutorials)
- [Debian Admin Handbook](https://debian-handbook.info/)

## AI Usage

**Tasks with AI assistance:**
- README structure and documentation
- Configuration explanations and comparisons
- System design rationale
- Testing procedures
- Monitoring script development

**Manual implementation:**
- Debian installation and configuration
- Security policy testing
- Troubleshooting and validation

---

**Before Submission:**
- [ ] Debian installed with LVM partitions
- [ ] SSH on port 4242, PermitRootLogin no
- [ ] UFW firewall enabled, port 4242 allowed
- [ ] Password policy enforced (30-day max, 2-day min)
- [ ] Account lockout (3 attempts, 600s timeout)
- [ ] Sudo logged to `/var/log/sudo/sudo.log`
- [ ] AppArmor enabled
- [ ] Monitoring script working
- [ ] All security measures functional

---

*Last Updated: January 29, 2025*
