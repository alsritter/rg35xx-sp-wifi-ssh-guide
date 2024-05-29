# 从 x86 通过 SSH 连接到 rg35xx-plus
本指南告诉你如何通过 SSH 连接到 rg35xx-plus，以便通过 wifi 管理你的 ROM 或做其他有趣的事情。我希望不会，但它可能会破坏某些东西，所以要小心。本指南是为 Ubuntu 23.10 编写的。

## 安装 qemu
在你的 x86 机器上运行模拟的 chroot 需要 qemu。你可以在这里了解更多信息：
https://unix.stackexchange.com/questions/41889/how-can-i-chroot-into-a-filesystem-with-a-different-architechture
或者只需运行以下命令：
```bash
apt-get install qemu-user-static
cp $(which qemu-arm-static) /mnt/usr/bin
```

## 从 x86 chroot
首先，你需要插入原装 SD 卡并找到 linuxrootfs 和 boot 分区。对于我来说，它们是 /dev/mmcblk0p5 和 /dev/mmcblk0p6。以下命令将 chroot 到 SD 卡。

```bash
sudo mount /dev/mmcblk0p5 /mnt
sudo mount /dev/mmcblk0p6 /mnt/boot
for dir in /dev /dev/pts /proc /sys /run; do sudo mount --bind $dir /mnt$dir; done
sudo cp /proc/mounts /mnt/etc/mtab
sudo mount -o bind /etc/resolv.conf /mnt/etc/resolv.conf
sudo chroot /mnt qemu-arm-static /bin/bash
```

现在在 chroot 环境中执行以下命令

```bash
apt install openssh-server
systemctl enable ssh
```

## 允许 root 用户通过 ssh 登录
安装 ssh 后，编辑 /etc/ssh/sshd_config 并在文件底部添加以下内容：
```
PermitRootLogin yes
```
现在你可以关闭 chroot 并将 SD 卡重新插入设备。你可以启用 wifi 并使用用户名 root 和密码 root 登录。

## 可选且可能有风险：
由于某些原因，这台设备上的 sudo 二进制文件和配置文件属于游戏用户。可以这样更改：
```bash
chown root:root /usr/bin/sudo && chmod 4755 /usr/bin/sudo
chown root:root /usr/lib/sudo/sudoers.so && chmod 4755 /usr/lib/sudo/sudoers.so
chown root:root /etc/sudoers
```

祝你好运 ;)