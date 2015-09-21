title: 使用 liftoff 创建 iOS 项目
date: 2015-09-20 09:31:55
tags: Liftoff
categories: 开发笔记
description: 用上脚手架，和体力劳动说拜拜。
---

最近新起了3个新项目，每次配置项目都是重复的体力劳动，刚好前段时间看到一个不错的脚手架工具：[liftoff](https://github.com/thoughtbot/liftoff)，试上一发看下效果如何。

## 简介

## 定制

如果需要定制自己的模板和配置也非常容易，我们只需要将原始配置拷贝到用户目录即可。

首先查看一下 liftoff 的路径（当前版本号为1.5.0）：

    ➜  ~  which liftoff
    /usr/local/bin/liftoff

    ➜  ~  ls -l /usr/local/bin/liftoff
    /usr/local/bin/liftoff -> ../Cellar/liftoff/1.5.0/bin/liftoff

可以看到，liftoff 的真实地址是：`/usr/local/Cellar/liftoff/1.5.0/bin/liftoff` 。

将 liftoffrc 和 template 拷贝到用户目录下：

    cd /usr/local/Cellar/liftoff/1.5.0
    cp defaults/liftoffrc ~/.liftoffrc
    mkdir ~/.liftoff
    cp -r templates ~/.liftoff

这样配置文件就部署好了，后面都会加载用户目录下的 liftoffrc 和 templates 。