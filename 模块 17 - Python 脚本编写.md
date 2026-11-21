# 黑客 Linux 基础（Linux Basics for Hackers）

## 模块 17 - Python 脚本编写（Python Scripting）

---

## 概述

面对简单的自动化和快捷的单行命令，Bash 脚本能带你走很远；但当事情变得更复杂时——比如需要解析数据、处理网络交互、构建能做逻辑判断的工具，或者看懂别人编写的漏洞利用（exploit）代码——你就需要 Python 了。本模块将介绍从事安全工作所需的基础 Python 知识。

---

## 为什么安全工作选 Python

很长一段时间以来，Python 一直是安全与黑客领域的主流语言，原因并不复杂：

- 语法清晰易读——你能很快看懂一个脚本在做什么
- 标准库自带网络、文件 I/O、加密等功能，无需额外安装
- 针对安全任务的第三方库成千上万
- 你在网上能找到的大多数漏洞利用、工具和概念验证（PoC）代码都是用 Python 写的
- 只要能跑 Linux 的地方，它就能运行

即便在你还没开始自己写工具之前，能够看懂 Python 代码就已经非常实用了。你遇到的大多数漏洞利用都是 Python 脚本，如果看不懂，就等于在“盲干”。

---

## Python 2 与 Python 3

Python 2 已经淘汰——自 2020 年起已正式停止官方支持。你写的所有代码都应该是 Python 3，Kali 默认预装的就是 Python 3。当你看到老教程里写着不带括号的 `print "hello"` 时，那是 Python 2 的语法；在 Python 3 中则是 `print("hello")`。

查看版本：

```bash
ahegazy0@kali:~$ python3 --version
```

运行 Python 3 解释器：

```bash
ahegazy0@kali:~$ python3
```

这会让你进入交互式命令行，可以直接输入 Python 代码，适合测试小片段；输入 `exit()` 或按 `Ctrl+D` 退出。

---

## 你的第一个脚本

创建一个名为 `hello.py` 的文件：

```python
#!/usr/bin/env python3

print("Hello, world")
```

这里的 shebang 行和 Bash 略有不同——`/usr/bin/env python3` 会在系统中查找 Python 3 解释器的位置，无论它装在哪里，这比写死路径更具可移植性。

运行它：

```bash
ahegazy0@kali:~$ python3 hello.py
```

或者赋予可执行权限后直接运行：

```bash
ahegazy0@kali:~$ chmod 755 hello.py
ahegazy0@kali:~$ ./hello.py
```

---

## 变量与数据类型

Python 是动态类型语言——你不需要声明类型，只需赋值，Python 会自动判断。

```python
name = "Alice"           # 字符串（string）
port = 80                # 整数（integer）
pi = 3.14                # 浮点数（float）
active = True            # 布尔值（boolean）
```

**字符串（Strings）** - 文本内容，始终放在引号中：

```python
target = "192.168.1.1"
print("Scanning: " + target)
print(f"Scanning: {target}")    # f-string，更清爽的变量嵌入写法
```

**整数（Integers）** - 整数，常用于端口号、计数、索引：

```python
port = 22
print(port + 1)    # 23
```

**列表（Lists）** - 有序集合，类似数组：

```python
ports = [22, 80, 443, 3306]
print(ports[0])      # 22 - 索引从 0 开始
print(ports[-1])     # 3306 - 负索引从末尾往前数
```

**字典（Dictionaries）** - 键值对，类似查询表：

```python
user = {"username": "admin", "password": "password123", "role": "root"}
print(user["username"])    # admin
```

字典在安全脚本中非常常见——你经常会看到它用来存放解析后的数据、HTTP 请求头、配置项等。

---

## 获取用户输入

```python
target = input("Enter target IP: ")
print("Scanning " + target)
```

`input()` 会暂停等待，然后把用户输入的内容存为字符串。如果你需要把它当作数字使用：

```python
port = int(input("Enter port: "))
```

`int()` 会把字符串转换成整数。如果没有这一步转换，`"80" + 1` 就会报错——Python 不会在背后偷偷混用不同类型。

---

## 条件分支

```python
password = input("Enter password: ")

if password == "secretpass":
    print("Access granted")
elif password == "admin":
    print("Admin access")
else:
    print("Wrong password")
```

Python 使用缩进来划分代码块，没有花括号，标准缩进是 4 个空格。如果缩进不统一，Python 就会报错——这是几乎所有从其他语言转过来的新手最容易踩的坑。

常见比较运算符：

| **运算符** | **含义** |
| --- | --- |
| `==` | 等于 |
| `!=` | 不等于 |
| `>` `<` | 大于 / 小于 |
| `>=` `<=` | 大于等于 / 小于等于 |
| `in` | 检查某个值是否存在于列表或字符串中 |

---

## 循环

**For 循环** - 遍历列表或数值范围：

```python
ports = [22, 80, 443]
for port in ports:
    print(f"Checking port {port}")
```

```python
for i in range(1, 256):
    print(f"192.168.1.{i}")
```

`range(1, 256)` 会生成 1 到 255 的数字，这就是构建基础 IP 扫描范围的方式。

**While 循环** - 条件为真时持续执行：

```python
attempts = 0
while attempts < 3:
    password = input("Password: ")
    if password == "secret":
        print("Access granted")
        break
    attempts += 1
print("Too many attempts")
```

`break` 会立即跳出循环；`continue` 会跳过当前这次循环，进入下一次迭代。

---

## 函数

函数让你可以把一段代码写好一次，然后在需要时通过名称反复调用：

```python
def scan_port(ip, port):
    print(f"Scanning {ip}:{port}")

scan_port("192.168.1.1", 80)
scan_port("192.168.1.1", 443)
```

函数也可以返回值：

```python
def add(a, b):
    return a + b

result = add(3, 4)
print(result)    # 7
```

结构良好的脚本会把逻辑放进函数里，然后在文件末尾调用它们，这样代码既易读又方便复用。

---

## 导入库

Python 真正的威力来自它的库。你可以通过 `import` 把它们引入进来：

```python
import socket
import os
import sys
```

**socket** - 用于网络连接，是网络工具的基础：

```python
import socket

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("google.com", 80))
print("Connected")
s.close()
```

这会向 `google.com` 的 80 端口建立一条 TCP 连接——和浏览器通过 HTTP 访问网站时做的事情一样。

**os** - 用于与操作系统交互：

```python
import os

os.system("ls -la")           # 运行 shell 命令
cwd = os.getcwd()             # 获取当前目录
files = os.listdir(".")       # 列出当前目录下的文件
```

**sys** - 用于系统级事务，例如命令行参数：

```python
import sys

print(sys.argv)               # 传给脚本的参数列表
# python3 script.py 192.168.1.1 80
# sys.argv = ['script.py', '192.168.1.1', '80']
```

这让你在运行脚本时可以通过参数传入目标，而不是把值写死在代码里。

---

## 安装第三方库

标准库已经覆盖了很多内容，但 Python 生态还有数十万个用于特定任务的第三方包：

```bash
ahegazy0@kali:~$ pip3 install requests
ahegazy0@kali:~$ pip3 install scapy
ahegazy0@kali:~$ pip3 install paramiko
```

- `requests` - 发送 HTTP 请求比直接用 `socket` 更清爽
- `scapy` - 强大的数据包构造与分析工具
- `paramiko` - 在 Python 中建立 SSH 连接

安装完成后即可导入使用：

```python
import requests

response = requests.get("http://example.com")
print(response.status_code)
print(response.text)
```

---

## 实用示例 - 基础端口扫描器

这个例子把前面大部分知识点串起来，做成了一个真正能用的小工具：

```python
#!/usr/bin/env python3
import socket
import sys

def scan_port(ip, port):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(1)
    result = s.connect_ex((ip, port))   # 连接成功返回 0
    s.close()
    return result == 0

target = input("Enter target IP: ")
print(f"\nScanning {target}...\n")

for port in range(1, 1025):
    if scan_port(target, port):
        print(f"Port {port} is OPEN")

print("\nScan complete.")
```

`connect_ex` 会尝试连接并返回错误码，而不是直接抛出异常——返回 `0` 表示成功（端口开放）；`settimeout(1)` 表示每个端口最多等 1 秒就继续下一个。

这是 `nmap` 工作原理的简化版。看懂它之后，再看 `nmap` 的输出会明白得多。

---

## 错误处理

事情总会出现意外：网络连接失败、文件不存在、用户输入输错了。Python 使用 `try/except` 来优雅地处理这些情况：

```python
try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(("192.168.1.1", 80))
    print("Connected")
except socket.error as e:
    print(f"Connection failed: {e}")
finally:
    s.close()
```

`try` - 尝试执行这段代码 `except` - 如果出错，改执行这里 `finally` - 无论如何都会执行这里（清理代码通常放在这里）

没有错误处理时，一次连接失败就可能让整个脚本崩溃；加上之后，脚本可以继续运行并告诉你哪里出了问题。

---

## 命令速查表

| **命令** | **作用** |
| --- | --- |
| `python3 script.py` | 运行 Python 脚本 |
| `python3` | 打开交互式 Python 命令行 |
| `pip3 install [package]` | 安装第三方库 |
| `pip3 list` | 显示已安装的包 |
| `pip3 show [package]` | 显示已安装包详细信息 |

---

## Python 速查表

| **概念** | **语法** |
| --- | --- |
| 打印输出 | `print("text")` |
| 变量 | `name = "value"` |
| 用户输入 | `x = input("prompt: ")` |
| 字符串格式化 | `f"Hello {name}"` |
| 条件分支 | `if x == y:` / `else:` |
| For 循环 | `for item in list:` |
| While 循环 | `while condition:` |
| 函数 | `def name(params):` |
| 导入模块 | `import socket` |
| 列表 | `items = [1, 2, 3]` |
| 字典 | `d = {"key": "value"}` |
| 异常处理 | `try:` / `except Error as e:` |

---

## 练习清单

- [ ] 编写一个脚本，询问你的名字并使用 f-string 打印问候语
- [ ] 编写一个脚本，循环遍历 20–25 端口并逐一打印
- [ ] 写出上面的端口扫描器，针对 `127.0.0.1`（你自己的机器）进行测试，看看本机开放了哪些端口
- [ ] 编写一个脚本，询问密码并根据写死的值打印“correct”或“incorrect”
- [ ] 熟练掌握上述内容后，为端口扫描器加上错误处理，使网络错误不会导致整个扫描崩溃

> 💡 *如果想做更深入的练习，我也建议完成官方 **Linux Basics for Hackers** 书中每章末尾的习题。*

---

## 接下来往哪里走

本模块只是一个起点。书到这里就结束了，但用于安全工作的 Python 还可以走得更深：

- **Scapy** - 构造并发送自定义数据包，在数据包层面编写自己的扫描器和嗅探器
- **Paramiko** - 自动化 SSH 连接，适合用脚本批量操作远程系统
- **Requests + BeautifulSoup** - 网页抓取与 HTTP 交互
- **Subprocess** - 在 Python 中调用系统命令并捕获输出
- 阅读现有的漏洞利用代码 - GitHub 上大多数 CVE 概念验证都是 Python 代码；能够阅读并修改它们，是这个领域最实用的技能之一

往后的学习节奏和整个课程一样：理解概念、运行命令、在虚拟机里折腾“搞坏”它、遇到问题查资料。这才是真正把知识学到手的方式。

---

_《Linux Basics for Hackers》结束——全部 17 个模块已完成。_
