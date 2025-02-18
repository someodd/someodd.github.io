---
date: 2024-03-10
tags:
- sdl
- haskell
- game
- nix
- showcase
title: 'haskellcard: hypercard-like game engine'

---


[haskellcard](https://github.com/someodd/haskellcard) an SDL2 game engine based
around the idea of exploring different scenes. Ended up being similar to
HyperCard. Kind of a point-and-click type thing, but I wanted to have a lot of
real-time support, since I think feel like point-and-clicks are kind of boring
or not very fun.

The first release was Christmas day, I think.

Neat parts:

  * Build with Nix!
  * Develop in Nix environment!
  * JSON-configurable game logic, rendering, rooms, objects, sfx, and more!
  * Music queue system, so can loop, but also wait for a loop to finish before
    next song plays
  * Tweening system

Original content in gopherspace: gopher://gopher.someodd.zip:7071/phlog/
