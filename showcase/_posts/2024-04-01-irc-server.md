---
layout: post
title: "IRC Server"
date: 2024-04-01
categories: showcase
tags: irc
image:
  path: /assets/showcase/irc-server/someodd-irc-server.webp
  thumbnail: /assets/showcase/irc-server/someodd-irc-server.webp
  caption: AI-generated image representation of an IRC chat room.

---

Someodd IRC Server is open to the public, but I try to fill it with interesting developers, people I've enjoyed working with, especially to facilitate further open source collaboration. It features various services (like `nickserv`). Also, you can try to request a ZNC account (using contact info on [my about page](/about)).

 [I wrote some about my journey setting up this service](/notes/irc-server), so the setup is transparent, and I suppose in a way, open source. Try setting it up yourself!

* TOC
{:toc}
[screen shot of something fun here]

# Features

Some of the features provided:

* [Atheme services](https://atheme.dev/):
  * In general you usually can `/msg servicename help` to get info on usage
  * Nickserv: register your nickname
  * ChanServ: take ownership over a channel
  * InfoServ
  * SaslServ
  * MemoServ: kind of like email in IRC
  * GroupServ
  * StatServ: records various network statistics.
  * More
* ZNC: stays connected for you! Hopefully you'll never miss a message again!
* (Maybe) coming soon:
  * Web UI--join via the web
  * Chat via Gopher
  * Post to [Gopherden](/showcase/gopherden)


And likely more!

# Connecting

* SSL only
* Port 6697
* irc.someodd.zip
* Main room is #main

## Web Client

<iframe src="https://kiwiirc.com/client/irc.someodd.zip/+6697/#main" style="border:0; width:100%; height:450px;"></iframe>

## Hexchat (Linux, probably more)

![Screenshot of HexChat, connected to someodd IRC](/assets/showcase/irc-server/someodd-irc-hexchat.png)

How to connect using hexchat.

Using the Network List, click the *Add* button to bring up the window which allows you to edit network info:

* Servers: just `irc.someodd.zip/6697`
* You may want to tick "Use SSL for all the servers on this network"

## Weechat (Linux, probably more)

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

## Revolution IRC (Android)

I think ZNC doesn't play nice with Revolution IRC. When you add/edit a server just make sure the connection info is as follows:

* server address: irc.someodd.zip
* port: 6697
* Use SSL/TLS (checked)

You can get Revolution IRC on F-Droid.

# Rules/guidelines

* Nothing NSFW
* Nothing political
* Try to keep it about tech and collaboration

# ZNC maybe available on request

I am hosting ZNC and you can contact me using one of the methods on [my about page](/about) to request a ZNC account and info.

# News

[latest posts for the #irc-server tag in the /announcements or /news collection?]

# Known issues

* Not using Atheme's Global notice module to send maintenance notifications
* I'd like to enhance privacy and implement/utilize cloaks and the like more
* I'm not really doing much of any administration, there's no other operators
* Server isn't really fully set up (like services, permissions)

# Feedback

Take a look at [my about page](/about) for my contact info.

# Who's using, in the news

projects hosted here?

# Documentation and tutorials

* I documented [how you can run a setup similar to someodd IRC](/posts/irc-server)

# See also

* [Atheme](https://atheme.dev) (the IRC services daemon the server is running)
* [ngircd](https://ngircd.barton.de/) (the IRC daemon the server is running)