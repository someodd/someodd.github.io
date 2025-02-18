---
date: 2024-10-14
tags:
- haskell
- code
title: Setup Haskell verification with LiquidHaskell and Stack

---
Original content in gopherspace: gopher://gopher.someodd.zip:Just 7071/phlog/


This article helps you get started with LiquidHaskell, which I found confusing.

I use LiquidHaskell's refinement types which allows you to...

formal verification is like property testing, but instead of n statistical samples providing evidence it's a solver providing proof

ghcid -c "stack ghci"

put this in header of file to doctest:

```
{-# OPTIONS_GHC -fplugin=LiquidHaskell #-}
```

the example project has stack examples fore ach version. it's very tricky to
add the right dependencies to stack so use the examples from the
repository--there's a stack file per ghc version.

https://github.com/ucsd-progsys/lh-plugin-demo/tree/main

https://github.com/ndmitchell/ghcid

https://ucsd-progsys.github.io/liquidhaskell/install/

i'm actually using regular haskell plugin

