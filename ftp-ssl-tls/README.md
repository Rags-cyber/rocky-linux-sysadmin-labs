# FTP Server with SSL/TLS on Rocky Linux (vsftpd + OpenSSL)

> Securing an FTP server with SSL/TLS encryption using OpenSSL certificates and vsftpd, with FileZilla client testing on Ubuntu.

**Environment:** Rocky Linux (Server) + Ubuntu (Client) inside Oracle VirtualBox  
**Server IP:** `192.168.200.5`  
**Passive ports:** `40000–40001`  
**FTPS port:** `990`

📝 **Full write-up with screenshots:** [Medium Article](https://medium.com/@gurungatwork98)

---

## Table of Contents

- [Part 1: Rocky Linux — SSL Certificate + vsftpd Config](#part-1-rocky-linux--ssl-certificate--vsftpd-config)
- [Part 2: Ubuntu Client — FileZilla with SSL/TLS](#part-2-ubuntu-client--filezilla-with-ssltls)
- [Config File](#config-file)
- [What I Learned](#what-i-learned)

---

## Part 1: Rocky Linux — SSL Certificate + vsftpd Config

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

### Step 3: Install OpenSSL

```bash
sudo dnf install openssl -y
```

### Step 4: Create a directory to store the SSL files

```bash
sudo mkdir /etc/ssl/vsftpd
```

### Step 5: Generate a self-signed SSL certificate and private key

```bash
sudo openssl req -x509 -nodes \
  -keyout /etc/ssl/vsftpd/privatekey.key \
  -out /etc/ssl/vsftpd/selfsigned.crt \
  -days 365 \
  -newkey rsa:2048
```

**Flag explanation:**

| Flag | Meaning |
|---|---|
| `req -x509` | Generate a self-signed certificate (not a CSR) |
| `-nodes` | Do not encrypt the private key (no passphrase) |
| `-keyout` | Path to store the private key |
| `-out` | Path to store the certificate |
| `-days 365` | Certificate valid for 1 year |
| `-newkey rsa:2048` | Generate a new 2048-bit RSA key pair |

### Step 6: Fill in organizational details

You will be prompted for: Country, State, City, Organization, Common Name (use your server's IP or hostname), and Email. Fill these in as appropriate.

### Step 7: Open the vsftpd config file

```bash
sudo nano /etc/vsftpd/vsftpd.conf
```

### Step 8: Add SSL/TLS configuration at the end of the file

```
ssl_enable=YES
ssl_tlsv1_2=YES
ssl_sslv2=NO
ssl_sslv3=NO

rsa_cert_file=/etc/ssl/vsftpd/selfsigned.crt
rsa_private_key_file=/etc/ssl/vsftpd/privatekey.key
```

> TLS v1.2 is enabled; SSLv2 and SSLv3 are disabled as they are outdated and insecure.

### Step 9: Add additional SSL security settings

```
allow_anon_ssl=NO              # Disable anonymous SSL logins
force_local_data_ssl=YES       # Encrypt all data transfers
force_local_logins_ssl=YES     # Encrypt the login process
ssl_ciphers=HIGH               # Use only high-strength ciphers
require_ssl_reuse=NO           # Disable SSL session reuse
```

### Step 10: Add passive port configuration

```
pasv_min_port=40000    # Minimum passive mode port
pasv_max_port=40001    # Maximum passive mode port
debug_ssl=YES          # Log detailed SSL/TLS handshake info for debugging
```

### Step 11: Restart vsftpd

```bash
sudo systemctl restart vsftpd
sudo systemctl status vsftpd
```

### Step 12: Open firewall ports for FTPS and passive mode

```bash
sudo firewall-cmd --permanent --add-port=990/tcp
sudo firewall-cmd --permanent --add-port=40000-40001/tcp
sudo firewall-cmd --reload
```

---

## Part 2: Ubuntu Client — FileZilla with SSL/TLS

### Step 1: Update the system

```bash
sudo apt-get update && apt-get upgrade -y
```

### Step 2: Install FileZilla

```bash
sudo apt install filezilla -y
```

### Step 3: Open FileZilla and open Site Manager

Go to **File → Site Manager → New Site**

### Step 4: Configure the connection

| Field | Value |
|---|---|
| Protocol | FTP - File Transfer Protocol |
| Host | `192.168.200.5` |
| Encryption | **Require explicit FTP over TLS** |
| Logon Type | Normal |
| User | `server` |
| Password | Rocky server password |

Click **Connect**.

### Step 5: Accept the SSL certificate

FileZilla will display your certificate details. Check **"Always trust this certificate in future sessions"** and click **OK**.

### Step 6: Browse and transfer files

You will now see two panels:
- **Left:** Ubuntu local directories
- **Right:** Rocky server directories

### Step 7: Transfer a file

Right-click a file on either side and click **"Upload"** or **"Download"**, or drag and drop between panels. Then right-click the transfer in the queue at the bottom and click **"Process Queue"**.

### Step 8: Verify the transfer

Confirm the file appears in the destination directory on the other machine.

---

## Config File

| File | Description |
|---|---|
| [`vsftpd.conf`](./vsftpd.conf) | vsftpd configuration with SSL/TLS settings |

---

## What I Learned

- SSL/TLS encrypts both the login credentials and the data in transit — plain FTP sends both in cleartext
- OpenSSL can generate self-signed certificates for lab use, but production environments should use a Certificate Authority (CA)
- TLS v1.2 is the minimum acceptable version — SSLv2, SSLv3, and TLS v1.0/1.1 are all deprecated
- Passive mode ports must be opened on the firewall separately from the main FTPS port (990)
- FileZilla shows the certificate details before connecting — in a real environment, verify the fingerprint matches
