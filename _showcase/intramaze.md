---
layout: post
title: "Intramaze: Chat encounters in collab-built maze"
date: 2024-03-05
categories: showcase
tags: webdev haskell games
image:
  path: /assets/showcase/intramaze/intramaze.png
  thumbnail: /assets/showcase/intramaze/example.png
  caption: Screenshot of Waffle being used
---

[Intramaze](https://github.com/someodd/intramaze) is web software which can
serve a website allowing users to collaboratively build a maze together, by
creating rooms which then can be linked together by defining polygons that act
as links to other rooms.  Users can chat together if they're in the same room.

This project needs some polishing up, it was left in a strange state, I think,
and the naming could use some deciding (is it Intramaze? Interwebz?).

I was mostly dabbling with some experimental ways of doing things.

At-a-glance:

  * It can be browsed without JavaScript
  * Is basically kind of a static site generator
  * REST
  * PostgreSQL
  * Users can chat with each other if they bump into each other in a room
  * JSON Web Tokens for authentication
  * Custom merging of web socket server with Scotty, or the like

I've learned I really don't like `Database.Persist` and in the future I'd
rather just use `postgresql-simple`, which was much easier when I was
developing [gopherden](/showcase/gopherden).
