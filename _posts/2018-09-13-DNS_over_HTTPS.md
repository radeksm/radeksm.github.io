---
layout: post
title: DNS over HTTPS (DoH)
tags:
    - DNS over HTTPS
    - Firefox
    - DNS
    - DNS Trusted Recursive Resolver
author: Radosław Śmigielski
---

Why?
====
Don't let your ISP to track your DNS queries.

Depends on the country you live in, your local internet provider(s) may track
and record history of your DNS queries. In some countries they can keep your
history up to 5 years and in some countries like in US they have a right
to sell these data to third party companies, thanks to President Donald Trump.

How?
====
You could switch to one of the public DNS instead of using your local ISP DNS.
There is few choices of public DNS servers:
* Google
  * 8.8.8.8
  * 8.8.4.4
* Quad9
  * 9.9.9.9
* Cloudflare - offers DNS over HTTPS
  * 1.1.1.1
  * 1.0.0.1
* OpenDNS owned by Cisco
  * 208.67.222.222
  * 208.67.220.220
Using one of above helps but your DNS queries still fly over the network
unencrypted.

But there is a better way, __*DNS Trusted Recursive Resolver*__ known as
also as __*DNS over HTTPS (DoH)*__. DNS over HTTPS (DoH) support has been
added to Firefox 62 but it's disabled by default. And at the time of writing
this post. Firefox is the only browser which supports DoH.

Very good explaination of the problem and how DoH works:
[A cartoon intro to DNS over HTTPS](https://hacks.mozilla.org/2018/05/a-cartoon-intro-to-dns-over-https/)

Open Firefox settings
---------------------
Open below URL in address bar of Firefox
```
about:config
```
Search for __*network.trr*__.

All settings
------------
Taken from Firefox source code __*modules/libpref/init/all.js*__
* __network.trr.mode__ DNS Trusted Recursive Resolver
  * 0 - default off, use a native resolver only
  * 1 - race, native against TRR, whichever faster returns response
  * 2 - TRR first and native as fallback
  * 3 - TRR only, do not use native at all.
  * 4 - shadow, use native and TRR in parallel for timing and measurements but uses only the native resolver results
  * 5 - off by choice and not disabled by default
* __network.trr.uri__ DNS-over-HTTP service to use, must be HTTPS://
  * https://mozilla.cloudflare-dns.com/dns-query
  * https://dns.google.com/experimental
* __*network.trr.credentials*__ credentials to pass to DOH end-point
* __*network.trr.wait-for-portal*__ Wait for captive portal confirmation before
  enabling TRR. When you need to type some password before ISP, open WiFi let
  you access Internet.
* __*network.trr.allow-rfc1918*__ set to true to allows RFC 1918 private
  addresses in TRR responses.
* __*network.trr.useGET*__ Use GET (rather than POST), default POST.
* __*network.trr.confirmationNS*__ Before TRR is widely used the NS record
  for this host is fetched from the DOH end point to ensure proper
  configuration.
* __*network.trr.bootstrapAddress*__ hardcode the resolution of the hostname
  in network.trr.uri instead of relying on the system resolver
  to do it for you.
* __*network.trr.blacklist-duration*__ TRR blacklist entry expire time
  (in seconds). Default is one minute. Meant to survive basically a page load.
* __*network.trr.request-timeout*__ Single TRR request timeout,
  in milliseconds. Default: 3000 [ms].
* __*network.trr.early-AAAA*__ Allow AAAA entries to be used "early",
  before the A results are in.
* __*network.trr.disable-ECS*__ Explicitly disable ECS
  (EDNS Client Subnet, RFC 7871).

