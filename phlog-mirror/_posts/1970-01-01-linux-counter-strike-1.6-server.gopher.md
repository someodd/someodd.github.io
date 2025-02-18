---
date: 1970-01-01
tags:
- linux
- cs1.6
- debian
- gaming
- server
- nginx
- letsencrypt
title: Counter-Strike 1.6 Server (Linux)

---


How to run a [Counter-Strike 1.6](https://en.wikipedia.org/wiki/Counter-Strike_(video_game)) server in Linux (namely Debian). I even go into some bonus materials, like making bonus assets (maps, sounds, etc.) download quickly, making statistics accessible as JSON via HTTPS.

[I run my own Counter-Strike 1.6 server!](/0/services/counter-strike.md) Please check it out. On that page I also talk about my inspiration for starting a server and more.

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

## Bots

Back up the entire directory first.

### installing metamod

`cd ~/steamcmd`

Before installing Podbot, you need to install Metamod, which is a plugin interface for Half-Life modifications that allows you to use plugins  like Podbot.

**Download Metamod**: Visit the Metamod website and download the latest version appropriate for your server (P or V depending on your server's OS). http://metamod.org/ (i downloaded the regular linux, not x64 version).

**Extract the Files**: Unzip the downloaded file into the `cstrike/addons/` directory of your server.

```
mkdir -p cs16/cstrike/addons/metamod/dlls
tar -xf metamod-1.20-linux.tar.gz -C cs16/cstrike/addons/metamod/dlls
```

edit `c16/cstrike/liblist.gam` and edit the value of `gameddl_linux`:

```
gamedll_linux "addons/metamod/dlls/metamod.so"
```

### installing podbot

Download podbot, I got `podbot_full_V3B22.zip`.

extract the files:

```
unzip podbot_full_V3B22.zip -d cs16/cstrike/addons/
```

edit metamod's plugin config `cs16/addons/metamod/plugins.ini` (maybe create it?) and add the following line:

```
linux addons/podbot/podbot_mm_i386.so
```

now try starting the server! to debug (after launching server):

```
meta list
```

### a bit of configuring podbot

`cs16/cstrike/addons/podbot/podbot.cfg`

you may want to make bots easier, i found them very elite by default:

`pb_minbotskill 15` and `pb_maxbotskill 60`

### creating waypoints

Sadly, I'm not sure you can automatically create waypoints. So you'll maybe have ot do some work, especially if you're using custom maps. No waypoint, no bots.

Download waypoints here for podbot:

* https://gamebanana.com/mods/cats/3524
* https://github.com/ggoulart/cs1.6-server-more-maps/tree/master/podbot/wptdefault

If you can find a `.pwf` file online, then place it in the `cstrike/addons/podbot/wptdefault/` directory. It should be named like the map, just different extension.

Personally I did something like this:

```
git clone https://github.com/ggoulart/cs1.6-server-more-maps/
mv cs1.6-server-more-maps/podbot/wptdefault cs16/cstrike/addons/podbot/wptdefault
```

To test the above I switched to as_oilrig and then tried adding bots with `pb add` in steamcmd on server (rcon troubles)

If you can't find the waypoints online, you can create them manually using the Podbot menu in the game.

1. **Start the Server with the Map**:
   - Launch CS 1.6 and start a server with the map.
2. **Open the Podbot Menu**:
   - In the console, type `pb menu` to open the Podbot command menu.
   - Navigate to the Waypoint options.
3. **Create Waypoints**:
   - Use the menu to add waypoints manually. Walk around the map and place waypoints at strategic locations—pathways, sniping spots, common areas, etc.
   - Save the waypoints frequently to prevent data loss.
4. **Test the Waypoints**:
   - Add bots to see if they navigate using the newly created waypoints. Adjust if necessary

how do i add bots by default?

### sort of unethical: fake clients

#### installing amxmodx

amxmodx is maybe worth reading about.

make it seem like more people are on the server than there are to encourage people to join

https://www.amxmodx.org/downloads.php

```
mkdir amx
tar -xf amxmodx-1.8.2-base-linux.tar.gz -C amx/
tar -xf amxmodx-1.8.2-cstrike-linux.tar.gz -C amx/
cp -r amx/addons/amxmodx cs16/cstrike/addons/
```

once again edit `cs16/cstrike/addons/metamod/plugins.ini`:

```
linux addons/amxmodx/dlls/amxmodx_mm_i386.so
```

**Configure Basic Settings**:

- Navigate to `cstrike/addons/amxmodx/configs/` and adjust settings in the configuration files like `amxx.cfg`, `users.ini` (for admin privileges), and `plugins.ini` (for controlling which plugins are active).

**Verify Installation**:

- Restart your server.
- Connect to your server and type `meta list` in the server console. You should see both Metamod and AMX Mod X listed as loaded.

#### fakefull

Downloads: https://forums.alliedmods.net/showthread.php?p=254620

```
mv fakefull_original.amxx cs16/cstrike/addons/amxmodx/plugins
unzip fakefull_original.zip -d fakefull
cp -r fakefull/addons/amx/* cs16/cstrike/addons/amxmodx
unzip botnames.zip -d cs16/cstrike/addons/amxmodx/configs/
chmod 644 cstrike/addons/amxmodx/plugins/fakefull_original.amxx
```

try adding this to your `cs16/cstrike/addons/amxmodx/configs/amxx.cfg`:

```
// ******************  FakeFull Settings  ******************

//Turns Automatic mode on.
//ff_players must be set higher then 0.
//<1 = ON || 0 = OFF>
ff_automode 1

//Minimal number of fake and real clients on the server.
//Fake clients will be kicked so that the total players on the
//Server equals this number.  If there are more players than this
//All the fake players will be gone. Do NOT set this at or aboive
//your max player count or no one will be able to join
//REQUIRED to be above 0 for Automatic mode.
ff_players 3

//Delay between each fake client joins/leaves.
//REQUIRED to be above 0 for Automatic mode and regular bot add.
ff_delay 1
```

edit `cstrike/addons/amxmodx/configs/plugins.ini` and add:

```
fakefull_original.amxx
```

i'm not sure why it's not picking up on my settings in `amxx.cfg` but try this in steamcmd:

```
amx_addfake 3
```

at this point i see bot spectating but player count doesn't artificially increase

#### making our own

I'll make a repo for this and other mods...

Set up the fake queries depedency. Download this https://forums.alliedmods.net/showthread.php?t=244450 and install:

1. Download `fakequeries_binary.zip` from the attachments.

2. Extract to game directory
   ```
   unzip fakequeries_binary.zip -d cs16/cstrike/
   ```

3. Add "fake_queries" in your modules.ini (in `cs16/cstrike/addons/amxmodx/configs/modules.ini`).

Now save below as `addons/amxmodx/scripting/botcount.sma`:

```
#include <amxmodx>
#include <fake_queries>

public plugin_init()
{
    register_plugin("Count Bots as Players", "1.0", "someodd");

    // Update player counts periodically without changing other server info
    set_task(1.0, "update_player_counts", _, _, _, "b");
}

public update_player_counts()
{
    // Calculate total players including bots
    new total_players = count_total_players();
    new max_players = get_maxplayers();  // Use the server's configured max players

    // Set the fake server query responses for player and bot counts
    fq_set_players(total_players);  // Total players including bots
    fq_set_botsnum(0);              // Report 0 bots

    // Log the operation results
    log_status(fq_set_players(total_players), "players");
    log_status(fq_set_maxplayers(max_players), "max players");
    log_status(fq_set_botsnum(0), "bots");
}

count_total_players()
{
    new count = 0, maxPlayers = get_maxplayers();
    for (new i = 1; i <= maxPlayers; ++i)
    {
        if (is_user_connected(i))  // Check if the slot is occupied
            count++;
    }
    return count;
}

log_status(result, const description[])
{
    if (result != 0)  // Change to check for non-zero return, assuming non-zero means failure
        server_print("^4[Count Bots as Players]^1 Failed to set %s.", description);
}
```

prepare:

1. `cd addons/amxmodx/scripting/`
2. compile with `./amxxpc botcount.sma` 
3. put the result in the `addons/amxmodx/plugins` with `mv botcount.amxx ../plugins`
4. enable by adding `botcount.amxx` to `addons/amxmodx/configs/plugins.ini`

#### making podbots less obviously bots

finally make it less obvious that there's podbots.

`cd cs16/cstrike/addons/podbot`

edit `podbot.cfg` specifically these:

* `pb_detailnames 0`

* `pb_chat 0`

* `pb_radio 0`

* Finally read this section carefully. I thought just having four would be enticing to people, while being maybe kind of plausible if they all dropped off or something when there's not a waypoint file for next map or whatever:
  ```
  # These lines below are adding automatically the bots to the server when^M
  # the new game is created on the listenserver or the dedicated server^M
  # is starting.^M
  # As many such lines like "pb add" is here as many bots will be added ^M
  # to the server (unless You will not exceed pb_maxbots setting).^M
  pb add 100^M
  pb add 100^M
  pb add 100^M
  pb add 100^M
  ```

Now edit the bot names in `botnames.txt`.

#### restrict over-powered weapons with amxmodd

enable restmenu in `cstrike/addons/amxmodx/configs/plugins.ini`:

```
restmenu.amxx           ; restrict weapons menu
```

you'll also want to set up admin by editing `configs/users.ini` and adding your steam ID you may find from something like https://www.steamidfinder.com/lookup/someodd/

here i added:

```
"STEAM_0:0:854249781" "" "abcdefghijklmnopqrstu" "ce"
```

then in the console in game `~`:

```
amx_restmenu
```

or something like that?

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

# todo

i want to add quake sounds

# See also

* [Counter-Strike 1.6 on GameBanana](https://gamebanana.com/games/4254), a good place to get mods (like maps) for CS1.6
* [My CS1.6 server](/services)

Original content in gopherspace: gopher://gopher.someodd.zip:7071/phlog/
