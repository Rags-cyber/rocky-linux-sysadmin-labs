# Secure Shell (SSH) Configuration on Rocky Linux

> Setup and hardening of SSH on Rocky Linux — disabling root login, opening firewall ports, and testing remote access from Ubuntu.

**Environment:** Rocky Linux (Server) + Ubuntu (Client) inside Oracle VirtualBox  
**Server IP:** `192.168.200.5`  
**SSH Port:** `22`

📝 **Full write-up with screenshots:** [Medium Article](https://medium.com/@gurungatwork98/how-to-configure-and-harden-ssh-on-rocky-linux-d530269ea955)

---

## Table of Contents

- [Part 1: Rocky Linux — SSH Server Setup](#part-1-rocky-linux--ssh-server-setup)
- [Part 2: Ubuntu Client — SSH Testing](#part-2-ubuntu-client--ssh-testing)
- [What I Learned](#what-i-learned)

---

## Part 1: Rocky Linux — SSH Server Setup

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

### Step 3: Verify SSH is installed

SSH (OpenSSH) comes pre-installed on Rocky Linux. Verify the version:

```bash
sshd --version
```

### Step 4: Open the SSH configuration file

```bash
sudo nano /etc/ssh/sshd_config
```

### Step 5: Enable SSH on port 22

Find the following line and remove the `#` to uncomment it:

```
Port 22
```

### Step 6: Disable root SSH login

Find:

```
#PermitRootLogin prohibit-password
```

Below it, add:

```
PermitRootLogin no
```

> This prevents anyone from logging in directly as root over SSH. Users must log in with their own account and use `sudo` if needed.

### Step 7: Enable and start SSH, then check status

```bash
sudo systemctl enable sshd
sudo systemctl start sshd
sudo systemctl status sshd
```

### Step 8: Allow SSH through the firewall

```bash
sudo firewall-cmd --zone=public --permanent --add-service=ssh
sudo firewall-cmd --add-port=22/tcp --permanent
sudo firewall-cmd --reload
```

### Step 9: Restart SSH to apply config changes

```bash
sudo systemctl restart sshd
```

---

## Part 2: Ubuntu Client — SSH Testing

### Step 1: Connect to Rocky Server via SSH

```bash
ssh server@192.168.200.5
```

- `server` — the username on Rocky Linux
- `192.168.200.5` — Rocky's IP address

Accept the fingerprint prompt on first connection by typing `yes`.

### Step 2: Verify access and run commands remotely

Once logged in, you are now controlling the Rocky Server from Ubuntu:

```bash
ls -l         # List directories on Rocky
whoami        # Confirm the logged-in user
hostname      # Confirm you are on Rocky
```

---

## Key sshd_config Settings Reference

| Setting | Value | Meaning |
|---|---|---|
| `Port` | `22` | Default SSH port |
| `PermitRootLogin` | `no` | Block direct root login |
| `PasswordAuthentication` | `yes` | Allow password login (set to `no` for key-only) |
| `PubkeyAuthentication` | `yes` | Allow SSH key login |
| `AllowUsers` | `username` | Restrict SSH to specific users only |

---

## What I Learned

- SSH is pre-installed on Rocky Linux — it just needs to be enabled and configured
- Disabling root login is a fundamental security hardening step — attackers always try root first
- Both the firewall service rule (`--add-service=ssh`) and the port rule (`--add-port=22/tcp`) are used for redundancy
- `sshd_config` changes only take effect after restarting `sshd`
- The first time you SSH into a server, the client saves the server's fingerprint — this protects against man-in-the-middle attacks on future connections
