---
layout: post
title: "gopherden: Forum via Gopher Protocol"
date: 2024-03-08
categories: showcase
tags: gopher haskell nix
image:
  path: /assets/showcase/gopherden/gopherden-demo.gif
  thumbnail: /assets/showcase/gopherden/gopherden-demo.gif
  caption: Screenshot of gopherden being used
---

[gopherden](https://github.com/someodd/gopherden/) is forum software for the [Gopher Protocol](/tag/gopher). View the official gopherden server [via the web](https://gopher.floodgap.com/gopher/gw?a=gopher%3A%2F%2Fgopher.someodd.zip%3A7070%2F) or [via the Gopher Protocol (as intended)](gopher://gopher.someodd.zip:7070/) (you may want to read [my post about the Gopher Protocol](/posts/gopher)).

* TOC
{:toc}
# Features (Users/Visitors)

Features for the people who use a gopherden:

* Transient content. Old threads are deleted. There's max thread count, when reached and a new post is made, the oldest one gets erased.
* Anonymous, except IPs are associated with posts for banning and legal purposes
* View threads as gopher maps or as text files (ASCII art, kind of)
* Prevents some accidental reposting by refreshing
* Prevents new threads being created where the message/content is the same as another in the DB
* "Secret codes" which allow you to use codes like `<jgsBagOfMoney>` to potentially show an ASCII art picture
* Rate-limiting (how fast you can make new threads or posts)
* Handles/parses Gopher and HTTP(S) URIs into menu/map items

# News

[news from my blog about gopherden]

# Known issues

* No real bot prevention. A captcha-like system could be implemented.

# Feedback

* Contact me using a method mentioned on [my about page](/about)
* [Create an issue on the gopherden GitHub repo](https://github.com/someodd/gopherden/issue)

# Who's using, in the news

Some other instances, notable users...

# Documentation and tutorials

* [The project's README.md](https://github.com/someodd/gopherden/blob/master/README.md)

# See also

* [This site's gopher tag](/tag/gopher)
* [My post about the Gopher Protocol](/notes/gopher)
* [gopherden GitHub repo](https://github.com/someodd/gopherden)

# Post on the official server!

This feels kind of redundant.

Official *gopherden* server at [gopher://gopher.someodd.zip:7070](gopher://gopher.someodd.zip:7070). You may want to read [my article on the Gopher Protocol](/notes/gopher) for help on browsing gopherspace.

You can use [my Gopher Protocol client, "waffle"](/showcase/waffle) to visit the forum!

You can use `gopher` (`sudo apt-get install gopher`) and then use `gopher -p "/" gopher.someodd.zip 7070`.

You could also use [Lagrange](https://gmi.skyjake.fi/lagrange/), which has a
client for Linux, MacOS, Windows, Android, and iOS.

Below is a photo of me using Lagrange to visit the forum on my phone:

![Langrange client on a phone, visiting a gopherden forum](/assets/showcase/gopherden/lagrange-gopherden-phone.png).

# Want your own gopherden or want to develop?

[gopherden is open source](https://github.com/someodd/gopherden).

## Developer and Server Features

Features for those who host or develop (a) gopherden:

* Written in Haskell
* TOML configuration spec. First time I implemented a TOML config and I find it decent.
* Uses [Spacecookie](https://github.com/sternenseemann/spacecookie), a wonderful Gopher Protocol library. I've worked with the author before.
* CLI for banning

I'm leveraging `nixpkgs` to:

  * Make it very easy on me to build and deploy, particularly on my server
  * Have a development environment that sets up/takes down postgresql for
    me, which also helps with deploying a little demo server I'm hosting
    now. It also wipes the database on exit, too.

## Setting up a server

As of the time I'm writing this I don't have a `systemd` integration or Debian
package, so a set up I like and think is simple/easy:

1. clone the repo and `cd` into it
1. make sure `nixpkgs` is installed on your
   system with `experimental-features = flakes nix-command` in your
   `~/.config/nix/nix.conf`
1. start a GNU Screen session `screen -S services`
1. `nix develop`
1. `nix run .#gopherden -- launch`

There you go! Keep in mind if you send the exit signal (I think) inside of `nix
develop` the database gets wiped.
