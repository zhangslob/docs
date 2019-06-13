# Scraoy Selectors

当然了，首先给大家推荐的还是官方文档：[Selectors](https://docs.scrapy.org/en/latest/topics/selectors.html)

Scrapy选择器构建于 lxml 库之上，这意味着它们在速度和解析准确性上非常相似。

本页面解释了选择器如何工作，并描述了相应的API。不同于 lxml API的臃肿，该API短小而简洁。这是因为 lxml 库除了用来选择标记化文档外，还可以用到许多任务上。

# 使用选择器(selectors)

一般我会使用xpath来解析

```python
>>> from scrapy.selector import Selector

>>> body = '<html><body><span>good</span></body></html>'
>>> s = Selector(text=body)
>>> s.xpath('//span/text()').extract()
['good']
```

将html导入`Selector`就可以得到一个选择器对象，直接使用xpath方法，接上xpath语法，就可以得到我们想要的。

# 在scrapy中使用Selector

举个例子，直接在scrapy中使用xpath

```python
import scrapy
from scrapy.crawler import CrawlerProcess


class TestSpider(scrapy.Spider):
    name = "test"
    start_urls = ["https://www.baidu.com/"]

    def parse(self, response):
        title = response.xpath('//*[@id="su"]/@value').extract_first()
        self.logger.info(title)


if __name__ == "__main__":
    process = CrawlerProcess({
        'USER_AGENT': 'Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)'
    })

    process.crawl(TestSpider)
    process.start()
```

文件可以直接运行，可以看到日志输出：`2019-06-13 11:20:48 [test] INFO: 百度一下`，title已经取到了，

为了配合XPath，Scrapy除了提供了 `Selector` 之外，还提供了方法来避免每次从response中提取数据时生成selector的麻烦。

Selector有四个基本的方法(点击相应的方法可以看到详细的API文档):
- xpath(): 传入xpath表达式，返回该表达式所对应的所有节点的selector list列表 。
- css(): 传入CSS表达式，返回该表达式所对应的所有节点的selector list列表.
- re(): 根据传入的正则表达式对数据进行提取，返回unicode字符串list列表。

# extract与extract_first

我们修改上面的代码，将spider修改下：
```python
class TestSpider(scrapy.Spider):
    name = "test"
    start_urls = ["https://www.baidu.com/"]

    def parse(self, response):
        title = response.xpath('//*[@id="su"]/@value').extract_first()
        title_ = response.xpath('//*[@id="su"]/@value').extract()
        self.logger.info(title)
        self.logger.info(title_)
```

输出是：
```bash
2019-06-13 11:23:26 [test] INFO: 百度一下
2019-06-13 11:23:26 [test] INFO: ['百度一下']
```
可以看到`extract_first`返回的是字符串，`extract`是列表。

更准确的说，我们用xpath解析`列表页`时，`extract`的是整个列表，`extract_first`是列表里的第一个字符串。

比如：

```python
>>> response.css('a::attr(href)').extract()
['image1.html', 'image2.html', 'image3.html', 'image4.html', 'image5.html']
>>> response.css('a::attr(href)').extract_first()
'image1.html'
```

# css与re

我们知道，scrapy的selector是支持css和re的，但是我在平时的解析工作中几乎用不到这两种方法，所以这里不重点讲。

其实优先推荐的选择方法应该是css，因为会和前端开发中的css是一致的。如果你是前端开发工作者，那你一定会很快上手css选择器。




