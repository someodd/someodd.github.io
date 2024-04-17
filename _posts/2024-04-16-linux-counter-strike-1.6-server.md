---
layout: post
title: "Counter-Strike 1.6 Server (Linux)"
date: 2024-04-16
categories: notes
tags: linux cs1.6 debian gaming server nginx letsencrypt
image:
  path: /assets/showcase/counter-strike-1.6-server/counter-strike-1.6.jpg
  thumbnail: /assets/showcase/counter-strike-1.6-server/counter-strike-1.6.jpg
  caption: Banner or whatever for counter strike from steam.
---

How to run a [Counter-Strike 1.6](https://en.wikipedia.org/wiki/Counter-Strike_(video_game)) server in Linux (namely Debian). I even go into some bonus materials, like making bonus assets (maps, sounds, etc.) download quickly, making statistics accessible as JSON via HTTPS.

[I run my own Counter-Strike 1.6 server!](/showcase/counter-strike-1.6-server/) Please check it out. On that page I also talk about my inspiration for starting a server and more.

* TOC
{:toc}
# Initial setup

Helpful:

* https://www.amoradi.org/20210912011151.html
* I'm using gnu screen for now, because I'm too lazy to daemonize things properly, I guess, at the moment

Get the basics set up:

```
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install libc6:i386 libncurses5:i386 libstdc++6:i386 -y


mkdir ~/steamcmd && cd ~/steamcmd
wget https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz
tar -xvzf steamcmd_linux.tar.gz
./steamcmd.sh
```

Now you should be in `steamcmd`:

```
force_install_dir ./cs16/
login anonymous
app_update 90 -beta beta validate
quit
```

Now you should be back outside of `steamcmd`:

```
mkdir -p ~/.steam/sdk32/
ln -s /home/baudrillard/steamcmd/linux32/steamclient.so ~/.steam/sdk32/steamclient.so
```

You now might be able to:

```
cd ~/steamcmd/cs16
./hlds_run -game cstrike +map de_dust2 +maxplayers 20 +port 27015
```

Make a ufw rule for this service:

```
sudo ufw allow 27015/tcp
sudo ufw allow 27015/udp
```

# Additional tweaks

Edit `~/steamcmd/cs16/cstrike/server.cfg` (might not exist):

some things to set:

* `hostname "Your Server Name"`

* `rcon_password "your_rcon_password"`

* `mapcyclefile "mapcycle.txt"`

* You may to get bot support from like ZBot or POD-Bot?
  ```
  bot_add
  bot_quota 10  # Number of bots
  bot_difficulty 1  # 0 - Easy, 1 - Normal, 2 - Hard, 3 - Expert
  bot_prefix ""
  ```



To add maps check out:

* https://gamebanana.com/mods/cats/5474?_sSort=Generic_MostLiked
* Place the maps in something like `/home/someuser/steamcmd/cs16/cstrike/maps`
  * you can use this command to get the list of the maps to put in the file...
  * try using `cp -r` to merge directories
  * `for file in maps/*.bsp; do basename "$file" .bsp; done > mapcycle.txt`
* Add the map name to `mapcycle.txt`
* i should make a script

## Resource server

I'm not sure if this is working.

```
sv_allowupload 1
sv_allowdownload 1
sv_downloadurl "http://cs16.someodd.zip/cstrike/" 
```

you could just serve the same cstrike dir you're already serving! and then hide the server.cfg to hide the rcon. just modify the nginx config i have here. it'll make it super fast.

add to the server block:

```
    location /cstrike/ {
        root /home/someuser/steamcmd/cs16/;  # Specific root for the cstrike directory
        index index.html index.htm;

        location = /cstrike/server.cfg {
            deny all;  # Deny access to the server.cfg file, returning 403 Forbidden
        }

        try_files $uri $uri/ =404;  # Correctly handle the file paths within cstrike
    }
```

this is working super fast!

## Set up statistics

Basically assumes you have an nginx setup running, see my [#nginx tag](/tags/#nginx).

https://developer.valvesoftware.com/wiki/Server_queries#Goldsource_Server

```
sudo apt-get update
sudo apt-get install qstat
```

Try this command:

```
 % quakestat -a2s localhost:27015
ADDRESS           PLAYERS      MAP   RESPONSE TIME    NAME
localhost:27015        0/20  0/0   de_dust      0 / 0  cstrike Counter-Strike 1.6 Server
```

then make this in `/usr/local/bin/cs_stats.sh`:

```
#!/bin/bash

# Run quakestat and capture the output, skipping the header line
output=$(quakestat -a2s localhost:27015 | tail -n +2)

# Find the PID of the process named 'hlds_linux'
PID=$(pgrep -f hlds_linux)

# Check if the PID was found
if [ -z "$PID" ]; then
    echo "Process not found."
    exit 1
fi

# Calculate the uptime in days and store it in a variable
uptime_days=$(ps -p $PID -o etime= | awk -F'[:-]' '{if (NF == 4) {print $1 " days"} else {print "0 days"}}')

# Parse the output using awk
json=$(echo "$output" | awk -v uptime="$uptime_days" '
BEGIN {
    # Initialize JSON string
    printf "{"
}
{
    # Split the players field into current and max players
    split($2, players, "/");

    # Replace all double quotes with escaped double quotes in the server name
    gsub(/"/, "\\\"", $6);

    # Construct JSON
    printf "\"address\": \"%s\", ", $1;
    printf "\"currentPlayers\": \"%s\", ", players[1];
    printf "\"maxPlayers\": \"%s\", ", players[2];
    printf "\"map\": \"%s\", ", $4;
    printf "\"uptimeDays\": \"%s\"", uptime; # Add uptimeDays to the JSON
}
END {
    # Close JSON string
    printf "}"
}')

echo $json
```

then `sudo chmod +x /usr/local/bin/cs_stats.sh`

```
sudo mkdir /var/www/cs16.someodd.zip
% sudo chown -R someuser:someuser /var/www/cs16.someodd.zip
```

edit `sudo vi /etc/nginx/sites-available/cs16.someodd.zip.conf`:

```
server {
    listen 8765;
    listen 8888 ssl;
    server_name cs16.someodd.zip;
    root /var/www/cs16.someodd.zip;

    ssl_certificate /etc/letsencrypt/live/cs16.someodd.zip/cert.pem;
    ssl_certificate_key /etc/letsencrypt/live/cs16.someodd.zip/privkey.pem;
    
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

```
sudo ln -s /etc/nginx/sites-available/cs16.someodd.zip.conf /etc/nginx/sites-enabled/
```

`crontab -e`:

```
*/30 * * * * /usr/local/bin/cs_stats.sh > /var/www/cs16.someodd.zip/stats.json 2>&1
```

Create the certificate or whatever I selected *webroot* when prompted):

```bash
sudo certbot certonly --webroot-path="/var/www/cs16.someodd.zip" -d 'cs16.someodd.zip'
```

maybe setup the hook for renewal edit `/etc/letsencrypt/renewal/cs16.someodd.zip.conf` under `[renewalparams]`:

```
post_hook = service nginx restart
```

and maybe the other one in that config too...

Validate that the above command works correctly:

```
sudo certbot renew --dry-run
```

# Linux client setup

You can try using Steam:

```
sudo dpkg --add-architecture i386
sudo apt-get update
sudo apt-get install steam-installer 
```

I had trouble using Steam from the Debian Sid repo, so I instead [used the Steam flatpak](https://flathub.org/apps/com.valvesoftware.Steam):

```
flatpak install ./com.valvesoftware.Steam.flatpakref
flatpak run com.valvesoftware.Steam
```

# See also

* [Counter-Strike 1.6 on GameBanana](https://gamebanana.com/games/4254), a good place to get mods (like maps) for CS1.6
* [My CS1.6 server](/showcase/counter-strike-1.6-server)
