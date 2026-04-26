# Rocky Linux Sysadmin Labs — Bash Scripting

A collection of Bash scripts for common Linux sysadmin tasks built as part of the Rocky Linux Sysadmin Lab Series.

📄 Full article: [Linux Bash Scripting: A Complete Practical Guide](https://medium.com/@gurungatwork98/linux-bash-scripting-a-complete-practical-guide-ffe8f9e120b6)

---

## Scripts

### 1. `backup.sh` — Automated Backup with Rotation
Compresses a source directory into a timestamped `.tar.gz` archive, logs the result, and automatically deletes backups older than 7 days.

**Usage:**
```bash
chmod +x backup.sh
sudo ./backup.sh
```

**What it does:**
- Backs up `/etc` by default (change `SOURCE_DIR` to any directory)
- Saves archives to `~/backups/` with timestamp in the filename
- Logs all activity to `~/backup.log`
- Auto-deletes backups older than 7 days (`KEEP_DAYS=7`)

---

### 2. `service_report.sh` — Multi-Service Status Report
Checks a list of services and prints a formatted status report — both to the terminal and to a log file simultaneously.

**Usage:**
```bash
chmod +x service_report.sh
sudo ./service_report.sh
```

**What it does:**
- Checks sshd, httpd, named, dhcpd, vsftpd, postfix, dovecot
- Prints `[OK]` or `[FAIL]` for each service
- Counts and reports total failed services
- Logs everything to `~/service_report.log`

---

### 3. `user_provisioning.sh` — Bulk User Provisioning
Reads a list of usernames from `users.txt`, creates each account if it does not already exist, assigns it to a group, sets a default password, and forces a password change on first login.

**Usage:**
```bash
# Create users.txt with one username per line
nano users.txt

chmod +x user_provisioning.sh
sudo ./user_provisioning.sh
```

**What it does:**
- Reads usernames from `users.txt` (one per line)
- Creates a group (`staff` by default) if it does not exist
- Skips users that already exist
- Sets default password `ChangeMe@123` via `chpasswd`
- Forces password change on first login with `passwd --expire`
- Logs all activity to `~/user_provision.log`

> **Note:** Make sure `users.txt` uses Unix line endings (`LF`), not Windows (`CRLF`). If you created the file on Windows, run `sed -i 's/\r//' users.txt` before running the script.

---

### 4. `disk_alert.sh` — Disk Space Alert
Scans all mounted filesystems, compares usage against a threshold, and logs a warning for any partition approaching capacity.

**Usage:**
```bash
chmod +x disk_alert.sh
./disk_alert.sh
```

**What it does:**
- Scans all real mounted filesystems (excludes tmpfs, cdrom)
- Logs `WARNING` if usage is at or above 80% (change `THRESHOLD` as needed)
- Logs `OK` for partitions below the threshold
- Appends all results to `~/disk_alert.log`

---

## Automating with Cron

Schedule all scripts to run automatically:

```bash
crontab -e
```

Add these lines:

```
# Backup every night at 2 AM
0 2 * * * /home/server/scripts/backup.sh

# Disk space check every hour
0 * * * * /home/server/scripts/disk_alert.sh

# Service report every 30 minutes
*/30 * * * * /home/server/scripts/service_report.sh >> /home/server/logs/service.log 2>&1
```

> Always use full paths in cron. Always redirect output to a log file using `>>` and `2>&1`.

---

## Requirements

- Rocky Linux 8/9 (or any RHEL-based distro)
- Bash 4+
- `systemctl` for service management scripts
- Run service and user scripts with `sudo`

---

## Author

**Bishal Gurung**
- Medium: [medium.com/@gurungatwork98](https://medium.com/@gurungatwork98)
- Linked: [https://www.linkedin.com/in/bishal-gurung-491171292/]
