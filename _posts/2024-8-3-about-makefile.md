---
title: 玩转 Makefile（更新中）
author: shane
date: 2024-08-03
description: 关于 Makefile 的基础语法
categories: [Linux]
tags: [Makefile]
comments: true
toc: true
---

## Makefile 简介

> 本文章基于 [Learn Makefiles with the tastiest examples](https://makefiletutorial.com/)
{: .prompt-tip }

Makefiles 是管理项目的配置文件，有 2 个主要作用：

- 组织工程文件，编译成复杂的程序
- 安装及卸载程序

当执行 `make` 命令的时候，如果当前目录下存在 Makefile，那么 `make` 命令会根据 Makefile 进行编译、连接、安装、卸载等一系列操作。简单来说，**Makefile 是配方，而 make 是厨师**。

Makefiles 帮助我们决定项目中哪些部分需要被重新编译。Makefile 最初是为 C 语言等编译型语言设计的，但他的灵活性使其可以管理各种项目。

## 一个简单的例子

安装 `make` 后，我们新建一个名为 `Makefile` 的文件，写入以下内容：

```makefile
hello:
	echo "Hello, World"
```

> **Note:** Makefiles 中的索引一定要使用 `TABs` 而不是空格，否则 make 会失败。
{: .prompt-warning }

## Makefile 语法

Makefile 的语法如下：

```makefile
targets: prerequisites (dependencies)
	command
	command
	command
```

- 目标是用空格隔开的文件名称。
- `command` 是生成 `targets` 所需要执行的一系列步骤。
- `dependencies` 也是用空格分开的文件列表，`command` 中涉及到的文件必须在这部分中罗列出来。

## Make 的实质

我们创建一个文件 `blah.c`：

```c
// blah.c
int main() {
    return 0;
}
```

然后在 Makefile 中写入以下内容：

```makefile
blah:
	cc blah.c -o blah
```

> 其中 `cc` 是 C 编译器（C Compiler），在很多操作系统中，`cc` 是指向系统默认 C 编译器的符号链接或别名，通常是 GCC。

