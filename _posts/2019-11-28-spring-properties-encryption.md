---
layout: post
title: "Spring Properties Encrypton"
description: ""
category:
tags: [secrets, security, spring, encryption, spring-boot]
---

Any application that is not a thing in itself is bound to deal with something sensitive protected by keys, passwords or other secrets. While it is true that the best way to store secrets in your system is not to store them at all, more often than not you have to deal at least with a "bootstrap" secret of sorts and you need to encrypt it. 

For years I have been adding code to my Spring and Spring boot applications to deal with this. But it always seemed to be slightly contrived and creating unnecessary clutter in the code. On top of that integration with Spring was never quite right. Also I always had a soft spot for automatic encryption of open passwords placed in config files. It is just so convenient(albeit less secure) to place your secret in a file as is, start your application and have it encrypt it on the startup. 

It would be neat to know all configuration sources and properties and transparently encrypt and decrypt them without limiting Spring's ability to load from all kinds of sources. I ended up implementing a [library](https://github.com/arykov/spring-properties-encryption) that does just that. It is built around an aspect intercepting constructor and getProperty invocations on all implementations of org.springframework.core.env.PropertySource subclasses. It supports automatic encryption and decryption of values in property and yaml files(decryption is not limited to just files). You can use it via load time weaving or preprocess your application libraries with the help of ajc.

Note that it does not solve the zero secret "key under the mat problem". Your secret is still symmetrically encrypted and the decyphering key is somewhere nearby. 

One other thing worth noting is that in order to modify property files while preserving comments and key order I ended up using dependency [openprops](https://github.com/zanata/openprops) distributed under GPLv2 + Classpath Exception. I am not a lawyer but it seems that this should not affect how this library is distributed. We are not modifying openprops here and shading is more akin to static linking that classpath exception entitles us to without subjecting to forcing code distribution under GPLv2. If this logic is not convincing, openprops can be replaced by good old java properties or commons config that also plays nice with properties order and comments.
