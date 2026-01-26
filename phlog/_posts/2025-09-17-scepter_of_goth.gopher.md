---
date: 2025-09-17
image:
  caption: Handmade map of the world of Scepter of Goth.
  path: /assets/posts/scepter_of_goth/ScepterMap02.jpg
  thumbnail: /assets/posts/scepter_of_goth/ScepterMap02.jpg
tags:
- gaming
title: Scepter of Goth

---


I had the pleasant experience of playing the first MMORPG with one of the people involved, who also happens to be a co-author of The Internet Gopher Protocol: Bob Alberti. I met him because I interviewed him for a documentary.

## Bonus material

Here are some images I got from Bob Alberti himself, including handmade maps.

![handmade map 01](/assets/posts/scepter_of_goth/ScepterMap01.jpg)

![handmade map 02](/assets/posts/scepter_of_goth/ScepterMap02.jpg)

![some other map](/assets/posts/scepter_of_goth/001_Boldhome-Jroldham.jpg)

An open source version: https://github.com/shelches/scepter

## Review of stats!

[Looking at code of a 40+ year old MMORPG to minmax my fighter.](https://fosstodon.org/@someodd/115222607021233114)

this game seems to me a bit brutal.

The code: https://github.com/shelches/scepter/

line where character sets stats on creation: https://github.com/shelches/scepter/blob/2e04898cd8c523fbad849d5da86b9b421a3a77d0/log.c#L141

dex dodging, attacking first, running?

piety: resserection penalties? luck?

in the code it's abbreviated str, int, dex, pty, con

this isn't everything but it'll do (not all inclusive)

also idk which weapon skill is best for this kinda build

Looks like there are criticals? But unfortunately, it doesn't seem like there's any way to increase the chance, except for just leveling up?
https://github.com/shelches/scepter/blob/2e04898cd8c523fbad849d5da86b9b421a3a77d0/cmd3.c#L88

your weapons can break!

some kind of calculation for time until next allowed attack (cooldown)
https://github.com/shelches/scepter/blob/2e04898cd8c523fbad849d5da86b9b421a3a77d0/cmd3.c#L367

## Strength (str)

Affects your ability to carry heavier objects
https://github.com/shelches/scepter/blob/2e04898cd8c523fbad849d5da86b9b421a3a77d0/cmd1.c#L177
i think it's like str * 10

... and even steal heavier objects.
https://github.com/shelches/scepter/blob/2e04898cd8c523fbad849d5da86b9b421a3a77d0/cmd3.c#L1124

Also affects if your hit lands.
https://github.com/shelches/scepter/blob/2e04898cd8c523fbad849d5da86b9b421a3a77d0/cmd3.c#L72

Decreases hit cooldown/strike more often?
https://github.com/shelches/scepter/blob/2e04898cd8c523fbad849d5da86b9b421a3a77d0/cmd3.c#L367

## Constitution (con)

Affects vulnerability to monster's drain ability?
https://github.com/shelches/scepter/blob/2e04898cd8c523fbad849d5da86b9b421a3a77d0/matt.c#L364

Prevents you from permadeath!
https://github.com/shelches/scepter/blob/2e04898cd8c523fbad849d5da86b9b421a3a77d0/cmd2.c#L704

Involved in calculating if you're too exhausted to strike!
https://github.com/shelches/scepter/blob/2e04898cd8c523fbad849d5da86b9b421a3a77d0/cmd3.c#L276

## Dexterity (dex)

Affects logic for if the monster hits you or not (?)
https://github.com/shelches/scepter/blob/2e04898cd8c523fbad849d5da86b9b421a3a77d0/matt.c#L259

Affects chance of landing a hit
https://github.com/shelches/scepter/blob/2e04898cd8c523fbad849d5da86b9b421a3a77d0/cmd3.c#L160

Affects steal chance
https://github.com/shelches/scepter/blob/2e04898cd8c523fbad849d5da86b9b421a3a77d0/cmd3.c#L1075

## Piety (pty)

Your DM goes down when you wake up the DM?
https://github.com/shelches/scepter/blob/2e04898cd8c523fbad849d5da86b9b421a3a77d0/cmd4.c#L15

It seems like monsters will sometimes attack or not attack a player based off their piety, or something?
https://github.com/shelches/scepter/blob/2e04898cd8c523fbad849d5da86b9b421a3a77d0/matt.c#L153

Affects how much money you spawn with when you ressurect?
https://github.com/shelches/scepter/blob/2e04898cd8c523fbad849d5da86b9b421a3a77d0/cmd2.c#L742

Bless spell
https://github.com/shelches/scepter/blob/2e04898cd8c523fbad849d5da86b9b421a3a77d0/spells.c#L139

# my fighter build

You die in this game. A lot. I considered intelligence and piety two stats I
could leave at minimum, while for a fighter, strength, dexterity, and
constitution are the most important. The minimum you can have is 5, so that
leaves us with 45 points to divide amont our three selected stats.

* Strength: 18: land those hits (+3 bonus?), carry heavy, hit harder, hit faster/more often
* Intelligence: 5
* Dexterity: 9: dodge those hits (only a +1 bonus?)
* Piety: 5
* Constitution: 18, crucial for not permadeath and absorbing and enduring

Skills were a bit hard for me to reason about. I think I just will play the game to really figure out which skill/weapon type is best for which scenarios kinda...

Original content in gopherspace: [gopher://gopher.someodd.zip:70/1/phlog/scepter_of_goth.gopher.txt](gopher://gopher.someodd.zip:70/1/phlog/scepter_of_goth.gopher.txt)
