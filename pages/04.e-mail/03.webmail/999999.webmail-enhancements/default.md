---
title: 'Webmail Enhancements'
---

You may increase the user experience for the webmail users by enabling various plugins. Here are a few samples.

## Multiple Identites for outgoing Mails<a id="smtp-identities"></a>
You may want to add additional identities for outgoing mails in the Roundcube settings. Please note that those identites are by default solely for outgoing mails and the same credentials as the logged in user will be used for any sent mail. If you want to add accounts instead, see the below section.

Adding an identity might be useful if you want to send an e-mail with an alias of your account. For this to work however, sending with the aliases set as FROM address should be accepted by the server. See [Exim Configuration](../exim-configuration) for further on the server-side configuration.

## Multiple Accounts <a id="from-identities"></a>
You may want to manage multiple e-mail accounts within a single interface. However, you might not want to log into each of them manually, but log into your personal account and access the other mailboxes from there. Luckily for us, there are many plugins for Roundcube available, one of which addresses this issue. The plugin is called "ident_switch" and can be found in BitBucket [BoresExpress/ident_switch](https://bitbucket.org/BoresExpress/ident_switch/src/master/).

![](https://i.imgur.com/rRIqtA8.jpg)

## Filters <a id="filter"></a>
You may use the managesieve plugin for this. See [Sieve Filter](../../sieve-filter) for further on this.

## Notification <a id="notification"></a>
You may want to be notified on incoming mails when the webmail is open in some tab. That can be achieved with the [stremlau/html5_notifier](https://github.com/stremlau/html5_notifier) plugin.

![](https://raw.githubusercontent.com/stremlau/html5_notifier/master/screenshot.png)