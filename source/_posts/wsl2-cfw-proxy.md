---
title: WSL2 配置 Clash 代理
tags: wsl2
categories:
  - [Tech,Linux,wsl2]
---

# 防火墙设置
+ `控制面板` > `系统和安全` > `Windows Defender 防火墙` > `允许应用通过 Windows 防火墙`，勾选上所有 Clash 相关的应用，包括但不限于`Clash for Windows`、`clash-win64`等,专用和公共需要同时勾选。

# Clash设置
+ 正常配置好 `Clash For Windows`，并启用`Allow LAN`

# WSL2配置
+ 在WSL2的虚拟机中运行以下命令后，可以正常使用代理（鼠标悬浮在 Allow LAN 上，可以看到 `WSL宿主机ip`，clash 默认代理端口为`7890`）
````bash
export https_proxy="http://<WSL宿主机ip>:7890"
export http_proxy="http://<WSL宿主机ip>:7890"
export all_proxy="sock5://<WSL宿主机ip>:7890"
export ALL_PROXY="sock5://<WSL宿主机ip>:7890"
````
+ 但是在重启后，WSL宿主机ip可能会改变；而且每次都要输入命令，比较麻烦，为了避免重复劳动，我们可以写个脚本，动获取`WSL宿主机ip`，比如：
```bash
hostip=$(cat /etc/resolv.conf |grep -oP '(?<=nameserver\ ).*')
alias setp='export https_proxy="http://${hostip}:7890";export http_proxy="http://${hostip}:7890";export all_proxy="socks5://${hostip}:7890";export ALL_PROXY="socks5://${hostip}:7890";'
alias unsetp='unset https_proxy; unset http_proxy; unset all_proxy; unset ALL_PROXY;'
```
+ 现在，在终端输入 setp 即可开启代理， 输入 unsetp 即可解除代理，经过测试，一切正常：
  
![](https://raw.githubusercontent.com/Lukrisum/temp-asset/main/wsl2-cfw-proxy-test.png)

> 相关链接：
> [Windows 原生运行Linux如何自由访问互联网 WSL2 使用 Clash for Windows做代理](https://www.v2fy.com/p/2021-09-24-windows-clash-wsl2-1632440722000/)