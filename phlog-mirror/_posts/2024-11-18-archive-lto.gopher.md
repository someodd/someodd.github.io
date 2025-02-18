---
date: 2024-11-18
tags:
- sysadmin
- linux
- archival
- lto
title: Archive using LTO Tape

---
Original content in gopherspace: gopher://gopher.someodd.zip:Just 7071/phlog/


A new guide to archving with LTO5 and Debian.

I've found tapes are just best to write once and forget about it. Trying to do
updates over time is kind of a pain and I've found it unreliable in some ways.

I have an exteranl LTO5 drive: HPE StorageWorks LTO-5 Ultrium 3000 SAS.

Install `stunnel` and `stenc`:

```
sudo apt-get update
sudo apt-get install stunnel stenc
```

Generate key (max is 256 bits):
```
sudo stenc -g 256 -k /etc/2024-11-lto5.key -kd "November 2024 LTO5 Tape Key"
```

Turn on encryption:

```
 % sudo stenc -a 1 -f /dev/st0 -e on -k /etc/2024-11-lto5.key --protect
Provided key length is 256 bits.
Key checksum is 1a1c.
Turning on encryption on device '/dev/nst0'...
Success! See '/var/log/stenc' for a key change audit log.
```

At this point I noticed the blue encryption indicator lit up on my LTO5 drive.

## Compression

I think compression is enabled by default.

Look up the compression page:

```
baudrillard@simulacra ~ % sudo sg_logs -p 0x32 /dev/sg4
    HP        Ultrium 5-SCSI    Z3ED
Data compression page  (LTO-5 specific) [0x32]
  Read compression ratio x100: 114
  Write compression ratio x100: 0
  Megabytes transferred to server: 4
  Bytes transferred to server: 129839
  Megabytes read from tape: 3
  Bytes read from tape: 633400
  Megabytes transferred from server: 0
  Bytes transferred from server: 0
  Megabytes written to tape: 0
  Bytes written to tape: 0
```

I'm going to use software compression, though.

## Making the archive

Choose between `zstd` (faster) and `xz` (better compression ratio), but both
are built for streams, I think.

```
sudo tar \
    --exclude=/home/baudrillard/.bitmonero \
    --exclude=/root/.bitmonero \
    --exclude=/nix \
    --exclude=/snap \
    --exclude=/var/cache \
    --exclude=/mnt \
    --exclude=/tmp \
    --exclude=/media \
    --exclude=/run \
    --exclude=/var/tmp \
    --exclude=/lost+found \
    --exclude=/sys \
    --exclude=/usr/share/ollama/.ollama/models/blobs \
    --exclude=/proc \
    --exclude=/dev \
    --totals --checkpoint=100 --checkpoint-action=dot \
    --use-compress-program="zstd" -cvf /dev/st0 /
```

This is crazy fast. But if blocking factor is large you'll run out of space
quickly. The solution is to perhaps place a single archive onto the tar.

## Tips

* Tapes will like just writing one big file--so don't be afraid to just slap a
  highly compressed archive onto there. It might be fun for me to show how to
  zpaq to tape, especially incrementally. Or using restic?
* Bigger block sizes and such for larger data

