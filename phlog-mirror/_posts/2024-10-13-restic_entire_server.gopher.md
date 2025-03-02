---
date: 2024-10-13
tags:
- linux
- sysadmin
title: Restic Backup Entire Server

---


Let's make a backup of the entire server.

I like `restic`.

I think it's used by the LHC or something to store/move like... an exabyte of data?

I found out about it because of DeAndre Queary's guides on YouTube:

https://www.youtube.com/@DreQueary

Install:

```
sudo apt-get install restic
```

## Initialize the repo

Restic backups are referred to as "repositories." These repos will contain versions of files.

Initialize the repository:

```
restic init -r /media/root/BackupRAID/server-backups/restic-simulacra-entire-server-start-2024-10-13
```

It will ask you for a password for the new repository, in order to encrypt the contents. This is not required.

## Create the password file

Time to store a password in plaintext (yikes!), just put your password into `/etc/restic-password` (make sure no extra spaces or newlines, etc., or you'll get wrong password/no key found errors). Then set the permissions so only root has access:

```
sudo chown root:root /etc/restic-password
sudo chmod 600 /etc/restic-password
```

You will then reference this file with the environmental variable `RESTIC_PASSWORD_FILE`.

The reason I'm not too worried about this kind of set up is someone with the permission to access the file, will have access to all the things we are backing up, anyway, but there may be a better way, anyway. This is also a security feature for if I move the backup to a tape or RDX.

## Run the backup

Here we will create the first backup manually, before adding it to cron. Note all the directories I am excluding. I think having a blacklist for backups, rather than a whitelist, is a better idea.

```
sudo RESTIC_PASSWORD_FILE=/etc/restic-password /usr/bin/restic backup / --exclude /home/baudrillard/.bitmonero --exclude /root/.bitmonero --exclude /nix --exclude /mnt --exclude /tmp --exclude /run --exclude /var/tmp --exclude /lost+found --exclude /media --exclude /sys --exclude /proc --exclude /dev -r /media/root/BackupRAID/server-backups/restic-simulacra-entire-server-start-2024-10-13
```

## Test the backup

```
sudo RESTIC_PASSWORD_FILE=/etc/restic-password restic check -r /media/root/BackupRAID/server-backups/restic-simulacra-entire-server-start-2024-10-13
```

You can also get stats on the repo:

```
sudo RESTIC_PASSWORD_FILE=/etc/restic-password restic stats -r /media/root/BackupRAID/server-backups/restic-simulacra-entire-server-start-2024-10-13
```

## Automate the backup (cron)

I made a script for backups (`/usr/local/bin/restic-backup.sh` [make sure to mark it as executable]):

```
#!/bin/bash

LOG_FILE="/var/log/restic-backup.log"
export RESTIC_PASSWORD_FILE=/etc/restic-password

/usr/bin/restic backup / --exclude /home/baudrillard/.bitmonero --exclude /root/.bitmonero --exclude /nix --exclude /mnt --exclude /tmp --exclude /run --exclude /var/tmp --exclude /lost+found --exclude /media --exclude /sys --exclude /proc --exclude /dev -r /media/root/BackupRAID/server-backups/restic-simulacra-entire-server-start-2024-10-13 > "$LOG_FILE" 2>&1

if [ $? -eq 0 ]; then
    DATE=$(date "+%Y-%m-%d %H:%M:%S")
    echo "Backup completed successfully at $DATE" >> "$LOG_FILE"
else
    DATE=$(date "+%Y-%m-%d %H:%M:%S")
    echo "Backup failed at $DATE" >> "$LOG_FILE"
fi
```

I also created a script to check the integrity and prune restic (`/usr/local/bin/restic-maintain.sh`):

```
#!/bin/bash

LOG_FILE="/var/log/restic-maintain.log"
export RESTIC_PASSWORD_FILE=/etc/restic-password
export RESTIC_REPOSITORY=/media/root/BackupRAID/server-backups/restic-simulacra-entire-server-start-2024-10-13

# Check integrity and prune
restic check > $LOG_FILE
restic forget --prune --keep-daily=7 --keep-weekly=4 --keep-monthly=12 >> $LOG_FILE

# Appends a timestamp to the log
date "+%Y-%m-%d %H:%M:%S" >> $LOG_FILE
```

Let's simply add the script to root's crontab:

```
sudo crontab -e
```

Add these entries (backup daily at 2am, maintain weekly Saturday at 6am):

```
0 2 * * * /usr/local/bin/restic-backup.sh
0 6 * * 6 /usr/local/bin/restic-maintain.sh
```

## Restore from backup (when needed)

### List snapshots

```
restic snapshots -r /media/root/BackupRAID/server-backups/restic-simulacra-entire-server-start-2024-10-13
```

View files in a specific snapshot:

```
restic ls [snapshot-id] -r /path/to/repo
```

Example:

```
restic ls latest -r /media/root/BackupRAID/server-backups/restic-simulacra-entire-server-start-2024-10-13
```

### Restoring

Restore a specific file or directory:

```
restic restore latest --target /restore/path --include /home/user/docs -r /path/to/repo
```

Restore and overwrite:

```
restic restore latest --target / --include /etc/hosts --overwrite -r /path/to/repo
```

Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/restic_entire_server.gopher.txt](gopher://gopher.someodd.zip:70/0/phlog/restic_entire_server.gopher.txt)
