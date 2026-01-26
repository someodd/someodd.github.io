---
date: 2025-03-08
tags:
- mumble
- linux
title: How to set up a Mumble server

---


Mumble is an old-school, low-latency voice chat app, perfect for games like
Counter-Strike 1.6. It’s a bit tricky to set up but offers strong encryption
and full control. My server provides a lightweight, private alternative for
high-performance voice chat.

[I run a Mumble server!](gopher://gopher.someodd.zip:70/0/services/mumble.md)

I set this up on my Debian server.

## Basic setup

Install and configure:

```
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install mumble-server
sudo dpkg-reconfigure mumble-server
```

Confirm to start the server on boot and also set your `SuperUser` password, which will be used to administer the server (you log in with the username `SuperUser`).

The config file is `/etc/mumble-server.ini`. A fun thing to tweak is `welcometext`.

## Firewall

The default port is 64738.

```
sudo ufw allow 64738/tcp comment "Allow Mumble server (default port TCP)"
sudo ufw allow 64738/udp comment "Mumble voice traffic"
```

Don't forget to port forward!

## LetsEncrypt/SSL

Mumble offers a self-signed certificate by default, but I'm a little bit extra and want to have my own SSL certificate just for Mumble.


Create `sites-available/mumble.someodd.zip.conf`:

```
server {
    listen 8765;
    server_name mumble.someodd.zip;
    root /var/www/mumble.someodd.zip;

    location ^~ /.well-known/acme-challenge/ {
        default_type "text/plain";
        root /var/www/mumble.someodd.zip;
    }

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Now "enable" the configuration and restart `nginx`:

```
mkdir /var/www/mumble.someodd.zip
sudo ln -s /etc/nginx/sites-available/mumble.someodd.zip.conf /etc/nginx/sites-enabled/
sudo service nginx restart
```

Finally, since the point of setting up this `nginx` virtual host was to allow for a webroot during the LetsEncrypt renewal process, let's actually start working on LetsEncrypt.

Create the new cert for `mumble.someodd.zip`. Copy the key and cert to a custom directory for `mumble-server`. I did this to avoid permission complications.

```
sudo certbot certonly --webroot-path="/var/www/mumble.someodd.zip" -d 'mumble.someodd.zip'
sudo mkdir /etc/mumble-server/
sudo cp /etc/letsencrypt/live/mumble.someodd.zip/fullchain.pem /etc/mumble-server/
```

Set decent security permissions/ensure the files are readable by `mumble-server`:

```
sudo chown root:mumble-server /etc/mumble-server
sudo chown root:mumble-server /etc/mumble-server/privkey.pem
sudo chown root:mumble-server /etc/mumble-server/fullchain.pem
sudo chmod 750 /etc/mumble-server
sudo chmod 640 /etc/mumble-server/privkey.pem
sudo chmod 640 /etc/mumble-server/fullchain.pem
```

Point `/etc/mumble-server.ini` to the SSL files:

```
sslCert=/etc/mumble-server/fullchain.pem
sslKey=/etc/mumble-server/privkey.pem
```

Update letsencrypt `/etc/letsencrypt/renewal/mumble.someodd.zip.conf` by
putting this under renewalparams:

```
renew_hook = cp /etc/letsencrypt/live/mumble.someodd.zip/privkey.pem /etc/mumble-server/privkey.pem && cp /etc/letsencrypt/live/mumble.someodd.zip/fullchain.pem /etc/mumble-server/fullchain.pem && chown root:mumble-server /etc/mumble-server/*.pem && chmod 640 /etc/mumble-server/*.pem && systemctl restart mumble-server
```

Test it out with:

```
sudo certbot renew --cert-name mumble.someodd.zip --dry-run
```

Although since this is fresh why not just test with:

```
sudo certbot renew --cert-name mumble.someodd.zip --force-renewal
```

Original content in gopherspace: [gopher://gopher.someodd.zip:70/1/phlog/mumble-server.gopher.txt](gopher://gopher.someodd.zip:70/1/phlog/mumble-server.gopher.txt)
