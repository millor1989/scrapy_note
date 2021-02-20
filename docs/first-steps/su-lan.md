### 速览

尽管Scrapy最初是为网站获取（web scraping）而设计的，它也能用于使用APIs（比如Amazon Associates Web Services）提取数据或者作为一个通用目的的网络爬虫（web crawler）。

#### 1、Scrapy Spider例子

如下代码是一个爬取网站[http://quotes.toscrape.com](http://quotes.toscrape.com/)名言的爬虫，分页的：

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = 'quotes'
    start_urls = [
        'http://quotes.toscrape.com/tag/humor/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'author': quote.xpath('span/small/text()').get(),
                'text': quote.css('span.text::text').get(),
            }

        next_page = response.css('li.next a::attr("href")').get()
        if next_page is not None:
            yield response.follow(next_page, self.parse)
```

将其保存为`quotes_spider.py`文件，使用`runspider`命令运行这个爬虫：

```shell
$ scrapy runspider quotes_spider.py -o quotes.jl
```

执行结束后，会得到一个名为`quotes.jl`内容为名言的JSON Lines格式的文件，包含了文本和作者：

```ascii
{"author": "Jane Austen", "text": "\u201cThe person, be it gentleman or lady, who has not pleasure in a good novel, must be intolerably stupid.\u201d"}
{"author": "Steve Martin", "text": "\u201cA day without sunshine is like, you know, night.\u201d"}
{"author": "Garrison Keillor", "text": "\u201cAnyone who thinks sitting in church can make you a Christian must also think that sitting in a garage can make you a car.\u201d"}
...
```

#### 2、爬虫的运行

当执行命令`scrapy runspider quotes_spider.py`，Scrapy会在`quotes_spider.py`中查找Spider的定义并使用爬虫引擎（crawler engine）来运行它。

  通过执行属性`start_urls`中定义的URLs（这个例子中，只有*humor*类别名言的URL）请求来开始爬取，并且调用了默认的回调方法（callback method）`parse`，将请求的响应对象`response`作为`parse`的参数。在`parse`回调中，使用CSS Selector来遍历名言元素，返回一个包含了名言文本和作者的Python dict。遍历完成后，查找下一页的链接并使用相同的`parse`方法作为回调方法去调度下一次请求。

通过这个例子，可以看到**Scrapy的主要优点：异步地调度和处理请求**。这意味着Scrapy不需要等待请求结束并处理完成，它可以同时发送另一个请求或者做其它的事情。这也就意味着，即使某些请求失败或者请求的处理发生错误，其它的请求也可以继续执行。

在能够进行非常快速的爬取（以容错的方式，同时发送多个并发请求）的同时，Scrapy也可以通过一些设置进行优雅的（politeness）爬取。可以设置每次请求之间的下载延时（download delay），限制每个域名或者每个IP并发请求的数量，或者使用auto-throttling扩展来自动地进行这些配置。

#### 3、其它

这里的例子只展示了如何使用Scrapy从网站提取和保存项目。Scrapy还有很多强大的特性使得抓取更容易和高效，比如：

- 内置了使用扩展的**CSS selectors**和**XPath**表达式来筛选和提取HTML/XML的数据，并具有使用**正则表达式**进行提取的helper方法。
- 一个**交互性的shell控制台**（IPython aware）以试验抓取数据的CSS和XPath表达式，当编写和debug爬虫时非常有用。
- 内置了对多种格式feed export的支持，并支持将它们保存到多种后端（backend，FTP，S3，本地文件系统）
- 强大的编码支持和自动检测功能，用于处理外来的，非标准的和损坏的编码声明。
- 强大的扩展性支持，可以在使用**signals**和定义好的API（**middlewares**，**extensions**，和**pipelines**）进行扩展。
- 用于处理的丰富的内置扩展（extensions）和中间件（middlewares）：
  - cookies和session处理
  - HTTP特性，比如压缩（compression）、认证（authentication）、缓存（caching）
  - user-agent欺骗（spoofing）
  - robots.txt
  - 爬取深度限制
  - 等等
- 一个**Telnet控制台**，在Scrapy进程中挂载一个Python控制台，以检验和debug爬虫
- 还有其它好东西，比如从Sitemaps和XML/CSV Feed爬取网站的可重复使用的爬虫，与爬取项目（scraped items）相关的用于自动地下载图片（或其他媒体）的媒体管道（media pipeline），缓存DNS解析器（caching DNS resolver），等等。