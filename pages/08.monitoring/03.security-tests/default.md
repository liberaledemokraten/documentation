---
title: 'Security Tests'
---

The here suggested protocols, ciphers, versions, etc. are not set in stone and the recommended ones may be subjected to change over time. Older ones may become obsolete or insecure and should not be used anymore if the case.

But staying up to date about the newest standards and vulnaribilities is not always as easy. To ease things up, any services on the server may be tested against the following services. In fact, the server should be checked against the prior two regularly while the other tests may be done with the initial configuration and otherwise occassionally.

## Qualys SSL Labs
With [SSL Labs](https://www.ssllabs.com/) you may test the server against common vulnarabilities and whether the server is configured properly as per today's standards. But remember that this only checks the transport-level encryption and some other related matters. The aim should be to achieve a grade of A or A+.

## Security Headers
[Security Headers](https://securityheaders.com/) is a service of Scott Helme who is also the guy behind report-uri.com. It will look into the HTTP reponse and the headers specifically. It evaluates and rates the headers with modern secure and good practices in mind. The aim should be to achieve a grade of A.

## MTA-STS Validator
The [MTA-STS Validator](https://aykevl.nl/apps/mta-sts/) validates the MTA-STS configuration of a domain as the name suggests. The aim should be to pass the test without any warnings.

## ImmuniWeb Security Test
The [ImmuniWeb Security Test](https://www.immuniweb.com/websec/) is a test that combines the perks of the SSL Labs and Security Headers tests. Additionally, it evaluates sites by looking at whether a CMS footprint is left and whether the site possibly meets some US and EU standards. The aim should be to achieve an A grade.

## Hardenize
[Hardenize](https://www.hardenize.com/) is a great tool evaluating both web and email server security. It also looks into the DNS for DNSSEC. The aim should be to pass all tests without any critical warning. Using DNSSEC is not a must for now until it has become a standard that is widely utilized by registrars too.

## Mozilla Observatory
[Mozilla Observatory](https://observatory.mozilla.org/) is yet another great tool to evaluate your server including the SSH server even. It is a little more strict with evaluations than all the other services above and as such, receiving an A is very hard to achieve. The aim here should be to achieve a B grade or better.