---
layout:     post  
title:      shadowsocks 工作原理分析    
subtitle:   实践出真知    
date:       2018-07-02  
author:     techkang  
header-img: img/post-bg-os-metro.jpg  
catalog: true  
tags:  
    - shadowsocks
    - vpn   
---  

## 介绍

同学去美国暑研，到达美国后发现 MATLAB 许可证一天后过期，于是向我寻求帮助。我在 Windows 下用 pip 安装 ss 服务端后开启服务，并指导同学安装客户端，选择全局模式，之后果然成功更新，之前我已经知道 ss 采用了 Socks5 协议，不能代理命令行。于是趁这个机会我希望彻底搞清楚 ss 的工作机制。

## 简述

简单来说，Socks5 协议是运行在传输层的一个协议，网络资料不是很全，但我认为这个协议类似于 ICMP 和 IP 的关系一样，因为 ss 流量最后还是通过 tcp 建立连接，故 Socks5 应该是 tcp 层之上的一个协议。故不能代理命令行（ping 被屏蔽的网站依然失效），然而可以代理网页。那么同学的 MATLAB 激活时，发送的数据包被 ss 客户端劫持，转发到我的电脑上，我的电脑再次转发，此时源 IP 变成了我的 IP，故会认为在校园网环境下。解决这个问题的一个简单策略就是 MATLAB 尝试读取本机 IP，并写入发送的数据包中，在服务器端进行校验，然而如果要更新的机器是机房或实验室的机器就会出问题，因为这些机器一般共用 IP，这么说来，IPv6 可以解决这个问题，一个杀手级应用(薛sir语)？hhh




