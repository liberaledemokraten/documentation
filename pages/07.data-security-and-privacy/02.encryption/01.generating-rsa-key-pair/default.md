---
title: 'Generating RSA Key Pair'
---

! If RSA happens to be deemed insecure (e.g. due to quantum computing), you should switch to a safer algorithm ASAP. FOr now, this is however sufficiently secure in foreseeable future. Especially as quantum safe cryptography is not yet widely tested and implemented, thus out of question for us.

Generating an RSA key pair is quite easy. Just enter the following line into the CLI.

```
$ cd /path/to/external/drive
$ openssl req -x509 -newkey rsa:4096 -sha512 -keyout key.pem -out cert.pem -days 7300
```

Now you will be prompted the following questions, just fill them in.

```
writing new private key to 'key.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:
```

That's it, you just generated a 4096-bit sized RSA key pair. Now store the keys as specified under [Encryption](../) and share the public key / certificate with the maintainers.