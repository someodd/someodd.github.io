---
date: 2024-11-26
tags:
- sysadmin
- linux
- crypto
title: Running Bitcoin on my server

---
Original content in gopherspace: gopher://gopher.someodd.zip:Just 7071/phlog/


Actually own  your BTC by running `bitcoind` on your (Debian, Linux) server.

This is in preperation for another project I'm working on, too.

I like to use the `prune` 

## Installing Bitcoin Core on Debian

Install the depends:

```
sudo apt update
sudo apt install software-properties-common
sudo apt install wget gpg
```

Download from the official Bitcoin Core website: https://bitcoincore.org/en/download/

Verify the download using GPG to ensure it wasn't tampered with.

```
export VERSION="28.0"
wget "https://bitcoincore.org/bin/bitcoin-core-${VERSION}/SHA256SUMS"
wget "https://bitcoincore.org/bin/bitcoin-core-${VERSION}/SHA256SUMS.asc"
wget "https://bitcoincore.org/bin/bitcoin-core-${VERSION}/bitcoin-${VERSION}-x86_64-linux-gnu.tar.gz"
sha256sum --ignore-missing --check SHA256SUMS
```

To go further with verification, namely to find an author to trust and verify
the GPG signature, read the instructions on the Bitcoin Core website.

https://bitcoincore.org/en/download/

```
tar -xvf "bitcoin-${VERSION}-x86_64-linux-gnu.tar.gz"
cd "bitcoin-${VERSION}"
```

Copy binaries:

```
sudo install -m 0755 -o root -g root -t /usr/local/bin bin/*
```

Create the data dir:

```
mkdir -p ~/.bitcoin
```

## Configuring

Edit `~/.bitcoin/bitcoin.conf`:

```
# Run as a server
server=1

# Enable pruning (size in MB)
prune=550

# RPC username and password (set these to something secure)
rpcuser=yourusername
rpcpassword=yoursecurepassword

# Reduce disk space and bandwidth usage
maxconnections=20

# Optionally, run only on Tor for enhanced privacy
onlynet=onion
proxy=127.0.0.1:9050

# Enable logging (optional)
debug=1
```

## First run

Run as background daemon:

```
bitcoind -daemon
```

Monitor sync process:

```
bitcoin-cli getblockchaininfo
```

## Autostart with systemd

Create systemd service file:

```
sudo vi /etc/systemd/system/bitcoind.service
```

Service file contents (use YOUR username):

```
# Install this in /etc/systemd/system/
# See below for more details and options
# https://raw.githubusercontent.com-/bitcoin/bitcoin/76deb30550b2492f9c8d9f0302da32025166e0c5/contrib/init/bitcoind.service
# Then run following to always start:
# systemctl enable bitcoind
#
# and the following to start immediately:
# systemctl start bitcoind

[Unit]
Description=Bitcoin daemon
After=network.target

[Service]
ExecStart=/usr/local/bin/bitcoind-start.sh
TimeoutStartSec=600

# Process management
####################

Type=forking
PIDFile=/home/baudrillard/.bitcoin/bitcoind.pid
Restart=on-failure

# Directory creation and permissions
####################################

# Run as bitcoin:bitcoin or <youruser>
User=youruser
Group=youruser

# Hardening measures
####################

# Provide a private /tmp and /var/tmp.
PrivateTmp=true

# Use a new /dev namespace only populated with API pseudo devices
# such as /dev/null, /dev/zero and /dev/random.
PrivateDevices=true

# Deny the creation of writable and executable memory mappings.
MemoryDenyWriteExecute=true

[Install]
WantedBy=multi-user.target
```

Create script and set the permissions:

```
sudo vi /usr/local/bin/bitcoind-start.sh
```

The file:

```
#!/bin/bash

# Just a simple wrapper to start bitcoind.
#
# If using systemd, simply create a file (e.g. /etc/systemd/system/bitcoind.service)
# from example file below and add this script in ExecStart.
# https://raw.githubusercontent.com-/bitcoin/bitcoin/76deb30550b2492f9c8d9f0302da32025166e0c5/contrib/init/bitcoind.service
#
# Then run following to always start:
# systemctl enable bitcoind
#
# and the following to start immediately:
# systemctl start bitcoind

# If you are mounting a secondary disk, find the UUID of your
# disk and a line entry in /etc/fstab e.g.
#
# UUID=foo-bar-1234 /path-to-dir/.bitcoin ext4 defaults 0 0

set -e

# Let's wait for 30 seconds in case other processes need to come up first.
sleep 30

echo "Starting bitcoind..."

bitcoind --daemon --server -pid=/home/baudrillard/.bitcoin/bitcoind.pid -conf=/home/baudrillard/.bitcoin/bitcoin.conf

echo "Done!"
```

```
sudo chmod +x /usr/local/bin/bitcoind-start.sh
```

Enable:

```
sudo systemctl enable bitcoind
sudo systemctl start bitcoind
sudo systemctl status bitcoind
```

Check logs too:

```
sudo journalctl -u bitcoind.service 
```

## Wallet setup

See which wallets are available:

```
% bitcoin-cli listwallets      
[
]
```

I have none, so I'll create one and encrypt it:

```
bitcoin-cli createwallet "main_2024-11-23" false false "your-strong-passphrase" false true true false
```

Check new wallet status:

```
bitcoin-cli -rpcwallet="main_2024-11-23" getwalletinfo
```

Ensure it's added to startup in `~/.bitcoin/bitcoin.conf`:

```
wallet=main_2024-11-23
```

To ensure the wallet starts on startup:

```
sudo systemctl restart bitcoind
bitcoin-cli listwallets
```

Also check to make sure the wallet is actually encrypted, with a bogus password:

```
bitcoin-cli -rpcwallet="main_2024-11-23" walletpassphrase "asdf" 10
```

## Backup your wallet

Create `/home/baudrillard/.bitcoin/backups`.

Let's create this backup script below `~/.bitcoin/backup_script.sh`:

```
#!/bin/bash
# Backup bitcoin 
timestamp=$(date +"%Y-%m-%d_%H-%M-%S")
backup_dir="/home/baudrillard/.bitcoin/backups"
wallet_name="main_2024-11-23"
backup_file="${backup_dir}/${wallet_name}_backup_${timestamp}.dat"
mkdir -p "${backup_dir}" && bitcoin-cli -rpcwallet="${wallet_name}" backupwallet "${backup_file}"
find "${backup_dir}" -name "${wallet_name}_backup_*.dat" -type f | sort | head -n -5 | xargs -r rm -f
```

Mark as executable `chmod +x ~/.bitcoin/backup_script.sh` and also add to user cron weekly backup (`crontab -e`):

```
0 2 * * 0 /home/baudrillard/.bitcoin/backup_script.sh
```

Be sure to actually try running this backup script and test if the backup is valid:

```
bitcoind -datadir=/tmp/bitcoin-test -daemon && sleep 5 && bitcoin-cli -datadir=/tmp/bitcoin-test loadwallet "/path/to/backup/wallet.dat" && bitcoin-cli -datadir=/tmp/bitcoin-test getwalletinfo && bitcoin-cli -datadir=/tmp/bitcoin-test stop
```

## Basic usage

## Mining

