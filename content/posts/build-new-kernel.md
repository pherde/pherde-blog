---
title: "Build a new Linux Kernel"
date: 2023-06-27
description: How to build, and install a new linux kernel.
draft: false
tags: [linux]
---

## How to build and install a new linux kernel from source

Remember, this how to works in a debian-like systems. It's necessary some adjustments to use in another system.

1. Install dependencies:

```shell
$ sudo apt -y -q install bc flex bison build-essential git libncurses-dev libssl-dev libelf-dev wget xz-utils
```

2. Go to the kernel.org, choose the repository, and clone it. For exemple, if you want the new stable:

```shell
$ git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git --depth=1
```

Why --depth=1? This way just the last commit will be cloned. This is a good thing if you want to spend less space in disk (a completed cloned linux repo has more than 8GB). 

3. Now it's necessary to create a valid config file to build, the easy way is to copy a valid old .config inside the */boot* folder. First enter in the cloned folder and copy the config file from the default kernel running in the system:

```shell
cd linux
cp -v /boot/config-$(uname -r) .config
```

4. To avoid some build issues, run the command:

```shell
$ make oldconfig
```

*make oldconfig* is a command used during the linux kernel compilation process to upgrade an existing kernel configurations to a new kernel version. Probably several questions will be asked after run this command, just press *Enter* to get through them all.

If it's necessary more configurations, just run: `make menuconfig` do the adjusts and exit.

5. Disable the security certificates (basicaly if you are in Ubuntu):

```shell
$ scripts/config --disable SYSTEM_TRUSTED_KEYS
$ scripts/config --disable SYSTEM_REVOCATION_KEYS
```

6. Build the kernel:

```shell
$ make -j$(nproc)
```

7. Install the modules:

```shell
$ sudo make modules_install
```

8. Install the new kernel:

```shell
$ sudo make install
```

9. Reboot the system and choose the new kernel in grub if necessary:

```shell
$ sudo shutdown -r now
```

## How to unistall the new kernel

Just delete these files:

```shell
$ sudo rm /boot/vmlinuz-[TARGET]
$ sudo rm /boot/initrd-[TARGET]
$ sudo rm /boot/System-map-[TARGET]
$ sudo rm /boot/config-[TARGET]
$ sudo rm -rf /lib/modules/[TARGET]
```

And update grub:

```shell
$ sudo update-grub
```
