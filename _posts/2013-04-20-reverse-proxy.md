---
layout: post
title: "Web Testing with Reverse Proxy"
description: ""
category:
tags: [testing, web]
---
Recently I was working on an application that relied on SAML for its web users authentication. This made its shakedown in each environment somewhat challenging. Typical approach when you can use IP and port to verify deployment did not work since application required access by name only. There was an option to build a "shakedown back door", of course, but large chunks of configuration would have been missed during verification. I opted to write a [small reverse proxy](https://github.com/arykov/reverseproxy) instead. It is based on [little proxy](http://www.littleshoot.org/littleproxy/) and allows:
- Allows to dynamically switch the ultimate destinationi(handy when testing clusters/farms)
- Can terminate https and interact with the destination using http(useful when https does not flow all the way to the appserver)
- Works as browser proxy. This way you can assign myhost.com:2013 to point to 192.168.1.160:6400 without messing with hosts files and starting additional processes.
