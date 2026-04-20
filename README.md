# Rocky Linux Sysadmin Lab 🐧

Hands-on Linux system administration projects built on **Rocky Linux** using **Oracle VirtualBox**.  
Each project includes step-by-step configuration, config files, and a detailed Medium write-up with screenshots.

**Author:** Bishal Gurung  
**Medium:** [@gurungatwork98](https://medium.com/@gurungatwork98)  
**GitHub:** [Rags-cyber](https://github.com/Rags-cyber/rocky-linux-sysadmin-labs)  
**Environment:** Rocky Linux (Server) + Ubuntu (Client) on Oracle VirtualBox

---

## Projects

| # | Project | Skills Covered | Article |
|---|---|---|---|
| 1 | [Static IP Configuration](./static-ip/) | NetworkManager, nmcli, NIC config | [Medium](https://medium.com/@gurungatwork98) |
| 2 | [DNS — BIND9](./dns-bind9/) | DNS zones, A records, PTR records, forward/reverse lookup | [Medium](https://medium.com/@gurungatwork98) |
| 3 | [DHCP Server](./dhcp/) | ISC DHCP, lease management, VirtualBox NAT network | [Medium](https://medium.com/@gurungatwork98) |
| 4 | [NFS — Network File Sharing](./nfs/) | NFS exports, mounting, fstab, real-time file sharing | [Medium](https://medium.com/@gurungatwork98) |
| 5 | [FTP Server (vsftpd)](./ftp-server/) | vsftpd, chroot, SELinux, lftp client | [Medium](https://medium.com/@gurungatwork98) |
| 6 | [FTP with SSL/TLS](./ftp-ssl-tls/) | OpenSSL, FTPS, passive mode, FileZilla | [Medium](https://medium.com/@gurungatwork98) |
| 7 | [SSH — Secure Shell](./ssh/) | sshd_config, root login hardening, firewall, fingerprint | [Medium](https://medium.com/@gurungatwork98) |
| 8 | [SFTP — SSH File Transfer](./sftp/) | OpenSSH, chroot jail, internal-sftp, sftpusers group | [Medium](https://medium.com/@gurungatwork98) |
| 9 | [Apache Web Server](./apache-webserver/) | httpd, mod_ssl, virtual hosts, HTTPS, DNS integration | [Medium](https://medium.com/@gurungatwork98) |
| 10 | [Email Server — Postfix & Dovecot](./email-server/) | SMTP, IMAP, Maildir, MX records, Telnet, Thunderbird | [Medium](https://medium.com/@gurungatwork98) |
| 11 | [ACL — Access Control Lists](./acl/) | setfacl, getfacl, ACL mask, file and directory permissions | [Medium](https://medium.com/@gurungatwork98) |
| 12 | [Samba File Sharing](./samba/) | SMB/CIFS, smb.conf, SELinux, smbclient | [Medium](https://medium.com/@gurungatwork98) |
| 13 | [VM Backup — VirtualBox Clone](./backup-clone/) | Full clone, rollback strategy, snapshot best practices | [Medium](https://medium.com/@gurungatwork98) |

---

## Lab Environment
rocky-linux-sysadmin-labs/
│
├── README.md
│
├── static-ip/
│   ├── README.md
│   └── enp0s3.nmconnection
│
├── dns-bind9/
│   ├── README.md
│   ├── named.conf
│   ├── fwd.shadow.com.db
│   └── rvs.200.168.192.db
│
├── dhcp/
│   ├── README.md
│   └── dhcpd.conf
│
├── nfs/
│   ├── README.md
│   ├── exports
│   └── fstab
│
├── ftp-server/
│   ├── README.md
│   ├── vsftpd.conf
│   └── chroot_list
│
├── ftp-ssl-tls/
│   ├── README.md
│   └── vsftpd.conf
│
├── ssh/
│   ├── README.md
│   └── sshd_config
│
├── sftp/
│   ├── README.md
│   └── sshd_config
│
├── apache-webserver/
│   ├── README.md
│   └── shadow.com.conf
│
├── email-server/
│   ├── README.md
│   ├── main.cf
│   └── dovecot.conf
│
├── acl/
│   └── README.md
│
├── samba/
│   ├── README.md
│   └── smb.conf
│
└── backup-clone/
└── README.md
## How to Use This Repo

Each project folder contains:
- `README.md` — full step-by-step setup guide with command explanations and config breakdowns
- Config files ready to use as a reference for your own lab

Clone the repo:

```bash
git clone https://github.com/Rags-cyber/rocky-linux-sysadmin-labs.git
cd rocky-linux-sysadmin-labs
```

Browse any project folder and follow the README to reproduce the setup in your own lab environment.

---

## About

I am a system administration student building hands-on skills through practical projects,
documenting every step along the way. Each project is deployed on a real virtual lab,
tested end-to-end, and written up in detail on Medium with full screenshots.

The knowledge and foundation for these projects came from my college lectures and lecturer,
whose guidance made all of this possible.

Currently working towards: **RHCSA Certification**

---

## Connect

- 📝 **Medium:** [medium.com/@gurungatwork98](https://medium.com/@gurungatwork98)
- 💻 **GitHub:** [github.com/Rags-cyber/rocky-linux-sysadmin-labs](https://github.com/Rags-cyber/rocky-linux-sysadmin-labs)

---

*Built with 💻 on Rocky Linux | Bishal Gurung*

