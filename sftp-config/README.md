# SFTP Configuration on Rocky Linux

> Configuring a restricted SFTP-only server on Rocky Linux using OpenSSH, with a dedicated SFTP group, chroot directory, and SSH login disabled for SFTP users. Tested from Ubuntu.

**Environment:** Rocky Linux (Server) + Ubuntu (Client) inside Oracle VirtualBox  
**Server IP:** `192.168.200.5`  
**SFTP user:** `sftpuser`  
**SFTP group:** `sftpusers`  
**Chroot directory:** `/sftp`

📝 **Full write-up with screenshots:** [Medium Article](https://medium.com/@gurungatwork98/how-to-configure-a-secure-sftp-server-on-rocky-linux-2840aa7c2e57)

---

## Table of Contents

- [Part 1: Rocky Linux — SFTP Server Setup](#part-1-rocky-linux--sftp-server-setup)
- [Part 2: Ubuntu Client — SFTP Testing](#part-2-ubuntu-client--sftp-testing)
- [What I Learned](#what-i-learned)

---

## Part 1: Rocky Linux — SFTP Server Setup

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

### Step 3: Create a dedicated SFTP group

```bash
sudo groupadd sftpusers
```

### Step 4: Create an SFTP-only user with SSH disabled

```bash
sudo useradd -m -g sftpusers -s /sbin/nologin sftpuser
```

| Flag | Meaning |
|---|---|
| `-m` | Creates a home directory for the user |
| `-g sftpusers` | Adds the user to the `sftpusers` group |
| `-s /sbin/nologin` | Disables SSH shell login for this user |

### Step 5: Set a password for the SFTP user

```bash
sudo passwd sftpuser
```

### Step 6: Create the SFTP upload directory

```bash
sudo mkdir -p /sftp/sftpusers
```

### Step 7: Set ownership and permissions

```bash
sudo chmod 755 /sftp                              # Secure permission on chroot directory
sudo chown sftpuser:sftpusers /sftp/sftpusers     # Give sftpuser ownership of their directory
sudo chmod 755 /sftp/sftpusers                    # Full access for user, read-only for others
```

> The chroot directory `/sftp` must be owned by root and not writable by the group — this is an SSH security requirement.

### Step 8: Edit the SSH config file

```bash
sudo nano /etc/ssh/sshd_config
```

Make sure this line is **not commented**:

```
Subsystem sftp internal-sftp
```

### Step 9: Add the SFTP-only block at the end of the file

```
Match Group sftpusers
    ChrootDirectory /sftp
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no
```

**Explanation:**

| Directive | Meaning |
|---|---|
| `Match Group sftpusers` | Applies the following rules only to users in this group |
| `ChrootDirectory /sftp` | Locks users inside the `/sftp` directory — they cannot navigate outside |
| `ForceCommand internal-sftp` | Forces SFTP mode — shell access is completely blocked |
| `AllowTcpForwarding no` | Disables port forwarding for SFTP users |
| `X11Forwarding no` | Disables graphical forwarding |

### Step 10: Restart SSH

```bash
sudo systemctl restart sshd
```

---

## Part 2: Ubuntu Client — SFTP Testing

### Step 1: Verify SSH is blocked for the SFTP user

```bash
ssh sftpuser@192.168.200.5
```

Expected output: `This service allows sftp connections only.` (or similar rejection message). This confirms SSH shell access is disabled.

### Step 2: Connect via SFTP

```bash
sftp sftpuser@192.168.200.5
```

Enter the password when prompted.

### Step 3: Upload a file to the server

```bash
put sftp.txt
```

> You can only upload to the `/sftp/sftpusers` directory. Navigate there first if needed.

### Step 4: Download a file from the server

First, create a file on Rocky Server to download:

```bash
# On Rocky Server
cd /sftp/sftpusers
sudo touch download.txt
```

Then, from the Ubuntu SFTP session:

```bash
get download.txt
```

Verify the file was downloaded on Ubuntu:

```bash
ls -l
```

### Step 5: Exit the SFTP session

```bash
exit
```

---

## What I Learned

- SFTP runs over SSH and is more secure than plain FTP — all data is encrypted
- The chroot jail prevents SFTP users from seeing or accessing anything outside their designated directory
- The chroot directory itself must be owned by root and not group-writable — otherwise SSH rejects the chroot
- Using `/sbin/nologin` as the shell blocks interactive SSH login while still allowing SFTP
- `ForceCommand internal-sftp` is the key line that ensures the user can only do SFTP, even if they somehow bypass the shell restriction
