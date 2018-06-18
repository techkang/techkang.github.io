---
layout:     post  
title:      NEWS-网站迁移到https    
subtitle:   所有到达 80 端口的请求都会被重定向至 443 端口    
date:       2018-06-18  
author:     techkang  
header-img: img/post-bg-digital-native.jpg  
catalog: true  
tags:  
    - news
    - nginx   
---  

## 介绍

我的个人主页已经支持到 https。

## 实现过程

主要参考了 [Linux大神博客](https://www.linuxdashen.com/https%E5%8A%A0%E5%AF%86%E7%AE%80%E4%BB%8B%E4%BB%A5%E5%8F%8Anginx%E5%AE%89%E8%A3%85-lets-encrypt-%E5%85%8D%E8%B4%B9ssltls%E8%AF%81%E4%B9%A6) 。使用免费开源的 Let’s Encrypt 产生证书。为下一步配置邮箱服务器打下坚实的基础。


