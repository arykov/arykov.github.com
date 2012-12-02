---
layout: post
title: "M2E System Dependencies Manifest Classpath"
description: ""
category:
tags: [Eclipse, M2E, maven]
---
{% include JB/setup %}

Maven system scoped dependencies should only be used when these dependencies are sensitive to their location. I was unlucky enough to need them and happened to be using M2E plugin for Eclipse. Some of my code depended on classes packaged in jars that my system dependency referred to using its manifest classpath entry.

To my surprise Eclipse was showing errors in my code while compilation using command line mvn went smoothly. Some digging revealed the reason and the solution.

Turns out that starting with Java 1.5 javac started respecting classpath manifest entries in jar files - [bug 4212732](http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=4212732). Command line mvn relies on your java tooling for compilation. So it is clear why this works. 

M2E plugin relies on Eclipse compiler, which started respecting manifest classpath entries in 3.5 but [dropped this support in 3.6](http://lt-rider.blogspot.ca/2010/05/jdt-manifest-classpath-classpath.html). Hence the discrepancy in behaviour. 

Solution is simple. Just add -DresolveReferencedLibrariesForContainers=true to the Eclipse ini file as this [post suggests]((http://lt-rider.blogspot.ca/2010/05/jdt-manifest-classpath-classpath.html).
