---
# Required fields
title: "Arch Linux Installation Guide"
date: 2025-08-15
# Optional fields
description: "从零安装Arch Linux"
draft: false
categories: ["Linux"]
tags: ["Arch Linux"]
mermaid: false
comments: true
---

Install MINIMAL Arch Linux from the very beginning...

<!--more-->

## Installation Medium

1. Download the latest `iso` file: [Arch Linux Downloads](https://archlinux.org/download/)

2. Download [Rufus](https://rufus.ie/en/).

3. Prepare an USB memory stick (> 8GB) and flash the iso file into it.

## Boot the Live Env

Enter BIOS and change the startup device to the USB you just made. Then reboot the computer.

## Internet and Timezone

Unblock all software and hardware locks.

```shell
rfkill unblock all
```

Use `iwctl` to connect to the internet.

```shell
iwctl
[iwd] station wlan0 scan  # Scan
[iwd] station wlan0 get-networks  # List all available networks
[iwd] station wlan0 connect SSID  # Connect to the WIFI
[iwd] exit
```

Set timezone

```shell
timedatectl set-timezone Asia/Shanghai
```

## Partition the Disk

Check current disk partition.

```shell
lsblk
```

Find the main disk of the computer. Normally the name is `nvme0n1`. Then partition this disk.

```shell
cfdisk /dev/nvme0n1
```

My partition layout:

| Mount Point | Type | Size |
|:---:|:----:|:---:|
| /boot | EFI System | 1G |
| [SWAP] | Linux Swap | 8G |
| / | Linux filesystem | All left spaces |

Format the partitions.

```shell
mkfs.fat -F 32 /dev/nvme0n1p1
mkswap /dev/nvme0n1p2
mkfs.btrfs -f /dev/nvme0n1p3
```

Mount partitions.

```shell
mount /dev/nvme0n1p3 /mnt
mount --mkdir /dev/nvme0n1p1 /mnt/boot
swapon /dev/nvme0n1p2
```

## Installation

Get mirrorlist. For me, it's chinese.

```shell
curl -L 'https://archlinux.org/mirrorlist/?country=CN&protocol=https' -o /etc/pacman.d/mirrorlist
```

Then uncomment one line in `/et/pacman.d/mirrorlist`

Install basic packages. Mine is amd cpu, so I install the `amd-ucode`. If yours is intel cpu, you should install `intel-ucode`.

```shell
pacstrap -K /mnt base base-devel linux linux-firmware amd-ucode vi neovim fish grub efibootmgr networkmanager sddm cliphist brightnessctl pipewire pipewire-audio pipewire-pulse pipewire-jack pipewire-alsa bluez bluez-utils blueman pavucontrol
```

## Configure System

Generate `fatab` file.

```shell
genfstab -U /mnt > /mnt/etc/fstab
cat /mnt/etc/fstab  # Check
```

Enter the new system.

```shell
arch-chroot /mnt
```

Set timezone and localization.

```shell
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

Edit `/etc/locale.gen`, uncomment the line of `en_US.UTF-8 UTF-8`.

```shell
loclae-gen
```

Create `/etc/locale.conf` file and add the following.

```shell
LANG=en_US.UTF-8
```

Add the hostname to `/etc/hostname`.

## GRUB Configuration

We already installed `grub` and `efibootmgr` packages before. So now we need to install grub and genarate the configuration file.

```shell
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

Set password for root.

```shell
passwd
```

Exit and reboot system. Login as root.

Edit `/etc/pacman.conf` and uncomment the `[multilib]` part. At the bottom of the file, add the following text.

```shell
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```

Then

```shell
pacman -S archlinuxcn-keyring
pacman -S yay
```

Start services.

```shell
systemctl enable --now NetworkManager
systemctl enable --now sddm.service
systemctl enable --now bluetooth.service
```

Install the graphic driver. For me, it's amd gpu.

```shell
pacman -S mesa lib32-mesa xf86-video-amdgpu vulkan-radeon lib32-vulkan-radeon
```

Add a normal user.

```shell
useradd -G wheel -m shane
passwd shane
visudo  # uncomment #%wheel ALL=(ALL:ALL) ALL
su - shane  # switch to the normal user
```

## Next Step

So far you have installed the basic Arch Linux system. For next, you can choose your prefered window manager or the desktop environment.
