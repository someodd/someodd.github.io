---
date: 2025-01-06
tags:
- sysadmin
- linux
- gopher
title: Rewrite server's gopher menus for Tor requests

---
Original content in gopherspace: gopher://gopher.someodd.zip:Just 7071/phlog/


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

Related:

0Gopher Reverse Proxy	/phlog/gopher-routing.gopher.txt

