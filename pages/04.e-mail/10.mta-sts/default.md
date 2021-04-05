---
title: MTA-STS
---

MTA-STS (Mail Transfer Agent - Strict Transport Security, short STS) is a relatively new tool for securing the transport of emails in both ways (sending / receiving). It has been standardized in [RFC-8461](https://tools.ietf.org/html/rfc8461) and can be used in practice. MTA-STS is considered as an alternatice to DANE without requiring DNSSEC.

## STS Domain
To use MTA-STS, you need a subdomain for the root domain name that you want to configure it for. For instance mta-sts.example.com if you want to configure MTA-STS for the domain example.com. The Subdomain should be reachable via HTTPS and it should use a TLS certificate from a trusted CA..

## Text Entry
Within the newly created subdomain, the URI mta-sts.example.com/.well-known/mta-sts.txt has to return the configuration data. A sample configuration text entry looks like this:

```
version: STSv1
mode: enforce
mx: mail.example.com
max_age: 2500000
```

When initially configuring MTA-STS, it's recommended to use the following entry instead for testing purposes:

```
version: STSv1
mode: testing
mx: mail.example.com
max_age: 604800
```

There are two methods of returning the above configuration using nginx. The first method is to save the configuration as an actual text file and return it:

```nginx
location /.well-known/ {
    root /home/{{user}}/web/mta-sts.example.com/well-known;
    try_files $uri $uri.html $uri.txt @fallback;
}
```

or alternatively, you can implement the configuration statically into your server configuration. The response should be handled slightly faster this way, but especially only the root user (or a sudoer) could make modifications to the configuration without the actual user sniffing around to cause unwanted troubles. The configuration would then look as follows:

```nginx
location /.well-known/mta-sts.txt {
    return 200
"version: STSv1
mode: enforce
mx: mail.example.com
max_age: 2500000";
}
```

## DNS Entry
For the configuration to take effect, a DNS entry is required. The entry must be added for the actual domain name example.com and not for the newly created subdomain. The entry looks as follows:

`_mta-sts TXT IN "v=STSv1; id=202009152323;"`

For each modification to the configuration, the value of the `id` has to be modified too. The system we follow is to use the date of modification. In the above sample for instance, the last modification was in the year 2020 within the 09th month (September) and its 15th day at 23 hours and 23 minutes.