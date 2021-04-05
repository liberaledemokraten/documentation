---
title: 'DNS CAA'
---

!!! This article is not necessarily limited to web security, but has effects to the security to any servers using TLS (e.g. mail server). But for simplicity, it was added under the Web section.

DNS Certification Authority Authorization (DNS CAA) is a method of authorizing only specific CAs for issuing specified types of certificates for the specific domain. To achieve that, a CAA entry is added within the DNS server for the domain name in question. Here's a sample entry:

`@ CAA IN 0 issue "letsencrypt.org"`

To enable the issuance of wildcard certificates:

`@ CAA IN 0 issuewild "letsencrypt.org"`

To enable the issuance of wildcard but not common certificates:

```
@ CAA IN 0 issue ";"
@ CAA IN 0 issuewild "letsencrypt.org"
```

to enable reporting for invalid certificate requests, you should add the following entry too:

`@ CAA IN 0 iodef "mailto:caa@example.com"`
