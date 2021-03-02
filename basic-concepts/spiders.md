### Spiders

Spiders是指定义如何爬取某个（或一组网站）的classes，包括如何执行爬取（即，追踪链接）和如何从它们的页面提取结构化的数据（即，爬取items）。

对于爬虫来说，爬取循环大致如下：

1. 以生成爬取第一批URLs的初始Requests开始，指定一个处理这些请求的响应的回调函数

   执行的第一批请求是通过调用`start_requests()`方法生成的，这些请求默认是从`start_urls`中指定的URLs生成的，使用`parse`方法作为Requests的回调函数

2. 
