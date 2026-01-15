---
date: 2025-10-15
tags:
- sysadmin
- linux
- security
- debian
- window_maker
title: KeepassXC as Key Ring Manager

---


You can use [KeepassXC](https://keepassxc.org/) as your "key ring manager," specifically your *SSH Agent* and *Secret Service Integration*.

I am writing from the perspective of a [Window Maker](https://windowmaker.org) user. This was tricky for me because nothing else I tried seemed to integrate well into Window Maker, yet I was already using KeepassXC.

# Disable Gnome Keyring Daemon (if you have it)

I honestly recommend just removing the Gnome and KDE keyring managers because you may keep running into issues.

Kill the Gnome Keyring Daemon:

```
pkill -f gnome-keyring-daemon || killall gnome-keyring-daemon 2>/dev/null || true
pgrep -a gnome-keyring-daemon || echo "no gnome-keyring running"
```

This might work to permanently disable:

```
systemctl --user stop gnome-keyring.service gnome-keyring-secrets.service 
gnome-keyring-ssh.service gnome-keyring-gpg.service 2>/dev/null || true

systemctl --user mask gnome-keyring.service gnome-keyring-secrets.service 
gnome-keyring-ssh.service gnome-keyring-gpg.service
```

UPDATE: honestly, I just went ahead and uninstalled the Gnome Keyring manager and uh Seahorse or whatever it's called, entirely. It kept finding its way back into hijacking my secrets/key manager.

# Set up KeepassXC SSH Agent

Prepare the `SSH_AUTH_SOCK` environment variable (in y our `~/.bashrc` or `~/.zshrc` or whatever):

```
export SSH_AUTH_SOCK=$(ls /tmp/ssh-*/agent.* 2>/dev/null | head -n1)
```

Reload with `source ~/.zshrc` for example. You can check the change:

```
echo $SSH_AUTH_SOCK
ssh-add -l
```

In KeepassXC go to settings -> SSH Agent:

* Enable SSH Agent integration (checked)
* `SSH_AUTH_SOCK` override might be set to something like `/run/user/1000/ssh-agent.socket`, but the env var trick I did just detects the value (path changes each login?)

Now you can actually keep your keys in KeepassXC, including their passwords, and KeepassXC will handle it for you. For example, add a new entry to the database:

* Enter the password for the key in the "password" field.
* Attach the key under *advanced*
* Under *SSH Agent* section, tick "add key to agent when database is opened/unlocked" and for private key actually select the key you attached in the *attachment* drop down.

# KeepassXC Secret Service Integration

In KeepassXC *settings* -> *Secret Service Integration*:

* Enable KeepassXC Freedesktop.org Secret Service Integration
* Add a password group to be exposed to "exposed database groups."
* Unchecked: *confirm when passwords are retrieved by clients*

If you're not able to enable it, it very well may be that another integration is running.

Now that it's enabled, I believe whenever you enter passwords, it'll actually get saved to this group you created/use for the exposed database group. For example, my Gajim/XMPP passwords got saved there. I think you'll see KeepassXC pop up with some kinda confirm sometimes regarding this.

# Other notes

Make sure KeepassXC is unlocked if you want this system to work!

If you ever notice this system isn't working, check to make sure `gnome-keyring-daemon` isn't running.

Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/keepass-keyring-manager.gopher.txt](gopher://gopher.someodd.zip:70/0/phlog/keepass-keyring-manager.gopher.txt)
