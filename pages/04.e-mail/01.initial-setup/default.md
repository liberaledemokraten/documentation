---
title: 'Initial Setup'
---

VestaCP will have dealt with the most parts of the initial setup, but you also need to go through the following steps.

## Vesta Recommendation
In its [documentation](https://vestacp.com/docs/#how-to-setup-mail-server), VestaCP recommends to do the following:

1. Install Vesta Control Panel

2. Get a domain name

3. Link domain name with your new server.
[see how to set up vanity nameservers](https://vestacp.com/docs/#how-to-setup-vanity-nameservers)

4. Make sure port 25 is not firewalled
[you can use SMTP diagnostic tool](http://mxtoolbox.com/diagnostic.aspx)

5. Make sure server hostname is [FQDN](http://en.wikipedia.org/wiki/Fully_qualified_domain_name) compliant

```bash
root@localhost:~# hostname
localhost
root@localhost:~# v-change-sys-hostname mail.vestacp.com
root@localhost:~# hostname
mail.vestacp.com
```

6. Set PTR record.
Please contact your ISP to find out how to do it. Your ISP is most likely the company you are hosting your server with.

7. Check server IP reputation.
[Blacklist Check](http://mxtoolbox.com/blacklists.aspx)

8. Enable DKIM support.
This is optional step but highly recommended. DKIM can be enabled in the Vesta under MAIL tab.

9. Add email users and you are all set 


## Whitelisting
After having set the PTR records, you should also enter your domain name in the [DNS Whitelist](https://www.dnswl.org/selfservice/). That way, your server enters a global whitelist that some mail servers and clients may respect.