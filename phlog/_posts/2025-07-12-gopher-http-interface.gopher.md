---
date: 2025-07-12
tags:
- sysadmin
- linux
- gopher
- http
- server
- debian
title: Providing an HTTP interface for your gopherhole

---


It's hard to convince people to use the Gopher protocol. So, why not host your own HTTP interface so people can browse your gopherhole via a website?

I've talked a lot with [sternenseemann](https://github.com/sternenseemann) over the years because of their Gopher software and this is another good piece of Gopher software written in Haskell by them.

## Installation

```
git clone https://github.com/sternenseemann/gopher-proxy
cd gopher-proxy
```

Then I removed the override for `attoparsec` in `default.nix` and ran `nix-build`.

Something was messed up with my symbolic link of `~/.nix-profile/` so I also ran this:

```
nix-env -iA nixpkgs.haskellPackages.gopher-proxy
```

Then I was able to access `gopher-proxy`.

## nginx config

I wanted to put the proxy behind nginx.

```
server {
    listen 8765;
    listen 8888 ssl;
    server_name gopher.someodd.zip;

    ssl_certificate /etc/letsencrypt/live/gopher.someodd.zip/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/gopher.someodd.zip/privkey.pem;

    location ^~ /.well-known/acme-challenge/ {
        default_type "text/plain";
        root /var/www/gopher.someodd.zip;
    }

    location / {
        proxy_pass http://localhost:8070;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

}
```

I had to comment-out the SSL stuff above before actually doing the HTTP
verification with certbot (place files in directory, don't spin up server):

```
sudo ln -s /etc/nginx/sites-available/gopher.someodd.zip.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
sudo certbot certonly --webroot-path="/var/www/gopher.someodd.zip" -d 'gopher.someodd.zip'
```

After that I uncommented the SSL paprts in the nginx config and restarted nginx.

## daemonize gopher-proxy

Normally you'd just `gopher-proxy --host gopher.someodd.zip --http-port 8070` but in our case we want it as an actual systemd service at `/etc/systemd/system/gopher-proxy.service`:

```
[Unit]
Description=Gopher Proxy Service
After=network.target

[Service]
Type=simple
User=baudrillard
ExecStart=/home/baudrillard/.nix-profile/bin/gopher-proxy --host gopher.someodd.zip --http-port 8070
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

then

```
sudo systemctl daemon-reload
sudo systemctl start gopher-proxy
sudo systemctl enable gopher-proxy
sudo systemctl status gopher-proxy
journalctl -u gopher-proxy
sudo journalctl -xeu gopher-proxy
```

## Test it/conclusion

Now visit your gopherhole via http. I have my proxy at same domain as my gopher subdomain this way people can even accidentally type `gopher.someodd.zip` and find this proxy! This to me isn't perfect, but it works.

Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/gopher-http-interface.gopher.txt](gopher://gopher.someodd.zip:70/0/phlog/gopher-http-interface.gopher.txt)
