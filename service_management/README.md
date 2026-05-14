# systemd Service Management

A hands-on reference for managing services, targets, and custom units using `systemctl` on Linux (Rocky Linux / RHEL-based).

📝 **Full write-up with screenshots:** [Medium Article](https://medium.com/@gurungatwork98/mastering-systemd-managing-services-targets-and-custom-units-on-linux-32403fb35e7f?postPublishedType=initial)
---

## Contents

- [Inspecting Unit Files](#inspecting-unit-files)
- [Masking and Unmasking Units](#masking-and-unmasking-units)
- [Working with Targets](#working-with-targets)
- [Shutdown and Reboot](#shutdown-and-reboot)
- [Creating a Custom Service](#creating-a-custom-service-backup-service)
- [File Structure](#file-structure)

---

## Inspecting Unit Files

```bash
# View the unit file for a service
sudo systemctl cat dhcpd.service

# Show dependency tree
systemctl list-dependencies dhcpd.service

# Show all unit properties (key=value format)
systemctl show dhcpd.service
```

---

## Masking and Unmasking Units

Masking links a unit to `/dev/null`, making it completely unstartable — even as a dependency.

```bash
# Mask a unit (block it entirely)
sudo systemctl mask dhcpd.service

# Unmask to restore normal behaviour
sudo systemctl unmask dhcpd.service
```

> **Mask vs Disable:** `disable` stops a service from starting at boot. `mask` prevents it from running under any circumstances.

---

## Working with Targets

Targets are systemd's equivalent of SysV runlevels — they define the system state and group related units.

```bash
# Get the default boot target
systemctl get-default

# List all available targets
systemctl list-units --type=target --all

# List only active targets
systemctl list-units --type=target

# Switch to rescue (single-user) mode
sudo systemctl rescue

# Switch back to graphical target
sudo systemctl isolate graphical.target
```

**Common targets:**

| Target | Description |
|--------|-------------|
| `multi-user.target` | Multi-user, no GUI (typical for servers) |
| `graphical.target` | Multi-user with GUI |
| `rescue.target` | Single-user minimal environment |
| `emergency.target` | Minimal shell, no services |

---

## Shutdown and Reboot

```bash
sudo systemctl halt      # Stop the system
sudo systemctl poweroff  # Full shutdown
sudo systemctl reboot    # Restart
```

---

## Creating a Custom Service: Backup Service

### 1. Create the script

```bash
sudo nano /usr/local/bin/backup.sh
```

```bash
Check out my backup.sh file in bash-scripting.
```

```bash
sudo chmod +x /usr/local/bin/backup.sh
```

### 2. Create the unit file

```bash
sudo nano /etc/systemd/system/backup.service
```

```ini
[Unit]
Description=Custom Backup Service
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
User=root

[Install]
WantedBy=multi-user.target
```

### 3. Enable and start

```bash
# Reload systemd to pick up the new unit file
sudo systemctl daemon-reload

# Enable to start at boot
sudo systemctl enable backup.service

# Start immediately
sudo systemctl start backup.service

# Verify it ran
sudo systemctl status backup.service
```

A `oneshot` service will show `Active: inactive (dead)` after a clean run — that's expected behaviour.

---

## File Structure

```
/etc/systemd/system/
└── backup.service          # Custom unit file

/usr/local/bin/
└── backup.sh               # Backup script

/backup/
└── home-YYYY-MM-DD.tar.gz  # Output archives
```

---

## Environment

- OS: Rocky Linux 10 / RHEL-based
- Init system: systemd

---
*Part of my Linux System Administration series. Each folder in this repo corresponds to a lab covered in my Medium articles.*
