---
title: 'HTTP Strict Transport Security'
---

The HTTP Strict Transport Security (HSTS) header is sent as part of an HTTP response and indicates that the website and possibly subdomains should be called via HTTPS only. The header is added as follows:

`add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";`

This sets a proper max-age, includes subdomains and tells browsers that they may add the domain to the preload list. In fact, browsers do have a static preload list implemented and adding your domain to the list is as easy as entering the domain in the [HSTS Preload List Submission](https://hstspreload.org/) page. It may take a few months for all browsers including the domain in their preload list, but it will result in those browsers directly calling your site via HTTPS even if originally HTTP was requested by the user.