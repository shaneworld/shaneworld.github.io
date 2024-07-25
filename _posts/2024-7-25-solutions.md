---
title: 专治各种疑难杂症
author: shane
date: 2024-07-25
description: 解决 Linux 下遇到的各种小问题
categories: [Linux]
tags: [archlinux]
comments: false
toc: true
---

## Arch Linux 中的 emoji 显示问题

刚刚安装好 Arch Linux 后发现很多emoji都显示乱码。

### 安装 emoji 字体

```bash
sudo pacman -S noto-fonts-emoji
```

### 字体配置

在 `/etd/fonts` 目录下新建文件 `local.conf`，写入以下内容：

```conf
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
 <alias>
   <family>sans-serif</family>
   <prefer>
     <family>Noto Sans</family>
     <family>Noto Color Emoji</family>
     <family>Noto Emoji</family>
     <family>DejaVu Sans</family>
   </prefer> 
 </alias>
 
 <alias>
   <family>serif</family>
   <prefer>
     <family>Noto Serif</family>
     <family>Noto Color Emoji</family>
     <family>Noto Emoji</family>
     <family>DejaVu Serif</family>
   </prefer>
 </alias>
 
 <alias>
  <family>monospace</family>
  <prefer>
    <family>Noto Mono</family>
    <family>Noto Color Emoji</family>
    <family>Noto Emoji</family>
    <family>DejaVu Sans Mono</family>
   </prefer>
 </alias>
</fontconfig>
```

也可以在其中 `monospace` 的配置板块针对系统全局的 `monospace` 字体进行配置。执行 `fc-match monospace` 查看系统的 `monospace` 字体链接到哪个字体。

### 更新字体缓存

执行 `fc-cache -fv` 以更新字体缓存。

## Arch Linux 安装 yay 包管理器

由于国内的网络环境无法使用克隆编译的方法安装 yay 包管理器。

### 编辑 `pacman` 配置文件

`sudo vim /etc/pacman.conf` 编辑包管理器配置文件，在文件末尾添加以下内容：

```conf
[archlinuxcn]
SigLevel = Optional TrustAll
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```

上例中添加了清华镜像源，也可以自行更改其他镜像源。

### 安装 yay

执行 `sudo pacman -S yay` 安装 yay 包管理器。

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

## gradle 中增加键盘输入支持

`gradle` 默认不支持接收键盘输入，解决方法在[这里](https://stackoverflow.com/questions/13172137/console-application-with-java-and-gradle)

- 在 `build.gradle` (Groovy syntax) 中添加

```gradle
run {
    standardInput = System.in
}
```

- 或在 `build.gradle.kts` (Kotlin DSL syntax) 中添加

```gradle
tasks.named<JavaExec>("run") {
    standardInput = System.`in`
}
```

## SDKMAN 对 fish shell 的兼容问题

使用 `sdk` 安装 java 后，在 fish shell 里仍然无法识别 java 版本。解决方法在[这里](https://github.com/sdkman/sdkman-cli/issues/671#issuecomment-1130004319)

在 `~/.config/fish/config.fish` 中添加：

```fish
function sdk
    bash -c "source '$HOME/.sdkman/bin/sdkman-init.sh'; sdk $argv[1..]"
end
```

然后将安装的 java 版本添加到 `PATH`：

```bash
fish_add_path (find ~/.sdkman/candidates/*/current/bin -maxdepth 0)
```

## Arch Linux 开机自动启动蓝牙

每次开机发现蓝牙都默认关闭。使用 `rfkill list` 发现 bluetooth 被 `software blocked` 了。

解决方法：

```bash
sudo nvim /etc/systemd/system/bluetooth-unblock.service
```

添加以下内容：

```service
[Unit]
Description=Unblock Bluetooth

[Service]
Type=oneshot
ExecStart=/usr/bin/rfkill unblock bluetooth

[Install]
WantedBy=multi-user.target
```

其中，`ExecStart` 中的 `rfkill` 位置使用 `which rfkill` 命令查看。

然后启动服务

```bash
sudo systemctl enable bluetooth-unblock.service
```
