---
date: 2024-05-17
image:
  caption: My Window Maker setup (screenshot).
  path: /assets/phlog/screenshot-window-maker.png
  thumbnail: /assets/phlog/screenshot-window-maker.png
tags:
- windowmaker
- debian
- linux
title: Productivity in Window Maker

---


I use the old school, lightweight [Window Maker](http://www.windowmaker.org/)
[window manager](https://en.wikipedia.org/wiki/Window_manager) on Debian
Unstable.

Gallery: /home/tilde/Projects/gopherhole_bore/assets/posts/window-maker/

Since Window Maker is only a "window manager" and not a full environment it may
feel backwards to use it in 2024, there's a lot of things you may miss, a lot
of work to do from a fresh install.

I'm biased toward selecting apps that are more Window Maker-like in some way or
just fit the feel.

You may also want to take a look at [Window Maker
Live](https://wmlive.sourceforge.net/). Possibly a good way to try (or even
install) a Window-Maker-centric Debian setup.

[My Window Maker setup](gopher://gopher.someodd.zip:70/I/assets/screenshot-window-maker.png)


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

you should really read the manual...

### Preset themes

This is amazing: https://git.x1b.dev/waterjones/WindowMaker-Themes

[Nokia theme from waterjones](gopher://gopher.someodd.zip:70/I/assets/posts/window-maker/theme-nokia.png)

You can save your own theme using:

```
getstyle -p ~/GNUstep/Library/WindowMaker/Themes/MyTheme
```

For more info see: https://www.windowmaker.org/themes/themepacks.html

### handy hotkeys

f11: like alt tab, but a nice list

### How does this all work?

Some general tips:

try right clicking icons that pop up and goign to attributes. look through all the options. you can't just drag *anything* to the dockapp or clip. You can even set things to minimize to their icon in the dock.

Practical Workflow Tip

Use the Dock for things you want to launch.

Use the Clip to keep track of things you already have open and minimized.

If you want something to behave like a taskbar entry, keep its mini-icon in the Clip with Keep Icon.

### The clip

The clip icon allows you to manage workspaces (page through them, name them).

You can drag the clip itself to move a bunch of icons with it. Specifically, you can pin apps to the clip permanently, just like the dock--then the app will always live there for that workspace. The clip is workspace-aware. Use it as a lightweight way to keep different sets of apps organized between worskpaces. When you drag an app to the clip it will also auto-start minimized in that workspace. Beware, this isn't like, just dragging minimized things to it, but like things youc an pin to the dock you can also pin to the clip.

### GNUStep .appsm openstep de

You can install something like a DE with `sudo apt install gnustep`, which is kind of just a meta for all the `*.app` windowmaker apps that are kinda nifty, but also sometimes too severely dated for me to find them useful. I'm not sure how useful you'll find these apps day-to-day, but they're fun to look at.

Try `sudo apt install textedit.app` and run with `openapp TextEdit`.

Also try `apt search "\.app"`

These apps will feel right at home! Note when you use `openapp` you'll want to use `PascalCase`.

Some things I noticed:

* textedit.app -- rtf editor + more? seems kinda cool
* fontmanager.app
* gnumail.app seemed interesting, but I have to do extra work to get it to play nice with proton mail bridge i think
* fortunate.app -- simple fortunes. cute toy.
* gmastermind.app -- classic board game that I think inspired the fallout hacking game. dennis ritchi or the like came up with an algo for this i think?
* gorm.app: visual interface builder for gnustep!
* grr.app: RSS reader! It's honestly kind of nice! I wish it had support for custom commands to fetch RSS like liferea (?) does (so I can grab gopher URI feeds)
* gshisen.app -- some kind of mahjong tile game?
* gworkspace: i have no idea what this is but seems awesome. might explore later.
* gmpdcon.app -- perfect since I have an mpd server! Even lets you give ratings! kind of strange though, i haven't gotten used to it.
* pikopixel.app -- pixel art editor!
* preview.app -- maybe the best choice for previewing an image in WM!
* talksoup.app -- irc client for gnustep
* terminal.app -- terminal emulator!
* viewpdf.app
* Affiche.app -- sticky note app that I actually really like! has a bunch of nice features like saving importing/exporting. reminds me also of the dock app that does something similar, but affiche.app i could see using frequently.
* cynthiune.app -- A really neat music player. I think it has troubles adding my entire library at once, though.
  ```
  2025-09-21 12:20:01.408 Cynthiune[439584:439584] MP3.m: no handle...
  2025-09-21 12:20:01.408 Cynthiune[439584:439584] MP3.m: no handle...
  2025-09-21 12:20:01.408 Cynthiune[439584:439584] Failed to create pipe ... Error Domain=NSPOSIXErrorDomain Code=24 "Too many open files"
  2025-09-21 12:20:01.563 Cynthiune[439584:439584] NSTask.m:593  Assertion failed
  in NSConcreteUnixTask(instance), method setStandardOutput:.  NSInvalidArgumentException
  [1]    439584 segmentation fault  openapp Cynthiune
  ```

  

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

##### wmweather+

This weather dockapp rules. You even get a radar GIF and different weather views. The config can be a little confusing though. Here's what I have. I have it launch with `wmweather+ -c /home/tilde/.wmweather+/config` and then in that config:

```
# METAR station
-station KSFO

# Coordinates (Mission, SF)
-location 37.7599N 122.4148W

# Bay Area radar loop (KMUX)
-radar-uri https://radar.weather.gov/ridge/standard/KMUX_loop.gif

# Crop & animate with crosshair
-radar-crop 244x134+52+40
-radar-cross 244x134 animate
```

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

adding drawers is cool and you can have it autocollect which is super handy.

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

You can actually be more sophisticated than this, because annoyingly, if you restart your session or something, everything will get launched AGAIN even if it's already running!

So try something like this:

```
#!/bin/sh

# runonce CMD [args...]
# Starts CMD only if no process with the same basename is running.
runonce() {
  cmd="$1"; shift
  name=$(basename "$cmd")
  pgrep -x "$name" >/dev/null 2>&1 || "$cmd" "$@" &
}

# If you want a fresh xsettingsd each login, keep pkill; otherwise just `runonce xsettingsd`
pkill -x xsettingsd 2>/dev/null
runonce xsettingsd

runonce blueman-applet
runonce amor
runonce xscreensaver --no-splash
runonce xeyes
# runonce virt-manager

runonce protonmail-bridge

# For scripts, the process name is the shell (e.g., sh), so use a lock to avoid duplicates
flock -n /tmp/deadline.lock /home/tilde/scripts/deadline.sh &

runonce skippy-xd --start-daemon
runonce xpenguins -a -b -t "Big Penguins"
# runonce oneko
runonce redshift-gtk -m randr -l 37.8044:-122.2712
runonce nm-applet
runonce deja-dup
runonce ibus-daemon -drx

# One-shot tweak (not a daemon; don’t background)
xset m 20/10 4
```

It's worth mentioning you can just SAVE the wmaker sesssion and load it on startup (can be done automatically). But for some reason I prefer this method.

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

For GTK4 there's another project that tries to accomplish this but I just lazily used some GTK4 theme called Windows-95 and edited `~/.config/gtk-4.0/settings.ini` and `gtk-theme-name=Windows-95` then logged in and out.

Bonus: https://github.com/mgsander/wmstep/tree/master/WMStep:
something I found but I didn't get working.

I went ahead and disabled the GTK window decoration hints or the like in the
advanced section of WPrefs or something. I think this maybe makes things look
more consistent.

#### Icon theme

There's actually a GNUstep icon theme for GTK I believe: https://www.gnome-look.org/p/1239539

Something like this: `tar -xzf GNUstep.tar.gz   -C ~/.local/share/icons`

Then you can actually use `lxappearance` to change the icon theme, which this app is generally pretty useful for tweaking GTK theme stuff, more-or-less.

I went down a weird path where thunar wasn't picking up on the icon theme and I found out I need `sudo apt install xsettingsd`, then create ~/.xsettingsd:

```
Gtk/IconThemeName "GNUstep"
Gtk/ThemeName "Redmond97 CDE"
Gtk/FontName "Noto Sans 10"
Net/IconThemeName "GNUstep"
```

and make sure xsettingsd autostarts with gnustep by adding it to ~/GNUstep/Library/WindowMaker/autostart and add:

```
killall -q xsettingsd
xsettingsd &
```


or the like.

You may also wanna try the Chicago95 theme.

```
git clone https://github.com/grassmunk/Chicago95.git
cd Chicago95
./installer.py
```

You get cursors, sounds, fonts... i actually updated my ~/.xsettingsd to look like this:

```
#Gtk/IconThemeName "Chicago95"
Gtk/ThemeName "Redmond97 CDE"
#Gtk/FontName "Noto Sans 10"
#Net/IconThemeName "GNUstep"

# Icon Theme
Gtk/IconThemeName "Chicago95"

# Cursor Theme (The correct name for the large, animated version)
Gtk/CursorThemeName "Chicago95_Animated_Hourglass_Cursors_HiDPI"

# Font
Gtk/FontName "Plus! 8"

# Compatibility settings
Net/IconThemeName "Chicago95"
Net/ThemeName "Redmond97 CDE"
```

I feel like Chicago95 is fine until I find something that more suits my late 90s linux vibes.

I actually installed my own cursors, you can check this place out for more:

*  https://www.rw-designer.com/cursor-library/set-40

Although the cursor I'm using is [Golden-XCursors-3D-0.8 from Gnome-Look](https://www.gnome-look.org/p/999590) and setting this in my `.xsettingsd`:

```
Gtk/CursorThemeName "Gamma.Gold"
Gtk/CursorThemeSize 48
```

### Adding hotkeys

You can add hotkeys by editing the Window Maker menu through WPrefs, under
*Applications Menu Definition*. I like to create a special *submenu* that holds
all the *run program* entries which have hotkeys associated to them.

For example, I set lock (`xscreensaver-command -lock`) to my super key + l.

I think sometimes (?) you may need to restart the session for hotkeys to come into effect.

#### Control Screen Brightness

Use `brightnessctl set 10%-` and `brightnessctl set 10%+` then add to menu and assign hotkeys.

## Laptop: power management

I use `powerprofilesctl`, you can use commands like:

```
powerprofilesctl set power-saver
powerprofilesctl list
powerprofilesctl get
```

I'd like to have a GUI solution.

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

`firefox-esr`My main web browser is just the Debian-provided Firefox. You may want to tweak the scrollbar size (that's a thing you can do).

https://github.com/dillo-browser/dillo -- you actually may not want to use the repo version and build it from there, because of the time of writing this i'm told the repo verison is ten years old. https://github.com/dillo-browser/dillo/blob/master/doc/install.md gopher plugin: https://github.com/dillo-browser/dillo-plugin-gopher

### KeepassXC: Password manager, keyring manager!

Password manager. In my opinion this is crucial and fantastic for Window Maker, because Window Maker doesn't just include an SSH Agent and Secret Service integration. I wrote an article about this: [using KeepassXC as keyring manager](/phlog/keepass-keyring-manager.gopher.txt).

### deja-dup

I just find it's very reliable for backups and easy to use.

### audacious

Audio player.

The gtk audacious is great! the hotkeys seem to mostly work out-of-the-box for
what I've used, I think? You can use Winamp skins if you want to go the extra
mile. If you want to go even further, milkdrop is available for linux.

Don't forget, if you use the Winamp skins, you can right click the titlebars in audacious and then select attributes--disable the titlebars! A good thing to know in general. I also like making it 2x scale (but that's by right clicking the actual Winamp content/pane).

Please see my phlog article on projectM (old winamp/milkdrop) visualizations.

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

For thumbnails you may also want to install `tumbler`.

### Bluetooth and Wifi GUI

These two apps will enable a GUI through the system tray (dockapp).

I use `nm-applet` for all my needs. I add it to `autostart`.

i also use `blueman-applet` (you can just install through `blueman` package), add it to `autostart` for Window Maker.

### xfce4-terminal

I find that this is a nice terminal to use in Window Maker.

### Screenshots

Windowmaker has its own built-in screen capture you can configure undder keyboard shortcut preferences in wprefs.

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

This is the command that worked for me:

```
redshift-gtk -m vidmode -l 37.7749:-122.4194
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

#### "Fix:" selected item is black-on-black

You may have an issue where your theme, with Claws, causes the selected item to be illegible due to the text vs. background color of a selected item. I found this took me a while to figure out, so here I'm going to save you the pain.

You can edit `~/.config/gtk-3.0/gtk.css` and add these lines:

```
/* Only Claws: its main window has id #mainwindow */
window#mainwindow *:selected {
  background-color: #3584E4;
  color: #FBF6F0;
}
```

#### Start reply above the quote

I think it's a bit dated/annoying for others to have the email you're replying to quoted *above* your actual reply message. So you can go into preferences and then under "compose" is *templates." There's a "reply" tab and you can do something like this:

```
%cursor

On %d
%f wrote:

%q
```

For the above to work you need to disable *compose > writing > replyling > replyl with quote by default.*

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

Useful: http://xpenguins.seul.org/index-2.1.html -- they also have a gnome applet.

Penguins to walk and fall off windows and more.

```
xpenguins -ab t "Big Penguins"
```

I find it has a good deal of nice config options.

You can even install themes!

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

I made my own theme you can find here:

[My Amor themes](gopher://gopher.someodd.zip:70/1/assets/someodd_creations/amor_theme_peepy)

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
  * xmountains
  * xplanet
  * You can actually set screensavers as wallpapers with commands like `/usr/libexec/xscreensaver/glblur -root &`
  * You can run xscreensavers in a window, here's an "aquarium:" `/usr/libexec/xscreensaver/glschool -window -geometry 640x480+100+100 &`

Add some terminal whimsy:

* `sudo apt install sl` then try running `sl`

#### `xrootconsole`

Scrolling console text painted onto your wallpaper/desktop!

Here's an example:

```
tail -f ~/.zsh_history | xrootconsole
```

I actually have a setup that queries what's basically my "latest posts" script in gopherspace. I'll cover that here.

My setup is to add this to my crontab:

```
curl -s gopher://gopher.someodd.zip:70/0/gateway/status/feed > /tmp/someodd-feed.txt
```

Then I have this script:

```
# !/bin/sh
# feed-diff-ticker.sh — print only the added lines since last snapshot
# Usage:
#   feed-diff-ticker.sh /tmp/someodd-feed.txt /tmp/someodd-feed.prev 10 | xrootconsole &
# Env:
#   MAX_LINES=100   # optional cap on burst output

set -eu

FEED="${1:-/tmp/someodd-feed.txt}"
STATE="${2:-/tmp/someodd-feed.prev}"
INTERVAL="${3:-10}"
MAX_LINES="${MAX_LINES:-}"

WORK_DIR="$(dirname "$STATE")"
TMP_NEW="$WORK_DIR/.feed.new.$$"
TMP_DIF="$WORK_DIR/.feed.diff.$$"

cleanup() { rm -f "$TMP_NEW" "$TMP_DIF" 2>/dev/null || true; }
trap cleanup EXIT INT TERM

# Seed state file if missing
[ -f "$STATE" ] || : > "$STATE"

while :; do
  # Skip if feed missing or empty
  if [ ! -s "$FEED" ]; then
    sleep "$INTERVAL"; continue
  fi

  # Snapshot current feed (best with atomic mv in cron)
  cp -f -- "$FEED" "$TMP_NEW" 2>/dev/null || { sleep "$INTERVAL"; continue; }

  # Compute diff; exit codes: 0=same, 1=different, >1=error
  if diff -u --label old "$STATE" --label new "$TMP_NEW" >"$TMP_DIF" 2>/dev/null; then
    # No differences: do not touch STATE
    :
  else
    # We have differences (exit code 1). Extract ONLY added lines from “new”.
    # Strip headers (---/+++), hunks (@@), keep lines starting with '+', then drop the leading '+'.
    ADDED="$(sed -n '
      1,2d;                 # drop the first two header lines
      /^\+\+\+ /d;          # drop +++ header
      /^--- /d;             # drop --- header (in case)
      /^@@/d;               # drop hunk markers
      s/^\+//p              # print added lines with leading + removed
    ' "$TMP_DIF")"

    if [ -n "$ADDED" ]; then
      if [ -n "$MAX_LINES" ]; then
        printf "%s\n" "$ADDED" | tail -n "$MAX_LINES"
      else
        printf "%s\n" "$ADDED"
      fi
      # Update STATE only when we actually emitted new content
      mv -f -- "$TMP_NEW" "$STATE"
      : > "$TMP_NEW" 2>/dev/null || true
    fi
  fi

  sleep "$INTERVAL"
done
```

And my `xrootconsole` launch command is:

```
/home/tilde/bin/feed-diff-ticker.sh /tmp/someodd-feed.txt /tmp/someodd-feed.prev 10 | xrootconsole &
```

## Troubleshooting

### Apps keep autostarting when I don't want them to

I had to figure out why things kept autostarting despite not being in my `autostart` file nor was I saving session on exit nor set in `WPrefs`. Apparently checkout `~/GNUstep/Defaults`. You should make sure to edit it while windowmaker isn't running because of the `.lck` (lock) it creates in the same directory.

I saw in the aforementioned directory a file called `WMState`, which indeed had all the applications autostarting which I didn't want to do so.

https://www.windowmaker.org/docs/chap4.html

Original content in gopherspace: [gopher://gopher.someodd.zip:70/1/phlog/window-maker.gopher.txt](gopher://gopher.someodd.zip:70/1/phlog/window-maker.gopher.txt)
