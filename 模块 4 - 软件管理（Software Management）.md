# 黑客 Linux 基础（Linux Basics for Hackers）

## 模块 4 - 软件管理（Software Management）

---

## 概述

Kali 预装了很多工具，但你仍会不断需要添加新工具、更新现有工具，或者直接从开发者的 GitHub 拉取项目。本模块将介绍 Linux 处理软件安装的方式——这与 Windows 完全不同——以及如何获取标准软件源之外的工具。

---

## Linux 如何管理软件

在 Windows 中，你会从网站下载安装包、双击运行，然后一路点下一步完成安装向导。Linux 并不是这样工作的。

Linux 使用**软件源（repositories）**——由发行版维护并验证的大型在线软件库。你不需要四处找下载链接，只需告诉系统你想要什么软件的名称，它就会自动查找、下载并安装，连同运行该软件所需的其他依赖程序一起搞定。

在 Kali（以及基于 Debian 的系统）中，管理这一切的工具是 `apt-get`。

系统已知的软件源列表保存在 `/etc/apt/sources.list` 文件中。除非你清楚自己要添加什么，否则通常不要随意改动这个文件——非官方软件源可能包含恶意软件包。

---

## 常用核心命令

### apt-get update

在安装任何软件之前，先运行这个命令。它不会安装或升级任何软件，只是刷新本地关于软件源中可用软件的列表。如果你跳过这一步就去安装软件，可能会装到旧版本或遇到报错。

```bash
ahegazy0@kali:~$ apt-get update
```

养成每次安装前先运行它的习惯。

---

### apt-get install

下载并安装软件包。Kali 会自动处理依赖关系——如果某个工具运行需要另外五个库文件，它也会一并抓下来安装。

```bash
ahegazy0@kali:~$ apt-get install wireshark
```

开始下载前它会请求确认，输入 `y` 并按回车即可。

如果想跳过确认提示：

```bash
ahegazy0@kali:~$ apt-get install -y wireshark
```

`-y` 参数会自动回答“是”。当你明确知道自己在做什么时很好用。

---

### apt-get remove

卸载已安装的软件包。

```bash
ahegazy0@kali:~$ apt-get remove wireshark
```

这会移除程序本身，但会保留其配置文件。如果你想连同配置文件一起彻底删除：

```bash
ahegazy0@kali:~$ apt-get purge wireshark
```

---

### apt-get upgrade

将所有已安装的软件包更新到最新版本。

```bash
ahegazy0@kali:~$ apt-get upgrade
```

定期运行有助于保持工具处于最新状态。升级前务必先运行 `apt-get update` 刷新软件包列表。

---

### apt-cache search

在本地软件源数据库中搜索匹配关键词的软件包。当你大概知道要找什么，但记不清确切包名时很有用。

```bash
ahegazy0@kali:~$ apt-cache search wifi
```

这会返回名称或描述中包含“wifi”的软件包列表。接下来从中挑出合适的，再按包名安装即可。

```bash
ahegazy0@kali:~$ apt-cache search wireless
ahegazy0@kali:~$ apt-cache search password crack
```

---

### git clone

许多最新、最好用的黑客工具并不在任何软件源里，而是放在 GitHub 上。`git clone` 会把整个项目的副本从 GitHub 直接下载到你的机器上。

```bash
ahegazy0@kali:~$ git clone https://github.com/username/toolname
```

这会在当前目录下创建一个与该工具同名的文件夹。大多数工具都会附带一个 `README` 文件，说明如何从那里安装和运行它们。

克隆后的常规流程如下：

```bash
ahegazy0@kali:~$ git clone https://github.com/example/tool
ahegazy0@kali:~$ cd tool
```

然后阅读 README 中的安装步骤——通常类似 `pip install -r requirements.txt`，或者直接运行 Python 脚本。

> 很多最新的工具只存在于 GitHub 上。尽早学会 `git clone`，意味着你不会被官方软件源里的内容所局限。

---

## 关于 sources.list 的一点说明

`/etc/apt/sources.list` 文件包含了系统拉取软件时所用的所有软件源 URL。你可以用以下命令查看它：

```bash
ahegazy0@kali:~$ cat /etc/apt/sources.list
```

有时你在网上会看到教程让你往这个文件里加一行，以接入第三方软件源。对此要保持谨慎。官方 Kali 软件源经过维护和验证，而随意的第三方源则没有。添加错误的软件源很容易装上你原本不打算安装的程序。

---

## 命令速查表

| **操作** | **命令** |
| --- | --- |
| 刷新软件包列表 | `apt-get update` |
| 安装软件包 | `apt-get install [name]` |
| 无需确认直接安装 | `apt-get install -y [name]` |
| 卸载软件包 | `apt-get remove [name]` |
| 卸载并清除配置文件 | `apt-get purge [name]` |
| 更新所有软件包 | `apt-get upgrade` |
| 搜索软件包 | `apt-cache search [keyword]` |
| 从 GitHub 克隆项目 | `git clone [url]` |

---

## 练习清单

- [ ] 运行 `apt-get update` 刷新软件包列表
- [ ] 使用 `apt-cache search wifi` 查看返回结果
- [ ] 从结果中挑一个工具，用 `apt-get install` 安装它
- [ ] 在 GitHub 上找一个小工具，用 `git clone` 下载到你的机器上

> 💡 _如果想做更深入的练习，我也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的习题。_

---

_下一章：模块 5 - 控制文件和目录权限（Controlling File and Directory Permissions）_
