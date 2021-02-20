### 命令行工具

Scrapy是通过`scrapy`命令行工具控制的。

Scrapy工具提供了用于不同目的的命令，每个命令都有不同的参数和选项。

#### 1、配置设置

Scrapy从初始化风格的`scrapy.cfg`文件中查找配置参数，这个文件的标准位置：

1. `/etc/scrapy.cfg` 或者 `c:\scrapy\scrapy.cfg` (系统层),
2. `~/.config/scrapy.cfg` (`$XDG_CONFIG_HOME`) 和 `~/.scrapy.cfg` (`$HOME`)  (用户层)
3. Scrapy项目根目录的`scrapy.cfg` 

这三个位置的配置的优先级依次升高。

Scrapy还可以通过环境变量配置，目前支持的有：

- `SCRAPY_SETTINGS_MODULE` （[Designating the settings](https://docs.scrapy.org/en/latest/topics/settings.html#topics-settings-module-envvar)）
- `SCRAPY_PROJECT` （ [Sharing the root directory between projects](https://docs.scrapy.org/en/latest/topics/commands.html#topics-project-envvar)）
- `SCRAPY_PYTHON_SHELL` （ [Scrapy shell](https://docs.scrapy.org/en/latest/topics/shell.html#topics-shell)）

#### 2、Scrapy项目的默认结构

所有的Scrapy项目都有类似如下的结构：

```
scrapy.cfg
myproject/
    __init__.py
    items.py
    middlewares.py
    pipelines.py
    settings.py
    spiders/
        __init__.py
        spider1.py
        spider2.py
        ...
```

`scrapy.cfg`文件所在的目录被称为*项目根目录*。这个文件包含了定义项目设置的python模块的名字，比如：

```
[settings]
default = myproject.settings
```

#### 3、项目之间共享根目录

项目根目录，可以被多个Scrapy项目共享，每个项目都可以有他自己的设置模块。这种情况下，必须在`scrapy.cfg`文件的`[settings]`下，为这些设置模块定义一个或多个别名：

```
[settings]
default = myproject1.settings
project1 = myproject1.settings
project2 = myproject2.settings
```

默认情况下，`scrapy`命令行工具会使用`default`设置，可以使用`SCRAPY_PROJECT`环境变量为`scrapy`指定一个不同的项目设置：

```shell
$ scrapy settings --get BOT_NAME
Project 1 Bot
$ export SCRAPY_PROJECT=project2
$ scrapy settings --get BOT_NAME
Project 2 Bot
```

#### 4、`scrapy`工具使用

运行不带任何参数的`scrapy`命令，会显示帮助信息和可用的命令：

```shell
$ scrapy
Scrapy 2.4.1 - no active project

Usage:
  scrapy <command> [options] [args]

Available commands:
  bench         Run quick benchmark test
  commands
  fetch         Fetch a URL using the Scrapy downloader
  genspider     Generate new spider using pre-defined templates
  runspider     Run a self-contained spider (without creating a project)
  settings      Get settings values
  shell         Interactive scraping console
  startproject  Create new project
  version       Print Scrapy version
  view          Open URL in browser, as seen by Scrapy

  [ more ]      More commands available when run from project directory

Use "scrapy <command> -h" to see more info about a command
```

输出的第一行是Scrapy的版本和当前活跃的项目（如果是在Scrapy项目目录下执行命令，则显示` - project: <projectname>`）

##### 4.1、创建项目

```shell
$ scrapy startproject myproject [project_dir]
```

如果指定了`project_dir`，会在`project_dir`目录下创建Scrapy项目；如果不指定`project_dir`，`project_dir`将会是`myproject`。

##### 4.2、控制项目

进入`project_dir`后，可以使用`scrapy`命令管理项目。

比如，创建新的爬虫：

```shell
$ scrapy genspider mydomain mydomain.com
```

某些Scrapy命令（比如`crawl`）必须在Scrapy项目内部运行，详见下节。

还要记住，某些命令在项目内部运行时，可能会有一些不同的行为。比如`fecth`命令，如果被fetched的url与某些特定爬虫相关，则会使用爬虫覆盖行为（spider-overridden behabiours）（比如`user-agent`属性会覆盖user-agent）。这是有意的，因为`fetch`命令本身是用于检查爬虫是如何下载页面的。

#### 5、可用的`scrapy`命令

使用`-h`选项可以查看`scrapy`和`scrapy`命令的帮助信息：

```shell
$ scrapy <command> -h
```

命令可以分为两种，项目专用命令（只在Scrapy项目内部运行）和全局命令（没有Scrapy项目也能运行，可能在项目内部运行时行为稍有不同，因为会使用项目覆盖设置）。

全局命令：

- `startproject`
- `genspider`
- `settings`
- `runspider`
- `shell`
- `fetch`
- `view`
- `version`

项目专用命令：

- `crawl`
- `check`
- `list`
- `edit`
- `parse`
- `bench`

##### startproject

语法：`scrapy startproject <project_name> [project_dir]`

##### genspider

语法：`scrapy genspider [-t template] <name> <domain>`

作用：在当前目录或者（如果在项目内部执行）当前项目的`spiders`目录中创建一个新的爬虫。`<name>`参数被设置为爬虫的`name`，而`<domain>`参数被用于生成爬虫的`allowed_domains`属性和`start_urls`属性。

创建爬虫：

```shell
$ scrapy genspider example example.com
Created spider 'example' using template 'basic'
```

可以指定模板，使用`-l`选项查看可用的模板：

```shell
$ scrapy genspider -l
Available templates:
  basic
  crawl
  csvfeed
  xmlfeed
```

指定模板，创建爬虫：

```shell
$ scrapy genspider -t crawl scrapyorg scrapy.org
Created spider 'scrapyorg' using template 'crawl'
```

`genspider`只是根据预定义模板创建爬虫的一个快捷命令。当然，还可以通过编码创建爬虫。

##### crawl

语法：`scrapy crawl <spider>`

需要在项目内部运行，使用爬虫开启爬取。

```shell
$ scrapy crawl myspider
[ ... myspider starts crawling ... ]
```

##### check

语法：`scrapy check [-l] <spider>`

需要在项目内部运行，运行爬虫检查（contract checks）。

查看爬虫列表：

```shell
$ scrapy check -l
first_spider
  * parse
  * parse_item
second_spider
  * parse
  * parse_item
```

运行检查：

```shell
$ scrapy check
[FAILED] first_spider:parse_item
>>> 'RetailPricex' field is missing

[FAILED] first_spider:parse
>>> Returned 92 requests, expected 0..4
```

##### list

语法：`scrapy list`

需要在项目内部运行，查看当前项目中可用的爬虫。

```shell
$ scrapy list
spider1
spider2
```

##### edit

语法：`scrapy edit <spider>`

需要在项目内部运行，使用`EDITOR`环境变量或（如果没有设置）`EDITOR`设置中定义的编辑器来编辑指定的爬虫。

只是调出编辑器的快捷命令，当然可以直接在IDE中编辑爬虫。

##### fetch

语法： `scrapy fetch <url>`

使用Scrapy downloader下载指定的URL，并将内容写到标准输出。

有趣的地方是，这个命令是按照爬虫下载页面的方式去获取页面的。比如，如果爬虫有一个`USER_AGENT`属性覆盖了User Agent，它就会使用`USER_AGENT`。所以，这个命令可以用来“查看”爬虫是如何获取某个页面的。

如果在项目外使用这个命令，将不会应用特殊的爬虫行为，而是仅仅使用默认的Scrapy downloader设置。

支持的选项：

- `--spider=SPIDER`：跳过爬虫自动检测，并强制使用指定的爬虫
- `--headers`：输出响应的HTTP headers而不是响应的body
- `--no-redirect`：不追踪HTTP 3xx的转发（默认是追踪转发的）

```shell
$ scrapy fetch --nolog http://www.example.com/some/page.html
[ ... html content here ... ]

$ scrapy fetch --nolog --headers http://www.example.com/
{'Accept-Ranges': ['bytes'],
 'Age': ['1263   '],
 'Connection': ['close     '],
 'Content-Length': ['596'],
 'Content-Type': ['text/html; charset=UTF-8'],
 'Date': ['Wed, 18 Aug 2010 23:59:46 GMT'],
 'Etag': ['"573c1-254-48c9c87349680"'],
 'Last-Modified': ['Fri, 30 Jul 2010 15:30:18 GMT'],
 'Server': ['Apache/2.2.3 (CentOS)']}
```

##### view

语法： `scrapy view <url>`

在浏览器中打开指定的URL，就像Scrapy爬虫将会“看到”的那样。**有时，爬虫看到的页面与普通用户看到的不同**，可以使用这个命令检查爬虫“看到”的页面并确认是否和预期的一样。

支持的选项：

- `--spider=SPIDER`
- `--no-redirect`

```shell
$ scrapy view http://www.example.com/some/page.html
[ ... browser starts ... ]
```

##### shell

语法： `scrapy shell [url]`

为指定的URL启动Scrapy shell，如果不指定URL则是空。也支持UNIX风格的本地文件路径，支持相对路径`./`或`../`，也支持绝对路径。

支持的选项：

- `--spider=SPIDER`
- `-c code`：在shell中执行code，打印结果并退出
- `--no-redirect`：不追踪HTTP 3xx的转发（默认是追踪转发的）；只会影响作为在命令行参数传递的URL；一旦进入了shell，`fetch(url)`会默认的跟踪HTTP转发。

```shell
$ scrapy shell http://www.example.com/some/page.html
[ ... scrapy shell starts ... ]

# 执行代码
$ scrapy shell --nolog http://www.example.com/ -c '(response.status, response.url)'
(200, 'http://www.example.com/')

# shell follows HTTP redirects by default
$ scrapy shell --nolog http://httpbin.org/redirect-to?url=http%3A%2F%2Fexample.com%2F -c '(response.status, response.url)'
(200, 'http://example.com/')

# you can disable this with --no-redirect
# (only for the URL passed as command line argument)
$ scrapy shell --no-redirect --nolog http://httpbin.org/redirect-to?url=http%3A%2F%2Fexample.com%2F -c '(response.status, response.url)'
(302, 'http://httpbin.org/redirect-to?url=http%3A%2F%2Fexample.com%2F')
```

##### parse

语法： `scrapy parse <url> [options]`

需要在项目内部运行，获取指定的URL并用处理它的爬虫来进行解析，使用`--callback`选项传递的方法，或者，如果不指定回调方法，则是用`prase`方法。

支持的选项：

- `--spider=SPIDER`
- `--a NAME=VALUE`：设置爬虫参数（可能重复）
- `--callback`或`-c`：用作回调来解析响应的爬虫方法
- `--meta`或`-m`：传递给回调请求的额外的请求meta。必须是合法的json字符串，比如`--meta='{"foo":"bar"}'`
- `--cbkwargs`：传递给回调的额外的关键字参数。必须是合法的json字符串，比如`--cbkwargs='{"foo":"bar"}'`
- `--pipelines`：通过管道来处理items
- `--rules`或`-r`：使用`CrawlSpider` rules来发现用于解析响应的回调（即，爬虫方法）
- `--noitems`：不显示抓取得items
- `--nolinks`：不显示提取的链接
- `--nocolour`：不使用pygments来为输出着色
- `--depth`或者`-d`：请求递归地追踪深度层级（默认：1）
- `--verbose`或者`-v`：显示每个深度层级的信息
- `--output`或者`-o`：下载抓取的items到一个文件

```shell
$ scrapy parse http://www.example.com/ -c parse_item
[ ... scrapy log lines crawling example.com spider ... ]

>>> STATUS DEPTH LEVEL 1 <<<
# Scraped Items  ------------------------------------------------------------
[{'name': 'Example item',
 'category': 'Furniture',
 'length': '12 cm'}]

# Requests  -----------------------------------------------------------------
[]
```

##### settings

语法： `scrapy settings [options]`

获取Scrapy设置的值

如果在项目内部使用，则使用项目设置的值，否则显示某个设置的默认Scrapy值。

```shell
$ scrapy settings --get BOT_NAME
scrapybot
$ scrapy settings --get DOWNLOAD_DELAY
0
```

##### runspider

语法： `scrapy runspider <spider_file.py>`

运行Python文件中独立的爬虫，而不用创建项目。

```shell
$ scrapy runspider myspider.py
[ ... spider starts crawling ... ]
```

##### version

语法： `scrapy version [-v]`

打印Scrapy版本，如果加上`-v`选项，还会打印Python、Twisted和Platform信息。

##### bench

语法： `scrapy bench`

运行快速的benchmark测试。

#### 6、自定义项目命令

可以使用`COMMANDS_MODULE`设置来自定义项目命令。