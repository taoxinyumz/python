## 爬虫循环的步骤
+  定义爬虫初始的URL，然后设置回调函数来接收爬取的内容，默认start_urls = [] -> parse()函数，自定义是start_request() -> my_parse()
+  在回调函数中返回一个可迭代容器（包括Item对象，dict，Request，或同时几个），一般yield返回（或return[,]）,并设置针对返回Request的回调函数，如next_parse()
+  回调函数中就是下载的网页数据，在这里进行解析生成 yield item，可用BeautifulSoup，xpath，css，正则，自己习惯熟悉用哪个就用哪个
+  处理得到的item，数据库（Item Pipeline）或各种文件类型（Feed exports）

## Scrapy框架
+  Scrapy是一个Python编写的基于Twisted框架的开源网络爬虫框架，它提供了一整套高效的工具和API，可用于创建可扩展的网络爬虫，可以在互联网上抓取和提取结构化数据。
+  结构组成：（1）引擎(Engine)：负责控制整个框架的运作流程，包括调度器、下载器、爬虫、数据管道等组件之间的协调和数据传递。（2）调度器(Scheduler)：用于接收爬虫发来的请求，并根据一定的策略进行排队和调度，将请求发送给下载器进行下载。（3）下载器(Downloader)：用于下载网络资源，如网页、图片、视频等，根据请求生成HTTP请求并将响应返回给引擎。（4）爬虫(Spider)：用于定义如何爬取和解析目标站点的网页，可以提取目标数据并构建请求。（5）数据管道(Item Pipeline)：用于处理和保存从爬虫中提取出来的数据，可以进行数据清洗、去重、存储等操作。

## Scrapy进行爬取的一般工作流程
+  Engine通过Scheduler将第一个Spider的第一个请求添加到队列中。
+  Scheduler将队列中的请求发送到Downloader，下载器发送请求到互联网并返回响应。
+  Downloader将响应发送回给Engine，Engine将响应发送到Spider处理。
+  Spider处理响应并提取出Item（需要爬取的数据）和新的请求，将Item传递给Item Pipeline，将新的请求发送到Scheduler。
+  Item Pipeline对Item进行处理，如存储到数据库等。
+  Scheduler对新的请求进行去重处理，再次将其添加到队列中。
+  Engine重复上述步骤，直到队列中没有更多的请求为止
**举个例子**
~~~
import scrapy      #是在 Python 程序中引入 Scrapy 模块的语句，这样程序就可以使用 Scrapy 框架提供的各种功能和类
class MySpider(scrapy.Spider):     
    name = 'myspider'               #
    start_urls = ['http://www.example.com']   #
    def parse(self, response):           #这段代码是 Scrapy 框架中定义 Spider 的一个默认方法，用于处理从网站服务器返回的 HTTP 响应。
        # 处理初始响应的代码
        pass
    def my_parse(self, response):
        # 处理其他响应的代码
        pass
~~~





