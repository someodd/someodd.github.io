---
date: 2024-12-04
tags:
- sysadmin
- linux
- debian
title: Out of disk space!

---
Original content in gopherspace: gopher://gopher.someodd.zip:Just 7071/phlog/


I'm out of disk space and need to figure out how to free up space.

Find out how full your memory is:

```
df -h
```

I figured out which drive I wanted to check out and so knowing that I do a command like this:

```
sudo du -hxd1 / | sort -hr | head -n 10
```

I could tell something wrong based off what it told me:

```
416G	/
219G	/root
147G	/home
25G	/usr
17G	/var
11G	/nix
19M	/etc
1.2M	/tmp
28K	/mnt
20K	/snap
```

Why is `/root` taking up so much space? Let's find out:

```
sudo du -hxd1 /root | sort -hr | head -n 10
```

Ah, I see it's from an old Monero project leftover:

```
219G	/root
217G	/root/.bitmonero
2.0G	/root/.cache
4.2M	/root/stenc
248K	/root/.rpmdb
164K	/root/.local
100K	/root/.config
20K	/root/.dbus
16K	/root/snap
8.0K	/root/.gnupg
````

I can pretty much delete `.bitmonero` without any concern. Yes, I made sure I cleared out my wallet.

