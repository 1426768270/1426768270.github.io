---
title: Hello World
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

<!--more-->

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)

### 博客的搭建与备份

https://www.jianshu.com/p/9b87aae19093

## 备份博客

由于博客是在本地生成的，如果更换电脑，那我们是不是就不能用这个博客了？方法总比问题多，我们可以利用`github`来备份博客的文件和数据。

https://github.com/coneycode/hexo-git-backup

1.安装

```
npm install hexo-git-backup --save
```

2.在博客配置文件 _config.yml 中添加:

```
backup: 
	type: git 
	theme: next,... 
	message: update xxx 
	repository: 
		git@github.com:TRHX/TRHX.github.io.git,backup
```

配置中可以设置提交时的信息(message); 

可以指定一起备份的主题(支持多个, 逗号分隔);

 repository 也可以指定多个地方进行同步备份;

 网上说的比较多的是将备份放到博客项目的一个分支里, 即将上面的 branchName 设置成分支名称, git 路径与博客项目 github用户名.github.io 一致;

3.在命令行中执行

```java
hexo backup
```

如果原先是通过手动方式进行的备份, 即没有使用插件, 此时再用插件备份是会出错的. 解决方法是将博客目录下的 .git 目录删除, 再执行 hexo backup 命令;

原文

 https://memento.net.cn/post/b96cbb80.html