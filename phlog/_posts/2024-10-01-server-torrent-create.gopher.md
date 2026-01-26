---
date: 2024-10-01
tags:
- sysadmin
- linux
- archival
title: Torrents for data longevity

---


Creating torrents for data longevity, a distributed way to archive data.

I created an archive of an artist's content and wanted to ensure it's widely available and preserved for longevity.

This is written from the context of a Debian server.

## Torrent

I'm aiming to make a magnet torrent to share, but also to make the torrent widely available on a variety of trackers.

### Dependencies

```
sudo apt install transmission-cli
sudo apt install mktorrent
sudo apt install transmission-daemon
```

### Public torrent trackers

Trackers help make your torrent known to the outside world. You do not necessarily need trackers if you create a magnet link for a torrent, but it may make the torrent more accessible.

These trackers are open and do not require any authentication:

```
udp://tracker.openbittorrent.com:80/announce
udp://tracker.opentrackr.org:1337/announce
udp://tracker.coppersurfer.tk:6969/announce
udp://tracker.internetwarriors.net:1337/announce
udp://9.rarbg.to:2920/announce
udp://tracker.leechers-paradise.org:6969/announce
udp://explodie.org:6969/announce
udp://tracker.torrent.eu.org:451/announce
udp://tracker.moeking.me:6969/announce
udp://p4p.arenabg.com:1337/announce
udp://tracker.pirateparty.gr:6969/announce
udp://opentracker.i2p.rocks:6969/announce
udp://tracker.cyberia.is:6969/announce
udp://open.stealth.si:80/announce
```

### Create the torrent

First create and change to a directory to contain torrents you create:

```
mkdir /home/someuser/mytorrents
cd /home/someuser/mytorrents
```

This command will create a torrent file in the current directory and share it among a variety of public trackers, the command may take a while depending on how large:

```
mktorrent -a udp://tracker.openbittorrent.com:80/announce,udp://tracker.opentrackr.org:1337/announce,udp://tracker.coppersurfer.tk:6969/announce,udp://tracker.internetwarriors.net:1337/announce,udp://9.rarbg.to:2920/announce,udp://tracker.leechers-paradise.org:6969/announce,udp://explodie.org:6969/announce,udp://tracker.torrent.eu.org:451/announce,udp://tracker.moeking.me:6969/announce,udp://p4p.arenabg.com:1337/announce,udp://tracker.pirateparty.gr:6969/announce,udp://opentracker.i2p.rocks:6969/announce,udp://tracker.cyberia.is:6969/announce,udp://open.stealth.si:80/announce -o some.torrent /path/to/something
```

You can use this command to generate a magnet link:

```
transmission-show -m music.torrent
```

## Start seeding, server-style

Now you'll want to start seeding the torrent, although you'll want to daemonize it for server purposes:

```
transmission-cli -w /path/to/something /home/someuser/mytorrents/some.torrent
```

You don't want to actually use the above command. Let's get started with a server-style seeding of the torrent. Assuming the `transmission-daemon` is running:

```
transmission-remote -a /path/to/torrent/file.torrent -w /path/to/existing/files --auth username:password
```

You can find your `username` and `password` from `/etc/transmission-daemon/settings.json` as `rpc-username` and `rpc-password`. You could use `sudo cat /etc/transmission-daemon/settings.json | grep rpc`. I couldn't get this to work right so I just disabled the username and password since only localhost can access, anyway. I set `"rpc-authentication-required": false,` after i did `sudo systemctl stop transmission-daemon`, but after the edit i did `sudo systemctl start transmission-daemon`.

Then the command `transmission-remote -a vaervaf-soundcloud-archive-2024-01-08.torrent -w /media/root/BackupRAID/sftpbackups/backup_coffee/backups-manual/vaervaf-soundcloud-archive-2024-01-08` seemed to work.

## troubleshooting

Use `transmission-remote -l` to check.

You may run into some issues. try `transmission-remote -t 1 --verify`. you may also have permission errors... i ended up adding debian-transmission to a specific group so it could access my archive directory. `sudo usermod -aG sftpbackups debian-transmission`. then `sudo systemctl restart transmission-daemon`. you can even use something like, to test, `sudo -u debian-transmission ls -l /media/root/BackupRAID/sftpbackups/something`. also try `transmission-show vaervaf-soundcloud-archive-2024-01-08.torrent` to see the directory structure. i think weirdly it ended up expecting that `-w` passes the *parent* directory, not the actual directory i made a torrent out of? removed with `transmission-remote -t 1 -r` then re-add with `transmission-remote -a vaervaf-soundcloud-archive-2024-01-08.torrent -w /media/root/BackupRAID/sftpbackups/backup_coffee/backups-manual/`. then check with `transmission-remote -l`.

now i'm still having issue where people can't download. so `transmission-remote -t 2 -pi`. my port is set to `51413`. i used:

```
➜  ~ nc -zv 192.168.1.100 51413
simulacra.local [192.168.1.100] 51413 (?) open
➜  ~ nc -zv gopher.someodd.zip 51413
DNS fwd/rev mismatch: someodd.mooo.com != c-73-93-14-18.hsd1.ca.comcast.net
someodd.mooo.com [73.93.14.18] 51413 (?) : Connection refused
```

clearly i need to forward port on router and also open it on ufw.

```
sudo ufw allow 51413/tcp comment 'Allow Transmission-daemon TCP'
sudo ufw allow 51413/udp comment 'Allow Transmission-daemon UDP'
```

and maybe forward on router.

## ensure published to trackers

```
transmission-remote -t <torrent-id> -it
transmission-remote -t <torrent-id> --tracker-announce
transmission-remote -t <torrent-id> -it
transmission-remote -t 1 --reannounce
transmission-remote -t 1 -it
```

## More tips

You may want to edit `/etc/transmission-daemon/settings.json` and set `lpd-enabled` to `true` if you want to test downloading the torrent via local network (or even same VPN?).

Make sure the torrent status isn't `stopped`--it needs to be `idle` at worst to actually start seeding again.

I'll try to post my actual settings.json soon.

transmission-remote -t all --start

## my config

note that you need to make sure transmission-daemon has stopped or it will just overwrite the settings

look out for seed-queue-size, you may wanna disable that.

```
{
    "alt-speed-down": 50,
    "alt-speed-enabled": false,
    "alt-speed-time-begin": 540,
    "alt-speed-time-day": 127,
    "alt-speed-time-enabled": false,
    "alt-speed-time-end": 1020,
    "alt-speed-up": 50,
    "bind-address-ipv4": "0.0.0.0",
    "bind-address-ipv6": "::",
    "blocklist-enabled": false,
    "blocklist-url": "http://www.example.com/blocklist",
    "cache-size-mb": 4,
    "dht-enabled": true,
    "download-dir": "/var/lib/transmission-daemon/downloads",
    "download-limit": 100,
    "download-limit-enabled": 0,
    "download-queue-enabled": true,
    "download-queue-size": 5,
    "encryption": 1,
    "idle-seeding-limit": 30,
    "idle-seeding-limit-enabled": false,
    "incomplete-dir": "/var/lib/transmission-daemon/Downloads",
    "incomplete-dir-enabled": false,
    "lpd-enabled": true,
    "max-peers-global": 200,
    "message-level": 1,
    "peer-congestion-algorithm": "",
    "peer-id-ttl-hours": 6,
    "peer-limit-global": 200,
    "peer-limit-per-torrent": 50,
    "peer-port": 51413,
    "peer-port-random-high": 65535,
    "peer-port-random-low": 49152,
    "peer-port-random-on-start": false,
    "peer-socket-tos": "default",
    "pex-enabled": true,
    "port-forwarding-enabled": false,
    "preallocation": 1,
    "prefetch-enabled": true,
    "queue-stalled-enabled": false,
    "queue-stalled-minutes": 30,
    "ratio-limit": 2,
    "ratio-limit-enabled": false,
    "rename-partial-files": true,
    "rpc-authentication-required": false,
    "rpc-bind-address": "0.0.0.0",
    "rpc-enabled": true,
    "rpc-host-whitelist": "",
    "rpc-host-whitelist-enabled": true,
    "rpc-password": "ohfug:DDDDD",
    "rpc-port": 9091,
    "rpc-url": "/transmission/",
    "rpc-username": "transmission",
    "rpc-whitelist": "127.0.0.1",
    "rpc-whitelist-enabled": true,
    "scrape-paused-torrents-enabled": true,
    "script-torrent-done-enabled": false,
    "script-torrent-done-filename": "",
    "seed-queue-enabled": false,
    "seed-queue-size": 10,
    "speed-limit-down": 100,
    "speed-limit-down-enabled": false,
    "speed-limit-up": 100,
    "speed-limit-up-enabled": false,
    "start-added-torrents": true,
    "trash-original-torrent-files": false,
    "umask": 18,
    "upload-limit": 100,
    "upload-limit-enabled": 0,
    "upload-slots-per-torrent": 14,
    "utp-enabled": true
}
```

Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/Server Torrent Create.gopher.txt](gopher://gopher.someodd.zip:70/0/phlog/Server Torrent Create.gopher.txt)
