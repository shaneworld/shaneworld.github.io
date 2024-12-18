---
title: 疑难杂症
date: 2024-07-25
description: 解决 Linux 下遇到的各种小问题
categories: [linux]
tags: [linux, archlinux]
comments: true
toc: true
---

## Arch Linux 安装 yay 包管理器

由于国内的网络环境无法使用克隆编译的方法安装 yay 包管理器。

```shell
sudo vim /etc/pacman.conf
```

编辑包管理器配置文件，在文件末尾添加以下内容：

```conf
[archlinuxcn]
SigLevel = Optional TrustAll
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```

上例中添加了清华镜像源，也可以自行更改其他镜像源。

```shell
sudo pacman -S yay
```

安装 yay 包管理器。

## Arch Linux 开机自动启动蓝牙

每次开机发现蓝牙都默认关闭。使用 `rfkill list` 发现 bluetooth 被 `software blocked` 了。

```bash
sudo nvim /etc/systemd/system/bluetooth-unblock.service
```

添加

```
[Unit]
Description=Unblock Bluetooth

[Service]
Type=oneshot
ExecStart=/usr/bin/rfkill unblock bluetooth

[Install]
WantedBy=multi-user.target
```

其中，`ExecStart` 中的 `rfkill` 位置使用 `which rfkill` 命令查看。

启动服务

```bash
sudo systemctl enable bluetooth-unblock.service
```

## Pyright 无法导入引用的解决办法

在 neovim 使用 pyright 的时候会出现 `Pyright: Import "xxx" could not be resolved` 的报错。

解决方法参考此 [issue](https://github.com/microsoft/pyright/issues/3378#issuecomment-1417368768)，在项目根目录中新建 `pyrightconfig.json` 文件新增以下内容：

```json
{
    "include": [
         "core",
         "api"
    ]
}
```

其中 `core` 和 `api` 是需要导入的文件或者文件夹。
