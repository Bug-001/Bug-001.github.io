---
layout: post
title: 【新站第一篇】孩子们，我回来了（附Jekyll配置教程）
date: 2024-06-07
tags: [random]
mathjax: true
author: VOID*
---

孩子们，我回来了！花了一点时间，用jekyll建了全新的个人博客。

# 闲聊

新站建立的第一篇文章，总得聊点来龙去脉吧？

这是旧站：[VOIDs技术驿站](https://void-star.icu/)。为啥不想用了呢？当年大三的时候，对于技术一腔热血，觉得搭一个自己的网站是超酷的事情。所以我去腾讯云买了服务器，从阿里云买了域名，用宝塔面板和Wordpress开始搞事，搞了一堆工信部备案的手续，挂上了HTTPS，又踩了无数技术上的坑，总算搞起来了。纵然这很有趣，但真做过才会明白其中的代价：

- 服务器和域名都要收钱（虽然一年下来只是几顿饭钱）。
- 尽管Wordpress可定制性极强，允许自己设计动态的网页交互，但最后发现那并不是我想要的。我只想要一个安静的地方写字，jekyll这种静态编译生成的网页完全符合我的需求。
- 服务器有公网IP，已经被各种病毒渗透烂了，光是看日志就能看到每天四五万次的login attempt。甚至还收到过勒索威胁邮件。跟黑客斗智斗勇非常有趣，可是我实在没有时间。留着黑客在我机器上肆意妄为是很危险的，虽然我数据不值钱，但要是损毁就糟糕了。不如把安全交给GitHub。
- 自建网站如果想要在搜索引擎上排名比较好，还得认真搞SEO。
- HTTPS证书还得定期更新，我总是记不得那些命令。
- 国内云服务的监管问题，无需多言。

总而言之我对各种节外生枝的事情感到很痛苦，于是决定投奔Github Pages。唯一割舍不下的就是域名void-star.icu，很独特，很好记，很好看，很有意义。不过人在这世上行走总要放弃一些东西的嘛，复杂的事情都很难有全局最优解。

在新站写点什么呢？

- 尽量把旧站的东西搬过来，那边写了很多操作系统和计算机网络的内容，虽然我之后可能不太容易搞这些了，但总归是珍贵的记忆、丰富的知识库，未来也肯定能用得到。
- 数学。如果你是计算机专业的，感觉数学学了没用，多半是没有真正地理解。这半年我体会到了线性代数无穷的力量，有空会分享一些自己觉得有趣的东西。学习笔记将会整理在我的notion里，作为博客的外链。
- 技术。主要包括编程和工具链、system知识、AI进展等等。数学得以让我从不同的角度感受AI之力，相关内容也会分享。
- 思考。这是最不能放弃的东西，随着AGI的发展，深刻的思考变得愈发可贵。

# 建站教程

Jekyll是一个静态网站生成器，它用Ruby实现，会把你写的markdown文件编译成html文件。撰写者所需要做的所有事情，就是写markdown文件，然后push到GitHub上，GitHub会自动帮你编译成网页。

首先，建立一个和自己GitHub用户名相同的repo，这就是你的GitHub Page。然后，把[Beautiful Jekyll](https://github.com/daattali/beautiful-jekyll) clone到你的这个repo中。只要跟着README你就可以写文章了。

但是作为一个程序员，肯定还是希望对自己的网站有全面的控制权，而且让GitHub Action来构建网站比较慢，我希望能在本地实时看到效果。这就得搞到自己本地，安装ruby环境，然后安装一切所需要的依赖，这些教程里有写。在这里，我说说自己遇到的坑：

## `bundler install`权限问题

执行`bundler install`时，可能会提示权限不够，无法安装到`/usr/lib`下面。这时候我选择安装到`thirdparty`目录下：

```shell
bundle config set path 'thirdparty/bundle'
```

然后在`_config.yml`中添加路径忽略：

```yaml
exclude:
  - thirdparty/
```

确保在`.gitignore`中添加`thirdparty`。然后你就可以愉快地安装所有依赖了。

## Jekyll服务本地部署

在命令行执行`jekyll serve --incremental --livereload`，然后访问`http://127.0.0.1:4000`，就可以看到你的网站了。`--incremental`参数可以让jekyll只编译修改过的文件，加快编译速度。`--livereload`参数可以让你在修改文件后自动刷新网页。你就可以一屏写markdown，一屏看效果。

但美中不足的是，每次写博客之前，还要先启动这个命令才行吗？Nonono，我们完全可以用systemd来管理这个服务。打开`/etc/systemd/system/jekyll.service`，写入：

```shell
[Unit]
Description=Jekyll Serve in Incremental Mode
After=network.target

[Service]
Type=simple
User=sht
WorkingDirectory=/home/sht/Desktop/Project/Bug-001.github.io
ExecStart=/usr/bin/jekyll serve --incremental --livereload
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

然后更新systemd配置，启动jekyll服务，就大功告成了：

```shell
sudo systemctl daemon-reload
sudo systemctl enable jekyll.service
sudo systemctl start jekyll.service
```

可以通过下面的命令检查服务的输出：

```shell
journalctl -u jekyll.service
```

## 本地图片

你肯定会希望在文章中插入图片，还能自动保存到我们想要的目录下。VSCode的markdown插件可以实现这一点。打开`settings.json`，添加：

```json
"markdown.copyFiles.destination": {
    "/**/*": "${documentWorkspaceFolder}/assets/img/${documentBaseName}/"
}
```

这样markdown中插入的图片，就会自动保存到`assets/img/文章名/`目录下。这种方式对于复制粘贴插入的图片特别友好。

## 添加目录

从这个repo下载toc.html，保存到`_includes`目录下：[jekyll-toc](https://github.com/allejo/jekyll-toc)。最简单的方式就是：

```shell
wget https://github.com/allejo/jekyll-toc/releases/download/v1.2.1/toc.html -O _includes/toc.html
```

然后在`_layouts/post.html`中，\{\{ content \}\}的前面添加\{\% include toc.html html=content \%\}，就可以在文章中自动生成目录了。（把反斜杠去掉）