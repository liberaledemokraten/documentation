---
title: 'Reporting & Monitoring'
---

When managing a server, receiving reports on issues is very important for both maintainance and troubleshooting. There are basically two different methods of receiving such reports with regards to E-Mails.

## SMTP TLS Reporting
SMTP TLS Reporting (TLS-RPT) is an under [RFC 8460](https://tools.ietf.org/html/rfc8460) standardized tool that enables reporting of TLS connectivity problems experienced by applications that send email. The diagnostic reporting supports server maintainers with monitoring and troubleshooting of connectivity issues. It is set with a DNS entry as follows:

`_smtp._tls TXT IN "v=TLSRPTv1;rua=mailto:tls-rpt@example.com"`

If you want the mails to be sent to a tool which visualizes any reported issues, consider using Scott Helme's [report-uri.com](https://report-uri.com).

## DMARC Reporting
DMARC (Domain-based Message Authentication, Reporting, and Conformance) is a tool to protect organizations from spoofed emails being sent from unauthorized mail servers. With DMARC you can tell the receiver what it should do with emails it receives from unauthorized servers. It is standardized in [RFC 7489](https://tools.ietf.org/html/rfc7489).

SPF, DKIM and DMARC are set by default when enabling the use of our DNS server. However, you may modify the policy to harden it. For example, the following DNS entry is recommended to do so:

`_dmarc TXT IN "v=DMARC1; p=reject"`

This states that all unathorized mails shall be rejected. Alternatively, one can also tell the receiver to `p=quarantine` them, thus label it as suspicious (possibly spam). But of course, rejecting or quarantining emails is not the main topic in this article, but for us to be able to receive reports. To do so, go for the following sample DNS entry:

`_dmarc TXT IN "v=DMARC1; p=quarantine;  rua=mailto:dmarc@example.com"`

This way, any such reports will be emailed to the entered email address. Here too, you may use report-uri.