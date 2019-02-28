这一节会讲讲如何模拟登陆。

# 场景

模拟登陆是爬虫中非常常见的场景，网络上的数据有些是打开就看得到的，比如腾讯新闻、拉勾网等，有些是需要登录才可见的，比如上一节说的知乎首页，点评评论翻页等等。所以模拟登录时非常常见的一种数据获取手段。

模拟登陆一般有两种方法：

1. 手动
2. 自动

# 手动

比如模拟登陆知乎，我们可以先打开知乎首页，然后输入我们自己的账号密码，打开控制台，将其中的cookies复制下来，完成业务操作。

![](https://ws3.sinaimg.cn/large/006tKfTcly1g0l98w4dh9j31gl0u0qbq.jpg)

我们把cookies运用在爬虫中就可以抓取知乎网站对“我”这个用户所推送的个性信息，在requests中我们可以这样应用。

```python
>>> url = 'http://httpbin.org/cookies'
>>> cookies = dict(cookies_are='working')

>>> r = requests.get(url, cookies=cookies)
>>> r.text
'{"cookies": {"cookies_are": "working"}}'
```

例子：

```python
import requests

cookies = {
    'BAIDUID': '7B7D88053B10EB435DB1E212DBF145BF:FG=1',
    'BIDUPSID': '7B7D88053B10EB435DB1E212DBF145BF',
    'PSTM': '1551098874',
    'BDRCVFR[pNjdDcNFITf]': 'mk3SLVN4HKm',
    'delPer': '0',
    'BD_CK_SAM': '1',
    'BD_UPN': '123253',
    'BD_HOME': '1',
    'locale': 'zh',
    'H_PS_PSSID': '1444_21092_18559_26350_28415',
    'BDRCVFR[feWj1Vr5u3D]': 'I67x6TjHwwYf0',
    'PSINO': '1',
    'H_PS_645EC': 'b710sJmEk7esA9h2dxosAL9oFPrYrIk%2FdoimlGSz7DCKfuSVp27CVuygqGuvEqwhznOf',
}

headers = {
    'Connection': 'keep-alive',
    'Upgrade-Insecure-Requests': '1',
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.119 Safari/537.36',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8',
    'Accept-Encoding': 'gzip, deflate, br',
    'Accept-Language': 'zh-CN,zh;q=0.9,en;q=0.8,zh-TW;q=0.7',
}

params = (
    ('wd', 'python'),
)

response = requests.get('https://www.baidu.com/s', headers=headers, params=params, cookies=cookies)
```

cookies可以单独拆出来当做一个字典，也可以存放在headers中，如下：

```python
import requests

url = "https://www.baidu.com/s"

querystring = {"wd":"python"}

headers = {
    'Connection': "keep-alive",
    'Upgrade-Insecure-Requests': "1",
    'User-Agent': "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.119 Safari/537.36",
    'Accept': "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8",
    'Accept-Encoding': "gzip, deflate, br",
    'Accept-Language': "zh-CN,zh;q=0.9,en;q=0.8,zh-TW;q=0.7",
    'Cookie': "BAIDUID=7B7D88053B10EB435DB1E212DBF145BF:FG=1; BIDUPSID=7B7D88053B10EB435DB1E212DBF145BF; PSTM=1551098874; BDRCVFR[pNjdDcNFITf]=mk3SLVN4HKm; delPer=0; BD_CK_SAM=1; BD_UPN=123253; BD_HOME=1; locale=zh; H_PS_PSSID=1444_21092_18559_26350_28415; BDRCVFR[feWj1Vr5u3D]=I67x6TjHwwYf0; PSINO=1; H_PS_645EC=b710sJmEk7esA9h2dxosAL9oFPrYrIk%2FdoimlGSz7DCKfuSVp27CVuygqGuvEqwhznOf",
    'cache-control': "no-cache",
    }

response = requests.request("GET", url, headers=headers, params=querystring)

print(response.text)
```

> 小提示，使用curl复制非常方便，curl内容后面会讲到。



可以看到手动复制的方法很简单，但是效率低，而且不能自动登录，即cookies失效，还需要人手动再去浏览器中复制，不适合爬虫场景。



# 自动

这次我们说一个稍微简单点的网站：Github的模拟登陆。文章我只爱去哪写过，完成内容可以到这里来看：[使用Selenium与Requests模拟登陆](https://zhangslob.github.io/2018/07/17/%E4%BD%BF%E7%94%A8Selenium%E4%B8%8ERequests%E6%A8%A1%E6%8B%9F%E7%99%BB%E9%99%86/)

核心代码：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import re
import requests

headers = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8',
    'Accept-Encoding': 'gzip, deflate, br',
    'Accept-Language': 'zh-CN,zh;q=0.9,en;q=0.8',
    'Connection': 'keep-alive',
    'Host': 'github.com',
    'Upgrade-Insecure-Requests': '1',
    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36'
}
s = requests.session()
s.headers.update(headers)


def get_token():
    url = 'https://github.com/login'
    response = s.get(url, timeout=10)
    pat = 'name=\"authenticity_token\" value=\"(.*?)\"'
    return re.findall(pat, response.text)[0]


def login(authenticity_token, account, password):
    payload = {
        'commit': 'Sign in',
        'utf8': '\u2713',
        'authenticity_token': authenticity_token,
        'login': account,
        'password': password,
    }
    url = 'https://github.com/session'
    response = s.post(url, data=payload)
    print(response.text)
    # do whatever you want


if __name__ == '__main__':
    account, password = '', ''
    authenticity_token = get_token()
    login(authenticity_token, account, password)
```

有几点说下：

1. github首页有一个authenticity_token，需要这个token维持登录状态。
2. github的密码是明文，这是非常少见的，大多数网站都是有加密的。一般可以通过开发者工具打断点来解决。
3. github的登录是不需要输入验证码的，实在是太少见了。国内的基本上都需要验证码的，验证码也是爬虫中很令人头疼的一块，大体上分为两种方法：机器学习、神经网络训练模型；接第三方打码平台。这个我们以后再讲。



# 实践

尝试完成美团登录。

网址：[美团网](https://passport.meituan.com/account/unitivelogin?service=www&continue=http%3A%2F%2Fwww.meituan.com%2Faccount%2Fsettoken%3Fcontinue%3Dhttps%253A%252F%252Fwww.meituan.com%252Fmeishi%252F526725%252F)

![](https://ws1.sinaimg.cn/large/006tKfTcly1g0m4xnm044j30qy0inq7n.jpg)

有几个重点参数需要解决：

1. **password**
2. **fingerprint**
3. **csrf**
4. **_token**

---

如果有任何疑问，请[在此交流](https://github.com/zhangslob/docs/issues/2)。