---
date: 2024-04-22
tags:
- sysadmin
- linux
title: Set up an XMPP Server

---
Original content in gopherspace: gopher://gopher.someodd.zip:Just 7071/phlog/



XMPP is...

i have an invite-only xmpp server...

I think I like it more than IRC or Matrix because...

I'm chosing to use Prosody in Debian because...

I'm hosting my own XMPP server...

The setup I want is e2ee, offline...

* TOC
{:toc}


# Basic setup

HOLD UP IT MAY BE BAD THAT I'M INSTALLING SOME OF THESE THINGS I THINK ARE DEPENDS, LIKE REGISTER_WEB!

plugins:

* https://modules.prosody.im/mod_invites_page
* https://prosody.im/doc/modules/mod_invites
* https://prosody.im/doc/modules/mod_invites_register
* https://prosody.im/doc/modules/mod_muc

```
sudo apt-get install prosody luarocks libjs-bootstrap4 libjs-jquery liblua5.4-dev -y
sudo prosodyctl install --server=https://modules.prosody.im/rocks/ mod_conversejs
sudo prosodyctl install --server=https://modules.prosody.im/rocks/ mod_invites_api
sudo prosodyctl install --server=https://modules.prosody.im/rocks/ mod_register_apps
sudo prosodyctl install --server=https://modules.prosody.im/rocks/ mod_register_web
sudo prosodyctl install --server=https://modules.prosody.im/rocks/ mod_invites_page
sudo prosodyctl install --server=https://modules.prosody.im/rocks/ mod_invites_register_web
sudo prosodyctl install --server=https://modules.prosody.im/rocks/ mod_http_libjs
sudo vi /etc/prosody/prosody.cfg.lua
```

Now that the config file should be open:

* Change `VirtualHost` to your domain you want the XMPP service to be accessible through.

* add this to the end of the config and make sure the domain you use has a valid ssl cert for it (files expire after a week!) also [note this important docs on how to change things like quotas, size limits, etc](https://prosody.im/doc/modules/mod_http_file_share):

  * ```
    Component "upload.xmpp.someodd.zip" "http_file_share"
    http_file_share_size_limit = 25*1024*1024
    ```

* edit `modules_enabled`:

  * i'm thinking about uncommenting `http_openmetrics`

  * `offline` is auto-loaded! I love that.

  * enable `mam`

  * i enabled `websocket` for web client?

  * note of the log locations. mine are

    * `/var/log/prosody/prosody.log`
    * `/var/log/prosody/prosody.err`

  * note the certificate location... mine just said `certs`, I believe that's `/etc/prosody/certs`

  * there's something about enabling statistics too

  * definitely set this:

    * `admins = { "someodd@xmpp.someodd.zip" }`

  * careful with registration:

    * https://prosody.im/doc/creating_accounts -- you may want to read this.
    * `allow_registration = true` is supposedly dangerous.
      * "Servers with unrestricted registration are open to abuse and provide an easy method for spammers to get onto the XMPP network. If you do not control this, your server may be blocked by other servers on the network."
      * https://prosody.im/doc/public_servers
    * create users, especially your admin account, using this:
      * `sudo prosodyctl adduser someodd@xmpp.someodd.zip`
    * manage invites cli:
      * `sudo prosodyctl mod_invites generate xmpp.someodd.zip` it'll then give you a URL
    * check out this module for invitation page: https://modules.prosody.im/mod_invites_page
      * enable the `"invites_page"`module in `/etc/prosody/prosody.cfg.lua`
      * `invites_page = "https://xmpp.someodd.zip/invites_page?{invite.token}"` in top level scope
      * create an invnite with `sudo prosodyctl mod_invites generate xmpp.someodd.zip` or you may be able to generate invites in your client
    * add `http_libjs` to modules

  * i decided to enable BOSH. I think this supposedly this helps with people who have unreliabel connectsions, or say are using XMPP from a phone.

  * enable chat groups:

    * i may want to set `restrict_room_creation = local`

    * add these (use a real domain) (not sure if I need the `modules_enabled` bit:

      ```
      Component "conference.xmpp.someodd.zip" "muc"
          name = "Prosody Chatrooms"
          restrict_room_creation = "local"
          modules_enabled = { "muc_mam" }
      ```

      

A part of my enabled modules section looks like:

```
        -- installed plugins
                "http_libjs";             -- depend for invites_page
                "invites_api";            -- depend for invites_page
                "register_apps";          -- depend for invites_page
                "register_web";           -- depend for invites_page?
                "invites_register_web";   -- depend for invites_page?
                "invites_page";           -- Adds a web page to create and manage invites
                "conversejs";             -- Adds a web page chat
```

When you search rooms/groupchats in conversejs you search `xmpp.someodd.zip`. In dino i created a room through `conference.xmpp.someodd.zip` idk.


## Setup SSL + Basic nginx

There's an interesting bit in the config:

> -- For more information, including how to use 'prosodyctl' to auto-import certificates
> -- (from e.g. Let's Encrypt) see https://prosody.im/doc/certificates

This link is useful: https://prosody.im/doc/letsencrypt

Instructions on using LetsEncrypt and a basic nginx setup for the SSL setup. nginx for the ACME request and for a web interface (later).

You can check my [#nginx tag](/tags/nginx) for more info, but this config in `/etc/nginx/sites-available/xmpp.someodd.zip.conf` (uncomment the SSL lines once we've set up LetsEncrypt):

```
server {
    listen 8765;
    #listen 8888 ssl;
    server_name xmpp.someodd.zip;
    root /var/www/xmpp.someodd.zip;

    #ssl_certificate /etc/letsencrypt/live/xmpp.someodd.zip/cert.pem;
    #ssl_certificate_key /etc/letsencrypt/live/xmpp.someodd.zip/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:5280;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Frame-Options SAMEORIGIN;  # Security header
    }

    location ^~ /.well-known/acme-challenge/ {
        default_type "text/plain";
        root /var/www/xmpp.someodd.zip;
    }

}

```

Although maybe in future I'd want to redirect port 5281 and 5280 to just 443 and 80.

Then:

```
mkdir /var/www/xmpp.someodd.zip
sudo ln -s /etc/nginx/sites-available/xmpp.someodd.zip.conf /etc/nginx/sites-enabled/
sudo service nginx restart
sudo certbot certonly --webroot-path="/var/www/xmpp.someodd.zip" -d 'xmpp.someodd.zip'
```

Select web root. Now go back and uncomment those SSL-related lines in the nginx config above and restart nginx.

Now that we have the certificates, link them to the certificate directory defined in the prosody config **make sure to do this for all prosody-related domains**:

```
sudo prosodyctl --root cert import /etc/letsencrypt/live
```

Edit `/etc/letsencrypt/renewal/xmpp.someodd.zip.conf` and add the following to `renewalparams`:

```
deploy_hook = /usr/bin/prosodyctl --root cert import /etc/letsencrypt/live
```

I will probably update this article later to use a system-wide notification about the service going down for maintenance. Test out renewing:

```
sudo certbot renew --dry-run --cert-name xmpp.someodd.zip
```

Not done, just now do the same thing but for `upload.xmpp.someodd.zip`.

here's the general idea with the `/etc/letsencrypt/renewal/xmpp.someodd.zip.conf` file, please note the `deploy_hook` which restarts prosody after renewal:

```
# renew_before_expiry = 30 days
version = 2.1.0
archive_dir = /etc/letsencrypt/archive/xmpp.someodd.zip
cert = /etc/letsencrypt/live/xmpp.someodd.zip/cert.pem
privkey = /etc/letsencrypt/live/xmpp.someodd.zip/privkey.pem
chain = /etc/letsencrypt/live/xmpp.someodd.zip/chain.pem
fullchain = /etc/letsencrypt/live/xmpp.someodd.zip/fullchain.pem

# Options used in the renewal process
[renewalparams]
account = redacted
authenticator = webroot
webroot_path = /var/www/xmpp.someodd.zip,
server = https://acme-v02.api.letsencrypt.org/directory
key_type = ecdsa
deploy_hook = prosodyctl --root cert import /etc/letsencrypt/live
[[webroot_map]]
xmpp.someodd.zip = /var/www/xmpp.someodd.zip
```

don't forget to uncomment/edit ssl_certificate* fields in nginx then restart nginx

**I THINK YOU'LL WANT TO ALSO DO THAT FOR irc.xmpp.someodd.zip** AND conference.xmpp.someodd.zip if you're going to do biboumi/irc connection. except i don't think you need the ssl or location / block for those.

## Test what we have so far + firewall

```
sudo prosodyctl check
sudo systemctl restart prosody
sudo service nginx restart
sudo ufw allow 5222/tcp comment 'Allow XMPP client connections (c2s)'
sudo ufw allow 5269/tcp comment 'Allow XMPP server connections (s2s)'
sudo ufw allow 5280/tcp comment 'Allow HTTP connections (BOSH/WebSocket)'
sudo ufw allow 5281/tcp comment 'Allow HTTPS connections (Secure BOSH/WebSocket)'
```

Now you'll want to do some forwarding on your router.

Try connecting with `dino-im`. `sudo apt-get install dino-im` using the user like `someodd@xmpp.someodd.zip`.

This UFW/firewall config isn't ideal because it exposes some web interface stuff on the weird port...

**NOW TEST OUT THE INVITES PAGE MODULE (CURRENTLY BROKEN/cant figure out)**

test out conversejs module: https://xmpp.someodd.zip/conversejs

handy commands:

* `sudo systemctl restart prosody`
* `sudo prosodyctl check`

# Backups

Add `/var/lib/prosody/` to backups.

# Caveats

If someone can't join your groups or IRC in XMPP you should be aware that a S2S error with SSL verification may be failing. You need to have separate letsencrypt setups for all the xmpp subdomains basically. For instance `irc.xmpp.someodd.zip` and `conference.xmpp.someodd.zip`.

# Coming soon

* pubsub?
* maybe i need certs for irc.xmpp.someodd.zip and conference.xmpp.someodd.zip
* specific public, anonymous mucs, maybe.
  * I want to make it so I can provide people a conversejs anonymous/public access to #main on my irc server
* It turns out I do have another question: if I enter a key/password for a channel and I have persistence turned on, I'm not going to have a nasty situation where others can now access the channel without the key/password, am I?  Like, how about if someone uses the same generic name I used to access the password-protected channel?
* Public Statistics: https://prosody.im/doc/modules/mod_http_openmetrics -- [http_openmetrics](https://prosody.im/doc/modules/mod_http_openmetrics) is built in!

# Bonus: link to IRC

First look at:

* https://doc.biboumi.louiz.org/admin.html#configuration
* https://prosody.im/doc/components#adding_an_external_component

We need to configure Prosody to add an external component. Edit the config `/etc/prosody/prosody.cfg.lua` (put it in the components section):

```
Component "irc.xmpp.someodd.zip"
    component_secret = "secret_password"
```

need to use a real domain by the way if you want people from other servers to use it.

```
sudo apt-get install biboumi
sudo mkdir -p /var/lib/biboumi/.config/biboumi/
sudo chown -R _biboumi:_biboumi /var/lib/biboumi/
sudo chmod -R 750 /var/lib/biboumi/
```

edit `/etc/biboumi/biboumi.cfg`:

```
hostname=irc.xmpp.someodd.zip
password=secret_password
xmpp_server_ip=127.0.0.1
port=5347
admin=someodd@xmpp.someodd.zip
realname_customization=true
realname_from_jid=false
ca_file=
outgoing_bind=
log_level=3
fixed_irc_server=irc.someodd.zip
persistent_by_default=true
```

`fixed_irc_server=irc.someodd.zip` makes it so only that server is accessible via the XMPP-IRC gateway.

maybe `persistent_by_default` is a bad idea for security and I think a user can mark a connection as persistent on their end, so they don't miss messages?

you'll also want to enable `mod_muc` for group chats maybe in the component?

* `sudo journalctl -xeu biboumi.service`
* https://doc.biboumi.louiz.org/9.0/user.html
* https://doc.biboumi.louiz.org/admin.html
* `sudo service biboumi start`

Add `/var/lib/biboumi/.config/biboumi/` to backups.

# Bonus: anonymous usage of chat room? but don't allow c2c?

...

## See also

* https://metajack.im/2008/07/02/xmpp-is-better-with-bosh/
* https://doc.biboumi.louiz.org/9.0/user.html
* https://shadowkat.net/blog/30.html
* https://prosody.im/doc/modules/mod_muc
* https://joinjabber.org/
* https://prosody.im/doc/jingle#server_support -- calling and video support

