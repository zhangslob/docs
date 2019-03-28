# Requests基础使用

在google中搜索requests的结果：

![](https://ws3.sinaimg.cn/large/006tKfTcly1g1igx3ei57j313a0pfn38.jpg)

有中文也有英文文档，如果英文还OK就可以去撸英文文档。中文文档地址：[Requests: 让 HTTP 服务人类](<http://docs.python-requests.org/zh_CN/latest/>)

requests作为Python最出名的第三方库，各方面的教程已经非常完善，尤其是基础用法。



# 安装

最简单的当然是使用pip安装：

```bash
$ pip install requests
```

如果你的Python3版本对应的pip软链是pip3，记得更换为pip3。

如果安装有问题，参考：[安装 Requests](<http://docs.python-requests.org/zh_CN/latest/user/install.html>)



# 发送请求

```python
>>> import requests
>>> r = requests.get('https://api.github.com/events')
>>> r = requests.post('http://httpbin.org/post', data = {'key':'value'})
>>> r = requests.put('http://httpbin.org/put', data = {'key':'value'})
>>> r = requests.delete('http://httpbin.org/delete')
>>> r = requests.head('http://httpbin.org/get')
>>> r = requests.options('http://httpbin.org/get')
```

参见：[快速上手](<http://docs.python-requests.org/zh_CN/latest/user/quickstart.html#id2>)

可以看到使用requests发送请求非常简单，我们来看看使用urllib的代码

```python
import urllib.request
 
req = urllib.request.Request('http://www.example.com/')
req.add_header('Referer', 'http://www.python.org/')
r = urllib.request.urlopen(req)
 
result = f.read().decode('utf-8')
```

关于requests与urllib，强烈建议是选择requests，这里有一个非常经典的回答 [What are the differences between the urllib, urllib2, and requests module?](https://stackoverflow.com/questions/2018026/what-are-the-differences-between-the-urllib-urllib2-and-requests-module)



至于其他的参见的添加headers，这里就不说了，看文档即可。

这一节任务就是把[快速上手](<http://docs.python-requests.org/zh_CN/latest/user/quickstart.html>)的内容看完，并完成下面的作业。

# 作业

给你三个网址，试试使用requests抓取html

1. `https://www.baidu.com`
2. `https://www.toutiao.com`
3. `https://www.zhihu.com`
4. `https://www.google.com`

你可能会发现一些奇怪的现象，别着急，下一节会继续讲。