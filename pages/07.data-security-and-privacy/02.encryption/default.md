---
title: Encryption
---

## What to Encrypt
We will be storing some files encrypted. This will include the following:

* Account Credentials
* Externally Stored Backup Files
* Sensitive information only the BuVo needs to know

## Basic Rules
The basic idea will look as follows:

1. The BuVo receives a public/private RSA key pair no shorter than 4096 bits of size. More on it in the specific article.
2. We are generally using AES encryption to encrypt any files. AES keys may be encrypted with the public key of the BuVo.
3. Any account credentials or sensitive information the BuVo needs to or should know about must be stored as described under 2
4. Any backups stored externally must be stored as described in the second point
5. Encrypted data must not reveal the data structure (e.g. file names)

## Requirements
The requirements for fulfilling the methods described in the articles under this category are:

1. OpenSSL. LibreSSL should work too
2. CLI (Command Line Interface)

You should preferably use Linux. However, a WSL application on Windows or CygWin should work too.

## Key Storage
In general, it is advised to use a Hardware Security Module (HSM) to generate and use any such sensitive keys such as the BuVo key which acts similar to a master key. Yet due to financial reasons as well as the fact that the usage of the keys should be rarely necessary, it is not significantly insecure to store them offline in a flash drive and that's the recommended method of storage here.

Preferably, it should be a FIPS 140-2 level 3 compatible drive, however storing the data within a common USB flash drive which is stored in a safe or drawer solely accessible using a physical key or code is sufficient too. Especially in the latter case, the private key must be stored AES encrypted using a passphrase known to the BuVo members only.

The private key must under no circumstances be stored in a device usually connected to any type of network, regardless of whether it is the internet, the public switch telephone network, an intranet or the local area network.