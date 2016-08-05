---
layout: post
title: My Own Debian Repo
date: 2015-09-06
tags:
  - linux
  - debian
---
``` bash
wget http://repo.haguro.moe/dists/sid/Release.key
sudo apt-key add - < Release.key
sudo apt-get update```
to add key.

Add ``deb http://repo.haguro.moe/ sid main`` to ``/etc/apt/sources.list``.

This is my own debain repo,where packages are built in debian sid.

# Package list:
* baka-mplayer
* libqtshadowsocks
* mozc-ut
* shadowsocks-qt5
* shadowsocks-libev
