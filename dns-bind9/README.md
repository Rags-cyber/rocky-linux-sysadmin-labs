# DNS Server Configuration — BIND9 on Rocky Linux

> Full setup of a BIND9 DNS server on Rocky Linux with forward and reverse zone configuration, tested from both Rocky Linux and Ubuntu clients.

**Environment:** Rocky Linux (Server) + Ubuntu (Client) inside Oracle VirtualBox  
**Domain used:** `shadow.com`  
**Server IP:** `192.168.200.5`  
**Client IP:** `192.168.200.4`

📝 **Full write-up with screenshots:** [Medium Article](https://medium.com/@gurungatwork98/linux-dns-configuration-da5b6426a8cb)

---

## Table of Contents

- [Part 1: Rocky Linux — Server Setup](#part-1-rocky-linux--server-setup)
- [Part 2: Ubuntu Client — DNS Configuration](#part-2-ubuntu-client--dns-configuration)
- [Part 3: Testing DNS Connectivity](#part-3-testing-dns-connectivity)
- [Config Files](#config-files)

---

## Part 1: Rocky Linux — Server Setup

### Step 1: Check OS release

```bash
cat /etc/os-release
# or
uname -a
```

---

### Step 2: Update the system

```bash
sudo dnf update -y
```

Then reboot your system.

---

### Step 3: Install BIND9

```bash
sudo dnf install -y bind bind-utils
```

This will download all the required packages.

---

### Step 4: Enable and start the DNS service

```bash
sudo systemctl enable named --now
```

> `--now` immediately starts the service alongside enabling it.

Verify it started:

```bash
sudo systemctl start named
```

---

### Step 5: Check service status

```bash
sudo systemctl status named
```

Confirm the service shows as **active (running)**.

---

### Step 6: Locate the named.conf file

```bash
ll /etc/named.conf
```

---

### Step 7: Backup named.conf

```bash
sudo cp -p /etc/named.conf /etc/named.conf.bak
```

> `-p` preserves the original file's permissions and ownership.

Confirm the backup exists:

```bash
ls -l /etc | grep named
```

---

### Step 8: Open named.conf for editing

Login as root first:

```bash
sudo su
```

Then open the config:

```bash
nano /etc/named.conf
```

---

### Step 9: Edit listen-on and allow-query

Find these two lines and change their values to `any`:

```
listen-on port 53 { any; };
allow-query     { any; };
```

- `listen-on port 53` — allows other machines on the network to reach your DNS.
- `allow-query` — allows any client to query your DNS server.

---

### Step 10: Add forward and reverse zone blocks

Find the `zone "." IN { };` line and add the following **below** it:

**Forward zone:**

```
zone "shadow.com" IN {
    type master;
    file "fwd.shadow.com.db";
    allow-update { none; };
};
```

> This is the primary authority for `shadow.com`. Records are loaded from `fwd.shadow.com.db`. Dynamic updates are disabled.

**Reverse zone:**

```
zone "200.168.192.in-addr.arpa" IN {
    type master;
    file "rvs.200.168.192.db";
    allow-update { none; };
};
```

> This is the master reverse DNS zone for the `192.168.200.0` network. PTR records are stored in `rvs.200.168.192.db`. Dynamic updates are disabled.

Save the file.

---

### Step 11: Check named.conf for errors

```bash
named-checkconf
```

If there are no errors, the command returns no output (that is correct).

---

### Step 12: Navigate to the zone files directory

```bash
cd /var/named
ls -l
```

> ⚠️ Important step — all zone files must be created here.

---

### Step 13: Create the forward lookup zone file

```bash
nano fwd.shadow.com.db
```

Use the Red Hat zone file template as a starting reference:  
https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/reference_guide/s2-bind-zone-examples

---

### Step 14: Configure the forward lookup zone file

Edit the file to match the following:

```
$TTL 86400
@   IN  SOA     server.shadow.com.  root.shadow.com. (
                    2024010101  ; Serial
                    3600        ; Refresh
                    1800        ; Retry
                    604800      ; Expire
                    86400 )     ; Minimum TTL

@       IN  NS      server.shadow.com.
server  IN  A       192.168.200.5
client  IN  A       192.168.200.4
```

**Explanation:**

| Directive | Meaning |
|---|---|
| `$TTL 86400` | Default cache time for DNS records — 1 day |
| `@ IN SOA` | Start of Authority — defines primary DNS and admin email |
| `Serial` | Version number — increment whenever you change the zone |
| `Refresh` | How often a secondary server checks for updates |
| `Retry` | How long to wait before retrying after a failed refresh |
| `Expire` | How long the zone is valid if the master is unreachable |
| `Minimum` | Negative cache TTL for failed DNS queries |
| `@ IN NS` | Declares `server.shadow.com` as the authoritative name server |
| `server IN A` | Maps `server.shadow.com` → `192.168.200.5` |
| `client IN A` | Maps `client.shadow.com` → `192.168.200.4` |

---

### Step 15: Verify the forward zone file is saved

```bash
ls -l /var/named
```

> ⚠️ Important — confirm `fwd.shadow.com.db` appears in the listing.

---

### Step 16: Create the reverse lookup zone file

```bash
nano rvs.200.168.192.db
```

Reference template:  
https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/reference_guide/s2-bind-configuration-zone-reverse

---

### Step 17: Configure the reverse lookup zone file

```
$TTL 86400
@   IN  SOA     server.shadow.com.  root.shadow.com. (
                    2024010101  ; Serial
                    3600        ; Refresh
                    1800        ; Retry
                    604800      ; Expire
                    86400 )     ; Minimum TTL

@   IN  NS      server.shadow.com.
5   IN  PTR     server.shadow.com.
4   IN  PTR     client.shadow.com.
```

**Explanation:**

| Directive | Meaning |
|---|---|
| `5 IN PTR` | Maps `192.168.200.5` → `server.shadow.com` |
| `4 IN PTR` | Maps `192.168.200.4` → `client.shadow.com` |

This reverse zone file resolves IP addresses in the `192.168.200.0` network back to hostnames.

---

### Step 18: Verify both zone files are saved

```bash
cd /var/named
ls -l
```

Both `fwd.shadow.com.db` and `rvs.200.168.192.db` should be listed.

---

### Step 19: Restart and verify named

```bash
systemctl restart named
systemctl status named
```

---

### Step 20: Open firewall ports for DNS

```bash
sudo firewall-cmd --permanent --add-port=53/tcp
sudo firewall-cmd --permanent --add-port=53/udp
firewall-cmd --reload
```

Then point the server's own DNS to itself via NetworkManager:

```bash
sudo nano /etc/NetworkManager/system-connections/enp0s3.nmconnection
```

Set DNS to `192.168.200.5` (Rocky DNS) and `8.8.8.8` (Google DNS).

---

### Step 21: Verify resolv.conf

```bash
nano /etc/resolv.conf
```

Confirm the Rocky DNS server IP appears as the first `nameserver` entry.

---

### Step 22: Final restart

```bash
sudo systemctl restart named
```

DNS server setup is now complete.

---

## Part 2: Ubuntu Client — DNS Configuration

### Method 1: GUI (Network Settings)

1. Open **Settings → Network**
2. Click the gear icon next to your active connection
3. Go to the **IPv4** tab
4. Turn off **Automatic DNS**
5. Enter `192.168.200.5` (Rocky DNS) and `8.8.8.8` (Google DNS)
6. Click **Apply**

---

### Method 2: NetworkManager (CLI)

**Step 1:** Install NetworkManager

```bash
apt install network-manager -y
```

**Step 2:** Enable and start the service

```bash
sudo systemctl enable NetworkManager
sudo systemctl start NetworkManager
sudo systemctl status NetworkManager
```

**Step 3:** Check available connections

```bash
nmcli connection show
```

**Step 4:** Configure DNS on the connection

```bash
nmcli connection modify netplan-enp0s3 ipv4.dns "192.168.200.5 8.8.8.8"
nmcli connection modify netplan-enp0s3 ipv4.dns-search "shadow.com"
nmcli connection modify netplan-enp0s3 ipv4.ignore-auto-dns yes
nmcli connection down netplan-enp0s3 && nmcli connection up netplan-enp0s3
```

**Step 5:** Verify DNS is applied

```bash
nmcli device show enp0s3
```

**Step 6:** Check resolv.conf

```bash
nano /etc/resolv.conf
```

Confirm `nameserver 192.168.200.5` appears in the file.

**Step 7:** Test connectivity

```bash
ping server.shadow.com
```

---

### Method 3: Permanent DNS Setup via resolvconf

**Step 1:** Install resolvconf

```bash
sudo apt install resolvconf
```

**Step 2:** Edit NetworkManager config

```bash
nano /etc/NetworkManager/NetworkManager.conf
```

Add under `[main]`:

```
dns=none
```

This allows you to configure DNS manually without NetworkManager overwriting it.

**Step 3:** Verify resolv.conf

```bash
cat /etc/resolv.conf
```

Confirm `nameserver` and `search` entries show the Rocky DNS server.

---

## Part 3: Testing DNS Connectivity

### nslookup

```bash
nslookup server.shadow.com
nslookup client.shadow.com
```

Expected: returns the correct IP address for each hostname.

---

### ping

```bash
ping server.shadow.com
ping client.shadow.com
```

Expected: replies from both server and client — confirms DNS resolution is working.

---

### dig (forward lookup)

```bash
dig server.shadow.com
dig client.shadow.com
```

---

### dig -x (reverse lookup)

```bash
dig -x 192.168.200.5
dig -x 192.168.200.4
```

Expected: PTR records resolve back to `server.shadow.com` and `client.shadow.com` — confirms the reverse zone is working.

---

## Config Files

| File | Description |
|---|---|
| [`named.conf`](./named.conf) | Main BIND9 configuration with zone declarations |
| [`fwd.shadow.com.db`](./fwd.shadow.com.db) | Forward lookup zone file |
| [`rvs.200.168.192.db`](./rvs.200.168.192.db) | Reverse lookup zone file |

---

## What I Learned

- How DNS forward and reverse resolution works at the zone file level
- Difference between A records (forward) and PTR records (reverse)
- How to configure DNS clients on both Rocky Linux and Ubuntu
- How to validate DNS using `nslookup`, `ping`, `dig`, and `dig -x`
- How to persist DNS settings across reboots using NetworkManager and resolvconf
