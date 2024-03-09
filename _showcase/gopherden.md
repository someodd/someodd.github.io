---
layout: post
title: "gopherden: Gopher Protocol Server"
date: 2024-03-08
categories: showcase
tags: gopher haskell
image:
  path: /assets/showcase/gopherden/gopherden-demo.gif
  thumbnail: /assets/showcase/gopherden/gopherden-demo.gif
  caption: Screenshot of gopherden being used
---

[gopherden](https://github.com/someodd/gopherden/) is forum software I wrote
with some features I find interesting:

  * Written in Haskell
  * Ephemeral (old threads are deleted)
  * Anonymous (IPs are held for banning purposes, though)
  * View threads as menus or files

I'm leveraging `nixpkgs` to:

  * Make it very easy on me to build and deploy, particularly on my server
  * Have a development environment that sets up/takes down postgresql for
    me, which also helps with deploying a little demo server I'm hosting
    now. It also wipes the database on exit, too.

## It's alive!

You can try out a live server running `gopherden`! You just need a Gopher client!

You can use `gopher` (`sudo apt-get install gopher`) and then use `gopher -p "/" someodd.duckdns.org 7070`.

You could also use [Lagrange](https://gmi.skyjake.fi/lagrange/), which has a
client for Linux, MacOS, Windows, Android, and iOS. Then visit:

gopher://someodd.duckdns.org:7070/1/

Below is a photo of me using Lagrange to visit the forum on my phone:

![Langrange client on a phone, visiting a gopherden forum](/assets/showcase/gopherden/lagrange-gopherden-phone.png).
