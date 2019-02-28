---
layout: post
title: "Maven release almost nirvana"
description: ""
category:
tags: [mvn, maven, jenkins, java, git]
---

Axel Fontaine had a number of posts on improving release for maven projects. His latest [installment](https://axelfontaine.com/blog/dead-burried.html)
suggests releasign using: 
```shell
mvn deploy scm:tag -Drevision=$BUILD_NUMBER
``` 
It is done without using release plugin which became possible with mvn 3.2.1+ that introduced continuous delivery friendly versions.

This leaves/creates two problems:

- POM published this way is not usable by downstream artifacts. You could conceivably live with it with artifacts that will have no one ever referencing them, but is a deal breaker for libraries and is totally contrary to what systems like maven are all about. To solve this issue use mvn flatten plugin. To see the difference try to compare poms installed using mvn install command on [this](https://github.com/arykov/mvn-release-nirvana/releases/tag/notflat) and [this](https://github.com/arykov/mvn-release-nirvana/releases/tag/flat).

- When you are running release job, you need to specify SCM location in your job for it to know where to get the code from as well as in your pom. Having this information in two places seems wrong and since having it on your job server is unavoidable I suggest the following solution when using git.
```shell
git tag -a "$BUILD_NUMBER" -m "Jenkins released $BUILD_NUMBER"
mvn deploy -DskipTests -Drevision=$BUILD_NUMBER
git push origin "$BUILD_NUMBER"
```
We eliminated scm information from the source pom. It would be nice to embed it in the pom at publish time, but an elegant solution for this eludes me. 

Sample multi-module project with flatten plugin is available [here](https://github.com/arykov/mvn-release-nirvana)


