---
draft: true
comments: true
date: 2025-12-12
tags: []
---

# Proxy via frp and shadowsocks
买了一个frpc账号，用来将NAS服务暴露到公网。但是有很多不安全问题
1. frp服务器不是我的，老板可以监控隧道的流量。
2. 没有认证，公网上谁都能访问。
于是另外加了端口，专门用来访问受限内容，加上了Shadowsocks, 再套上自签名的SSL证书，太完美了。