---
title: 使用 jekyll 搭建自己的博客网站
author: shane
date: 2024-07-25
description: Linux 下使用 jekyll-theme-chirpy 建站
categories: [Blog]
tags: [archlinux, blog, jekyll]
comments: true
toc: true
---

最近发现了一个非常好看的博客主题 [jekyll-theme-chirpy](https://github.com/cotes2020/jekyll-theme-chirpy)，基于 [jekyll](https://jekyllrb.com/) 搭建，瞬间被吸引，果断放弃 [Hugo](https://gohugo.io/)。记录一下我的建站过程。

> 以下建站过程基于 Arch Linux。
{: .prompt-tip }

## 创建新站点

官方 [wiki](https://chirpy.cotes.page/posts/getting-started/) 提供了两种方法：

- 使用 [chirpy-starter](https://github.com/cotes2020/chirpy-starter) 模板：易于升级。
- GitHub Fork：自定义化程度高，但是升级困难。

这里采用第一种方式。登录 GitHub，打开 [Chirpy Starter](https://github.com/cotes2020/chirpy-starter)，单击按钮 `Use this template` > `Create a new repository`，然后将新存储库命名为 `USERNAME.github.io`，其中 `USERNAME` 代表 GitHub 用户名。然后将新创建的仓库克隆到本地。

> 仓库名称一定要按照这样的格式，只有该名称可以触发 Github 自动部署，也是你的博客域名。
{: .prompt-warning }

## 配置环境

`jekyll` 是一个 `Ruby Gem` 组件，使用 Ruby 开发，因此需要先搭建 Ruby 环境。

`Ruby` 是一种纯粹的面向对象编程语言，语法简单，可扩展性强。在 Arch Linux 中，我推荐使用 [rbenv](https://github.com/rbenv/rbenv) 来对其进行版本控制。使用 `yay` 安装：

```shell
yay -S rbenv
```

如果使用 `fish shell` 则需要安装 [rbenv-fish](https://github.com/rbenv/fish-rbenv) 插件。

由于 `rbenv install` 命令并不随 rbenv 一起提供，而是由 ruby-build 插件提供，因此需要安装 [ruby-build](https://github.com/rbenv/ruby-build)。

```shell
yay -S rb-build
```

使用 rbenv 安装指定版本的 Ruby

```shell
# 列出最新稳定版
rbenv install -l

# 列出所有本地已安装版本
rbenv install -L

# 安装指定的 Ruby 版本
rbenv install 3.3.4
```

> 该主题需要安装 > 3.1 版本的 Ruby
{: .prompt-warning }

以下是 rbenv 管理系统 Ruby 版本的基本命令：

```shell
# 设置系统 Ruby 版本
rbenv global 3.3.4

# 设置本地文件夹的 Ruby 版本
rbenv local 3.3.4
```

接着进入之前克隆到本地的博客文件夹，运行 `rbenv local 3.3.4` 设置本地 Ruby 版本，然后执行 `gem install bundler` 安装 `gem`。`RubyGems` 是 Ruby 的一个包管理器，提供了分发 Ruby 程序和库的标准格式“gem”，旨在方便地管理gem安装的工具，以及用于分发gem的服务器。这类似于 Python 的 pip 和 Arch Linux 中的 pacman 包管理器。

然后执行

```shell
bundle
```

## 修改配置文件

默认的模板中有很多需要修改的地方。主要配置都在 `_config.yml` 中，修改项包括但不限于：

- `url` 
- `avatar`
- `timezone`
- `lang`

修改方法在配置文件的注释中写的很清楚，自行阅读。

## 写文章

文章是位于 `_post` 文件夹下的 `markdown` 文件，关于文章的命名格式、[Front matter](https://jekyllrb.com/docs/front-matter/) 以及特殊格式的书写详见[官方 wiki](https://chirpy.cotes.page/posts/write-a-new-post/) 以及 [Jekyll 官方 wiki](https://jekyllrb.com/docs/)。

## 本地启动站点

```shell
bundle exec jekyll s
```

## 更新站点

如果写了新文章想要发表，只需要使用 git 将本地的更改提交到 GitHub，GitHub Actions 会自动部署（[Auto deploy](https://docs.github.com/en/actions/deployment/about-deployments/deploying-with-github-actions)）。站点域名为 `https://USERNAME.github.io`。
