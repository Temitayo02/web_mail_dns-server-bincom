# Web Server, Mail Server and DNS Server Setup

## 1. Introduction

This project involved setting up three network services on an Ubuntu Linux environment running under WSL:

1. Web Server — Apache2
2. Mail Server — Postfix
3. DNS Server — BIND9

The objective was to install, configure, and test each service to confirm that it was functioning correctly.

---

# 2. Web Server — Apache2

## Installation

Apache2 was installed using:

```bash
sudo apt-get install apache2
```

During the configuration, Apache initially attempted to use port 80. However, Nginx was already using port 80 on the system.

To avoid disrupting the existing Nginx service, Apache was configured to listen on port **8080** instead.

The Apache configuration was changed from:

```text
Listen 80
```

to:

```text
Listen 8080
```

The virtual host configuration was also changed from port 80 to port 8080.

The configuration was tested using:

```bash
sudo apache2ctl configtest
```

Result:

```text
Syntax OK
```

## Website Configuration

The default Apache website file was located at:

```text
/var/www/html/index.html
```

The page was edited to display:

```text
Hello, World!
```

The web server was then accessed through:

```text
http://localhost:8080
```

## Result

The Apache web server successfully displayed the custom webpage.

**Web Server Status: Successful**

**Software:** Apache2
**Port:** 8080
**Document Root:** `/var/www/html/`

---

# 3. Mail Server — Postfix

## Installation

Postfix and the mail utilities were installed using:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install postfix mailutils -y
```

During installation, **Internet Site** was selected as the Postfix configuration type.

The system mail name was configured as:

```text
mailserver.local
```

## Mailbox Configuration

Postfix was configured to use Maildir format:

```bash
sudo postconf -e 'home_mailbox = Maildir/'
```

Postfix was then restarted:

```bash
sudo systemctl restart postfix
```

## Test User

A test Linux user named `alice` was created:

```bash
sudo adduser alice
```

The Maildir directories were created using:

```bash
sudo -u alice mkdir -p /home/alice/Maildir/{cur,new,tmp}
```

## Test Email

A test email was sent to the `alice` mailbox:

```bash
echo "Hello Alice, this is a test email from the Postfix mail server." | mail -s "Postfix Test" alice
```

The email was successfully delivered to:

```text
/home/alice/Maildir/new/
```

The message was verified by viewing the contents of the mailbox.

## SMTP Verification

Postfix was confirmed to be listening on SMTP port 25 using:

```bash
sudo ss -tulpn | grep :25
```

The result showed Postfix's `master` process listening on:

```text
0.0.0.0:25
[::]:25
```

## Result

The Postfix mail server successfully accepted and delivered a test email to the local mailbox.

**Mail Server Status: Successful**

**Software:** Postfix
**Protocol:** SMTP
**Port:** 25
**Test User:** alice
**Mailbox:** `/home/alice/Maildir/`

---

# 4. DNS Server — BIND9

## Installation

BIND9 and its utilities were installed using:

```bash
sudo apt update
sudo apt install bind9 bind9utils bind9-doc dnsutils -y
```

The BIND9 service was started and confirmed to be running.

## DNS Zone Configuration

A local DNS zone named:

```text
myserver.local
```

was created in:

```text
/etc/bind/named.conf.local
```

The zone was configured as a master zone with the following configuration:

```text
zone "myserver.local" {
    type master;
    file "/etc/bind/db.myserver.local";
};
```

A zone database file was created at:

```text
/etc/bind/db.myserver.local
```

The DNS records were configured so that:

```text
myserver.local → 127.0.0.1
www.myserver.local → 127.0.0.1
ns1.myserver.local → 127.0.0.1
```

## Configuration Testing

The main BIND configuration was tested using:

```bash
sudo named-checkconf
```

No errors were returned.

The DNS zone was then checked using:

```bash
sudo named-checkzone myserver.local /etc/bind/db.myserver.local
```

The result confirmed that the zone was loaded successfully and returned:

```text
OK
```

## DNS Service Verification

BIND9 was restarted and its status was checked:

```bash
sudo systemctl restart bind9
sudo systemctl status bind9
```

The service was confirmed to be:

```text
Active: active (running)
```

## DNS Resolution Test

The DNS server was tested directly using:

```bash
dig @127.0.0.1 myserver.local
```

The query returned:

```text
;; ANSWER SECTION:
myserver.local.    604800    IN    A    127.0.0.1
```

The DNS server used for the query was:

```text
127.0.0.1#53
```

This confirmed that BIND9 was successfully resolving the configured domain to the correct IP address.

## Result

The BIND9 DNS server successfully resolved `myserver.local` to `127.0.0.1`.

**DNS Server Status: Successful**

**Software:** BIND9
**Protocol:** DNS
**Port:** 53
**DNS Zone:** `myserver.local`
**DNS Server:** `127.0.0.1`

---

# 5. Final Verification

| Service     | Software | Port | Test Result |
| ----------- | -------- | ---: | ----------- |
| Web Server  | Apache2  | 8080 | Successful  |
| Mail Server | Postfix  |   25 | Successful  |
| DNS Server  | BIND9    |   53 | Successful  |

All three required services were successfully installed, configured, and tested on the Ubuntu Linux environment.

# 6. Conclusion

The project successfully demonstrated the installation and basic configuration of a web server, mail server, and DNS server.

Apache2 was configured to host a webpage on port 8080, Postfix was configured to provide local SMTP mail delivery on port 25, and BIND9 was configured as a local DNS server capable of resolving `myserver.local` to `127.0.0.1`.

The successful tests confirmed that all three services were operational.
