---
date: 2025-03-02
tags:
- sysadmin
- linux
title: 'LetsEncrypt renewal hooks: I was doing it all wrong!'

---


I kept saying `deploy_hook` and `post_hook`, but no it's `renew_hook`, and if
there's unrecognized entry it'll just remove it! I have no idea how I made this mistake...

It should look like this:

```
[renewalparams]
account = redacted
authenticator = webroot
webroot_path = /var/www/xmpp.someodd.zip, 
server = https://acme-v02.api.letsencrypt.org/directory
key_type = ecdsa
renew_hook = prosodyctl --root cert import /etc/letsencrypt/live
```

Use this command now, to test:

```
sudo certbot renew --cert-name irc.xmpp.someodd.zip --force-renewal
```

I have to go through and update all my certs now...

What's frustrating is it'll fail silently and just remove the line from the config. Thanks to this thread: https://community.letsencrypt.org/t/certbot-renew-overwriting-renewal-file/119107/2

Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/letsencrypt-renewal-hook-mistake.gopher.txt](gopher://gopher.someodd.zip:70/0/phlog/letsencrypt-renewal-hook-mistake.gopher.txt)
