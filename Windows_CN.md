
注意！！ Windows 的 WSL 有很多坑，建议还是使用 VMware

### 步骤 1：安装 VMware 并设置 Ubuntu 虚拟机

1. 下载并安装 VMware Workstation Player 或 VMware Workstation Pro。
2. 下载 Ubuntu 的 ISO 文件（推荐 20.04 或更高版本）。
3. 打开 VMware 并创建一个新的虚拟机，选择下载的 Ubuntu ISO 文件进行安装。
4. 按照屏幕提示完成 Ubuntu 的安装。

### 步骤 2：设置 Ubuntu 并安装 qemu

1. 启动 Ubuntu 虚拟机。
2. 更新包管理器并安装 qemu：

```bash
sudo apt-get update
sudo apt-get install qemu-user-static
```

### 步骤 3：准备 SD 卡

1. 将 rg35xx-plus 的 SD 卡插入你的计算机。
2. 在虚拟机中查找 SD 卡的设备信息：

```bash
sudo fdisk -l
```

这将列出所有连接的磁盘设备。找到你的 SD 卡的设备 ID（例如，`/dev/sdb`）。

3. 创建挂载点并挂载 SD 卡的分区：

```bash
sudo mkdir /mnt/sdcard
sudo mount /dev/sdb1 /mnt/sdcard
```

### 步骤 4：在 Ubuntu 中 chroot 到 SD 卡

1. 挂载分区并准备 chroot 环境：

```bash
# 绑定必要的系统目录，以便在 chroot 环境中访问这些资源
for dir in /dev /dev/pts /proc /sys /run; do sudo mount --bind $dir /mnt/sdcard$dir; done

# 复制当前的挂载信息到 chroot 环境的 /etc/mtab 文件中
sudo cp /proc/mounts /mnt/sdcard/etc/mtab

# 绑定当前的 DNS 配置文件，以便 chroot 环境内能够解析域名
sudo mount -o bind /etc/resolv.conf /mnt/sdcard/etc/resolv.conf

# 进入 chroot 环境，并指定使用 qemu-arm-static 进行模拟
sudo chroot /mnt/sdcard qemu-arm-static /bin/bash
```

### 步骤 5：在 chroot 环境中安装和配置 SSH

1. 在 chroot 环境中安装 `openssh-server` 并启用 SSH 服务：
    ```bash
    apt install openssh-server
    systemctl enable ssh
    ```
2. 编辑 SSH 配置文件以允许 root 登录：
    ```bash
    nano /etc/ssh/sshd_config
    ```
    添加以下内容：
    ```
    PermitRootLogin yes
    ```
3. 退出 chroot 环境并取消挂载分区：
    ```bash
    exit
    sudo umount /mnt/sdcard/dev/pts
    sudo umount /mnt/sdcard/dev
    sudo umount /mnt/sdcard/proc
    sudo umount /mnt/sdcard/sys
    sudo umount /mnt/sdcard/run
    sudo umount /mnt/sdcard/etc/resolv.conf
    sudo umount /mnt/sdcard
    ```

### 步骤 6：启用设备上的 WiFi 并通过 SSH 登录

1. 将 SD 卡重新插入设备并启动设备。
2. 启用设备上的 WiFi。
3. 使用 SSH 客户端（如 PuTTY 或 Windows 内置的 `ssh` 命令）连接到设备：
    ```powershell
    ssh root@<设备的IP地址>
    ```

### 可选步骤：修复 sudo 权限

如果需要修复 sudo 权限，可以在连接到设备后执行以下命令：
```bash
chown root:root /usr/bin/sudo && chmod 4755 /usr/bin/sudo
chown root:root /usr/lib/sudo/sudoers.so && chmod 4755 /usr/lib/sudo/sudoers.so
chown root:root /etc/sudoers
```

祝你好运！如果有任何问题，请随时询问。