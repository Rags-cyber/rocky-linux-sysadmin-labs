# Apache Web Server Configuration on Rocky Linux

> Setting up an Apache (httpd) web server on Rocky Linux with SSL/TLS, a custom virtual host, and DNS integration — accessed from Ubuntu via browser.

**Environment:** Rocky Linux (Server) + Ubuntu (Client) inside Oracle VirtualBox  
**Domain:** `shadow.com`  
**Server IP:** `192.168.200.5`

📝 **Full write-up with screenshots:** [Medium Article](https://medium.com/@gurungatwork98/deploying-a-secure-apache-web-server-on-rocky-linux-9ecb5c885598)

---

## Table of Contents

- [Part 1: Apache Installation and Basic Setup](#part-1-apache-installation-and-basic-setup)
- [Part 2: SSL Certificate Generation](#part-2-ssl-certificate-generation)
- [Part 3: Virtual Host with HTTPS](#part-3-virtual-host-with-https)
- [Part 4: DNS Integration](#part-4-dns-integration)
- [Config Files](#config-files)
- [What I Learned](#what-i-learned)

---

## Part 1: Apache Installation and Basic Setup

### Step 1: Install Apache (httpd)

```bash
sudo dnf install httpd -y
```

### Step 2: Start and enable httpd

```bash
sudo systemctl enable httpd --now
sudo systemctl start httpd
sudo systemctl status httpd
```

### Step 3: Configure the firewall

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

### Step 4: Install mod_ssl for HTTPS support

```bash
sudo dnf install mod_ssl -y
```

### Step 5: Restart httpd after installing mod_ssl

```bash
sudo systemctl restart httpd
```

### Step 6: Test the default page from Ubuntu

On Ubuntu, open a browser and go to:

```
http://192.168.200.5
```

You should see the default Apache test page — this confirms Apache is running and reachable.

---

## Part 2: SSL Certificate Generation

### Step 7: Generate a self-signed SSL key and certificate

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/pki/tls/private/shadow.com.key \
  -out /etc/pki/tls/certs/shadow.com.crt
```

Fill in the details when prompted. For **Common Name**, use `www.shadow.com`.

---

## Part 3: Virtual Host with HTTPS

### Step 8: Create the web root directory and index page

```bash
sudo mkdir -p /var/www/shadow.com/html
sudo nano /var/www/shadow.com/html/index.html
```

Add some basic HTML:

```html
<!DOCTYPE html>
<html>
<head>
  <title>shadow.com</title>
</head>
<body>
  <h1>Welcome to shadow.com</h1>
  <p>Apache web server is running on Rocky Linux.</p>
</body>
</html>
```

### Step 9: Set file permissions for Apache

```bash
sudo chown -R apache:apache /var/www/shadow.com
sudo chmod -R 755 /var/www/shadow.com
```

### Step 10: Update the virtual host config to add HTTPS

```bash
sudo nano /etc/httpd/conf.d/shadow.com.conf
```

Add the following virtual host configuration:

```apache
<VirtualHost *:80>
    ServerName shadow.com
    ServerAlias www.shadow.com
    DocumentRoot /var/www/shadow.com/html
    Redirect permanent / https://www.shadow.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName www.shadow.com
    DocumentRoot /var/www/shadow.com/html

    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/shadow.com.crt
    SSLCertificateKeyFile /etc/pki/tls/private/shadow.com.key

    <Directory /var/www/shadow.com/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

### Step 11: Check the Apache config syntax

```bash
sudo apachectl configtest
```

Expected output: `Syntax OK`

### Step 12: Restart Apache

```bash
sudo systemctl restart httpd
```

---

## Part 4: DNS Integration

### Step 13: Add www record to the DNS forward zone

Edit your forward zone file `/var/named/fwd.shadow.com.db` and add:

```
www     IN  A   192.168.200.5
```

Then restart DNS:

```bash
sudo systemctl restart named
```

### Step 14: Test from Ubuntu browser

On Ubuntu, open a browser and go to:

```
https://www.shadow.com
```

You should see your custom web page. Accept the self-signed certificate warning (expected in lab environments).

---

## Config Files

| File | Description |
|---|---|
| [`shadow.com.conf`](./shadow.com.conf) | Apache virtual host configuration with HTTP and HTTPS |

---

## What I Learned

- Apache uses virtual host config files in `/etc/httpd/conf.d/` to serve different sites
- `mod_ssl` must be installed separately before Apache can handle HTTPS connections
- Self-signed certificates work for labs but browsers will show a warning — production sites need a CA-signed certificate (e.g., Let's Encrypt)
- `apachectl configtest` is essential before restarting Apache — it catches syntax errors that would take the server down
- HTTP to HTTPS redirect using `Redirect permanent` ensures all traffic is encrypted
- DNS must be updated with a `www` A record before the domain resolves correctly in the browser
