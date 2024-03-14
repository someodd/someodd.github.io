---
layout: post
title: "gopherden: Forum via Gopher Protocol"
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

I also started trying out TOML as a configuration spec for the first time and
I'm enjoying it, the TOML library I'm using for Haskell is decent.

I'm using Spacecookie as the Gopher server. I've worked with the author of
Spacecookie before!

## It's alive!

You can try out a live server running `gopherden`! You just need a Gopher client!

You can use `gopher` (`sudo apt-get install gopher`) and then use `gopher -p "/" someodd.duckdns.org 7070`.

You could also use [Lagrange](https://gmi.skyjake.fi/lagrange/), which has a
client for Linux, MacOS, Windows, Android, and iOS. Then visit:

gopher://someodd.duckdns.org:7070/1/

Below is a photo of me using Lagrange to visit the forum on my phone:

![Langrange client on a phone, visiting a gopherden forum](/assets/showcase/gopherden/lagrange-gopherden-phone.png).

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
