# Samba File Sharing — Rocky Linux to Ubuntu

> Setup of a Samba server on Rocky Linux to share files with a Windows/Linux network, with access tested from Ubuntu using smbclient.

**Environment:** Rocky Linux (Server) + Ubuntu (Client) inside Oracle VirtualBox

📝 **Full write-up with screenshots:** [Medium Article](https://medium.com/@gurungatwork98/setting-up-a-samba-file-server-on-rocky-linux-a-sysadmins-walkthrough-7d05d4e7a55b)

---

## Table of Contents

- [Part 1: Rocky Linux — Samba Server Setup](#part-1-rocky-linux--samba-server-setup)
- [Part 2: Ubuntu Client — Samba Access](#part-2-ubuntu-client--samba-access)
- [Config File](#config-file)
- [What I Learned](#what-i-learned)

---

## Part 1: Rocky Linux — Samba Server Setup

### Step 1: Install Samba

```bash
sudo dnf install samba samba-common samba-client -y
```

### Step 2: Create a shared directory with proper permissions

```bash
sudo mkdir -p /srv/samba/shared
sudo chmod 0770 /srv/samba/shared
sudo chown root:sambashare /srv/samba/shared
```

### Step 3: Configure /etc/samba/smb.conf

```bash
sudo nano /etc/samba/smb.conf
```

### Step 4: Edit the global section

At the top of the file, in the `[global]` section, confirm or set:

```ini
[global]
   workgroup = WORKGROUP
   server string = Samba Server
   netbios name = rockyserver
   security = user
   map to guest = bad user
   dns proxy = no
```

### Step 5: Add a shared folder section at the bottom

```ini
[shared]
   path = /srv/samba/shared
   browsable = yes
   writable = yes
   guest ok = no
   valid users = @sambashare
   create mask = 0660
   directory mask = 0770
```

**Explanation:**

| Option | Meaning |
|---|---|
| `path` | Directory being shared |
| `browsable = yes` | Share is visible when browsing the network |
| `writable = yes` | Allows file uploads and edits |
| `guest ok = no` | Requires authentication — no anonymous access |
| `valid users = @sambashare` | Only users in the `sambashare` group can access |
| `create mask` | Permissions for newly created files |
| `directory mask` | Permissions for newly created directories |

### Step 6: Add a Samba user

First, the user must exist as a system user:

```bash
sudo useradd -M -s /sbin/nologin sambauser
sudo usermod -aG sambashare sambauser
```

Then set a Samba password:

```bash
sudo smbpasswd -a sambauser
```

### Step 7: Set SELinux context for the shared directory

```bash
sudo setsebool -P samba_export_all_rw on
sudo chcon -t samba_share_t /srv/samba/shared
```

### Step 8: Enable and start Samba services

```bash
sudo systemctl enable smb nmb --now
sudo systemctl start smb nmb
sudo systemctl status smb
```

### Step 9: Configure firewall for Samba

```bash
sudo firewall-cmd --permanent --add-service=samba
sudo firewall-cmd --reload
```

### Step 10: Test the Samba configuration file

```bash
sudo testparm
```

> If there are no errors, the config is valid.

---

## Part 2: Ubuntu Client — Samba Access

### Step 1: Install Samba client tools

```bash
sudo apt install samba smbclient cifs-utils -y
```

### Step 2: List shares on the Rocky server

```bash
smbclient -L //192.168.200.5 -U sambauser
```

Enter the Samba password when prompted. You should see the `shared` share listed.

### Step 3: Connect to the Rocky server share

```bash
smbclient //192.168.200.5/shared -U sambauser
```

Once connected, you can use commands like:

```bash
ls                      # List files in the share
put localfile.txt       # Upload a file
get remotefile.txt      # Download a file
mkdir NewFolder         # Create a directory
exit                    # Disconnect
```

---

## Config File

| File | Description |
|---|---|
| [`smb.conf`](./smb.conf) | Full Samba server configuration |

---

## What I Learned

- Samba bridges Linux and Windows file sharing using the SMB/CIFS protocol
- SELinux must be configured alongside the firewall — SELinux has its own separate access controls that can block Samba even if the firewall allows it
- `testparm` is the fastest way to validate your `smb.conf` before restarting the service
- Samba users are separate from Linux system users — `smbpasswd` manages the Samba-specific password database
- `smbclient` works like an FTP client — you connect and then use interactive commands to transfer files
