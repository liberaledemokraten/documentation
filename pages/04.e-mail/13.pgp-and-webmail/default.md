---
title: 'PGP & Webmail'
---

## Roundcube Plugin
Roundcube offers a PGP encryption through a plugin called "Enigma". Enable it by following these steps:

1. `apt-get install `gnupg gnupg2 gnupg-agent`
2. `cd` to the directory where roundcube is installed
3. Rename `plugins/enigma/config.inc.php.dist` to `plugins/enigma/config.inc.php`
4. Create a directory at a place outside the webroot (at least not accessible through HTTP), but accessible via PHP
5. Add the path to the newly created directory for `$config['enigma_pgp_homedir']` in the configuration file
6. Edit `config/config.inc.php` for it to enable the plugin: `$config['plugins'] = array('enigma');`

## Browser Extension
Alternatively, there's also a client-side solution that will store keys on the users computer instead of in the server. That can be done through browser extensions such as [Mailvelope](https://www.mailvelope.com/de) which is available for Firefox, Chrome and Edge. You should add the webmail domain as a domain and enable both HTTPS and API options in the extension for better support.

## PGP Web Key Directory
The Web Key Directory (WKD) allows a client to request a public key from the e-mail provider through HTTPS. Further details can be found on the [German BSI](https://www.bsi.bund.de/DE/Themen/Kryptografie_Kryptotechnologie/Kryptografie/EasyGPG/EasyGPG_node.html) and the [GnuPG Wiki](https://wiki.gnupg.org/WKD). Alot of major clients support WKD.