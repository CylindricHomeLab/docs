---
title: 'CaddyServer and Cloudflare'
taxonomy:
    category:
        - Tech
    tag:
        - caddy
        - ssl
        - cloudflare
theme: learn2
---

CaddyServer usually forces all traffic to use SSL encryption - that's pretty much the main point of it. However, for the automatic LetsEncrypt certificate generation to work, the CaddyServer needs to be directly accessible from the internet, because that's how the ACME protocol confirms validity of the domain. 

===

This becomes a problem when the CaddyServer is put behind CloudFlare, for two reasons:

* the ACME servers can't actually reach CaddyServer to confirm the domain
* CloudFlare by default will talk to the backend servers using plain HTTP

The first will prevent CaddyServer from successfully requesting certificates, and the second will usually cause an infinite redirect loop, because Caddy will see unencrypted traffic coming in, send a `Redirect` to https://, so the browser reloads the page, but Caddy again only sees plain http, so repeats...

# The Solution
The solution consists of two elements, one at the CloudFlare end, and one at the CaddyServer end. We'll look at CaddyServer first...

## CaddyServer Configuration for CloudFlare
The main issue is using direct validation failing due to ACME hitting the CF servers and not the Caddy server. This is resolved by telling Caddy to use DNS validation instead. Because we're using CloudFlare, we can use the handy "dns-cloudflare" plugin. Download and install a version of CaddyServer including that plugin, for example:

``` curl https://getcaddy.com | bash -s personal tls.dns.cloudflare ```

Then configure caddy (`/etc/caddy/Caddyfile`) to use the CloudFlare plugin instead of the default process, by editing the site configuration:

```
wiki.cylindric.net {
    root /path/to/site
    tls email@example.com
    
    tls {
        dns cloudflare
    }
}
```

Next make sure that the CloudFlare credentials are avaliable to CaddyServer, which is done via environment variables.
Edit the `/etc/systemd/system/multi-user.target.wants/caddy.service` file and in the `[Service]` section add two Environment variables:

```
Environment=CLOUDFLARE_EMAIL=you_email@example.com
Environment=CLOUDFLARE_API_KEY=123456abcd123sadfs345sdge4t5fdsg34
```

Reload that config and then restart the service:
```
systemctl daemon-reload
service caddy restart
```

## CloudFlare Configuration for CaddyServer
* In the "DNS" page, simply make sure that the hostname for the website is being routed through the CloudFlare magic, so the little cloud icon should be orange.
* In the "Crypto" page, simply make sure that the SSL option is set to "Full (Strict)" which forces CloudFlare to talk to the CaddyServer using a secure channel.
