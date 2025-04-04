---
date: 2025-02-25
tags:
- sysadmin
- gopher
- linux
title: Routing gopher requests (reverse proxy) + Tor

---


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

### Serve default menu

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

### Handle rewriting for Tor


in `/etc/xinetd.d/gopher`:

```
service gopher
{
    type           = UNLISTED
    port           = 70
    socket_type    = stream
    wait           = no
    user           = root
    server         = /bin/bash
    server_args    = /usr/local/bin/gopher_router.sh
    log_on_success += USERID
    log_on_failure += USERID
    log_type       = FILE /var/log/xinetd_gopher.log
    disable        = no
}
```

now i updated my routing script:

```
#!/bin/bash

# Read the selector from stdin
read selector

# Function to check if the connection is coming from Tor
is_tor_connection() {
    # Check if the remote IP address matches known Tor patterns
    if [[ "$REMOTE_HOST" == "::ffff:127.0.0.1" ]]; then
        # Assuming Tor traffic comes from localhost for hidden services
        return 0
    fi
    return 1
}

# Function to process and replace paths and ports in the response
process_response() {
    local prefix="$1"
    local port="$2"
    local target_host="gopher.someodd.zip"
    local target_port="70"

    # Check if Tor was used and adjust the target host
    if is_tor_connection; then
        target_host="xj2o2wylbqkprajldswuyxm6dffca4eepegelblgvux3uuqmtb2l56id.onion"
    fi

    # Strip the prefix from the selector and ensure the leading slash is retained
    local stripped_selector="${selector#$prefix}"
    if [[ "$stripped_selector" != /* ]]; then
        stripped_selector="/$stripped_selector"
    fi

    # Send the selector to the appropriate service and process the response
    echo "$stripped_selector" | nc localhost $port | \
    sed -e "s|\tgopher.someodd.zip\t${port}|\t${target_host}\t${target_port}|g" \
        -e "s|\tgopher.someodd.zip/|\t${target_host}/|g"
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

## yet another update

sorry i need to clean up this article, but i realized for phorum until i release a prefix-adding feature for slectors in phorum i need to rewrite the links in phorum.

```
#!/bin/bash
# Read the selector from stdin
read selector

# Function to check if the connection is coming from Tor
is_tor_connection() {
    # Check if the remote IP address matches known Tor patterns
    if [[ "$REMOTE_HOST" == "::ffff:127.0.0.1" ]]; then
        # Assuming Tor traffic comes from localhost for hidden services
        return 0
    fi
    return 1
}

# Function to process and replace paths and ports in the response
process_response() {
    local prefix="$1"
    local port="$2"
    local rewrite_selectors_to_use_prefix="$3"
    local target_host="gopher.someodd.zip"
    local target_port="70"

    # Check if Tor was used and adjust the target host
    if is_tor_connection; then
        target_host="xj2o2wylbqkprajldswuyxm6dffca4eepegelblgvux3uuqmtb2l56id.onion"
    fi

    # Strip the prefix from the selector and ensure the leading slash is retained
    local stripped_selector="${selector#$prefix}"
    if [[ "$stripped_selector" != /* ]]; then
        stripped_selector="/$stripped_selector"
    fi

    # Send the selector to the appropriate service and process the response
    if [[ "$rewrite_selectors_to_use_prefix" == "true" ]]; then
        # When rewriting is enabled, add the prefix to all selectors in the response
        echo "$stripped_selector" | nc localhost $port | \
        sed -e "s|\tgopher.someodd.zip\t${port}|\t${target_host}\t${target_port}|g" \
            -e "s|\tgopher.someodd.zip/|\t${target_host}/|g" \
            -e "s|\t/|\t${prefix}/|g"
    else
        # Original behavior without selector rewriting
        echo "$stripped_selector" | nc localhost $port | \
        sed -e "s|\tgopher.someodd.zip\t${port}|\t${target_host}\t${target_port}|g" \
            -e "s|\tgopher.someodd.zip/|\t${target_host}/|g"
    fi
}

# Handle the selector
case "$selector" in
    "/phorum"*)
        process_response "/phorum" 7070 "true"
        ;;
    *)
        process_response "/" 7071 "false"
        ;;
esac
```

Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/gopher-routing.gopher.txt](gopher://gopher.someodd.zip:70/0/phlog/gopher-routing.gopher.txt)
