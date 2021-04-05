---
title: 'nginx Configuration'
---

We are using nginx as a reverse proxy or rather a frontend server while Apache is running in the backend. That means that clients will connect to nginx only. So, most configuration will be handled within nginx configurations and possibly some within an `.htaccess` file.

While modifying the standard nginx configuration, a few matters are important:

* Always test your configuration with `nginx -t` instead of assuming "it should work"
* You must restart nginx with `service nginx restart` for the changes to take effect
* Do not change in the domain's configuration file itself if possible. Instead, use an existing template or create a new template and set the domain to use that template. Otherwise, Vesta may remove your modifications with an update.
* Never add headers directly within the domain's configuration file. Instead, write it in a separate configuration file and `include` it at the end of the server section within the domain's configuration file. Again, use templates for the includes if possible.

The main configuration can be found at `/etc/nginx/nginx.conf` and the domain-specific configuration can be found within the directory `/home/{{user}}/conf/web`.