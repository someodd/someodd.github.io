---
layout: post
title: "gopherdashboard: Sysadmin via Gopher"
date: 2024-03-07
categories: showcase
tags: gopher haskell
image:
  path: /assets/showcase/gopherdashboard/gopherdashboard-demo.gif
  thumbnail: /assets/showcase/gopherdashboard/gopherdashboard-demo.gif
  caption: Demo of gopherdashboard being used
---

[gopherdashboard](https://github.com/someodd/gopherdashboard).

Anonymous forum software written for the Gopher Protocol in Haskell. You may
want to read [my article on using the Gopher
Protocol](/posts/2024/03/10/gopher.md).

Visit the official live demo of the forum at
[gopher://someodd.duckdns.org:7070/](gopher://someodd.duckdns.org:7070/). For
some reason [Lagrange](https://gmi.skyjake.fi/lagrange/) wants the address
[gopher://someodd.duckdns.org:7070/1/](gopher://someodd.duckdns.org:7070/1/).

Some core things I learned or did:

  * systemd daemon managed
  * release management
  * packaging for debian
  * worrying about privileges/infosec (I am, after all, allowing people to
    configure the dashboard to execute potentially any arbitrary command they
    want)

Cool features:

  * Execute arbitrary commands if you want (flag to enable this not-so-safe feature)
  * Monitor arbitrary files
  * Configure through a JSON file

## How I use it

Some configuration I have as of March 7, 2024 is:

  * Ability to query/get status of arbitrary services
  * Look at disk usage
  * Show the uptime and various backup ages
  * Quip of the latest APC UPS events (ability to view full log)
  * `debsecan` report, with CVEs found count displayed on dashboard
  * `chrootkit` log
  * `restic` backup information (for server)
