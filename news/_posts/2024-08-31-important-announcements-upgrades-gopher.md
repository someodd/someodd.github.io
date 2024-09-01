---
layout: post
title: "Important Announcements! Service Availability & Changes!"
date: 2024-08-31
categories: news
tags: server, gopher, irc, znc
---

Some important news about my services.

* TOC
{:toc}

## Moving to gopher

I'll be winding down the website in favor of being active on my personal
gopherhole:

gopher://gopher.someodd.zip:7070/

You may have noticed the new frontpage of this website simply points to the
Gopher URL and also a link to the old index.

## Planned extended downtime

The server will be down for maintenance. I'm not sure for exactly how long and
when, but if I were to guess I'd say down for 1 week and within 30 days.

The server will come back from the downtime with:

* More RAM
* GPU installation
* Hopefully more reliable and better network/bandwidth for services
* Storage upgrade (maybe)

## Winding down ZNC service

I've decided to stop serving ZNC. I don't want to spread myself thin and give
every service the attention it deserves so things can be reliable and receive
good maintenance.

The IRC server will remain up! No plans to axe that.

The further rationale behind retiring the ZNC service is that for most people
who would have a ZNC account with me, simply using XMPP to connect to my IRC
server is possible and maybe easier.

## Security Post-mortem

I marked my GitHub Pages repo for this website as private for a while,
and I ran into a vulnerability with something like a domain-takeover.

GitHub Pages allows you to host a website using GitHub. This site is my
`someodd.github.io` repo, which is like the main website for your account.
GitHub Pages allows for using your own domain. As you can see you're accessing
it using `www.someodd.zip`. In my domain's DNS settings, I have a CNAME entry
for `www.someodd.zip` that points to `someodd.github.io`.

Everything was working well for a while, the site/repo was public. Then one day
recently I marked it as *private.* Privating a GitHub Pages repo will take
*this particular site down,* yet your domain will still be pointing to GitHub
Pages. I'm not sure why they can't see the CNAME record pointing to a specific
user GitHub, or something like that in order to prevent at least some
takeovers. So some scammy-looking site was sitting where `www.someodd.zip` left
off.

I made the website public again and the problem isn't a threat for now. Also, I
did submit a report to GitHub. They said they are aware and that's why they
made the DNS Validation tool (or whatever it's called). I made sure to verify
my domain.
