---
title: Webmail
---

!!! Note that any webmail client such as roundcube, rainloop, etc. can be used. We are using roundcube.

## Login
The webmail can be found under [mail.liberale-demokraten.de](https://mail.liberale-demokraten.de). Use the following login credentials to log into your email account:
* **username**: your full email address (e.g. john.doe@example.com)
* **password**: your password

## Basic Information
Please note the following basic information when configuring the webmail client or any other email client:

* IMAP Port: 993 
* SMTP Port: 465
* Encryption: SSL/TLS
* Username: full email address
* Password: password
* IMAP / SMTP Server: The domain name both are reachable from and the certificate has been issued for is ([mail.liberale-demokraten.de](https://mail.liberale-demokraten.de))

!!! Some e-mail clients may work without manually entering the above details as the environment is possibly configured to support [Auto Discover](../auto-discover).

## Vesta's Webmail
Vesta comes with roundcube preinstalled. It however is the version distributed by the Debian repos, so very likely an older version. The installed version as this doc was written was 1.3.15. That version also does not have a mobile-friendly skin included.

### Setting Up
The database and configuration files may not be set to the required permissions. `chmod 655` or `chmod 755` them. Thenafter, the webmail should be reachable under [liberale-demokraten.de/webmail](https://liberale-demokraten.de/webmail). We're not using this, so I won't be writing any further on this part. For more information, check out the VestaCP forum.

### Disabling
Vesta uses the roundcube version provided by the Debian repos. So uninstalling can be done via `apt-get`. The more recommended method however is to not remove any packages installed by Vesta as there may be some dependencies. Instead you can simply remove the /webmail alias from the server. To do so, just do as follows:

```sh
rm /etc/apache2/conf.d/roundcube.conf
service apache2 restart
```

### Re-Enable
If you have not uninstalled roundcube but simply removed the alias as described above, you can simply re-enable the roundcube webmail client. To do so, do as follows:

```sh
ln -s /etc/roundcube/apache.conf /etc/apache2/conf.d/roundcube.conf
service apache2 restart
```

More information on:
* [VestaCP Forum](https://forum.vestacp.com/viewtopic.php?t=13344#p54106)
* [DWAVES](https://dwaves.de/2018/08/09/vestacp-disable-roundcube-webmail/)

## Own Roundcube (Recommended)
To use a more recent Roundcube version, we cannot use the roundcube package delivered by Debian. So we can simply disable it by removing the alias as described above. 

### Set Up
To set it up, do as follows:

* Log into the Vesta panel as admin
* Click "Apps" on the top banner
* Search for "Roundcube"
* Install Roundcube under [mail.liberale-demokraten.de](https://mail.liberaler-demokraten.de)
* Configure the installation accordingly

### Update Vesta Link
In Vesta, you can also set up, which URL to call for the webmail service. Since we are not using the default /webmail directory anymore, we should update that too. You can do this as follows:

* Log into Vesta as admin
* Click on "Server" in the top banner
* Hover over the domain name and click on "configure"
* expand "Mail"
* Set Webmail URL to [https://mail.liberale-demokraten.de](https://mail.liberale-demokraten.de)
* You can select the domain SSL/TLS Certificate too