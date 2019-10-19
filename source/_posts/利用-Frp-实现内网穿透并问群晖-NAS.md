---
title: '利用 Frp 实现内网穿透并问群晖 NAS'
tags:
  - NAS
  - 群晖
  - frp
  - 内网穿透
categories:
  - 技术应用
summary: 将内网的黑群晖暴露到外网
date: 2019-10-16 08:37:32
---

服务器：阿里云Ubuntu 16.04；
NAS：黑群晖 6.1.x;
Frp: v0.18.0
-------
## 背景

因不可名状的原因，在家搭建了一台黑群晖。NAS有外网访问的需求，但家里的宽带运营商是移动的，公网IP无望，所以动态DNS方案不适用，只能考虑内网穿透。
内网穿透技术比较流行的应该是`花生壳`、`Ngrok`、`Frp`。花生壳免费版是鸡肋，首先Pass，`Ngrok`相对成熟，但配置繁琐，也Pass了，那就选择`Frp`。`Frp`不得了，配置简单，跨平台，看到第一眼就喜欢上了。

## 1 前提条件 ##

 1. 拥有公网IP的服务器：因为国外VPS访问速度的原因，仅作梯子用。在内网穿透上，选择了阿里云的ECS。
 2. 一台NAS：原本购入`DS218`，因性价比原因，出掉了`DS218`，闲鱼上买了含CPU的主板和电源机箱，结合自身已有的配件，搭建黑群晖(黑白群晖区别在使用上各取所需就好，综合自身原因，黑群晖更为划算。因常年开机，千万注意功耗！)。

## 2 基本步骤 ##

`Frp`的[项目主页][1]已经给出十分详细的使用文档，若有疑问或冲突，请以官方文档为准。

### 2.1 服务器端配置 ###

本身已购买阿里云的ECS的入门套餐，操作系统选择了自己较为熟悉的`Ubuntu 16.04`。

#### 2.1.1 安装软件并配置参数 ####

对于软件的安装(其实是将`Frp`程序弄到目标路径中)一般有两种方式：

##### 2.1.1.1 服务器端软件下载并配置相关参数 #####

1. 登录服务器，在 [Release页面][2] 下载自己服务器对应版本的 `Frp`。

```BASH
$ wget https://github.com/fatedier/frp/releases/download/v0.18.0/frp_0.18.0_linux_amd64.tar.gz
```

我 ECS 是`Ubuntu 16.04`64位操作系统，所以选择了`linux_amd64`，Frp选择了此时最新的`v0.18.0`。压缩包内包含了服务器和客户端的所有文件。

2. 使用 `tar` 指令解压 `tar.gz` 文件

```bash
$ tar -zxvf frp_0.18.0_linux_amd64.tar.gz
```

3. 进入 `frp` 目录

```bash
$ cd frp_0.18.0_linux_amd64
```

4. 删除不必要的客户端文件

```bash
$ rm -f frpc frpc_full.ini frpc.ini
```

> 版本不同可能内含文件稍有差异。

`frpc` 为客户端运行文件，`frpc.ini`为客户端配置文件，`frps` 为服务器端运行文件，`frps.ini` 为服务器端配置文件。

5. 配置服务器端文件
输入命令

```bash
$ vi frps.ini
```

修改配置文件

``` 
[common]
bind_port = 7000
bind_udp_port = 7001
vhost_http_port = 8080
vhost_https_port = 7433
dashboard_port = 7500
dashboard_user = 用户名
dashboard_pwd = 密码
authentication_timeout = 900  
```

简单释义：

> [common] 必填，公共参数
> bind_port Frp 服务端与客户端TCP通信端口（可自定义）
> bind_udp_port Frp 服务端与客户端UDP通信端口（可自定义）
> vhost_http_port 客户端的 http 访问端口（可自定义）
> vhost_https_port 客户端的 https 访问端口（可自定义）
> dashboard_port 访问 dashboard 界面端口 (dashboard 是 frp 的信息web展示。)
> dashboard_user 登录 dashboard 用户名
> dashboard_pwd 登录 dashboard 密码
> max_pool_count 最大连接池数量
> authentication_timeout 超时验证时间

详细配置请参考官方文档。

[官方文档][3]

6. 保存上面配置文件，进入`frp`目录并启动 `frps` 程序。未列出切换目录的命令，如果没有变动目录直接运行即可。
7. 
```bash
$ ./frps -c ./frps.ini
```
*请注意文件的权限，是否有可执行权限，开机自启动在 配置服务器 中。

##### 2.1.1.2 PC端下载，配置完成后上传至服务器的目标路径中。#####

 > 因为个人水平习惯等因素，我还是喜欢用这种方式，将压缩包直接下载到磁盘路径中，按需求修改配置文件，然后将所需文件上传至服务器目标路径中，修改文件权限。然后启动`frps`程序。

#### 2.1.2 配置服务器 ####

##### 2.1.2.1 在防火墙或云服务安全组策略中开放相关端口 #####

系统可能自带防火墙，且只开放了常用端口，如果有防火墙，你需要检查是否已正确开放端口。
阿里云的ECS在控制台有默认的安全组策略，就类似防火墙作用，如果你的云服务商也默认使用类似功能，请确保已正确的配置相关端口的安全组策略。详细操作参考云服务商资料。

##### 2.1.2.2 使用 Nginx 等软件隐藏非 web 访问默认端口 #####

`http`默认使用`80`端口，而`https`默认使用`443`端口,如果服务器上有web程序，`nginx`等代理软件会占用这些端口。`frp`就的选择其他的端口进行远程访问，比如上方服务器配置文件中的`8080`端口。假设我分配了一个子域名`nas.chafanzhai.com`作为访问NAS的域名，那我需在浏览器输入`nas.chafanzhai.com:8080`才能正确访问NAS，如果仅输入`nas.chafanzhai.com`，访问的是`80`端口，`frp`是无法监听到的。那怎样才能隐藏`8080`端口号呢？就是在`Nginx`中为`frp`配置反向代理。以下为个人使用的配置文件，酌情修改，仅供参考。

```
server {
    listen 80;
    listen 5000;
    server_name nas.chafanzhai.com;
    location / {
    proxy_pass http://127.0.0.1:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    }
}
server {
    listen 443;
    listen 5001;
    ssl on;
    ssl_certificate 证书文件的绝对路径;
    ssl_certificate_key 证书文件的绝对路径;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    server_name nas.chafanzhai.com;
    location / {
    proxy_pass https://nas.chafanzhai.com:7433;
    proxy_ssl_server_name on;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

> 几点解释
> 1. 对于`Nginx`配置的参考网络上的许多文章，看过哪些也忘记了，所以没做引用。对于配置参数的意义，请参考`nginx`相关资料。
> 2. 阿里云ECS域名访问要求域名已备案。
> 3. 因暴露在广域网下，所以配置了`HTTPS`，证书申请的阿里云免费证书，`HTTPS`配置仅供参考，需换成自己证书的绝对路径。
> 4. 反向代理同时监听了`5000`、`5001`端口，是因为需要支持群晖相关APP的远程访问，浏览器访问时使用web的默认端口，但群晖相关app中无法配置访问端口。具体群晖的服务使用了那些端口，你需要监听那些端口请参考群晖官网的[说明文档][4]。
> 5. 在搜索中发现一些同学的方向代理不起作用，需要将`127.0.0.1`替换成服务器的公网IP。我暂未遇到这种情况，列出仅供参考。
   
##### 2.1.2.3 配置frp服务端开机运行 #####

用`supervisor`方法。

1. 先安装`supervisor`

```BASH
$ sudo apt install supervisor
```

2. 创建 `supervisor`的`frps` 配置文件

在 `/etc/supervisor/conf.d` 创建 `frp.conf`，

```
[program:frp]
command = /usr/local/frps/frps -c /usr/local/frps/frps.ini
autostart = true
```

`command`应该是你放置`frp`软件的位置。

我的 `frp` 在 `/usr/local/frps` 这个目录下。

1. 查看状态

```
# 重启supervisor
$ sudo systemctl restart supervisor
# 查看supervisor运行状态
$ sudo supervisorctl status
```

### 2.2 客户端配置 ###

说句题外话，因为我的 CPU 是 AMD 的，开始考虑兼容性问题，一开始装的是`5.9`版本的黑群晖，应该是我不熟悉的原因，自带的终端太难弄了，果断换成了`DSM 6.1`。
好像群晖默认是不启用 `root` 账户，所有操作能用 `admin` 账户 + `sudo` 来完成。个人自带轻微强迫症，没有 `root` 账户，无法访问系统的所有路径，这些都不是我想要的安装位置，如果下载在本地磁盘，修改后上传，我还需要修改文件路径。所以我直接下载在NAS上修改配置文件了。

#### 2.2.1 安装软件并配置参数 ####

##### 2.2.1.1 登陆到NAS，获取`root`权限，进入软件目标路径 #####

输入以下命令

```
$ sudo -i
```

提示输入密码，只需输入`admin`账户的密码并可获取`root`权限，在进入目标路径。
```
$ cd /usr/local
```

##### 2.2.1.2 下载`frp`的目标版本 #####

群晖是基于`linux`的，但请注意你的 cpu 指令集类型，我的是黑群晖,当然还是`linux_amd64`啦。

```
$ wget https://github.com/fatedier/frp/releases/download/v0.18.0/frp_0.18.0_linux_amd64.tar.gz
```

##### 2.2.1.3 使用 `tar` 指令解压 `tar.gz` 文件 #####

```
$ tar -zxvf frp_0.18.0_linux_amd64.tar.gz
```

##### 2.2.1.4 重命名目录，并删除压缩包(强迫症作祟) #####

```
$ mv frp_0.18.0_linux_amd64 frpc
$ rm frp_0.18.0_linux_amd64.tar.gz
```

##### 2.2.1.5 进入 `frp` 目录 #####

```
$ cd frpc
```

##### 2.2.1.6 删除不必要的服务端文件 #####

```
$ rm -f frps frps_full.ini frps.ini
```

##### 2.2.1.7 配置服务器端文件 #####

输入命令

```
$ vi frps.ini
```

修改配置文件

```
[common]
server_addr = 服务器IP
server_port = 7000
# 客户端热加载配置文件
admin_addr = 127.0.0.1
admin_port = 7400

[nas]
type = http
local_port = 5000
custom_domains = nas.chafanzhai.com

[nass]
type = https
local_port = 5001
custom_domains = nas.chafanzhai.com
```

意义和上面服务器配置的差不多，主要多了个`admin_addr`选项用于客户端热加载配置文件的，对于`http`和`https`的端口，需要使用群晖默认的端口，如果是其他自定义的`web`程序需要暴露在公网下，记得在群晖的`Web Station`中配置相关端口噢。

#### 2.2.2 设置开机自启动 ####

##### 2.2.2.1 创建脚本文件 #####

1. 新建脚本文件

```
$ vi /usr/syno/etc.defaults/rc.sysv/S99frp.sh
```

1. 编辑脚本内容

```
#/bin/bash
$ cd /usr/local/frpc
$ nohup ./frpc -c ./frpc.ini &
```

3. 设置文件权限

```
$ chmod +x /usr/syno/etc.defaults/rc.sysv/S99frp.sh
```

##### 2.2.2.2 设置自启动 #####

1. 登录群晖 NAS 系统

2. 进入`控制面板`的`计划任务`中

3. 创建一个`触发的任务` -> `用户定义的脚本`，参数设置如下：

```
常规
设置名称名称，如：frp auto start
用户账号：root
事件：开机
```

![一张非常稀有的图片][5]

4. 任务设置

用户定义的脚本，填入上面创建的脚本

 ```
 /usr/syno/etc.defaults/rc.sysv/S99frp.sh
 ```

![另一张非常稀有的图片][6]

好了，到此为止所有操作已实施，如果你需要`frp`更丰富的功能，请配合官方文档服用。

  [1]: https://github.com/fatedier/frp
  [2]: https://github.com/fatedier/frp/releases
  [3]: https://github.com/fatedier/frp/blob/master/README_zh.md
  [4]: https://www.synology.com/zh-cn/knowledgebase/DSM/tutorial/General/What_network_ports_are_used_by_Synology_services
  [5]: https://assets.chafanzhai.com/usr/uploads/2018/06/2732919655.jpg
  [6]: https://assets.chafanzhai.com/usr/uploads/2018/06/2075144557.jpg