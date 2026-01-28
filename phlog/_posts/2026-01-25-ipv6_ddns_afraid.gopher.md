---
date: 2026-01-25
tags:
- tls
- letsencrypt
- nginx
- debugging
title: The Missing Link in My TLS Chain

---


I lost a few hours today to what looked like a cursed, haunted networking bug.

Some people could reach https://gopher.someodd.zip just fine.
Others got:

  "unable to get local issuer certificate"
  "unable to verify the first certificate"
  or Firefox HSTS hard failures.

At first it smelled like DNS.
Then IPv6.
Then routers.
Then ghosts.

But the truth was smaller and more embarrassing.

I was serving the wrong certificate file.

Let’s Encrypt gives you four files:

  cert.pem       - your site's leaf certificate
  chain.pem      - the intermediate CA (Lets Encrypt E7, etc)
  fullchain.pem  - cert.pem + chain.pem together
  privkey.pem    - your private key

I had nginx configured to serve:

  cert.pem

instead of:

  fullchain.pem

Browsers usually fetch missing intermediates from the web, so most people
never noticed. But strict TLS clients (curl, openssl, some bots, some
browsers in certain modes) do not do that.

They saw:

  "issued by Lets Encrypt E7"
  but were not given the E7 certificate
  so they could not build a trust chain.

Hence:

  verify error:num=20
  verify error:num=21

The bug only became visible because earlier, a separate IPv6 misrouting
sent some users to my router instead of my server, which poisoned HSTS
caches and made everyone start testing harder. That chaos exposed the
real problem.

The fix was simple:

In nginx:

  ssl_certificate     /etc/letsencrypt/live/gopher.someodd.zip/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/gopher.someodd.zip/privkey.pem;

Reload nginx.

Suddenly:

  openssl s_client -connect gopher.someodd.zip:443 -servername gopher.someodd.zip

goes from:

  Verify return code: 21 (unable to verify the first certificate)

to:

  Verify return code: 0 (ok)

This is one of those bugs that teaches humility.

Everything "worked" in the browser.
Everything was "green" in the padlock.
But the cryptographic truth was broken.

The missing link in the chain was literally missing.

And I only found it because someone ran a real tool.


Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/ipv6_ddns_afraid.gopher.txt](gopher://gopher.someodd.zip:70/0/phlog/ipv6_ddns_afraid.gopher.txt)
