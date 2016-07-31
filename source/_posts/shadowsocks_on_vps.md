---
title: 如何在VPS上搭建shadowsocks
date: 2016-07-31
tags: 翻墙
categories: 科学上网
---

本教程已经经过本人三番四次检验。

之前一直使用vpn进行翻墙，但一直想找一个用在多个客户端上的翻墙方式，且使用PAC模式进行科学上网，经过一番学习，终于找到了。

Shadowsocks的正常使用需要服务端，有很多提供Shadowsocks账号的网站，在此就不一一列举了，想知道的直接google吧。不过安全性就不知道了。我是自己购买的VPS，然后搭建Shadowsocks进行科学上网，需要有一点点的linux基础，想学习的可以搞下，生命在于折腾嘛！

# VPS推荐 #
所有的翻墙方式都需要提供服务的服务端，如果要自己搭建服务器，就需要拥有一个属于自己的服务器。我用的是VPS，即虚拟专用服务器。下面我是自己选择的VPS供应商。

搬瓦工（Bandwagonhost）
[OpenVZ架构 256MB内存  10GB硬盘 500GB流量/月 19.99美元/年（折合人民币10元/月）](https://bandwagonhost.com/cart.php?a=confproduct&i=0)，这个是我目前找到的比较靠谱，也是比较便宜的，适合轻度折腾的。如果希望配置更高，稳定性更好，可以找配置更高的，也可以去看看以下两个：

[DigitalOcean](https://www.digitalocean.com/?refcode=03e3e84b8f22)

[Linode](https://www.linode.com/?r=69edd5eafe47ed8a7e128c057f3367a90ce51135)（土豪必备有木有）

# VPS的创建与使用 #

搬瓦工的默认系统为Centos，我用的是ubuntu-14.04-x86_64系统，所以首先切换到ubuntu。具体的流程为：

1. 登录自己的账号，点击“Services”里的“My Services”
2. 点击KiwiVm Control Panel，跳到配置页面
2. 点击Admin functions菜单下的Install new OS
3. 选择ubuntu-14.04-x86_64，并且同意所有数据丢失
4. 点击Reload，开始安装新的系统。

> ***针对搬瓦工的专属补充教程：***可能由于搬瓦工官方发现很多人都在他家的VPS上面搭建了Shadowsocks，于是机灵的官方在自家的控制面板里集成了一键搭建服务，大大降低了新手搭建难度，实现了傻瓜化的操作。我简述下流程：点击“Services”里的“My Services”，点击“KiwiVM Control Panel”，这时会跳转到一个新页面。将新页面左侧的滚动条拉到底，找到“Shadowsocks Server”字样并点击进入，然后点击“Install Shadowsocks Server”，几秒钟之后显示Completed的字样就代表完成了。这时候点击“Go back”或者直接点击左侧的“Shadowsocks Server”，你会看到出现了一个叫做“Shadowsocks server controls”的东西，上面有默认的加密方式(encryption)、服务器端口(port)以及密码(password)，你可以直接使用默认的，也可以点击旁边的“Change xx”按钮进行修改，最后检查一下Status是不是Running，不是的话就点一下旁边的“Start”。至此，你的Shadowsocks服务端就搭建完成了，你可以直接跳转到本教程后面的客户端配置部分了(搬瓦工也自带了简易的客户端配置说明“What's next?”)。注意，此方法为搬瓦工专属，仅供懒人和纯新手，你如果想在更好更快的其他家的VPS上面搭建Shadowsocks服务，你还是需要老老实实的学习下面的手工搭建方法，而且我强烈建议你如果有能力也有精力，最好掌握搭建的原理，对于你以后维护自己的VPS或者再学习搭建其他的翻墙服务都大有脾益。

## VPS的使用 ##


1. 注册并且购买之后，搬瓦工需要在“My Services”里进入“KiwiVM Control Panel”点击“Root password modification”来获得root密码，SSH端口在邮件或者控制面板里可看到，用户名是root；
2. 下载[SecureCRT](https://www.vandyke.com/download/securecrt/download.html)，用于在你的windows系统中远程登录你的VPS，具体用法请自行搜索，网上一大堆。
3. 登录成功之后，就可以对VPS进行相应的配置。

# Shadowsocks服务端搭建 #
这里，主要参考[Shadowsocks](https://shadowsocks.org/en/index.html)的官方教程来就好。
## 安装软件 ##
1. 更新软件源
```sh 
apt-get update
```
2. 安装python-pip
```sh
apt-get install python-pip
```
3. 安装Shadowsocks服务端
```sh
pip install shadowsocks
apt-get install python-pip
```
## 添加配置文件 ##


- 首先输入以下命令，进入vi编辑器，编辑文件
```sh
vi /etc/shadowsocks.json
```

- 然后，先按i键，进入vi的编辑模式，然后输入以下内容，注意my_server_ip和mypassword的替换，如果你想更改端口号，就修改8388成你想设置的端口号。
```sh
{
	"server":"my_server_ip",
	"server_port":8388,
	"local_address": "127.0.0.1",
	"local_port":1080,
	"password":"mypassword",
	"timeout":300,
	"method":"aes-256-cfb",
	"fast_open": false
}
```

- 完成之后，按下Esc键，之后再输入:wq退出vi编辑器并保存。

## 启动Shadowsocks服务端 ##
配置好了之后，就准备启动Shadowsocks服务端了，输入以下命令，启动Shadowsocks
```sh
sssserver -c /etc/shadowsocks.json -d start
```

## 添加开机启动 ##

最后要添加开机启动，这样，重启VPS后，Shadowsocks会自动运行，无需再进行开启。

- 打开/etc/rc.local文件
```sh
vi /etc/rc.local
```

- 在末尾添加如下代码
```sh
/usr/bin/python /usr/local/bin/ssserver -c /etc/shadowsocks.json -d start
```

# 客户端的搭建 #


1. 对于windows、android以及mac OS来说，都有专门的客户端，见[shadowsocks官网](https://shadowsocks.org/en/download/clients.html)即可。
2. 对于ios来说，就需要下载surge或者shadowrocket软件了，前者比较好用，但是价格现在648，也是醉醉地；后者比较便宜，只要6块钱。
