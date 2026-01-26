---
date: 2024-11-18
tags:
- sysadmin
- linux
- lto
- backup_archive
title: Simpler Encrypted LTO Tape Archives

---


Simple setup for encrypted backups using LTO6 on Debian. I have an older, very similar article:

[Archiving with LTO & zpaq](gopher://gopher.someodd.zip:70/1/phlog/archive_lto_zpaq.gopher.html)

I've found tapes are just best to write once and forget about it. Trying to do
updates over time is kind of a pain and I've found it unreliable in some ways.

I have an external LTO6 drive.

## Drive-based key encryption, if you want (I don't suggest)

I actually have found this extremely unreliable and frustrating. I suggest just handling encryption yourself, not through the drive. I believe this is because of [a bug](https://serverfault.com/questions/864580/what-could-cause-a-sense-error-when-setting-lto-encryption) where, basically, you have to avoid `--details` at all costs or it'll put the drive in a weird state. You can do streaming-based encryption with GPG or something.

Install from here: https://github.com/scsitape/stenc (do NOT grab what's available in Debian). Dont' forget to `sudo make install`.

Generate key (max is 256 bits):
```
sudo stenc -g 256 -k /etc/2024-11-lto5.key -kd "November 2024 LTO5 Tape Key"
```

Turn on encryption (you may want to first power cycle [wait for indicators to be stable on lto bay] and then do this BEFORE you put in the cartridge):

```
 % sudo stenc -f /dev/st0 -a 1 -e on -k /etc/tape-stenc-2025-05-11.key
Decrypt mode not specified, using decrypt = on
Changing encryption settings for device /dev/st0...
Success! See system logs for a key change audit log.
```

At this point I noticed the blue encryption indicator lit up on my LTO5 drive.

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

NOte for above: actually to be encrypted may want to do nst0, like this command, which uses pgp to encrypt, instead of relying on firmware encryption of the drive (I like using pgp more [make sure key light is off!]):

```
# first the passphrase creation
sudo sh -c 'umask 077; openssl rand -base64 48 > /etc/backup.passphrase'

# now create the archive
sudo sh -c '
  tar --totals \
      --checkpoint=100 \
      --checkpoint-action=dot \
      --use-compress-program="zstd" \
      -cvf - /media/root/BackupRAID \
  | gpg --symmetric --cipher-algo AES256 \
        --batch --yes \
        --pinentry-mode loopback \
        --passphrase-file /etc/backup.passphrase \
  | dd of=/dev/nst0 bs=1M status=progress
'
```

This is crazy fast. But if blocking factor is large you'll run out of space
quickly. The solution is to perhaps place a single archive onto the tar.

## Test archive, restore

See status:

```
sudo stenc -f /dev/st0
```

Rewind and list contents:

```
sudo mt -f /dev/nst0 rewind
sudo tar -tvf /dev/nst0 --use-compress-program=zstd
```

### if you used pgp (best imo)

Read test successful with:

```
sudo mt -f /dev/nst0 rewind
sudo dd if=/dev/nst0 bs=64k count=1 | file -
# Expect: "GPG symmetrically encrypted data"
```

and...

```
sudo mt -f /dev/nst0 rewind
sudo dd if=/dev/nst0 bs=1M \
| gpg --decrypt --batch --yes \
      --pinentry-mode loopback \
      --passphrase-file /etc/backup.passphrase \
| tar --use-compress-program="zstd" -tvf -
```

you can confirm integrity this way:

```
sudo mt -f /dev/nst0 rewind
sudo dd if=/dev/nst0 bs=1M \
| gpg --decrypt --batch --yes \
      --pinentry-mode loopback \
      --passphrase-file /etc/backup.passphrase \
| tar --use-compress-program="zstd" -tvf - > /dev/null
```

extract...

```
sudo mt -f /dev/nst0 rewind
sudo dd if=/dev/nst0 bs=1M \
| gpg --decrypt --batch --yes \
      --pinentry-mode loopback \
      --passphrase-file /etc/backup.passphrase \
| sudo tar --use-compress-program="zstd" -xvf -
```

## Tips

* Tapes will like just writing one big file--so don't be afraid to just slap a
  highly compressed archive onto there. It might be fun for me to show how to
  zpaq to tape, especially incrementally. Or using restic?
* Bigger block sizes and such for larger data
* If you have tape labels you can use a program on your phone like Orca Scan to keep a tape catalog

Original content in gopherspace: [gopher://gopher.someodd.zip:70/1/phlog/archive-lto.gopher.txt](gopher://gopher.someodd.zip:70/1/phlog/archive-lto.gopher.txt)
