---
title: 'Security Monitoring'
---

!!! Note that the below methods is implemented easiest if used together with Scott Helme's [Report URI](https://report-uri.com) service. It however features email notifications only in paid plans. Otherwise, you have to log in and check the stats manually from time to time.

### Certification Issuance
Using CAA and CT, it is possible to be notified by email when a CA issues a certificate for your domain name. Also see the [DNS CAA](web/dns-caa) and [Web Reporting and Monitoring](web/reporting-and-monitoring) articles.

### NEL, MTA-TLS, DMARC & CSP Reporting
As outlined in the [Web Reporting and Monitoring](web/reporting-and-monitoring) and [Mail Reporting and Monitoring](e-mail/smtp-tls-and-dmarc-reporting) articles, you can also monitor multiple different things related to your web and mail security.