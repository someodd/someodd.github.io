---
date: 2024-11-18
tags:
- sysadmin
- linux
- hardware
- server
title: Making my server quieter

---
Original content in gopherspace: gopher://gopher.someodd.zip:Just 7071/phlog/


Entered BIOS and looked for acoustic/power/performance settings which led me to
some way to reduce fan intensity in BIOS.

What I installed and did that helped:

```
sudo apt install cpufrequtils
sudo cpufreq-set -g powersave
```

I could've used fancontrol + pwmconfig if my server supported it.

I decided to unplug all fans but the CPU fan and just use `cpufreq-set` to
powersave. Check current mode with `cpufreq-info` to make sure it's set to
`powersave`.

I think it's already consistent across boots from the command I tried?

I checked temps with `sensors`.

