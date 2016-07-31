---
title: 在VPS上搭建Hexo博客并使用Git Hooks更新
date: 2016-06-28
tags: 建站
category: 网站建设
---
## 安装

### 安装前提

安装Hexo相当简单。然而在安装前，必须检查电脑中是否已安装下列应用程序：

- [Node.js](https://nodejs.org/en/)
- [Git](https://git-scm.com/)


如果已经安装，接下来只需要使用npm即可完成Hexo的安装
```sh
$ npm install -g hexo-cli
```
如果电脑中尚未安装，请参考[hexo官方链接](https://hexo.io/docs/index.html)

## 本地建站

### 创建网站目录
安装完Hexo完成后，请执行下列命令，Hexo将会在指定的文件夹下新建所需要的文件

```sh
$ hexo init <folder>
$ cd <folder>
$ npm install
```
执行以下命令：

```sh
$ hexo d -fg
$ hexo serve
```
打开[http://localhost:4000](http://localhost:4000)即可看到你的站点，只不过还没有发布到网络。

### 本地Git配置

通过cd命令进行文件夹，执行

```sh
$ git init
$ git add .
$ git commit -m "Initial commit"
```

## VPS

此处为ubuntu在root用户下的操作。执行以下命令，注意**example.com**的替换：

```sh
$ apt-get update && apt-get upgrade -y
$ apt-get install git-core -y
$ curl -sL https://deb.nodesource.com/setup | bash
$ apt-get install nodejs -y
$ apt-get install nginx -y
$ cd /etc/nginx/sites-available
$ rm -rf default
$ touch example.com
$ vi example.com
```

在其中输入如下内容并保存，注意**IP**和**example.com**的替换

```sh
server {
listen IP: 80;
	root /var/www/example.com/public;
	server_name example.com;
	access_log /var/log/nginx/example_access.log;
	error_log /var/log/nginx/example_error.log;
	location ~* ^.+\.(ico|gif|jpg|jpeg|png)$ {
		root /var/www/example.com/public;
		access_log off;
		expires 1d;
	}
	location ~* ^.+\.(css|js|txt|xml|swf|wav)$ {
		root /var/www/example.com/public;
		access_log off;
		expires 10m;
	}
	location / {
		root /var/www/example.com/public;
		if (-f $request_filename) {
			rewrite ^/(.*)$  /$1 break;
		}
	}
}
```

之后执行

```sh
$ ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
$ cd ~
$ mkdir repos && cd repos
$ mkdir example.com.git && cd example.com.git
$ git init --bare
$ cd hooks
$ touch post-receive
$ vi post-receive
```

在其中输入以下内容并保存，同样注意前四行的替换

```sh
#!/bin/bash -l
GIT_REPO=$HOME/repos/example.com.git
TMP_GIT_CLONE=$HOME/tmp/git/example.com
PUBLIC_WWW=/var/www/example.com
rm -rf ${TMP_GIT_CLONE}
git clone $GIT_REPO $TMP_GIT_CLONE
rm -rf ${PUBLIC_WWW}/*
cp -rf ${TMP_GIT_CLONE}/* ${PUBLIC_WWW}
cd ~
cd ${PUBLIC_WWW}
hexo d -fg
cd ~
exit
```

最后执行

```sh
$ chmod +x post-receive
$ cd ~
$ service nginx restart
```

## 本地推送

以下操作需要先***cd***到网站目录，注意替换
```sh
$ git remote add example root@IP:repos/example.com.git
$ git push example master
```
## 关于更新博客

使用一款MarkDown编辑器写博客。写完后将文件以***.md的格式保存在本地:[网站目录]/source/_posts***中。
每篇博客都有固定的参数必须填写，参数如下，注意每个参数的：后都有一个空格。
```sh
title: title
date: yyyy-mm-dd
categories: category  
tags: tag
#多标签请这样写：  
#tags: [tag1,tag2,tag3]
#或者这样写： 
#tags:
#- tag1 
#- tag2  
#- tag3  
---  
正文  
```

***cd***网站目录后执行

```sh
$ hexo d -fg
$ git add .
$ git commit -m "Post A New Blog"
$ git push example master
```
