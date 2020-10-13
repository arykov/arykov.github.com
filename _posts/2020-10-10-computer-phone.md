---
layout: post
title: "Phone as a speaker and mic for your PC"
description: ""
category:
tags: [raspberry pi, pi-kvm, pstn, voip]
---

[Pi-KVM](https://pikvm.org/) is a really handy software when you need to control your PC all the way through boot sequence or using remote desktop is not possible. Unfortunately, it does not support remote audio/microphone yet.

Until this is implemented, you can use your phone(it does not need to be smart) in conjunction with a VOIP provider that can accept calls on a regular phone number and a Raspberry Pi.
1. Hardware(no soldering required):
* Get a USB audio card for your Raspberry PI. It is way simpler if it has separate audio out and mic in/line in jacks
*  If the machine running voice communication software has combined audio out/mic in, I suggest you also get a USB audio card. It is way simpler than trying to cross over TRRS.
*  Use two male to male TRS cables connect out of one card to in of the other.
2. VOIP Provider
Configure an account with one of VOIP providers. I use [voip.ms](https://voip.ms). You will need a direct inward dial(DID) number. It is also a good idea to make sure some sort of security is in place. Otherwise anyone who knows the phone number will be able to hear what is going on and "speak into the microphone". In case of voip.ms, callback or IVR would do the trick. 
3. Software:
  * Install [Pi-KVM](https://pikvm.org/download.html) on your raspberry pi
  * Build pjsua
      SSH into pikvm box and execute the following
```bash
pacman -S alsa-lib
wget http://www.pjsip.org/release/2.3/pjproject-2.3.tar.bz2
tar xvfj pjproject-2.3.tar.bz2
cd pjproject-2.3/
./configure --disable-video
echo "#define PJMEDIA_AUDIO_DEV_HAS_ALSA       1" >>  pjlib/include/pj/config_site.h
echo "#define PJMEDIA_AUDIO_DEV_HAS_PORTAUDIO  0" >>  pjlib/include/pj/config_site.h
echo "#define PJMEDIA_CONF_USE_SWITCH_BOARD    1" >>  pjlib/include/pj/config_site.h
make && make install
```
* Create a pjsua.conf file that in my case looks like this
```
--registrar sip:toronto1.voip.ms
--id sip:<voip.ms account or subaccount>@toronto1.voip.ms
--realm toronto1.voip.ms
--username <voip.ms account or subaccount>
--password <voip.ms account or subaccount password>
--auto-answer 200
--max-calls 1
--playback-dev 0
--capture-dev 0
```
* Now you can run like this(consider creating a service for autostart). 
```
./pjsua-armv7l-unknown-linux-gnueabihf --config-file ./config.conf
```
