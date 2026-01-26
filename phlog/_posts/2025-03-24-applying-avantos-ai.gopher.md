---
date: 2025-03-24
tags:
- career
- ai
- code
title: Applying to Avantos AI (job)

---


I saw a job listing on [Ask HN: Who is hiring (March 2025)](https://news.ycombinator.com/item?id=43243024). Seemed to be one of those fancy new cryptic challenge-based interviews.

At first I tried doing a command like this:

```
curl -X POST https://apply-to-avantos.dev-sandbox.workload.avantos-ai.net/ \   
  -H "Content-Type: application/json" \
  -d '{"email": "my email"}'
```

That gave me a forbidden response. I was used to these, so I figured it had to do with blocking spammy-seeming requests. I neglected to read this part:

> You'll need to make sure the user agent isn't something too bot-seeming or WAF will block you. (Sorry we're a startup no time to fix.) This user agent works: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/133.0.0.0 Safari/537.36

So here I ran the command in verbose mode:

```
➜  post-apply curl -v -X POST https://apply-to-avantos.dev-sandbox.workload.avantos-ai.net/ \
  -H "Content-Type: application/json" \
  -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:117.0) Gecko/20100101 Firefox/117.0" \
  -d '{"email": "my email"}'

Note: Unnecessary use of -X or --request, POST is already inferred.
* Host apply-to-avantos.dev-sandbox.workload.avantos-ai.net:443 was resolved.
* IPv6: (none)
* IPv4: 18.190.42.252, 3.22.13.83, 18.117.63.32
*   Trying 18.190.42.252:443...
* GnuTLS ciphers: NORMAL:-ARCFOUR-128:-CTYPE-ALL:+CTYPE-X509:-VERS-SSL3.0
* ALPN: curl offers h2,http/1.1
* found 152 certificates in /etc/ssl/certs/ca-certificates.crt
* found 456 certificates in /etc/ssl/certs
* SSL connection using TLS1.3 / ECDHE_RSA_AES_128_GCM_SHA256
*   server certificate verification OK
*   server certificate status verification SKIPPED
*   common name: apply-to-avantos.dev-sandbox.workload.avantos-ai.net (matched)
*   server certificate expiration date OK
*   server certificate activation date OK
*   certificate public key: RSA
*   certificate version: #3
*   subject: CN=apply-to-avantos.dev-sandbox.workload.avantos-ai.net
*   start date: Mon, 03 Mar 2025 00:00:00 GMT
*   expire date: Wed, 01 Apr 2026 23:59:59 GMT
*   issuer: C=US,O=Amazon,CN=Amazon RSA 2048 M02
* ALPN: server accepted h2
* Connected to apply-to-avantos.dev-sandbox.workload.avantos-ai.net (18.190.42.252) port 443
* using HTTP/2
* [HTTP/2] [1] OPENED stream for https://apply-to-avantos.dev-sandbox.workload.avantos-ai.net/
* [HTTP/2] [1] [:method: POST]
* [HTTP/2] [1] [:scheme: https]
* [HTTP/2] [1] [:authority: apply-to-avantos.dev-sandbox.workload.avantos-ai.net]
* [HTTP/2] [1] [:path: /]
* [HTTP/2] [1] [accept: */*]
* [HTTP/2] [1] [content-type: application/json]
* [HTTP/2] [1] [user-agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:117.0) Gecko/20100101 Firefox/117.0]
* [HTTP/2] [1] [content-length: 28]
> POST / HTTP/2
> Host: apply-to-avantos.dev-sandbox.workload.avantos-ai.net
> Accept: */*
> Content-Type: application/json
> User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:117.0) Gecko/20100101 Firefox/117.0
> Content-Length: 28
> 
* upload completely sent off: 28 bytes
< HTTP/2 200 
< date: Tue, 25 Mar 2025 00:33:13 GMT
< content-type: application/json
< content-length: 404
< 
* Connection #0 to host apply-to-avantos.dev-sandbox.workload.avantos-ai.net left intact
{"challenge_link":"https://fluttering-atmosphere-1b5.notion.site/Journey-Builder-React-Coding-Challenge-190d5fe264fa80cba39ec21afc6d42ec","instructions":"Submit your solutions as a link to a github repo named `045eb7` to the following email address: challenge-request-sub-aaaapriy22ktilybsd6gv7beiq@avantos.slack.com. Questions can be sent to challenge-help-aaaaprjeraoxiaa2wlnlda7vsi@avantos.slack.com"}%                   
```

I opened up [the challenge link](https://fluttering-atmosphere-1b5.notion.site/Journey-Builder-React-Coding-Challenge-190d5fe264fa80cba39ec21afc6d42ec). It kind of seemed like they didn't want me just copying-and-pasting the challenge into Claude or ChatGPT or something because they disabled ctrl+a (maybe not intentional), so I also assumed there was more logging than that going on. So I used `wget` to get the web page and then fed it into Claude. I also fed the API documentation into Claude.

Seemed like a lot of work for a role I didn't want at all, but overall a fun little thing. Maybe a bit much to expect, though. Just thought I'd post here because I found it rather interesting.

Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/applying-avantos-ai.gopher.txt](gopher://gopher.someodd.zip:70/0/phlog/applying-avantos-ai.gopher.txt)
