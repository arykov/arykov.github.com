---
layout: post
title: "M2E System Dependencies Manifest Class-path"
description: ""
category:
tags: [Eclipse, M2E, maven]
---

Recently I had to use a system scoped dependency in my pom. Some of my code used classes packaged in the jars pulled in through system dependency's manifest Class-Path entry.

To my surprise my M2E enabled Eclipse project was showing "import cannot be resolved" errors while compilation using command line mvn went smoothly. Some digging revealed the root cause and the solution.

Turns out that starting with Java 1.5 javac started respecting classpath manifest entries in jar files - [bug 4212732](http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=4212732). Command line mvn relies on your java tooling for compilation. So it is clear why this works. 

M2E plugin relies on Eclipse compiler, which started respecting manifest classpath entries in 3.5 but [dropped this support in 3.6](https://bugs.eclipse.org/bugs/show_bug.cgi?id=313965). Hence the discrepancy in behaviour. 

Solution is simple. Just add -DresolveReferencedLibrariesForContainers=true to the Eclipse ini file as this [post suggests](http://lt-rider.blogspot.ca/2010/05/jdt-manifest-classpath-classpath.html).
