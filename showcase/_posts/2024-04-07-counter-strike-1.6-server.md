---
layout: landing
title: "someodd CS 1.6"
date: 2024-04-07
logo: /assets/showcase/counter-strike-1.6-server/counter-strike-logo.png
categories: showcase
tagline: Counter Strike 1.6 Server
tags: gaming someodd-cs16 nginx
image:
  path: /assets/showcase/counter-strike-1.6-server/counter-strike-1.6.jpg
  thumbnail: /assets/showcase/counter-strike-1.6-server/counter-strike-1.6.jpg
  caption: Banner or whatever for counter strike from steam.

---

## Good ol' head shots

Counter Strike 1.6 server. It's at `cs16.someodd.zip`!

##  Statistics
{:.stats_section}

At-a-glance info about the Counter Strike 1.6 server. 

These stats are fetched using JavaScript.

* fa-user:ERROR:current players:currentPlayers
* fa-calendar:ERROR:max players:maxPlayers
* fa-square:ERROR:current map:map
* fa-star:ERROR:days uptime:uptimeDays
* fa-thumbs-up:ERROR:other data:other data
* https://cs16.someodd.zip/stats.json
{:.statistics}

[How to Get Stats](#running-it-yourself){:.button}


## See also

FPS Banana

## Running it yourself

Helpful:

* https://www.amoradi.org/20210912011151.html

i'm using gnu:

```
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install libc6:i386 libncurses5:i386 libstdc++6:i386 -y


mkdir ~/steamcmd && cd ~/steamcmd
wget https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz
tar -xvzf steamcmd_linux.tar.gz
./steamcmd.sh


```

once in ssteamcmd

```
force_install_dir ./cs16/
login anonymous
app_update 90 -beta beta validate
quit
```

now:

```
mkdir -p ~/.steam/sdk32/
ln -s /home/baudrillard/steamcmd/linux32/steamclient.so ~/.steam/sdk32/steamclient.so
```

now you might be able to:

```
cd ~/steamcmd/cs16
./hlds_run -game cstrike +map de_dust2 +maxplayers 20 +port 27015
```

ufw:

```
sudo ufw allow 27015/tcp
sudo ufw allow 27015/udp
```

on client:

```
sudo dpkg --add-architecture i386
sudo apt-get update
sudo apt-get install steam-installer 
```

or https://flathub.org/apps/com.valvesoftware.Steam

and then

```
flatpak install ./com.valvesoftware.Steam.flatpakref
flatpak run com.valvesoftware.Steam
```

### Additional tweaks

...

### Set up statistics

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

**FIX THIS SHIT NEEDS HTTPS AND LETSENCRYPT**

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