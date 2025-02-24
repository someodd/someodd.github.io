---
date: 2024-11-29
tags:
- haskell
- code
- gopher
title: Formal verification and property testing of a custom algorithm

---


How do you know your code does what it says it does?

i'm also writing an article on the formal verification and property testing of an algorithm i made for the gopher protocol

[How to set up LiquidHaskell](gopher://gopher.someodd.zip:70/0/phlog/liquidhaskell-verification-stack.txt)

## A little rant about Haskell, correctness

Haskell It's such a different way of thinking fundamentally due to the alfonzo/turing split and this has lead to unique features that slowly bleed into the general language ecosystem. One of my favorite features of Haskell is exactness and correctness. The type system lends itself to not just catching the kind of errors that would be normally runtime errors in some other languages, but it it also lends itself to things like formal verification and proof assistants like...

... #LiquidHaskell (restriction types, SMT solver) and the Isabelle proof assistant.

For example, look at the kind of assurances we can make about data at the type-level. This means the SMT solver can assure that certain logical constraints on types are provably true.

My code to verify a file ranking algo I made:

https://github.com/someodd/bore/blob/master/src/Bore/SpacecookieClone/Search/Verified.hs

But also, I think Haskell was maybe one of the first places you saw property testing, and I think it's kind of the norm in this ecosystem. Just look at a unique way of thinking testing *properties* rather than just "dumb" unit tests are:

https://github.com/someodd/bore/blob/master/test/Spec.hs

What's happening here is a kind of statistical verification of *properties* that should prove true about the actual entrypoint/main functions that *do the thing*...

In this code above is I actually "verify" important properties of the important thing: file ranking works as expected.

It's explicitly NOT "if I give x input I get y output" style unit test. The "statistical verification" comes from the fact that you can programmatically generate inputs and verify the outputs, and thus generate >1,000 test "cases" from just one actual property test.

And this is because, I think, you're dealing with more abstract logic, which Haskell really lends itself to.

## more on the alonzo-church split

Original content in gopherspace: gopher://gopher.someodd.zip:7071/phlog/
