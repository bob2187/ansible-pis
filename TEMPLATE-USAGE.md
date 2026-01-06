# Pi Configuration Template Usage

## Files
- `gus-config.yml` - Gus (control machine) specific config
- `template-new-pi.yml` - Generic template for new Pis
- `inventory.yml` - Current inventory

## How to Configure a New Pi

### Step 1: Add to Inventory
Edit `inventory.yml`:
```yaml
all:
  hosts:
    gus:
      ansible_host: 127.0.0.1
      ansible_connection: local
      new_hostname: gus
    
    mike:  # NEW PI
      ansible_host: 192.168.25.xxx  # Local IP for first run
      new_hostname: mike
```

### Step 2: Run Template
```bash
# First run with local IP
ansible-playbook template-new-pi.yml -i inventory.yml --limit mike

# Manually connect Tailscale
ssh bobstephen@192.168.25.xxx
sudo tailscale up --hostname=mike
# Click link in browser

# Get Tailscale IP
tailscale ip -4
```

### Step 3: Update Inventory with Tailscale IP
```yaml
    mike:
      ansible_host: 100.x.x.x  # Tailscale IP
      new_hostname: mike
```

### Step 4: Future runs use Tailscale
```bash
ansible-playbook template-new-pi.yml -i inventory.yml --limit mike
```

## Your Pis
- **gus** (100.103.213.74) - Control machine
- **naspi** (100.85.39.44) - Storage → rename to "mike"
- **projectorpi** (100.72.62.109) - Projector → rename to "jesse"
- **vaultpi** (100.87.155.74) - Vaultwarden → rename to "saul"
- **winchesterpi** (100.73.161.51) - Chicago remote → rename to "heisenberg"
```

Save it!

## Summary

You now have:
```
~/ansible-pis/
├── inventory.yml           # Your Pi inventory
├── gus-config.yml         # Gus-specific (control machine)
├── template-new-pi.yml    # Generic template for any new Pi
└── TEMPLATE-USAGE.md      # How to use the template
