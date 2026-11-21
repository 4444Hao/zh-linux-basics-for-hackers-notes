# 黑客 Linux 基础（Linux Basics for Hackers）

## 模块 8 - Bash 脚本编写（Bash Scripting）

---

## 概述

如果黑客一遍又一遍地手动输入相同的命令，就是在浪费时间。脚本编写正是实现自动化的方式——你把命令写进一个文件里一次，之后直接运行这个文件即可。本模块将介绍如何编写 Bash 脚本，从最基础的“Hello, world”到真正能够接收输入并做有用操作的脚本。

---

## 什么是脚本

脚本就是一个包含命令列表的纯文本文件。当你运行它时，Bash 会逐行读取并按顺序执行，就像你自己手动输入每条命令一样。不同之处在于，这一切会在几秒内完成，而且你再也不用重复敲那些命令了。

你在终端里能运行的任何命令都可以放进脚本中。`ls`、`ping`、`nmap`、`grep`——全都可以。你还可以结合逻辑判断来使用它们：比如“如果这段输出包含某个字符串，就运行另一条命令”。真正的威力就源于此。

---

## Shebang 行

每个 Bash 脚本的第一行都必须写上这一句：

```bash
#!/bin/bash
```

这叫做 **shebang**（或 hashbang）。它会明确告诉系统使用哪个程序来解释这个文件。没有它，Linux 可能不知道该如何处理该文件——或者会用错 shell，导致脚本以莫名其妙的方式报错。

务必把它放在第一行。前面不要有空行，也不要有其他内容。

---

## 创建你的第一个脚本

打开文本编辑器，创建一个名为 `myscript.sh` 的文件：

```bash
#!/bin/bash
echo "Hello, world"
```

保存文件。现在，在运行它之前，你需要先赋予它可执行权限：

```bash
ahegazy0@kali:~$ chmod 755 myscript.sh
```

这会让你拥有把它作为程序运行的权限。如果跳过这一步，Linux 会拒绝执行，并提示“Permission denied（权限被拒绝）”。

现在运行它：

```bash
ahegazy0@kali:~$ ./myscript.sh
Hello, world
```

开头的 `./` 意思是“从当前目录运行这个文件”。之所以需要它，是因为当前目录通常不在你的 `PATH` 中，否则 Linux 找不到这个脚本。

---

## echo - 输出到屏幕

`echo` 会把你传给它的内容打印到终端上，这是你的脚本向你反馈信息的方式。

```bash
ahegazy0@kali:~$ echo "Scan starting..."
ahegazy0@kali:~$ echo "Done."
```

你也可以用 `echo` 打印变量的值：

```bash
ahegazy0@kali:~$ echo "Your username is: $USER"
Your username is: kali
```

---

## 脚本中的变量

变量可以让你存储数据并重复使用。赋值时像下面这样写——`=` 左右不要留空格：

```bash
ahegazy0@kali:~$ name="Bob"
ahegazy0@kali:~$ echo "Hello, $name"
Hello, Bob
```

当你想使用变量中存储的值时，在变量名前加上 `$`；当你在给变量赋值时，不要加 `$`。

---

## read - 获取用户输入

`read` 会暂停脚本，等待用户输入内容，然后把输入的内容存进一个变量中。

```bash
#!/bin/bash
echo "What is your name?"
read name
echo "Hello, $name"
```

运行后会停下来等待。你输入“Alice”并按回车：

```
What is your name?
Alice
Hello, Alice
```

你也可以用 `-p` 参数（prompt，提示语）在一行内完成：

```bash
ahegazy0@kali:~$ read -p "Enter your name: " name
```

更简洁，效果完全一样。

---

## 实用示例 - Ping 扫描器

这是一个简单的脚本，它会询问一个 IP 地址并对其执行 ping。这是你实际会用到的那种小工具：

```bash
#!/bin/bash
read -p "Enter IP address to ping: " target
echo "Pinging $target..."
ping -c 4 $target
```

`-c 4` 告诉 `ping` 只发送 4 个数据包就停止；不加这个参数，`ping` 会一直运行下去。

把它保存为 `pinger.sh`，执行 `chmod 755 pinger.sh`，然后用 `./pinger.sh` 运行。

---

## 注释

任何以 `#` 开头的行都是注释。Bash 会完全忽略它。用注释来解释脚本在做什么，尤其是那些不够直观的部分。

```bash
#!/bin/bash
# This script pings a target IP address
# Written for practice - Module 8

read -p "Enter target IP: " target
ping -c 4 $target   # send 4 packets only
```

注释是写给现在的你（以及以后读这份脚本的任何人）看的。写注释时就当自己两周后就会忘掉这段逻辑——事实也确实如此。

---

## 条件逻辑 - if/else

一旦脚本能够做判断，它就开始变得真正有用了。基础语法如下：

```bash
#!/bin/bash
read -p "Enter a number: " num

if [ $num -gt 10 ]; then
echo "That number is greater than 10"
else
echo "That number is 10 or less"
fi
```

`fi` 用于闭合 `if` 代码块（其实就是把 `if` 反过来写）。`-gt` 的意思是“大于”（greater than）。常见的比较运算符如下：

| **运算符** | **含义** |
| --- | --- |
| `-eq` | 等于 |
| `-ne` | 不等于 |
| `-gt` | 大于 |
| `-lt` | 小于 |
| `-ge` | 大于或等于 |
| `-le` | 小于或等于 |

如果在方括号中比较字符串，请使用 `=` 和 `!=`。

---

## 循环 - 重复执行操作

`for` 循环可以让一段命令块执行多次：

```bash
#!/bin/bash
for i in 1 2 3 4 5; do
echo "Pinging 192.168.1.$i"
ping -c 1 192.168.1.$i
done
```

这会依次 ping 五个不同的 IP 地址。这本质上就是一个原始的网络扫描器。像 `nmap` 这样的专业扫描器也会做类似的成千上万次探测，只是外围逻辑复杂得多，但核心思路是一样的。

---

## 让脚本真正实用——建议遵循的结构

值得保留下来的脚本通常长这样：

```bash
#!/bin/bash
# Script name and what it does
# Your name, date

# --- Variables ---
target=""

# --- Input ---
read -p "Enter target: " target

# --- Main logic ---
echo "Running scan on $target..."
nmap -sV $target

# --- Done ---
echo "Scan complete."
```

分区清晰、需要说明的地方加上注释、逻辑自上而下顺畅推进。你不需要完全照搬，但保持一定结构能避免脚本写到 20 行以上时变得难以阅读。

---

## 文件权限回顾 - chmod

当你创建一个脚本时，它默认只是一个普通文本文件。要运行它，需要设置可执行权限。

```bash
ahegazy0@kali:~$ chmod 755 myscript.sh
```

`755` 的含义：

- `7` - 所有者可读、可写、可执行
- `5` - 所属组可读、可执行
- `5` - 其他所有人可读、可执行

对于只需要你自己运行的个人脚本，用 `chmod 700` 也可以——只有你自己能对它做任何操作。

---

## 命令速查表

| **命令 / 概念** | **作用** |
| --- | --- |
| `#!/bin/bash` | Shebang——每个脚本的第一行都必须写 |
| `echo "text"` | 将文本打印到屏幕上 |
| `read var` | 接收用户输入并存入变量 |
| `read -p "prompt" var` | 作用相同，但可附带行内提示语 |
| `name="value"` | 给变量赋值 |
| `$name` | 使用变量的值 |
| `chmod 755 file.sh` | 让脚本具有可执行权限 |
| `./script.sh` | 运行当前目录下的脚本 |
| `# comment` | Bash 会忽略的行——写给人看的备注 |
| `if [ ] then / fi` | 条件逻辑 |
| `for x in ... do / done` | 遍历列表执行循环 |

---

## 练习清单

- [ ] 编写一个脚本，询问你的名字并打印“Hello, [name]”
- [ ] 使用 `chmod 755` 赋予执行权限，并用 `./` 运行它
- [ ] 编写第二个脚本，询问一个 IP 地址并对其运行 `ping -c 4`
- [ ] 调试成功后，修改它，使其循环遍历 `192.168.1.1` 到 `192.168.1.5` 并逐一 ping

这个 ping 循环练习值得认真做一遍。它是一种真实的技巧，理解它的原理后，以后看 `nmap` 的输出也会更顺畅。

> 💡 *如果想做更深入的练习，我也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的习题。*

---

_下一章：模块 9 - 归档与压缩（Archiving & Compression）_
