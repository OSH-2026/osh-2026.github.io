# 构建 RISCV-64 Linux 内核

## 环境准备

使用了 Vlab 的 Ubuntu 24.04 LXC 虚拟环境。

更新并安装所需的软件：

```shell
sudo apt update
sudo apt upgrade -y && sudo apt install -y build-essential git bc bison flex libssl-dev libncurses-dev qemu-system-misc gcc-riscv64-linux-gnu gdb gdb-multiarch ncat telnet
```

下载 Linux 源代码：

```shell
git clone --filter=tree:0 https://mirrors.ustc.edu.cn/linux.git
cd linux
du -h | tail
```

可以看到 Linux 内核的源代码大小在 clone 后达到了 3GB，这其中 1.3GB 是 `.git` 目录下的 Git 仓库数据。

然后，设置交叉编译选项：

```shell
export ARCH=riscv
export CROSS_COMPILE=riscv64-linux-gnu-
```

提示：如果你在中间做错了某一步骤，可以用下面的命令来清理环境并重新开始，无需再次下载：

```shell
git reset --hard
git clean -fdx
```

## 第一次交叉编译

### 初始化配置

我们只需要一个极小的内核，因此：

```shell
make defconfig
```

然后，由于 Qemu 里面没有什么设备，因此不需要编译很多驱动程序。因此只保留我们需要的。这里可以使用 `make menuconfig` 来进行交互式的配置，但是由于需要修改的地方比较多，交互式操作可能不太方便。所以，使用 `config` 切片来完成配置。编写 `qemu.config` 并填入下面的内容：

```plain
CONFIG_SERIAL_8250=y
CONFIG_SERIAL_8250_CONSOLE=y
CONFIG_OF=y
CONFIG_OF_FLATTREE=y
CONFIG_VIRTIO=y
CONFIG_VIRTIO_MMIO=y

CONFIG_MODULES=n
CONFIG_NET=n
CONFIG_USB=n
CONFIG_SOUND=n
CONFIG_DRM=n

CONFIG_CC_OPTIMIZE_FOR_SIZE=y
```

### 构建

接下来将它合并到 `defconfig` 中并且开始编译：

```shell
make olddefconfig
make -j$(nproc)
```

在这个过程中，make 可能会询问一些问题，类似

```plain
Vector extension support (RISCV_ISA_V) [Y/n/?] (NEW)
```

一般来说，直接使用默认值（按 Enter）即可。

如果直接按照 `defconfig` 构建的话，make 在 `drivers/` 阶段花费了大量的时间——但其实这里很多驱动不是我们所需要的，它们也同时占据了编译出来的内核的相当大小。因此，建议从精简驱动开始裁剪嵌入式内核。此外，还可以通过 `tinyconfig` 得到一个精简的配置。

最后的输出是：

```plain
  OBJCOPY vmlinux
  GEN     modules.builtin.modinfo
  GEN     modules.builtin
  OBJCOPY arch/riscv/boot/Image
  Kernel: arch/riscv/boot/Image is ready
  GZIP    arch/riscv/boot/Image.gz
  Kernel: arch/riscv/boot/Image.gz is ready
```

这就说明我们的内核准备好了！

在构建过程中，都可以使用 `file` 命令来看我们的构建产物的信息。例如：

```shell
$ file vmlinux
vmlinux: ELF 64-bit LSB shared object, UCB RISC-V, RVC, soft-float ABI, version 1 (SYSV), dynamically linked, BuildID[sha1]=618e9ee6b09fff369073f9dc28538a44cfcb5b3e, not stripped
```

这说明我们的交叉编译是正确的，产物 `vmlinux` 确实是 RISCV-64 架构的二进制文件。

### 构建 `initramfs` 和 Busybox

为了让我们的内核有事情可做，可以使用 Busybox，提供一个极小化的环境。首先需要建立起最基本的文件系统结构：

```shell
mkdir -p rootfs/{bin,sbin,proc,sys,usr/bin,usr/sbin}
```

然后，获取 Busybox 的源代码：

```shell
git clone --depth 1 https://git.busybox.net/busybox.git
```

你可以使用 `make menuconfig` 来进行交互式的配置。需要配置的有：

- 选中 Settings -> Build static binary，确保产物是静态链接的；
- 取消选中 Networking Utilities 下的所有选项；

然后使用 `make` 构建。构建完成后，产物应该具有如下的信息：

```shell
$ file busybox_unstripped
busybox_unstripped: ELF 64-bit LSB executable, UCB RISC-V, RVC, double-float ABI, version 1 (SYSV), statically linked, BuildID[sha1]=35d654cb925b199e53af27ddff8cd4b6c0b7995b, for GNU/Linux 4.15.0, not stripped
```

接下来，使用

```shell
make CONFIG_PREFIX=../rootfs install
```

将 Busybox 安装到你的 Initramfs 中。

然后，创建 `init` 程序，这样我们才能让内核创建第一个进程：

```shell
cat > rootfs/init << 'EOF'
#!/bin/sh
mount -t proc none /proc
mount -t sysfs none /sys
echo "Hello from RISC-V init!"
exec /bin/sh
EOF

chmod +x rootfs/init
```

### 最后的打包

需要构建的东西已经完成了，接下来进行一下打包：

```shell
cd rootfs/
find . | cpio -o --format=newc | gzip > ../initramfs.cpio.gz
cd ..
```

## 运行内核

我们已经成功地准备好了第一次构建的产物。现在来运行一下。

```shell
qemu-system-riscv64 \
  -machine virt \
  -nographic \
  -kernel arch/riscv/boot/Image \
  -initrd initramfs.cpio.gz \
  -append "console=ttyS0"
```

你应该可以看到：

```shell
[    1.431314] Run /init as init process
Hello from RISC-V init!
/bin/sh: can't access tty; job control turned off
~ # ls
bin      init     proc     sbin     usr
dev      linuxrc  root     sys
~ #
```

成功了！
