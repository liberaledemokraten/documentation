---
title: Backups
---

Data Security not only includes principles to prevent access by unauthorized parties, but also securing data from possibly losing it. So, that means that baclups are necessary. To that end, Vesta by default makes daily backups for each user. The own backed up data can be downloaded by each user.

Additional to that, all of the user backups shall be zipped together with a timestamp as its filename. This zipped file shall then be encrypted using a random passphrase that shall be encrypted using the BuVo's public key. Basically, the encryption process shall follow the principles as outlined under [Encryption](../encryption).

A shell script doing that will be uploaded here shortly, including a command for automizing the process.