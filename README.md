# Raspberry Pi Fleet Management with Ansible

Declarative configuration management for Breaking Bad themed Raspberry Pi fleet.

## Overview

This repository contains Ansible playbooks to configure and manage a distributed Raspberry Pi infrastructure across multiple locations (Hermosillo, Mexico and Chicago, IL).

## Infrastructure

### Control Machine
- **Gus** (100.103.213.74) - Raspberry Pi 5 running Ansible

### Managed Pis
- **Mike** (100.96.132.126) - NAS/Storage (Hermosillo)
- **Jesse** (100.72.62.109) - Projector system (Hermosillo)
- **Saul** (100.87.155.74) - Vaultwarden password manager (Hermosillo)
- **Heisenberg** (100.73.161.51) - Remote node (Chicago)

## File Structure
```
~/ansible-pis/
├── ansible.cfg              # Ansible configuration (fixes sudo issues)
├── inventory.yml            # Pi fleet inventory
├── gus-config.yml          # Control machine specific config
├── template-new-pi.yml     # Generic template for new Pis
├── TEMPLATE-USAGE.md       # Quick reference guide
└── README.md               # This file
```

## What Gets Configured

Each Pi receives:

### System Configuration
- ✅ **cloud-init** - Fully disabled and masked
- ✅ **Hostname** - Set via Breaking Bad naming scheme
- ✅ **Timezone** - America/Hermosillo
- ✅ **User** - bobstephen with passwordless sudo
- ✅ **SSH** - Enabled and secured

### Network & Security
- ✅ **Tailscale VPN** - Installed (manual connection required)
- ✅ **Firewall** - Basic security via sudo configuration

### Management Tools
- ✅ **Webmin** - Installed with Tailscale optimization
  - SSL disabled (Tailscale already encrypts)
  - Trusted Tailscale network (100.64.0.0/10)
  - CPU fix applied (prevents authentication loops)
- ✅ **Base packages** - vim, htop, curl, git

### Automation
- ✅ **Cron jobs**
  - 2:05 AM: System updates (apt update/upgrade/autoremove)
  - 3:05 AM: Daily reboot

## Quick Start

### Prerequisites
- Fresh Raspberry Pi OS install
- SSH enabled
- User created: bobstephen
- Pi connected to network

### Configure a New Pi

1. **Find the Pi's IP address**
```bash
   nmap -sn 192.168.25.0/24 | grep -B 2 "Raspberry Pi"
```

2. **Add to inventory**
```bash
   nano inventory.yml
```
   
   Add:
```yaml
       newpi:
         ansible_host: 192.168.25.xxx
         new_hostname: newpi
```

3. **SSH once to accept host key**
```bash
   ssh bobstephen@192.168.25.xxx
   exit
```

4. **Set up passwordless sudo**
```bash
   ssh bobstephen@192.168.25.xxx
   echo "bobstephen ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/bobstephen-nopasswd
   sudo chmod 0440 /etc/sudoers.d/bobstephen-nopasswd
   exit
```

5. **Run the template**
```bash
   ansible-playbook template-new-pi.yml -i inventory.yml --limit newpi --ask-pass
```

6. **Connect to Tailscale**
```bash
   ssh bobstephen@192.168.25.xxx
   sudo tailscale up --hostname=newpi
   # Open URL in browser and authorize
   tailscale ip -4
   exit
```

7. **Update inventory with Tailscale IP**
```bash
   nano inventory.yml
```
   
   Change:
```yaml
       newpi:
         ansible_host: 100.x.x.x  # Tailscale IP
         new_hostname: newpi
```

8. **Verify**
```bash
   ansible newpi -i inventory.yml -m ping --ask-pass
```

## Configuration Files

### ansible.cfg

Critical settings for passwordless sudo:
```ini
[defaults]
host_key_checking = False
timeout = 30

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
```

### inventory.yml
```yaml
all:
  hosts:
    gus:
      ansible_host: 127.0.0.1
      ansible_connection: local
      new_hostname: gus
    
    mike:
      ansible_host: 100.96.132.126
      new_hostname: mike
  
  vars:
    ansible_user: bobstephen
    ansible_python_interpreter: /usr/bin/python3
```

## Common Tasks

### Run playbook on specific Pi
```bash
ansible-playbook template-new-pi.yml -i inventory.yml --limit mike --ask-pass
```

### Test connection
```bash
ansible mike -i inventory.yml -m ping --ask-pass
```

### Check configuration without changing
```bash
ansible-playbook template-new-pi.yml -i inventory.yml --limit mike --check --ask-pass
```

### Run on all Hermosillo Pis
```bash
ansible-playbook template-new-pi.yml -i inventory.yml --limit 'all:!gus:!heisenberg' --ask-pass
```

### Access Webmin
- Via Tailscale: `http://[tailscale-ip]:10000`
- Via local: `http://192.168.25.xxx:10000`
- Username: bobstephen
- Password: (your password)

## Troubleshooting

### "Timeout waiting for privilege escalation"
- Ensure `ansible.cfg` exists in project directory
- Verify passwordless sudo: `ssh pi 'sudo -n whoami'`

### "Host key verification failed"
- SSH to the Pi manually once to accept the key
- Or set `host_key_checking = False` in ansible.cfg

### "Permission denied (publickey,password)"
- Verify username in inventory matches Pi's user
- Use `--ask-pass` flag

### Webmin high CPU usage
- Template already includes fix (SSL disabled, Tailscale trusted)
- Clear browser cache after accessing via Tailscale

### Tailscale not connecting
- Check auth key is still valid (90 days default)
- Manually run: `sudo tailscale up --hostname=piname`

## Key Features

### Idempotent
Run playbooks multiple times safely - only changes what needs changing.

### Version Controlled
All configs in git - track changes, roll back if needed.

### Tailscale Optimized
- Webmin CPU fix included
- SSL disabled (redundant with Tailscale encryption)
- Trusted network configuration

### Security Focused
- Passwordless sudo only for management user
- SSH enabled
- cloud-init disabled (prevents unwanted changes)
- Daily security updates

## Future Enhancements

Potential additions:
- [ ] Docker installation option
- [ ] K3s cluster setup
- [ ] Automated backup configuration
- [ ] Monitoring stack (Prometheus/Grafana)
- [ ] Log aggregation
- [ ] SSH key management

## Breaking Bad Pi Names

Remaining names available:
- Walter
- Skyler
- Hank
- Marie
- Gale
- Badger
- Skinny Pete
- Tuco
- Combo

## Notes

- **Control machine (Gus)** should always be managed with `gus-config.yml`
- **New Pis** use `template-new-pi.yml`
- **Existing Pis** can be converted by running the template
- **Remote Pi (Heisenberg)** - Always use Tailscale IP, never run risky operations without testing locally first

## Maintenance

### Update all Pis
```bash
# Dry run
ansible all -i inventory.yml -m apt -a "upgrade=dist" --check --ask-pass

# Actually update
ansible all -i inventory.yml -m apt -a "upgrade=dist update_cache=yes" --ask-pass
```

### Reboot all Pis
```bash
ansible all -i inventory.yml -m reboot --ask-pass
```

### Check Tailscale status on all Pis
```bash
ansible all -i inventory.yml -m command -a "tailscale status" --ask-pass
```

## Credits

- Platform: Ansible
- VPN: Tailscale
- Management: Webmin
- OS: Raspberry Pi OS (Debian Trixie)

## Version History

- **v1.0** (2026-01-05) - Initial production-ready configuration
  - Gus control machine config
  - Generic Pi template
  - Mike successfully configured from scratch
  - Full documentation

---

**Last Updated:** 2026-01-05  
**Maintained By:** Bob Stephen

