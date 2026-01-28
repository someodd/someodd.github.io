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
    bind           = 127.0.0.1
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

### Handle rewriting for Tor + my modern setup

This is the layout.

No heuristics.  
No guessing.  
Port defines intent.

### torrc

```conf
HiddenServiceDir /var/lib/tor/gopher
HiddenServicePort 70 127.0.0.1:7072
````

### xinetd (clearnet)

`/etc/xinetd.d/gopher_clear`

```conf
service gopher_clear
{
    type           = UNLISTED
    port           = 70
    bind           = 0.0.0.0

    socket_type    = stream
    protocol       = tcp
    wait           = no
    user           = root

    server         = /bin/bash
    server_args    = /usr/local/bin/gopher_router.sh

    disable        = no
}
```

### xinetd (onion)

`/etc/xinetd.d/gopher_onion`

```conf
service gopher_onion
{
    type           = UNLISTED
    port           = 7072
    bind           = 127.0.0.1

    socket_type    = stream
    protocol       = tcp
    wait           = no
    user           = root

    env            = ONION_MODE=1
    server         = /bin/bash
    server_args    = /usr/local/bin/gopher_router.sh

    disable        = no
}
```

### router

I'm not even using the port check nor the envvar here lol...

```bash
#!/bin/bash
# xinetd wrapper for Gopher services with /phorum branch
# - Preserves index-search tabs (type 7)
# - Sends CRLF to backend
# - Does NOT strip /phorum for the 7070 service (fixes /phorum/newthread)
# - Rewrites backend host/port in responses
# - Optional prefix reattach for relative selectors returned by /phorum

set -Eeuo pipefail

# --- Config -------------------------------------------------------------------
DEFAULT_HOST="gopher.someodd.zip"
DEFAULT_PORT="70"
PHORUM_BACKEND_PORT="7070"
OTHER_BACKEND_PORT="7071"

# If your /phorum backend returns relative selectors like "/newthread" and you
# want follow-up clicks to stay under /phorum, set this to "true".
PHORUM_REWRITE_PREFIX="false"  # "true" or "false"

# Onion to use when served via Tor hidden service
ONION_HOST="xj2o2wylbqkprajldswuyxm6dffca4eepegelblgvux3uuqmtb2l56id.onion"

# --- Read request line exactly (keep tabs), trim trailing CR -------------------
IFS= read -r selector || selector=""
# strip a single trailing CR if present
selector=${selector%$'\r'}

# --- Helpers ------------------------------------------------------------------
is_tor_connection() {
  # xinetd exports remote endpoint in REMOTE_HOST/REMOTE_ADDR
  case "${REMOTE_HOST:-${REMOTE_ADDR:-}}" in
    127.0.0.1|::1|::ffff:127.0.0.1) return 0 ;;
    *) return 1 ;;
  esac
}

current_target_host() {
  if is_tor_connection; then
    printf '%s' "$ONION_HOST"
  else
    printf '%s' "$DEFAULT_HOST"
  fi
}

rewrite_backend_links() {
  # $1 = backend_port, $2 = rewrite_prefix (true/false), $3 = (optional) prefix text like "/phorum"
  local backend_port="$1"
  local rewrite_prefix="$2"
  local prefix="${3:-/phorum}"
  local tgt_host; tgt_host="$(current_target_host)"

  # Replace host:port emitted by backend with public host:70
  # Also normalize any literal "host/path" occurrences emitted by some servers
  # Finally, optionally reattach prefix to *returned* selectors that start with "/"
  if [[ "$rewrite_prefix" == "true" ]]; then
    sed -e "s|\t${DEFAULT_HOST}\t${backend_port}|\t${tgt_host}\t${DEFAULT_PORT}|g" \
        -e "s|\t${DEFAULT_HOST}/|\t${tgt_host}/|g" \
        -e "s|\t/|\t${prefix}/|g"
  else
    sed -e "s|\t${DEFAULT_HOST}\t${backend_port}|\t${tgt_host}\t${DEFAULT_PORT}|g" \
        -e "s|\t${DEFAULT_HOST}/|\t${tgt_host}/|g"
  fi
}

proxy_to() {
  # $1 = backend_port
  local port="$1"
  # Forward the request exactly as received (including any \tquery), with CRLF
  # Use -w to avoid hanging if the backend closes slowly; -N if your nc supports it
  printf "%s\r\n" "$selector" | nc localhost "$port" | rewrite_backend_links "$port" "false"
}

proxy_phorum() {
  # For /phorum we forward *as-is* (no prefix stripping). This fixes /phorum/newthread.
  # Optionally reattach /phorum to returned relative selectors if desired.
  local port="$PHORUM_BACKEND_PORT"
  local tgt_host; tgt_host="$(current_target_host)"
  if [[ "$PHORUM_REWRITE_PREFIX" == "true" ]]; then
    printf "%s\r\n" "$selector" | nc localhost "$port" | rewrite_backend_links "$port" "true" "/phorum"
  else
    printf "%s\r\n" "$selector" | nc localhost "$port" | rewrite_backend_links "$port" "false"
  fi
}

# --- Routing ------------------------------------------------------------------
case "$selector" in
  /phorum*)
    proxy_phorum
    ;;
  *)
    proxy_to "$OTHER_BACKEND_PORT"
    ;;
esac
```

Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/gopher-routing.gopher.txt](gopher://gopher.someodd.zip:70/0/phlog/gopher-routing.gopher.txt)
