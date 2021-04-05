---
title: 'TLS Security'
---

! The used protocols, ciphers and curves here are not set in stone. They may be subject to change if they happen to be deemed weak. Always use an ordered preference from the safest to the weakest and never support weak ones unless absolutely necessary for compatibility with older clients. Have a look at [cipherlist.eu](https://cipherlist.eu) and test your configuration with [ssllabs.com](https://ssllabs.com/ssltest).

!!! After each modification of the nginx configuration, test it by running `nginx -t`. Later on, load the new configuration by restarting nginx via `service nginx restart`.

Check the SSL parts in `etc/nginx/nginx.conf`. It should look similar to the following:

```nginx
ssl_session_timeout 10m;
ssl_session_cache   shared:SSL:10m;
ssl_session_tickets off;

ssl_protocols   TLSv1.3 TLSv1.2;
ssl_prefer_server_ciphers on;
ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:DHE-RSA-ARIA256-GCM-SHA384:DHE-RSA-ARIA128-GCM-SHA256:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-RSA-AES256-SHA:!aNull:!eNull:!EXPORT:!DES:!MD5:!PSK:!RC4";
# for TLSv1 enable the following only, if v1 is needed
# ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA

ssl_dhparam /etc/nginx/dhparam.pem; # openssl dhparam -out /etc/nginx/dhparam.pem 4096
ssl_ecdh_curve X448:X25519:secp521r1:secp384r1:prime256v1;

ssl_stapling on;
ssl_stapling_verify on;
resolver 1.1.1.1 8.8.8.8 valid=300s;
resolver_timeout 5s;
```

Don't forget to run `openssl dhparam -out /etc/nginx/dhparam.pem 4096` once after having set up the nginx configuration as outlined above.

Within the domain specific configuration, you may additionally add the following:

```
# verify chain of trust of OCSP response using Root CA and Intermediate certs
ssl_trusted_certificate /path/to/root_CA_cert_plus_intermediates;
```