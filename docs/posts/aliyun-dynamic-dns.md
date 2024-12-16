---
title: 阿里云动态设置 dns
date: 2021-01-04 18:23:15
tags: [ddns,nat]
---

# 前言

如果服务器的公网ip动态变化的情况下，如何访问，甚至如何通过域名访问？ 

例如公司自己搭建的服务器如何暴露在公网上， 如果请求固定ip听说很贵， 还可以通过`frp`实现, 这里介绍2种方案

## 动态dns 

前提是服务器有出口ip, 而不是在路由器下。 
  * 如果服务器在路由器下， 通过设置nat,将外网访问的请求的目的ip转换为局域网'192.168.*.*'，也能实现公网请求局域网的服务器
  
阿里云（相信大多数云厂商都支持） 可以支持动态改变dns的解析地址，即通过api调用就能改变dns的解析， 这样当出口ip变化时， 立即调用api来修改dns解析  

实现原理:

1. 定时检测出口ip, 例如每5分钟执行一次。 可以通过crontab和以下命令实现
```
curl https://httpbin.org/ip
```
2. 通过阿里云的api操作dns
3. 如果服务器在内网，添加nat规则，将目的ip转换为内网ip

这里有个现成的[项目](https://github.com/NewFuture/DDNS)

## frp

frp 就是内网穿透了， 没有出口ip的情况下，例如在路由器下且路由器不支持nat的时候， 或者是在运营商级NAT的模式下，就可以采取这中方式。 但是前提是需要有公网的服务器

实现原理：

1. 通过在公网服务器运行frp 服务端， 在没有出口ip的局域网服务器上运行frp客户端
2. 客户端主动去连接服务端， 连接上之后， 服务端会为客户端创建一个端口， 所有的向这个端口的请求都被转发到局域网的服务器，实现公网访问局域网的服务器

## wireguard
wireguard的功能更加强大，配对后就相当于互连了，不需要frp那样配置端口服务。而且自带加密，更安全。

## zero-trust
研究过一会，发现这个和wireguard非常类似的功能。而且zero-trust有cloudflare提供了公网接入点，所以公网服务器的钱也省了, 再加上zero-trust自带的加密，所以是连cloudflare也不能侵入。这就比国内的NAT服务安全多了。唯一缺点估计是zero-trust的公网接入点都在国外，所以延迟会高些。

## 总结 

如果有出口ip, 即使经常变化， 可以使用动态dns的技术实现暴露到公网， 成本低廉。 否则使用frp,需要额外购买服务器，或者使用花生壳类似的穿透技术， 但是有被掏裤裆的风险。
有公网服务器就用wireguard， 否则用zero-trust。

