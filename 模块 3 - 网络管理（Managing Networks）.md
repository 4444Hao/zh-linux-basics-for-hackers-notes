# 黑客 Linux 基础（Linux Basics for Hackers）

## 模块 3 - 网络管理（Managing Networks）

---

## 概述

黑客攻击几乎总是发生在网络之上。在网络上进行任何操作之前，你需要先了解自己在网络上的身份——你的 IP 地址和 MAC 地址——以及如何管理它们。本模块将介绍如何查看网络信息、修改网络配置，以及了解 DNS 的基础知识。

![网络路由示意图](assets/network_routing_diagram_1789213191969.jpg)

---

## 每台设备都有的两个地址

网络上的每台设备都有两个标识符：

**IP 地址** - 这是你在网络上的逻辑地址。它由路由器分配（或由你手动设置），用来决定数据该往哪里去。可以把它想象成你的街道地址——它告诉网络你在哪里，但它是可以变化的。

**MAC 地址** - 这是由厂商烧录在物理网卡中的地址。它是硬件层面的标识符，由 12 个十六进制字符组成，全球每个网络接口都是唯一的。它看起来像这样：`00:1A:2B:3C:4D:5E`。与 IP 地址不同，你无法永久修改它——但可以在软件层面临时伪造。

理解这两个地址很重要，因为当你在网络中时，它们就是用来识别你的身份的。学会查看和修改它们是一项基础技能。

---

## 常用核心命令

### ifconfig

这是你查看和管理网络接口的主要工具。不带参数直接运行它，就会显示当前网络配置的全部信息。

```bash
ahegazy0@kali:~$ ifconfig

```

输出会显示你机器上的各个网络接口。常见的接口包括：

| 接口 | 说明 |
| --- | --- |
| `eth0` | 你的有线以太网连接 |
| `wlan0` | 你的无线（Wi-Fi）连接 |
| `lo` | 回环接口（Loopback）——系统与自身通信，固定为 127.0.0.1 |

在输出中留意 `inet`——那就是你当前的 IP 地址；留意 `ether`——那就是你的 MAC 地址。

只查看某个特定接口：

```bash
ahegazy0@kali:~$ ifconfig eth0

```

为接口分配新的 IP 地址：

```bash
ahegazy0@kali:~$ ifconfig eth0 192.168.1.100

```

同时设置子网掩码：

```bash
ahegazy0@kali:~$ ifconfig eth0 192.168.1.100 netmask 255.255.255.0

```

启用或禁用网络接口：

```bash
ahegazy0@kali:~$ ifconfig eth0 up
ahegazy0@kali:~$ ifconfig eth0 down

```

修改配置后，有时需要先将接口禁用再重新启用。

---

### iwconfig

和 `ifconfig` 类似，但专门用于无线接口。它会显示 Wi-Fi 相关的信息，例如网络名称（ESSID）、信号强度和传输速率。

```bash
ahegazy0@kali:~$ iwconfig

```

大多数时候你只是用它来查看信息，而不是修改配置。不过它在确认当前连的是哪个无线网络、信号状况如何时很有用。

---

### dhclient

当你用 `ifconfig` 手动设置了 IP 地址后，就不再从路由器自动获取 IP 了。如果你想恢复从网络自动获取 IP，可以使用 `dhclient`。

```bash
ahegazy0@kali:~$ dhclient eth0

```

这会向网络的 DHCP 服务器发送请求，相当于说“给我分配一个 IP 地址”，随后服务器就会分配给你一个。如果你手动改了 IP 导致断网，通常用这个就能修好。

如果你觉得当前的 IP 导致了问题，也可以用它来重新申请一个新 IP：

```bash
ahegazy0@kali:~$ dhclient -r eth0     释放当前 IP
ahegazy0@kali:~$ dhclient eth0        申请新 IP

```

---

### 修改 MAC 地址

MAC 地址本意是永久不变的，但在软件层面很容易伪造。这种修改只在重启前有效——重启之后，真实的硬件 MAC 地址就会恢复。

先禁用网络接口：

```bash
ahegazy0@kali:~$ ifconfig eth0 down

```

然后设置新的 MAC 地址：

```bash
ahegazy0@kali:~$ ifconfig eth0 hw ether 00:11:22:33:44:55

```

再重新启用接口：

```bash
ahegazy0@kali:~$ ifconfig eth0 up

```

运行 `ifconfig eth0` 确认修改是否生效。此时 `ether` 那一行应该显示你的新 MAC 地址。

> MAC 地址的修改只存在于内存中，重启后不会保留。如果你需要让修改持久生效，可以使用 `macchanger` 等工具更妥善地处理。

---

### dig

`dig` 用于查询 DNS——也就是把 `google.com` 这类域名解析为 IP 地址的系统。它比单纯 `ping` 一个域名能提供更详细的信息。

```bash
ahegazy0@kali:~$ dig google.com

```

输出中的 `ANSWER SECTION`（回答部分）会显示 `google.com` 解析出的 IP 地址。

专门查询邮件服务器记录：

```bash
ahegazy0@kali:~$ dig google.com mx

```

`mx` 代表 Mail Exchange（邮件交换）——它会告诉你哪些服务器负责处理该域名的电子邮件。这在信息收集（recon）阶段很有用。

其他值得了解的记录类型：

| 记录类型 | 显示内容 |
| --- | --- |
| `a` | 域名的 IPv4 地址 |
| `aaaa` | IPv6 地址 |
| `mx` | 邮件服务器 |
| `ns` | 域名服务器（哪些 DNS 服务器是权威服务器） |
| `txt` | 文本记录——通常包含验证信息 |

```bash
ahegazy0@kali:~$ dig google.com ns

```

---

### DNS——为什么它很重要（简要说明）

DNS（Domain Name System，域名系统）就是互联网的“通讯录”。当你输入 `google.com` 时，你的电脑会问 DNS 服务器：“google.com 的 IP 是多少？”然后拿到对应的地址。如果没有 DNS，你就得记住每个网站的 IP 地址。

黑客关注 DNS 有几个原因。首先，DNS 查询可以暴露目标基础设施的大量信息——比如邮件服务器、子域名、域名服务器。其次，DNS 是可以被操纵的——如果你能重定向 DNS 响应，就能把别人引导到完全错误的服务器上（这就叫 DNS 投毒或 DNS 欺骗，后续模块会讲到）。

就目前而言，知道如何用 `dig` 查看 DNS 记录就足够了。

---

## 命令速查表

| 任务 | 命令 |
| --- | --- |
| 查看所有网络接口 | `ifconfig` |
| 查看单个接口 | `ifconfig eth0` |
| 设置 IP 地址 | `ifconfig eth0 192.168.1.100` |
| 禁用/启用接口 | `ifconfig eth0 down / up` |
| 查看无线网络信息 | `iwconfig` |
| 从 DHCP 请求 IP | `dhclient eth0` |
| 释放当前 IP | `dhclient -r eth0` |
| 伪造 MAC 地址 | `ifconfig eth0 hw ether 00:11:22:33:44:55` |
| DNS 查询 | `dig google.com` |
| 查找邮件服务器 | `dig google.com mx` |

---

## 练习清单

* [ ] 运行 `ifconfig`，查看 `eth0` 或 `wlan0` 上的 IP 地址和 MAC 地址
* [ ] 运行 `dig google.com` 并查看回答部分（answer section），记下返回的 IP 地址
* [ ] 运行 `dig google.com mx`，看看哪些服务器负责处理 Google 的邮件
* [ ] 尝试把 IP 改为 `192.168.1.100`，用 `ifconfig` 确认，然后运行 `dhclient eth0` 从 DHCP 重新获取一个 IP

> 💡 *如果想做更深入的练习，我也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的习题。*

---

*下一章：模块 4 - 软件管理（Software Management）*
