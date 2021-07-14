---
layout: post
title: "GitHub in corporate envirinments"
description: ""
category:
tags: [security, git, github, squid, mitm, ssl, development]
---

A growing number of corporations is justifiably concerned with data exfiltration risks. They often use a sledgehammer of banning access to services like GitHub. This severely handicaps their developers. Is there a better way?

For companies whose arsenal includes corporate proxies with man in the middle(MITM) capabilities(not advocating for or against them or discuss legal implications) can easily block git and other upload functionality while allowing looking at the code.

Here's a simple way of checking this approach using Squid on Ubuntu. It is for testing only and further hardening of squid setup and configuration is likely required.

Squid has had SSL bump feature(MITM supporting functionality) for a while. But it seems that due to [opensssl GPL license incompatibility](opensssl license incompatibility) Squid version packaged for debian based distributions omits this feature. Luckily it is easy enough to build yourself.

1. Install build tools
```bash
sudo apt install gcc
sudo apt install g++
sudo apt install libssl-dev
sudo apt install make
```

2. Download, build and install
```bash
wget http://www.squid-cache.org/Versions/v4/squid-4.16.tar.xz
tar xJvf squid-4.16.tar.xz
cd squid-4.16
./configure --with-openssl --enable-ssl-crtd
make -j $(($(grep ^cpu\ cores /proc/cpuinfo | uniq | sed s/[^0-9]//g)+1))
sudo make install
```
3. Configure and start
```bash
#generate ssl certs
sudo openssl req -new -newkey rsa:4096 -sha512 -days 365 -nodes -x509 -extensions v3_ca -keyout /usr/local/squid/etc/ssl/cakey.pem  -out /usr/local/squid/etc/ssl/cacrt.pem
sudo openssl dhparam -outform PEM -out /usr/local/squid/etc/ssl/dhparam.pem 4096#initialize
#initialize squid ssl_db
sudo /usr/local/squid/libexec/security_file_certgen  -c  -s  /usr/local/squid/var/lib/ssl_db -M 4MB
```
Place squid.conf similar to [this](/attachments/2021-07-14-github-safely/squid.conf) into /usr/local/squid/etc/

4. Test
```bash
export https_proxy=http://localhost:3128
#for simplicity ignoring the fact that our CA is untrusted. 
#Don't do it in real life!!! Install your trusted CA cert instead
curl -k https://github.com
```
This should return html produced by github.
```bash
curl -k -X POST https://github.com
```
This should return 403 and html produced by squid, saying that access is denied.
You could further ensure that git does not work, by running the following:
```bash
#configure git to use proxy
git config --global https.proxy http://localhost:3128
git config --global http.proxy http://localhost:3128
#Never in real life!!!
git config --global http.sslVerify false
#checkout
git clone https://github.com/github/training-kit
```
You will receive 403 error
Github has been blocking [git dumb protocol](https://git-scm.com/docs/http-protocol) for [ten years now](https://github.blog/2011-03-09-git-dumb-http-transport-to-be-turned-off-in-90-days/). To verify let us rerun with GIT_SMART_HTTP variable set to 0
```bash
export GIT_SMART_HTTP=0
#checkout
git clone https://github.com/github/training-kit
```
This will result in 403 with a link to the [github dumb git protocol retirement announcement](https://github.blog/2011-03-09-git-dumb-http-transport-to-be-turned-off-in-90-days/)
