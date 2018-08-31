---
title: 搭建 Hexo 部署服务器
tags:
  - Hexo
  - Git
  - Nginx
date: 2018-08-31 13:33:46
---


## 前言

为了方便地对日常工作中遇到的问题进行分析、总结，我在本地使用 Hexo 博客框架以及 Wixo 主题搭建了一个 Wiki 进行记录和查询。然而由于本地的 Windows 机器经常会自动更新重启，导致 Wiki 挂掉，同时也不方便分享给别人，因此想尝试将其部署到远程服务器上，并且可以通过 hexo deploy 命令快捷地进行部署更新，实现类似 GitHub Pages 的效果。

## 系统环境

- 本地客户端：
    - Windows 10 Enterprise
    - Git 2.16.2
- 远程服务器：
    - Ubuntu 16.04.5 LTS
    - Nginx 1.10.3
    - Git 2.7.4

## 搭建 Nginx 服务器

这里我们使用 Nginx 服务器来为我们的 Wiki 提供静态站点托管服务，因此第一步先要在远程服务器上安装并启动 Nginx 服务器。

```bash
sudo apt-get install nginx
sudo nginx
```

可以通过 ps 命令查看 Nginx 进程的运行情况：

```bash
ps -aux | grep nginx
```

```bash
root      2815  0.0  0.0 116524  1384 ?        Ss   04:10   0:00 nginx: master process
www-data  2819  0.0  0.0 116876  3024 ?        S    04:10   0:00 nginx: worker process
www-data  2820  0.0  0.0 116876  3024 ?        S    04:10   0:00 nginx: worker process
penx      6969  0.0  0.0  12944   984 pts/0    S+   05:27   0:00 grep --color=auto nginx
```

通过 netstat 命令查看端口使用情况，Nginx 的默认端口号为 80：

```bash
sudo netstat -ntlp | grep 80
```

```bash
tcp        0      0 0.0.0.0:80        0.0.0.0:*         LISTEN      2815/nginx -g daemo
tcp6       0      0 :::80             :::*              LISTEN      2815/nginx -g daemo
```

最后通过 curl 命令访问默认首页，测试 Nginx 启动是否成功。如果无法正确访问，还可以通过查看 /var/log/nginx/access.log 和 /var/log/nginx/error.log 进行进一步的分析调试。

```bash
curl http://127.0.0.1/
```

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## 搭建 Git 服务器

接下来，我们要在远程服务器上搭建 git server，用于对 Wiki 进行版本控制。

首先我们创建一个独立的 git 用户，并设置密码：

```bash
sudo useradd git
sudo passwd git
```

为了避免每次提交都需要输入密码，我们将客户端的 ssh 密钥加入到服务器端 git 账户下的 authorized_keys 文件中，这里的客户端是指 Windows 系统下的 git 客户端，其默认使用的 ssh 密钥位于 ~/.ssh/id_rsa.pub 中。我们在服务器端先为 git 账户创建 authorized_keys 文件：

```bash
su git
mkdir ~/.ssh
touch ~/.ssh/authorized_keys
```

使用本地 git 客户端将 ssh 密钥添加到服务器的 authorized_keys 文件中：

```bash
cat ~/.ssh/id_rsa.pub | ssh git@123.45.67.89 "cat >> ~/.ssh/authorized_keys"
```

在服务器端修改 authorized_keys 文件权限，避免他人对 ssh 密钥进行改动:

```bash
chmod 700 ~/.ssh && chmod 400 ~/.ssh/authorized_keys
```

在服务器端创建名为 wiki 的 git 远程仓库：

```bash
git init --bare ~/wiki.git
```

为客户端 Hexo 安装 hexo-deployer-git 插件，并在 Wiki 的配置文件 _config.yml 中添加部署信息：

```yml
deploy:
  type: git
  repo: git@123.45.67.89:wiki.git
  branch: master
  message:
```

我们尝试将本地文件同步到远程仓库：

```bash
hexo generate
hexo deploy
```

结果并没有成功，出现了如下错误。这是由于 deploy 过程使用 ssh 协议访问远程服务器，如果远程服务器的公钥（public key）没有记录在本地的 ~/.ssh/known_hosts 文件中，就无法进行访问。

```bash
Error: Host key verification failed.
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

    at ChildProcess.<anonymous> (C:\Users\penx\Wiki\node_modules\hexo-util\lib\spawn.js:37:17)
    at emitTwo (events.js:126:13)
    at ChildProcess.emit (events.js:214:7)
    at ChildProcess.cp.emit (C:\Users\penx\Wiki\node_modules\cross-spawn\lib\enoent.js:40:29)
    at maybeClose (internal/child_process.js:925:16)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:209:5)
```

简单的解决方案是在 git 客户端下 ssh 一下远程服务器，这一操作会自动将远程主机的公钥记录在 ~/.ssh/known_hosts 中。

```bash
ssh git@123.45.67.89
```

另一种方案就是使用 ssh-keyscan 命令:

```bash
ssh-keyscan -H 123.45.67.89 >> ~/.ssh/known_hosts
```

再次执行 hexo deploy 命令，可以看到这次成功将本地代码推送到了远程仓库：

```bash
INFO  Deploying: git
INFO  Clearing .deploy_git folder...
INFO  Copying files from public folder...
INFO  Copying files from extend dirs...
On branch master
nothing to commit, working tree clean
Branch 'master' set up to track remote branch 'master' from 'git@123.45.67.89:wiki.git'.
To 123.45.67.89:wiki.git
 * [new branch]      HEAD -> master
INFO  Deploy done: git
```

## 配置自动部署

Hexo deploy 命令是将 .deploy_git 目录下的内容（静态网站文件）推送到远程仓库。然而由于 git 远程仓库中只存储了历史和元信息，并不维护工作目录，因此我们还需要建立另一个 git 仓库来从远程仓库同步网站内容，这里我们选择放在 /usr/share/nginx/html 下，也就是 Nginx 默认的目录：

```bash
ls -al ~/wiki.git

total 40
drwxrwxr-x 7 git git 4096 Aug 28 06:28 .
drwxr-xr-x 5 git git 4096 Aug 28 07:03 ..
drwxrwxr-x 2 git git 4096 Aug 28 06:28 branches
-rw-rw-r-- 1 git git   66 Aug 28 06:28 config
-rw-rw-r-- 1 git git   73 Aug 28 06:28 description
-rw-rw-r-- 1 git git   23 Aug 28 06:28 HEAD
drwxrwxr-x 2 git git 4096 Aug 28 07:03 hooks
drwxrwxr-x 2 git git 4096 Aug 28 06:28 info
drwxrwxr-x 4 git git 4096 Aug 28 06:28 objects
drwxrwxr-x 4 git git 4096 Aug 28 06:28 refs
```

我们在这一路径下建立一个 git 仓库，并指向本地的远程仓库 wiki.git：

```bash
cd /usr/share/nginx/html/
sudo mkdir wiki
cd wiki
git init
git remote add origin git@localhost:wiki.git
```

修改权限使得 git 账户可以修改网站内容：

```bash
sudo chown git:git /usr/share/nginx/html/wiki -R
```

我们为远程仓库添加 post-receive 的 git hook，用于在每次成功接收客户端推送后自动将内容同步到 Nginx 路径下：

```bash
cd ~/wiki.git/hooks/
vi post-receive
```

在文件中添加如下内容（注释掉的几行 echo 信息是为了 debug 使用）：

```bash
#!/bin/sh

unset $(git rev-parse --local-env-vars)
cd /usr/share/nginx/html/wiki
# echo PWD is $PWD
git fetch origin
# echo Complete remote fetch
git reset --hard origin/master
# echo Complete branch setting
```

为用户和所在的组添加 git hook 的执行权限，否则该文件无法被 git 执行：

```bash
sudo chmod ug+x post-receive
```

和本地 git 客户端同理，我们也需要将 git 用户的 ssh 密钥添加到 authorized_keys 文件中：

```bash
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

创建新的 Nginx 服务器配置文件 wiki.com 来代表我们的网站配置，这里我们将端口号设置为 8080：

```bash
sudo vi /etc/nginx/sites-available/wiki.com
```

```bash
server {
    listen 8080;
    root /usr/share/nginx/html/wiki;
    index index.html;
    server_name _;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

在 sites-enabled 目录中创建 wiki.com 的链接来让网站生效：

```bash
sudo ln -s /etc/nginx/sites-available/wiki.com /etc/nginx/sites-enabled/
```

重启 nginx 服务器：

```bash
sudo nginx -s reload
```

## 测试

在客户端清空 .deploy_git 目录，执行 hexo deploy 命令，还是遇到了问题：

```bash
remote: PWD is /usr/share/nginx/html/wiki
remote: git
remote: Host key verification failed.
remote: fatal: Could not read from remote repository.
remote:
remote: Please make sure you have the correct access rights
remote: and the repository exists.
remote: Complete remote fetch
remote: fatal: ambiguous argument 'origin/master': unknown revision or path not in the working tree.
remote: Use '--' to separate paths from revisions, like this:
remote: 'git <command> [<revision>...] -- [<file>...]'
remote: Complete branch setting
Branch 'master' set up to track remote branch 'master' from 'git@123.45.67.89:wiki.git'.
To 13.76.153.195:wiki.git
 + 556bdf3...88b1e06 HEAD -> master (forced update)
INFO  Deploy done: git
```

这里我们通过 echo 信息可以判断出是在 git fetch origin 这一步出了错，问题和前面一样，是由于远程服务器的 git 账户下的 know_hosts 文件中没有包含其自己的 ssh 密钥：

```bash
ssh git@localhost
```

再次测试，成功！访问 http://123.45.67.89:8080/ 就可以看到我们的 Wiki 内容了：

```bash
remote: PWD is /usr/share/nginx/html/wiki
remote: git
remote: From localhost:wiki
remote:  * [new branch]      master     -> origin/master
remote: Complete remote fetch
remote: HEAD is now at 95805a2 Site updated: 2018-08-28 15:13:36
remote: Complete branch setting
Branch 'master' set up to track remote branch 'master' from 'git@123.45.67.89:wiki.git'.
To 13.76.153.195:wiki.git
 + f7c6c5b...95805a2 HEAD -> master (forced update)
INFO  Deploy done: git
```

在本地对 Wiki 内容稍作修改，重新生成并部署，可以立刻在浏览器中观察到最新的改动。

## 参考

- [Searene: Set Up A Git Server To Deploy With Hexo](http://searene.me/2015/12/05/set-up-a-git-server-to-deploy-with-hexo/)
- [阮一峰: SSH原理与运用（一）：远程登录](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)