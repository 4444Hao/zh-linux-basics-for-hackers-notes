# 黑客 Linux 基础（Linux Basics for Hackers）

## 模块 2 - 文本操作（Text Manipulation）

---

## 概述

在 Linux 中，几乎所有东西都是文本文件。系统设置、日志、配置文件、用户数据——全都是文本。一旦你学会如何搜索和处理文本，就能在成千上万行输出中抽丝剥茧，几秒钟内提取出真正需要的内容。这就是本模块的核心意义。

---

## 这对黑客技术为什么重要

扫描工具运行并产生输出时，并不会给你一份清爽的摘要。它会把所有内容一股脑倒出来——动辄成千上万行。你不可能手动逐行去看，而是要用文本工具过滤出你关心的部分。举个真实的例子：扫描完一个网络后，你可以用 `grep` 只提取包含“open”的行来找出开放端口，忽略其余所有内容。

此外，几乎所有系统配置都保存在 `/etc` 下的某个纯文本文件中。只要你能读取并编辑这些文件，就可以重新配置系统中的几乎任何功能。

---

## 常用基础命令

### cat

“concatenate（连接/串联）”的缩写。读取文件最简单的方法——直接把整个文件内容输出到屏幕上。

```bash
ahegazy0@kali:~$ cat /etc/passwd
```

> **避坑提示：** 千万不要对编译后的二进制文件运行 `cat`（例如 `cat /bin/ls`）。它会把原始字节码倾倒到屏幕上，把你的终端字体搞乱成看不懂的“天书”。如果真发生了这种情况，盲敲 `reset` 并按回车即可恢复。
它也很适合用来快速创建小文件：

```bash
ahegazy0@kali:~$ cat > targets.txt
```

运行后，逐行输入你的内容。完成后按 `Ctrl+D` 保存并退出。你输入的内容现在就在 `targets.txt` 里了。

如果想向已有文件追加内容而不覆盖原内容，请改用 `>>`：

```bash
ahegazy0@kali:~$ cat >> targets.txt
```

`>` 和 `>>` 的区别很重要：`>` 会覆盖，`>>` 是追加。

---

### grep

这是你用得最多的命令之一。`grep` 会在文件中搜索，只返回包含特定单词或模式的那些行。

```bash
ahegazy0@kali:~$ grep "password" logs.txt
```

返回 `logs.txt` 中包含“password”这个词的每一行。

实用选项：

| **选项** | **作用**                             |
| ------ | ---------------------------------- |
| `-i`   | 忽略大小写搜索（可匹配“Password”、“PASSWORD”等） |
| `-r`   | 递归搜索文件夹中的所有文件                      |
| `-n`   | 在结果旁显示行号                           |
| `-v`   | 反向匹配——显示不包含该词的行                    |

示例：

```bash
ahegazy0@kali:~$ grep -i "admin" access.log
```

会把“admin”、“Admin”、“ADMIN”全部找出来。

---

### head 与 tail

当文件非常大，而你只想看一小部分时：

```bash
ahegazy0@kali:~$ head /etc/snort/snort.conf
```

默认显示前 10 行。

```bash
ahegazy0@kali:~$ tail /var/log/syslog
```

默认显示最后 10 行。

控制显示行数的方法：

```bash
ahegazy0@kali:~$ head -n 20 file.txt     前 20 行
ahegazy0@kali:~$ tail -n 20 file.txt     最后 20 行
```

`tail` 有一个特别实用的技巧——`-f` 参数，可以实时追踪文件内容的变化：

```bash
ahegazy0@kali:~$ tail -f /var/log/syslog
```

这会让终端保持打开状态，并在文件新增内容时实时显示新行。非常适合在运行某些任务时实时查看日志。

---

### nl

为文件的输出加上行号。当你需要引用特定行时非常顺手。

```bash
ahegazy0@kali:~$ nl /etc/snort/snort.conf
```

---

### less

当文件太长、用 `cat` 看会直接刷屏过去时，改用 `less`。它可以让你逐页翻阅文件。

```bash
ahegazy0@kali:~$ less /etc/snort/snort.conf
```

操作按键：
- `Space`（空格键）—— 下一页
- `b` —— 上一页
- `/word` —— 搜索某个词
- `q` —— 退出

---

### sed

“stream editor（流编辑器）”的缩写。它会在文件中查找某个词或模式，并将其替换为其他内容。

```bash
ahegazy0@kali:~$ sed 's/mysql/MySQL/g' config.txt
```

拆解一下这串命令：
- `s/` —— 替换（substitute）
- `mysql` —— 要查找的内容
- `/MySQL/` —— 替换成的目标内容
- `g` —— 全局替换（每一处匹配都替换，而不仅是第一处）

默认情况下，`sed` 只会打印修改后的输出，并不会真正改动原文件。如果想把修改保存回文件，需要加上 `-i`：

```bash
ahegazy0@kali:~$ sed -i 's/mysql/MySQL/g' config.txt
```

> 使用 `-i` 之前先做备份。`sed -i` 会永久修改文件，且没有“撤销”按钮。

```bash
ahegazy0@kali:~$ cp config.txt config.txt.bak
ahegazy0@kali:~$ sed -i 's/old/new/g' config.txt
```

---

## 管道（pipe）——把命令串联起来

管道符 `|` 会把前一个命令的输出直接传给下一个命令作为输入。真正的威力就在这里体现出来。

![Linux 命令管道示意图](assets/linux_pipe_diagram_1789213616381.jpg)

```bash
ahegazy0@kali:~$ cat access.log | grep "failed"
```

这样你不用看整个日志，只看包含“failed”的行。

你可以按需串联任意多个管道：

```bash
ahegazy0@kali:~$ cat access.log | grep "failed" | tail -n 20
```

这条命令会读取日志，过滤出包含“failed”的行，然后只显示过滤结果中的最后 20 行。三个命令，一行搞定。

---

## 命令速查表

|**命令**|**作用**|**示例**|
|---|---|---|
|`cat file`|将整个文件内容打印到屏幕|`cat /etc/passwd`|
|`cat > file`|创建文件并输入内容|`cat > notes.txt`|
|`grep "word" file`|查找包含指定词的行|`grep "root" passwd`|
|`grep -i`|忽略大小写搜索|`grep -i "admin" log`|
|`head -n 20 file`|显示前 20 行|`head -n 20 file.txt`|
|`tail -n 20 file`|显示最后 20 行|`tail -n 20 file.txt`|
|`tail -f file`|实时追踪文件内容|`tail -f syslog`|
|`nl file`|显示带行号的文件内容|`nl config.conf`|
|`less file`|分页翻阅文件|`less bigfile.txt`|
|`sed 's/a/b/g' file`|将所有“a”替换为“b”|`sed 's/old/new/g' f`|
|`cmd1 \| cmd2`|将 `cmd1` 的输出通过管道传给 `cmd2`|`cat log \| grep fail`|

---

## 练习清单

- [ ] 前往 `/etc/snort/`，用 `less` 打开 `snort.conf`——翻阅一下，感受一下配置文件长什么样

- [ ] 使用 `grep "output" /etc/snort/snort.conf`，看看返回了哪些行

- [ ] 使用 `cat > targets.txt`，输入三个 IP 地址（每行一个），然后按 `Ctrl+D`——再用 `cat targets.txt` 读回来核对

- [ ] 尝试串联命令：`cat /etc/snort/snort.conf | grep "output" | nl`

> 💡 _如果想做更深入的练习，我也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的习题。_

_下一章：模块 3 - 网络管理（Managing Networks）_

---
