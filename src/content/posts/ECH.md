---
title: 免翻访问greasyfork等网站
published: 2025-12-12
description: ''
image: ''
tags: [网络]
category: '网络'
draft: false 
lang: ''
---

# ECH

ECH可以加密**Inner ClientHello**包，所以使用ECH后可以加密sni字段，让gfw看不到sni了，利用这一点就可以免翻访问某些网站了。

ECH需要服务器支持，greasyfork就支持，同时还需要DoH服务器，如果用chromium系浏览器的话，设置竟然大概可以找到一个加密dns选项，开始选择一个就好了

ECH只是加密**Inner ClientHello**包,所以不是所有使用了ECH的网站都可以免翻，如果想要免翻更多网站的话，可以试试`sni伪装`(不安全)