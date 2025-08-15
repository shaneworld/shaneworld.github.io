---
# Required fields
title: "Arch Linux 安装指南"
date: 2025-08-15
# Optional fields
description: "从零安装Arch Linux"
draft: false
categories: ["Linux"]
tags: ["Arch Linux"]
mermaid: false
comments: true
---

从零安装 minimal Arch Linux

<!--more-->

## 安装媒介

1. 从 [Arch Linux Downloads](https://archlinux.org/download/) 下载最新的 ISO 文件

2. 下载 [Rufus](https://rufus.ie/en/) 制作系统盘工具

3. 准备一个空间 > 8GB 的闲置 U 盘作为系统盘

4. 进入 BIOS 并设置从 U 盘启动。

## 网络和时区

解锁所有硬件和软件锁：

```shell
rfkill unblock all
```

使用`iwctl` 工具联网：

```shell
iwctl
[iwd] station wlan0 scan  # 扫描
[iwd] station wlan0 get-networks  # 列出所有可用网络
[iwd] station wlan0 connect SSID  # 联网
[iwd] exit
```

设置时区：

```shell
timedatectl set-timezone Asia/Shanghai
```

## 分区

首先检查目前的磁盘分区情况：

```shell
lsblk
```

找到电脑的主磁盘，通常名字是 `nvme0n1`，开始分区：

```shell
cfdisk /dev/nvme0n1
```

我的分区如下：

| 挂载点 | 分区类型 | 大小 |
|:---:|:----:|:---:|
| /boot | EFI System | 1G |
| [SWAP] | Linux Swap | 8G |
| / | Linux filesystem | All left spaces |

分区完后需要格式化分区：

```shell
mkfs.fat -F 32 /dev/nvme0n1p1
mkswap /dev/nvme0n1p2
mkfs.btrfs -f /dev/nvme0n1p3
```

挂载分区：

```shell
mount /dev/nvme0n1p3 /mnt
mount --mkdir /dev/nvme0n1p1 /mnt/boot
swapon /dev/nvme0n1p2
```

## 安装系统

首先需要通过下载 mirrorlist 更改下载源解决网络问题：

```shell
curl -L 'https://archlinux.org/mirrorlist/?country=CN&protocol=https' -o /etc/pacman.d/mirrorlist
```

取消注释 `/etc/pacman.d/mirrorlist` 中的内容。

安装基础组件。我目前的电脑是 AMD GPU，因此我需要安装 `amd-ucode`。如果你是 Intel 的 CPU，你应该安装 `intel-ucode`：

```shell
pacstrap -K /mnt base base-devel linux linux-firmware amd-ucode vi neovim fish grub efibootmgr networkmanager sddm cliphist brightnessctl pipewire pipewire-audio pipewire-pulse pipewire-jack pipewire-alsa bluez bluez-utils blueman pavucontrol
```

## 初始化配置系统

创建 `fatab` 文件。

```shell
genfstab -U /mnt > /mnt/etc/fstab
cat /mnt/etc/fstab  # 用于检查
```

从 live 环境进入系统：

```shell
arch-chroot /mnt
```

时区和本地化设置：

```shell
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

编辑`/etc/locale.gen`，取消注释 `en_US.UTF-8 UTF-8`：

```shell
loclae-gen
```

创建 `/etc/locale.conf` 并添加：

```shell
LANG=en_US.UTF-8
```

在 `/etc/hostname` 中添加 hostname。

## GRUB 配置

在基础组件安装时我们已经安装了 `grub` 和 `efibootmgr`，现在需要配置 GRUB：

```shell
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

设置 root 密码：

```shell
passwd
```

重启系统并以 root 身份登陆。编辑 `/etc/pacman.conf` 然后取消注释 `[multilib]` 的部分，并在文件末尾添加：

```shell
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```

安装 yay：

```shell
pacman -S archlinuxcn-keyring
pacman -S yay
```

启动一些基础服务：

```shell
systemctl enable --now NetworkManager
systemctl enable --now sddm.service
systemctl enable --now bluetooth.service
```

安装显卡驱动：

```shell
pacman -S mesa lib32-mesa xf86-video-amdgpu vulkan-radeon lib32-vulkan-radeon
```

添加一个普通用户：

```shell
useradd -G wheel -m shane
passwd shane
visudo  # 取消注释 #%wheel ALL=(ALL:ALL) ALL
su - shane  # 切换到普通用户
```

## 后续

至此，我们已经成功从零安装了一个最小最简洁的 Arch Linux，后续可以根据喜好和需求选择安装窗口管理器还是桌面环境。

> 2025 年 8 月 15 日追记 \
> 对我来说，自从接触了窗口管理器便难以离开它，我怀念全键盘操作的快乐以及配置工具的满足感。时隔三个月没有使用 Linux 和窗口管理器了，我准备重拾它们。敬请期待后续关于 Linux 的文章，我们一起探索 Linux 世界的乐趣！！