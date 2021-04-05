---
title: 'Exim Configuration'
---

!!! Also see [SSL/TLS Configuration](../ssl-tls-configuration).

Exim is a quite powerful MTA allowing you to define ACLs (Access Control Lists) with various conditions. Please see the [Specification of the Exim Mail Transfer Agent](https://www.exim.org/exim-html-current/doc/html/spec_html/) for more on configuring Exim.

## Deny Outgoing Mails from Unknown Domains
! Caution: The below suggestion may also affect incoming mails. See the second rule below to avoid that.
Under `acl_smtp_mail`, we can define what shall be checked for outgoing mails from users authorized via SMTP. That is where we may add the following entry:

```conf
  deny
    ! condition = ${if exists{/etc/exim4/domains/${sender_address_domain}}}
    message = You are not authorized to send emails with FROM domain $sender_address_domain.  
```

What this entry does is quite simple: It checks whether the FROM domain has been added as a mail domain to vesta and denies outgoing mails if that is not the case.

To ensure that this does not affect incoming mails, you should prepend the following rule:

```
  accept
    condition = ${if eq{}{$authenticated_id}}
```

This entry aims to accept non-smtp mails (e.g. using sendmail) and incoming mails will without further checking them within this ACL. Thus, any rules below this should solely affect outgoing mails from authenticated SMTP users.

## Allow Only Authorized User in FROM
! Caution: The below suggestion may also affect incoming mails. You may also want to see the above notes on this.
We may want to make sure an authorized user may only send emails with the identity of itself. With this rule, it will be impossible for an authorized user to use another FROM address. To achieve that, add the following entry similarly to above:

```conf
  deny
    ! condition = ${if eq{$authenticated_id} {$sender_address}}
    message = You are not authorized to send emails for $sender_address.
```

Please see the addition to the previous rule. If you have already prepended it once, you won't have to add it again for each rule, though.

## Add Custom Headers
In case you want to add a custom header when sending mails, you may do so in the `transports` section, e.g. under `remote_smtp`. To do so, just add the header using `headers_add = <headerName>: <headerValue>`.