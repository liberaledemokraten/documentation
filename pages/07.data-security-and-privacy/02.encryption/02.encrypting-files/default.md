---
title: 'Encrypting Files'
---

! Note that this article assumes using AES encryption is safe. It is for now and in foreseeable future even where quantum computers are considered. But if the circumstances change, this article as well as any encrypted content shall be updated ASAP


## AES Encryption
Encrypting using AES is pretty easy. There are multiple implementations supporting AES. Here's one example

```
openssl enc -aes-256-cbc -md sha512 -pbkdf2 -iter 100000 -salt -in plain.file -out encrypted.file -pass <your_passphrase>
```

Decrypting works as follows in this case

```
openssl enc -d -aes-256-cbc -md sha512 -pbkdf2 -iter 100000 -salt -in encrypted.file -out plain.file -pass <your_passphrase>
```

## Random Password Generation
!!! The generated passphrases here will be 256 bytes (octets) large. The general rule however should be that the password length should be no lesser than 64 bytes of size and no larger than the upper limit your RSA key can encrypt. With our 4096-bit RSA key, the upper limit is set at 510 bytes. So any size n with 63 < n < 510 is fine.

If you do not add the `-pass` argument, you will be prompted for a passphrase. If you prefer using a rather random and safe passphrase, you may use `pwgen` or `openssl`. The following two options are available.

```
pwgen -s -y 256 1 > passphrase.txt
```

or

```
openssl rand -hex 256 -out passphrase.txt
```

Your passphrase now may look similar to the following

```
=r?{#B*kaA+{]b:yLRktvgmhsR?}E.jTI;=symEITfPZ[(i>fNl\NvGJ:JIcq+8@"K{rqX)wK4UY_t\?B7O92$G2(Jnz`i]Df.x&?*iytpC[\2My.uP^'$GY}3QzWA7(n9fTn4q:V0H]=6,F|y|/!OFI=o;h0fDr$+i8gb$7dvDt/MXC=L_m9i3`=Hu&w3$w~#+#@h]:g`XBBox/-~/8=BDDlURoQ~X,;OA]tzVJyn`@j8+|b;5WV}0-w_zDV6EY
```


## Encrypt with Random Password
You can now use the generated passphrase to encrypt your file using AES as follows

```
openssl enc -aes-256-cbc -md sha512 -pbkdf2 -iter 100000 -salt -in plain.file -out encrypted.file -pass file:./passphrase.txt
```

The decryption works by adding a `-d` the same way as described previously.


## Encrypting the Passphrase
Obviously, we don't want anyone to gain access to the passphrase we used to encrypt our content. So, we can now encrypt it using our public RSA key and preferably store the encrypted passphrase and the encrypted file in different locations. To do so, you may use the following command

```
openssl rsautl -encrypt -inkey cert.pem -pubin -in passphrase.txt -out passphrase.enc
```

You can decrypt the encrypted passphrase through the following command

```
openssl rsautl -decrypt -inkey key.pem -in passphrase.enc -out passphrase.txt
```

## Further Reading
If you want to read more about the recommended process, you may read [Encrypt and decrypt files to public keys via the OpenSSL Command Line](https://raymii.org/s/tutorials/Encrypt_and_decrypt_files_to_public_keys_via_the_OpenSSL_Command_Line.html) as well as [Sign and verify text/files to public keys via the OpenSSL Command Line](https://raymii.org/s/tutorials/Sign_and_verify_text_files_to_public_keys_via_the_OpenSSL_Command_Line.html). Also, read [File Encryption | Tower Floor -- Encryption](https://antofthy.gitlab.io/info/crypto/file_encrypt.txt).