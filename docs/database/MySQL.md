# MySQL使用

MySQL是最常见的关系型数据库，应用很广泛。

## 环境安装

docker: `docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql`

`pip3 install PyMySQL`

## 连接

先创建一张表

```sql
CREATE TABLE `users` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `email` varchar(255) COLLATE utf8_bin NOT NULL,
    `password` varchar(255) COLLATE utf8_bin NOT NULL,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin
AUTO_INCREMENT=1 ;
```

```python
import pymysql.cursors

# Connect to the database
connection = pymysql.connect(host='localhost',
                             user='user',
                             password='passwd',
                             db='db',
                             charset='utf8mb4',
                             cursorclass=pymysql.cursors.DictCursor)
```

## 创建cursor、执行SQL

```python
import pymysql.cursors

# Connect to the database
connection = pymysql.connect(host='localhost',
                             user='user',
                             password='passwd',
                             db='db',
                             charset='utf8mb4',
                             cursorclass=pymysql.cursors.DictCursor)


try:
    with connection.cursor() as cursor:
        # Create a new record
        sql = "INSERT INTO `users` (`email`, `password`) VALUES (%s, %s)"
        cursor.execute(sql, ('webmaster@python.org', 'very-secret'))

    # connection is not autocommit by default. So you must commit to save
    # your changes.
    connection.commit()

    with connection.cursor() as cursor:
        # Read a single record
        sql = "SELECT `id`, `password` FROM `users` WHERE `email`=%s"
        cursor.execute(sql, ('webmaster@python.org',))
        result = cursor.fetchone()
        print(result)
finally:
    connection.close()
```

将会输出：`{'password': 'very-secret', 'id': 1}`


## 查询数据

```python
# 执行查询 SQL
cursor.execute('SELECT * FROM `users`')
# 获取单条数据
cursor.fetchone()
# 获取前N条数据
cursor.fetchmany(3)
# 获取所有数据
cursor.fetchall()
```

## 设置游标类型

查询时，默认返回的数据类型为元组，可以自定义设置返回类型。支持5种游标类型：

- Cursor: 默认，元组类型
- DictCursor: 字典类型
- DictCursorMixin: 支持自定义的游标类型，需先自定义才可使用
- SSCursor: 无缓冲元组类型
- SSDictCursor: 无缓冲字典类型

无缓冲游标类型，适用于数据量很大，一次性返回太慢，或者服务端带宽较小时。

创建方法.创建连接时，通过 cursorclass 参数指定类型：

```python
connection = pymysql.connect(host='localhost',
                             user='root',
                             password='root',
                             db='demo',
                             charset='utf8',
                             cursorclass=pymysql.cursors.DictCursor)
```

也可以在创建游标时指定类型：

`cursor = connection.cursor(cursor=pymysql.cursors.DictCursor)`

