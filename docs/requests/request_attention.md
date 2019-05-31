# 注意事项

有时间一定要熟悉Requests的文档：[Requests: 让 HTTP 服务人类¶](https://2.python-requests.org//zh_CN/latest/#requests-http)

Requests 完全满足今日 web 的需求。

- Keep-Alive & 连接池
- 国际化域名和 URL
- 带持久 Cookie 的会话
- 浏览器式的 SSL 认证
- 自动内容解码
- 基本/摘要式的身份认证
- 优雅的 key/value Cookie
- 自动解压
- Unicode 响应体
- HTTP(S) 代理支持
- 文件分块上传
- 流下载
- 连接超时
- 分块请求
- 支持 `.netrc`



我会列举一些使用Requests时需要注意的点。



## SSL证书认证

Requests 可以为 HTTPS 请求验证 SSL 证书，就像 web 浏览器一样。SSL 验证默认是开启的，如果证书验证失败，Requests 会抛出 SSLError:

```python
>>> requests.get('https://requestb.in')
requests.exceptions.SSLError: hostname 'requestb.in' doesn't match either of '*.herokuapp.com', 'herokuapp.com'
```

如果你将 `verify` 设置为 False，Requests 也能忽略对 SSL 证书的验证。

```python
>>> requests.get('https://kennethreitz.org', verify=False)
<Response [200]>
```

默认情况下， `verify` 是设置为 True 的。选项 `verify` 仅应用于主机证书。

> 对于私有证书，你也可以传递一个 CA_BUNDLE 文件的路径给 `verify`。你也可以设置 # `REQUEST_CA_BUNDLE` 环境变量。



## 超时设置

为防止服务器不能及时响应，大部分发至外部服务器的请求都应该带着 timeout 参数。在默认情况下，除非显式指定了 timeout 值，requests 是不会自动进行超时处理的。如果没有 timeout，你的代码可能会挂起若干分钟甚至更长时间。

```python
r = requests.get('https://github.com', timeout=10)
```



我一般会把超时时间设为10秒。



## 封装

如果你需要频繁使用Requests发送请求是，最好是将其封装成一个通用方法。

例如这里是对post方法的简单封装，当然需要根据自己需求来做。

```python
import requests
import json as json_


class Session(object):

    def __init__(self):
        self.session = requests.session()

    def post(self, url, data=None, json=None, **kwargs):
        retry_times = 0
        while True:
            try:
                res = self.session.post(url, data=data, json=json, **kwargs)
            except (requests.exceptions.ConnectionError, requests.ReadTimeout):
                pass
            else:
                try:
                    res.json()
                except json_.JSONDecodeError:
                    retry_times += 1
                    if retry_times > 20:
                        return None
                    else:
                        continue

                return res
```



还可以看看这里，借鉴别人的思想：[Python’s encapsulation of requests](https://developpaper.com/pythons-encapsulation-of-requests/)

```python
# -*- coding:utf-8 -*-
 
import requests
from concurrent.futures import ThreadPoolExecutor
Read from Tools.Config import Config configuration file
From Tools. Log import Log # log management
From Tools. tools import decoLOG log decoration
 
'''
  Functions: Requests class
  Usage method: 
  Author: Guo Kechang
  Time of completion: 20180224
  Update:
  Update time:
'''
class Requests(object):
  def __init__(self):
    self.session = requests.session()
    self.header = {}
    # By default, URLs come from configuration files to facilitate switching between different test environments, and can also be set dynamically.
    self.URL = Config().getURL()
    # Default 60s, can be set dynamically
    self.timeout = 60
    # When HTTP connection is abnormal, the number of reconnections is 3 by default. It can be set dynamically.
    self.iRetryNum = 3
 
    self.errorMsg = ""
    # Content = {Use Case Number: Response Data}
    self.responses = {}
    # Content = {Use Case Number: Exception Information}
    self.resErr={}
 
 
  # Retention of original post usage
  # bodyData: request's data
  @decoLOG
  def post(self, bodyData):
    response = None
    self.errorMsg = ""
 
    try:
      response = self.session.post(self.URL, data=bodyData.encode('utf-8'), headers=self.header, timeout=self.timeout)
      response.raise_for_status()
    except Exception as e:
      self.errorMsg = str(e)
      Log (). logger. error ("HTTP request exception, exception information:% s"% self. errorMsg)
    return response
 
 
  # Complex request concurrent processing, in the form of thread pool, use case > thread pool capacity: thread pool capacity is concurrent, otherwise, use case number is concurrent
  # dicDatas: {Use Case Number: Use Case Data}
  @decoLOG
  def req_all(self, dicDatas, iThreadNum=5):
 
    if len(dict(dicDatas)) < 1:
      Log (). logger. error ("No test object, please confirm and try again..." ""
      return self.responses.clear()
 
    # Request use case set conversion (use case number, use case data)
    seed = [i for i in dicDatas.items()]
    self.responses.clear()
 
    # Thread pool concurrent execution, iThreadNum concurrent number
    with ThreadPoolExecutor(iThreadNum) as executor:
      executor.map(self.req_single,seed)
 
    # Returns response information for all requests ({use case number: response data}), HTTP connection exception: corresponding to None
    return self.responses
 
  # For single use case submission, HTTP connection failures can be reconnected, and the maximum number of reconnections can be dynamically set
  def req_single(self, listData, reqType="post", iLoop=1):
    response = None
    # If the maximum number of reconnections is reached, the submission ends after the connection
    if iLoop == self.iRetryNum:
      if reqType == "post":
        try:
          response = requests.post(self.URL, data=listData[1].encode('utf-8'), headers=self.header,
                       timeout=self.timeout)
          response.raise_for_status()
        except Exception as e:
          # Exception information is saved only when the maximum number of connections is reached, but the maximum number of connections is not reached. Exception information is empty.
          self.resErr[listData[0]] = str(e)
          Log (). logger. error ("HTTP request exception, exception information:% s [% d]% (str (e), iLoop))
 
        self.responses[listData[0]] = response
      else:
        # for future： other request method expand
        pass
    # The maximum number of connections is not reached, and if an exception occurs, a reconnection attempt is made
    else:
      if reqType == "post":
        try:
          response = requests.post(self.URL, data=listData[1].encode('utf-8'), headers=self.header,
                       timeout=self.timeout)
          response.raise_for_status()
        except Exception as e:
          Log (). logger. error ("HTTP request exception, exception information:% s [% d]% (str (e), iLoop))
          # Increased number of reconnections
          iLoop += 1
          # Reconnect
          self.req_single(listData, reqType, iLoop)
          # Current connection termination
          return None
        self.responses[listData[0]] = response
      else:
        # for future： other request method expand
        pass
 
  # Set up SoapAction to complete the header setting of web service interface quickly
  def setSoapAction(self, soapAction):
    self.header["SOAPAction"] = soapAction
    self.header["Content-Type"] = "text/xml;charset=UTF-8"
    self.header["Connection"] = "Keep-Alive"
    self.header["User-Agent"] = "InterfaceAutoTest-run"
```











