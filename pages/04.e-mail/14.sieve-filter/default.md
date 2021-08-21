---
title: 'Sieve Filter'
---

[Sieve](https://en.wikipedia.org/wiki/Sieve_(mail_filtering_language)) is a languae used for email filtering. It is commonly utilized for server-side filtering. That is also what we are going to do.

## Installing
We are using Dovecot IMAP server. Thus, we will use Dovecot's Pigeonhole Sieve interpreter that will filter incoming mails according to defined rules written in the sieve language. For this to work, we will first add the [dovecot repos](https://doc.dovecot.org/installation_guide/dovecot_community_repositories/) to our `apt` lists.

```
echo "deb https://repo.dovecot.org/ce-2.3-latest/debian/`lsb_release -cs` `lsb_release -cs` main" | tee /etc/apt/sources.list.d/dovecot.list
curl -fsSL https://repo.dovecot.org/DOVECOT-REPO-GPG | apt-key add -
apt-get update
```

The next step now is pretty easy. Update dovecot and install the `managesieve` plugin:

```
apt-get upgrade
apt-get install dovecot-sieve dovecot-managesieved
```

Now sieve is installed, but not yet enabled.

## Enabling Sieve
To enable sieve, just modify the Dovecot config to add the `sieve` to the list of supported protocols. You may do so by uncommenting the respective line in `/etc/dovecot/conf.d/20-managesieve.conf`. That's it.

## Managesieve in Roundcube
To enable sieve filtering in Roundcube, go to the `managesieve` plugin and `cp config.inc.php.dist config.inc.php`. Thenafter enable the plugin in the general configuration file of Roundcube.