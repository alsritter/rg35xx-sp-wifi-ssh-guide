使用 Windows Subsystem for Linux (WSL) 来执行相关操作。

### 步骤 1：安装 Windows Subsystem for Linux (WSL)
1. 打开 PowerShell 作为管理员运行以下命令以启用 WSL：

```powershell
wsl --install
```

2. 安装完成后，重新启动你的计算机。
3. 在 Microsoft Store 中搜索并安装 Ubuntu（推荐 20.04 或更高版本）。

### 步骤 2：设置 WSL 和安装 qemu
1. 启动你安装的 Ubuntu 发行版。
2. 更新包管理器并安装 qemu：

```bash
sudo apt-get update
sudo apt-get install qemu-user-static
```

### 步骤 3：准备 SD 卡
1. 将 rg35xx-plus 的 SD 卡插入你的计算机。
2. 通过 WSL 挂载 SD 卡。你需要知道 Windows 分配给 SD 卡的驱动器号（例如 D:）。

```bash
sudo mkdir /mnt/sdcard
sudo mount -t drvfs D: /mnt/sdcard
```

3. 找到 SD 卡上的 `linuxrootfs` 和 `boot` 分区。例如，它们可能是 `/mnt/sdcard/partition1` 和 `/mnt/sdcard/partition2`。

上面的步骤中，我们将 SD 卡的分区直接作为 chroot 的虚拟目录。这使我们能够在 chroot 环境中对 SD 卡上的文件系统进行操作。

> 在 Linux 文件系统中，/mnt 目录通常用于临时挂载文件系统。例如，当你挂载一个外部存储设备（如 USB 驱动器或 SD 卡）时，通常会将其挂载到 /mnt 目录或其子目录。挂载是指将设备的文件系统连接到现有的目录结构中，使其内容可用。

### 步骤 4：在 WSL 中 chroot 到 SD 卡
1. 挂载分区并准备 chroot 环境：

```bash
# 将 SD 卡的主要分区挂载到 /mnt 目录
sudo mount /mnt/sdcard/partition1 /mnt

# 将 SD 卡的引导分区挂载到 /mnt/boot 目录
sudo mount /mnt/sdcard/partition2 /mnt/boot

# 绑定必要的系统目录，以便在 chroot 环境中访问这些资源
for dir in /dev /dev/pts /proc /sys /run; do sudo mount --bind $dir /mnt$dir; done

# 复制当前的挂载信息到 chroot 环境的 /etc/mtab 文件中
sudo cp /proc/mounts /mnt/etc/mtab

# 绑定当前的 DNS 配置文件，以便 chroot 环境内能够解析域名
sudo mount -o bind /etc/resolv.conf /mnt/etc/resolv.conf

# 进入 chroot 环境，并指定使用 qemu-arm-static 进行模拟
sudo chroot /mnt qemu-arm-static /bin/bash
```

> chroot 是一种将进程的根目录更改为新位置的技术，主要用于隔离文件系统。它不会对宿主机本身造成影响，因为它只是在特定进程的视角中更改了根目录，类似于创建了一个独立的操作环境。chroot 将进程的根目录更改为指定的目录。这个进程及其子进程只能访问这个目录及其子目录中的文件，而不能访问宿主机文件系统的其他部分。虽然文件系统是隔离的，但进程可能仍会共享其他资源，如网络堆栈或硬件设备。因此，对网络配置的更改可能会影响宿主机。



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
    sudo umount /mnt/dev/pts
    sudo umount /mnt/dev
    sudo umount /mnt/proc
    sudo umount /mnt/sys
    sudo umount /mnt/run
    sudo umount /mnt/etc/resolv.conf
    sudo umount /mnt/boot
    sudo umount /mnt
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