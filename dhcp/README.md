# DHCP Server Configuration on Rocky Linux

> Setup of an ISC DHCP server on Rocky Linux to automatically assign IP addresses to clients on the network, replacing VirtualBox's built-in DHCP.

**Environment:** Rocky Linux (Server) + Ubuntu (Client) inside Oracle VirtualBox  
**Network:** `192.168.200.0/24`  
**IP Range assigned:** `192.168.200.100 – 192.168.200.200`  
**Server IP:** `192.168.200.5` (static)

📝 **Full write-up with screenshots:** [Medium Article](https://medium.com/@gurungatwork98)

---

## Table of Contents

- [Part 1: Rocky Linux — DHCP Server Setup](#part-1-rocky-linux--dhcp-server-setup)
- [Part 2: VirtualBox Network Configuration](#part-2-virtualbox-network-configuration)
- [Part 3: Verify IP Assignment on Ubuntu Client](#part-3-verify-ip-assignment-on-ubuntu-client)
- [Config File](#config-file)
- [What I Learned](#what-i-learned)

---

## Part 1: Rocky Linux — DHCP Server Setup

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

### Step 3: Check current IP address

```bash
ip addr
```

### Step 4: Check network connection settings

```bash
sudo ls -l /etc/NetworkManager/system-connections
```

### Step 5: Verify network config file

```bash
sudo nano /etc/NetworkManager/system-connections/enp0s3.nmconnection
```

Confirm the static IP and DNS settings are correct before proceeding.

### Step 6: Install the DHCP server

```bash
sudo dnf install -y dhcp-server
```

> **If you get an error:** The `dhcp-server` package may be in the disabled Devel repo on Rocky Linux 10. Enable it with:

```bash
sudo dnf config-manager --set-enabled devel
sudo dnf makecache
sudo dnf install -y dhcp-server
```

### Step 7: Edit the DHCP configuration file

```bash
sudo nano /etc/dhcp/dhcpd.conf
```

Add the following configuration:

```
default-lease-time 3600;
max-lease-time 86400;
authoritative;

subnet 192.168.200.0 netmask 255.255.255.0 {
    range 192.168.200.100 192.168.200.200;
    option routers 192.168.200.1;
    option subnet-mask 255.255.255.0;
    option domain-name-servers 192.168.200.5, 8.8.8.8;
}
```

**Explanation:**

| Directive | Meaning |
|---|---|
| `default-lease-time 3600` | Default lease duration in seconds (1 hour) |
| `max-lease-time 86400` | Maximum lease duration in seconds (24 hours) |
| `authoritative` | Declares this as the official DHCP server — prevents rogue DHCP servers |
| `range` | Pool of IPs the server can hand out to clients |
| `option routers` | Default gateway for clients |
| `option subnet-mask` | Network subnet mask |
| `option domain-name-servers` | DNS servers pushed to clients |

### Step 8: Enable the DHCP service permanently

```bash
sudo systemctl enable --now dhcpd.service
```

### Step 9: Start DHCP and check status

```bash
sudo systemctl start dhcpd
sudo systemctl status dhcpd
```

### Step 10: Allow DHCP through the firewall

```bash
sudo firewall-cmd --permanent --add-service=dhcp
sudo firewall-cmd --reload
```

---

## Part 2: VirtualBox Network Configuration

### Step 1: Disable VirtualBox's built-in DHCP

In VirtualBox:
- Go to **Tools → Network Manager → NAT Network**
- **Uncheck Enable DHCP**

From this point, clients will receive their IP from Rocky Server instead of VirtualBox.

### Step 2: Confirm RockyServer is on NAT Network

- Go to **RockyServer → Settings → Network**
- Confirm it is attached to your **NAT Network**

### Step 3: Confirm Ubuntu is also on the same NAT Network

- Go to **Ubuntu → Settings → Network**
- Confirm it is on the same **NAT Network** as Rocky

---

## Part 3: Verify IP Assignment on Ubuntu Client

### Step 1: Restart NetworkManager on Rocky and verify

```bash
sudo systemctl restart NetworkManager
sudo systemctl status NetworkManager
nmcli
```

### Step 2: Start Ubuntu Client

Boot Ubuntu while Rocky Server is running. Ubuntu should automatically receive an IP from Rocky's DHCP server.

Verify the IP is within the configured range (`192.168.200.100–200`):

```bash
ip addr
```

### Step 3: Test connectivity between server and client

On Ubuntu, confirm DNS resolv.conf is pointing to Rocky:

```bash
cat /etc/resolv.conf
```

Then ping the Rocky server:

```bash
ping server.shadow.com
```

### Step 4: Test internet connectivity

Open a browser on Ubuntu and load any website to confirm internet is working through the Rocky DHCP setup.

---

## Config File

| File | Description |
|---|---|
| [`dhcpd.conf`](./dhcpd.conf) | Main DHCP server configuration |

---

## What I Learned

- How DHCP leases work — default vs. maximum lease times
- Why `authoritative` is important to prevent IP conflicts from rogue DHCP servers
- How VirtualBox's internal DHCP must be disabled for a custom DHCP server to take over
- How to verify DHCP lease assignment on a Linux client using `ip addr`
- How DHCP and DNS work together — DHCP pushes DNS server IPs to clients automatically
