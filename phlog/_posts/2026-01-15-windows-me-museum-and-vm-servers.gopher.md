---
date: 2026-01-15
image:
  caption: Screenshot from Windows ME Museum in Dillo.
  path: /assets/posts/me_museum/dillo_screenshot.png
  thumbnail: /assets/posts/me_museum/dillo_screenshot.png
tags:
- sysadmin
- linux
- archival
- my_servs
title: Windows ME Museum

---


Introducing the ME Museum. Play around with Windows ME from The Internet Gopher Protocol (you can also use my gopher proxy).

gopher://gopher.someodd.zip/1/gateway/me_museum/screen

![Screenshot from the Windows ME Museum in Dillo](/assets/posts/me_museum/dillo_screenshot.png)

## Why?

I wanted to play around and let others play around with weird operating systems I like, by setting up a virtual machine server on my Debian server. Why not try out various whacky operating systems and have them networked. Some things I'd like to do are DOS 4.0 (now that it's open source) and an extremely streamlined NetBSD.

# How I did it (rough sketch)

I'll try to open source the scripts. These are the notes I did save. Who knows if this is actually what I did or not, however.

Running to install (this process was so long):

```
sudo qemu-system-i386 \
  -m 256M \
  -cpu pentium2 \
  -hda winme.qcow2 \
  -cdrom 'floppies/Microsoft Windows ME - Millennium Edition (2000).iso' \
  -boot d \
  -vga cirrus \
  -device rtl8139,netdev=net0 \
  -netdev user,id=net0 \
  -display vnc=0.0.0.0:1 \
  -daemonize
```

I was able to use Remmina to VNC into my server like `serverip:5901`. Of course don't forget ufw (since there's no password, I made sure to just restrict it to a specific IP):

```
sudo ufw allow from 10.1.0.2 to any port 5901 proto tcp comment 'VNC for qemu'
sudo ufw allow from 192.168.1.211 to any port 5901 proto tcp comment 'VNC for qemu'
```

You can also use this tool to get snapshots:

```
/usr/bin/vncsnapshot localhost:1 output.png
```

Then just to run the OS:

```
sudo qemu-system-i386 \       
  -m 256M \
  -cpu pentium2 \
  -hda winme.qcow2 \
  -boot d \
  -boot order=dc \
  -vga cirrus \
  -device rtl8139,netdev=net0 \
  -netdev user,id=net0 \
  -display vnc=0.0.0.0:1 \
  -daemonize
```

Now I had to grab [the realtek drivers for LAN](https://archive.org/details/rtl-8139-full-drivers) floppy and mount it.

```
sudo qemu-system-i386 \
  -m 256M \
  -cpu pentium2 \
  -hda winme.qcow2 \
  -fda 'floppies/PCI_100M_ethernet_drivers.img' \
  -boot d \
  -boot order=dc \
  -vga cirrus \
  -device rtl8139,netdev=net0 \
  -netdev user,id=net0 \
  -display vnc=0.0.0.0:1 \
  -daemonize
```

Now I had to go to control panel then add new hardware, select hardware type from list (network) and then open the windows me dir on the floppy. Do not pick `RTL8139C+`,  QEMU’s device rtl8139 emulates the plain 8139, not the C+ variant. Restart. You might have to pkill qemu.

Actually, fuck all that regarding the realtek shit let's just use amdpcnet built into winme:

```
qemu-system-i386 \
  -m 256M -cpu pentium2 \
  -hda winme.qcow2 \
  -boot order=c \
  -vga cirrus \
  -nic user,model=pcnet \
  -display vnc=0.0.0.0:1 \
  -daemonize
```

You'll need to control panel add new hardware, 

Original content in gopherspace: [gopher://gopher.someodd.zip:70/1/phlog/windows-me-museum-and-vm-servers.gopher.txt](gopher://gopher.someodd.zip:70/1/phlog/windows-me-museum-and-vm-servers.gopher.txt)
