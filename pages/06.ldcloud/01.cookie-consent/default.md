---
title: 'Cookie Consent'
---

Nextcloud uses only necessary cookies out of the box. Thus, only a banner stating that should suffice. However, we don't want to modify the content itself. So we can configure a cookie consent page that loads if and only if a consent cookie is not set yet. All while the consent page itself does not set or require any cookies to load. That way, LD Cloud remains fully GDPR-compliant.

!!!! Also check out the `LD-Cloud.stpl` template in [Templates](../../server-setup/templates). Everything explained here and in the CDN section is implemented there as far as the nginx configuration is concerned. Just some some placeholders may need to be replaced.

## Cookie Consent Configuration
As outlined above, we can configure our web server to load a cookie consent page if and only if the consent cookie has not been set yet. This ensures that people that have already given their consent can load LD Cloud without any further messages while new visitors are kept informed and have a choice of their own whether they want to continue or not.

The following nginx configuration achieves that

```nginx
location / {
    if ($http_cookie ~* "consent" ) {
        proxy_pass https://{{backend_ip}}:{{backend_port}};
    }

    # using `return 302` directly doesn't work here
    try_files $uri/$arg_key @noconsent;
}

location @noconsent {
    return 302 "https://$host/cookieconsent";
}

# the page to display for asking for consent
location /cookieconsent {
    root   {{path_to_static_files_outside_nextcloud_dir}};
    try_files $uri/index.html $uri.html @fallback;
}

# path to the legal notice
location /impressum {
    root   {{path_to_static_files_outside_nextcloud_dir}};
    try_files $uri/index.html $uri.html @fallback;
}

# path to the privacy page
location /datenschutz {
    root   {{path_to_static_files_outside_nextcloud_dir}};
    try_files $uri/index.html $uri.html @fallback;
}

# path to call to give consent
location /datenschutz/consent {
    add_header Set-Cookie 'consent=true;Domain=$host;Path=/;Max-Age=7776000;SameSite=Strict;HTTPOnly;Secure';
    return 302 https://$host;
}
```

If you want to enable the option to revoke consent, you may add the following to your configuration:

```nginx
location /revoke {
    add_header Set-Cookie 'consent=true;Domain=$host;Path=/;Max-Age=-3600;SameSite=Strict;HTTPOnly;Secure';
	return 302 https://$host/cookieconsent;
}
```

## Proper Redirects
Using the above way of asking for consent results in any new visitor being redirected to the standard login page. That is even if he called a link to a public ressource. So, he'd have to call the link a second time to actually access it. But as powerful as nginx is, we can of course modify our configuration in a way the visitor is redirected to the resource he had initially called. To do so, modify the following parts accordingly:

```nginx
location @noconsent {
    return 302 "https://$host/cookieconsent?$request_uri";
}
```
This results in the requested resource being passed as an argument to the cookie consent page.

```nginx
location /cookieconsent {
    # the above parts have been cropped
    sub_filter_once off;
    sub_filter https://$host/datenschutz/consent https://$host/datenschutz/consent?$args;
    try_files $uri/index.html $uri.html @fallback;
}
```
This results in the arguments being passed as an argument to the consent link.

```nginx
location /datenschutz/consent {
    add_header Set-Cookie 'consent=true;Domain=$host;Path=/;Max-Age=7776000;SameSite=Strict;HTTPOnly;Secure';
    return 302 https://$host$args;
}
```
This results in the passed arguments being used as a path to call. The returned URL should be exactly the same as the initially called URL here. So, the visitor does not have to call the link a second time but gets redirected instead.