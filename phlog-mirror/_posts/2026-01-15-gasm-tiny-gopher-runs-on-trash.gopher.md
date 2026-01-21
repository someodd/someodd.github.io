---
date: 2026-01-15
image:
  caption: A whacky photo of me showing the file size of the gasm binary.
  path: /assets/phlog/posts/gasm/gasm-showing-size.jpg
  thumbnail: /assets/phlog/posts/gasm/gasm-showing-size.jpg
tags:
- linux
- gopher
- my_warez
title: 'gasm: 1.5 KB Gopher server that runs on 24 KB of RAM'

---


I wanted to find the absolute floor.

We know that abstractions cost resources, but I wanted to quantify exactly
how much. So, I wrote a fully functional Gopher server in pure i386
assembly with zero dependencies—no libc, no runtime, just raw `int 0x80`
syscalls.

The result is GASM.

![A whacky photo of me showing the file size of the gasm binary](/assets/posts/gasm/gasm-showing-size.jpg)

[Whacky photo of gasm's binary size](gopher://gopher.someodd.zip:70/I/assets/posts/gasm/gasm-showing-size.jpg)

By manually managing the ELF headers, disabling page alignment, and
using the legacy Linux `socketcall` interface, the final efficiency
metrics are distinct:

* Binary Size: 1,512 bytes (static, stripped)
* RAM Usage:   24 KB (Verified RSS via pmap)
* Deps:        None.

It serves Directory listings (Type 1), Text (Type 0), and Binary (Type 9).
It implements bounds checking and traversal blocking manually, without
the overhead of a standard library.

Because it targets the original i386 instruction set, it runs natively on
everything from a 1985 386DX to a modern Ryzen 9.

It is not a replacement for Nginx. It is a study in how much computer
you actually need to serve a file.

The source is available for audit.

[Source Code and Download (GitHub)](https://github.com/someodd/gasm)

Original content in gopherspace: [gopher://gopher.someodd.zip:70/1/phlog/gasm-tiny-gopher-runs-on-trash.gopher.txt](gopher://gopher.someodd.zip:70/1/phlog/gasm-tiny-gopher-runs-on-trash.gopher.txt)
