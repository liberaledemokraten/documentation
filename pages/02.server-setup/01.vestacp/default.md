---
title: 'Vesta CP'
taxonomy:
    category:
        - vesta
    tag:
        - vesta
        - configuration
---

VestaCP is a server management platform or a control panel easing up the maintainment of our server in many regards as well as allowing for us to assign different users for each branch of the party for their own domain and space.

## Requirements

* A minimal OS, preferably Debian
* `wget` and `curl` should be installed
* installing `vim` is recommended
* Following softwares shouldn't be installed:
 * nginx
 * Apache httpd
 * MySQL / MariaDB
 * PostgreSQL
 * Dovecot
 * Exim
 * ClamAV

NOTE: If using Debian 10, go for myVestaCP instead. The configuration is pretty much the same.

## Optional Preconfiguration

You might also want to add the official nginx and sury's PHP repositories to have up to date software. To do so, go through the following steps:

```
apt-get install gnupg2 ca-certificates lsb-release apt-transport-https vim

curl -fsSL https://packages.sury.org/php/apt.gpg | sudo gpg --dearmor -o /usr/share/keyrings/sury_php.gpg
curl -fsSL https://nginx.org/keys/nginx_signing.key | sudo gpg --dearmor -o /usr/share/keyrings/nginx.gpg

echo "deb [arch=amd64,arm64 signed-by=/usr/share/keyrings/sury_php.gpg] https://packages.sury.org/php `lsb_release -cs` main" | tee /etc/apt/sources.list.d/sury_php.list
echo "deb [arch=amd64,arm64 signed-by=/usr/share/keyrings/nginx.gpg] http://nginx.org/packages/debian `lsb_release -cs` nginx" | tee /etc/apt/sources.list.d/nginx.list

apt-get update
apt-get upgrade
```


## Installation

First download the installer and run it

```
wget https://vestacp.com/pub/vst-install.sh
./vst-install.sh --nginx yes --apache yes --phpfpm no --named yes --remi yes --vsftpd yes --proftpd no --iptables yes --fail2ban yes --quota yes --exim yes --dovecot yes --spamassassin yes --clamav yes --softaculous yes --mysql yes --postgresql yes --hostname example.com --email mail@example.com --password deed
```

If using myVestaCP, go for the following instead:

```
wget https://c.myvestacp.com/vst-install-debian.sh
./vst-install-debian.sh --nginx yes --apache yes --phpfpm no --named yes --remi yes --vsftpd yes --proftpd no --iptables yes --fail2ban yes --quota yes --exim yes --dovecot yes --spamassassin yes --clamav yes --softaculous yes --mysql yes --postgresql yes --hostname example.com --email mail@example.com --password deed
```

If using HestiaCP - a fork of VestaCP:

```
wget https://raw.githubusercontent.com/hestiacp/hestiacp/release/install/hst-install.sh
bash hst-install.sh --port 8083 --hostname example.com --email mail@example.com --password deed --multiphp yes --postgresql yes --sieve yes --quota yes
```

Now wait until the installation is finished.

!!! The defined hostname above is the main domain of the server.

## Templates
VestaCP supports the use of templates. That way, you can automatically configure some things instead of doing everything manually. See the [Templates](../templates) article for further information.