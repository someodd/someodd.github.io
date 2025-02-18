---
date: 2024-04-01
tags:
- irc
- someodd-irc
- nginx
- letsencrypt
- showcase
title: Someodd IRC

---
Original content in gopherspace: gopher://gopher.someodd.zip:Just 7071/phlog/


## The skinny

Place for people to talk using [a text-based chat system from 1988 (IRC)](https://en.wikipedia.org/wiki/IRC). Some people may find IRC dated and counter-intuitive (partially for them, I have something in the works). Think of it like an old [Discord](https://discord.com/), where people can create their own channels (and sort of be the boss of them).

Since IRC is old there may be some features lacking that you're used to, like the ability to upload media. Also, to receive messages while you're not around/connected you might want to [ask me for a ZNC account](#znc-service), or you may miss messages while you're away.

There's different ways to join (called "clients"). For example, you can join using your phone with a client like [Revolution IRC](https://github.com/MCMrARM/revolution-irc), or on your laptop using [Hexchat](https://hexchat.github.io/).

Open to the public, but I try to fill it with people I respect, interesting developers, people I've enjoyed working with, hopefully sparking further collaboration and "cross-pollination."

 [I wrote some about my journey setting up this service](/notes/irc-server), so the setup is transparent, and I suppose in a way, open source.

The central place (channel, room) to talk is `#main`.

[Read the Rules](#rules){: .button .primary}
[Get Connected (No ZNC)](#connecting-without-znc){: .button}
[Get Connected With My ZNC Service](#connecting-with-znc){: .button}
[Read About Nickname Registration](#register-your-nickname){: .button}

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
  * Chat via Gopher
  * [Whisper Radio integration](/showcase/whisper-radio)
  * [Gopherden](/showcase/gopherden) notifications

This is probably a non-fully-inclusive list of features!

[ZNC Service](#znc-service){:.button}
[Another Button](#znc-service){:.button}

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

## Connecting without ZNC

Only use this connection info if you do not use [my ZNC service](#znc-service).

* SSL only
* Port 6697
* irc.someodd.zip
* Main room is #main

After you  have a client you may be able to simply use the "open in your client" button below.

[Open in Your Client](ircs://irc.someodd.zip:6697/main){: .button .primary}
[Web Client](#web-ui){: .button}

### Web UI

{:.special}

You can chat through your web browser using Kiwi IRC.

Sorry, this seems to be broken.

[Use Kiwi IRC](https://kiwiirc.com/nextclient/#irc://irc.someodd.zip:+6697/#main){:.button .primary}
[Open in Your Client](ircs://irc.someodd.zip:6697/main){: .button}
[Client Connection Info](#connecting){:.button}

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

## Rules

Here are rules which breaking may result in a ban:

* Nothing NSFW
* Nothing political
* Keep it positive
* Keep it about tech and collaboration
* Be nice, no trolling

## Register your nickname!

IRC is maybe a weird beast. One example is that anyone can take your nickname unless you register it with `NickServ`:

```
/msg REGISTER <password> <email>
```

And to login (I think this is the command):

```
/msg IDENTIFY <password>
```

Note you don't actually include the `<>` brackets.

### Note for ZNC users

I know it may be confusing considering you have a password for your ZNC, but you still may want to register your nickname. Once you've done that you can [manage your ZNC account's someodd network](https://znc.someodd.zip) to:

* Enable the `nickserv` module (check the checkbox)
* input your password into the text box immediately to the right
* Click the "save and return" button

**Sorry, the above does not seem to work for me. I think in the future I may post SASL instructions. So for now manually IDENTIFY.** The below might work, though:

* While you're editing the `someodd` network for your user, notice in the general navigation section there's a link or something like *network modules (someodd) SASL*.
* Enter your username and password (from `NickServ`)
* Click *save* button

[Manage ZNC (Web)](https://znc.someodd.zip/){: .button}

## ZNC service

* **What is ZNC?** Basically, it is something that stays connected to IRC for you and you in turn connect to the ZNC.
* **Why use ZNC?** Primarily because if you're not connected to IRC you may miss messages.
* **How can I get an account?** Contact me using one of the methods on [my about page](/about) to request a free ZNC account. You need to actually use the IRC server.
* Some offerings: playback, clientbuffer

### Connecting with ZNC

Information on connecting to my IRC service using the ZNC account I made for you. You use this information *instead* of the regular connection info.

* **Manage your account:** https://znc.someodd.zip
  * You may want to change your password straightaway. You can also use `/msg *controlpanel set password myusername mynewpassword` once you're connected.
* **Recommended connection settings:**
  * **Host:** znc.someodd.zip/+6669
    * SSL, port 6669
  * **Username:** `username@identifier/network` (for example `someuser@mylaptop/someodd`)
    * `username` is your ZNC username
    * `identifier` is used for [ClientBuffer](https://wiki.znc.in/Clientbuffer), basically just set it to an arbitrary name of your device. For example, I use an identifier `hexchat` for my laptop and could use `revolution` for my phone.
    * `network`: the name of the IRC network to connect to by default. This is the name for the IRC network in your ZNC settings. Specifying this is nice because you can have one connection to ZNC open per network, so you can perhaps better use many networks at once (without having to use `JumpNetwork`, which I think is annoying).
  * **Password:** your ZNC password.
* **Alternative connection settings:**
  * Some clients like [Revolution IRC](https://github.com/MCMrARM/revolution-irc) may not let you enter in a regular username (you don't want to use SASL, I don't think), so you can actually use a password in this format:
    * `username@identifier/someodd:yourpassword`

* Let me know if you want me to make changes to your account
* Feel free to connect to other servers using this service.

I have some note about these commands which may come in handy, but don't just blindly use them:

```
/msg *controlpanel set AutoClearChanBuffer $me False
/msg *controlpanel set AutoClearQueryBuffer $me False
```

[Manage ZNC (Web)](https://znc.someodd.zip/){: .button .primary}

## News

someodd-irc

## Known issues

* Not using Atheme's Global notice module to send maintenance notifications
* I'm not really doing much of any administration, there's no other operators
* Server isn't really fully set up (like services, permissions)
* Web UI (KiwiIRC) seems to be broken
* no real email system. this may mean you can't recover your account/nickname if you forget your password or the like

## Feedback

Take a look at [my about page](/about) for my contact info. Also, you can contact me on the IRC server, I'm `someodd`. Let me know if you have issues with the server or want help.

## Who's using, in the news

Some interesting people use this service. I also hope that some projects will soon be hosted here.

## Documentation and tutorials

* I documented [how you can run a setup similar to someodd IRC](/notes/irc-server)

## See also

* [Atheme](https://atheme.dev) (the IRC services daemon the server is running)
* [ngircd](https://ngircd.barton.de/) (the IRC daemon the server is running)

