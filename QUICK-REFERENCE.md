# Quick Reference

## Most Common Commands

### New Pi Setup
```bash
# 1. Add to inventory, SSH to accept key, setup sudo
# 2. Run template
ansible-playbook template-new-pi.yml -i inventory.yml --limit newpi --ask-pass

# 3. Connect Tailscale
ssh bobstephen@IP "sudo tailscale up --hostname=newpi"

# 4. Update inventory with Tailscale IP
```

### Test Connection
```bash
ansible mike -i inventory.yml -m ping --ask-pass
```

### Run Playbook
```bash
ansible-playbook template-new-pi.yml -i inventory.yml --limit mike --ask-pass
```

### Check Mode (Dry Run)
```bash
ansible-playbook template-new-pi.yml -i inventory.yml --limit mike --check --ask-pass
```

### Access Webmin
```
http://[tailscale-ip]:10000
```

## File Locations

- Configs: `~/ansible-pis/`
- Webmin: Port 10000
- Cron: 2:05 AM updates, 3:05 AM reboot

## Tailscale IPs

- Gus: 100.103.213.74
- Mike: 100.96.132.126
- Jesse: 100.72.62.109
- Saul: 100.87.155.74
- Heisenberg: 100.73.161.51
