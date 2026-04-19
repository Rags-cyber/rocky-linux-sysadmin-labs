# Email Server Configuration — Postfix & Dovecot on Rocky Linux

> Setup of a full email server using Postfix (SMTP) for sending mail and Dovecot (IMAP) for receiving, with testing via Telnet and Thunderbird on Ubuntu.

**Environment:** Rocky Linux (Server) + Ubuntu Client inside Oracle VirtualBox  
**Domain used:** `shadow.com`  
**Server IP:** `192.168.200.5`  
**Test users:** `user1`, `user2`

📝 **Full write-up with screenshots:** [Medium Article](https://medium.com/@gurungatwork98/setting-up-a-mail-server-on-linux-a-complete-guide-to-postfix-and-dovecot-eedc64b60bdf)
---

## Table of Contents

- [Part 1: Postfix (SMTP) Setup](#part-1-postfix-smtp-setup)
- [Part 2: Dovecot (IMAP) Setup](#part-2-dovecot-imap-setup)
- [Part 3: SSL Certificate](#part-3-ssl-certificate)
- [Part 4: Ubuntu — Thunderbird Testing](#part-4-ubuntu--thunderbird-testing)
- [Errors Faced](#errors-faced)
- [What I Learned](#what-i-learned)

---

## Part 1: Postfix (SMTP) Setup

### Step 1: Install Postfix

```bash
sudo dnf install postfix -y
```

### Step 2: Enable Postfix and check status

```bash
sudo systemctl enable postfix --now
sudo systemctl status postfix
```

### Step 3: Configure the main Postfix config file

```bash
sudo nano /etc/postfix/main.cf
```

Find and uncomment/edit the following lines:

```
myhostname = server.shadow.com
mydomain = shadow.com
myorigin = $mydomain
inet_interfaces = all
mynetworks = 192.168.200.0/24, 127.0.0.0/8
home_mailbox = Maildir/
```

**Explanation:**

| Directive | Meaning |
|---|---|
| `myhostname` | Full hostname of the mail server |
| `mydomain` | The mail domain |
| `myorigin` | Domain appended to outgoing mail addresses |
| `inet_interfaces = all` | Listen on all network interfaces |
| `mynetworks` | Trusted networks that can relay mail through this server |
| `home_mailbox = Maildir/` | Store mail in Maildir format in each user's home directory |

### Step 4: Restart Postfix and check status

```bash
sudo systemctl restart postfix
sudo systemctl status postfix
```

### Step 5: Install Telnet for testing

```bash
sudo dnf install telnet -y
```

### Step 6: Create test users

```bash
sudo useradd user1
sudo passwd user1

sudo useradd user2
sudo passwd user2
```

### Step 7: Add mail DNS record to the forward zone

Edit your DNS forward zone file at `/var/named/fwd.shadow.com.db` and add:

```
mail    IN  A       192.168.200.5
@       IN  MX  10  mail.shadow.com.
```

Then restart DNS:

```bash
sudo systemctl restart named
```

### Step 8: Test Postfix with Telnet

```bash
telnet localhost 25
```

Inside the Telnet session:

```
EHLO shadow.com
MAIL FROM:<user1@shadow.com>
RCPT TO:<user2@shadow.com>
DATA
Subject: Test Email
This is a test email from Postfix.
.
QUIT
```

### Step 9: Verify the email was received by user2

```bash
su - user2
ls ~/Maildir/new/
cat ~/Maildir/new/<filename>
```

Postfix is now successfully configured.

---

## Part 2: Dovecot (IMAP) Setup

### Step 10: Install Dovecot

```bash
sudo dnf install dovecot -y
```

### Step 11: Configure protocols in dovecot.conf

```bash
sudo nano /etc/dovecot/dovecot.conf
```

Go to line 24 and uncomment:

```
protocols = imap pop3 lmtp
```

> Use `nano -c` to show line numbers.

### Step 12: Configure mail location

```bash
sudo nano /etc/dovecot/conf.d/10-mail.conf
```

Go to line 24 and uncomment:

```
mail_location = maildir:~/Maildir
```

### Step 13: Disable plaintext authentication restriction

```bash
sudo nano /etc/dovecot/conf.d/10-auth.conf
```

Go to line 10 and change to:

```
disable_plaintext_auth = no
```

### Step 14: Add login mechanism

In the same file, go to line 100 and add `login` to the auth mechanisms:

```
auth_mechanisms = plain login
```

### Step 15: Configure Postfix socket in Dovecot

```bash
sudo nano /etc/dovecot/conf.d/10-master.conf
```

Go to line 110 and uncomment, then add Postfix user and group:

```
unix_listener /var/spool/postfix/private/auth {
    mode = 0666
    user = postfix
    group = postfix
}
```

### Step 16: Start Dovecot and check status

```bash
sudo systemctl enable dovecot --now
sudo systemctl start dovecot
sudo systemctl status dovecot
```

---

## Part 3: SSL Certificate

Generate a self-signed SSL certificate to secure communication between clients and the mail server:

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/pki/dovecot/private/dovecot.pem \
  -out /etc/pki/dovecot/certs/dovecot.pem
```

---

## Part 4: Ubuntu — Thunderbird Testing

### Step 1: Install Thunderbird

```bash
sudo apt install thunderbird -y
```

### Step 2: Add user accounts in Thunderbird

Open Thunderbird and add both `user1@shadow.com` and `user2@shadow.com` with their passwords.

### Step 3: Send a test email

Send an email from `user1` to `user2` and verify it is received in Thunderbird.

---

## Errors Faced

> **Mail domain missing from forward lookup zone**

The mail domain was not added to the DNS forward zone file initially, which made it impossible to send and receive mail. Adding the `MX` record and the `mail` A record to the DNS zone file and restarting `named` resolved the issue.

**Lesson:** Email servers depend on correct DNS MX records. Always add the mail record to your DNS zone before testing.

---

## What I Learned

- Postfix handles outgoing mail (SMTP) and Dovecot handles incoming mail retrieval (IMAP)
- The `Maildir/` format stores each email as a separate file — easier to manage than `mbox`
- DNS MX records are required for mail delivery — the email server will not work without them
- Telnet is a useful tool for testing SMTP manually before setting up a full mail client
- Thunderbird requires correct IMAP/SMTP settings and reachable DNS to connect to the mail server
