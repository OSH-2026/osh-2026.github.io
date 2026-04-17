# su/sudo：用户切换

Linux 是一个多用户系统，每个用户拥有独立的身份标识符和权限。本节中，你需要为 shell 实现 `su` 和 `sudo` 两个内建命令，让 shell 能够切换用户身份或以提升的权限执行命令。

## Linux 用户模型

Linux 内核通过 **UID**（User ID）和 **GID**（Group ID）来标识每个进程的运行身份，并以此进行权限判断。每个进程同时具有以下两种 ID：

`getuid()` / `getgid()`

> 返回进程的**实际 UID / GID**（Real UID/GID），即进程真正属于的用户/组。

`geteuid()` / `getegid()`

> 返回进程的**有效 UID / GID**（Effective UID/GID），即内核实际用于权限检查的 ID。多数情况下与实际 UID/GID 相同，但在执行带有 setuid 位的程序时会不同。

通过 `setuid(uid)` 和 `setgid(gid)` 可以更改当前进程的有效（乃至实际）UID/GID。要查询系统中已有的用户信息（如用户名、UID、GID、家目录、默认 shell），可以使用 `getpwnam(name)` 或 `getpwuid(uid)`，它们定义在 `<pwd.h>` 中，返回指向 `struct passwd` 的指针。

???+ note "setuid 位"

    你可能注意到系统中的 `/usr/bin/su`、`/usr/bin/sudo` 等程序拥有一个特殊的 setuid 位（`-rwsr-xr-x`）。当普通用户执行这类程序时，内核会将其有效 UID 自动切换为文件所有者（通常是 root），从而让程序可以以 root 权限完成特权操作。

    在本实验中，`su` 和 `sudo` 作为 shell 的**内建命令**实现，因此不需要 setuid 位——但你的 shell 本身需要以 root 权限运行才能调用 `setuid(0)`。测试时建议以 root 身份运行你的 shell，或使用 `sudo ./shell` 启动。

## 实现 su

`su [username]` 命令将当前 shell 切换为另一个用户的身份。若不指定 `username`，则默认切换为 `root`。

例如：

```shell
$ whoami
student
$ su alice
$ whoami
alice
$ exit            # 退出切换后的 shell，回到原来的 shell
$ whoami
student
```

实现思路：

1. 调用 `getpwnam(username)` 查询目标用户的 UID 和 GID；若用户不存在，打印错误并返回。
2. 调用 `setgid(pw->pw_gid)` 切换到目标用户的主组。
3. 调用 `setuid(pw->pw_uid)` 切换到目标用户的 UID。
4. 调用 `execvp(shell, args)` 启动目标用户的默认 shell（`pw->pw_shell`），将当前进程替换为新 shell。

???+ question "思考"

    `su` 为什么需要先调用 `setgid` 再调用 `setuid`？如果顺序颠倒会发生什么？

    真实的 `su` 命令在切换用户之前还会验证密码。密码信息存储在哪里？如何验证？（提示：`man 3 crypt`，`/etc/shadow`）

???+ note "关于 getpwnam"

    需要包含头文件 `<pwd.h>` 和 `<sys/types.h>`。`struct passwd` 的关键字段包括：

    - `pw_uid`：用户 UID
    - `pw_gid`：用户主组 GID
    - `pw_dir`：家目录路径
    - `pw_shell`：默认登录 shell 路径

|                              要求                              | 满分 |
| :------------------------------------------------------------: | :--: |
| 正确实现 `su username`：切换至指定用户并启动其默认 shell       | 10%  |
| **「可选」** `su` 无参数时默认切换至 root                      |  5%  |
| **「可选」** 切换用户后同时更新 `$HOME`、`$USER` 等环境变量    |  5%  |

## 实现 sudo

`sudo command [args...]` 命令以 root 权限运行指定的命令，同时保持当前 shell 不变。

例如：

```shell
$ whoami
student
$ sudo whoami
root
$ whoami
student
```

实现思路：

1. 调用 `fork()` 创建子进程。
2. 在子进程中：调用 `setgid(0)` 和 `setuid(0)` 提升至 root 权限，再用 `execvp()` 执行目标命令。
3. 在父进程中：调用 `waitpid()` 等待子进程结束，shell 本身的 UID 不变。

???+ note "提醒"

    与 `su` 不同，`sudo` 的 setuid 操作发生在 **fork 出的子进程**中，不影响 shell 进程本身的 UID。因此在子进程中需在 `exec` 之前完成 `setuid`/`setgid`。

    更多信息可参考 `man 2 setuid`、`man 2 setgid` 和 `man 3 getpwnam`。

|                          要求                           | 满分 |
| :-----------------------------------------------------: | :--: |
| 正确实现 `sudo command`：以 root 权限执行命令，shell 自身 UID 不变 | 10%  |
