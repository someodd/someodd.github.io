---
date: 2026-01-20
tags:
- gopher
- my_warez
- my_servs
title: 'ChatGopherPT: talking to LLMs over Gopher'

---


I'm the bastard who brought Chinese AI to the pre-web Internet.

I wanted a way to talk to a language model that felt closer to a daemon than a platform.

So I put `ollama` behind Gopher, using DeepSeek's models.

Available here: gopher://gopher.someodd.zip/1/gateway/ollama/list

ChatGopherPT runs on my own server. It exposes local models as a plain text Gopher menu, accepts a prompt, and streams the reply line by line. Port 70 in, text out. That is the whole interface.

There are no accounts to create and nothing to install beyond a Gopher client. There is no analytics pipeline, no consent banner, no invisible third party deciding how fast you get an answer. When it responds, it is because my machine did the work.

Gopher keeps things honest. The protocol is small enough to understand fully. You can watch the traffic, reason about the behavior, and write a client in an afternoon. The result feels calm and legible.

I like that this works from a terminal. I like that it works from Lynx. I like that it works from something you hacked together after dinner. It fits naturally alongside the other quiet services that make up a personal server.

This is not a startup and not a demo. It is a thing I wanted, so I built it and left it running.

If you miss small servers, plain text, and systems that stay out of your way, you might enjoy talking to a language model over Gopher.

It's open source: https://github.com/someodd/small-gopher-applets

The source is a single README, which is a Literate Haskell file. You literally mark the Markdown file as an executable and run it.

Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/chatgopherpt.gopher.txt](gopher://gopher.someodd.zip:70/0/phlog/chatgopherpt.gopher.txt)
