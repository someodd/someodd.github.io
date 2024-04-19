---
layout: landing
title: "Someodd IRC"
date: 2024-04-01
logo: /assets/showcase/irc-server/someodd-irc-server-logo.png
categories: showcase
tagline: Boutique IRC server by someodd.
tags: irc someodd-irc nginx letsencrypt
image:
  path: /assets/showcase/irc-server/someodd-irc-server.webp
  thumbnail: /assets/showcase/irc-server/someodd-irc-server.webp
  caption: AI-generated image representation of an IRC chat room.

---

## The skinny

Someodd IRC Server is open to the public, but I try to fill it with interesting developers, people I've enjoyed working with, especially to facilitate further open source collaboration. It features various services (like `nickserv`). Also, you can try to request a ZNC account (using contact info on [my about page](/about)).

 [I wrote some about my journey setting up this service](/notes/irc-server), so the setup is transparent, and I suppose in a way, open source. Try setting it up yourself!

[Get Connected](#connecting){: .button}

## Features

* Atheme services: More IRC functionality! Lots of services like memos (it's like email), SaslServ, Nickserv.: fa-cog
* ZNC: It can keep the messages while you're gone. Be sure to inquire about a free account!: fa-cloud-download-alt
* Collaboration: Some enforcement of only productivity-related discussion I try to invite some developers I like.: fa-people-arrows
* [Atheme services](https://atheme.dev/):
  * In general you usually can `/msg servicename help` to get info on usage
  * Nickserv: register your nickname
  * ChanServ: take ownership over a channel
  * InfoServ
  * SaslServ
  * MemoServ: kind of like email in IRC
  * GroupServ
  * StatServ: records various network statistics.
* [Server statistics publicly available via JSON](https://irc.someodd.zip/stats.json)
* (Maybe) coming soon:
  * Web UI--join via the web
  * Chat via Gopher
  * Post to [Gopherden](/showcase/gopherden)

This is probably a non-fully-inclusive list of features!

[ZNC Service](#znc-service){:.button}
[Another Button](#znc-service){:.button}

## Web UI
{:.special}

You can chat through your web browser using Kiwi IRC.

[Use Kiwi IRC](https://kiwiirc.com/nextclient/#irc://irc.someodd.zip:+6697/#main){:.button .primary}
[Open in Your Client](ircs://irc.someodd.zip:6697/main){: .button}
[Client Connection Info](#connecting){:.button}

##  Statistics
{:.stats_section}

At-a-glance info about someodd IRC. 

These stats are fetched using JavaScript.

* fa-user:ERROR:current users:current number of users
* fa-calendar:ERROR:days uptime:ngircd uptime days
* fa-square:ERROR:channels:number of channels formed
* fa-star:ERROR:opers online:operators online
* fa-thumbs-up:ERROR:most users:highest connection count
* https://irc.someodd.zip/stats.json
{:.statistics}

[Atheme Statistics HTTP](https://github.com/someodd/atheme-stats-http){:.button}

## Connecting

* SSL only
* Port 6697
* irc.someodd.zip
* Main room is #main

After you  have a client you may be able to simply use the "open in your client" button below.

[Open in Your Client](ircs://irc.someodd.zip:6697/main){: .button .primary}
[Web Client](#web-ui){: .button}

### Hexchat (Linux, probably more)

![Screenshot of HexChat, connected to someodd IRC](/assets/showcase/irc-server/someodd-irc-hexchat.png)

How to connect using hexchat.

Using the Network List, click the *Add* button to bring up the window which allows you to edit network info:

* Servers: just `irc.someodd.zip/6697`
* You may want to tick "Use SSL for all the servers on this network"

### Weechat (Linux, probably more)

How to connect using weechat.

![Screenshot of Weechat, connected to someodd IRC, in a terminal](/assets/showcase/irc-server/someodd-irc-weechat.png)

Simply use this command:

```
/server add someodd irc.someodd.zip/6697 -ssl
```

Then connect and join the main channel:

```
/connect someodd
/join #main
```

### Revolution IRC (Android)

I think ZNC doesn't play nice with Revolution IRC. When you add/edit a server just make sure the connection info is as follows:

* server address: irc.someodd.zip
* port: 6697
* Use SSL/TLS (checked)

You can get Revolution IRC on F-Droid.

![Revolution IRC connected to someodd IRC](/assets/showcase/irc-server/someodd-irc-revolution-irc.png)

## Guidelines

In order of how much I care to enforce:

1. Nothing NSFW
1. Nothing political
1. Keep it about tech and collaboration

## ZNC Service

I am hosting ZNC and you can contact me using one of the methods on [my about page](/about) to request a free ZNC account. I just ask you actually use my IRC server in return.

Feel free to connect to other servers using this service.

Let me know if you want me to make changes to your account for you.

### General info

* Point to znc.someodd.zip port 6669
* your username and password i gave to you
* Change your password and other settings here, i believe: https://znc.someodd.zip
  * Try the 6669 (note you may need to change browser security perms to allow accessing a website on that port).
  * you can also try, once logged in to znc, something like `/msg *controlpanel set password myusername mynewpassword`
* You can set an IRC password like `username/network:password` to connect to a specific network by default. This will let you have different ZNC connections (say one for someodd and one for libera.chat)

### Client configs

* Revolution IRC
  * In the `Password` field, input your ZNC credentials in the format `username/network:password`. This tells Revolution IRC how to log in to ZNC and specifies which of your ZNC-configured networks to connect to. 
* [HexChat - ZNC](https://wiki.znc.in/HexChat)
  * Add your ZNC server’s address and port to the server list, like `znc.example.com/6667` for non-SSL connections or `znc.example.com/+6697` for SSL connections (prepend the port number with a `+` for SSL).
  * In the `Password` field, input your ZNC credentials in the format `username/network:password`. This tells HexChat how to log in to ZNC and specifies which of your ZNC-configured networks to connect to. This will let you have different ZNC connections (say one for someodd and one for libera.chat)

### Don't let your clients miss messages!

You may want to make sure you don't miss messages on one device, just because you saw it on another, or just general concerns about missing messages.

These days you might have a laptop client and a phone client connected to the ZNC.

Go to https://znc.someodd.zip/ and then edit the network you want and make sure `clientbuffer` mod enabled and has the argument `autoadd`.

if that seems insufficient you *may* want to use this in case you have more than one device using the znc account:

```
/msg *controlpanel set AutoClearChanBuffer $me False
/msg *controlpanel set AutoClearQueryBuffer $me False
```

I think the `playback` module is globally autoloaded.

## News

someodd-irc

## Known issues

* Not using Atheme's Global notice module to send maintenance notifications
* I'd like to enhance privacy and implement/utilize cloaks and the like more
* I'm not really doing much of any administration, there's no other operators
* Server isn't really fully set up (like services, permissions)

## Feedback

Take a look at [my about page](/about) for my contact info.

## Who's using, in the news

projects hosted here?

## Documentation and tutorials

* I documented [how you can run a setup similar to someodd IRC](/notes/irc-server)

## See also

* [Atheme](https://atheme.dev) (the IRC services daemon the server is running)
* [ngircd](https://ngircd.barton.de/) (the IRC daemon the server is running)
