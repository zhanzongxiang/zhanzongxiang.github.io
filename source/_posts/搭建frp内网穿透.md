---
title: 搭建frp内网穿透
date: 2019-07-02 09:08:51
tags:
---
## 概述
## 环境
1：云服务器一台
2：内网服务器一台
3：Frp，可以到[这里](https://link.zhihu.com/?target=https%3A//github.com/fatedier/frp/releases)下载。
4：域名一个（可选）

## 内网穿透实现ssh连接
首先需要到GitHub上去下载Frp的相关脚本。下载时需要注意的是，服务器端和内网机器端下载的版本要相同，否则不能完成内网穿透。还有就是对应的系统也要正确，由于我的服务器是安装的linux，处于内网的机器是windows，所以服务器就需要下载Linux对应的，内网机器上下载windwos系统的。
![](file:///your file directory/img.jpg)
下载完成后需要分别对文件进行配置，服务器端用到的是Frps和Frps.ini这两个文件，客户端用到的是Frpc和Frpc.ini这两个文件。

### 配置服务器端
Frps.ini最初始的配置如下：
``` bash
[common]
bind_port = 7000
```
这段代码表示，frp最初始默认只监听7000端口。此端口为与内外互动的端口，如果默认端口被占用可以改为其他端口。
监听端口指定好以后就可以通过指令 ./frps -c frps.ini 来启动Frp服务。
``` bash
2018/09/26 17:04:31 [I] [service.go:130] frps tcp listen on 0.0.0.0:7000
2018/09/26 17:04:31 [I] [root.go:207] Start frps success
```
如果出现以上提示则说明启动成功，监听的端口为7000。

### 配置内网机器上的Frp服务
Frpc.ini的初始配如下：
``` bash
[common]
server_addr = 127.0.0.1
server_port = 7000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```
server_addr是你公网服务器的公网IP，
server_port为服务器端Frp监听的端口，必须与Frps.ini中的配置端口一致，否则无法连接到服务器，
[ssh]部分是开启SSH的相关配置，其中type说明该配置的类型为tcp协议，
local_ip为你内网机器的IP，可以填IP和可以填127.0.0.1，local_port是内网需要监听的端口，
ssh服务需要指定的端口为22端口，
remote_port是你指定的需要映射到公网服务器上的端口，以后进行ssh连接就需要用到该端口。

修改后的配置如下：
``` bash
[common]
server_addr = xxx.xxx.xxx.xxx
server_port = 7000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 8003
```
通过执行指令 ./frpc -c frpc.ini 来启动客户端的Frp服务，这样我以后就可以通过服务器公网IP和7000端口来连接我的内网机器了（ssh）

需要注意的是：上面用到的端口都需要服务器端已经开放这些端口，否则不能正常启动服务，我用的是阿里云服务器，开启对应的端口可以在添加安全组里添加对应的端口。

### 内网穿透实现web服务
前面实现了ssh的连接，接下来实现访问内网web服务。还是需要求改服务器的配置文件，在之前的基础上增加一条配置：
``` bash
vhost_http_port = 8003
```
添加了一个http请求的监听端口 8003 ，也是以后要访问web服务时需要用到的端口号。
服务器修改完成后接下来再给客户端添加配置项：
``` bash
[web]
type = http
local_port = 8000
custom_domains = xxx.xxxx.xxxx
```
其中local_port是监视本地的http服务端口（也就是http服务占用的端口），custom_domains为你公网服务器的IP或者已解析的域名。

再启动服务器和内网的Frp服务就可以通过custom_domains:vhost_http_port的方式来访问你本地的http服务了。