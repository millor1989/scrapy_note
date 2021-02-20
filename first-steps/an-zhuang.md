### 安装

#### 1、安装Scrapy

pip安装：

```shell
$ pip install Scrapy
```

#### 1.1、Scrapy依赖

Scrapy使用纯粹的Python编写的，它依赖一些关键的Python模块：

- lxml，高效地的XML和HTML解析器
- parsel，基于lmxl编写的HTML/XML数据提取库
- w3lib，用于处理URLs和网页编码的多用途helper
- twisted，一个异步网络框架
- cryptography和pyOpenSSL，用以处理多种网络层安全性的需要

Scrapy需要的最小版本是：

- Twisted 14.0
- lxml 3.4
- pyOpenSSL 0.14

#### 1.2、使用虚拟环境（推荐）

对于所有的平台，都推荐将Scrapy安装在一个虚拟环境中。

Python模块可以全局安装（即，系统层，system wide），也可以安装在用户空间。不推荐系统层的安装Scrapy。

推荐将Scrapy安装在虚拟环境中（`venv`）。虚拟环境可以不与已经安装的Python系统模块冲突（可能会破坏系统工具和脚本），仍然可以使用`pip`正常地安装。详见[虚拟环境和模块](https://docs.python.org/3/tutorial/venv.html#tut-venv)。创建了虚拟环境后，可以在里面用`pip`进行安装。

