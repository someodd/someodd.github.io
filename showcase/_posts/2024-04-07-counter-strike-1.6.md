---
layout: landing
title: "someodd CS 1.6"
date: 2024-04-01
logo: /assets/showcase/irc-server/someodd-irc-server-logo.png
categories: showcase
tagline: Counter Strike 1.6 Server
tags: gaming someodd-cs16
image:
  path: /assets/showcase/irc-server/someodd-irc-server.webp
  thumbnail: /assets/showcase/irc-server/someodd-irc-server.webp
  caption: AI-generated image representation of an IRC chat room.

---

## Good ol' head shots

It's at `cs16.someodd.zip`!

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