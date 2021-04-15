---
title: 'Exim Configuration'
---

!!! Also see [SSL/TLS Configuration](../ssl-tls-configuration).

Exim is a quite powerful MTA allowing you to define ACLs (Access Control Lists) with various conditions. Please see the [Specification of the Exim Mail Transfer Agent](https://www.exim.org/exim-html-current/doc/html/spec_html/) for more on configuring Exim.

## Allow Authenticated Users to Send from Aliases
!!! [Suffix aliases](../suffix-aliasing) (i.e. `jane.doe+alias@example.com`) are also covered by this configuration.

We are saving aliases and forwardings in the file `\etc\exim4\domains\<domain>\aliases`. The file looks like this:

```
alias@example.com:main@example.com
forward@example.com:target@example.com,another-target@example.com
```

It is notable that aliases always have a single target while a forwarding has at least one target but may have multiple comma-separated targets. That means that a forwarding will be saved just the same way an alias is saved in our file. So, allowing aliases will also allow sending mails from a forwarding address from the target domain, if there is exactly one target. Here another example on that:

```
forward@example.org:target@example.com
```

In the sense of the above shown example, that'd mean that `target@example.com` could send an email with the FROM address `forward@example.org`. That's a case which is exclusively with forwardings possible as aliases are always within the same domain.

To achieve that, we can define under `acl_smtp_mail` what shall be checked for outgoing mails from users authorized via SMTP. That is where you may add the following entry:

```conf
    # accept if FROM address is a suffix alias of authenticated address
  accept
    condition = ${if match {$sender_address} {(${local_part:$authenticated_id}\+.*@${domain:$authenticated_id})}}
    logwrite = AUTH OK - FROM address $sender_address is a suffix alias of authenticated user

  # accept if FROM address is an alias for the authenticated user $authenticated_id.
  accept
    condition = ${if and{\
                   { exists{/etc/exim4/domains/${sender_address_domain}/aliases} }\
                   { eq{$authenticated_id} {${lookup{$sender_address}lsearch{/etc/exim4/domains/${sender_address_domain}/aliases}}} }\
                 }}
    logwrite = AUTH OK - FROM address is an alias or unique forwarding target of the authenticated user $authenticated_id
```

Please note that this option may require that both accounts are hosted on the same server. The domains however may be managed by different users. That is not a problem, as only permitted users are able to set the aliases and forwardings.

## Deny Outgoing Mails from Unknown Domains
! Caution: The below suggestion may also affect incoming mails. See the second rule below to avoid that.

Just as described above, you may add the following entry under `acl_smtp_mail`:

```conf
  deny
    ! condition = ${if exists{/etc/exim4/domains/${sender_address_domain}}}
    message = You are not authorized to send emails with FROM domain $sender_address_domain.
    logwrite = AUTH ERROR - authenticated user $authenticated_id set disallowed FROM domain $sender_address_domain
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
    logwrite = AUTH ERROR - authenticated user $authenticated_id is not permitted to set FROM address $sender_address
```

Please see the addition to the previous rule. If you have already prepended it once, you won't have to add it again for each rule, though.

## Add Custom Headers
In case you want to add a custom header when sending mails, you may do so in the `transports` section, e.g. under `remote_smtp`. To do so, just add the header using `headers_add = <headerName>: <headerValue>`.