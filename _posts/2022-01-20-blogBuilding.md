---
title: blog搭建
tags: ruby
# article_header:
#   type: cover
#   image:
#     src: /assets/images/helloworld.jpg
---

<!-- write excerpt here -->
搭blog时的一些问题总结

<!--more-->

## macos 上本地环境搭建

基本上按照这个[知乎文章](https://zhuanlan.zhihu.com/p/350462079?ivk_sa=1024320u)可以完成，需要注意ruby版本尽量升到3以上，不然mac上安装编译过程中还会有其他问题

## gem 和 gemfile

ruby 的其中一个“程序”叫 rubygems ，简称 gem，而用来管理项目的 gem 的，叫 bundle ，他俩完全是不同的东西，相同的只是都可以管理gem。

Gemfile.lock为bundle生成的进一步细化的Gem版本控制文件，当需要对特定的依赖包进行版本限定时，不要直接修改Gemfile.lock，在Gemfile中对相应的包版本进行限定后gem update即可。