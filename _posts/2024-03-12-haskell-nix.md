---
layout: post
title: "Nix Flake a Haskell Project"
date: 2024-03-12
categories: notes
tags: haskell nix
---

I wrote these notes because I wante to reduce the friction for myself and
others to getting a Nix Flake working for their Haskell Project.

Some of these tips might be useful for Haskell in general.

* TOC
{:toc}


## Why use a Nix Flake?

  * Reproducibility.
  * nix build
  * nix develop

## Resources

https://input-output-hk.github.io/haskell.nix/tutorials/getting-started.html

https://input-output-hk.github.io/haskell.nix/tutorials/getting-started-flakes.html

## Installing nix

https://nixos.org/download#nix-install-linux

Then start a new terminal.

Add this to `~/.config/nix/nix.conf`:

```
experimental-features = nix-command flakes
```

Now add caching if you want to avoid taking forever, to your `/etc/nix/nix.conf`:

```
trusted-public-keys = cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY= hydra.iohk.io:f/Ea+s+dFdN+3Y/G+FDgSq+a5NEWhJGzdjvKNGv0/EQ=
substituters = https://cache.nixos.org/ https://cache.iog.io
```

Be sure to pay attention to things it's building when it should just be
grabbing from cache.

Whenever you modify `/etc/nix/nix.conf` you may want to use `sudo systemctl
restart nix-daemon`.

Although, it seems cabal is being built, so I'm not sure if it's actually using
the cache.

## Start setting up your project/flake

We're going to use this: https://community.flake.parts/haskell-flake/start

I'm going to be going off the example of adding a nix flake to an old-ish project of mine.

In your project repo/root:

```
nix flake init -t github:srid/haskell-flake
```

Edit the new `flake.nix` to set `packages.default` to something like ` packages.default = *self'*.*packages*.yourproject;` .

### Use a specific GHC

I came back to a project (burrow) that was difficult to compile because I neglected to pin the GHC version and dependency versions or whatever. My goal was to modify the project as little as possible while introducing nix to build it and making compilation in general more stable for longevity and other users.

My project used:

```
    base                 >=4.7 && <5,
```

And Here's a list where you can tell which version of GHC you may want to use based off your `base` constraints: https://wiki.haskell.org/Base_package

This tells me it can use anything from 7.8.1 to (as of now) 9.8.1. Based off of deduction of APIs I kenw I had to set the versions of a few dependencies in my cabal file, basically, too. However, one of the dependencies  (`errata`) I had to pin wanted something between `4.12` (GHC 8.6.1) and `4.17` (GHC 9.4.1). I changed my `base` constraints around this in the hopes it'd make things easier.

Things to pay attention to in the `flake.nix` generated:

* Is the cache being used?
* `nixpkgs.url`: this may come in handy: https://lazamar.co.uk/nix-versions/
* `basePackages`

I also found out a package I was using is broken in nixpkgs right now, so I just added it myself and removed it from dependencies (a library to convert a number to a roman numeral).

Another issue I encountered was I needed a specific version of errata, basically, based off the API.

I also searched nixpkgs to see which version holds errata 0.3.0.0 so I could set `nixpkgs.url` basically, luckily the flake made this easy:

```
          packages = {
            errata.source = "0.3.0.0"; # Hackage version override
            # shower.source = inputs.shower;
          };
```

after searching for the right errata version in https://lazamar.co.uk/nix-versions/ -- I found it in nixos-22.11. After doing so I think it was clear that I should use something older and more stable than the nixpkgs `unstable`, so I did this:

```
    nixpkgs.url = "github:nixos/nixpkgs/nixos-22.11";
```

Even so I still have this left over:

```
hspec-golden >=0.1 && <0.2
```

Which I found strange because of these results: https://lazamar.co.uk/nix-versions/?channel=nixos-22.11&package=hspec-golden

So i just made sure to set to the latest version that comes before 0.2:

```
          packages = {
            errata.source = "0.3.0.0"; # Hackage version override
            hspec-golden.source = "0.1.0.3";
          };
```

I then got a complaint about missing `word-wrap ==0.4.1`, which was in `nixos-22.11`, so I did the same basically:

```
          packages = {
            errata.source = "0.3.0.0"; # Hackage version override
            hspec-golden.source = "0.1.0.3";
            word-wrap.source = "0.4.1";
          };
```

Here's where I learned that `0.4.1.0` does not equal `0.4.1` to nix or something... anyway after this the project built!

I made sure that there was a `flake.lock` and I also did a `cabal freeze` inside of `nix develop` and then I tried using cabal to build inside and outside of `nix develop`:

1. `nix develop`
1. `cabal build`
1. `cabal freeze` -- although if i freeze inside nix develop then i can't build with cabal outside and vice-versa!
1. `ghc --version`: note the version
1. exit `nix develop`
1. Use `ghcup` to set the GHC version to the version shown earlier
1. `cabal build`

Done!

I actually had a bunch of trouble trying to run `cabal build` *outside* of `nix
develop`, because I believe `cabal` kept preferring to use already installed
packages that weren't compatible.

It turns out that for some reason with ghcup I'm restricted to one version of
base, yet I can cabal build with ghc 9.0.2 in develop just fine, so I ended up
restricting the ghc version to 8.10.7 because it's the latest before 4.15 is
hit. All because of the table layout library or whatever. I was able to build once I set my ghcup GHC version to 8.10.7. So I decided to also use that in my `flake.nix`:

```
basePackages = pkgs.haskell.packages.ghc8107;
```

Actually I decided NOT to use the above because it will take SUCH A LONG
TIME/not use the cache.

I'm not sure why I could see the GHC version in my `nix develop` was `9.0.2`
and could access the earlier `base` dependency, yet outside of `nix develop` it
could not.

I set this in my `burrow.cabal`:

```
tested-with: GHC==8.10.7
```

I also created a `cabal.project` (weirdly the order of the fields mattered, I think, for `nix develop`):

```
packages:
    ./.
with-compiler: ghc-8.10.7
```

Although the `with-compiler` will break buildling in nix develop? I had lots of
weird issues with `nix-develop`, like it takes so incredibly long to launch
(not caching, yet `nix build` does?).

Then after all that I advice checking doing `cabal freeze` in `nix develop` and
rebuilding.

after a lot of troubleshooting trying to get cabal freeze to work in both nix develop and regular cabal i noticed the cabal produced by regular had later zlib version than nix, so I tried setting the nix version in the nix file

don't forget to use your `ghcup` install carefully (using right ghc).

In the end I decided not to worry about `cabal freeze`.

## VS Code

direnv (carefully follow instructions) and add this to project root (`hie.yaml` I guess so HLS knows to use cabal not stack or whatever):

```
cradle:
  cabal:

```

i think this file above helps HLS detect that you're only using cabal and not using Stack (you may get errors about cradle otherwise). generally helpful maybe

install these plugins to vscode: 

* direnv by cab404
* haskell by haskell
* i think nix env selector doesn't work with flakes?
* nix ide

This link (reddit) was helpful: https://www.reddit.com/r/NixOS/comments/v23c3k/how_are_nix_flakes_used_in_vscode/

```
echo use flake > .envrc
direnv allow
```

## Crazy caveat

I ran into this weird issue where when I try to `nix build` it'd complain about
a file not existing,e ven though I could `cabal build` and it'd detect the
source file just fine. It turns out I had to `git add` the file for nix to pick
up on the file?!
