---
date: 2025-11-17
tags:
- gopher
- games
- haskell
- my_warez
- my_servs
title: I Made a Multiplayer RPG Where You Explore Gopherspace With Others

---


An MMORPG where the game world is literally live gopherspace itself. Find treasure where people haven't been before, so the more gopherspace pioneering you do, the better your stats can be. You can leave your words in gopherspace, but also kill people and steal their loot.

The idea of this game is to encourage people to *really* explore gopherspace and to showcase how [Venusia](https://github.com/someodd/venusia) (my Gopher daemon) lets you, for example, run [Literate Haskell](https://wiki.haskell.org/index.php?title=Literate_programming) as scripts (`runghc`), to give rich application experiences in gopherspace.

[The game code (open source)](https://github.com/someodd/grpg) is literally a `README.md` -- literally try running it `./README.md`. This is part of a format I've been using I call "gopher applets," which is probably a more generally useful concept, about Markdown README files which are also the code for the game (even tests), so the documentation and code are all-in-one and it's all nicely self-contained.

Play the game by finding the right link at [the root of my gopherhole](/)

# More info

The script is very half-baked and will likely need to undergo many iterations. The biggest glaring issue is that there's no password system so you can literally just type in someone's username to hijack their account. I plan to implement tripcodes.

At the moment, I don't really care as I'm busy with other things, but I will get around to fixing that. This just started off as a proof-of-concept of my Gopher state system/script.

Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/gopherspace-rpg.gopher.txt](gopher://gopher.someodd.zip:70/0/phlog/gopherspace-rpg.gopher.txt)
