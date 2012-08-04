---
layout: post
title: "JDBC deployment plan"
description: ""
category: 
tags: [weblogic,wlst,deployment,jdbc]
---
{% include JB/setup %}
Idea of deployable jms and jdbc modules that can be bundled in an application that needs them simplifies relationships between app developers and those who end up deploying them. No chance of forgetting to create datasources, queues, etc. No need to check whether they already exist and if they are created the way developer expected them to.

Nevertheless, I don't see this feature being used much or at least not too many people care to admit to it. Otherwise, where are all those examples to save time? Mine is [here](https://github.com/arykov/wlst_samples/tree/master/jdbcdeployplan). Don't forget to read readme.txt for a couple gotchas.
