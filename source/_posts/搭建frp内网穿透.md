---
title: 搭建frp内网穿透
date: 2019-07-02 09:08:51
tags: "运维"
---
### 环境
1：云服务器一台
2：内网服务器一台
3：Frp，可以到[这里](https://link.zhihu.com/?target=https%3A//github.com/fatedier/frp/releases)下载。
4：域名一个（可选）

### 内网穿透实现ssh连接
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

还有以下设置可以选择写入服务端配置文件：
``` bash
[common]
#服务端监听地址和端口，该端口是和客户端通信的端口，底层通信端口，tcp
#bind_addr为0.0.0.0含义就是服务端服务器上的所有IP都可以监听外部连接请求
#如果需要配置单个就将服务器的某一个公网IP写在bind_addr
bind_addr = 0.0.0.0
bind_port = 7000

#可以使用nat打洞，进行点对点的连接，这时候数据通信不仅过frp服务端
bind_udp_port = 7001

#可以使用底层通信端口为kcp
kcp_bind_port = 7000

#考虑到这个场景：内网的的数据库是个备库，frps这边要把本地的数据库备份到内网机器上，在此使用proxy_bind_addr = 127.0.0.1 就可以达到这个效果，还很安全
proxy_bind_addr = 127.0.0.1

#域名访问内网的web服务，需要将域名解析到服务器，A记录或者CNAME记录
vhost_http_port = 80
vhost_https_port = 443

#http相应超时，默认60s
vhost_http_timeout = 60

#frpweb统计界面
dashboard_addr = 0.0.0.0
dashboard_port = 7500

#web统计界面认证信息
dashboard_user = admin
dashboard_pwd = admin

#frp服务端日志文件
log_file = ./frps.log

#日志等级 trace, debug, info, warn, error
log_level = info

#日志保留天数
log_max_days = 3

#frp服务端和客户端通过bind_port端口进行认证的token，服务端和客户端都要一直
token = 12345678

#服务器上可以用映射的端口
allow_ports = 2000-3000,3001,3003,4000-50000


#默认情况下：当用户请求建立连接后，frps 才会请求 frpc 主动与后端服务建立一个连接 
#如果由大量短连接的情况呢，是不是很和耗时？因此，该字段会在和frp客户端连接后，主动建立max_pool_count个连接，当有用户来访问业务的是时候就会从该连接池内取出连接来用，使用于大量短连接的情况
max_pool_count = 5

#限制frpc使用服务端端口的数量。0为不限制
max_ports_per_client = 0

#该功能需要泛解析，即把*.frps.com A记录到frps在的服务器上，然后frpc就可以只配置subdomain = test 就可以使用 test.frps.com来使用，多人使用很方便
subdomain_host = frps.com

#启动tcp多路复用
tcp_mux = true
```
通过执行指令 ./frpc -c frpc.ini 来启动服务器端的Frp服务。
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
server_addr是你Frp服务器的公网IP，
server_port为服务器端Frp监听的端口，必须与Frps.ini中的配置端口一致，否则无法连接到服务器，
[ssh]部分是开启SSH的相关配置，其中type说明该配置的类型为tcp协议，如果有多个SSH端口配置[ssh]标题则必须唯一！
local_ip为你内网机器的IP，可以填IP和可以填127.0.0.1，local_port是内网需要监听的端口，
ssh服务需要指定的端口为22端口，
remote_port是你指定的需要映射到公网服务器上的端口，以后进行ssh连接就需要用到该端口。

还有以下设置可以选择写入客户端配置文件：
``` bash
[common]
#和frps底层通信的信息，和bind_addr = 0.0.0.0 bind_port = 7000 保持一致
server_addr = 0.0.0.0
server_port = 7000

#frpc通过代理去连接frps。http或socks5代理方式
#http_proxy = http://user:passwd@192.168.1.128:8080
http_proxy = socks5://user:passwd@192.168.1.128:1080

#日志文件
log_file = ./frpc.log

#日志等级 trace, debug, info, warn, error
log_level = info

#日志最大保留天数
log_max_days = 3

#和frps底层通信认证token和frps一致
token = 12345678

#热加载frpc配置使用 frpc reload -c ./frpc.ini
admin_addr = 127.0.0.1
admin_port = 7400
admin_user = admin
admin_pwd = admin

#和frps刚开始通信时建立的连接数量
pool_count = 5

#开启tcp多路复用 和frps一致
tcp_mux = true

#加入frpc标识，下面的模块使用 如 your_name.ssh
user = your_name

#frpc和frps登陆失败后是否继续持续登陆
login_fail_exit = true

#和frps通信的底层协议 tcp kcp websocket,  默认是 tcp
protocol = tcp

```
通过执行指令 ./frpc -c frpc.ini 来启动客户端的Frp服务，这样我以后就可以通过服务器公网IP和7000端口来连接我的内网机器了（ssh）

注意：windows客户端需在cmd中在frp更目录下执行 frpc -c frpc.ini 来启动客户端服务。且开启后不能关闭cmd，否则会中断连接！

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

### 遇到的一些问题

如果你使用的是云服务器，请把Frp使用到的端口全部加入出入白名单。否则会出现端口连接报错问题。

如果开启了防火墙，请把所有Frp使用到的端口全部加入出入规则当中。否则会出现端口连接报错问题。