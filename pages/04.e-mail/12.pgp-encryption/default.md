---
title: 'PGP Encryption'
---

## General
There are two common standards when it comes to end to end encrypted emailing. One is called [S/MIME](https://en.wikipedia.org/wiki/S/MIME) and it requires that certificates be issued by trusted CAs similar to SSL/TLS certificates. In fact, they do follow the same standards anyways. The other one is [PGP](https://en.wikipedia.org/wiki/Pretty_Good_Privacy) and it has a completely decentralized infrastructure. Everyone can issue their own certificates and publish it.

Surely, that sounds lesser secure and anyone could impose being someone else. But actually, it does not matter as we are not using PGP certificates to identify a person in our case. Nor do we use it to be the one initiating encrypted communications. In our case, we will be responding to received messages, so we can safely assume that the certificate of the sender we received the message from can be used to contact them.

So, PGP is a rather unexpensive and still safe way to go for us. To that end, the public PGP certificate should be published on our website. That way, people can find it and use it to contact us.

For this to work, anybody who needs access to the encrypted mails and the mailbox need access to the PGP key. Some clients supporting PGP are [Mozilla Thunderbird](https://www.thunderbird.net/) (Windows, Linux & mac OS) as well as [Microsoft Outlook](https://www.microsoft.com/de-de/microsoft-365/outlook/email-and-calendar-software-microsoft-outlook) (Windows with [Gpg4Win](https://www.gpg4win.de/index.html)) and [FairEmail](https://email.faircode.eu/) (on Android with [OpenKeychain](https://www.openkeychain.org/)). Aternatively a Roundcube webmail client with the Enigma plug-in or the [Mailvelope](https://www.mailvelope.com/de) browser add-on enabled can be used too.

## Roundcube Plugin
Roundcube offers a PGP encryption through a plugin called "Enigma". Enable it by following these steps:

1. `apt-get install gnupg gnupg-agent`
2. `cd` to the directory where roundcube is installed
3. Rename `plugins/enigma/config.inc.php.dist` to `plugins/enigma/config.inc.php`
4. Create a directory at a place outside the webroot (at least not accessible through HTTP), but accessible via PHP
5. Add the path to the newly created directory for `$config['enigma_pgp_homedir']` in the configuration file
6. Edit `config/config.inc.php` for it to enable the plugin: `$config['plugins'] = array('enigma');`

## Browser Extension
Alternatively, there's also a client-side solution that will store keys on the users computer instead of in the server. That can be done through browser extensions such as [Mailvelope](https://www.mailvelope.com/de) which is available for Firefox, Chrome and Edge. You should add the webmail domain as a domain and enable both HTTPS and API options in the extension for better support.

## PGP Web Key Directory
The Web Key Directory (WKD) allows a client to request a public key from the e-mail provider through HTTPS. Further details can be found on the [German BSI](https://www.bsi.bund.de/DE/Themen/Kryptografie_Kryptotechnologie/Kryptografie/EasyGPG/EasyGPG_node.html) and the [GnuPG Wiki](https://wiki.gnupg.org/WKD). Alot of major clients support WKD.