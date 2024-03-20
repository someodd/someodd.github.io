---
layout: post
title: "Whisper Radio: Radio Station Hosted by Bots"
date: 2024-03-18
categories: showcase
tags: ai gopher haskell media
---

[The Whisper Radio software project](https://github.com/someodd/whisper-radio) is software which hosts an automated Internet radio station, using text-to-speech and...

Uses some old school TTS and AI TTS.

I got the idea one night to have a radio station that is just TTS reading out
random things like... or whatever. I quickly hacked together some using bash
within 24 hours, basically.

You can view a live demo on [the official Whisper Radio station](http://radio.someodd.zip:8000/stream)! You can also open that link in an audio player (like Audacious) which supports streaming data/links or whatever. It's a regular HTTP link, not HTTPS. Which some browsers these days may have trouble with, because they'll try to force HTTPS, maybe.

## set up+securing (walk through what i did)

read the project's readme. it'll be more reliable for big parts of this.

I started setting up inside a screen session i use for server stuff. i created a new window (Simply press `Ctrl-a` followed by `c`).

```
git clone https://github.com/someodd/whisper-radio
cd whisper-radio
```

here's the dependencies as of this time:

```
sudo apt-get update
sudo apt-get install espeak jq curl ezstream icecast2 metar ffmpeg pipx ffmpeg
pipx install piper-tts
```

I also ran `pipx ensurepath`.

Icecast may ask for FQDN--this is what people will listen on. I entered `radio.someodd.zip`. Then maybe enter and note your source password (I think this is to manage the server?) and some other credenitals I guess. Note these!

Download the Piper voices:

```
curl -o en_US-hfc_female-medium.onnx.json "https://huggingface.co/rhasspy/piper-voices/resolve/v1.0.0/en/en_US/hfc_female/medium/en_US-hfc_female-medium.onnx.json"
curl -o en_US-hfc_female-medium.onnx -L "https://huggingface.co/rhasspy/piper-voices/resolve/v1.0.0/en/en_US/hfc_female/medium/en_US-hfc_female-medium.onnx?download=true"
```

Remember the credentials you entered? Edit `ezstream.xml` to reflect your source password (replace `hackme`) and I guess also change the path to the playlist (project root + `playlist-main.m3u`):

```
<ezstream>

  <servers>
    <server>
      <hostname>127.0.0.1</hostname>
      <port>8000</port>
      <password>hackme</password>
    </server>
  </servers>

  <streams>
    <stream>
      <mountpoint>/stream</mountpoint>
      <format>MP3</format>
    </stream>
  </streams>

  <intakes>
    <intake>
      <filename>/home/you/Projects/whisper-radio/playlist-main.m3u</filename>
      <stream_once>0</stream_once>
      <shuffle>No</shuffle>
    </intake>
  </intakes>
```

edit `whisper.sh` especially project root

### icecast2 server setup

when you first install it'll ask you for username and password or whatever.

here's some stuff I set in the `/etc/icecast2/icecast.xml` (don't just blindly copy):

`sudo vi /etc/icecast2/icecast.xml`

```
<hostname>radio.someodd.zip</hostname>
```

... was set in the TUI I guess... replace with your FQDN

you may want to set up the contact info.

### ufw

```
sudo ufw allow 8000/tcp comment 'icecast/whisper radio'
```

### start the server + crontab it!

```
sudo service icecast2 start
./whisper.sh
```

If you get `./whisper.sh: line 25: piper: command not found` just try starting new session. In my case i closed the gnu screen window (ctrl d) and then started a new window and started over basically (relaunched whisper).

`crontab -e`:

```
*/5 * * * * /home/you/dir/whisper-radio/whisper.sh > /home/you/dir/whisper-radio/logfile 2>&1
```

don't forget to port forward on router! tcp.

### Troubleshooting/bug

there's currently a bug where the script just uses `piper` for the piper command, but pipx installs things into a local/user directory or whatever, not globally, by default, so you need to edit `whisper.sh` and change `piper` to whatever `which piper` evaluates to.
