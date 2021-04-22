---
title: 'Security Headers'
---

To keep the visitors to the site safe and make their browsers block not required or not wanted resources and features, there are several response headers. In our view, it's just not right to patch privacy and security atop but there must be a basic foundation everything is built on instead. To measure how well that works, we are using the [Security Headers](https://securityheaders.com/) tool by Scott Helme.

## Vesta Template <a id="vesta"></a>
To achieve our goal, we may utilize a Vesta template for nginx that adds some security-related headers if not present. Here's a sample template:

```

####################################################################
# Sets some headers if not set. Don't modify this part if possible #
####################################################################
map $sent_http_permissions_policy $permissions {
    ""         "notifications=(self), push=(self), vibrate=(self), fullscreen=(self), sync-xhr=(self), speaker=(*), microphone=(), geolocation=(), camera=(), gyroscope=(), payment=(), interest-cohort=()";
    default    "${sent_http_permissions_policy}, interest-cohort=()";
}

# The Feature-Policy header is replaced by the above header in newer browsers
map $sent_http_feature_policy $features {
    ""         "notifications 'self'; push 'self'; vibrate 'self'; fullscreen 'self'; sync-xhr 'self'; speaker *; microphone 'none'; geolocation 'none'; camera 'none'; gyroscope 'none'; payment 'none'; interest-cohort 'none'";
    default    "";
}

map $sent_http_x_frame_options $x_frame {
    ""         "SAMEORIGIN";
    default    "";
}

map $sent_http_x_content_type_options $x_content_type {
    ""         "nosniff";
    default    "";
}

map $sent_http_x_xss_protection $x_xss {
    ""         "1; mode=block";
    default    "";
}

map $sent_http_referrer_policy $referrer_policy {
    ""         "strict-origin-when-cross-origin";
    default    "";
}
####################################################################
####################### Also see below notes #######################
####################################################################

server {
    listen      %ip%:%proxy_ssl_port% http2 ssl;
    listen      [::]:%proxy_ssl_port% http2 ssl;
    server_name %domain_idn% %alias_idn%;
    
    ssl_certificate      %ssl_pem%;
    ssl_certificate_key  %ssl_key%;
    error_log  /var/log/%web_system%/domains/%domain%.error.log error;

    location / {
        proxy_pass      https://%ip%:%web_ssl_port%;
        location ~* ^.+\.(%proxy_extentions%)$ {
            root           %sdocroot%;
            access_log     /var/log/%web_system%/domains/%domain%.log combined;
            access_log     /var/log/%web_system%/domains/%domain%.bytes bytes;
            expires        max;
            try_files      $uri $uri/index.html $uri.html @fallback;
        }
    }

    location /error/ {
        alias   %home%/%user%/web/%domain%/document_errors/;
    }

    location @fallback {
        proxy_pass      https://%ip%:%web_ssl_port%;
    }

    location ~ /\.ht    {return 404;}
    location ~ /\.svn/  {return 404;}
    location ~ /\.git/  {return 404;}
    location ~ /\.hg/   {return 404;}
    location ~ /\.bzr/  {return 404;}

    disable_symlinks if_not_owner from=%docroot%;

    include %home%/%user%/conf/web/snginx.%domain%.conf*;
    include %home%/%user%/conf/web/%domain%/*.conf;

    ############################################################################
    # comment these lines if headers are defined in the included files already #
	############################################################################
    add_header Permissions-Policy $permissions;                                #
    add_header Feature-Policy $features;                                       #
    add_header X-Frame-Options $x_frame;                                       #
    add_header X-Content-Type-Options $x_content_type;                         #
    add_header X-XSS-Protection $x_xss;                                        #
    add_header Referrer-Policy $referrer_policy;                               #
	############################################################################
}
```

Note that this will cause adding the `Feature-Policy` twice if it was already present as we are adding `interest-cohort=()`. The reason for insisting on disabling this feature is as Google has enabled browser-history based FLoC (Federated Learning of Cohorts) tracking by default and without consent for many Google Chrome users. Thus ethically speaking, it is only the right choice to disable this wherever possible. Also see [Am I FloCed (EFF)](https://amifloced.org/) on this.

The advantage of this setting is that it adds preset values for the headers if the headers are not present. If the headers are already present, it doesn't modify the set values with the above mentioned exception of the `Feature-Policy` header.

If you are already setting custom values for those same headers elsewhere, consider adding the `add_header` lines in that file and comment them above. Or just don't add those headers some other place and modify the values in the `map` instead. However, it's probably simplest to use the [Headers More module for nginx](https://github.com/openresty/headers-more-nginx-module#readme) and use e.g. `more_set_headers "Permissions-Policy: $permissions";` instead of `add_header Permissions-Policy $permissions;` in the template. This would also prevent adding the header twice.

## HTTP Strict Transport Security <a id="hsts"></a>
The HTTP Strict Transport Security (HSTS) header is sent as part of an HTTP response and indicates that the website and possibly subdomains should be called via HTTPS only. The header is added as follows:

`add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";`

This sets a proper `max-age`, includes subdomains and tells browsers that they may add the domain to the `preload` list. In fact, browsers do have a static preload list implemented and adding your domain to the list is as easy as entering the domain in the [HSTS Preload List Submission](https://hstspreload.org/) page. It may take a few months for all browsers including the domain in their preload list, but it will result in those browsers directly calling your site via HTTPS even if originally HTTP was requested by the user.

## CSP Header <a id="csp"></a>
Content Security Policy (CSP) is utilized to ensure that only authorized resources are loaded. It is perhaps one of the most important headers in terms of enhancing the security for your visitors. However, explicitly telling which resources should be permitted is quite hard to achieve. Especially when you are using a rather dynamic environment such as most CMS applications require.

When using a CMS software, you should possibly avoid modifying this header and let the CMS set the right headers instead if possible. But luckily, Scott Helme has also prepared a quite informative [Introduction to Content Securoty Policy](https://scotthelme.co.uk/content-security-policy-an-introduction/). Once you have finished reading this, go to "[Introducing the CSP Wizard on Report URI](https://scotthelme.co.uk/report-uri-csp-wizard/)" which will explain how you can set it up in a rather comfortable way.

!! Follow the principles as outlined in the [nginx Configuration](../nginx-configuration) article. It is highly advised to add the header in a separate nginx configuration file and include it at the end of the server section within the specific domain's configuration file.

!!! Make full use of reporting and add `report-to groupname` to the CSP header value. More on [Reporting & Monitoring](../reporting-and-monitoring).

## CORS Headers <a id="cors"></a>
Obviously, the above set headers are by far not the only headers you should add for enhanced security. Instead, also don't forget several CORS related headers. Or as Scott Helme says: "[COEP COOP CORP CORS CORB - CRAP](https://scotthelme.co.uk/coop-and-coep/)", and yes, I **CROP**ped the title slightly. Jokes aside, please read the linked article. It provides sufficient knowledge such that any further details here are unnecessary.

## Bottom Line
The bottom line is that enabling all those headers is quite useful in terms of ensuring both privacy and security. It certainly takes some time to get there, but it's definitely worth it to set up the basic settings ASAP and also implement the more advanced CSP and CORS policies over time.