---
date: 2024-11-16
tags:
- sysadmin
- linux
- debian
- networking
title: Dynamic DNS in Linux (Debian)

---
Original content in gopherspace: gopher://gopher.someodd.zip:Just 7071/phlog/


I used to use DuckDNS, but then I started using afraid.org.

I just moved and it was nice knowing that my server would just update the new
IP/location for me.

In a script like `~/duckdns/duck.sh`:

```
sleep 21 ; wget -O - http://freedns.afraid.org/dynamic/update.php?YORTOKEN >> /tmp/freedns_someodd_mooo_com.log 2>&1 &
```

Something like the above command, that you get from Afraid.org.

You can crontab:

```
*/5 * * * * ~/duckdns/duck.sh >/dev/null 2>&1
```

You could also use systemd. Above does every five minutes.


