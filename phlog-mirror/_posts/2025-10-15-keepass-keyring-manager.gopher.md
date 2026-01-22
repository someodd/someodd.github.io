---
date: 2025-10-15
image:
  caption: KeepassXC's SSH Agent Settings
  path: /assets/posts/keepass_keyring/keepass_ssh_agent.png
  thumbnail: /assets/posts/keepass_keyring/keepass_ssh_agent.png
tags:
- sysadmin
- linux
- security
- debian
- window_maker
title: KeepassXC as Key Ring Manager for Minimal DEs & WMs

---


Use KeePassXC as your Key Ring Manager in non-GNOME/KDE big DE setups, e.g.,
Window Maker, i3, Xmonad.

This is how to use KeePassXC as:
- your SSH key manager
- your Secret Service (org.freedesktop.secrets)
- without GNOME, KDE, or keyring daemons

The goal:
One stable ssh-agent socket.
Everything talks to it.
KeePassXC loads keys into it.

No popups. No race conditions. No broken SSH.

--------------------------------------------------

## 1. Create a systemd user ssh-agent with a fixed socket

Create:

~/.config/systemd/user/ssh-agent.socket

```
[Unit]
Description=SSH Agent Socket

[Socket]
ListenStream=%t/ssh-agent.socket
SocketMode=0600

[Install]
WantedBy=sockets.target
```

~/.config/systemd/user/ssh-agent.service

```
[Unit]
Description=SSH agent
Requires=ssh-agent.socket
After=ssh-agent.socket

[Service]
Type=simple
Environment=SSH_AUTH_SOCK=%t/ssh-agent.socket
ExecStart=/usr/bin/ssh-agent -D -a $SSH_AUTH_SOCK
```

Enable it:
```

systemctl --user daemon-reload
systemctl --user enable --now ssh-agent.socket

```

Your agent now lives at:

```
/run/user/<UID>/ssh-agent.socket
```

NOTE: `<UID>` is probably `1000`.

--------------------------------------------------

## 2. Export SSH_AUTH_SOCK everywhere

Create:
~/.config/environment.d/10-ssh-agent.conf

```

SSH_AUTH_SOCK=${XDG_RUNTIME_DIR}/ssh-agent.socket

```

Also force it in zsh (important for minimal WMs):
Put this at the very top of ~/.zshrc

```

export SSH_AUTH_SOCK="${XDG_RUNTIME_DIR:-/run/user/$(id -u)}/ssh-agent.socket"

```

This prevents old or broken agents from hijacking your shell.

--------------------------------------------------

## 3. Tell KeePassXC to use that socket

![KeepassXC's SSH Agent Settings](/assets/posts/keepass_keyring/keepass_ssh_agent.png)

KeePassXC -> Settings -> SSH Agent

Enable:
- SSH Agent integration

Set:
- SSH_AUTH_SOCK override:
/run/user/1000/ssh-agent.socket
(use your UID)

KeePassXC is now a client of the real agent.

--------------------------------------------------

## 4. Enable Secret Service (Linux keyring)

![KeepassXC's Secret Service Settings](/assets/posts/keepass_keyring/keepass_secret_service.png)

KeePassXC -> Settings:
Enable "Freedesktop.org Secret Service integration"

Open your database:
Database Settings -> Secret Service Integration
Choose a group to expose.

KeePassXC now replaces:
- gnome-keyring
- kwallet
- gcr

--------------------------------------------------

## 5. Verify

You'll likely want to logout/login.

```

echo $SSH_AUTH_SOCK
ssh-add -l

```

You should see:
```

/run/user/UID/ssh-agent.socket

```
and your keys listed.

Don't forget to check the box to add a key to the keyring for the respective
ssh key entries in KeepassXC!

If you have any troubles, it may just be that another keyring manager is being
annoying. I found Gnoe's keyring manager was such a hinderence I uninstalled
it.

--------------------------------------------------

## Result

KeePassXC is now:
- your SSH key loader
- your password store
- your Linux keyring
- your single source of trust

Works in:
- Window Maker
- i3
- sway
- Xmonad
- bare X11
- no GNOME
- no KDE

One agent.
One socket.
No magic.

Original content in gopherspace: [gopher://gopher.someodd.zip:70/1/phlog/keepass-keyring-manager.gopher.txt](gopher://gopher.someodd.zip:70/1/phlog/keepass-keyring-manager.gopher.txt)
