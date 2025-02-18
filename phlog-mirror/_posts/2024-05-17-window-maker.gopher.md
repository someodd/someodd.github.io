---
date: 2024-05-17
tags:
- linux
title: Productivity in Window Maker

---
Original content in gopherspace: gopher://gopher.someodd.zip:Just 7071/phlog/


I use the old school, lightweight [Window Maker](http://www.windowmaker.org/)
[window manager](https://en.wikipedia.org/wiki/Window_manager) on Debian
Unstable.

Since Window Maker is only a "window manager" and not a full environment it may
feel backwards to use it in 2024, there's a lot of things you may miss, a lot
of work to do from a fresh install.

I'm biased toward selecting apps that are more Window Maker-like in some way or
just fit the feel.

You may also want to take a look at [Window Maker
Live](https://wmlive.sourceforge.net/). Possibly a good way to try (or even
install) a Window-Maker-centric Debian setup.


## Reasons you may want to use Window Maker

  * Looks very cool
  * Lots of handy (and just plain cool) dockapps (think built-in system info
    tools)

## Things I want to add to this article

- [ ] key ring manager
- [ ] custom obj for wmcube (maybe write a script)
- [ ] my amor buddy--could share.
- [ ] deja-dup
- [ ] cursor theme
- [ ] icon theme
- [ ] power daemon management (efficient/battery save vs performance)
- [ ] touchpad
- [ ] i don't like middle click paste.
- [ ] xsreensaver lock broken? hotkey... startup problem?
- [ ] battery performance change (like battery savery mode)
- [ ] adjust brightness with gui
- [ ] tap touchpad to click

## Window Maker-specific and Appearance

### Dock apps

Dock apps for Window Maker. Dock apps are such a cool feature of Window Maker.

Check out [dockapps.net](https://www.dockapps.net/)!

You can `sudo apt-get install wm..` for these dockapps I mention below. The
dockapps I've tried seem to all have good man pages, I think.

#### What I'm using

  * `wmclock`: I like this because it shows the time and the date as a
    tear-away date pad graphic.
  * `wmbattery` and `wmacpi`: two different battery-relaed dockapps! A special note for launch command for `wmbattery`:

   * `wmbattery` lets you execute a command when the battery is below critical.
     Here's a command that will send a notification that the battery is
     critically low):
     ```
     wmbattery -c 10 -l 30 -a /home/tilde/Music/sfx/sosumi.au -x "/usr/bin/espeak -v en-us+whisper 'critically low battery' -a 200 -s 130 && /usr/bin/notify-send -w -u critical -i /usr/share/WindowMaker/Icons/timer.tiff 'Low Battery' 'Battery at %percent%%, with %minutes% minutes left.'"
     ```
     Note the full/abs/real paths. The audio file is an `.au`, I feel it's kind
     of hard to find `.au` files these days. Also this command seems to do
     something strange to my audio, so maybe don't use `-a`.

  * `wmbubble`, `wmcube`, `wmforkplop`, `wmmon`, `wmtop`: fun and/or
    informative dockapps for system information, a few of which are veyr
    visually interesting/fun to me.

    * `wmbubble`: bubbles and duck--animations get more intense/fills up as the
      system resources get more utilized.

  * `wmcliphist`: keeps (some?) clipboard history
  * `wmsystemtray`: I feel this one is sort of crucial. A system tray for app
    icons like `blueman-applet`, `nm-applet`, `redshift-gtk`, or the like.

    * I have some notes about how it can't be ran along side other system trays, or something?

  * `wmmixer`: I really like the way it looks when I turn the volume up and
    down with this app and the fact that I can twist a graphical knob to do so
    as well.
  * `wmweather+`: gives me weather data, but it's sort of complicated to use. I
    think you need to find a "metar- station", and define the long+lat, with a
    command like this: `wmweather+ -metar-station SOMEID -location "0.0N 0.0W"`.  It
    seems to have some neat features if your metar station supports it? You may
    want to also get radar image from [National Weather
    Service](https://forecast.weather.gov/MapClick.php). Please `man wmweather+`.

    * I haven't figured out getting actual forecast working outside of current weather
    * `wmweather+ -s SOMEMETARHERE -location "0.0N 0.0W" -radar-uri https://radar.weather.gov/ridge/standard/SOMESTATION_loop.gif -radar-crop POSXxPOSY+WIDTH+HEIGHT -radar-cross 244x194 -animate`

  * `wmforecast` simple forecast/temperature app. More iconified and simple
    than `wmweather+`. Displays handy info in a hover/tooltip, including
    forecast. This may be the easiest to work with and is perhaps the easiest to
    work with.

##### wmbiff

wmbiff needs a config
i had an example at `/usr/share/doc/wmbiff/examples/sample.wmbiffrc`

```
cp /usr/share/doc/wmbiff/examples/sample.wmbiffrc ~/.wmbiffrc
```

supports pop3 and imap
i think i was able to configure this to work with Protonmail Bridge pretty easily.

```
label.4=IMAP4
path.4=imap:me@pm.me:password@127.0.0.1:1143
interval.4=300          # 5 minutes
action.4=claws-mail
```

I also set this to notify me:

```
globalnotify=aplay /home/tilde/Music/sfx/youvegotmail.wav
```

I don't reall like how this app looks, but its features are great!

#### Recommendations

  * `wmnut`: Keep track of a Network UPS on the network via NUT
  * `wmitime`:  I feel it's a nice, but confusing clock display. I think it
    supports [Swatch Internet Time](https://en.wikipedia.org/wiki/Swatch_Internet_Time)

  * `wmfire`: monitor cpu, memory, network, or file with a fiery animation. I
    think this is cool, but I think there's maybe a bug where I can't drag it
    into my dock.

#### Honorable mentions

These may be handy or neat:

  * `wmoonclock`: shows phase of the moon! can click to view other info. how to i configure this?
  * `wmxres`:  set the resolution/change between xorg modes!
  * `wmressel`: I think this is basically the same as `wmxres`
  * `wmsun`: displays current day's sun rise and sun set times.
  * `wmshutdown`: button for shutting down
  * `wmstickynotes`: a really great sticky notes dockapp. You click the pad and
    a sticky note pops up you can place anywhere on the screen.
  * `wmpinboard`: notes you can place (and maybe draw in) and have pinned on a
    little corkboard graphic
  * `wmpuzzle`: a sliding puzzle game
  * `wmclockmon`: lcd clock that seems to work well, I just don't like the look.

Some more:

  * `wmsysmon`: monitors CPU usage, memory, swap, uptime, and interrupts, but I'm not very fond of it.
  * `wmgtemp`: system temp dockapp
  * `wmtemp`: system temp app with lcd screen
  * `wmcore`: graph-like display of the usage of each core
  * `wmmon`: monitors realtime CPU load as well as average system load (graph I think)
  * `wmcalc`: a little calculator
  * `wmfrog`: I think it's a kind of silly-looking weather app. Launch with
    something like `wmfrog -s METARSTATIONID`.
  * `wmxmms2`: "A dockable XMMS2 client." I don't use xmms2. Seems pretty nice.
  * `wmtv`: I find this interesting and wish I could see it in action! maybe one
    day. video4linux TV player.

#### I didn't like/didn't work for me

  * `wmbutton`: a grid of buttons which launch apps or whatever
  * `wmail`: email/inbox docklet which wants qmail's Maildir format or mbox.
  * `wmfsm`: disk space avaiable
  * `wmweather`: another weather app. i find it hard to read and I don't like the look.
  * `wmrack`: crashes for me. "CD Player and Mixer dock applet."
  * `wmwave`: "statistical information for wireless ethernet."
  * `wmdocker`: I had a note about this working funny. I think it's probably
    better to just use `wmsystemtray` instead.
  * `wmcpuload`: i don't like how it's displayed
  * `wmcpu`: I had a note about it being hard to read, basically.

#### Not relevant to me, or just other ones I tried

In this section the DockApps just weren't relevant to me, I didn't care to put
much time into using them, or they're just here...

  * `wmifs`: I think it's an OK bandwidth usage indicator. I dislike that it
    didn't seem to display the names of the interfaces.
  * `wmmon`: works but its not pretty enough. other apps do same but better.
  * `wmcdplay`: I don't have a CD player on my laptop!
  * `wmget`: maybe a download manager? seems kind of annoying to use.
  * `wmdiskmon`: I get some kind of error. LCD-display-style disk usage monitor?
  * `wmhdplop`: "monitor hard-drive (or partition) activity."
  * `wmnet`: I have a note "broke or too hard configure"
  * `wmdrawer` needs config file. didn't really try. maybe i'm wrong, but I'm prety
    happy with what I think is the built-in drawer thing in Window Maker.

#### custom wmcube object

You can actually use a custom object for the wmcube, but I think it needs a
weird custom format. Maybe I'll write a conversion script.

#### Dockapp Archive

[Archive.org archvie of dockapps.windowmaker.org](https://web.archive.org/web/20121122074114/http://dockapps.windowmaker.org/)

#### Making a dock app

There's a ruby SDK, but I wonder if I could bind haskell to the c++ myself?
that'd be a great project.

[ruby-dockapp](https://www.dockapps.net/ruby-dockapp): "Ruby-DockApp is a ruby extention library for making dockapps."

#### Have a favorite dockapp I haven't mentioned?

Please email me!

### Window doesn't actually focus on click?

It took me a while to realize, because of my track pad settings, that I would
think I'm left clicking to swith focus to another window, and it wouldn't bring
it to the front yet it would focus it. I initially thought it was a bug. Turns
out I was middle clicking the window instead of left clicking it. Interesting.

### Application Menu

edit `~/GNUstep/Defaults/WMRootMenu`

```
(
  Debian,
  (
    Applications,
    OPEN_PLMENU,
    "|| wmmenugen -parser:xdg /usr/share/applications/"
  ),
  (Run..., EXEC, "%A(Run,Type command to run)"),
  (
    "Window Maker",
    ("Info Panel ...", INFO_PANEL),
    ("Legal Panel ...", LEGAL_PANEL),
    (Preferences, EXEC, WPrefs),
    ("Refresh screen", REFRESH),
    (Restart, RESTART)
  ),
  (
    WorkSpace,
    (Appearance, OPEN_MENU, appearance.menu),
    ("Arrange Icons", ARRANGE_ICONS),
    ("Clear Session", CLEAR_SESSION),
    ("Hide Others", HIDE_OTHERS),
    ("Save Session", SAVE_SESSION),
    ("Show All", SHOW_ALL),
    (Workspaces, WORKSPACE_MENU)
  ),
  (Exit, EXIT),
  ("Exit session", SHUTDOWN)
)
```

Just edit the rest through the app editor thing.


### Add things to startup

I add things I want to autostart to the Window Maker autostart script (followed
by `&`, like below):

```
➜  ~ cat ~/GNUstep/Library/Window Maker/autostart
#!/bin/sh
blueman-applet &
amor &
xscreensaver --no-splash &
xeyes &
xpenguins -a -b -t "Big Penguins" &
oneko &
redshift-gtk -m randr -l 37.8044:122.2712 &
deja-dup &
nm-applet &
```

### App icons

I wanted to change the Thunar icon for my launcher (or whatever it's called) so
I did this and was able to select it through something like *settings > icon
image*:

```
cp /usr/share/icons/hicolor/48x48/apps/org.xfce.thunar.png ~/GNUstep/Library/Icons/thunar.png
```


### GTK theme switch & theme

Some apps use GTK for their GUI.

For appearance consistency you may want to install a GTK theme which
compliments Window Maker and some tool for managing the GTK theme you're using.

I like to use `lxappearance` to manage my GTK theme (I used `sudo apt-get install lxappearance`).

I installed and use [the redmond97 GTK
theme](https://github.com/matthewmx86/Redmond97/tree/master/Theme),
specifically the *redmond cde* theme, to match the overal purplish thing I have
going on. I think it matches the *SteelBlueSilk* Window Maker theme.

Bonus: https://github.com/mgsander/wmstep/tree/master/WMStep:
something I found but I didn't get working.

I went ahead and disabled the GTK window decoration hints or the like in the
advanced section of WPrefs or something. I think this maybe makes things look
more consistent.

### Adding hotkeys

You can add hotkeys by editing the Window Maker menu through WPrefs, under
*Applications Menu Definition*. I like to create a special *submenu* that holds
all the *run program* entries which have hotkeys associated to them.

For example, I set lock (`xscreensaver-command -lock`) to my super key + l.

I think sometimes (?) you may need to restart the session for hotkeys to come into effect.

#### Control Screen Brightness

Use `brightnessctl set 10%-` and `brightnessctl set 10%+` then add to menu and assign hotkeys.


## Essential programs

### Other/quick mentions

  * RSS: Liferea

### Login manager

I recommend using LightDM as your login manager. One thing I like about it is I
can switch the environment/WM I log into. This can be handy because sometimes
you come across something like how Waydroid only runs in Wayland.

### Archivers

xarchiver

### Browsers

`firefox-esr`My main web browser is just the Debian-provided Firefox.

https://github.com/dillo-browser/dillo -- you actually may not want to use the repo version and build it from there, because of the time of writing this i'm told the repo verison is ten years old. https://github.com/dillo-browser/dillo/blob/master/doc/install.md gopher plugin: https://github.com/dillo-browser/dillo-plugin-gopher

### KeepassXC

Password manager.

### deja-dup

I just find it's very reliable for backups and easy to use.

### audacious

Audio player.

The gtk audacious is great! the hotkeys seem to mostly work out-of-the-box for
what I've used, I think? You can use Winamp skins if you want to go the extra
mile. If you want to go even further, milkdrop is available for linux.

### Language switcher

I'm using IBUS.

IBus is an intelligent input bus for Linux/Unix.

it shows up as a language switcher in the system tray. i don't know how or why.
i can switch with super + space.

### Thunar

I feel like Thunar is a wonderful file manager with a great amount of features
and fits the Window Maker feel and lightness and kidn of retro look.

```
sudo apt-get install thunar
```

### Bluetooth and Wifi GUI

These two apps will enable a GUI through the system tray (dockapp).

I use `nm-applet` for all my needs. I add it to `autostart`.

i also use `blueman-applet` (you can just install through `blueman` package), add it to `autostart` for Window Maker.

### xfce4-terminal

I find that this is a nice terminal to use in Window Maker.

### Screenshots

Scrot and Maim didn't seem to work well with WindowMaker hotkey or the like. So I'm using `xfce4-screenshooter`, which seems to segfault if I capture window border when capturing the active window.

I made two entries:

* Active window (alt + prt scr): `xfce4-screenshooter -w --no-border`
* Default (prt scr): `xfce4-screenshooter`

### xscreensaver

The power management features are also nice and it provides the ability to lock
the screen.

don't forget to add to autostart

For images you may be able to set both the text manipulation and the random image to atom/rss feeds:

* https://planet.debian.org/rss20.xml - debian news
* You can search a "booru" like Konachan which has wallpapers and provides RSS/ATOM feeds for search results, and you can specifically filter by "safe"
  * Some early 2000s, late 90s vibes: https://konachan.net/post/atom?tags=SOMETAG+rating%3Asafe
* https://photojournal.jpl.nasa.gov/rss/index.html

### redshift

I like using redshift to change the color temperature. I installed `redshift-gtk` to get the system tray:

```
sudo apt-get install redshift redshift-gtk
```

You can run with a command like `redshift-gtk -m randr -l long:lat`.

Beware as north is represented as a positive number and west is represented as
a negative number. This messed me up for a bit.

Don't forget to add to Window Maker's startup.

Example `~/.config/redshift.conf` config:

```
[redshift]
; Set the day and night screen temperatures
temp-day=5700
temp-night=3500

; Enable/Disable fade effect (0 or 1)
fade=1

; Set the location provider: 'manual' (for manual geolocation) or 'geoclue2' (for automatic geolocation)
;location-provider=geoclue2
location-provider=manual

; Set your location if you're using manual geolocation
[manual]
lat=...
lon=...
```

### Claws/claws-mail

I use this email client with [Protonmail Bridge](https://proton.me/mail/bridge)
and `wmbiff` (mentioned in this document).

Really feels like it fits the spirit of Window Maker, to me.

I also recommend installing `claws-mail-plugins` and `laws-mail-extra-plugins`.

#### Configuration

I think I did create `~/.claws-mail/queue` and set it as the
*Preferences for current account* then *advanced* tand *put queued messages in*
and I used the absolute path/realpath, because it complained about the queue
directory or something. even then it didn't work

I finally set the queue folder to the IMAP *Drafts* folder or whatever and now
it works!

It may also complain about not being able to open signature.

### Bonus software+ apps i like using with

shotwell for viewing photos

deja-dup has been good to me. but does it need something to actually launch backups in gui more than just running in bg or whatever?

For torrents I tried using `transmission-qt` for a while, but actually that doesn't look right and I experienced some problems with it. I highly recommend just using Deluge for torrents. I do think `transmission-daemon` is great for servers, though.

ted
https://www.nllgg.nl/Ted/#How_to_install_Ted

I recommend installing libreoffice-gtk3 for office. lyx seems interesting but I haven't used it much yet. also sudo apt install texlive-full

XMPP client: gajim (seen below, connected to my XMPP server which is also connected to my IRC server, edited to respect privacy):

![Gajim XMPP client connected to my XMPP server which is also connected to my IRC server](/showcase/xmpp-server/gajim-connected-someodd-xmpp-irc-censored.png)


## Set default apps

sudo update-alternatives --config x-www-browser

xdg-mime default Thunar.desktop inode/directory

```
➜  ~ vim .config/mimeapps.list 
➜  ~ xdg-mime default Thunar.desktop inode/directory

➜  ~ update-mime-database ~/.local/share/mime

➜  ~ mimeopen -d Downloads
Please choose a default application for files of type inode/directory

	1) Visual Studio Code  (code)
	2) VSCodium  (codium)
	3) Konqueror  (kfmclient_dir)
	4) Files  (org.gnome.Nautilus)
	5) Disk Usage Analyzer  (org.gnome.baobab)
	6) Dolphin  (org.kde.dolphin)
	7) Gwenview  (org.kde.gwenview)
	8) Kate  (org.kde.kate)
	9) Thunar File Manager  (thunar)
	10) Other...

use application #9
Opening "Downloads" with Thunar File Manager  (inode/directory)
➜  ~ 

```

## Bonus

### rofi

great for...

```
sudo apt-get install rofi
```

create `~/.config/rofi/config.rasi`:

```
configuration {
  display-drun: "Applications:";
  display-window: "Windows:";
  drun-display-format: "{icon} {name}";
  show-icons: true;
  icon-theme: "Papirus";
}
```

look at the commands possible:

```
rofi
```

you can use as an application launcher, window switcher... i'd like to install the emoji selector but i'm feeling lazy.

add to various key shortcuts hotkeys, i did this by making entries to my applications menu. some i did:

* windows: `rofi -show window` (win+w)
* apps: `rofi -show drun` (win+r)
* files: `rofi -show filebrowser` (win+f)

### skippy-xd

```
sudo apt-get install git build-essential libx11-dev libxcomposite-dev libxdamage-dev libxrender-dev libxext-dev libxft-dev libxinerama-dev libpng-dev libimlib2-dev libwnck-dev libstartup-notification0-dev
sudo apt-get install libgif-dev

git clone https://github.com/richardgv/skippy-xd.git
cd skippy-xd
make
sudo make install
```

now you can add `skippy-xd --start-daemon` to your autostart... and assign the `skippy-xd --toggle-window-picker` to a hotkey (i added toggle skippy to my hotkeys section in applications menu, basically assigned to super + s)

in `~/GNUstep/Library/WindowMaker/autostart` add `skippy-xd --start-daemon &`

it can also be configured...

## Laptop

## tap to click

```
sudo apt update
sudo apt install xserver-xorg-input-libinput
sudo mkdir -p /etc/X11/xorg.conf.d
sudo nano /etc/X11/xorg.conf.d/40-libinput.conf
```

contents:

```
Section "InputClass"
    Identifier "libinput touchpad catchall"
    MatchIsTouchpad "on"
    MatchDevicePath "/dev/input/event*"
    Driver "libinput"
    Option "Tapping" "on"
EndSection
```

restart your x session.

## Language switcher

I use `ibus-pinyin`.

Add the following command to your Window Maker startup script, such as ~/GNUstep/Library/WindowMaker/autostart:

```
ibus-daemon -drx
```

It should appear in system tray for you to switch between.

I have it configured so windows+ space switches.

ibus-setup


## Toys

### xpenguins

Penguins to walk and fall off windows and more.

```
xpenguins -ab t "Big Penguins"
```

I find it has a good deal of nice config options.

### oneko

Cat chases your cursor. Note there are different flags you can use for different skins.

### kde amor

```
# sudo apt-get update
# sudo apt-get install libqt5x11extras5-dev
# sudo apt-get install libkf5doctools-dev


# apt install build-essential cmake qtbase5-dev libqt5svg5-dev qttools5-dev libkf5windowsystem-dev libkf5configwidgets-dev libkf5xmlgui-dev libkf5dbusaddons-dev libkf5archive-dev libkf5notifications-dev libkf5completion-dev libkf5iconthemes-dev libkf5globalaccel-dev libkf5crash-dev libkf5kcmutils-dev libkf5declarative-dev libkf5service-dev libkf5parts-dev libkf5kio-dev libkf5coreaddons-dev libkf5guiaddons-dev

$ git clone https://github.com/KDE/amor.git
$ cd amor
$ mkdir build
$ cd build
$ cmake ..
# make install
```

install theme: https://www.opendesktop.org/p/1219081

copy theme files something like:

```
 $ mkdir outputtheme
 $ tar zxf sometheme.tar.tar -C outputtheme
 $ sudo cp -r . /usr/share/amor
 
```

#### Make your own custom AMOR buddy

You could do something like the following for a quick-ish custom AMOR buddy or whatever:

```
➜  amor cat examplerc 
# KDE Config File
[Config]
PixmapPath=pics/static
Static=true
Description=Unanimated example
Icon=../preview/example.png

[Base]
Sequence=example.png
HotspotX=0
HotspotY=58

➜  amor realpath examplerc
/usr/share/amor/examplerc
```

With the image in `/usr/share/amor/pics/static/example.png`.

### Expose-like

Want something like the Gnome or Mac OSX Expose feature, where you can hit some hotkey and then see all the windows at once? `skippy-xd` is actually being maintained:

https://github.com/dreamcat4/skippy-xd

All you have to do is:

```
git clone https://github.com/dreamcat4/skippy-xd
cd skippy-xd
make
sudo make install
skippy-xd
```

You can map skippy-xd to whatever.

Unfortunately, this will also show all the little `wm*` dockapps.

### Other

  * xeyes: eyes that look at your cursor.
  * [A thread on Linux Questions about Virtual Pets](https://www.linuxquestions.org/questions/linux-general-1/virtual-pets-for-the-linux-desktop-418186/page2.html)
  * [xsnow](https://en.wikipedia.org/wiki/Xsnow)
  * [xteddy](https://weber.itn.liu.se/~stegu/xteddy/)


## Troubleshooting

### Apps keep autostarting when I don't want them to

I had to figure out why things kept autostarting despite not being in my `autostart` file nor was I saving session on exit nor set in `WPrefs`. Apparently checkout `~/GNUstep/Defaults`. You should make sure to edit it while windowmaker isn't running because of the `.lck` (lock) it creates in the same directory.

I saw in the aforementioned directory a file called `WMState`, which indeed had all the applications autostarting which I didn't want to do so.

https://www.windowmaker.org/docs/chap4.html

