# FTP Server Configuration on Rocky Linux (vsftpd)

> Full setup of an FTP server using vsftpd on Rocky Linux, with client testing from Ubuntu using lftp — including file upload, download, directory management, and deletion.

**Environment:** Rocky Linux (Server) + Ubuntu (Client) inside Oracle VirtualBox  
**Server IP:** `192.168.200.5`  
**Domain:** `server.shadow.com`

📝 **Full write-up with screenshots:** [Medium Article](https://medium.com/@gurungatwork98/rocky-linux-ftp-server-ftp-server-configuration-417cc68c1992)

---

## Table of Contents

- [Part 1: Rocky Linux — vsftpd Server Setup](#part-1-rocky-linux--vsftpd-server-setup)
- [Part 2: Ubuntu Client — FTP Access](#part-2-ubuntu-client--ftp-access)
- [Part 3: FTP Testing](#part-3-ftp-testing)
- [Config File](#config-file)
- [What I Learned](#what-i-learned)

---

## Part 1: Rocky Linux — vsftpd Server Setup

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

### Step 3: Install vsftpd

```bash
sudo dnf -y install vsftpd
```

### Step 4: Enable and start vsftpd

```bash
sudo systemctl enable vsftpd
sudo systemctl start vsftpd
```

### Step 5: Open the vsftpd config file

```bash
sudo nano /etc/vsftpd/vsftpd.conf
```

### Step 6: Core security settings

```
anonymous_enable=NO      # Disables anonymous access — login requires valid credentials
local_enable=YES         # Allows local system users to log in
write_enable=YES         # Allows file uploads to the server
```

### Step 7: File permissions and directory messages

```
local_umask=022          # New files: owner gets full (6), group/others get read only (4)
dirmessage_enable=YES    # Shows a message when a user enters a directory
```

### Step 8: Logging and port settings

```
xferlog_enable=YES           # Enables transfer logging
connect_from_port_20=YES     # Uses standard FTP data port 20
xferlog_std_format=YES       # Logs in standard format
```

### Step 9: ASCII mode

```
ascii_upload_enable=YES      # Enables ASCII mode for uploads
ascii_download_enable=YES    # Enables ASCII mode for downloads
```

### Step 10: Chroot (user jail) settings

```
chroot_local_user=YES                        # Restricts users to their home directory
chroot_list_enable=YES                       # Enables a list of users exempt from chroot
allow_writeable_chroot=YES                   # Allows writing inside the chroot directory
chroot_list_file=/etc/vsftpd/chroot_list     # Path to the chroot exemption list
```

### Step 11: IPv6 and auth settings

```
listen=NO                    # Disables IPv4 standalone listening (using xinetd or IPv6)
listen_ipv6=YES              # Enables listening on IPv6
pam_service_name=vsftpd      # Uses PAM for authentication
userlist_enable=YES          # Enables user access control list
```

### Step 12: Recursive directory listing

```
ls_recurse_enable=YES    # Allows recursive ls -R listings
```

### Step 13: Additional settings at the end of the file

```
local_root=public_html       # Sets /public_html as the default root for FTP sessions
use_localtime=YES            # Uses server's local time for file timestamps
seccomp_sandbox=NO           # Disables seccomp security sandbox (required for some setups)
```

Save and exit the config file.

---

### Chroot List Configuration

Edit the chroot list file:

```bash
sudo nano /etc/vsftpd/chroot_list
```

Add the username of any user who should be **exempt** from the chroot restriction (one per line):

```
server
```

---

### SELinux Configuration

#### Step 1: Check SELinux status

```bash
sudo sestatus
```

#### Step 2: Allow FTP full access via SELinux

```bash
sudo setsebool -P ftpd_full_access on
```

> This allows the FTP server to read and write anywhere on the system without SELinux blocking it.

#### Step 3: Start vsftpd and verify

```bash
sudo systemctl enable --now vsftpd
sudo systemctl start vsftpd
sudo systemctl status vsftpd
```

#### Step 4: Allow FTP through the firewall

```bash
sudo firewall-cmd --add-service=ftp
sudo firewall-cmd --runtime-to-permanent
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

---

## Part 2: Ubuntu Client — FTP Access

### Step 1: Update the system

```bash
sudo apt-get update
sudo apt-get upgrade
```

### Step 2: Install lftp

```bash
sudo apt install lftp -y
```

### Step 3: Connect to Rocky FTP server by IP

```bash
sudo lftp -u server 192.168.200.5
```

### Step 4: Connect by domain name

```bash
sudo lftp -u server server.shadow.com
```

---

## Part 3: FTP Testing

### Uploading files to the server

Create test files on Ubuntu:

```bash
mkdir Test_1
echo "This is FTP server testing" > ftp.txt
cat > one.txt << EOF
This is file one
EOF
touch two.txt
```

Connect and upload:

```bash
sudo lftp -u server server.shadow.com
!pwd                    # Check current directory on Ubuntu
sudo chmod 777 Test_1   # Ensure the local directory is writable
cd Test_1               # Go to server directory
mput ftp.txt one.txt    # Upload multiple files
put two.txt             # Upload single file
```

Verify on Rocky server:

```bash
cd Test_1
ls -l
```

---

### Directory management from the client

```bash
ls -l                   # List server directory contents
mkdir New_Dir           # Create a new directory on the server
ls -l                   # Verify it was created
rmdir New_Dir           # Remove the directory
```

---

### Downloading files from the server

On Rocky server, create files to download:

```bash
touch 1.txt 2.txt 3.txt
```

On Ubuntu, inside the FTP session:

```bash
set xfer:clobber on                     # Allow overwrite of existing files
ls -l                                   # See the new files
lcd ~/FTP_Test                          # Change local Ubuntu directory
get 1.txt                               # Download single file
mget 2.txt 3.txt                        # Download multiple files
!ls -l                                  # Verify download on Ubuntu side
```

---

### Deleting files on the server from the client

```bash
rm 1.txt                # Delete single file on server
mrm 2.txt 3.txt         # Delete multiple files on server
ls -l                   # Verify deletion
```

---

## Config File

| File | Description |
|---|---|
| [`vsftpd.conf`](./vsftpd.conf) | Full vsftpd server configuration |
| [`chroot_list`](./chroot_list) | Users exempt from chroot restriction |

---

## What I Learned

- `vsftpd` is a lightweight, secure FTP server ideal for Linux environments
- Chroot jailing restricts users to their home directory, improving security
- SELinux must be configured alongside the firewall — both need to allow FTP independently
- `lftp` is more powerful than basic `ftp` — supports `mput`, `mget`, and local command execution with `!`
- The `set xfer:clobber on` setting is needed to avoid download errors when a file already exists locally
