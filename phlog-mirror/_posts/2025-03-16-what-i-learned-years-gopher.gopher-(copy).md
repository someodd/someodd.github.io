---
date: 2025-03-16
tags:
- haskell
- gopher
title: What I learned writing Haskell code for a 33 year old pre-web Internet protocol
  (Gopher)

---


If you take a look at [my GitHub](https://github.com/someodd/) you'll notice two things:

1. It's pretty much all Haskell
2. Wow, there's a lot of stuff written for "the Gopher Protocol," what even is that?

I had encountered The Gopher Protocol as a child, but never really got to know it until the last few years. I had heard about it from a friend who was describing a *debloated* and organized net. I'd come to learn that's fairly accurate, especially considering both how Gopher is mostly simple text and how it was even involved in some librarian conferences.

A programmer I looked up to championed Haskell. In fact, lots of programmers I thought were cool were using Haskell. It tickled my fancy when it came to craft and the esoteric. I think really what started my journey was finally being fed up with Objected-Oriented Programming and (inconsistent) mutability. I also didn't like a lot of the conventions that were going on in that sphere, I often felt like people were twisting their code to be more sophisticated than just accepting the brutalist aesthetic of the fact that they were dealing with structured data and ways of mutating that data, which I would come to learn Haskell really drove home.

**work on this**
I started dabbling with the Gopher Protocol to learn Haskell. I'm a bit of a techno-romanticist. I like to host my own services on my server. Even old stuff. In fact, I think there's a lot of wisdom in the old stuff. Of those unix legends. Those old programmers I looked up for so long. I remember for a long time in programming, before AI kicked off, it really did feel like everything was just reinventing the wheel and you really could just use a tool made or started 30+ years ago to do "the latest thing." I really enjoyed the idea of such stable and historical software that was so practical. I really loved running my own software, writing my own code, fusing craft and art with it all (even with the concept of server-as-an-art-installation). It also was a gateway to interesting friendships. In many ways I still feel this holds true. I like the BSDs, I like Debian, I like the idea of code as a craft, or even code and serveradmin as an art, for example:

* netring
* I even like running Window Maker sometimes, swith all kinds of arts dockapps (link here, i should really update that shit with pictures too)
* whisper-radio: an internet radio station (icecast) which reads the latest news in gopherspace, mastodon, responds to users, plays music, reads text

However, I think over the years, for one reason or another, my enthusiasm waned, my ability to hold such a romantic view, have such a romantic relationship with tech waned. IT all just became a tool, a utility, plain and simple.

My last attempt at rekindling this started with when I found Haskell and the Gopher Protocol.

I found Haskell wasn't just one of these things I could quickly learn by doing--I actually felt the need to do a little bit of book learning.
I decided to really start understanding Haskell, after reading the obligatory [Learn You a Haskell for Great Good!](https://learnyouahaskell.com/), by finally getting around to starting a project. I decided to make a text UI client for the Gopher Protocol, which would speedrun me understanding both, also incurring heavy tech debt and mistakes along the way, but that's what learning projects are for!

I might use some terms:

* gopherspace: think of this like "the world wide web" but a bizzaro universe version for the gopher protocol
* gopherhole: like a bizzaro universe version of a "website," but for the gopher protocol
* menu: a file presented, basically, as a page of links, like a webpage

## First real Haskell project: Waffle

This all started before people were relying on Chat gippity to do all the thinking for them.

It's funny to think that to learn the protocol, I didn't even start by using Gopher much at all, I just started diving deep into the protocol, like looking at the Wikipedia entry for the protocol. I learned about how basically everything in gopherspace is a file, the client doesn't really know what type the file is, unless whatever link brought you there tells you what kind of file it is. There's one very special kind of file called a menu or "gophermap," but I'll stick to calling them "menus." Now menus, at this point in time, were where the meat was at. They had an interesting way of formatting. A menu is formatted like this: `$itemChar$Label\t$selector\t$host\t$port`:

* `$itemChar`: the protocol defines various one-character item types, which describe the actual type of file being linked to. IN a sense, you basically have no other way of telling what a file in gopherspace is (there's no mimetype header liek HTTP or anythign like that, you could use magic numbers/strings or whatever they're called). However, with URIs, you can define this $itemChar in the URI so the client easily knows how to interpret the file it's accessing, e.g., `gopher://gopher.someodd.zip:70/I/assets/posts/richmond-library/extension-staircase-atrium.jpg` (here the `/I/` bit helps the client know it's an image). See also, [RFC 4266](https://datatracker.ietf.org/doc/html/rfc4266). There are a list of item types, like `0` for a plain ol' text file, `1` for another menu, `7` for index search (we'll get to what that is later), etc. People even extended the item types later on, but I like to just think of it as: search, plain text, menu, or some kind of download, as far as I was concerned.
* `$Label`: this is the text that will be displayed for the menu item
* `$selector`: just a path that accesses some file on the gopher server, very similar to to HTTP
* `$host`: the hostname/ip through which to attempt to access said file
* `$port`: the port to use when making a request to `$host`

Here's an example of a menu file that gets interpreted by the client:

```
0Here's a label for a text file	/some/path/to/file.txt	example.org	70
1Label for some menu	/path/to/some/menu	place.example.com	7070
```

There were other things added later on, I've seen called "non-canonical types," to make the Gopher protcol more useful in more modern contexts, like `h' to open web URLs.

But now these simple at-a-glance examples weren't enough. I could render such a format as an actual menu where you could move through the options, but I actually needed to start looking at the protocol specification--RFC 1436--it was my first time actually implementing something from the ground-up based off a protocol specification like that.

As I started implementing details of the protocol specification I started running into problems and things I hadn't thought of. One major issue I encountered was that bizarrely, for this protcol which was proving to be so charmingly simple (a reason I think it's being adopted by the smallnet movement), my software was *slow* on decently sized menus I was encountering in gopherspace. I had to start profiling why Brick, a text UI library for Haskell, was so slow when scrolling the menu--I could actually hear the fans spin up on my laptop, which is pretty funny. Thankfully, the functional programming community seems pretty supportive--as I began to look into profiling my client someone pointed me to their tool which made reading the profiling output much more legible--namely [profold](https://github.com/Garmelon/profold). It turned out I was using certain TUI elements in a very foolish way and had to chose a TUI element that was much more suited to scrolling through a long list of lines, that could select the "current line" (so when I hit enter, it would open the menu item).

I also had to learn about managing the application state, something I didn't really initially do well (I just passed around a giant state data type). I could've just used a state monad, but I think I was afraid of monads and especially monad transformers, especially when all this stuff with concurrency and IO was going on. Honestly it wasn't the worst choice. I still kind of find monad transformers to be kind of annoying. I probably would've just made things more loosely coupled. The project turned into what was basically a bizarre state machine with at least a clear separation of responsibilities in some aspects.

## After Waffle: a few projects

I also worked on some other tools for gopher, around this time I started thinking about ways I could abuse the protocol (largely thanks to the indexsearch feature being used for user input), because of my intimate knowledge of the protocol and perhaps because of seeing things like games in gopherspace, myself:

* gopherdashboard: A tool which... I think this tool was maybe the one to really introduce me to the idea of creating service daemons, and packaging them for Debian so my software could actually be... useful.
* phorum: a forum, but for the Gopher Protocol.
* burrow: like a jjj

I started playing around with Nix and Nixpkgs around here.

I did some whacky project involving docker and burrow and a docker setup for making a git repo that will on hook update the gopherhole/serve it: https://github.com/someodd/docker-spacecookie/tree/feature/git-server-burrow

## After all that: bore

I stopped using nixpkgs and started using Stack, something I avoided for so long, but made my life so much easier.I wanted to start getting more practical in general. I learned the basics of releasing/packaging and creating daemons for Debian with Haskell, and making more reliable software.

I might save some of this for another article that gets into the specifics, but I did some fun stuff liek refinement types (formal verification) for deductive proofs and started looking into proof assistants, and started doing property-based testing i.e., inductive proofs. For now I highly recommend this article by [MVT I have stored in gopherspace](gopher://gopher.someodd.zip:70/0/archives/mvt_joy_archive/articles/a_conversation_with_manfred_von_thun.txt), which i found very inspiring, because I find the man and his work so incredibly interesting. He actually has a very concrete way of thinking about functional programming and language design in general--he was a philosophy professor that leveraged his philosophy of logic (or whatever its called) in computer science and language design and made him one of the (if not the most) respected computer science figures on campus.

I used LiquidHaskell for refinement types (formal verification, deductive proofs) to verify properties of my search implementation/file ranking algorithm. This works by using an SMT solver to prove using logical deduction that some property you write using a refinement type will actually hold true. For example, if you say "this will return a percentage represented as a float between 0.0 and 1.0," but all Haskell is checking for with its type system is if it's a float, you can write a refinement type that will say the value will indeed be between 0.0 and 1.0 and the solver will check your logic, using deductive reasoning, to ensure that will always hold true.
This was such a pain-in-the-ass to set up, though. I think I wrote an article about it on my phlog.

This project was the first time I had a wild, fun time using QuickCheck for property-based testing. This can be thought of as inductive proof. I used this to verify that my search results will always have certain properties I desire (the search results should always look like x under y circumstances and the like). Think of it like this: instead of writing...

## Conclusion

The Gopher Protocol gave me a very simple way to experiment with all these academic concepts in haskell and bring them down into something more pragmatic that I could show the world.

I'm actually starting a documentary about the gopher protocol and its revival/indieweb/smallnet movement (link to it here) for The MADE.

i got to make a lot of connections i treasure. I got to meet some of my idols, including the gopher team, I got to reach out to people in smallnet, the wider gopher community, make relationships in the "fediverse" (namely mastodon), interact and work with other people in open source, I got to meet the bitreich people, I got to share my smallnet services with people, people are actually on my XMPP/IRC server now, come join us:

* Via XMPP: [xmpp:%23main%25irc.someodd.zip@irc.xmpp.someodd.zip?join](xmpp:#main%irc.someodd.zip@irc.xmpp.someodd.zip?join)

* Via IRC: ircs://irc.someodd.zip/#main

I also feel like all of this has me doubling-down on my sense of romanticism and artfulness and craft when it comes to technology, after having long since drifted apart from it. I'm very happy about hosting my own server, with all its little services, a boutique server with boutique code made for special people.

## Special thanks

Bitreich

original gopher team

mvt

Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/what-i-learned-years-gopher.gopher (copy).txt](gopher://gopher.someodd.zip:70/0/phlog/what-i-learned-years-gopher.gopher (copy).txt)
