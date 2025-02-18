---
date: 2024-10-20
tags:
- gopher
- retro tech
title: Using old terminals (like the VT320) with Linux

---


## VT320/server setup

I have a USB/Serial adapter, but you can just connect via serial. Serial has
different formats. For example, a common one is called something like RS232.

I have a VT320 with a unix-style LK401 (?) keyboard.

### Terminal/VT320 setup: in the terminal config screen

Display:

  * 80 columns
  * interpret controls
  * no auto wrap
  * smooth scroll
  * ligh ttext, dark screen
  * cursor
  * block cursor style
  * no status display

General setup:

  * vt100 mode
  * vt320 id
  * user defined keys unlocked
  * user featuers unlocked
  * numeric keypad
  * normal cursor keys
  * no new line
  * UPSS DEC Supplemental

Communications setup:

  * transmit=9600
  * receive=transmit
  * xoff at 64
  * 8 bits, no parity
  * 1 stop bit
  * no local echo
  * dec-423, data leads only
  * limited transmit
  * no answerback
  * answerback=
  * not concealed

Printer setup:

  * speed=4800
  * no printer to host
  * normal print mode
  * xoff
  * 8 bits, no parity
  * 1 stop bit
  * print full page
  * print national only
  * no terminator

Keyboard setup:

  * caps lock
  * auto repeat
  * keyclick
  * margin bell
  * warnign bell
  * break
  * compose
  * `<X] Delete`
  * `,, and .. keys`
  * `<> key`
  * `n key

Tab setup:

  * clear all tabs
  * set 8 column tabs

I think this is a reference to the tab setup:

```
        T       T       T       T       T               T       T       T       
12345678901234567890123456789012345678901234567890123456789012345678901234567890
```

### Server setup

Let's get the server to accept serial login/TTY. As you can see the device I'm
dealing with is a USB/serial adapter/TTY.

```
systemctl enable serial-getty@ttyUSB0.service
reboot
systemctl start getty@ttyUSB0
ps -ef | grep agetty
```

I think I also edited `/etc/default/grub` (yikes!):

```
#GRUB_CMDLINE_LINUX_DEFAULT="quiet"
GRUB_CMDLINE_LINUX_DEFAULT="console=ttyUSB0 console=tty0"
GRUB_CMDLINE_LINUX=""
```

After editing grub, run `updategrub`.

I think I may have set the speed of the TTY, but I'm not sure if that actually
helped much: `stty -F /dev/ttyUSB0 speed 9600`.

A handy test command: `cat /dev/ttyUSB0` and then try typing stuff on the
terminal.

I got a tip to set `$TERM` variable on the server. Try `echo $TERM`. I think
it may help to set `TERM=vt220` or whatever you have a terminfo entry for.
Try checking `/lib/terminfo/v/` and see the entries.

You don't want to set that in your login script, since it'll be different if
you're in an xterm versus on the terminal itself. Set it in the systemd script
for the getty on the serial line.

Edit `/lib/systemd/system/serial-getty@.service` and change:

```
ExecStart=-/sbin/agetty -o '-p -- \\u' --keep-baud 115200,38400,9600 %I $TERM
```

to:

```
ExecStart=-/sbin/agetty -o '-p -- \\u' --keep-baud 115200,38400,9600 %I vt220
```

## Terminal advice: living inside screen

This advice comes from someone (sorry I forget their name, so no credit).

Type 'screen', and then you get basically a fresh terminal instance.  Now, you
can run whatever programs you want. When you want a new "window", hit
Control-a, then press c.  Tapping control-a a will switch back and forth
between this window and the window you were in.  You can have lots of windows
open, and switch between them with control-a #, where # is the window number
you want to go to. (it starts at 0) There are a whole ton of other things you
can do with screen, of course, but one of the more powerful things you can do
is to type control-a d - which detaches the screen session. Then you can log
out - and when you log back in, you can type 'screen -r' to reattach to the
session you detached.  And... it gets better. If you connect to this machine
from another terminal, or from the network, you can type 'screen -x' on the
other terminal, and attach to the same running screen session again.  So you
have two terminals, both interacting with the same set of "windows".

And you can have different windows open on both terminals.

I just keep reattaching to that same screen session from multiple computers and
terminals, and just use my running environment from multiple places.  I can
even attach to it from my cell phone.

You won't run into this issue with a your terminal, since the command set is
completely compatible, but some other, older, or more unusual terminals kinda
bork when you try to run screen based editors, IRC clients, or stuff like top.
And one of the other nice things about screen is that it properly obeys the
terminfo definition and acts as a translator between poorly coded applications
and your terminal. The application thinks it's using a VT100, but screen
translates it using the terminfo entry and makes it work with whatever you
happen to be using.  So if you're on an old H19 or something, you can still use
irssi without the screen getting all corrupted..

## Recommended apps for terminal users

* sacc: I really enjoy this Gopher protocol client
* weechat: I like this IRC client

Original content in gopherspace: gopher://gopher.someodd.zip:7071/phlog/
