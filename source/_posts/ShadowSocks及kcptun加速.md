---
title: ShadowSocks及kcptun加速
date: 2017-02-25 16:47:17
tags: 工具
---


前段时间买了个vps.主要是为了科学上网及linux练手.使用了`shadowsocks`来科学上网.后来听说`kcptun`可以对`shadowsocks`加速,所以又折腾了一下.

`Shadowsocks`现在是科学上网的主选.在windows,Linux及Mac平台下都有对应的图形化程序可以直接使用.所以配置容易.

KCP 是一个快速可靠协议，能以比 TCP 浪费10%-20%的带宽的代价，换取平均延迟降低 30%-40%，且最大延迟降低三倍的传输效果

`kcptun`是一个非常简单和快速的，基于 KCP 协议的 UDP 隧道，它可以将 TCP 流转换为 KCP+UDP 流,分为服务端和客户端,都需要使用命令行模式来配置.所以有点麻烦.

<!-- more -->
### shadowsocks

`shadowsocks`的原理是通过把`http`连接转换成`socks5`连接来进行科学上网的.

一般的网络连接有四种协议

1. `http` 通常浏览器都使用
2. `mail` 收发邮件使用
3. `ftp` 上传下载
4. `socks` 目前有v4和v5两个版本

`shadowsocks`的作用就是把本来通过`http`协议的请求通过`socks`协议进行发送,然后把收到的响应转发到指定的端口\(默认是**1080**\).

#### 全局和自动代理

全局模式就是拦截所有的`http`,全通过`socks`连接.但这样不够效率,而且浪费服务器端的流量,所以大多都选择自动代理模式.也就是只有需要科学上网的时候才使用`shadowsocks`来代理连接.

除了mac可以自动配置系统代理外,在`Linux`和`window`平台都需要在配置完`shadowsocks`之外还要配置浏览器.

通常浏览器使用代理插件时需要pac文件来判断是否需要代理.如果需要就使用配置的代理,也就是我们配置的`shadowsocks`,否则就是直接连接.

也就是说自动代理是浏览器端是可以选择把请求直接发送或交给`shadowsocks`来处理.而全局模式就是浏览器发送的任何请求都会被`shadowsocks`给拦截并处理.

#### shadowsocks的配置

`shadowsocks`的配置是可以保存在一个`json`文件中的.目前的图形化程序都是加载`json`文件来配置的.如

```json
{
"server":"10.10.10.10",
"server_port":10080,
"local_address":"127.0.0.1",
"local_port":"1080",
"password":"xxxxx",
"timout":300,
"method":"ase-256-cfb"
"fast_open":true
}
```

很简单的几项.

* `local`

接收和发送http连接的ip地址和端口.通常ip是`127.0.0.1`也就是本机,端口通常为**1080**,有的图形化程序不能设置端口

浏览器或系统代理会把需要科学上网的请求发送到这个端口上,并监听这个端口获取响应.

* `server`

需要把`socks`请求发送的服务器地址及端口

* `password`及`method`

加密方式及密码.这两个由服务器决定

简单的来说就是`shadowsocks`提供了一个本地的入口`local`,通过这个入口接收所有需要科学上网的请求,然后把请求通过`method`加密,附带`password`发送到`server`的`server_port`端口,并接收响应.

### kcptun

[官网地址](https://github.com/xtaci/kcptun)

#### 原理

详细原理请自行`google`

#### 安装

##### 服务器端

一般服务器都是`Linux`系统的,需要使用`ssh`方式连接.`windows`下需要使用`Xshell`或`puttyssh`来连接,`mac`和`linux`可以直接使用`ssh`命令

> `ssh -p [服务器的ssh端口] [帐号]@[服务器地址]`

连接到服务器后使用的是命令行.

最简单的安装方式就是使用别人配置好的脚本自动化安装,需要分别输入以下指令,每个命令输入后按回车执行.执行完比后再输入下一条.

此方式需要`root`权限.可以用su临时切换成`root`帐户

> 1. `wget https://raw.githubusercontent.com/kuoruan/kcptun_installer/master/kcptun.sh`
> 2. `chmod +x ./kcptun.sh`
> 3. `./kcptun.sh`

这三行命令的意思是从网上下载`kcptun.sh`脚本,给这个脚本设置可执行权限,然后执行这个脚本.
执行这个脚本后会自动安装`kcptun`服务端及相关软件.然后进行配置.

打开这个脚本可以看到具体的安装方式

1. 服务端的下载地址`https://api.github.com/repos/xtaci/kcptun/releases`,脚本会根据下载地址查找最新的版本,及下载地址列表,然后根据系统的版本决定具体的下载链接
2. 把服务端下载到本地并解压,把其中的服务端保存在`/usr/share/kcptun`目录下.
3. 下载并安装`json`解析软件.脚本使用的是`https://github.com/stedolan/jq/releases/download/jq-1.5/jq-linux64`或`jq-linux32`,根据服务器版本决定
4. 使用默认的配置来配置`kcptun`
5. 使用`json`解析器读取默认配置.并写入安装目录，默认保存为config.json
6. 下载并安装`supervistor`来管理`kcptun`

个人觉得这个脚本用起来已经很方便了.不过可以不使用默认配置,而是改为自己的配置.免得安装完以后再配置一次

1. 修改默认配置为你自定义的配置
2. 使用`scp`命令把改好的脚本上传至服务器,或把脚本放在网上然后让服务器通过`wget`下载
3. 让脚本可执行,并执行脚本

##### 客户端

1. 从`https://github.com/xtaci/kcptun/releases/tag/v20161222`下载客户端.目前的机型都用带`amd-64`的.`386`是给虚拟机和老式机型使用的.`linux`平台下载带`linux`的.
2. 解压后`client`开头的就是客户端,如`client_linux_amd64`
3. 官网上提供了简单的客户端命令行配置。但建议客户端也使用`json`文件来配置。`client_linux_amd64 -c kcptun.json`,`kcptun.json`是配置文件
4. 建议写个脚本。开机运行。如`startKcp.sh`

```
#!sh
client_linux_amd64 -c /etc/share/kctpun/kcptun.json
```

##### 配置

在服务端安装脚本中定义了一个`DEFAULT`的数组就是默认配置数据。对应的是下面初始化参数的各项属性。写成的`json`文件如下

```json
{
"listen_port" : 29900
"target_ip" : "127.0.0.1"
"target_port":12948
"key":"it's a secrect"
"crypt":"aes"
"mode":"fast"
"mtu":1350
"sndwnd":1024
"rcvwnd":1024
"datashard":10
"parityshard":3
"dscp":0
"nocomp":false
"nodelay":0
"interval":20
"resend":2
"nc":1
"acknodelay":false
"sockbuf":4194304
"keepalive":10
"run_user":"root"
}
```

服务端与客户端使用配置格式是一样的。其中需要自己配置的就几项

1. 端口`linsten_port`,`target_ip`,`target_port`

| 属性 | 客户端 | 服务器 |
| --- | --- | --- |
| listener\_port | 与shadowsocks交互的端口 | 对外的端口 |
| target\_ip | 服务器的IP | 本机 |
| target\_port | 服务器的端口（listen\_port） | 与shadowsocks交互的端口 |

2. `key`和`crypt`，密码和加密方式。`key`是密码，`crypt`为加密方式，常用的有`ase`，`ase-128`。
3. `mode` 加速模式。从低到高分别是`default`,`noraml`,`fast`,`fast2`,`fast3`,越高越耗费带宽
4. `mtu`,`sndwnd`,`rcdwnd`,`UDP`包的设置.在100M带宽下建议使用以下设置,其它带宽客户端的`sndwnd`,`rcdwnd`按比例设置
```
服务端: -mtu 1400 -sndwnd 2048 -rcvwnd 2048
客户端: -mtu 1400 -sndwnd 256 -rcvwnd 2048 -dscp 46
```

##### 客户端的连接
1. 浏览器,如果不是Mac则需要使用插件配置代理.代理设置为`sock5`,地址为`127.0.0.1`,端口为`shadowsocks`的`local_port`,默认是`1080`
2. `shadowsocks`,需要根据服务端设置密码及加密方式,地址为`127.0.0.1`,端口为`kcptun`的`listener_port`
3. `kcptun`,地址为服务器地址,端口为服务器`kcptun`的`listener_port`

[原文地址](http://www.jianshu.com/p/ad53a46e6d90)
