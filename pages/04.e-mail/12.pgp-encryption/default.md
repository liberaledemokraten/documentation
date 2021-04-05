---
title: 'PGP Encryption'
---

There are two common standards when it comes to end to end encrypted emailing. One is called S/MIME and it requires that certificates be issued by trusted CAs similar to SSL/TLS certificates. In fact, they do follow the same standards anyways. The other one is PGP and it has a completely decentralized infrastructure. Everyone can issue their own certificates and publish it.

Surely, that sounds lesser secure and anyone could impose being someone else. But actually, it does not matter as we are not using PGP certificates to identify a person in our case. Nor do we use it to be the one initiating encrypted communications. In our case, we will be responding to received messages, so we can safely assume that the certificate of the sender we received the message from can be used to contact them.

So, PGP is a rather unexpensive and still safe way to go for us. To that end, the public PGP certificate should be published on our website. That way, people can find it and use it to contact us.

For this to work, anybody who needs access to the encrypted mails and the mailbox need access to the PGP key. The supported clients are [Mozilla Thunderbird](https://www.thunderbird.net/) (Windows, Linux & mac OS) as well as [Microsoft Outlook](https://www.microsoft.com/de-de/microsoft-365/outlook/email-and-calendar-software-microsoft-outlook) (Windows with [Gpg4Win](https://www.gpg4win.de/index.html)) and [FairEmail](https://email.faircode.eu/) (on Android with [OpenKeychain](https://www.openkeychain.org/)). Aternatively a roundcube webmail client with the Enigma plug-in enabled can be used too.