---
layout: post
title: "Set up an IRC Server"
date: 2024-03-30
categories: notes
tags: debian irc
---

* TOC
{:toc}
My journey setting up a little IRC server, with SSL support and services, on Debian 12.

I feel like I went through a lot of struggles and research to get to the point of having a functional server. I tried various IRC daemons services, but ultimately decided to go with ngircd and atheme. I feel these are well supported, maintained and current. I also thought they were easier and simpler to understand and set up. An almost out-of-the-box experience.

This is the setup I use for [my IRC server](/showcase/irc-server). Try joining!

If you're struggling feel free to contact me using info from [my about page](/about).

# ngircd

I think ngircd is very simple and easy to work with.

## Basic ngircd installation and configuration

Install:

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install ngircd
```

Now edit `/etc/ngircd/ngircd.conf`. We will come back to this config file later, but there are some preliminary configurations you can test out and configure:

* `[Global]` settings
  * `Name`: I set mine to `irc.someodd.zip`
  * The admin info and email
  * `Listen`: You might want to set which IP addresses the server listens on. For example, I listen on `0.0.0.0`.
  * `MotdFile` (at `/etc/ngircd/motd.txt`)
  * `Ports`: I think actually resolved a connection issue for `atheme-services` by explicitly setting this to `6667`
  * `ServerGID` & `ServerUID`: I have these set to `irc` (user and group server runs under, be aware of file permissions to things ngircd needs to access)

Try `sudo service ngircd restart`.

A handy command for debugging ngircd is `sudo journalctl -xeu ngircd.service`.

## ngircd + LetsEncrypt (SSL)

Get SSL connections working on the IRC server by using LetsEncrypt to manage the certificate automatically. 

I was able to specify my icecast2 web root (selecting 2, place in web root when asked) since I already have that in order to do the ACME verification or whatever! Otherwise you might wanna do something like `sudo certbot certonly --standalone -d irc.someodd.zip` (which will start a web server for you).

```
sudo certbot certonly --webroot-path="/usr/share/icecast2/web" -d 'irc.someodd.zip'
```

Do a bunch of file management for ngircd's permissions sake, but also create a `dhparams.pem`:

```
sudo mkdir /etc/ngircd/ssl
sudo openssl dhparam -out /etc/letsencrypt/live/irc.someodd.zip/dhparams.pem 2048
sudo cp /etc/letsencrypt/live/irc.someodd.zip/fullchain.pem /etc/ngircd/ssl/
sudo cp /etc/letsencrypt/live/irc.someodd.zip/dhparams.pem /etc/ngircd/ssl/
sudo cp /etc/letsencrypt/live/irc.someodd.zip/privkey.pem /etc/ngircd/ssl/
sudo chown -R irc:irc /etc/ngircd/ssl/
```

Point to those files in `/etc/ngircd/ngircd.conf`, in the `[SSL]` section. In fact here's basically my entire `[SSL]` block, with comments removed:

```
[SSL]
        CertFile = /etc/ngircd/ssl/fullchain.pem
        CipherList = SECURE128:-VERS-SSL3.0
        DHFile = /etc/ngircd/ssl/dhparams.pem
        KeyFile = /etc/ngircd/ssl/privkey.pem
        Ports = 6697
```

Note I have SSL connections accepted on port 6697.

Allow connections to the configured SSL port with:

```
sudo ufw allow 6697/tcp comment 'SSL IRC'
```

Of course if you want to test external connections, do some port forwarding on your router. Finally, test it out by first restarting `ngircd`  (`sudo service ngircd restart`) and then connecting to your server on that port with SSL!

### Handling LetsEncrypt renewals

Certificates expire or something, I guess. Luckily LetsEncrypt makes automation fairly simple.

Edit `/etc/letsencrypt/renewal/irc.someodd.zip.conf`, so we can reload ngircd when the SSL certificate renews. Let's make a script and then use it as the post_hook in `[renewalparams]` in said config. Let's also configure `ii` to announce to a channel that the server is about to be restarted.

```
sudo apt-get update
sudo apt-get install ii
```

Create `/usr/local/bin/update_ngircd_ssl.sh` (don't forget to `sudo chmod +x /usr/local/bin/update_ngircd_ssl.sh`, and to configure the script for your needs):

```
#!/bin/bash

# Configuration variables
SERVER="127.0.0.1"
PORT=6667
NICK="Urgent"
FULLNAME="Urgent"
CHANNEL="#main"
MESSAGE="Attention all users: The IRC service will be restarted shortly for maintenance. Please expect a brief disconnection. If service does not resume normally please contact someodd@pm.me or @someodd@fosstodon.org."

# Directory where ii will create its server/channel structure
IIDIR="/tmp/ii_$$" # Using PID for uniqueness

# Start ii in the background
ii -s "$SERVER" -p "$PORT" -n "$NICK" -f "$FULLNAME" -i "$IIDIR" &

IIPID=$!

# Give ii a moment to connect and join the channel
sleep 5

# Send the message
echo "/j $CHANNEL" > "$IIDIR/$SERVER/in"  # Join the channel
sleep 2  # Wait a bit to ensure the join has been processed
echo "$MESSAGE" > "$IIDIR/$SERVER/$CHANNEL/in"  # Send the message

# Wait 1 hour to give users have a chance to see the message.
sleep 1h

# One last announcement from ii, now it's a five minutes heads up.
echo "$MESSAGE" > "$IIDIR/$SERVER/$CHANNEL/in"  # Send the message

# Wait 5 minutes to give users one last chance to see the message.
sleep 5m

# Clean up: kill ii and remove its directory
kill $IIPID
rm -rf "$IIDIR"

# The "actual" renewal work
cp /etc/letsencrypt/live/irc.someodd.zip/fullchain.pem /etc/ngircd/ssl/
cp /etc/letsencrypt/live/irc.someodd.zip/privkey.pem /etc/ngircd/ssl/
chown -R irc:irc /etc/ngircd/ssl/
service ngircd restart
```

Configure this in `/etc/letsencrypt/renewal/irc.someodd.zip.conf` under `[renewalparams]`:

```
post_hook = /usr/local/bin/update_ngircd_ssl.sh
```

Validate that the above command works correctly:

```
sudo certbot renew --dry-run
```

Please note, something I don't like about this script and would like to update in the future, is that I think it should use Atheme's global notice module.

# Atheme

I like Atheme because it seems to have an active community, basically works well out-of-the-box, and is relatively straightforward.

I strongly recommend reading [the official ngircd's services.txt for service configuration instructions](https://github.com/ngircd/ngircd/blob/master/doc/Services.txt).

Install Atheme:

```
sudo apt-get update
sudo apt-get install atheme-services
```

A handy command for debugging `atheme-services` is `sudo journalctl -xeu atheme-services.service`.

## Preparing ngircd for Atheme

Add a new `[Server]` block in `ngircd.conf` for Atheme:

```
[Server]
    Name = services.irc.someodd.zip
    Pass = abc 
    MyPassword = abc
    Type = Service
    SSLConnect = no
    MyPassword = abc
    PeerPassword = abc
    ServiceMask = *Serv
```

Caveat: in this server block, don't define host and port, because that'll make ngircd try to connect to said host/port. My mistake was thinking that I was using such to define what the block would listen on. You make Atheme just connect to your server like a regular client almost. I made this mistake and it lead to a lot of confusion. The idea is Atheme connects to ngircd, not the other way around!

Go ahead and `sudo service ngircd restart`.

## Actually configuring Atheme

Copy an example configuration to the expected config file location:

```
sudo cp /usr/share/doc/atheme-services/examples/atheme.conf.example /etc/atheme/atheme.conf
```

A few things you really should set in `atheme.conf`:

* Ensure `loadmodule "modules/protocol/ngircd";` is set (protocol module)
* In `serverinfo` ensure the `name` setting reflects the `Name` setting configured earlier when configuring the Atheme `[Server]` block in ngircd. For me that was  `name = "services.irc.someodd.zip";`
* Edit the uplink:
  * For me, because of my `ngircd` server name, I have `uplink "irc.someodd.zip" {`
  * Set `password` to `abc` (or whatever!) as reflected in the Atheme `[Server]` block made in `ngircd.conf` as shown earlier in this document.
  * Connect on port `6667` with `port = 6667` (non-SSL).

Now have fun and test out with `sudo service atheme-services restart`! Try to `/msg nickserv help`.

# Bonus

Stability, backup tips

## ZNC

Install:

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install znc
```

Run the configuration wizard:

```
znc --makeconf
```

Since I already have port 6697 for SSL on the main IRC server I'll use `6669`.

This just makes it run as the current user you have it set up as. I probably will make a better set up in the future, but this is fine right now. Also the SSL cert it generates is self-signed and doesn't use letsencrypt.

Let's make it start at startup:

```
sudo systemctl enable znc
```

And run:

```
sudo service znc start
```

UFW:

```
sudo ufw allow proto tcp from any to any port 6669 comment 'ZNC SSL'
```

You can access the web UI through the same port, but firefox will require setting `network.security.ports.banned.override` to the string value of `6669` in `about:config`. Also forward port on router.

Only edit `~/.znc/configs/znc.conf` if ZNC not running.

### SSL + more

Create the certificate or whatever (note I'm using the web root from [my Whisper Radio setup](/showcase/whisper-radio) but you may wanna just use the `--standalone` option instead; I selected *webroot* when prompted):

```bash
sudo certbot certonly --webroot-path="/usr/share/icecast2/web" -d 'znc.someodd.zip'
```

Copy the files, ensure ownership, create a directory (we will automate this with a hook, too):

```
mkdir ~/.znc/ssl
sudo cp /etc/letsencrypt/live/znc.someodd.zip/fullchain.pem /home/someuser/.znc/ssl/fullchain.pem
sudo cp /etc/letsencrypt/live/znc.someodd.zip/privkey.pem /home/someuser/.znc/ssl/privkey.pem
sudo chown -r someuser:someuser ~/.znc/ssl
```

Now in `~/.znc/configs/znc.conf` (make sure ZNC isn't running):

```
SSLCertFile = /home/someuser/.znc/ssl/fullchain.pem
SSLKeyFile = /home/someuser/.znc/ssl/privkey.pem
<Listener l>
        Port = 6669
        IPv4 = true
        IPv6 = true
        SSL = true
</Listener>
```

check service file see which user runs as or make znc run as a user...

in hex chat i have `znc.someodd.zip/6669` and default login method with my admin password and I mamke sure the username is the right username (don't use default info)

NEEDS TO USE HOOK POST HOOK FOR RENEWAL... LETSENCRYPT

### backing up

...
