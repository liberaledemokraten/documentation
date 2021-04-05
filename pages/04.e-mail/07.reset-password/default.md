---
title: 'Reset Password'
aura:
    pagetype: article
    description: 'How to reset the password of an e-mail account on the LD Documentation'
metadata:
    description: 'How to reset the password of an e-mail account on the LD Documentation'
    'og:url': 'https://doc.ld-online.net/e-mail/reset-password'
    'og:type': article
    'og:title': 'Reset Password | LD Documentation'
    'og:description': 'How to reset the password of an e-mail account on the LD Documentation'
    'og:author': 'Liberale Demokraten | Documentation'
    'twitter:card': summary_large_image
    'twitter:title': 'Reset Password | LD Documentation'
    'twitter:description': 'How to reset the password of an e-mail account on the LD Documentation'
    'twitter:site': '@LDSozialliberal'
    'twitter:creator': '@LDSozialliberal'
    'article:published_time': '2021-02-05T16:08:30+01:00'
    'article:modified_time': '2021-02-05T16:08:30+01:00'
    'article:author': 'Liberale Demokraten | Documentation'
---

There are three ways to change an e-mail account's password.

## Vesta
### Web UI
Log into the web UI of Vesta as the user the domain is hosted in and change the password.

### CLI
It is also possible to call vesta's scripts under `vesta/bin`:

```
v-change-mail-account-password USER DOMAIN ACCOUNT PASSWORD
```

## Webmail
If the `password` plug-in for roundcube is enabled and set to the `vesta` driver, you can also reset the password by logging in to the werbmail, and selecting "Password" in the settings. For this to work, you'll have to make sure the driver exists in `plugins/password/drivers/vesta.php`. Be reminded that Vesta comes with a driver preconfigured under `/usr/share/roundcube/plugins/password/drivers/vesta.php`.