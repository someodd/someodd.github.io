---
date: 2024-10-01
tags:
- sysadmin
- linux
- archival
title: Torrents for data longevity

---


# Torrents for data longevity

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

Original content in gopherspace: gopher://gopher.someodd.zip:7071/phlog/
