---
title: 'Webmail: Multiple Identites'
---

## Multiple Identites for outgoing Mails
You may want to add additional identities for outgoing mails in the Roundcube settings. Please note that those identites are by default solely for outgoing mails and the same credentials as the logged in user will be used for any sent mail. If you want to add accounts instead, see the below section.

Adding an identity might be useful if you want to send an e-mail with an alias of your account. For this to work however, sending with the aliases set as FROM address should be accepted by the server. See [Exim Configuration](../exim-configuration) for further on the server-side configuration.

## Multiple Accounts 
You may want to manage multiple e-mail accounts within a single interface. However, you might not want to log into each of them manually, but log into your personal account and access the other mailboxes from there. Luckily for us, there are many plugins for Roundcube available, one of which addresses this issue. The plugin is called "ident_switch" and can be found in BitBucket [BoresExpress/ident_switch](https://bitbucket.org/BoresExpress/ident_switch/src/master/).

![](https://i.imgur.com/rRIqtA8.jpg)

