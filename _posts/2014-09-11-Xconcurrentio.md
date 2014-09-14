---
layout: post
title: "Windward high CPU low throughput"
description: ""
category:
tags: [troubleshooting, concurrency, java, windward]
---

Ran into a curious performance problem the other day. Folks running load tests were puzzled with high CPU utilization during one of the tests. GC was ruled out very quickly. Thread dumps revealed the bottleneck - windward library responsible for document generation. 

Thinking that vendor involvment is inevitable(obfuscated library left very little choice) I started working on a contained reproducer. To my surprize running reproducer on a windows development workstation showed much better results than expected. Although workstation was pretty beefy it seemed a little strange. Was the bottleneck identified incorrectly? Or was the test off? 

Reproducer on our Solaris server also performed quite well until JVM was "tuned" to have the same parameters as the container in which original application ran. The culprit was –Xconcurrentio that caused throughput to fall as concurrency and CPU utilization increased. This undocumented flag was introduced back in Java 1.3 days and is still mentioned in [places in Oracle documentation](http://www.oracle.com/technetwork/java/hotspotfaq-138619.html#threads_general) as a way to improve performance of applications with a large number of threads especially on Solaris. Well, aparently not on JDK 1.6.0_35 and not on this application.

I did not dig deeper than this. For those who are keen to get to the bottom of this problem -XX:+PrintFlagsFinal would give a good starting point if there is a specific optimization that causes this problem.

