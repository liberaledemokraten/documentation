---
title: 'IMAP Sync'
---

## General Information
[IMAP Sync](https://imapsync.lamiral.info) is an open source application by Gilles Lamiral available in [this GitHub repository](https://github.com/imapsync/imapsync). It's license is a "no limit public license" which can be interpreted as a Public Domain license, visible [here](https://github.com/imapsync/imapsync/blob/master/LICENSE).

## Installation
The installation on Debian is quite easy. Just follow these steps:

```sh

apt-get install            \
  libauthen-ntlm-perl     \
  libcgi-pm-perl          \
  libcrypt-openssl-rsa-perl   \
  libdata-uniqid-perl         \
  libencode-imaputf7-perl     \
  libfile-copy-recursive-perl \
  libfile-tail-perl        \
  libio-socket-inet6-perl  \
  libio-socket-ssl-perl    \
  libio-tee-perl           \
  libhtml-parser-perl      \
  libjson-webtoken-perl    \
  libmail-imapclient-perl  \
  libparse-recdescent-perl \
  libmodule-scandeps-perl  \
  libreadonly-perl         \
  libregexp-common-perl    \
  libsys-meminfo-perl      \
  libterm-readkey-perl     \
  libtest-mockobject-perl  \
  libtest-pod-perl         \
  libunicode-string-perl   \
  liburi-perl              \
  libwww-perl              \
  libtest-nowarnings-perl  \
  libtest-deep-perl        \
  libtest-warn-perl        \
  make                     \
  cpanminus


cd /usr/local/src
git clone https://github.com/imapsync/imapsync.git
cd imapsync
chmod +x imapsync
ln -s /usr/local/src/imapsync/imapsync /usr/bin/imapsync
```

## Update
Updating the application is quite straightforward either. Just do as follows:

```sh

cd /usr/local/src/imapsync
git pull origin
chmod +x imapsync
```

!!! If you wish to automate the updating process, you can do so through a shell script and cron.


## Uninstall

```sh

rm /usr/bin/imapsync
rm -rf /usr/local/src/imapsync
```

!!! You may also want to uninstall the previously installed dependency packages.