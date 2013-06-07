---
layout: post
title: "Testing ldap authentication with production users"
description: ""
category:
tags: [testing, ldap]
---
{% include JB/setup %}
It is never easy to remove components that have been in production for a while. This obvious fact was further reinforced when I was switching authentication from security product that cannot be named to a simple LDAP or AD more specifically. 

AD to application group mappings was the only function performed by that product, but there was quite a few - thousands to be exact. How would one verify that all mappings get converted and noone looses access? Test  accounts is a good start but you never know how representative they are. Wouldn't it be great to verify every single user? The only problem is - everyone has their own password which makes ldap binds impossible. 

The first thing I tried was cloning AD data to my local ADAM instance. Reset password and you are in business. Right? Well it might work better for you than for me, but my ADAM proved way too slow to make this work. So I opted for a different solution instead.

Minor modifications to Sun's/ForgeRock's sample ldap proxy allowed me to accept any password on the bind and forward all the lookups to the real production AD. Worked like a charm. Proved handy for performance testing as well - bind is not measured and proxy itself eats into numbers, but it is priceless to be able to use real users. Besides it is all those group lookups that are most expensive. [Here's my code](https://github.com/arykov/ldapproxy).
