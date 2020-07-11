---
title: 搭建超好玩的ts3server重点事项
urlname: breakpointofbuildts3server
tags: 手记
abbrlink: 6705
date: 2019-08-04 14:24:29
---

# 【teamspeak&VPS】搭建超好玩的ts3server的突破点

> teamspeak这款语音社交软件真是太棒了，可惜就是高版本得服务器不能被低版本的客户端连接上。
>
> 在这里我就不再详细阐述具体如何搭建的了。

搭建概要：创建专有非登录用户，解压软件包并添加到机器的服务中，开启端口

---



## 需要注意的点

1. 接受协议(软件目录生成此文件)

``` bash
touch .ts3server_license_accepted
```

2. 记得将所有权交给专属用户

```bash
chown -R <username>:<group> <path_teamspeak>
```

3. 允许端口

``` bash
root@teamspeak:~# iptables -A INPUT -p udp --dport 9987 -j ACCEPT
root@teamspeak:~# iptables -A INPUT -p udp --sport 9987 -j ACCEPT
root@teamspeak:~# iptables -A INPUT -p tcp --dport 30033 -j ACCEPT
root@teamspeak:~# iptables -A INPUT -p tcp --sport 30033 -j ACCEPT
root@teamspeak:~# iptables -A INPUT -p tcp --dport 10011 -j ACCEPT
root@teamspeak:~# iptables -A INPUT -p tcp --sport 10011 -j ACCEPT
# 或者
root@teamspeak:~# ufw allow 9987
root@teamspeak:~# ufw allow 10011
root@teamspeak:~# ufw allow 30033
```

4. 服务配置

```bash
root@teamspeak:~# vim /lib/systemd/system/teamspeak.service
# 适当根据目录修改
[Unit]
Description=Team Speak 3 Server
After=network.target
 
[Service]
WorkingDirectory=/home/teamspeak/
User=teamspeak
Group=teamspeak
Type=forking
ExecStart=/home/teamspeak/ts3server_startscript.sh start inifile=ts3server.ini
ExecStop=/home/teamspeak/ts3server_startscript.sh stop
PIDFile=/home/teamspeak/ts3server.pid
RestartSec=15
Restart=always
 
[Install]
WantedBy=multi-user.target
```



[**What is ufw？**](https://linux.cn/article-8087-1.html)