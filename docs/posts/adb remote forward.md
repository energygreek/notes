---
title: adb remote forward
comments: true
date: 2025-04-24 17:53:37
tags: [adb]
---

# 连接安卓设备后，通过adb连接实现远程桌面

1. 首先跳板机连接安卓，这里是wifi方式，adb connect 192.168.0.32:5555
2. 启动隧道转发，