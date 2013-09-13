---
layout: post
title: "Load WebLogic Libraries into Maven"
description: ""
category: 
tags: [weblogic, maven]
---
Starting with version 9 of WebLogic server BEA and later Oracle have been pursuing componentization of its own products. This dramatically increased the number of jars in its installation. Most well written applications have no dependencies on these jars. Yet there are some cases when this is unavoidable. 

Some jars get placed on the classpath automatically when you add weblogic.jar. However it only works when you are using regular WebLogic installation. This is not the case if you are using maven. To simplify loading all the jars into your maven repository you can use [this script](https://github.com/arykov/weblogic-security-provider-mvn/blob/master/beatomvn.sh). Dependencies still need to be specified manually.
