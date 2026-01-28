---
date: 2024-04-01
tags:
- irc
- nginx
- letsencrypt
- my_servs
title: Someodd IRC

---


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

## Connecting without ZNC

Only use this connection info if you do not use [my ZNC service](#znc-service).

* SSL only
* Port 6697
* irc.someodd.zip
* Main room is #main

After you  have a client you may be able to simply use the "open in your client" button below.

[Open in Your Client](ircs://irc.someodd.zip:6697/main){: .button .primary}
[Web Client](#web-ui){: .button}

## Rules

Here are rules which breaking may result in a ban:

* Don't be lewd
* Don't be rude
* Be a rad dude

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

## Documentation and tutorials

I documented [how you can run a setup similar to someodd IRC](/phlog/irc-server.gopher.html)

## See also

* [Atheme](https://atheme.dev) (the IRC services daemon the server is running)
* [ngircd](https://ngircd.barton.de/) (the IRC daemon the server is running)

Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/someodd-irc-server.gopher.txt](gopher://gopher.someodd.zip:70/0/phlog/someodd-irc-server.gopher.txt)
