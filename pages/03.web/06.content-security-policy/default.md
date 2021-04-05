---
title: 'Content Security Policy'
---

Content Security Policy (CSP) is one of the central CORS (Cross-Origin Resource Sharing) policies utilized to ensure that only authorized resoirces are loaded. It sets which servers are authorized for sending which type of resources (e.g. fonts, scripts, CSS styles, etc.).

When using a CMS software, you should possibly avoid modifying this header and let the CMS set the right headers instead if possible. Otherwise, you should look into where which types of resources are loaded from and set the header accordingly as outlined in the [Mozilla MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy).

In nginx, you'll have to add the CSP header as follows:

```
add_header Content-Security-Policy "default-src 'self'";
```

!! Follow the principles as outlined in the [nginx Configuration](../nginx-configuration) article. It is highly advised to add the header in a separate nginx configuration file and include it at the end of the server section within the specific domain's configuration file.

!!! Make full use of reporting and add `report-to groupname` to the CSP header value. More on [Reporting & Monitoring](../reporting-and-monitoring).