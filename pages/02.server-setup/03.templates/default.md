---
title: 'Vesta Templates'
media_order: 'http2ipv6.stpl.txt,http2ipv6.tpl.txt,LD-Cloud.stpl.txt,LD-Cloud.tpl.txt'
aura:
    pagetype: website
    description: ''
    image: ''
taxonomy:
    category:
        - vesta
    tag:
        - vesta
        - template
---

## General Information
All templates for VestaCP can be found under  `/usr/local/vesta/data/templates`. This includes templates for Apache httpd, nginx, bind and the skeleton files set per default whenever a new webdomain is added. If using a different platform, the path may vary.

## General nginx Template
The `http2ipv6.tpl` and `http2ipv6.stpl` templates per default force the visitor to use HTTPS. It also enables HTTP/2 and the server to receive requests over IPv6 if your server supports it and your firewall does not block them. Feel free to use them instead of the default templates.

## Apache httpd Template
The templates for Apache httpd may be found in `/usr/local/vesta/data/templates/web/apache2`. At least whenever using a different PHP version, consider changing the template.

## nginx Template for the LD CLoud
In the [LD Cloud](../../ldcloud) article, we have shown how to set up the nginx web or reverse proxy server to

1. Place a Cookie Consent Page
2. Revoke your consent
3. Set up a CDNized Nextcloud instance

The files `ldcloud.tpl` and `ldcloud.stpl` are template configuration files containing the configuration for all the points. You can use them as a base after having set up your Nextcloud instance. But keep the following prequisites in mind:

1. You are using nginx as web server or reverse proxy
2. Your Nextcloud instance is fully functional, set up and running
3. Appropriate files for the paths `cookieconsent`, `datenschutz` and `impressum` are set up
4. Your CDN is up an running in a way that it is able to deliver content
5. The LD-Cloud template has not pre-set the appropriate CDN Domain yet. You must modify those parts

Once all above criteria are fulfilled, you can use the TPL template for HTTP and STPL template for HTTPS. That will result in HTTP being redirected to HTTPS.

!!! if you are using a server management software supporting such templates (e.g. VestaCP and forks thereof), you can add the template files under `/usr/local/vesta/data/templates/web/nginx` after having modified it in a way that it uses your CDN domain.

## Sample Templates
Here are some templates as text files:

* [http2ipv6.tpl](http2ipv6.tpl.txt) & [http2ipv6.stpl](http2ipv6.stpl.txt)
* [lD-Cloud.tpl](LD-Cloud.tpl.txt) & [LD-Cloud.stpl](LD-Cloud.stpl.txt)