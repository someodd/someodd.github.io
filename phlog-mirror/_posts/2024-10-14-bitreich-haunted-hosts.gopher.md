---
date: 2024-10-14
tags:
- sysadmin
- linux
- halloween
- events
title: My entry for Bitreich's "Haunted Hosts" Halloween Event

---


For Bitreich's "Haunted Hosts" Hallowe'en event I made this, try a `trick`:

```
ssh -p 6666 trick@someodd.zip
```

.. or how about a `treat`?

```
ssh -p 6666 treat@someodd.zip
```

roygbyte of bitreich mentioned and summarized all the entries (including mine!):

[Read the article on roybyte's gopherhole](gopher://roygbyte.com:70/1/haunted_hosts_hallowe_en_2024.gph)

## Background

[Bitreich "Haunted Hosts" Hallowe'en event announced!](gopher://bitreich.org:70/0/usr/roygbyte/phlog/2024-10-12T14-01-34-582764.md)
	
I would like to thank Bitreich member ROYGBYTE for nudging me toward a simpler approach with this writeup:

[ROYGBYTE's guide for authless SSH toy accounts](gopher://roygbyte.com:70/1/haunted_hosts_tutorial_for_running_spooky_server_daemons.gph)

This guide was written from a Debian perspective, but should work for all Linux users, pretty much.

## What I did, how you can too

### Setup `trick` and `treat users


Create the users:

```
sudo adduser --home /home/trick --shell /bin/sh --disabled-password trick
sudo passwd -d trick


sudo adduser --home /home/treat --shell /bin/sh --disabled-password treat
sudo passwd -d treat
```

### Create the spooky `trick` script `/home/trick/spooky_animation.sh`

Don't forget to mark the script as executable.

`/home/trick/spooky_animation.sh`:

```bash
#!/bin/bash

# First frame
frame1=$(cat << 'EOF'
                     
                     
   (       "     )   
    ( _  *           Double, double
       * (     /      \    ___
          "     "        _/ /
         (   *  )    ___/   |
           )   "     _ o)'-./__
          *  _ )    (_, . $$$
          (  )   __ __ >_ $$$$
           ( :  { _)  '---  $\
      ______'___//__\   ____, \
       )           ( \_/ _____\_
     .'             \   \------''.
     |='           '=|  |         )
     |               |  |  .    _/
      \    (. ) ,   /  /__I_____\
  snd  '._/_)_(\__.'   (__,(__,_]
      @---()_.'---@
EOF
)

# Second frame
frame2=$(cat << 'EOF'


  (       "     )    Double, double
   ( _  *            Toil and trouble
     * (     /        \    ___
         "     "         _/ /
         (   *  )    ___/   |
          )   "      _ o)'-./__
          *  _ )    (_, . $$$
          (  )  __ __ 7_ $$$$
          ( :  { _)  '---  $\
      _____'___//__\   ____, \
       )           ( \_/ _____\_
     .'             \   \------''.
     |='           '=|  |         )
     |               |  |  .    _/
      \    (. ) ,   /  /__I_____\
  snd  '._/_)_(\__.'   (__,(__,_]
      @---()_.'---@
EOF
)

# Third frame
frame3=$(cat << 'EOF'
                     
                     Double, double
   (       "     )   Toil and trouble
    ( _  *           Fire burn and
       * (     /      \    ___
          "     "        _/ /
         (   *  )    ___/   |
           )   "     _ o)'-./__
          *  _ )    (_, . $$$
          (  )   __ __ >_ $$$$
           ( :  { _)  '---  $\
      ______'___//__\   ____, \
       )           ( \_/ _____\_
     .'             \   \------''.
     |='           '=|  |         )
     |               |  |  .    _/
      \    (. ) ,   /  /__I_____\
  snd  '._/_)_(\__.'   (__,(__,_]
      @---()_.'---@
EOF
)

# Fourth frame
frame4=$(cat << 'EOF'
                     Double, double
                     Toil and trouble
  (       "     )    Fire burn and
   ( _  *            Cauldron bubble
     * (     /        \    ___
         "     "         _/ /
         (   *  )    ___/   |
          )   "      _ o)'-./__
          *  _ )    (_, . $$$
          (  )  __ __ 7_ $$$$
          ( :  { _)  '---  $\
      _____'___//__\   ____, \
       )           ( \_/ _____\_
     .'             \   \------''.
     |='           '=|  |         )
     |               |  |  .    _/
      \    (. ) ,   /  /__I_____\
  snd  '._/_)_(\__.'   (__,(__,_]
      @---()_.'---@
EOF
)

# FIXME: could define witch frames as an array?
# Function to display the animation
witch_animation() {
    count=1
    while [ $count -le 3 ]; do
        # Show frames with a pause between each
        clear
        echo "$frame1"
        sleep 0.5
        clear
        echo "$frame2"
        sleep 0.5
        clear
        echo "$frame3"
        sleep 0.5
        clear
        echo "$frame4"
        sleep 0.5
        ((count++))  # Increment the counter
    done
}

# Define an array of fake system files and directories to "delete"
files=(
    "/bin/bash"
    "/etc/passwd"
    "/usr/local/bin"
    "/home/trick"
    "/var/log/syslog"
    "/boot/vmlinuz"
    "/lib/modules"
    "/tmp/systemd-private"
    "/sbin/init"
    "/root/.bashrc"
    "/dev/null"
    "/proc/cpuinfo"
    "/usr/lib/systemd/system"
    "/var/cache/apt"
    "/usr/share/icons"
    "/boot/initrd.img"
    "/var/spool/cron"
    "/srv"
    "/opt"
    "/home/treat/Documents"
    "/media/usb"
    "/mnt/data"
    "/sys/kernel/debug"
)

# Function to display the fake deletion
fake_deletion_animation() {
    for file in "${files[@]}"; do
        echo "rm -rf $file"
        sleep 0.1  # Delay between each fake deletion
    done

    # Final spooky message
}

# Function to display jumbled/corrupted data stream
corrupted_data_stream() {
    for i in {1..30}; do
        # Output a random string of characters to simulate corruption
        echo "$(head /dev/urandom | tr -dc 'a-zA-Z0-9!@#$%^&*()_+-=[]{}|;:,.<>?~' | head -c 80)"
        sleep 0.1  # Fast stream of corrupted data
    done
}

# Function to simulate a broken input prompt
broken_prompt() {
    while true; do
        # Display a fake prompt symbol
        echo -n "$ "
        
        # Read user input (but don't execute it)
        read user_input
        
        # Simulate "command not found" for any input
        echo "bash: $user_input: command not found"
    done
}

# Show animation
witch_animation

# Call the animation function
fake_deletion_animation

corrupted_data_stream

clear
echo "ENJOY YOUR TRICK."
echo "HAPPY HALLOWEEN 2024!"

echo "Connection to someodd.zip closed."

broken_prompt
```

### Create the spooky `treat` script `/home/treat/ascii_video.sh`

Please ensure `mpv` is installed for this script to work.

Don't forget to mark as executable (`chmod +x /path/to/script.sh`).

```
#!/bin/bash
clear

# Path to the video file you want to play (change this to your own video file)
VIDEO_PATH="/home/treat/felix_the_cat_switches_witches.mp4"

# Check if mpv is installed and then play the video using ASCII output with no sound
if command -v mpv &> /dev/null; then
    echo "Welcome! Enjoy this ASCII video!"
    echo "Press Q to quit the video."

    # Play the video in ASCII mode with no audio output
    mpv --vo=tct --no-audio "$VIDEO_PATH"
else
    echo "mpv is not installed, please install it first."
    exit 1
fi
```

### Setup `sshd`

A lot of what I did was struggle because of PAM and not noticing that I was
using `AllowUsers` (whitelisting which users are allowed).

Add these lines to `/etc/ssh/sshd_config`:

```
# This port for halloween
Port 6666

# FOR HALLOWEEN
# First, deny all users access to port 6666 except "trick" and "treat"
Match LocalPort 6666 User *,!trick,!treat
    PasswordAuthentication no
    PubkeyAuthentication no
    ForceCommand /bin/false
# Now setup "trick"
Match User trick LocalPort 6666
    PasswordAuthentication yes
    PermitEmptyPasswords yes
    PermitTunnel no
    PermitListen none
    PermitOpen none 
    PubkeyAuthentication no
    PermitRootLogin no
    UnusedConnectionTimeout 30
    X11Forwarding no
    ForceCommand /home/trick/spooky_animation.sh
    GatewayPorts no
# Now setup "treat"
Match User treat LocalPort 6666
    PasswordAuthentication yes
    PermitEmptyPasswords yes
    PermitTunnel no
    PermitListen none
    PermitOpen none
    PubkeyAuthentication no
    PermitRootLogin no
    UnusedConnectionTimeout 30
    X11Forwarding no
    # ForceCommand could be set to something specific for 'treat', like a different script or a fun command
    ForceCommand /home/treat/ascii_video.sh
    GatewayPorts no
# Deny 'trick' on the default port 22
Match User trick LocalPort 22
    PasswordAuthentication no
    PubkeyAuthentication no
    ForceCommand /bin/false 
# Deny 'treat' on the default port 22
Match User treat LocalPort 22
    PasswordAuthentication no
    PubkeyAuthentication no
    ForceCommand /bin/false
```

If you're using PAM (`UsePAM yes`), add this to the top of `/etc/pam.d/sshd`:

```
# Halloween
auth [success=1 default=ignore] pam_exec.so seteuid /usr/bin/allow_empty_password.sh
auth [success=1 user!=trick default=ignore] pam_unix.so nullok
```

and also for PAM users create `sudo vi /usr/bin/allow_empty_password.sh` (don't forget to `sudo chmod +x /usr/bin/allow_empty_password.sh`):

```
#!/bin/bash
if [[ "$PAM_USER" == "trick" || "$PAM_USER" == "treat" ]]; then
    exit 0  # Allow passwordless login
else
    exit 1  # Deny empty password
fi
```

Restart sshd with `sudo service sshd restart`.

Add port 6666 to UFW (you may also want to port forward on your router):

```
sudo ufw allow 6666 comment "trick or treat"
```

## Test it out

While testing the new setup you may want to disable fail2ban, so you don't get locked out of your box, in case something goes wrong with authentication (`sudo service fail2ban stop`). Don't forget to re-enable after testing.

You should be able to run this command successfully now (on a client):

```
ssh -p 6666 trick@simulacra 
```

## Copy of the event text

```
# 2024-10-12 14:01:34.582764 UTC (+0000)

Bitreich "Haunted Hosts" Hallowe'en event announced!

                         .=-.
                        / .`
              |\_/|    |  |       ,=+=,
              |-,-|     \ ',     ; ^v^ ;
             _|(=)|      `..+    ;'|+|''       /\_/\
            |    /  |            /;_Y_;\      /     \
            |   /|  |            |\_:_/ \    /  O O  \
            |  / \  |            |/ ' \ /    |  \./  |
            | / _ \ |            /_____\`    |       |
            |/| | |\|              |||       |       |
              | | |     __/__      |||       ;~,~.~,~;
              | | |    //  |`\    _|||_        | | |
         ...._|_|_|_...\`___,/....II'II...... /__|__\rgb...

                     Announcing the first annual:
                 Bitreich "HAUNTED HOSTS" Hallowe'en
                    October 31, 2024, 9:00PM CEST               

This Hallowe'en, hosts from around the world open their ports to
festive trick or treaters. Be spooked, scared, or delighted by hosts
haunting their `ssh` connections with a ghoulish `Banner`, cob-webbed
`ChrootDirectory`, or evil `ForceCommand`!

To participate as a host: Announce your intent to participate by
contacting ROYGBYTE on #bitreich-en:irc.bitreich.org before the event
date. Then, prepare your hauntings: make or modify your =sshd= to
include passwordless authentication for =trick= and/or =treat= users;
and, configure your choice of =sshd= options to create a
correspondingly delightful... or frightful... visitor experience!

To participate as a trick or treater: on October 31, 2024, 9:00PM
CEST, connect via `ssh` as `trick` or `treat` user to participating
hosts. Hosts may be using non-standard `sshd` ports, so for full
connection details check the event page!

Event page: gopher://bitreich.org/1/haunted-hosts
```

Source: gopher://bitreich.org/0/usr/roygbyte/phlog/2024-10-12T14-01-34-582764.md

Original content in gopherspace: gopher://gopher.someodd.zip:7071/phlog/
