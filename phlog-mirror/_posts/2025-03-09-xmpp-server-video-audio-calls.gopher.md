---
date: 2025-03-09
tags:
- sysadmin
- linux
- xmpp
- prosody
title: 'XMPP: audio/video call support (prosody)'

---


Some sloppy notes for you about how to enable audio/video calling for a prosody server in debian.

https://prosody.im/doc/turn
https://gist.github.com/iNPUTmice/a28c438d9bbf3f4a3d4c663ffaa224d9

This is an extension of my other article

reliability of calls bad, basically not work

prosody stun/turn testing: https://prosody.im/doc/turn#testing-your-setup
also prosodyctl check turn


sudo apt-get update
sudo apt-get install coturn

vi /etc/turnserver.conf

i set `realm=xmpp.someodd.zip` uncommented `use-auth-secret` then set `static-auth-secret` to some gibberish set by keepassxc.

then `sudo systemctl restart coturn`

sudo ufw allow 3478,5349/tcp comment 'Allow TURN server TCP ports'
sudo ufw allow 3478,5349/udp comment 'Allow TURN server UDP ports'

dont' forget forward ports router

## prosody

sudo vi /etc/prosody/prosody.cfg.lua

i actually just uncommented `turn_external` in the `modules_enabled` section.

then i had to set `turn_external_host` to `xmpp.someodd.zip` and use the secret i set earlier in `turn_external_secret`.

then i think you can use sudo prosodyctl reload
OR systemctl reload prosody

baudrillard@simulacra ~ % sudo prosodyctl check turn
Identified 1 TURN services.

Testing TURN service xmpp.someodd.zip:3478...
1 warnings:

    STUN returned a private IP! Is the TURN server behind a NAT and misconfigured?

Success!

All checks passed, congratulations!

sudo journalctl -u coturn

sudo prosodyctl check turn -v --ping=stun.conversations.im

baudrillard@simulacra ~ % sudo prosodyctl check turn -v --ping=stun.conversations.im
Identified 1 TURN services.

Testing TURN service xmpp.someodd.zip:3478...
2 warnings:

    STUN returned a private IP! Is the TURN server behind a NAT and misconfigured?
    TURN external IP vs relay address mismatch! Is the TURN server behind a NAT and misconfigured?

External IP: 192.168.1.1
Relayed address 1: 192.168.1.100:50367
TURN external address: 24.130.51.197:50367
Success!

All checks passed, congratulations!

# further debugging

https://webrtc.github.io/samples/src/content/peerconnection/trickle-ice/
you have to edit /etc/turnserver.conf and sudo systemctl restart coturn and 

# Uncomment to use long-term credential mechanism.
# By default no credentials mechanism is used (any user allowed).
# 
#lt-cred-mech
#user=testuser:testpassword

i feel like i'm still having issues between clients. dino works best?

Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/xmpp-server-video-audio-calls.gopher.txt](gopher://gopher.someodd.zip:70/0/phlog/xmpp-server-video-audio-calls.gopher.txt)
