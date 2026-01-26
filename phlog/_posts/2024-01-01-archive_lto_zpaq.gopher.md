---
date: 2024-01-01
tags:
- sysadmin
- linux
- backup_archive
- zpaq
title: Archive using LTO Tape + zpaq

---


# Archive using LTO Tape

I think encryption is enabled by default.

I'm not sure i understand appending to the archive fully like I should... each "append" with `c` makes a new file you have to skip to on tape... need learn efficiently adding?

I think this was started in Jan 2024?

do inside screen `screen -S archival`

## What I did

```bash
sudo apt-get update
sudo apt-get install stunnel
```

Generate key (max is 256 bits):
```
sudo stenc -g 256 -k /etc/2024-01-lto6.key -kd "January 2024 LTO6 Tape Key"
```

Turn on encryption:

```
 % sudo stenc -a 1 -f /dev/st0 -e on -k /etc/2024-01-lto6.key --protect
Provided key length is 256 bits.
Key checksum is 1a1c.
Turning on encryption on device '/dev/nst0'...
Success! See '/var/log/stenc' for a key change audit log.
```

Check status:

```
 % sudo stenc -f /dev/st0 --detail
Status for /dev/nst0
--------------------------------------------------
Device Mfg:              HP      
Product ID:              Ultrium 6-SCSI  
Product Revision:        353D
Drive Encryption:        on
Drive Output:            Decrypting
                         Unencrypted data not outputted
Drive Input:             Encrypting
                         Protecting from raw read
Key Instance Counter:    1
Encryption Algorithm:    1
Drive Key Desc.(uKAD):   January 2024 LTO6 Tape Key
Volume Encryption:       Encrypted, but unable to decrypt due to invalid key. 
Volume Key Desc.(uKAD):  Created 2021-10-18
                         Protected from raw read
Volume Algorithm:        1
```

I wanted to wipe the tape before putting stuff on it (I'm in a `archival` GNU Screen session or whatever):

```
sudo mt -f /dev/nst0 rewind
```

## Example job I did: remove old autobackups but archive first

Start archiving stuff I wanna remove (namely old backups I wanna remove! even adding a README)

```bash
echo "Archival of automatic backup data I want to remove from BackupRAID." > README
sudo mt -f /dev/nst0 rewind
sudo tar -cvf /dev/nst0 README
```

The above made sure the tape was rewound, and we added README. Let's rewind the tape and check the contents, and then after we check the contents move back to end of memory or whatever:

```
sudo mt -f /dev/nst0 rewind
sudo tar tvf /dev/nst0
sudo mt -f /dev/nst0 eom
```

I don't think zpaq can stream so I can't pipe to tar I don't think, so I zpaq the older automatic backup directories i want to remove soon (before putting on tape):

```
mkdir /home/baudrillard/zpaq-to-tape
sudo zpaq a /home/baudrillard/zpaq-to-tape/simulacra-server-restic-backups.zpaq /media/root/BackupRAID/simulacra-server-restic-backups
sudo zpaq a /home/baudrillard/zpaq-to-tape/duplicity-coffeepad-deja-dup.zpaq /media/root/BackupRAID/sftpbackups/backup_coffee/backups-auto/coffeepad-deja-dup
sudo zpaq a /home/baudrillard/zpaq-to-tape/duplicity-fzg2.zpaq /media/root/BackupRAID/sftpbackups/backup_coffee/backups-auto/fzg2
sudo zpaq a /home/baudrillard/zpaq-to-tape/duplicity-simulation.zpaq /media/root/BackupRAID/sftpbackups/backup_coffee/backups-auto/simulation-deja-dup
sudo zpaq a /home/baudrillard/zpaq-to-tape/duplicity-toughbook-begin-2021-03-13.zpaq /media/root/BackupRAID/sftpbackups/backup_coffee/backups-auto/toughbook-begin-2021-03-13
sudo zpaq a /home/baudrillard/zpaq-to-tape/duplicity-x390_waiting_on_fzg2.zpaq /media/root/BackupRAID/sftpbackups/backup_coffee/backups-auto/x390_waiting_on_fzg2
```

Now I can add them all (make sure we're at eom):

```
sudo mt -f /dev/nst0 eom
sudo tar -cvf /dev/nst0 /home/baudrillard/zpaq-to-tape/*.zpaq
```

Finally check the contents again:

```
sudo mt -f /dev/nst0 rewind
sudo tar tvf /dev/nst0
sudo mt -f /dev/nst0 eom
```

Eject and mark as read-only!

Don't be alarmed if it looks like only README is there! You just have to skip through files:

```
sudo mt -f /dev/nst0 rewind
sudo mt -f /dev/nst0 fsf 1
sudo tar tvf /dev/nst0
```

### ChatGPT: It looks like it's not all there!

> The behavior you're experiencing with `tar` and your tape drive (`/dev/nst0`) suggests that the `README` file was the first file written to the tape and the subsequent `*.zpaq` files were appended after it. When you used `tar tvf /dev/nst0` to list the contents of the tape, it showed only the first file (`README`), which is expected behavior due to the sequential nature of tape drives.
>
> ### Understanding Tape Drive Behavior
>
> 1. **Sequential Access**: Tape drives are sequential access devices. When you list the contents of a tape with `tar tvf`, it starts reading from the beginning and stops after the first file or archive it encounters.
>
> 2. **Appending Files**: When you used `tar -cvf /dev/nst0 /home/baudrillard/zpaq-to-tape/*.zpaq`, it appended these files to the tape after the `README` file. However, `tar tvf` will not automatically continue reading through the entire tape to list all files.
>
> ### Accessing Subsequent Files on the Tape
>
> To access and list the subsequent `.zpaq` files, you need to manually navigate through the tape. This can be done using the `mt` command to skip over files until you reach the ones you're interested in. However, this process can be cumbersome and time-consuming, as you need to know exactly how many files or blocks to skip.
>
> ### Example: Skipping Files
>
> If you know how many files are before the one you want to access, you can use `mt fsf` (forward space file) to skip over them. For example, to skip the first file:
>
> ```bash
> sudo mt -f /dev/nst0 fsf 1
> ```
>
> Then, you can use `tar tvf` again to list the next file:
>
> ```bash
> sudo tar tvf /dev/nst0
> ```
>
> Repeat this process to skip through the files on the tape.
>
> ### Important Considerations
>
> - **Tape Management**: Frequent forwards and rewinds can wear out the tape. Plan your backup strategy to minimize unnecessary tape movement.
> - **Indexing Tape Contents**: It's a good practice to maintain an index or log of what is written to the tape and in what order, to facilitate easier access to specific files.
> - **Tape Drive Limitations**: The limitations of tape drives, including their sequential nature, make them less suited for tasks requiring frequent access to individual files. They are typically used for long-term storage where sequential access or full restores are acceptable.
>
> In summary, the behavior you're seeing is consistent with how tape drives operate. To list or access specific files written later on the tape, you'll need to use the `mt` command to navigate the tape sequentially.

More suggestions:

> Maintaining an index or log of the contents of a tape is a crucial practice for efficient data retrieval, especially given the sequential nature of tape storage. Here are some suggestions for indexing tape contents:
>
> ### 1. Create a Digital Index File
>
> Each time you write to the tape, create a digital record of what was written. This index file should include:
>
> - **File Names and Paths**: List all files and directories written to the tape.
> - **Date and Time of Backup**: Record when the backup was made.
> - **Tape Identifier**: If you use multiple tapes, note which tape the backup is on.
> - **Order of Files**: Since tapes are sequential, note the order in which files are written.
> - **Size of Files**: Recording the size can help estimate where on the tape a particular file might be.
>
> After creating this index file, store it in a secure and easily accessible location, separate from the tape.
>
> ### 2. Use a Database or Spreadsheet
>
> For more complex tape libraries, consider using a database or spreadsheet to manage your tape index. This can be as simple as a spreadsheet with columns for each piece of information, or a more sophisticated database for larger archives.
>
> ### 3. Automate the Indexing Process
>
> If possible, automate the creation of the index file. You can write a script that runs after each backup operation, pulling the list of files, their sizes, and other relevant information, and then appending this to your index file or database.
>
> ### 4. Include a Physical Log
>
> For additional redundancy, consider keeping a physical logbook or printed copies of the tape indexes. This can be useful in situations where digital records are not accessible.
>
> ### 5. Store Indexes Offsite
>
> Store backup copies of your indexes in an offsite location to protect against data loss in case of disasters at the primary storage location.
>
> ### 6. Regularly Update and Verify the Index
>
> Regularly update your index with new backups and periodically verify its accuracy against the actual tape contents. This practice helps in maintaining the reliability of the index.
>
> ### 7. Use Tape Management Software
>
> If available, use tape management software that can automatically track and index tape contents. Some enterprise-level backup solutions offer features for tape management and indexing.
>
> ### 8. Barcode Labels
>
> If your tape drive supports it, use barcode labels on each tape and record these in your index. This helps in quickly identifying tapes and correlating them with your digital index.
>
> ### 9. Consider Tape Drive Features
>
> Some modern tape drives and libraries offer features for indexing and managing media. Explore these options as they can significantly simplify tape management.
>
> By maintaining a thorough index, you can significantly reduce the time and effort required to locate and retrieve data from your tapes. This practice is essential for effective tape backup and archival strategies.

### A simpler way (single archive)

Instead of globbing the files, do this:

```
echo "Archival of automatic backup data I want to remove from BackupRAID." > /home/baudrillard/zpaq-to-tape/README
# all the zpaq shit here
sudo mt -f /dev/nst0 rewind
sudo tar -cvf /dev/nst0 /home/baudrillard/zpaq-to-tape/
```

Verify:

```
sudo mt -f /dev/nst0 rewind
sudo tar tvf /dev/nst0
```

can even do this to check if backed up a directory right (if you tar'd using full path):

```
diff <(sudo tar -tf /dev/nst0 | sort) <(sudo find /media/root/BackupRAID/samba/public/ -type f | sort)
```


## KINDA IMPORTANT: ChatGPT nst0 vs. st0

> In the context of tape drives in Linux, `nst0` and `st0` refer to two different device files that represent the same physical tape drive, but they behave differently in terms of how they manage the tape position after operations. Here's the difference:
>
> 1. **`/dev/st0`**: This device file represents the tape drive in "auto-rewind" mode. When you access the tape drive using `/dev/st0`, the tape will automatically rewind to the beginning when the device is closed. This means that if you open the device, perform some operations (like reading or writing), and then close it, the tape will automatically rewind to the start. This behavior can be useful for certain operations but might not be ideal if you want to write multiple files to the tape in sequence without rewinding between writes.
>
> 2. **`/dev/nst0`**: This device file represents the tape drive in "non-rewind" mode. When you use `/dev/nst0`, the tape will not automatically rewind when the device is closed. This allows you to perform multiple operations on the tape (like writing several files in sequence) without the tape rewinding between each operation. It gives you more control over the tape position but requires you to manage the tape position manually, using commands like `mt -f /dev/nst0 rewind` to rewind the tape when needed.
>
> In summary, the choice between `/dev/st0` and `/dev/nst0` depends on how you want to manage the tape's position:
>
> - Use `/dev/st0` if you want the tape to automatically rewind after each set of operations.
> - Use `/dev/nst0` if you want to manually control the tape's position, which is useful for writing multiple files to the tape in sequence or for performing multiple reads/writes at specific positions on the tape.

## Old notes

### Encryption (stenc)

`stenc` is a very good and simple tool with a great manpage. It does one thing
and it does it right. I have had success with this on my LTO5 drive (I should
name the model...).

[stenc on Ubuntu](http://manpages.ubuntu.com/manpages/cosmic/man1/stenc.1.html)

### Check encryption status

Check the encryption status of your drive like this:

```
$ sudo stenc -f /dev/nst0
Status for /dev/nst0
--------------------------------------------------
Drive Encryption:        off
```

... or get more detailed output:

```
$ sudo stenc -f /dev/st0 --detail
Status for /dev/nst0
--------------------------------------------------
Device Mfg:              HP      
Product ID:              Ultrium 5-SCSI  
Product Revision:        Z3ED
Drive Encryption:        off
Drive Output:            Not decrypting
                         Raw encrypted data not outputted
Drive Input:             Not encrypting
Key Instance Counter:    0
Volume Encryption:       Not encrypted
```

**Note:** encryption will be turned off after power cycle. This is a good
feature because otherwise someone could just use your drive to decrypt your
tapes!

### Create a tape encryption key

I recommend giving the encryption key a great description:

-- WAIT WHAT IS THE COMMAND THO

```
generate key (max is 256 bits):
sudo stenc -g 256 -k /etc/september2020tape.key -kd "Semptember 2020 Tape Key"
```

### Turn on encryption

Enable encryption of tapes on the tape drive with this command:

```
$ sudo stenc -a 1 -f /dev/st0 -e on -k september2020tape.key --protect
Provided key length is 256 bits.
Key checksum is 10a0.
Turning on encryption on device '/dev/nst0'...
Success! See '/var/log/stenc' for a key change audit log.
```

Now when you check the status you should see this:

```
$ sudo stenc -f /dev/st0 --detail
Status for /dev/nst0
--------------------------------------------------
Device Mfg:              HP      
Product ID:              Ultrium 5-SCSI  
Product Revision:        Z3ED
Drive Encryption:        on
Drive Output:            Decrypting
                         Unencrypted data not outputted
Drive Input:             Encrypting
                         Protecting from raw read
Key Instance Counter:    2
Encryption Algorithm:    1
Drive Key Desc.(uKAD):   Semptember 2020 Tape Key
Volume Encryption:       Not encrypted
```

On my LTO5 drive a blue light indicating encryption turned on after this
command.

About "volume encryption: not encrypted," don't worry, once you write using
`tar` you'll see this:

```
$ sudo stenc -f /dev/st0 --detail                                         [19:36:47]
Status for /dev/nst0
--------------------------------------------------
Device Mfg:              HP      
Product ID:              Ultrium 5-SCSI  
Product Revision:        Z3ED
Drive Encryption:        on
Drive Output:            Decrypting
                         Unencrypted data not outputted
Drive Input:             Encrypting
                         Protecting from raw read
Key Instance Counter:    1
Encryption Algorithm:    1
Drive Key Desc.(uKAD):   Semptember 2020 Tape Key
Volume Encryption:       Encrypted and able to decrypt
                         Protected from raw read
Volume Algorithm:        1
```

#### A bug I encountered while trying to enable encryption

If you get an error like this when trying to enable encryption:

```
$ sudo stenc -a 1 -f /dev/st0 -e on -k september2020tape.key
Provided key length is 256 bits.
Key checksum is 10a0.
Turning on encryption on device '/dev/nst0'...
Sense Code:              Illegal Request (0x05)
 ASC:                    0x26
 ASCQ:                   0x00
 Additional data:        0x00000000000000000000000000000000
Error: Turning encryption on for '/dev/nst0' failed!
Usage: stenc --version | -g <length> -k <file> [-kd <description>] | -f <device> [--detail] [-e <on/mixed/rawread/off> [-k <file>] [-kd <description>] [-a <index>] [--protect | --unprotect] [--ckod] ]
Type 'man stenc' for more information.
```

... try just running these commands:

```
sudo mt -f /dev/nst0 status
sudo stenc --detail -f /dev/st0
```

... and trying again! If you're still having trouble keep reading on in this
section ("in-depth").

##### In-depth

I covered this in the section about enabling encryption a bit, but if you run into a bug like this:

```
$ sudo stenc -a 1 -f /dev/st0 -e on -k september2020tape.key
Provided key length is 256 bits.
Key checksum is 10a0.
Turning on encryption on device '/dev/nst0'...
Sense Code:              Illegal Request (0x05)
 ASC:                    0x26
 ASCQ:                   0x00
 Additional data:        0x00000000000000000000000000000000
Error: Turning encryption on for '/dev/nst0' failed!
Usage: stenc --version | -g <length> -k <file> [-kd <description>] | -f <device> [--detail] [-e <on/mixed/rawread/off> [-k <file>] [-kd <description>] [-a <index>] [--protect | --unprotect] [--ckod] ]
Type 'man stenc' for more information.
```

... then this section might be for you!

##### Bug description

There's a [documented bug in stenc](https://github.com/scsitape/stenc/issues/11)
that took some work to patch and workaround. Basically, using `--detail` (or
some related command?) will result in putting the drive into an "inconsistent
state" which will make it reject all new keys.

I tried both stenc 1.0.7 and 1.0.8 to no avail. I kept getting this error:

```
$ stenc -f /dev/nst0 -a 1 -e on -k test.key
Provided key length is 256 bits.
Key checksum is 7a43.
Turning on encryption on device '/dev/nst0'...
Sense Code:              Illegal Request (0x05)
 ASC:                    0x26
 ASCQ:                   0x00
 Additional data:        0x00000000000000000000000000000000
Error: Turning encryption on for '/dev/nst0' failed!
Usage: stenc --version | -g <length> -k <file> [-kd <description>] | -f <device> [--detail] [-e <on/mixed/rawread/off> [-k <file>] [-kd <description>] [-a <index>] [--protect | --unprotect] [--ckod] ]
Type 'man stenc' for more information
```

This bug in `stenc` is discussed in [a ServerFault post about the above error](https://serverfault.com/questions/864580/what-could-cause-a-sense-error-when-setting-lto-encryption):

> There's a bug in stenc 1.0.7 which causes a crash if you use --detail on a blank tape. I have tried to contact the author with a fix but can't get hold of him.
>
> It seems that this crash leaves the drive in an inconsistent state, where it refuses to accept further keys. Fixing the bug and then running stenc --detail with no crash seems to have fixed the problem. I can now set any keys any number of times and there have been no further issues.

The solution mentioned, that worked for me, is:

> If anyone else is having the same problem, in stenc-1.0.7/sec/scsiencrypt.cpp at line 176 it says delete status;. You need to add a new line directly below this that reads status=NULL;. This fixes a double-free error causing the crash.

They give this `diff` as an example:

```diff
--- a/src/scsiencrypt.cpp
+++ b/src/scsiencrypt.cpp
@@ -174,6 +174,7 @@ SSP_NBES* SSPGetNBES(string tapeDevice,bool retry){
            if(status->nbes.encryptionStatus!=0x01)break;
            if(moves>=MAX_TAPE_READ_BLOCKS)break;
            delete status;
+           status=NULL; //double free bug fix
            if(!moveTape(tapeDevice,1,true))break;
            moves++;
            status=SSPGetNBES(tapeDevice,false);
```

##### Fixing the bug: making the patch

So what I did was add `status=NULL;` after `delete status;` (on line 176) as
seen above. However, I used this copy of `stenc` to work off of: [stenc
v1.0.7](https://github.com/scsitape/stenc/archive/1.0.7.tar.gz) and then I
edited `src/scsiencrypt.ccp` as noted above, and then I ran these commands
after extracting and `cd`'ing into the `.tar.gz`:

```bash
sudo autoreconf --install
sudo ./configure
sudo make
sudo make install
```

I wasn't sure if i just needed to run both `sudo make` and `sudo make install`,
but I get the feeling only `sudo make install` is required.

The weird thing is this didn't work immediately. I got the same error at first,
but I had to do some of these commands and fiddle around with it until it
magically worked, which kind of makes sense since the bug is about `--detail`
on a blank tape leaving it in an "inconsistent state:"

```
sudo mt -f /dev/nst0 status
sudo stenc --detail -f /dev/st0
```

... I also power cycled the device a couple times.

Eventually this command worked and I saw my blue encryption light turn on my external LTO5 drive:

```
sudo stenc -a 1 -f /dev/st0 -e on -k september2020tape.key --protect
```

i entered this over and over and eventually it worked after turning it off!

```
sudo stenc -a 1 -f /dev/st0 -e on -k september2020tape.key --protect
```

### Hardware note

I have an HP drive so I use an algo index of 1 (the `-a 1` part of the command).

### Security

Read the key change auditing section... look at `/var/log/stenc`

don't forget to look at the key checksum (looks like `Key checksum is 10a0.`) when enabling encryption

### Try disabling encryption

For education's sake and verification, try turning off encryption:

```
$ sudo stenc -a 1 -f /dev/st0 -e off
Turning off encryption on device '/dev/nst0'...
Success! See '/var/log/stenc' for a key change audit log.
```

Now try listing the contents of the tape archive after encryption has been turned off:

```
$ sudo tar tvf /dev/st0
tar: /dev/st0: Cannot read: Input/output error
tar: At beginning of tape, quitting now
tar: Error is not recoverable: exiting now
```

Double-check the status:

```
$ sudo stenc --detail -f /dev/st0
Status for /dev/nst0
--------------------------------------------------
Device Mfg:              HP      
Product ID:              Ultrium 5-SCSI  
Product Revision:        Z3ED
Drive Encryption:        off
Drive Output:            Not decrypting
                         Raw encrypted data not outputted
Drive Input:             Not encrypting
Key Instance Counter:    2
Volume Encryption:       Encrypted, but unable to decrypt due to invalid key. 
Volume Key Desc.(uKAD):  Semptember 2020 Tape Key
                         Protected from raw read
Volume Algorithm:        1
```

Now turn it back on, using the key file that was used to write the tar to begin
with:

```
$ sudo stenc -a 1 -f /dev/st0 -e on -k september2020tape.key --protect
Provided key length is 256 bits.
Key checksum is 10a0.
Turning on encryption on device '/dev/nst0'...
Sense Code:              Illegal Request (0x05)
 ASC:                    0x26
 ASCQ:                   0x00
 Additional data:        0x00000000000000000000000000000000
Error: Turning encryption on for '/dev/nst0' failed!
Usage: stenc --version | -g <length> -k <file> [-kd <description>] | -f <device> [--detail] [-e <on/mixed/rawread/off> [-k <file>] [-kd <description>] [-a <index>] [--protect | --unprotect] [--ckod] ]
Type 'man stenc' for more information.
```

This error is expected! Don't worry! Just power cycle and then use these commands:

Now you can read just like before:

```
$ sudo mt -f /dev/nst0 status
$ sudo stenc --detail -f /dev/st0
$ sudo stenc -a 1 -f /dev/st0 -e on -k september2020tape.key --protect
$ sudo tar tvf /dev/st0
```

Sometimes you can just do the above series of commands twice and it'll work
(instead of power cycling).

### Don't put everything in big compressed archives

It increases the chance that your entire backup is fucked.

Just avoid compression and use multiple copies.

http://ask-leo.com/does_archiving_compressed_files_increase_the_chance_of_corruption.html

### Manging the cartridge with tar

The joys of `tar`!

Use `tar` for it's original purpose: tape archives (that's where the name comes
from!). I like to think of a tape cartrdige as a physical `tar` (that's pretty
much what it is, but you'll see what I mean).

Get good with `tar`. Learn `tar`. There are some `tar` caveats when working
with LTO, as well as some general tips:

  * Something about incremental and differential...

#### Writing to the tape archive

Backup to the device/add to the archive:

```
$ sudo tar -cvf /dev/st0 Docker
```

#### Check tape archive contents

You can check what you've written to the `/dev/st0` tape archive like this:

```
$ sudo tar tvf /dev/st0
```

Note: `t` is to list (you can also use `--list`), `v` is verbose and `f` is to
name a file instead of using `stdin`. Although listing `/dev/st0` instead of
`nst0` might've been a mistake because this command took *forever* to complete.


### Storing the tapes for longevity

https://www.google.com/search?client=firefox-b-1-e&q=how+to+store+lto+cartridges

**bonus:** since tapes have a barcode you can even use a library sofware and
barcode scanner to scan tapes...

#### Keeping a tape catalog

#### Verifying tape archives

## 

## ltofs

https://mirrors.ustc.edu.cn/bjlx/pool/main/l/ltfs/

wget https://mirrors.ustc.edu.cn/bjlx/pool/main/l/ltfs/ltfs_2.4.1-1_amd64.deb
sudo dpkg -i ltfs_2.4.1-1_amd64.deb

```
sudo wget https://mirrors.ustc.edu.cn/bjlx/pool/main/l/ltfs/ltfs_2.5.0.0~20240731-2_amd64.deb
sudo apt-get install ./ltfs_2.5.0.0\~20240731-2_amd64.deb
```

## Tips: using streaming compression

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

Note for above: actually to be encrypted may want to do nst0?

This is crazy fast. But if blocking factor is large you'll run out of space
quickly. The solution is to perhaps place a single archive onto the tar.

Rewind and list contents:

```
sudo mt -f /dev/nst0 rewind
sudo tar -tvf /dev/nst0 --use-compress-program=zstd
```



Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/archive_lto_zpaq.gopher.txt](gopher://gopher.someodd.zip:70/0/phlog/archive_lto_zpaq.gopher.txt)
