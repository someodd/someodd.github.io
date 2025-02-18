---
date: 2024-11-18
tags:
- sysadmin
- linux
- archival
- lto
- debian
- ltfs
title: Restic on LTFS

---
Original content in gopherspace: gopher://gopher.someodd.zip:Just 7071/phlog/


Archiving to tape with restic: how I processed 414980 files, roughtly 60 GiB in
about 10 minutes, with incremental updates too.

I've messed around with various solutions and this is just so simple and easy
to use.

My hardware:

* ...

using restic with ltfs: backup to an LTO drive the easy and fast way.

what im using

  * server running 12 i think (debian)

why this combo?

  * great way to do system snapshots! can store away. long shelflife.
  * cheap per TB
  * set and forget
  * incremental updates 
  * encryption
  * compression
  * ease-of-use and simplicity: restic makes backups quick and painless, but
    also LTFS removes a lot of the headaches associated with managing tapes

what first?

  * buy lto drive, maybe use sas connection

## prepare ltfs

https://github.com/LinearTapeFileSystem/ltfs/blob/master/README.md

install and list ssci devices:

```
sudo apt-get install lssci
sudo lsscsi -g 
```

may not need above because `ltfs -o device_list`

pay close attention to what's listed from command above.

install ltfs:


```
sudo wget https://mirrors.ustc.edu.cn/bjlx/pool/main/l/ltfs/ltfs_2.5.0.0~20240731-2_amd64.deb
sudo apt-get install ./ltfs_2.5.0.0\~20240731-2_amd64.deb
```

format the tape for ltfs or something:

```
sudo mkltfs -d /dev/st0 -f
```

note that for the above, you may want look at something like this (note that
larger blocksizes may be optimal in some situations), depends on what your tape
setup supports:

```
sudo mkltfs -d /dev/st0 --volume-name all-start-2024-11-18 --blocksize 1048576 -f
```

the above block size takes into account planned restic pack-size.

create the mount point:

```
sudo mkdir /mnt/tape
```

try mounting

```
sudo ltfs /mnt/ltfs -o devname=/dev/sg4
```

### debugging ltfs

has good manpage.

check the ltfs:

```
sudo ltfsck /dev/st0
```

verify the mount status:

```
mount | grep ltfs
```

check what's on the tape:

```
ls /mnt/ltfs
```

really test it out:

```
echo "test" >> /mnt/ltfs/test.txt
sudo umount /mnt/ltfs
sudo mt -f /dev/st0 eject
ls /mnt/ltfs
```

then try mounting again and checking the file.

## restic

check out these tweaks:

https://restic.readthedocs.io/en/latest/manual_rest.html

from above i was sad to find that `--read-concurrency` is not available in the
stable repo for debian as of right now (current restic verison is 0.14.x or
whatever).

Prepare the restic repository:

```
sudo RESTIC_PASSWORD_FILE=/etc/restic-password /usr/bin/restic init -r /mnt/ltfs
```

To find what you should exclude you should exclude firstly, the things in Linux
that don't make much sense to me to backup (things like `/proc`), but also see
what the largest files are, which may indicate which directories to ignore:

```
sudo find / -type f -exec du -h {} + | sort -rh | head -n 10
```

Here's my command for backups, note the `pack-size` (chosen relative to LTFS
block size), and be sure to `exclude` what you can (lowest hanging fruit, at
least):

```
sudo RESTIC_PASSWORD_FILE=/etc/restic-password /usr/bin/restic backup / \
    --exclude /home/baudrillard/.bitmonero \
    --exclude /root/.bitmonero \
    --exclude /nix \
    --exclude /mnt \
    --exclude /tmp \
    --exclude /run \
    --exclude /var/tmp \
    --exclude /lost+found \
    --exclude /media \
    --exclude /sys \
    --exclude /usr/share/ollama/.ollama/models/blobs \
    --exclude /proc \
    --exclude /dev \
    -r /mnt/ltfs \
    --compression max \
    --pack-size 128
```

Pretty impressive speed for me:

```
processed 414986 files, 64.833 GiB in 11:22
```

Don't forget to keep `/etc/restic-password` backed up, like in a password manager!

verify the backup:

```
sudo RESTIC_PASSWORD_FILE=/etc/restic-password /usr/bin/restic snapshots -r /mnt/ltfs
```

You can do a differential/incremental update by running the same backup
command, but here I try adding over 1TB of data from my `/media` backup
directories by simply running the backup command again, but removing `/media`
from `exclude`.

## Things to think about

handy debug:

```
sudo ltfsck /dev/sg4
```

If you interrupt a backup it'll just totally mess up LTFS, I think.

make sure hardware compression is enabled?

Install this tool and check if compression:

```
sudo umount /mnt/ltfs
sudo apt install sg3-utils
sudo sginfo --all /dev/sg4
```

```
sudo mt -f /dev/st0 status
sudo mt -f /dev/st0 comp on
sudo mt -f /dev/st0 status
```

