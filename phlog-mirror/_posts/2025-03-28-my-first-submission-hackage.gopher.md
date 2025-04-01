---
date: 2025-03-28
tags:
- haskell
- code
title: Becoming a Hackage uploader

---


I was working on [Venusia](https://github.com/someodd/venusia) because I keep writing Gopher servers. I realized I wanted to just create a library/dependency. So I wanted to actually put this on Hackage, which is Haskell's package repository.

I had to go through an approval process and I'm now a Hackage uploader!

## Prepping my project and then getting it uploaded!

These are some messy notes, but I hope it's still useful for you! This is what I did, roughly, to prepare Venusia for Hackage and getting it on Hackage.

I edited my `package.yaml`. Then I used these commands:

```
cabal check
stack sdist
```

That command might suggest you use upper bounds. I lazily used `cabal gen-bounds`. I had to wrestle with the bounds because stack complained about them not being available in the snapshot. I made one of the bounds more relaxed.

This also generated a file `/home/tilde/Projects/Venusia/.stack-work/dist/x86_64-linux/ghc-9.8.4/Venusia-0.1.0.0.tar.gz` which I uploaded as a package candidate. I'll get back to that later.

Registered for an account: https://hackage.haskell.org/users/register-request

I then emailed the hackage trustees to get approved, although they had an option to send people an endorsement link, and someone responded rather quickly. They gave me some notes about the first package I wanted to submit (one of the requirements they listed for asking to be approved).

he told me of these notes:

> 1. Hackage is intended to be a permanent record. Therefore uploads
cannot be changed or removed.
> 2. Only upload things that work, are useful to other people, and that
you intend to maintain.
> 3. Use `cabal gen-bounds` to put PVP-compliant version bounds (lower
> AND upper) on ALL your unique dependencies so your package will still
> be buildable years down the road. One important thing to note is that
> you only need to include version bounds once. For example, if you
> depend on the same package in your library and your test suite, you
> only need to put the version bounds for that dependency in one place.
> This keeps the dependency bounds information DRY.
> 4. Package candidates CAN be changed, so use them to test things out
> and get everything right before you publish permanently to the main
> index.

You should check that your source bundle builds, including the Haddock documentation if it's a library. 

Here's my candidate, for now, which may be a dead link: https://hackage.haskell.org/package/Venusia-0.1.0.0/candidate

Be careful, Hackage, at least when defining a dependency, is case-sensitive! This really threw me off! I feel like this is actually really bad and has potential to be exploited if, say someone uploads a package of the same name, just lowercase, if it's capitalized, or something like that. Hopefully they prevent that somehow.

I couldn't figure out how to install a candidate dependency using Stack, so in the project I wanted to use Venusia as a depnency in I just used:

```
extra-deps:
- git: https://github.com/someodd/venusia.git
  commit: ffae72d03d3b24e7507354071490f539868c6821
```

Oh also i think i may have had to create a `src/` for my project? Maybe that was just a warning.

## Haskell's versioning: PVP

Package Versioning Policy and have version bounds on all dependencies indicating which versions they are known to build with. See: [Package versioning and curation](https://hackage.haskell.org/upload#versioning_and_curation) and [Haskell PVP Specification](https://pvp.haskell.org/). Something that really helped me was in the Haskell PVP FAQ ["How does the PVP relate to SemVer"](https://pvp.haskell.org/faq/#semver)?

I took the time to understand the Haskell PVP Spec and the section on "package
versioning and curation." I think I understand, mostly, now, that the spec is
about not breaking other people's code, but also describing how much it might
break code (almost like an estimate of how long it'd take to adapt new code) as
well as signaling if there were non-api-breaking features/changes, for the most
part. So in that sense for A.B.C, 'a' seems to be like "super major" like "the
API changed DRASTICALLY" and 'b' is like "major" as in "this breaks the API and
might break your code" and 'c' is for most other things that don't do either of
those things (like feature additions that don't change existing API). Of course
I'm glossing over all the small things like orphaned instances. I read the FAQ
on SemVer vs. PVP. Also there's basically no "releasing something as unstable"
like you might do with 0.1.x in semver, and reaching 1.x.x in PVP is mostly a
marketing thing (considered "mature")--so basically you're always tracking API
changes. I actually really like this.

## Package candidates

Package candidates sound interesting and so do the main guidlines:

> The hackage home page that contains some of the main guidelines for
> Hackage packages:
> https://hackage.haskell.org/

> Information about package candidates can be found here:
> https://hackage.haskell.org/packages/candidates/

Package candidates are important to learn about since you cannot delete/modify uploads that aren't package candidates. https://hackage.haskell.org/upload#candidates -- I don't fully understand but it seems like you just manually upload a candidate tar...

Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/my-first-submission-hackage.gopher.txt](gopher://gopher.someodd.zip:70/0/phlog/my-first-submission-hackage.gopher.txt)
