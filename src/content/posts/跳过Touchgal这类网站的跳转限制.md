---
title: 跳过Touchgal这类网站的跳转限制的思路
published: 2025-11-28
description: ''
image: ''
tags: [js, tampermonkey]
category: 'js'
draft: false 
lang: ''
---

# 跳过Touchgal这类网站的跳转限制

  Touchgal在从主站跳转到网盘链接的时候，会有5s的时长限制，于是想写一个油猴脚本来跳过，常规思路是直接截取跳转界面网址`redirect=`后面的目标链接，但是对于`Touchgal`这类网站来说弹到跳转界面时不会触发tampermonkey注入脚本。

  我的思路是重写`history.pushState`和`history.replaceState`,触发时，调用自动跳转方法。
  
代码实现:
```javascript
// ==UserScript==
// @name         TouchGal 自动跳转
// @namespace    http://tampermonkey.net/
// @version      1.0
// @description  跳过5s等待
// @match        https://www.touchgal.us/*
// @grant        none
// @run-at       document-start
// ==/UserScript==

(function() {
    'use strict';

    let lastUrl = location.href;

    //截取并跳转
    function autoRedirect() {
        const currentUrl = location.href;

        // 防止重复执行
        if (currentUrl === lastUrl) return;
        lastUrl = currentUrl;

        // 检查是否是重定向页面
        if (currentUrl.includes('/redirect?url=')) {
            // 截取 url= 后面的内容
            const targetUrl = getUrlParameter('url');

            if (targetUrl) {
                console.log('检测到重定向链接:', targetUrl);
                console.log('正在跳转...');

                // 执行跳转
                window.location.href = decodeURIComponent(targetUrl);
            } else {
                console.warn('未找到目标链接');
            }
        }
    }

    // 提取 URL 参数的函数
    function getUrlParameter(name) {
        const urlParams = new URLSearchParams(window.location.search);
        return urlParams.get(name);
    }

    autoRedirect();
    // 保存原始方法
    const originalPushState = history.pushState;
    const originalReplaceState = history.replaceState;

    // 重写 pushState
    history.pushState = function() {
        originalPushState.apply(history, arguments);
        setTimeout(autoRedirect, 100);
    };

    // 重写 replaceState
    history.replaceState = function() {
        originalReplaceState.apply(history, arguments);
        setTimeout(autoRedirect, 100);
    };

    // 监听浏览器前进/后退
    window.addEventListener('popstate', function() {
        setTimeout(autoRedirect, 100);
    });

    window.addEventListener('hashchange', function() {
        setTimeout(autoRedirect, 100);
    });
})();
```
## github
::github{repo="nostalgia296/backup"}

或者`[greasyfork](https://greasyfork.org/zh-CN/scripts/532618-touchgal-%E8%87%AA%E5%8A%A8%E8%B7%B3%E8%BD%AC/code)`