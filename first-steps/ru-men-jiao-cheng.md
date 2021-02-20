### 入门教程

以爬取[quotes.toscrape.com](http://quotes.toscrape.com/)为例

#### 1、创建项目

进入想要保存项目的目录，运行如下命令：

```shell
$ scrapy startproject tutorial
```

运行后，会创建一个`tutorial`目录和如下的内容：

```
tutorial/
    scrapy.cfg            # 部署配置文件

    tutorial/             # 项目的Python模块，从这里开始导入代码
        __init__.py

        items.py          # project items definition file

        middlewares.py    # project middlewares file

        pipelines.py      # project pipelines file

        settings.py       # project settings file

        spiders/          # 稍后放置爬虫的目录
            __init__.py
```

#### 2、第一个爬虫

爬虫（Spiders）是指用户定义的，Scrapy用来从网站（或一组网站）获取信息的那些类。**必须是`Spider`的子类**，并定义了要进行的初始请求，以及如何解析下载的页面内容以提取数据，追踪页面的链接（follow links）是可选的。

如下为第一个爬虫的代码，保存到`tutorial/spiders`目录下的`quotes_spider.py`文件中：

```python
class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        urls = [
            'http://quotes.toscrape.com/page/1/',
            'http://quotes.toscrape.com/page/2/',
        ]
        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse)

    def parse(self, response):
        page = response.url.split("/")[-2]
        filename = f'quotes-{page}.html'
        with open(filename, 'wb') as f:
            f.write(response.body)
        self.log(f'Saved file {filename}')
```

爬虫`quotes`继承了`scrapy.Spider`并定义了一些属性和方法：

- `name`：区分Spider。在一个项目中必须是唯一的。

- `start_requests()`：必须返回一个可迭代的Requests（可以反回一个请求集合或者写一个生成器函数），Spider从它开始爬取。后续的请求是从这些初始请求中依次生成的。

- `parse()`：对于每个请求，都会调用`parse()`方法来处理下载的响应。响应参数是一个`TextResponse`实例，它持有页面内容的并有处理内容的方法。

  `parse()`方法通常解析响应，将获取的数据提取为dict，并找出新的URLs来追踪并对它们创建新的请求（`Request`）。

##### 2.1、运行爬虫

在项目的顶层目录运行命令：

```shell
$ scrapy crawl quotes
```

这个命令会执行名为`quotes`的爬虫，会得到类似如下的输出：

```ascii
... (omitted for brevity)
2016-12-16 21:24:05 [scrapy.core.engine] INFO: Spider opened
2016-12-16 21:24:05 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
2016-12-16 21:24:05 [scrapy.extensions.telnet] DEBUG: Telnet console listening on 127.0.0.1:6023
2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (404) <GET http://quotes.toscrape.com/robots.txt> (referer: None)
2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/> (referer: None)
2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/2/> (referer: None)
2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-1.html
2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-2.html
2016-12-16 21:24:05 [scrapy.core.engine] INFO: Closing spider (finished)
...
```

执行结束，会在当前目录下创建两个文件：*quotes-1.html*和*quotes-2.html*，分别保存了对应URLs的内容，就像`parse()`方法指示的一样。

Scrapy调度Spider的`start_requests`方法返回的`scrapy.Request`对象。每当收到请求的响应，它会实例化`Response`对象并调用与请求相关的回调方法（本例中，是`parse`方法）传递响应作为参数。

除了实现`start_requests()`方法来从URLs生成`scrapy.Request`对象，还可以使用URLs集合定义一个`start_urls`类属性。这个属性会被默认的`start_requests()`实现用来创建爬虫的初始请求：

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"

    start_urls = [
        'http://quotes.toscrape.com/page/1/',
        'http://quotes.toscrape.com/page/2/',
    ]

    def parse(self, response):
        page = response.url.split("/")[-2]
        filename = f'quotes-{page}.html'
        with open(filename, 'wb') as f:
            f.write(response.body)
```

即使不明确地告诉Scrapy使用`parse()`方法，`parse()`方法也会被调用来处理这些URLs的请求。因为`parse()`是Scrapy的默认回调方法，当没有明确地指定回调方法时就会调用它。

##### 2.2、提取数据

学习用Scrapy提取数据的最好方式是使用Scrapy shell。运行命令：

```shell
$ scrapy shell 'http://quotes.toscrape.com/page/1/'
```

###### 注意，命令行运行Scrapy shell时<u>一定要用引号（Windows系统使用双引号）包括urls</u>，否则urls包含的参数（即，`&`字符）将不生效。

命令执行后会类似如下结果：

```
[ ... Scrapy log here ... ]
2016-09-19 12:09:27 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/> (referer: None)
[s] Available Scrapy objects:
[s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
[s]   crawler    <scrapy.crawler.Crawler object at 0x7fa91d888c90>
[s]   item       {}
[s]   request    <GET http://quotes.toscrape.com/page/1/>
[s]   response   <200 http://quotes.toscrape.com/page/1/>
[s]   settings   <scrapy.settings.Settings object at 0x7fa91d888c10>
[s]   spider     <DefaultSpider 'default' at 0x7fa91c8af990>
[s] Useful shortcuts:
[s]   shelp()           Shell help (print this help)
[s]   fetch(req_or_url) Fetch request (or URL) and update local objects
[s]   view(response)    View response in a browser
```

此时，在命令行窗口可以交互式的操作response对象（和其它可用Scrapy对象）。

使用**CSS选择器**：

```python
>>> response.css('title')
[<Selector xpath='descendant-or-self::title' data='<title>Quotes to Scrape</title>'>]
```

结果是一个类似集合的对象`SelectorList`，它表示一个`Selector`对象（包含了XML\HTML元素）集合，并且可以对它进行更进一步的查询——更细粒度地进行筛选或者提取数据。

提取标题文本：

```python
>>> response.css('title::text').getall()
['Quotes to Scrape']
```

对CSS查询增加**`::text`**表示只筛选`<title>`元素中的文本。如果不指定`::text`，会得到完整的`title`元素，包括它的标签：

```python
>>> response.css('title').getall()
['<title>Quotes to Scrape</title>']
```

另外**`getall()`**返回的结果是一个集合，因为一个选择器有可能返回多个结果，所以提取所有的可能结果。如果要提取第一个结果，就像这个例子，可以：

```python
>>> response.css('title::text').get()
'Quotes to Scrape'
```

也可以用如下的写法：

```python
>>> response.css('title::text')[0].get()
'Quotes to Scrape'
```

直接对`SelectorList`实例使用**`.get()`**可以避免`IndexError`，并且当没有任何匹配的筛选结果时会返回`None`。

经验是：对于大多数的Scrapy代码，应该对页面上没有找到的东西有**弹性**，以便即使部分爬取失败，至少还能获取一些数据。

除了`getall()`和`get()`，还可以使用`re()`方法通过正则表达式进行提取：

```python
>>> response.css('title::text').re(r'Quotes.*')
['Quotes to Scrape']
>>> response.css('title::text').re(r'Q\w+')
['Quotes']
>>> response.css('title::text').re(r'(\w+) to (\w+)')
['Quotes', 'Scrape']
```

为了找到合适的CSS选择器，可以运行`view(response)`在浏览器中打开响应页面。

可以使用浏览器插件**Selector Gadget**来提取CSS选择器。

##### 2.3、XPath简介

除了CSS选择器，Scrapy选择器也支持使用XPath表达式：

```python
>>> response.xpath('//title')
[<Selector xpath='//title' data='<title>Quotes to Scrape</title>'>]
>>> response.xpath('//title/text()').get()
'Quotes to Scrape'
```

XPath表达式很强大，它是Scrapy Selectors的基础，事实上，CSS选择器在底层被转换为了XPath。

尽管不如CSS筛选器流行，XPath表达式更加强大，因为除了对结构导航，它还可以查找内容。使用XPath，可以筛选包含了文本”下一页“的链接。**推荐使用XPath**，它会使爬取更加简单。

[http://quotes.toscrape.com](http://quotes.toscrape.com/)中的名言由类似如下的HTML元素表示：

```html
<div class="quote">
    <span class="text">“The world as we have created it is a process of our
    thinking. It cannot be changed without changing our thinking.”</span>
    <span>
        by <small class="author">Albert Einstein</small>
        <a href="/author/Albert-Einstein">(about)</a>
    </span>
    <div class="tags">
        Tags:
        <a class="tag" href="/tag/change/page/1/">change</a>
        <a class="tag" href="/tag/deep-thoughts/page/1/">deep-thoughts</a>
        <a class="tag" href="/tag/thinking/page/1/">thinking</a>
        <a class="tag" href="/tag/world/page/1/">world</a>
    </div>
</div>
```

打开一个Scrapy shell来尝试作者和名言的提取：

```shell
$ scrapy shell 'http://quotes.toscrape.com'
```

首先获取名言的筛选器集合：

```python
>>> response.css("div.quote")
[<Selector xpath="descendant-or-self::div[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype...'>,
 <Selector xpath="descendant-or-self::div[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype...'>,
 ...]
```

选择一个筛选器来提取：

```
>>> quote = response.css("div.quote")[0]
>>> text = quote.css("span.text::text").get()
>>> text
'“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'
>>> author = quote.css("small.author::text").get()
>>> author
'Albert Einstein'
```

名言的标签是一个字符串集合，可以使用`getall()`来获取它们：

```python
>>> tags = quote.css("div.tags a.tag::text").getall()
>>> tags
['change', 'deep-thoughts', 'thinking', 'world']
```

知道如何提取一条名言的数据后，就可以遍历所有的名言元素，并将它们一起保存到一个Python字典中：

```python
>>> for quote in response.css("div.quote"):
...     text = quote.css("span.text::text").get()
...     author = quote.css("small.author::text").get()
...     tags = quote.css("div.tags a.tag::text").getall()
...     print(dict(text=text, author=author, tags=tags))
{'text': '“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”', 'author': 'Albert Einstein', 'tags': ['change', 'deep-thoughts', 'thinking', 'world']}
{'text': '“It is our choices, Harry, that show what we truly are, far more than our abilities.”', 'author': 'J.K. Rowling', 'tags': ['abilities', 'choices']}
...
```

接下来，在爬虫程序中提取数据。

Scrapy爬虫一般会生成很多的字典（包含了从页面获取的数据）。在回调方法中使用`yield`关键字：

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
        'http://quotes.toscrape.com/page/2/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').get(),
                'author': quote.css('small.author::text').get(),
                'tags': quote.css('div.tags a.tag::text').getall(),
            }
```

运行这个爬虫，会看到日志中输出了提取的数据：

```
2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
{'tags': ['life', 'love'], 'author': 'André Gide', 'text': '“It is better to be hated for what you are than to be loved for what you are not.”'}
2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
{'tags': ['edison', 'failure', 'inspirational', 'paraphrased'], 'author': 'Thomas A. Edison', 'text': "“I have not failed. I've just found 10,000 ways that won't work.”"}
```

#### 3、保存爬取的数据

保存爬取的数据的最简单方式是使用Feed exports，通过命令：

```shell
$ scrapy crawl quotes -O quotes.json
```

会生成一个`quotes.json`文件，它包含了所有的爬取数据，以JSON序列化。

`-O`命令开关会覆盖已经存在的文件；使用`-o`则追加新的内容到已经存在的文件。但是，往JSON文件追加内容会使文件内容变为非法的JSON。所以，当往文件追加内容时，可以考虑使用不同的序列化格式，比如**JSON Lines**：

```shell
$ scrapy crawl quotes -o quotes.jl
```

JSON Lines格式很有用，因为它是像流一样的（stream-like），可以很容易地对它追加新的记录。另外，因为每条记录是独立的行，可以在不把所有数据放进内存的情况下处理大文件。

对于小型项目，这就够了，但是如果要对抓取得项目进行复杂处理，可以写一个Item Pipeline。创建项目的时候，已经创建了Item Pipeline的占位文件`tutorial/pipelines.py`。如果只是想保存抓取得项目，那么不用实现任何的item pipelines。

#### 4、追踪链接

在前面的例子中，除了从[http://quotes.toscrape.com](http://quotes.toscrape.com/)抓取前两页的数据，还可以抓取这个网站的所有页面。

要追踪链接（follow links），首先，要从页面提取到要追踪的链接。通过检查页面，可以发现页面上有下一页链接的标记：

```html
<ul class="pager">
    <li class="next">
        <a href="/page/2/">Next <span aria-hidden="true">&rarr;</span></a>
    </li>
</ul>
```

在Scrapy shell中尝试提取下一页的链接：

```python
>>> response.css('li.next a::attr(href)').get()
'/page/2/'
```

还可以使用**`attrib`属性**来提取这个链接：

```python
>>> response.css('li.next a').attrib['href']
'/page/2/'
```

修改爬虫代码，以追踪下一页的链接：

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').get(),
                'author': quote.css('small.author::text').get(),
                'tags': quote.css('div.tags a.tag::text').getall(),
            }

        next_page = response.css('li.next a::attr(href)').get()
        if next_page is not None:
            next_page = response.urljoin(next_page)
            yield scrapy.Request(next_page, callback=self.parse)
```

在提取数据之后，`parse()`方法查找下一页的链接，使用`urljoin()`构建一个完整的绝对URL（因为下一页的链接是相对的），产生一个到下一页的新请求（使用`parse()`方法本身作为回调函数来处理从下一页提取的数据，并持续地爬取所有的页面）。

Scrapy追踪链接的机制是：当在回调方法中生成一个`Request`时，Scrapy会调度发送这个请求并注册一个请求结束时执行的回调方法。

使用这种机制，可以根据定义的规则追踪连接，并根据访问的页面提取不同的数据，从而构架复杂的爬虫。

这个例子中，相当于创建了一个循环，追踪所有的下一页链接，直到没有下一页——便于爬取博客、论坛和其它有分页的网站。

##### 4.1、创建Requests快捷方式

创建`Request`对象的一个快捷方式是使用**`response.follow`**：

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').get(),
                'author': quote.css('span small::text').get(),
                'tags': quote.css('div.tags a.tag::text').getall(),
            }

        next_page = response.css('li.next a::attr(href)').get()
        if next_page is not None:
            yield response.follow(next_page, callback=self.parse)
```

于`scrapy.Request`不同，`response.follow`支持直接使用相对URLs——不用调用`urljoin`。`response.follow`只是返回一个`Request`实例，还是需要`yield`。

除了传递链接字符串到`response.follow`，还可以传递筛选器。但是，传递的筛选器应该提取必要的属性：

```python
for href in response.css('ul.pager a::attr(href)'):
    yield response.follow(href, callback=self.parse)
```

对于`<a>`元素也有一个快捷方式——`response.follow`会自动地使用它们的`href`属性。所以，代码可以简化为：

```python
for a in response.css('ul.pager a'):
    yield response.follow(a, callback=self.parse)
```

如果要在一次迭代中创建多个请求，可以使用**`response.follow_all`**：

```python
anchors = response.css('ul.pager a')
yield from response.follow_all(anchors, callback=self.parse)
```

还可以进一步简化为：

```python
yield from response.follow_all(css='ul.pager a', callback=self.parse)
```

##### 4.2、其它例子

追踪连接，解析作者信息的爬虫：

```python
import scrapy


class AuthorSpider(scrapy.Spider):
    name = 'author'

    start_urls = ['http://quotes.toscrape.com/']

    def parse(self, response):
        author_page_links = response.css('.author + a')
        yield from response.follow_all(author_page_links, self.parse_author)

        pagination_links = response.css('li.next a')
        yield from response.follow_all(pagination_links, self.parse)

    def parse_author(self, response):
        def extract_with_css(query):
            return response.css(query).get(default='').strip()

        yield {
            'name': extract_with_css('h3.author-title::text'),
            'birthdate': extract_with_css('.author-born-date::text'),
            'bio': extract_with_css('.author-description::text'),
        }
```

这个爬虫会从主页开始爬取，它会追踪所有的作者页面的链接并对每个请求调用回调方法`parse_author`，并且它还会使用`parse`回调方法解析分页链接。可见，**`parse`方法是可以生成多个不同`Request`的**。

这里将回调方法作为位置参数传递给`response.follow_all`以简化代码；这对`Request`也是使用的。

`parse_author`回调方法中定义了一个辅助函数来提取和整理CSS查询的数据，并且返回一个包含作者数据的Python dict。

这个爬虫展示了另一个有趣的事情，即使相同作者有很多的名言，也不必担心会请求相同的作者页面多次。**默认情况下，Scrapy会过滤掉对已经访问过的URLs的重复请求**，从而避免由于编码失误而对服务器造成冲击的问题。这个功能可以通过设置`DUPEFILTER_CLASS`进行配置。

还有另外一种追踪链接的爬虫，`CrawlSpider`类，它允许定义一个`rules`（一个`Rule`对象的集合），每个`Rule`对象定义一些爬取网站的行为。

另外，通常的模式是构建item。

#### 5、使用爬虫参数

当运行爬虫的时候，可以使用`-a`选项为爬虫提供命令行参数：

```shell
$ scrapy crawl quotes -O quotes-humor.json -a tag=humor
```

这些参数是传递给Spider的`__init__`方法的，默认变为了爬虫的属性。

在这个例子中，`tag`参数提供的值，可以通过`self.tag`使用。这样，通过标签参数来构建URL，就可以让爬虫指定标签的名言：

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        url = 'http://quotes.toscrape.com/'
        tag = getattr(self, 'tag', None)
        if tag is not None:
            url = url + 'tag/' + tag
        yield scrapy.Request(url, self.parse)

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').get(),
                'author': quote.css('small.author::text').get(),
            }

        next_page = response.css('li.next a::attr(href)').get()
        if next_page is not None:
            yield response.follow(next_page, self.parse)
```

如果传递参数`tag=humor`到爬虫，将只访问标签为`humor`的URLs，比如`http://quotes.toscrape.com/tag/humor`。