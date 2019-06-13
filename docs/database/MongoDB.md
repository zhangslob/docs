# MongoDB使用

MongoDB在爬虫中是最常见的数据库选择，因为够灵活多变，简单好用。

# 环境安装

使用docker一键安装：`docker run --name some-mongo -d mongo`

`pip3 install pymongo`

# Python操作MongoDB

## 连接


```python
import pymongo

client = pymongo.MongoClient()
```

推荐使用`MongoDB_URL`传递MongoDB地址，`MongoDB_URL`格式：`mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]

例如连接本地的`test`数据库：`mongodb://localhost/test`
例如连接本地需要验证的`test`数据库：`mongodb://admin:123456@localhost/test`

获取数据库的方法如下：
```python
import pymongo

client = pymongo.MongoClient('mongodb://localhost/test')
db = client.get_database()
```

## 创建索引

```python
import pymongo

client = pymongo.MongoClient('mongodb://localhost/test')
db = client.get_database()

collection = db['test_collection']
collection.create_index('index')
collection.create_index('id', unique=True)
```

我们这里连接到本地MongoDB的test数据库下的test_collection，然后在test_collection这张表上面创建了两个索引：

1. index：普通索引
2. id：唯一索引，不允许重复

> 不要建太多无用索引，也不要重复建索引

## 插入数据

单条数据插入：
```python
import pymongo

client = pymongo.MongoClient('mongodb://localhost/test')
db = client.get_database()

collection = db['test_collection']
# 不要再创建索引了

data = {'name': "zhang", 'age': 18}
collection.insert_one(data)
```

然后就可以查看了，这里使用的是Robot 3t

![](https://i.loli.net/2019/06/13/5d01f286151c437037.png)

插入多条。如果有很多数据插入，可以选择此种方法，效率会比单条插入高，对数据库压力也会更小。

```python
import pymongo

client = pymongo.MongoClient('mongodb://localhost/test')
db = client.get_database()

collection = db['test_collection']

data = [
    {"name": "Taobao", "alexa": "100", "url": "https://www.taobao.com"},
    {"name": "QQ", "alexa": "101", "url": "https://www.qq.com"},
    {"name": "Facebook", "alexa": "10", "url": "https://www.facebook.com"},
    {"name": "知乎", "alexa": "103", "url": "https://www.zhihu.com"},
    {"name": "Github", "alexa": "109", "url": "https://www.github.com"}
]
collection.insert_many(data, ordered=False)
```

如果我们直接运行是会报错的，因为我们上面有创建唯一索引，而且文档中都没有`id`这个字段，所以我们需要先删除索引`id`，然后再插入数据。

![](https://i.loli.net/2019/06/13/5d01f426cce8f41482.png)

## 查询数据

这里就包含很多知识了，MongoDB查询方法真的有很多，[看文档](https://docs.mongodb.com/manual/tutorial/query-documents/)

## 错误判断

上面我们说到，我们有建了唯一索引，比如是`id`，如果插入重复的`id`，是会报错的，我们可以捕获这个异常。

```python
try:
    collection.insert_one(data)
except pymongo.errors.DuplicateKeyError:
    print('重复插入数据：{}'.format(data))
```


# 作业

阅读理解这些文章：[MongoDB 标签](https://zhangslob.github.io/tags/MongoDB/)

