---
date: 2024-03-30
tags:
- linux
- sysadmin
title: Set up an IRC server

---


My journey setting up a little IRC server, with SSL support and services, on Debian 12.

I feel like I went through a lot of struggles and research to get to the point of having a functional server. I tried various IRC daemons services, but ultimately decided to go with ngircd and atheme. I feel these are well supported, maintained and current. I also thought they were easier and simpler to understand and set up. An almost out-of-the-box experience.

This is the setup I use for [my IRC server](/services/irc-server.md). Try joining!

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
  * Set an `[Oper]` block.
    * I have mine set to the same username I have registered with atheme services

Try `sudo service ngircd restart`.

Handy commands:

* A handy command for debugging ngircd is `sudo journalctl -xeu ngircd.service`.

* Reload config (including letsencrypt/ssl cert) without restarting service/interrupting:
  ```
  pgrep ngircd
  sudo kill -HUP <PID>
  ```



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

```

Configure this in `/etc/letsencrypt/renewal/irc.someodd.zip.conf` under `[renewalparams]`:

```
renew_hook = cp /etc/letsencrypt/live/irc.someodd.zip/fullchain.pem /etc/ngircd/ssl/ && cp /etc/letsencrypt/live/irc.someodd.zip/privkey.pem /etc/ngircd/ssl/ && chown -R irc:irc /etc/ngircd/ssl/ && kill -HUP $(pidof ngircd)
```

Validate that the above command works correctly:

```
sudo certbot renew --dry-run --cert-name irc.someodd.zip
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

## Make sure to back up the database!

My database, I think, was at `/var/lib/atheme/services.db`. I added it to the list of things that restic backs up.

## Known issues

**I think it's not saving nicknames/registration after restart? That's severe!** or maybe it's that i need to verify  by email for it to work right? i should disable that? i have to follow up on this. 

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

Honestly, somehow I'm just running it as my regular user so I just launch `znc` as that regular user.

You may want this in your nginx for znc if you setup ZNC:

```
    location ^~ /.well-known/acme-challenge/ {
        default_type "text/plain";
        root /var/www/znc.someodd.zip;
    }
```



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
sudo chown -R someuser:someuser ~/.znc/ssl
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

something is wrong with the ZNC configuration (nginx)

### ZNC user config, tips

To get messages on all devices (like if you have a laptop and a phone) and want to make sure you don't miss anything, even if you receive on one device:

```
/msg *controlpanel set AutoClearChanBuffer $me False
/msg *controlpanel set AutoClearQueryBuffer $me False
```

Maybe I should make this for all users by default!

switching...

/znc JumpNetwork netname

you can login using the default login message with username/pasword, but you can also specify the network to connect by default to make connecting to multiple networks easier:

Client configs:

* You can set an IRC password like `username/network:password` to connect to a specific network by default. This will let you have different ZNC connections (say one for someodd and one for libera.chat).
  * I think you can even set `username/network` as the username and just have your ZNC password as the password.

* [HexChat - ZNC](https://wiki.znc.in/HexChat)
  * Add your ZNC server’s address and port to the server list, like `znc.example.com/6667` for non-SSL connections or `znc.example.com/+6697` for SSL connections (prepend the port number with a `+` for SSL).
  * In the `Password` field, input your ZNC credentials in the format `username/network:password`. This tells HexChat how to log in to ZNC and specifies which of your ZNC-configured networks to connect to.

### Playback, Clientbuffer Modules

I feel this is kind of required in the modern times where you may have phone and laptop connected. You may want to have the same playback/buffer for all clients.

https://wiki.znc.in/Playback

https://wiki.znc.in/Clientbuffer

https://wiki.znc.in/Modules

https://wiki.znc.in/Compiling_modules


```
sudo apt-get install znc-buildmod
git clone https://github.com/jpnurmi/znc-playback
cd znc-playback
znc-buildmod playback.cpp
mkdir -p ~/.znc/modules
mv playback.so ~/.znc/modules
```

Then send `LoadMod playback` to `*status`.

Now `clientbuffer`:

```
git clone https://github.com/CyberShadow/znc-clientbuffer
cd znc-clientbuffer
znc-buildmod clientbuffer.cpp
mv clientbuffer.so ~/.znc/modules
```

Use the autoadd argument.

```
/msg *status Broadcast about to restart ZNC for maintenance. Please reconnect in a few minutes.
/msg *status SaveConfig

```
now `pkill znc` and `znc`.

now when i checked global modules, playback was set. `clientbuffer` seems to only show up in editnetwork.



It may be very handy to use the ClientBuffer module. In which case you can set an identifier either way:

* `username@identifier/network:password` as the password and your username as the username
* `username@identifier/network` as the username, and your password as the password

### tor

...

### tips

debug with:

znc -D

### backing up

...

## XMLRPC

Enable the httpd/XMLRPC in server. Here are some examples...

I had to read the source code and use chatgpt to figure this out.

```
curl -X POST -H 'Content-Type: text/xml' -d '<?xml version="1.0"?>
<methodCall>
   <methodName>atheme.login</methodName>
   <params>
      <param>
         <value><string>username</string></value>
      </param>
      <param>
         <value><string>password</string></value>
      </param>
   </params>
</methodCall>' http://localhost:8080/xmlrpc
```

ideally i want to access statistics without logging in.

finally found this? https://raw.githubusercontent.com/atheme/atheme/master/doc/XMLRPC

```
curl -X POST -H 'Content-Type: text/xml' -d '<?xml version="1.0"?>
<methodCall>
   <methodName>atheme.command</methodName>
   <params>
      <param><value><string>authcookie</string></value></param>
      <param><value><string>someodd</string></value></param>
      <param><value><string>sourceIP</string></value></param>
      <param><value><string>ChanServ</string></value></param>
      <param><value><string>info</string></value></param>
      <param><value><string>#channel</string></value></param>
   </params>
</methodCall>' http://localhost:8080/xmlrpc
```

maybe set up a dummy user that will do this.

## tor

...

## setup to share statistics

I think the easiest way to set up statistics haring is just to have nginx share a web root and then have a cron job run a script to echo the contents to the web root:

```
sudo apt-get install nginx

```

then edit `sudo vi /etc/nginx/sites-available/irc.someodd.zip.conf`  (we also want to set cors header or whatever so it's easier to fetch the site using javascript):

```
server {
    listen 8765;
    listen 8888 ssl;
    server_name irc.someodd.zip;
    root /var/www/irc.someodd.zip;

    ssl_certificate /etc/letsencrypt/live/irc.someodd.zip/cert.pem;
    ssl_certificate_key /etc/letsencrypt/live/irc.someodd.zip/privkey.pem;
    
    location /stats.json {
        # Add CORS headers
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'Origin, X-Requested-With, Content-Type, Accept';
        add_header 'Access-Control-Allow-Credentials' 'true';
    }

    location / {
        try_files $uri $uri/ =404;
    }
}
```

link the config to make it active

```
sudo ln -s /etc/nginx/sites-available/irc.someodd.zip.conf /etc/nginx/sites-enabled/
```

...

make the web root:

```
mkdir /var/www/irc.someodd.zip/
```

finally create this bash script to `/usr/local/bin/irc_stats.sh` with these contents, and you must install `ii` to get it to work: ...

```
#!/bin/bash

# Configuration variables
SERVER="127.0.0.1" # Change to your IRC server
PORT=6667          # Default IRC port
NICK="iistats"     # Change to your desired nickname
IIDIR="/tmp/ii_$$" # Temporary directory for ii's output

# Start ii in the background
ii -s "$SERVER" -p "$PORT" -n "$NICK" -f "ii User" -i "$IIDIR" >/dev/null 2>&1 &
IIPID=$!

# Allow some time for ii to connect
sleep 4

input=$(cat "$IIDIR/$SERVER/out")

# Parsing the information using grep and regex
number_of_services=$(echo "$input" | grep -oP '(?<=and )\d+(?= services on)' | head -1)
current_number_of_users=$(echo "$input" | grep -oP '(?<=I have )\d+(?= users,)' | head -1)
highest_connection_count=$(echo "$input" | grep -oP '(?<=Highest connection count: )\d+')
number_of_channels_formed=$(echo "$input" | grep -oP '\d+(?= channels formed)' | head -1)
operators_online=$(echo "$input" | grep -oP '\d+(?= operator\(s\) online)')

# ngIRCd uptime (days)
ngircd_pid=$(pgrep ngircd)
if [ -n "$ngircd_pid" ]; then
    ngircd_uptime_seconds=$(ps -p "$ngircd_pid" -o etimes= | tail -n 1 | tr -d ' ')
    ngircd_uptime=$(echo "$ngircd_uptime_seconds / 86400" | bc)
else
    ngircd_uptime="N/A"
fi

# Atheme uptime (days)
atheme_pid=$(pgrep atheme-services)
if [ -n "$atheme_pid" ]; then
    atheme_uptime_seconds=$(ps -p "$atheme_pid" -o etimes= | tail -n 1 | tr -d ' ')
    atheme_uptime=$(echo "$atheme_uptime_seconds / 86400" | bc)
else
    atheme_uptime="N/A"
fi

# output json
echo "{
  \"number of channels formed\": ${number_of_channels_formed:-0},
  \"number of services\": ${number_of_services:-0},
  \"current number of users\": ${current_number_of_users:-0},
  \"highest connection count\": ${highest_connection_count:-0},
  \"operators online\": ${operators_online:-0},
  \"ngircd uptime days\": \"$ngircd_uptime\",
  \"atheme uptime days\": \"$atheme_uptime\"
}"

# Clean up
kill $IIPID
rm -rf "$IIDIR"
```

chmod:

```
sudo chmod +x /usr/local/bin/irc_stats.sh
```

you can test it

```
sudo /usr/local/bin/irc_stats.sh > /var/www/irc.someodd.zip/stats.json
```

crontab it`crontab -e`:

```
*/30 * * * * /usr/local/bin/irc_stats.sh > /var/www/irc.someodd.zip/stats.json 2>&1
```

then

`sudo service nginx restart`

now change the web root path in the letsencrypt file...

ufw for nginx:

```
sudo ufw allow 8888/tcp comment 'general nginx https'
```

then you can just fetch this: https://irc.someodd.zip/stats.txt (port forward 443 -> 8888) for stats line-by-line it'll look like:

```
number of channels formed: 2
number of services: 8
current number of users: 5
highest connection count: 9
operators online: 1
ngircd uptime: 5 days
atheme uptime: 1 days
```

the example javascript:

```
fetch('https://irc.someodd.zip/stats.json')
  .then(response => response.json())
  .then(data => {
    // Parse the JSON object
    const numberOfChannelsFormed = data['number of channels formed'];
    const numberOfServices = data['number of services'];
    const currentNumberOfUsers = data['current number of users'];
    const highestConnectionCount = data['highest connection count'];
    const operatorsOnline = data['operators online'];
    const ngircdUptimeDays = data['ngircd uptime days'];
    const athemeUptimeDays = data['atheme uptime days'];

    // Display the parsed data
    console.log('Number of channels formed:', numberOfChannelsFormed);
    console.log('Number of services:', numberOfServices);
    console.log('Current number of users:', currentNumberOfUsers);
    console.log('Highest connection count:', highestConnectionCount);
    console.log('Operators online:', operatorsOnline);
    console.log('ngIRCd uptime days:', ngircdUptimeDays);
    console.log('Atheme uptime days:', athemeUptimeDays);
  })
  .catch(error => {
    console.error('Error fetching data:', error);
  });
```

you can see a demo of irc stats in action [here](/showcase/irc-server/#statistics)

gotta modify for letsencrypt `sudo vi /etc/letsencrypt/renewal/irc.someodd.zip.conf`:

```
webroot_path = /var/www/irc.someodd.zip,
```

you may also want to set the `[[webroot_map]]`

test with `sudo certbot renew --dry-run`

### ZNC BEHIND NGINX (stats continued)

this setup is kinda broken so use https://stackoverflow.com/questions/34236949/znc-on-a-subdomain-with-nginx-reverse-proxy ?

https://walkergriggs.com/2021/10/13/znc_the_right_way/

https://wiki.znc.in/Reverse_Proxy

edit the `~/.znc/configs/znc.conf`:

```
TrustedProxy = 127.0.0.1

<Listener listener1>
        AllowIRC = false
        AllowWeb = true
        IPv4 = true
        IPv6 = true
        Port = 6666
        SSL = false
        URIPrefix = /
</Listener>
```

you can safely shut down:

```
znc --quit
znc
```

HONESTLY DON'T EVEN NEED TO LISTNE ON 6666.. COULD JUST SSL TWICE.

for the sake of cerbot you may also want to add a config for znc, which you can use as an opportunity to use it through a better port, edit `/etc/nginx/sites-available/znc.someodd.zip.conf`:

```
server {
    listen      8888 ssl http2;
    listen 8765;
    server_name znc.someodd.zip;
    access_log  /var/log/nginx/irc.log combined;

    ssl_certificate /etc/letsencrypt/live/znc.someodd.zip/cert.pem;
    ssl_certificate_key /etc/letsencrypt/live/znc.someodd.zip/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:6666;
        proxy_set_header      Host             $host;
        proxy_set_header      X-Real-IP        $remote_addr;
        proxy_set_header      X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header      X-Client-Verify  SUCCESS;
        proxy_set_header      X-Client-DN      $ssl_client_s_dn;
        proxy_set_header      X-SSL-Subject    $ssl_client_s_dn;
        proxy_set_header      X-SSL-Issuer     $ssl_client_i_dn;
        proxy_read_timeout    1800;
        proxy_connect_timeout 1800;
    }
}

```

make the dir:

`sudo mkdir /var/www/znc.someodd.zip`

```
sudo ln -s /etc/nginx/sites-available/znc.someodd.zip.conf /etc/nginx/sites-enabled/
```

restart nginx.

you should be able to access znc on the regular https://znc.someodd.zip/

edit the znc letsencrypt:

```
webroot_path = /var/www/znc.someodd.zip,
[[webroot_map]]
znc.someodd.zip = /var/www/znc.someodd.zip
```

try:

```sudo certbot renew --dry-run 
sudo certbot renew --dry-run 
```


Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/irc-server.gopher.txt](gopher://gopher.someodd.zip:70/0/phlog/irc-server.gopher.txt)
