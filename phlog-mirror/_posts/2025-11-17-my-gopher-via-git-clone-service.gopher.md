---
date: 2025-11-17
tags:
- selfhosted
- gopher
- news
- announcements
title: My Gopher Host That Clones Your Repo

---


I started a service which aims to make starting a gopherhole easier.

I host the gopherhole for you--just provide a git URI and my service will clone it and turn it into a subdirectory of [gopher://gopher.someodd.zip/1/hosted](gopher://gopher.someodd.zip/1/hosted)/. This service has support/expects the [bore](https://github.com/someodd/bore) format (my phlog/gopherhole builder [think Jekyll, but for gopherspace]).

Currently only tested with GitHub repos.

To sign up you can [contact me](/1/about). Please look at [the hosted selector](/1/hosted) for more info.

# Background

I wanted to exemplify the ease by which you can create "addons" for [Venusia](https://github.com/someodd/venusia), my Gopher Protocol server daemon.

These "addons" are just gateways to scripts. I mapped a Gopher selector to a script `repobore.sh`.

This script is rather, admittedly, half-baked, and thus I expect iteration when user(s) demand it. Currently as of writing only one person uses this service and I haven't heard complaints yet.

Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/my-gopher-via-git-clone-service.gopher.txt](gopher://gopher.someodd.zip:70/0/phlog/my-gopher-via-git-clone-service.gopher.txt)
