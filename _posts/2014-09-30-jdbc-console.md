---
layout: post
title: "Web JDBC Console"
description: ""
category:
tags: [troubleshooting, jdbc, jndi, debug, webapp]
---
If you ever worked in a large organization this pain will sound familiar. Application promoted from one environment to the next misbehaves due to some SQL data. Now you need to go through hoops to run queries to figure out what went wrong. It will start with a poor JEE container admin who can tell you what database(s) you need since all you know is your datasource JNDI. Then comes a DBA. Hopefully just one. Sometimes you need to go back and forth a few times. 

What can one do short of making this process more efficient? How about a small web JDBC console that lets you run some SQL queries right in the container where datasources are already defined? [H2 console](http://www.h2database.com) can do this against any database not just H2. With a small addition to traverse JNDI for datasources [Web JDBC Console](https://github.com/arykov/jdbc-console) was born. 

Disclaimer: this tool is not for restricted environments despite of having access control.