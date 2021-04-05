---
title: 'Testing Sites'
---

Before going live with your actual domain name, you might want to test your web site first. This will be the case especially if the website is already in active use and you don't want to modify things before being sure that it works well. In that case, you can follow one of the two methods described here.

## Method 1: Reverse Proxy
We are using nginx as a reverse proxy and can take full advantage of its proxying features. This means that we can create a new subdomain for calling our website to test to, e.g. `proxy.ld-online.net`. We then configure nginx to proxy any requests to our website to test. To do so, we have to set the `Host` header accordingly and `proxy_pass` to the IP address of the server the website to test is hosted on. The nginx configuration may look like this:

```
location / {
    proxy_pass https://duckduckgo.com;
    proxy_set_header Host duckduckgo.com;
    proxy_set_header X-Forwarded-For $remote_addr;
}
```

another alternative way is as shown below:

```
upstream ddg {
    server 52.149.246.39:443;
    server duckduckgo.com:443;
}

server {
    listen 127.0.0.1:443 http2 ssl;
    server_name proxy.ld-online.net ;
    ssl_certificate /path/to/ssl.proxy.ld-online.net.pem;
    ssl_certificate_key /path/to/ssl.proxy.ld-online.net.key;

    location / {
        proxy_pass https://ddg;
        proxy_set_header Host duckduckgo.com;
        proxy_set_header X-Forwarded-For $remote_addr;
    }
}
```

### Vesta CP & nginx Reverse Proxy
Of course, we can test on our server right away. If the (sub)domain that is to be tested (e.g. `sub.example.com`) is yet not hosted on the server, you may log into Vesta CP and add the domain. Thenafter, upload and configure the site as you would. Now, use the following configuration for `proxy.ld-online.net.nginx.ssl.conf`:

```
location / {
    proxy_pass https://$server_addr;
    proxy_set_header Host sub.example.com;
    proxy_set_header X-Forwarded-For $remote_addr;
}
```

## Method 2: Local DNS
You can of course test your site without using the nginx reverse proxy. This will be a little more difficult to achieve and comes across some hardships, but it has the advantage that not `proxy.ld-online.net` is shown in the address bar, but `sub.example.com` (or rather the actual domain name you wanted to test with). 

!!! NOTE: This means that SSL/TLS will not work with a trusted certificate unless you copy the private and public certificates from the live server.

The first step here is to get the test server's IP address (e.g. 127.0.0.1) and host your test site there. Secondly, set up a recursive DNS server in your local network and configure your router or OS to send DNS requests to it instead of to public DNS servers. Thenafter, add a static entry `A sub.example.com. IN 127.0.0.1`, meaning that the domain `sub.example.com` can be found under the IPv4 address 127.0.0.1.

Now, your browser will send HTTP requests for that domain to 127.0.0.1 and you'll see the website you wanted to test whenever you call http://sub.example.com (same with HTTPS of course, if HTTPS is configured for the domain in the test server). Everyone else that is not using your DNS server will still send their requests to the actual server, so nobody else can see the website you want to test this way.