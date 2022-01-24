---
title: 'SSL/TLS Configuration'
---

## Get the Certificate
We are using Dovecot for IMAP/POP3 (receiving mails) and exim4 for SMTP (sending mails). Both server applications should support TLS for client to server and server to server transfer to occur encrypted and secure. Thus, we first need an SSL/TLS certificate for our mail server domain `mail.liberale-demokraten.de`. Please follow the following steps to achieve that:

1. Login as the admin user to the control panel
2. Point the A and AAAA records for the subdomain to the respective IP address of the server
3. Add the subdomain as a web domain (just Web, no E-Mail and DNS)
4. Choose "redirect-LD-DE" as proxy template (it will redirect all calls to liberale-demokraten.de)
5. Enable Let's Encrypt certificates for the subdomain


## Set the proper Permissions
Now, we have obtained a valid SSL/TLS certificate from Let's Encrypt and vesta will automatically update the certificate before expiry. However, we are not done yet. We need to copy the certificates to another location and set the ownership and permissions accordingly. To do so, go through the following steps:

1. Log in as root via SSH
2. `cd /usr/local/vesta/bin`
3. `vim v-update-host-certificate`
4. Modify the opened file in a way that the certificate for the mail subdomain is in `/usr/local/vesta/ssl` as `mail.crt` and `mail.key` with the same permissions as for `certificate.crt` and `certificate.key` there (just copy, paste and modify)
5. run `./v-update-host-certificate` in the shell

## Generate DH params
To ensure the so called Forward Secrecy, we will need to utilize DH params. That means that we first have to generate it. To do so, follow these steps:

1. Login via SSH as root user
2. `cd /usr/local/vesta/ssl`
3. `openssl dhparam -out /usr/local/vesta/ssl/dhparam.pem 4096`
4. Get yourself a cup of coffee and some cookies. It will most likely take quite a while

## Configuring the Server
We are done as far as the preperations go. Now, we have to configure dovecot and exim.

### Dovecot
Go to the control panel and enter the server settings on the top of the screen. Now search for dovecot and press the "configure" button. Seach for the editable file ending to `ssl.conf ` and enter the following:

```dovecot
ssl = yes
ssl_cert = </usr/local/vesta/ssl/mail.crt
ssl_key = </usr/local/vesta/ssl/mail.key
ssl_min_protocol = TLSv1.2
ssl_cipher_list = EECDH+AESGCM:EDH+AESGCM
ssl_prefer_server_ciphers = yes
ssl_dh = </usr/local/vesta/ssl/dhparam.pem
```

To exclusively accept encrypted IMAP and POP3 connections, enter `ssl=required` instead of `ssl=yes`.

### Exim
Go to the control panel and enter the server settings on the top of the screen. Now search for exim and press the "configure" button. Now search for "tls_" and modify the settings as follows:

```exim
tls_advertise_hosts = *
tls_certificate = /usr/local/vesta/ssl/mail.crt
tls_privatekey = /usr/local/vesta/ssl/mail.key
# tls_require_ciphers = NORMAL:%LATEST_RECORD_VERSION:-VERS-SSL3.0:-VERS-TLS1.0:-VERS-TLS1.1
tls_dhparam = /usr/local/vesta/ssl/dhparam.pem
```

Consider adding the following to disallow SMTP authentication via unencrypted connections:

```exim

auth_advertise_hosts = ${if eq{$tls_cipher}{}{}{*}}
```

!!! Note that exim in Debian comes with GnuTLS as per default instead of with OpenSSL. So we cannot modify alot, unfortunately. Especially as GnuTLS tends to be buggy. If it works, uncomment the `tls_require_ciphers` part, but make sure you test sending and receiving emails afterwards.