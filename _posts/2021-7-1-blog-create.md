---
layout: post
title:  "创建一个属于自己的博客"
categories: Blog
tags:  Blog Markdown
author: liubo
typora-root-url: ..
---

* content
{:toc}


## 前言
很多人可能都有一种想法，写点东西记录下学习或生活，好记忆不如烂笔头，现在我们用博客来开启文字记忆。

需要使用到的资源有：[GiteePages](https://gitee.com/help/articles/4136)，[jekyll](http://jekyllcn.com/docs/)，[Theme](https://github.com/xudailong/xudailong.github.io)，主题是一个大牛开源的，只需要根据他的[中文文档](https://github.com/liubo11/xudailong.github.io/blob/master/README-zh-cn.md)配置，基本可以完成搭建。

## 遇到的问题
1. 不蒜子的域名过期
根据[不蒜子网站](http://ibruce.info/2015/04/04/busuanzi/)说明更改即可。

2. 百度统计
需要在[百度统计官网](https://tongji.baidu.com/web/welcome/products)注册，更改_config.yml文件的配置，需要注意的是若网站1个月没有流量，可能失效。

3. 文档撰写
文档存放在_posts目录，使用Markdown语言编写，可参考[RUNOOB](https://www.runoob.com/markdown/md-tutorial.html)的教程。

4. 文档摘要
在`_config.yml`中使用excerpt_separator来标记的，在本项目中使用的是四个连续空行

## 参考资料

[MarkDown编辑器](https://pandao.github.io/editor.md/)