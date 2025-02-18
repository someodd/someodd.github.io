---
date: 2024-12-30
tags:
- gopher
- showcase
title: Now hosting gopherholes!

---
Original content in gopherspace: gopher://gopher.someodd.zip:Just 7071/phlog/


SEEKING PEOPLE INTERESTED IN #gopher 

Want to try drag-and-drop gopherhole hosting (sftp)? It'll get automatically build/fancy-ified by my bore gopherhole builder.

Reply below!


it's just sftp access to `/var/gopher/output/holes/someuser` with custom sftp thingy set. automagically rebuild with bore.

i'll make a service post and also can browse `~`.

my example bore.toml...

my sshd config on server...

## setup basic shit

create the directory structure

```
mkdir -p /var/gopher/output/users/
chown -R bore:bore /var/gopher/output
chmod -R 770 /var/gopher/output
```

make the group:

```
groupadd gopherusers
usermod -aG gopherusers bore
```

adjust perms:

```
chown -R bore:gopherusers /var/gopher/output
chmod -R 775 /var/gopher/output/users
```

## sftp shit setup

for each sftp user:

create account and set their password:

```
useradd -m -s /usr/sbin/nologin -G gopherusers $username
passwd $username
```

ensure they can only access their directory

```
mkdir -p /var/gopher/output/users/$username
chown $username:gopherusers /var/gopher/output/users/$username
chmod 770 /var/gopher/output/users/$username

```

configure sshd for sftp, edit `/etc/ssh/sshd_config`

```
Match Group gopherusers
    ChrootDirectory /var/gopher/output/users/%u
    ForceCommand internal-sftp
    AllowTCPForwarding no
    X11Forwarding no
```

restart that shit with `systemctl restart sshd`

do a quick test with `sftp $username@yourserver` and verify bore is still serving fine

you could even make a script:

```
#!/bin/bash
username=$1
useradd -m -s /usr/sbin/nologin -G gopherusers $username
passwd $username
mkdir -p /var/gopher/output/users/$username
chown $username:gopherusers /var/gopher/output/users/$username
chmod 770 /var/gopher/output/users/$username
```

and add that shitter to `/usr/local/bin/mkboreusr.sh`

what about password

## actually did commands

```
sudo mkdir -p /var/gopher/source/users/
sudo chown -R bore:bore /var/gopher/source
sudo chmod -R 770 /var/gopher/source

sudo groupadd gopherusers
sudo usermod -aG gopherusers bore

sudo chown -R bore:gopherusers /var/gopher/source
sudo chmod -R 775 /var/gopher/source/users
```

create this file and mark as executable `sudo vi /usr/local/bin/boremkusr.sh`:

```
#!/bin/bash
username=$1
useradd -m -s /usr/sbin/nologin -G gopherusers $username
passwd $username
mkdir -p /var/gopher/source/users/$username
chown $username:gopherusers /var/gopher/source/users/$username
chmod 770 /var/gopher/source/users/$username
```

mark as executable:

```
sudo chmod +x /usr/local/bin/boremkusr.sh
```

create first user:

```
sudo /usr/local/bin/boremkusr.sh lingunixtix
```

configure sshd for sftp, edit `/etc/ssh/sshd_config`

```
Port 7777
Match LocalPort 7777 Group *,!gopherusers
    PasswordAuthentication no
    PubkeyAuthentication no
    ForceCommand /bin/false
Match Group gopherusers LocalPort 7777
    ChrootDirectory /var/gopher/source/users/%u
    ForceCommand internal-sftp
    AllowTCPForwarding no
    X11Forwarding no
```

and restart with `sudo service sshd restart`.

now ufw shit:

```
sudo ufw allow 7777/tcp comment "Allow SFTP for gopherusers on port 7777 (TCP)"
```

don't forget to port fortward router.

testing with on client machine:

```
```

## docker setup



