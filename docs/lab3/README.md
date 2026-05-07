# 内核态 Echo Server 实验文档

## 1. 实验目标

这个实验的目标，是让你亲手完成一次最基础的 Linux 内核网络实验：

1. 编译一个内核模块。
2. 将模块加载进内核。
3. 在内核态创建一个 TCP Echo Server。
4. 用用户态客户端连接它并发送消息。
5. 通过 `dmesg` 和 `/proc/kecho` 观察模块运行状态。

如果实验成功，你会看到这样一个现象：用户态客户端发出去什么字符串，内核态服务器就原样返回什么字符串。

## 2. 这个实验在做什么

平时我们写的网络程序大多运行在用户态，比如用 `socket`、`bind`、`listen`、`accept` 写一个普通 TCP 服务端。本实验把这个过程搬到了内核态。

模块加载后会做三件事：

1. 在内核里创建监听 socket，并绑定到 `0.0.0.0:8888`。
2. 启动一个内核线程，循环等待客户端连接。
3. 收到客户端消息后，把消息再发送回去，也就是 echo。

为了方便观察，模块还提供了两种调试手段：

1. 通过 `dmesg` 打印连接、收包、断开等日志。
2. 通过 `/proc/kecho` 输出统计信息，例如连接次数、收发包次数、最近一次消息内容。

## 3. 适合谁做

这份文档面向初学者。如果你只具备下面这些基础，就可以开始：

1. 知道什么是 Linux 命令行。
2. 知道 C 语言最基本的编译过程。
3. 大概知道用户态和内核态的区别。

如果你还没有学过 Linux 内核模块，也没关系。本实验正适合作为入门练习。

## 4. 实验环境

建议环境：

1. Debian 12 或兼容的 Linux 发行版。
2. Linux Kernel 5.x 及以上。
3. 当前内核版本对应的头文件已经安装。
4. 具有 `sudo` 权限。

先安装依赖：

```bash
sudo apt update
sudo apt install -y build-essential linux-headers-$(uname -r)
```

## 5. 项目文件说明

项目中你主要关心下面几个文件：

```text
.
├── Makefile
├── README.md
├── client.c
└── kecho.c
```

它们的作用分别是：

1. `kecho.c`：内核模块主体。负责创建内核态 TCP 服务器、处理客户端连接、维护统计信息、注册 `/proc/kecho`。
2. `client.c`：用户态测试程序。用于连接 `127.0.0.1:8888`，发送消息，并打印服务器返回的内容。
3. `Makefile`：同时编译内核模块 `kecho.ko` 和用户态客户端 `client`。
4. `README.md`：实验说明文档，也就是你现在看到的这份文档。

## 6. 实验原理概览

这部分不展开讲源码细节，只建立一个整体认识。做实验前，你至少要知道每部分在做什么。

### 6.1 模块加载后发生了什么

模块初始化函数会先启动服务器，再创建 `/proc/kecho`，最后启动一个内核线程负责 `accept`。

你可以把它理解成下面这个流程：

```text
加载模块
	-> 创建内核 socket
	-> bind 到 0.0.0.0:8888
	-> listen
	-> 创建 /proc/kecho
	-> 启动 accept 线程
```

### 6.2 客户端连接后发生了什么

当用户态客户端执行 `./client hello` 时：

1. 客户端连接到 `127.0.0.1:8888`。
2. 内核线程接受连接。
3. 服务器接收字符串 `hello`。
4. 服务器把 `hello` 原样发回去。
5. 客户端打印接收到的数据。

### 6.3 为什么还要看 `/proc/kecho`

`dmesg` 更适合看“发生了什么事件”，比如有没有连接成功、有没有收到消息。

`/proc/kecho` 更适合看“当前统计状态”，比如：

1. 当前有几个客户端。
2. 一共连接过多少次。
3. 一共收了多少个包。
4. 最近一次收到的消息是什么。

这两个观察点配合起来，特别适合初学者理解内核模块的运行过程。

## 7. 实验步骤

建议你开两个终端：

1. 终端 A：持续观察日志。
2. 终端 B：编译、加载模块、运行客户端、卸载模块。

### 7.1 编译实验代码

在项目目录执行：

```bash
make
```

编译成功后应该看到两个关键产物：

1. `kecho.ko`：内核模块文件。
2. `client`：用户态测试程序。

如果你想清理构建产物，可以执行：

```bash
make clean
```

### 7.2 加载内核模块

执行：

```bash
sudo insmod kecho.ko
```

然后确认模块是否已经加载：

```bash
lsmod | grep kecho
```

如果看到了 `kecho`，说明模块已经在内核中。

### 7.3 观察内核日志

在终端 A 中执行：

```bash
sudo dmesg -w
```

如果模块加载正常，你应该能看到类似日志：

```text
[kecho] listening on 0.0.0.0:8888
```

这说明内核态服务器已经开始监听 8888 端口。

### 7.4 查看 `/proc/kecho`

在终端 B 中执行：

```bash
cat /proc/kecho
```

第一次查看时，通常能看到类似结果：

```text
Kernel Echo Server Status

Listen Port      : 8888
Current Clients  : 0
Connect Count    : 0
Disconnect Count : 0
Recv Packets     : 0
Send Packets     : 0
Last Message     : (none)
```

这一步的意义是确认两件事：

1. procfs 接口已经创建成功。
2. 模块统计信息处于初始状态。

### 7.5 运行用户态客户端测试

执行：

```bash
./client hello
```

预期输出：

```text
send: hello
recv: hello
```

如果返回内容和发送内容一致，说明 echo 功能正常。

你也可以发送带空格的消息：

```bash
./client hello kernel world
```

### 7.6 再次查看日志和统计信息

客户端运行之后，再执行一次：

```bash
cat /proc/kecho
```

此时你应该看到统计值发生变化，例如：

1. `Connect Count` 增加。
2. `Disconnect Count` 增加。
3. `Recv Packets` 增加。
4. `Send Packets` 增加。
5. `Last Message` 变成你刚才发送的内容。

同时，在 `dmesg` 中你也应该能看到类似日志：

```text
[kecho] client connected: 127.0.0.1:xxxxx
[kecho] recv: hello
[kecho] client disconnected: 127.0.0.1:xxxxx
```

这里的端口号会变化，这是正常现象，因为它是客户端的临时端口。

### 7.7 卸载模块

实验结束后执行：

```bash
sudo rmmod kecho
```

然后可以再次确认模块是否已经卸载：

```bash
lsmod | grep kecho
```

如果没有输出，说明卸载成功。

你还可以再看一眼日志，通常会出现：

```text
[kecho] module unloaded
```

## 8. 一次完整实验可以这样做

如果你想按最顺的节奏跑完整个实验，可以按下面顺序操作。

终端 A：

```bash
sudo dmesg -w
```

终端 B：

```bash
make
sudo insmod kecho.ko
cat /proc/kecho
./client hello
cat /proc/kecho
sudo rmmod kecho
```

## 9. 做实验时你应该关注什么

初学者容易把这个实验做成“命令跑通就结束”。这还不够。你至少应该观察下面几个点：

1. `insmod` 之后，为什么 `dmesg` 里会出现 listening 日志。
2. 为什么 `/proc/kecho` 在没有客户端连接时，统计值全是 0。
3. 为什么执行一次 `./client hello` 后，连接数、收包数、发包数会变化。
4. 为什么客户端打印的 `recv: hello` 可以证明内核态 echo 成功。
5. 为什么卸载模块后，`/proc/kecho` 会消失。

如果你能把这五个问题讲清楚，说明你已经真正理解了这个实验。

## 10. 实验中会用到哪些 API

这一节是给初学者的“快速认路图”。你不需要一开始就把所有 API 记住，但最好知道它们分别属于哪一类、解决什么问题。

## 10.1 模块生命周期相关 API

这些 API 决定模块什么时候启动、什么时候退出。

1. `module_init(kecho_init)`：注册模块加载时的入口函数。执行 `insmod kecho.ko` 时，最终会进入这里。
2. `module_exit(kecho_exit)`：注册模块卸载时的出口函数。执行 `rmmod kecho` 时会进入这里。
3. `__init`：标记初始化函数使用的内存可以在初始化完成后释放。
4. `__exit`：标记退出函数只在模块卸载时使用。
5. `MODULE_LICENSE`、`MODULE_AUTHOR`、`MODULE_DESCRIPTION`：给模块补充元信息。

你可以把它们理解成“这个模块从哪里开始、到哪里结束”。

## 10.2 内核网络相关 API

这是本实验最核心的一组 API，对应“在内核里写 TCP 服务器”。

1. `sock_create_kern`：在内核态创建 socket。它相当于用户态里的 `socket`。
2. `kernel_bind`：给 socket 绑定 IP 和端口。这里绑定的是 `0.0.0.0:8888`。
3. `kernel_listen`：让 socket 进入监听状态，准备接收客户端连接。
4. `kernel_accept`：接受一个客户端连接。它相当于用户态里的 `accept`。
5. `kernel_recvmsg`：从客户端读取数据。它相当于用户态里的 `recv`。
6. `kernel_sendmsg`：向客户端发送数据。它相当于用户态里的 `send`。
7. `sock_release`：释放 socket 资源。监听 socket 和客户端 socket 最后都要释放。
8. `htons`、`htonl`、`ntohs`：处理端口和地址的字节序问题。网络字节序通常是大端序，所以绑定端口和打印端口时都常会遇到它们。

如果你只想先抓主线，可以记住下面这个顺序：

```text
sock_create_kern -> kernel_bind -> kernel_listen -> kernel_accept
											-> kernel_recvmsg
											-> kernel_sendmsg
```

## 10.3 内核线程相关 API

因为服务器需要持续等待连接，所以这里用了一个内核线程。

1. `kthread_run`：创建并启动内核线程。这里用它启动 accept 线程。
2. `kthread_should_stop`：在线程循环里检查“现在是否应该退出”。
3. `kthread_stop`：通知线程停止，并等待线程退出。
4. `msleep_interruptible`：让线程短暂休眠，避免在没有连接或没有数据时空转占满 CPU。

这组 API 的作用可以概括为：让服务器在后台一直运行，但又能在模块卸载时干净退出。

## 10.4 procfs 和 seq_file 相关 API

这组 API 用来把内核里的统计信息导出到 `/proc/kecho`，让用户可以直接 `cat /proc/kecho` 查看。

1. `proc_create`：创建 `/proc` 下的文件节点，这里创建的是 `/proc/kecho`。
2. `proc_remove`：删除 procfs 节点，通常在模块退出时调用。
3. `single_open`：为只读、一次性输出的 proc 文件准备打开逻辑。
4. `seq_read`：负责把内容读给用户空间。
5. `seq_printf`：按格式输出字符串到 proc 文件。
6. `seq_puts`：输出普通字符串到 proc 文件。
7. `seq_lseek`：支持文件偏移操作。
8. `single_release`：释放 `single_open` 建立的资源。

对初学者来说，只要先理解一点就够了：`proc_create` 是“把窗口开出来”，`seq_printf` 是“往窗口里写内容”。

## 10.5 同步与共享数据相关 API

服务器在运行时会不断修改统计信息，所以需要保护共享数据，避免并发读写出问题。

1. `mutex_init`：初始化互斥锁。
2. `mutex_lock`：进入临界区之前加锁。
3. `mutex_unlock`：离开临界区之后解锁。

这里的锁主要保护这些统计字段：

1. 当前连接数。
2. 总连接次数。
3. 总断开次数。
4. 收发包次数。
5. 最近一次消息内容。

初学时你可以把 `mutex` 理解成一句话：同一时刻只允许一个执行流修改这份统计数据。

## 10.6 字符串和内存处理 API

这些 API 不算“实验主角”，但在内核代码里非常常见。

1. `memset`：清空结构体或缓冲区。
2. `memcpy`：复制内存数据，例如保存最近一次收到的消息。
3. `strscpy`：安全地复制字符串。
4. `min_t`：在两个值里取较小的那个，并显式指定类型。

它们主要用于“整理数据”和“避免越界”。

## 10.7 日志和错误处理相关 API

做内核实验时，日志比图形界面更重要。你排查问题基本都靠这类 API。

1. `pr_info`：打印内核日志。你在 `dmesg` 里看到的 `[kecho]` 日志就是这样出来的。
2. `IS_ERR`：判断某些返回值是不是错误指针。
3. `PTR_ERR`：把错误指针转换成具体的错误码。

对于初学者，一个很重要的习惯是：只要模块行为不符合预期，第一时间看 `dmesg`。

## 10.8 用户态客户端相关 API

这个实验不只有内核 API，还包含一份普通的用户态客户端代码。它的价值在于帮助你对照理解。

1. `socket`：创建用户态 socket。
2. `connect`：连接到服务器。
3. `send`：发送数据。
4. `recv`：接收数据。
5. `close`：关闭 socket。
6. `inet_pton`：把字符串形式的 IP 地址转换成二进制格式。
7. `snprintf`：把命令行参数拼接成一条完整消息。
8. `strlen`：计算消息长度。

你可以把本实验简单地记成下面这组对应关系：

1. 用户态 `socket` 对应内核态 `sock_create_kern`
2. 用户态 `send/recv` 对应内核态 `kernel_sendmsg/kernel_recvmsg`
3. 用户态客户端负责发消息，内核态服务器负责原样返回

## 10.9 阅读源码时的建议顺序

如果你现在就想开始读代码，建议按下面顺序看，最不容易迷路：

1. 先看模块入口 `kecho_init`，搞清楚模块加载后做了什么。
2. 再看 `kecho_start_server`，搞清楚 socket 是怎么创建、绑定、监听的。
3. 然后看 `kecho_accept_thread`，搞清楚客户端连接是怎么接入的。
4. 再看 `kecho_handle_client`，搞清楚 echo 是怎么完成的。
5. 最后看 `kecho_proc_show`，理解 `/proc/kecho` 为什么能显示统计信息。
6. 对照 `client.c` 再看一次，就能把用户态和内核态流程串起来。

## 11. 常见问题

### 11.1 `make` 失败，提示找不到内核头文件

原因通常是当前内核版本对应的头文件没有安装。

解决方法：

```bash
sudo apt install -y linux-headers-$(uname -r)
```

### 11.2 `insmod` 失败，提示权限不足

内核模块加载需要管理员权限，请使用：

```bash
sudo insmod kecho.ko
```

### 11.3 客户端连接失败

优先检查这几项：

1. 模块是否真的已经加载：`lsmod | grep kecho`
2. `dmesg` 是否出现 listening 日志。
3. 是否已经误卸载模块。
4. 是否修改过端口，导致客户端和服务器端口不一致。

### 11.4 `/proc/kecho` 不存在

通常说明模块初始化没有完整成功，或者模块已经被卸载。请先检查：

```bash
dmesg | tail -n 20
```

看看是否有模块启动失败日志。

## 12. 实验结论

做完这个实验，你应该掌握下面这些最基本的能力：

1. 会编译并加载一个 Linux 内核模块。
2. 知道内核态也可以使用 socket API 完成网络通信。
3. 知道如何用 `dmesg` 观察模块运行过程。
4. 知道如何用 procfs 导出模块状态。
5. 能把用户态客户端与内核态服务器的交互过程串起来理解。

## 13. 可选思考题

如果你做完基础实验，还可以继续思考下面几个问题：

1. 现在服务器一次只处理一个连接，如果想同时处理多个客户端，应该怎么改？
2. 现在统计信息只记录最后一条消息，如果想记录最近 N 条消息，应该怎么设计数据结构？
3. 现在消息长度有限，如果客户端发送更长的数据，会发生什么？
4. 如果把监听地址从 `0.0.0.0` 改成 `127.0.0.1`，行为会有什么不同？

这些问题很适合作为下一步实验。