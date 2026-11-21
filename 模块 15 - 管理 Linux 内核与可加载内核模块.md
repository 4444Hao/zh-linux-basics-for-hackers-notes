# 黑客 Linux 基础（Linux Basics for Hackers）

## 模块 15 - 内核与可加载内核模块（The Kernel & Loadable Kernel Modules）

---

## 概述

内核是操作系统的核心。其他一切——你的终端、你的程序、你的文件系统——都建立在它之上。本模块将介绍内核的作用、如何查看和调整内核设置，以及如何通过模块在不重启系统的情况下扩展内核功能。

---

## 内核究竟做了什么

内核位于你的软件与硬件之间。当某个程序想读取文件、通过网络发送数据，或在屏幕上绘制内容时，它不会直接与硬件对话——而是向内核提出请求，由内核来处理。

这使得内核成为整个系统中权限最高的软件。它运行在所谓的内核空间（kernel space）**中，这与你的程序所运行的**用户空间（user space）完全隔离。用户空间中的进程只能做内核允许它做的事；而内核本身没有这种限制，它可以访问任何内存、任何硬件以及系统中的任何资源。

这也是它如此容易成为攻击目标的原因：运行在内核中的代码拥有对整台机器的绝对控制权。

![Linux 内核架构示意图](assets/linux_kernel_architecture_1789213639316.jpg)

---

## 查看内核版本

```bash
ahegazy0@kali:~$ uname -a
Linux kali 6.1.0-kali9-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.27-1kali1 x86_64 GNU/Linux
```

从左到右解读输出：

- `Linux` - 操作系统名称
- `kali` - 主机名
- `6.1.0-kali9-amd64` - 内核版本与架构
- `x86_64` - 确认这是 64 位内核

其中的架构信息（`x86_64` 对比 `i686`）可以用来区分 64 位与 32 位系统。

---

## 可加载内核模块（Loadable Kernel Modules，LKMs）

每次需要支持新硬件时，不可能都把内核重新编译并替换一遍。取而代之的是，内核支持**可加载内核模块**——即可以在系统运行时动态插入和移除的内核代码片段，无需重启。

驱动程序是最常见的例子。当你插上一块 USB 无线网卡时，内核会加载对应的驱动模块来与它通信；当你拔掉它时，该模块可以被移除。内核本身并没有改变，模块只是临时扩展了它的功能。

由于模块运行在内核空间中，它们拥有与内核本身相同的绝对权限。这也是为什么它们也会被用来隐藏 Rootkit——以模块形式插入的恶意代码可以拦截系统调用、隐藏进程、隐藏文件，而且是在操作系统自身难以轻易察觉的层级上完成这一切。

---

## 列出已加载的模块

```bash
ahegazy0@kali:~$ lsmod
```

显示当前已加载到内核中的每一个模块：

```
Module                  Size  Used by
bluetooth             663552  2 btusb
snd_hda_intel         57344  3
usbcore              286720  5 btusb,xhci_hcd
```

各列的含义如下：

- `Module` - 模块名称
- `Size` - 占用的内存大小
- `Used by` - 依赖该模块的其他模块或进程

你通常会看到几十个已加载的模块，大多数都是硬件驱动——音频、USB、显示、网络等。

---

## 获取模块信息

```bash
ahegazy0@kali:~$ modinfo bluetooth
```

输出内容包括：

- `filename` - 模块文件在磁盘上的存放位置
- `description` - 模块的作用
- `author` - 编写者
- `depends` - 该模块正常运行所依赖的其他模块
- `vermagic` - 编译该模块时所针对的内核版本

`depends` 字段尤其有用。模块往往存在依赖关系——即必须先加载的其他模块，`modinfo` 会把这条依赖链展示给你。

---

## 加载与移除模块

**添加模块：**

```bash
ahegazy0@kali:~$ modprobe bluetooth
```

`modprobe` 是完成这项工作的合适工具。与 `insmod`（更底层的替代命令）不同，`modprobe` 会自动解析并加载该模块所需的依赖项。如果 `bluetooth` 需要先加载另外两个模块，`modprobe` 会自动帮你处理。

**移除模块：**

```bash
ahegazy0@kali:~$ modprobe -r bluetooth
```

`-r` 参数用于移除模块。同样，`modprobe` 会处理依赖关系——如果还有其他东西依赖某个模块，它不会贸然移除。

---

## sysctl - 在运行时调整内核设置

内核通过位于 `/proc/sys` 的虚拟文件系统暴露了大量可配置参数。`sysctl` 命令允许你在系统运行时读取和修改这些设置。

**查看当前所有内核设置：**

```bash
ahegazy0@kali:~$ sysctl -a
```

这个列表很长。如果想查看某一项特定设置：

```bash
ahegazy0@kali:~$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 0
```

这一项设置——IP 转发（IP forwarding）——控制着系统是否会将网络数据包从一个接口转发到另一个接口。默认情况下它是关闭的；关闭时，你的机器收到数据包后，若发现目标不是自己，就会将其丢弃。

**开启 IP 转发：**

```bash
ahegazy0@kali:~$ sysctl -w net.ipv4.ip_forward=1
```

`-w` 参数表示“write（写入）”。现在内核会在不同接口之间转发数据包。这在中间人攻击（Man-in-the-Middle）环境中是必需的——你的机器需要处在流量路径中间，接收发往别处的数据包并将其继续转发出去。

**让 sysctl 修改永久生效：**

和环境变量一样，`sysctl -w` 的改动只在下次重启前有效。要让它永久生效，请将其添加到 `/etc/sysctl.conf` 中：

```
net.ipv4.ip_forward = 1
```

然后无需重启即可应用配置：

```bash
ahegazy0@kali:~$ sysctl -p
```

**使用 `sysctl` 时要小心。** 某些参数控制着基础的内核行为，改错数值可能导致网络异常、系统不稳定甚至更严重的问题。在修改任何设置前，务必先弄清楚它的作用。

---

## /proc 文件系统

与 `sysctl` 值得一起了解的是 `/proc`——这是一个仅存在于内存中的虚拟文件系统，是内核以文件形式对外暴露自身及运行中进程信息的接口。

```bash
ahegazy0@kali:~$ ls /proc
```

你会看到许多带数字的目录——每个正在运行的进程对应一个（数字就是它的 PID）。你还会看到像 `cpuinfo`、`meminfo` 和 `version` 这样的文件，它们暴露了硬件和内核数据。

```bash
ahegazy0@kali:~$ cat /proc/cpuinfo
ahegazy0@kali:~$ cat /proc/meminfo
ahegazy0@kali:~$ cat /proc/version
```

这些并不是磁盘上的真实文件，而是每次你读取它们时由内核即时生成的。这是“一切皆文件”理念在逻辑上的自然延伸——连动态的内核数据也以文件的形式呈现出来。

---

## 命令速查表

| **命令** | **作用** |
| --- | --- |
| `uname -a` | 显示完整的内核版本与架构 |
| `lsmod` | 列出当前已加载的所有内核模块 |
| `modinfo [name]` | 显示指定模块的详细信息 |
| `modprobe [name]` | 加载模块（及其依赖项） |
| `modprobe -r [name]` | 移除模块 |
| `sysctl -a` | 显示所有内核参数 |
| `sysctl [parameter]` | 显示指定的内核参数 |
| `sysctl -w [param=value]` | 在运行时修改内核参数 |
| `sysctl -p` | 应用 `/etc/sysctl.conf` 中的更改 |
| `cat /proc/cpuinfo` | 查看内核提供的 CPU 信息 |
| `cat /proc/meminfo` | 查看内核提供的内存信息 |

---

## 练习清单

- [ ] 运行 `uname -a`，识别内核版本以及它是 32 位还是 64 位
- [ ] 运行 `lsmod` 并上下翻阅，大致数一数当前加载了多少个模块
- [ ] 运行 `modinfo bluetooth` 并找到 `depends` 字段，看看它依赖哪些其他模块
- [ ] 运行 `sysctl net.ipv4.ip_forward` 查看当前值，并思考修改它会带来什么效果

> 💡 *如果想做更深入的练习，我也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的习题。*

---

_下一章：模块 16 - 自动化与计划任务（Automation & Scheduled Jobs）_
