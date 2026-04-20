# Static IP Configuration on Rocky Linux & Ubuntu

> Configuring a static IP address on Rocky Linux using NetworkManager and verifying network settings on Ubuntu Client.

**Environment:** Rocky Linux (Server) + Ubuntu (Client) inside Oracle VirtualBox  
**Static IP set:** `192.168.200.5/24`  
**Gateway:** `192.168.200.1`  
**DNS:** `1.1.1.1`

📝 **Full write-up with screenshots:** [Medium Article](https://medium.com/@gurungatwork98)

---

## Table of Contents

- [Part 1: Rocky Linux — Static IP Setup](#part-1-rocky-linux--static-ip-setup)
- [Part 2: Ubuntu Client — Network Verification](#part-2-ubuntu-client--network-verification)
- [What I Learned](#what-i-learned)

---

## Part 1: Rocky Linux — Static IP Setup

### Step 1: Check OS release

```bash
cat /etc/os-release
# or
uname -a
```

### Step 2: Update and upgrade the system

```bash
sudo dnf update -y
sudo dnf upgrade -y
reboot
```

### Step 3: Verify NetworkManager is installed

```bash
sudo rpm -q NetworkManager
```

Enable and start NetworkManager:

```bash
sudo systemctl enable NetworkManager
sudo systemctl start NetworkManager
sudo systemctl status NetworkManager
```

### Step 4: Check current IP address

```bash
ifconfig
# or
ip addr
```

### Step 5: Check overall network status

```bash
sudo nmcli general status      # Overall NetworkManager status
sudo nmcli device status       # List all network devices and their state
```

### Step 6: Find the NIC config file

```bash
sudo ls -l /etc/NetworkManager/system-connections
```

The network interface config file is named `enp0s3.nmconnection`.

### Step 7: Open the config file

```bash
sudo nano /etc/NetworkManager/system-connections/enp0s3.nmconnection
```

### Step 8: Set the static IP, gateway, and DNS

Find the `[ipv4]` section and edit it to:

```ini
[ipv4]
method=manual
address1=192.168.200.5/24,192.168.200.1
dns=1.1.1.1;
```

**Explanation:**

| Setting | Value | Meaning |
|---|---|---|
| `method=manual` | — | Use static IP instead of DHCP |
| `address1` | `192.168.200.5/24` | IP address with subnet mask |
| `,192.168.200.1` | — | Default gateway (after the comma) |
| `dns` | `1.1.1.1` | Cloudflare DNS server |

Press `Ctrl+X`, then `Y`, then `Enter` to save.

### Step 9: Restart NetworkManager to apply changes

```bash
sudo systemctl restart NetworkManager
sudo systemctl status NetworkManager
```

### Step 10: Verify the static IP is applied

```bash
nmcli
```

If the changes are not reflected, reboot:

```bash
reboot
```

---

## Part 2: Ubuntu Client — Network Verification

### Step 1: Update Ubuntu as root

```bash
apt-get update && apt-get upgrade
```

### Step 2: Check if NetworkManager is installed

```bash
NetworkManager --version
```

### Step 3: Start NetworkManager

```bash
sudo systemctl enable NetworkManager
sudo systemctl start NetworkManager
sudo systemctl status NetworkManager
```

### Step 4: Check IP address

```bash
ifconfig
```

### Step 5: Check all network details

```bash
sudo nmcli
sudo nmcli general status
sudo nmcli device status
nmcli connection show
```

---

## What I Learned

- Static IP configuration on modern Rocky Linux is done through the `.nmconnection` file, not the old `/etc/sysconfig/network-scripts/` method
- NetworkManager must be restarted after editing the config file — changes are not applied automatically
- The `address1` field in `.nmconnection` combines both the IP/prefix and the gateway in one line separated by a comma
- `nmcli` is a powerful tool for verifying network config without needing to open any files
