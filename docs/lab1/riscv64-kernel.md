# 构建 RISCV-64 Linux 内核

## 环境准备

使用了 Vlab 的 Ubuntu 24.04 LXC 虚拟环境。

更新并安装所需的软件：

```shell
sudo apt update
sudo apt upgrade -y && sudo apt install -y build-essential git bc bison flex libssl-dev libncurses-dev qemu-system-misc gcc-riscv64-linux-gnu gdb gdb-multiarch ncat telnet
```
