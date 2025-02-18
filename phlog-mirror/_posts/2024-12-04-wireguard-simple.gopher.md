---
date: 2024-12-04
tags:
- sysadmin
- linux
- networking
- debian
title: Simple, quick VPN server using WireGuard

---
Original content in gopherspace: gopher://gopher.someodd.zip:Just 7071/phlog/


Why do I use a VPN? I personally use it because I think exposing a VPN service with
which to connect to all my services like SSH is a lot safer than just exposing
SSH to the wild, wild Internet.

## High level

You will need a domain that points to your VPN server. If you don't have a
static IP I recommend Afraid.org's Dynamic DNS service. This will be used on
VPN clients to specify an `Endpoint` which point to the server.

You'll have to concern yourself with private/public keys in WireGuard so make
sure to keep the keys very secure! You will generate a pub/priv key on each
machine (server and clients)! The public keys are a way of
identifying/whitelisting connections to other computers in wireguard, and the
private key will be used to prove we are who we say we are (corresponding to the
public key the other computer has for the connection).

**Please note this important caveat:** you need to be aware of `SaveConfig`
which basically means if WG0 is running and you make any changes, they may not
be permanent!

## Possible preliminary tweaks

I have some notes on how you may need to enable IP forwarding, by editing `/etc/sysctl.conf`:

```
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```

... then apply the changes:

```
sudo sysctl -p
```

## Server configuration

Generate the public/private keys using WireGuard.

`etc/wireguard/wg0.conf`:

```
[Interface]
Address = 10.1.0.1/24
SaveConfig = true
ListenPort = 51820
PrivateKey = THISSERVERPRIVATEKEY

[Peer]
PublicKey = mylaptopspubkey
AllowedIPs = 10.1.0.2/32
Endpoint = 192.168.1.1:40047

[Peer]
PublicKey = idkwhatthisis
AllowedIPs = 10.1.0.3/32
Endpoint = 192.168.1.1:43879

[Peer]
PublicKey = nordoiknowthispeeranymore
AllowedIPs = 10.1.0.4/32
Endpoint = 192.168.1.1:51224
```

As you can see there's pretty much a whitelist of peers identified by their
public key, assigned a static IP. I'm actually not sure why the endpoint is
just my router and on different ports. You'll want to make sure you're
generating the private/public keys on each client and then using their public
key for their corresponding `[Peer]` entry here.

Once you're sure everything is set up as expected then:

```
sudo systemctl start wg-quick@wg0
sudo systemctl enable wg-quick@wg0
```

I think one of these peers is my phone.

## Laptop client configuration

Don't forget to generate the public/private keys using WireGuard.

Here's the `etc/wireguard/wg0.conf` config for my laptop client (Debian), with comments:

```
# This laptop (client)
[Interface]
Address = 10.1.0.2/24
PrivateKey = THISLAPTOPPRIVATEKEY
#DNS = 1.1.1.1

# Configuring connection to my VPN server (host)
[Peer]
PublicKey = MYSERVERPUBLICKEYORISITMYLAPTOPUBLICKEY?
# Note the endpoint below, this needs to point to your VPN server!
Endpoint = someodd.zip:51820
# The AllowedIPs directive is important because it helps us whitelist
# which (IP address) destinations we actually want to use this connection for.
AllowedIPs = 10.1.0.0/24
PersistentKeepalive = 25
```

Restart the interface and test it with:

```
ping 10.1.0.1
```

## Phone client configuration (Android)

You can download "WG Tunnel" off F-Droid. I don't think the interface is great,
but once you wrestle with it, it's ok.

For the interface (basically defining how to identify TO the server, here) we
can give a name, I think it will generate a private and public key for us, and
we define our IP address. I kept everything else blank.

For the peer, which is the server, we have to use the server's public key, also
for the endpoint use that domain that points to your server. Don't forget
Allowed IPs should be `10.1.0.0/24` so you don't route everything.

You should now be able to use Termux to SSH in to `user@10.1.0.1` or whatever.

## General usage

Restart `wg0` (the Wireguard interface):

```
sudo systemctl restart wg-quick@wg0
```

Generate keys:

```
umask 077
wg genkey | tee /etc/wireguard/privatekey | wg pubkey > /etc/wireguard/publickey
```

