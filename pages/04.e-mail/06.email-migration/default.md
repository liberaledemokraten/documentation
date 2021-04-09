---
title: 'Email Migration'
aura:
    pagetype: website
metadata:
    'og:url': 'https://doc.ld-online.net/e-mail/email-migration'
    'og:type': website
    'og:title': 'Email Migration | LD Documentation'
    'og:author': 'Liberale Demokraten | Documentation'
    'twitter:card': summary_large_image
    'twitter:title': 'Email Migration | LD Documentation'
    'twitter:site': '@LDSozialliberal'
    'twitter:creator': '@LDSozialliberal'
    'article:published_time': '2021-02-06T01:05:04+01:00'
    'article:modified_time': '2021-02-06T01:05:04+01:00'
    'article:author': 'Liberale Demokraten | Documentation'
---

!!! If you are not interested in the technical details, but just want to migrate your LD E-Mails, check out [E-Mail Migration - LD Mail](https://mail.liberale-demokraten.de/setup/migration.html) (in German).

When switching from one webspace to another or from one email provider to another, one of course wants to move existing data too. Obviously, some of them will be critical for your own business. There are several methods to do so. All of them require that the email accounts in both mailservers are still up and running.

## Assisted Migration (Recommended)
This is most likely the easiest and safest methos for end-users. However, it requires that [IMAP Sync](./imap-sync) is configured as described in the linked page. Imap Sync will have to be triggered by using HTML & PHP afterwards. A proof of concept on how to achieve that may be found on GitHub under [liberaledemokraten/php-email-migration](https://github.com/liberaledemokraten/php-email-migration).

## Manual Migration
The manual approach is probably among the easiest to go as everyone would just register their accounts on both servers using the same email client (e.g. Thunderbird has the easiest approach here). Thenafter, he may just drag & drop copy or move all the emails from the old account to the new one. Each folder's content to the respective folder in the new account.

## ImapSync
To do so, an application must be used that gets feeded in the login credentials to both the old and new account and copy all emails from one account to the other. For instance, [imapsync](https://github.com/imapsync/imapsync) is one such application. More information can be found within this documentation under the sub-page [IMAP Sync](./imap-sync).

## Roundcube ImapSync (Deprecated)
!! This method seems to not work. The plugin is probably outdated and not working with newer roundcube versions. Thus this is still displayed here just to keep the method archived.

This is the easiest and a self-managed method. It is the recommended method as it can be done through our webmail interface without requiring any troublesome manual work. More details on this can be found in GitHub [server-gurus/RCimapSync](https://github.com/server-gurus/rcimapsync).

The usage is pretty easy. You just have to enter the IMAP credentials of the e-mail account with your previous provider and let the Roundcube plugib deal with downloading the messages into the mailbox that you are currently logged in to.

![](https://cloud.githubusercontent.com/assets/8064903/23556852/4b069624-002e-11e7-8313-0c9896b8efdb.png)

Essentially, it is also possible to run both accounts in parallel. In the future, it'd also be possible to set up a secondary mailserver on another location and snchronize the accounts on both server with each other. That way, we could improve the reachability in case of a downtime. In combination with a loadbalancer, we could even run both mailservers under the same domain name.

### Installation

1. Install ImapSync as described in [IMAP Sync](./imap-sync)
2. Do as follows:

```sh

cd /path/to/your/roundcube/plugins/
git clone https://github.com/server-gurus/RCimapSync.git ./imapsync
```

3. Open the roundcube configuration file and add `$config['plugins'] = array('plugin1','plugin2',[...],'imapsync');`
4. Import the supplied `mysql.initial.sql` to your mysql database
5. Create a [MySQL `view`](https://www.mysqltutorial.org/mysql-views-tutorial.aspx) for the table `imapsync` if your imported table name is different
6. Move the `/bin/` directory outside the webroot
7. Open the `/bin/` folder and rename `config.sample` to `config.conf`
8. Edit `config.conf` and add the credential information to your database
9. Set up a cronjob for `/bin/imapsync.pl`
10. You might also need to install `liblockfile-simple-perl` and ( `libsys-syslog-perl` or `libunix-syslog-perl` ) on Debian-based systems

