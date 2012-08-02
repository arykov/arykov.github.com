---
layout: post
title: "WebLogicMBeanMaker and Maven"
description: ""
category: 
tags: [weblogic, maven, security]
---
{% include JB/setup %}
Setting up Maven build of your custom WebLogic security providers is no trivial task. This [post](http://monduke.com/2007/08/29/maven-and-weblogicmbeanmaker-in-weblogic-910/) describes some of the pain you have to go to, but does not take you quite all the way. Yet this solution still depends on WebLogic. This makes assumptions about machines where maven will run on. 
 
To solve this problem you need to pass -Dweblogic.home=${project.build.sourceDirectory}/.. parameter to WebLogicMBeanMaker subprocesses to trick it into using location of your choice. This can be done like this 

> java weblogic.management.commo.WebLogicMBeanMaker -jvmArgs "-Dweblogic.home=${project.build.sourceDirectory}/.." ... 

This fake WebLogic home location should be part of your project and fixed relative to your source.  All you need to do to set it up is to add &lt;WEBLOGIC_HOME&gt;/lib/schema/weblogic-domain-binding.jar to your ${project.build.sourceDirectory}/../lib/schema/ as well as &lt;WEBLOGIC_HOME&gt;/lib/mbeantypes/wlManagementImplSource.jar and &lt;WEBLOGIC_HOME&gt;/lib/mbeantypes/wlManagementMBean.jar to ${project.build.sourceDirectory}/../lib/mbeantypes.