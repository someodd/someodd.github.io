---
date: 2024-09-02
tags:
- sysadmin
- gopher
- linux
title: Routing gopher requests (reverse proxy)

---
Original content in gopherspace: gopher://gopher.someodd.zip:Just 7071/phlog/


I host different Gopher Protocol services which use different ports on one box.
So I kind of need a *reverse proxy* of sorts. I wanted to make sure I could
access all of them using port 70.

The idea:

  * Don't touch my existing Gopher Protocol service
  * Have a new Gopher service running on port 70
  * On the new Gopher service, associate certain internal ports with certain
    selectors

   * for example if a selector beginning with `/phorum` is requested on port
     70, I want to strip the `/phorum` bit, then request localhost 7070 and
     serve that

  * For all internal links, prepend the selector and change the port to 70

In other words, I have various Gopher Protocol services running on different
ports, but with xinetd I serve them all on port 70, just under different
selectors. This is basically a Gopher Protocol routing/reverse proxy!

## It's running live!

I use this configuration on this server.

`gopher://gopher.someodd.zip:70/1/someodd/`: basically puts my localhost 7071
gopher service to be served on port 70, under the `/someodd` selector.

`gopher://gopher.someodd.zip:70/1/phorum/`: serves phorum (running on localhost
7070) on port 70, under the `/someodd` selector.

This configuration also offers a service index at `gopher://gopher.someodd.zip/`.

## Configure the server

Install `xnetd`:

```
sudo apt-get install -y xinetd
```

Add this file `/etc/xinetd.d/gopher`:

```
service gopher
{
    type           = UNLISTED
    port           = 70
    socket_type    = stream
    wait           = no
    user           = root
    server         = /usr/local/bin/gopher_router.sh
    log_on_success += USERID
    log_on_failure += USERID
    log_type       = FILE /var/log/xinetd_gopher.log
    disable        = no
}
```

Create this script `/usr/local/bin/gopher_router.sh`:

```
#!/bin/bash

# Read the selector from stdin
read selector

# Function to process and replace paths and ports in the response
process_response() {
    local prefix="$1"
    local port="$2"
    local target_host="gopher.someodd.zip"
    local target_port="70"

    # Strip the prefix from the selector and ensure the leading slash is retained
    local stripped_selector="${selector#$prefix}"
    if [[ "$stripped_selector" != /* ]]; then
        stripped_selector="/$stripped_selector"
    fi

    # Send the selector to the appropriate service and process the response
    echo "$stripped_selector" | nc localhost $port | \
    sed -e "/\t${target_host}\t${port}/s|\t/|\t${prefix}/|g" \
        -e "s|\(${target_host}\)\t${port}|\1\t${target_port}|g"
}

# Handle the selector
case "$selector" in
    "/phorum"*)
        process_response "/phorum" 7070
        ;;
    *)
        process_response "/" 7071
        ;;
esac
```

Restart:

```
sudo systemctl restart xinetd
```

For testing:

```
echo "/parent2/little-notes" | nc localhost 70
```

Also `ufw` (firewall entry):

```
sudo ufw allow 70/tcp comment 'gopher (router)' 
```

## Bonus

You could even serve a default menu:

```
# Handle the selector
case "$selector" in
    "/phorum"*)
        process_response "/phorum" 7070
        ;;
    "/someodd"*)
        process_response "/someodd" 7071
        ;;
    *)
        # Return a Gopher menu with links to /phorum and /someodd
        echo "1Phorum link      /phorum gopher.someodd.zip      70"
        echo "1someodd link     /someodd        gopher.someodd.zip      70"
        echo ""
        ;;
esac
```

