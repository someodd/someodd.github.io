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

Free ZNC accounts maybe available on request!

I am hosting ZNC and you can contact me using one of the methods on [my about page](/about) to request a ZNC account and info.

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
