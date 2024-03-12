---
title: "Whistle使用技巧"
description: 
date: 2024-03-12T15:50:26+08:00
image: 
math: 
license: 
hidden: false
comments: true
language: 汉语
categories:
    - 前端开发技巧
tags: 
    - Whistle
    - 开发工具
    - SwitchyOmega
---

通过Whistle.js配合浏览器插件SwitchyOmega，可以灵活的代理url。

eg：本地3000接口代理mysite.com域名下，除了```*/db```之外的所有请求，

```whistle
127.0.0.1:3000 mysite.com excludeFilter://*/db
mysite.com reqHeaders://{header}


// values
header = {
    "someHeader": "myParams"
}

```
