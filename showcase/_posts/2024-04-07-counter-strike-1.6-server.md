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

[I wrote an article on running your own Counter-Strike 1.6 server on Linux](/notes/linux-counter-strike-1.6-server), which should be very similar to the setup I used to configure my server.

## Why?

I thought it would be fun to run a CS1.6 server, especially since I think it's old enough to be very lightweight for my server, so I thought it was just something I could easily put on my server without worrying much about resources.

## Features

* Custom Maps: Non-standard maps!.: fa-cog
* Fast(er) Downloads: instead of the slow, default method, server configured to use fast(er) file server: fa-cloud-download-alt
* Accepts Uploads: Acceupts custom skins and the like, I think?.: fa-cloud-upload-alt
* Feel free to [contact the admin](/about) if you have any requests
* [Setup is kind of documented](/notes/linux-counter-strike-1.6-server)
* [Server statistics publicly available via JSON](https://cs16.someodd.zip/stats.json)

[Look Behind-the-Scenes](/notes/linux-counter-strike-1.6-server){:.button}
[Another Button](#znc-service){:.button}

##  Statistics
{:.stats_section}

At-a-glance info about the Counter Strike 1.6 server. 

[![Stats presented from GameTracker](https://cache.gametracker.com/server_info/cs16.someodd.zip:27015/b_560_95_1.png)](https://www.gametracker.com/server_info/cs16.someodd.zip:27015/)

* fa-user:ERROR:current players:currentPlayers
* fa-user-circle:ERROR:max players:maxPlayers
* fa-map:ERROR:current map:map
* fa-clock:ERROR:days uptime:uptimeDays
* fa-thumbs-up:ERROR:other data:other data
* https://cs16.someodd.zip/stats.json
{:.statistics}

[Set up your own CS1.6 stats service](/notes/linux-counter-strike-1.6-server){:.button}

## Ways to play

### Linux client setup

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

## Known issues

* bot config doesn't work

## Who's using, in the news

Nothing notable yet.

## Feedback

Contact someodd using a method on [the about page](/about).

## News

someodd-cs16

## See also

* [Counter Strike 1.6 on GameBanana](https://gamebanana.com/games/4254)
* [My article on running your own Counter-Strike 1.6 server on Linux](/notes/linux-counter-strike1.6-server)