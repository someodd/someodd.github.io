---
date: 2024-12-04
tags:
- sysadmin
- linux
- games
- cs16
- server
title: Counter Strike 1.6 Daemonize (Linux)

---
Original content in gopherspace: gopher://gopher.someodd.zip:Just 7071/phlog/


Let's basically daemonize my Counter Strike 1.6 server, using Systemd. It's a
little tricky because I wanted to use `screen` (to look at the interactive
server thingy) and we need to tell `systemd` to expect the process to fork
(detach itself into the background), to treat the child process (not the
parent) as the main process. This is the behavior of many legacy or standalone
server programs, like `hlds_run`, which often daemonizes itself.  We use a PID
file to keep track of the PID of the child process.  This way `systemd` will
know which process to monitor. Otherwise, `systemd` will just see that the
startup script we made exits and assume the service isn't running anymore.

Here's my normal start command:

```
./hlds_run -game cstrike +map de_dust2 +maxplayers 20 +port 27015
```

I actually like being able to use the terminal in there...

So I like using `screen`.

Write a startup script, which I actually just put as my `~/steamcmd/cs16/start_cs16.sh`:

```
#!/bin/bash

# Change to the server directory
cd /home/baudrillard/steamcmd/cs16 || exit

# Start the server inside a named screen session
screen -dmS cs_server ./hlds_run -game cstrike +map de_dust2 +maxplayers 16 -port 27015

# Get the PID of the screen session and write it to the PID file
screen_pid=$(screen -ls | grep cs_server | awk '{print $1}' | cut -d. -f1)
echo $screen_pid > /home/baudrillard/steamcmd/cs16/hlds.pid
```

Mark it as executable:

```
chmod +x ~/steamcmd/cs16/start_cs16.sh
```

Now let's make it a Systemd service by creating this service file `/etc/systemd/system/cs16.service` (be sure to change the paths, user, etc.):

```
[Unit]
Description=Counter-Strike 1.6 Server
After=network.target

[Service]
type=forking
PIDFile=/home/baudrillard/steamcmd/cs16/hlds.pid
User=baudrillard
Group=baudrillard
WorkingDirectory=/home/baudrillard/steamcmd/cs16
ExecStart=/bin/bash /home/baudrillard/steamcmd/cs16/start_cs16.sh
ExecStop=/usr/bin/screen -S cs_server -X quit
Restart=always

[Install]
WantedBy=multi-user.target
```

Now try to enable and start the service:

```
sudo systemctl enable cs16.service
sudo systemctl start cs16.service
```

Now let's check screen:

```
screen -ls
```

Tap into it:

```
screen -r cs_server
```

You can detach by doing `ctrl+a` followed by `d`.

Debugging service help:

```
sudo journalctl -u cs16.service -e
```

