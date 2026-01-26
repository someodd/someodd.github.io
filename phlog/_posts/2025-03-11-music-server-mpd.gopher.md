---
date: 2025-03-11
tags:
- sysadmin
- linux
- audio
title: Music server with Music Player Daemon

---


I had some requirements:

* Works on my xbox
* I can jam while working out at gym (android phone)
* I don't want anything complicated/heavy
* I just want to stream via wireshark vpn connection

## Install

sudo apt-get update
sudo apt-get install mpd mpc

## Configure

edit `/etc/mpd.conf`:

```
music_directory         "/media/root/BackupRAID/mpd/music"
playlist_directory              "/media/root/BackupRAID/mpd/playlists"
user                            "mpd"
group                           "mpd"
bind_to_address "0.0.0.0"
audio_output {
        type            "httpd"
        name            "My HTTP Stream"
        encoder         "vorbis"                # optional, vorbis or lame
        port            "9000"
        bind_to_address "0.0.0.0"               # optional, IPv4 or IPv6
        quality         "5.0"                   # do not define if bitrate is defined
#       bitrate         "128"                   # do not define if quality is defined
        format          "44100:16:1"
        max_clients     "0"                     # optional 0=no limit
	always_on	"yes"
}
```

binding to that address instead of just localhost so i can connect via wireshark (vpn)

should I use always_on so doesn't mess up when skipping songs or whatever sometimes? i notice this is an issue where the stream just stops. very annoying.

Let's create a `mpd` group we can add users to and then add our user to it (to be able to copy over files):

```
sudo groupadd mpd
```

also create the dirs:

```
sudo mkdir -p /media/root/BackupRAID/mpd/music
sudo mkdir /media/root/BackupRAID/mpd/playlists
sudo chown -R mpd:mpd /media/root/BackupRAID/mpd
sudo chmod -R 775 /media/root/BackupRAID/mpd/
```

I added a user to the mpd group:

```
sudo usermod -a -G mpd baudrillard
```

Now on client machine I could do this:

```
rsync -av --chown=:mpd --delete /home/tilde/Music/ baudrillard@10.1.0.1:/media/root/BackupRAID/mpd/music/
```

Once it finishes copying:

sudo systemctl restart mpd

if you need to you can use mpc update or sudo systemctl status mpd

firewalling:

sudo ufw allow 6600/tcp comment 'MPD streaming access'
sudo ufw allow 9000/tcp comment "MPD HTTP streaming"

Be careful, because if you allow people to listen to your stream/your stream is exposed to internet, that will let anyone listen to your internet radio station, which is generally illegal depending on what you're streaming.

### Weirdly complicated side quest: hdmi output from server

**These are just notes. I haven't actually figured this out yet.**

if you actually want to add an output for speakers connected to the server you can add another block. For me I wanted it to output via HDMI. I ran this command:

```
baudrillard@simulacra ~ % aplay -l

**** List of PLAYBACK Hardware Devices ****
card 0: PCH [HDA Intel PCH], device 0: ALC233 Analog [ALC233 Analog]
  Subdevices: 0/1
  Subdevice #0: subdevice #0
card 0: PCH [HDA Intel PCH], device 3: HDMI 0 [HDMI 0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 0: PCH [HDA Intel PCH], device 7: HDMI 1 [HDMI 1]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 0: PCH [HDA Intel PCH], device 8: HDMI 2 [HDMI 2]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: NVidia [HDA NVidia], device 3: HDMI 0 [RX-A1080]
  Subdevices: 0/1
  Subdevice #0: subdevice #0
card 1: NVidia [HDA NVidia], device 7: HDMI 1 [HDMI 1]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: NVidia [HDA NVidia], device 8: HDMI 2 [HDMI 2]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: NVidia [HDA NVidia], device 9: HDMI 3 [HDMI 3]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

then edit my config/add this block:

```
audio_output {
    type        "alsa"
    name        "RX-A1080 HDMI Output"
    device      "hw:1,3"       # or "plughw:1,3" if you need automatic format conversion
    mixer_type  "software"     # software volume control
}
```

I will also have to disable pipewire otherwise pipewire will take control of the device. you probably don't want ot do this if you're not on a server set up/are using a desktop or something where you like to produce sound. an alternative setup would be to use mpd in user mode.

```
systemctl --user stop pipewire.service pipewire.socket
systemctl --user stop wireplumber.service
systemctl --user stop pulseaudio.service pulseaudio.socket
killall pipewire
killall wireplumber
killall pulseaudio
lsof /dev/snd/*
speaker-test -D hw:1,3 -c2
sudo usermod -a -G audio mpd
sudo systemctl restart mpd
```

to trouble shoot i also enabled the log in the config and then:

```
sudo touch /var/log/mpd/mpd.log
sudo chown mpd:mpd /var/log/mpd/mpd.log
```

i could restart mpd and use `sudo tail -f /var/log/mpd/mpd.log`


## Client

sudo apt-get install ymuse

It might be confusing but this will basically just be like a remote control and you listen to http://10.1.0.1:9000 for the stream.... it's the same for malp on android (fdroid) which use vlc to stream fine...

## Bonus

On my server I like to also run projectM while just listening to the http stream...

Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/music-server-mpd.gopher.txt](gopher://gopher.someodd.zip:70/0/phlog/music-server-mpd.gopher.txt)
