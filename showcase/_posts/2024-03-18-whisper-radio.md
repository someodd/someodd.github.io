---
layout: landing
title: "Whisper Radio"
date: 2024-03-18
categories: showcase
tagline: Automated web radio, using AI.
tags: ai bash media whisper-radio
logo: /assets/showcase/whisper-radio/whisper-radio-logo.png
image:
  path: /assets/showcase/whisper-radio/ai-generated-robot-radio-host.webp
  thumbnail: /assets/showcase/whisper-radio/ai-generated-robot-radio-host.webp
  caption: AI-generated image of a robot radio host.
---

## Look ma, no human DJ!

Whisper Radio is a (hopefully) 24/7 Internet radio station "hosted by a bot."

Text-To-Speech (TTS, both AI and older-school) is leveraged to dynamically read news, texts, trends, announce the next song, report the weather, the freshest thread post from my forum, and maybe more! Configurable!

Maybe the coolest feature is the interactive feature where the "host" replies to posts on [Fosstodon](https://fosstodon.org), [under a specific hashtag](https://fosstodon.org/tags/whisperradio).

[Open Stream](https://radio.someodd.zip/stream){:.button .primary}
[Listen Details](#listening){:.button}
[View Source](https://github.com/someodd/whisper-radio){:.button}

## Features

* AI: AI voice announces some segments, reads news, and more!:  fa-robot
* News: Dynamically makes headlines, forum posts, and more a part of the broadcast!: fa-newspaper
* Interactive: Users can send in messages for the AI to respond to on the broadcast!: fa-reply
* Configurable: you can change the news sources and the like.
* Reads freshest thread's post from [gopherden](/showcase/gopherden)

This is probably a non-fully-inclusive list of features!

[Listen](#listening){:.button .primary}
[View Source](https://github.com/someodd/whisper-radio){:.button}

## Backstory

Lovingly named after the [espeak](https://espeak.sourceforge.net/) voice which reads some of the segments.

I got the idea one night to have a radio station that is just TTS reading out
random things like... or whatever. I quickly hacked together some using bash
within 24 hours, basically.

Making this project was fun! It was interesting hacking various tools together with Bash. I feel like my Haskell experience when it comes to statelessness/immutability maybe came in handy when it came to scripting parts of this project.

One of the most annoying, yet oddly fun, parts of the project was lots of copyright-compatible text and audio.

## Listening

Simply listen with the player below.

<audio controls>
  <source src="https://radio.someodd.zip/stream" type="audio/mp3">
</audio>

You can listen to the live demo on [the official Whisper Radio
station](https://radio.someodd.zip/stream)! You can also open that link in
an audio player (like Audacious) which supports streaming data/links or
whatever. It's a regular HTTP link, not HTTPS. Which some browsers these days
may have trouble with, because they'll try to force HTTPS, maybe.

I have used VLC to listen to it on my phone.

## News

whisper-radio

## Known issues

* I think I've noticed it stopp working a few times, but it's been stable for a long time now.

## Feedback

* Take a look at [my about page](/about) for my contact info.
* [Create an issue on the GitHub repo](https://github.com/someodd/whisper-radio/issues/)

## Who's using, in the news

Not much!

## Documentation & tutorials

* [Project README](https://github.com/someodd/whisper-radio/blob/master/README.md)

## See also

* [Icecast stream list](http://dir.xiph.org/)

## Running your own

Read [the project's README.md](https://github.com/someodd/whisper-radio). it'll be more reliable for big parts of this and I
won't be going over everything here.

I started setting up inside a screen session i use for server stuff. i created a new window (Simply press `Ctrl-a` followed by `c`).

```
git clone https://github.com/someodd/whisper-radio
cd whisper-radio
```

I installed the dependencies. I also ran `pipx ensurepath`.

Icecast may ask for FQDN--this is what people will listen on. I entered `radio.someodd.zip`. Then maybe enter and note your source password (I think this is to manage the server?) and some other credentials I guess. Note those!

Download the Piper voices:

```
curl -o en_US-hfc_female-medium.onnx.json "https://huggingface.co/rhasspy/piper-voices/resolve/v1.0.0/en/en_US/hfc_female/medium/en_US-hfc_female-medium.onnx.json"
curl -o en_US-hfc_female-medium.onnx -L "https://huggingface.co/rhasspy/piper-voices/resolve/v1.0.0/en/en_US/hfc_female/medium/en_US-hfc_female-medium.onnx?download=true"
```

### icecast2 server setup

when you first install it'll ask you for username and password or whatever.

here's some stuff I set in the `/etc/icecast2/icecast.xml` (don't just blindly copy):

`sudo vi /etc/icecast2/icecast.xml`

```
<hostname>radio.someodd.zip</hostname>
```

... was set in the TUI I guess... replace with your FQDN

you may want to set up the contact info.


#### HTTPS

Setting up HTTPS for the Icecast server.

To enable HTTPS (I recommend), you can comment out this line, so you can listen
via HTTP and HTTPS:

```
    <listen-socket>
        <port>8443</port>
        <ssl>1</ssl>
    </listen-socket>
```

For HTTPS (I found [this
guide](https://www.mediarealm.com.au/articles/icecast-https-ssl-setup-lets-encrypt/)
useful) you'll also uncomment (also change name to `bundle.pem` maybe):

```
<ssl-certificate>/etc/icecast2/bundle.pem</ssl-certificate>
```

Install certbot:

```
sudo apt-get install certbot
```

Then I did this (select `webroot` when asked):

```
certbot certonly --webroot-path="/usr/share/icecast2/web" -d 'radio.someodd.zip'
```

Please note for the above, it'll want to do an HTTP validation check. The way I
handled this was port forwarding 80 on my router to my server's 8000. I think
there's an alternative to HTTP validation or whatever, but this method just
seems the most painless. It'd just be security through obscurity to worry about
port number, here, I guess, anyway.

I also port forward 443 to 8443 on server.

I got something like this:

```
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/radio.someodd.zip/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/radio.someodd.zip/privkey.pem
This certificate expires on XXX
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.
```

Then create the bundle file referenced earlier:

```
sudo cat /etc/letsencrypt/live/radio.someodd.zip/fullchain.pem /etc/letsencrypt/live/radio.someodd.zip/privkey.pem > /etc/icecast2/bundle.pem
```

You may want to actually use `chown` to change the permissions of `bundle.pem`
to whatever user Icecast runs under. For me I saw (in `/etc/default/icecast2`):

```
USERID=icecast2
GROUPID=icecast
```

So I ran:

```
chown icecast2:icecast /etc/icecast2/bundle.pem
```

Ensure certificate renewals run correctly (I edited
`/etc/letsencrypt/renewal/radio.someodd.zip.conf`) in the section of `[renewalparams]`:

```
post_hook = cat /etc/letsencrypt/live/radio.someodd.zip/fullchain.pem /etc/letsencrypt/live/radio.someodd.zip/privkey.pem > /etc/icecast2/bundle.pem && service icecast2 restart
```

Validate that the above command works correctly:

```
sudo certbot renew --dry-run
```

*An interesting caveat: i noticed it could not renew if the stream wasn't up (certbot will fail).*

### ufw

For HTTP:

```
sudo ufw allow 8000/tcp comment 'icecast/whisper radio'
```

And for HTTPS (if you want):

```
sudo ufw allow 8443/tcp comment 'icecast/whisper radio (SSL)'
```

### Other security measures

I didn't like that the admin web interface was exposed to the internet...

### start the server + crontab it!

don't forget to port forward on router! tcp.

### Troubleshooting/bug

Try loading the main playlist with `ezstream` manually if you have any troubles:

```
ezstream -v -c ezstream.xml
```

The script just uses `piper` for the piper
command, but pipx installs things into a local/user directory by default or whatever, not
globally, by default, so you need to edit `whisper.sh` and change `piper` to
whatever `which piper` evaluates to. Of course this is just if you take care of
the `piper` dependency by simply doing a user-level install using `pipx install
piper` or whatever.

I think it may be a good idea to look out for outputted files that are VBR
(variable bit rate) or somehow vary from other files in bit rate. You may
experience something playing too fast, the stream seemingly ending until you
reload the stream, other weird glitches.

Don't forget to port forward! This setup uses port 8000 and 8443, so you'd want
to forward 80 to 8000 and 443 to 8443.

I had an issue where it was looping one of the segments (weather) over and
over. I `pkill ezstream`, deleted the playlists and output files and then ran
`./whisper.sh` again.

### Putting behind `nginx`

the benefit of this is i wanted to hide the admin stuff and only expose stream and plus i want to have other http stuff on this server.

```
sudo apt-get install nginx
sudo vi /etc/nginx/sites-available/radio.someodd.zip.conf
```

Then in the config:

```
server {
    listen 8765;
    listen 8888 ssl;
    server_name radio.someodd.zip;
    root /var/www/radio.someodd.zip;

    ssl_certificate /etc/letsencrypt/live/radio.someodd.zip/cert.pem;
    ssl_certificate_key /etc/letsencrypt/live/radio.someodd.zip/privkey.pem;

    location /stream {
        proxy_pass https://localhost:8443;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

}
```

Then:

```
sudo mkdir /var/www/radio.someodd.zip/
sudo ln -s /etc/nginx/sites-available/radio.someodd.zip.conf /etc/nginx/sites-enabled/
```

and `sudo service nginx restart`

If you do this you may want to delete the other whisper radio ufw entries and then add this new one:

```
sudo ufw allow 8888/tcp comment 'general nginx https'
sudo ufw allow 8765/tcp comment 'general nginx http'
```

gotta modify for letsencrypt...

don't forget forward the ports--80->8765 and 8888->443 (tcp)

gotta modify for letsencrypt `sudo vi /etc/letsencrypt/renewal/radio.someodd.zip.conf`:

```
webroot_path = /var/www/radio.someodd.zip/,
```

maybe even set the webroot_map to this too if that's a thing

test with `sudo certbot renew --dry-run`
