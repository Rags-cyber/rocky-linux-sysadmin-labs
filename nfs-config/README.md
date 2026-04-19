# Network File Sharing — NFS Configuration on Rocky Linux

> Setup of an NFS server on Rocky Linux to share a directory with an Ubuntu client, including permanent mount configuration via /etc/fstab.

**Environment:** Rocky Linux (Server) + Ubuntu (Client) inside Oracle VirtualBox  
**Server IP:** `192.168.200.5`  
**Ubuntu IP:** `192.168.200.100` (assigned by DHCP)  
**Shared directory:** `/var/nfs/general`  
**Mount point on Ubuntu:** `/nfs/Shared_dir`

📝 **Full write-up with screenshots:** [Medium Article](https://medium.com/@gurungatwork98/how-to-set-up-nfs-network-file-sharing-on-rocky-linux)

---

## Table of Contents

- [Part 1: Rocky Linux — NFS Server Setup](#part-1-rocky-linux--nfs-server-setup)
- [Part 2: Ubuntu Client — NFS Mount](#part-2-ubuntu-client--nfs-mount)
- [Part 3: Permanent Mount via fstab](#part-3-permanent-mount-via-fstab)
- [Config File](#config-file)
- [What I Learned](#what-i-learned)

---

## Part 1: Rocky Linux — NFS Server Setup

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

### Step 3: Install NFS server

```bash
sudo dnf install nfs-utils -y
```

### Step 4: Enable NFS server permanently

```bash
sudo systemctl enable nfs-server --now
```

### Step 5: Start NFS and check status

```bash
sudo systemctl start nfs-server
sudo systemctl status nfs-server
```

### Step 6: Check NFS version support

```bash
sudo cat /proc/fs/nfsd/versions
```

> NFSv3 and NFSv4 are enabled by default. NFSv2 is outdated and disabled.

### Step 7: Create the shared directory

```bash
sudo mkdir /var/nfs/general -p
```

> `-p` creates parent directories automatically if they do not exist.

### Step 8: Set ownership to nobody:nobody

```bash
sudo chown nobody:nobody /var/nfs/general
```

> `nobody` is a special unprivileged user used for processes that don't require specific user association — it limits the impact of any security issues.

### Step 9: Edit the NFS exports file

```bash
sudo nano /etc/exports
```

### Step 10: Add the export entry

```
/var/nfs/general    192.168.200.100(rw,sync,no_subtree_check)
```

**Explanation:**

| Option | Meaning |
|---|---|
| `/var/nfs/general` | The directory being shared |
| `192.168.200.100` | Only this IP (Ubuntu) can access the share |
| `rw` | Read and write access |
| `sync` | Writes are committed to disk before the server responds |
| `no_subtree_check` | Disables subtree checking — improves reliability |

### Step 11: Apply the export configuration

```bash
sudo exportfs -arv
```

### Step 12: Enable rpcbind and restart NFS

```bash
sudo systemctl enable --now rpcbind nfs-server
sudo systemctl restart nfs-server
sudo systemctl status nfs-server
```

### Step 13: Allow NFS through the firewall

```bash
sudo firewall-cmd --add-service={nfs,nfs3,rpc-bind,mountd} --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --zone=public --list-all
```

### Step 14: Reload and restart NFS

```bash
sudo systemctl daemon-reload
sudo systemctl restart nfs-server
sudo systemctl status nfs-server
```

### Create test files in the shared directory

```bash
sudo touch /var/nfs/general/nfstesting{1..4}.txt
ls /var/nfs/general
```

---

## Part 2: Ubuntu Client — NFS Mount

### Step 1: Update the system

```bash
sudo apt-get update && apt-get upgrade -y
```

### Step 2: Install NFS client packages

```bash
sudo apt install nfs-kernel-server -y
sudo apt install nfs-common -y
```

### Step 3: Create the local mount point

```bash
sudo mkdir /nfs/Shared_dir -p
```

### Step 4: Mount the remote NFS share

```bash
sudo mount 192.168.200.5:/var/nfs/general /nfs/Shared_dir
```

### Step 5: Verify the files from Rocky server are visible

```bash
cd /nfs/Shared_dir
ls -l
```

You should see the `nfstesting1.txt` through `nfstesting4.txt` files created on the server.

### Step 6: Verify the mount with df

```bash
df -h
```

The NFS mount should appear as `192.168.200.5:/var/nfs/general` in the output.

---

### NFS Testing

**Create a file from Ubuntu and verify on Rocky:**

```bash
# On Ubuntu
sudo touch /nfs/Shared_dir/NFS_Success.txt

# On Rocky
ls /var/nfs/general
# NFS_Success.txt should appear here
```

**Create a directory from Rocky and verify on Ubuntu:**

```bash
# On Rocky
sudo mkdir /var/nfs/general/Success

# On Ubuntu
ls /nfs/Shared_dir
# Success/ directory should appear here
```

---

## Part 3: Permanent Mount via fstab

The manual mount only lasts until reboot. To make it permanent:

### Step 1: Edit /etc/fstab

```bash
sudo nano /etc/fstab
```

### Step 2: Add the NFS mount entry

```
192.168.200.5:/var/nfs/general   /nfs/Shared_dir   nfs   auto,nofail,noatime,nolock,intr,tcp,actimeo=1800   0 0
```

**Option explanation:**

| Option | Meaning |
|---|---|
| `nfs` | Filesystem type |
| `auto` | Mount automatically at boot |
| `nofail` | Do not fail the boot if the mount fails |
| `noatime` | Do not update file access time (better performance) |
| `nolock` | Disable NFS file locking |
| `intr` | Allow the mount to be interrupted if the server goes down |
| `tcp` | Use TCP for the connection |
| `actimeo=1800` | Cache file attributes for 30 minutes |
| `0 0` | Skip dump and fsck |

### Step 3: Verify fstab syntax

```bash
sudo findmnt --verify
```

### Step 4: Reboot and verify

```bash
init 6
```

After reboot:

```bash
sudo ls -l /nfs/Shared_dir
```

All files and directories should still be present — confirming the permanent mount is working.

---

## Config File

| File | Description |
|---|---|
| [`exports`](./exports) | NFS export configuration |
| [`fstab`](./fstab) | fstab entry for permanent NFS mount |

---

## What I Learned

- NFS allows seamless file sharing between Linux machines as if they were local directories
- `nobody:nobody` ownership on the shared directory limits potential damage if a client is compromised
- The `/etc/exports` file controls exactly which clients can access which directories and with what permissions
- `exportfs -arv` re-reads the exports file without needing to restart the NFS service
- Without an fstab entry, the NFS mount is lost on reboot — always make mounts permanent for production use
