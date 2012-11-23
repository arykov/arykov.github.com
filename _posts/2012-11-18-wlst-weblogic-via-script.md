---
layout: post
title: "startWebLogic script invocation from wlst"
description: ""
category: 
tags: [weblogic, wlst]
---
{% include JB/setup %}
WLST has been goto tool for WebLogic domain creation and management for a better part of the last decade. Starting WebLogic from within WLST using startServer command has its limitations. It would be logical to call startWebLogic script directly. Module developed for this purpose is available [here](https://github.com/arykov/wlstscripts/blob/master/wlfunc.py). 
