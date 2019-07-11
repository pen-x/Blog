---
title: 搭建 shadowsocks 服务器 
tags:
  - shadowsocks
---


## 前言

本文基于MSDN订阅版本对应的Azure云服务提供的虚拟机服务，搭建自己的 shadowsocks 服务器。

## 系统环境

## 搭建 shadowsocks 服务器

这里我首先在 Azure 上创建了一台位于东南亚的 Ubuntu 虚拟机，创建完成后登陆该虚拟机：

```bash
ssh penx@123.45.67.89
```

一般默认已经预装了 python，我们直接查看一下 python 的版本：
```bash
$ python --version
Python 2.7.12
```

安装 pip 软件管理程序 并使用 pip 安装 shadowsocks：

```bash
$ sudo apt install python-pip
$ pip install shadowsocks
```

创建 shadowsocks 服务器配置文件，一般我们习惯将各个软件的配置文件放置在 /etc 目录下：

```bash
$ vi /etc/shaowsocks.json

{
    "server":"0.0.0.0",
    "server_port":8388,
    "local_port":1080,
    "password":"barfoo!",
    "timeout":600,
    "method":"aes-256-cfb"
}
```

可以根据自己的需要选择合适的端口号，加密方式等，这里我们保持默认的设置。

接着我们运行 ssserver 命令，启动 shadowsocks 服务器：

```bash
$ ssserver -c /etc/shadowsocks.json

2018-09-18 07:02:15 INFO     loading libcrypto from libcrypto.so.1.0.0
2018-09-18 07:02:15 INFO     starting server at 0.0.0.0:8388
```


## 测试

显示如下信息，表明服务器正常工作。

```bash
2018-09-18 07:57:53 INFO     connecting google.com:80 from 117.136.0.239:16836
2018-09-18 07:57:53 INFO     connecting www.google.com:80 from 117.136.0.239:16837
2018-09-18 07:57:53 INFO     connecting www.google.com:443 from 117.136.0.239:16838
2018-09-18 07:57:54 INFO     connecting ssl.gstatic.com:443 from 117.136.0.239:16840
2018-09-18 07:57:55 INFO     connecting www.gstatic.com:443 from 117.136.0.239:16841
2018-09-18 07:57:55 INFO     connecting www.google.com:443 from 117.136.0.239:16839
2018-09-18 07:57:56 INFO     connecting adservice.google.com:443 from 117.136.0.239:16842

```

## 高级配置

关闭 ssh 会话，我们发现无法继续访问了。

```
$ ssserver -h

usage: ssserver [OPTION]...
A fast tunnel proxy that helps you bypass firewalls.

You can supply configurations via either config file or command line arguments.

Proxy options:
  -c CONFIG              path to config file
  -s SERVER_ADDR         server address, default: 0.0.0.0
  -p SERVER_PORT         server port, default: 8388
  -k PASSWORD            password
  -m METHOD              encryption method, default: aes-256-cfb
  -t TIMEOUT             timeout in seconds, default: 300
  --fast-open            use TCP_FASTOPEN, requires Linux 3.7+
  --workers WORKERS      number of workers, available on Unix/Linux
  --forbidden-ip IPLIST  comma seperated IP list forbidden to connect
  --manager-address ADDR optional server manager UDP address, see wiki

General options:
  -h, --help             show this help message and exit
  -d start/stop/restart  daemon mode
  --pid-file PID_FILE    pid file for daemon mode
  --log-file LOG_FILE    log file for daemon mode
  --user USER            username to run as
  -v, -vv                verbose mode
  -q, -qq                quiet mode, only show warnings/errors
  --version              show version information

Online help: <https://github.com/shadowsocks/shadowsocks>
```

可以看到 ssserver 支持以守护进程形式运行。

```bash
$ which ssserver

/home/penx/.local/bin/ssserver
```

```bash
$ sudo /home/penx/.local/bin/ssserver -c /etc/shadowsocks.json -d start

INFO: loading config from /etc/shadowsocks.json
2018-09-18 08:00:15 INFO     loading libcrypto from libcrypto.so.1.0.0
started
```

还可以配置为系统启动服务：



## 参考

- [Shadowsocks Offical Site](https://shadowsocks.org/en/index.html)
- [Shadowsocks折腾记](https://thief.one/2017/02/22/Shadowsocks%E6%8A%98%E8%85%BE%E8%AE%B0/)
- [从零开始VPS (二): 使用更开放的互联网](https://jecvay.com/2015/01/learning-vps-2.html)