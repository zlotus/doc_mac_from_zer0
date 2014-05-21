# 数据持久化-`sqlite3`模块

大名鼎鼎的SQLite是一个使用C语言实现的存储于磁盘上的轻量级数据库。它不像大多数数据库那样需要一个独立的服务端、非标准的SQL语句操作数据库。很多软件使用SQLite作为软件内部数据存储。也有一些团队在开发软件是使用SQLite实现原型，然后再把软件移植到诸如PostgreSQL、Oracle之类的大型数据库上。

本模块服从[PEP 249](http://legacy.python.org/dev/peps/pep-0249/)中定义的DB-API 2.0规范。（PEP 249是一份出镜率很高的文档，值得阅读，因为Python与各大数据库的接口都服从这个文档规范。）

## 概述

要使用本模块，必须先创建一个`Connection`对象，以连接数据库。下面代码中的数据会写入example.db中

```
import sqlite3
conn = sqlite3.connect('example.db')
```

如果数据库名指定为`:memory:`时，则会建立一个内存数据库。

当有力`Connection`对象后，需要使用`Cursor`对象的`execute()`方法来执行SQL指令：

```
c = conn.cursor()

# Create table
c.execute('''create table stocks
(date text, trans text, symbol text,
 qty real, price real)''')

# Insert a row of data
c.execute("""insert into stocks
          values ('2006-01-05','BUY','RHAT',100,35.14)""")

# Save (commit) the changes
conn.commit()

# We can also close the cursor if we are done with it
c.close()
```

在程序中，通常需要执行的SQL语句需要使用Python代码在运行时的变量。请注意，直接使用字符串的`%`运算符组合运行时需要的SQL指令是非常危险的，这可能会导致恶意的SQL注入。

作为替代，推荐使用DB-API参数：`?`。在SQL语句中，可以使用`?`作为占位符，然后在`execute()`中将所需变量放在元组中替换这些占位符。

```
# Never do this -- insecure!
symbol = 'IBM'
c.execute("select * from stocks where symbol = '%s'" % symbol)

# Do this instead
t = ('IBM',)
c.execute('select * from stocks where symbol=?', t)

# Larger example
for t in [('2006-03-28', 'BUY', 'IBM', 1000, 45.00),
          ('2006-04-05', 'BUY', 'MSFT', 1000, 72.00),
          ('2006-04-06', 'SELL', 'IBM', 500, 53.00),
         ]:
    c.execute('insert into stocks values (?,?,?,?,?)', t)
```

要取回SQL选择语句执行后的结果集，有三种方法：直接将这时的`Curser`对象当做迭代器使用；调用游标的`fetchone()`方法取回一条记录；调用`fetchall()`方法取回结果集列表。

把`Curser`当做迭代器的例子：

```
>>> c = conn.cursor()
>>> c.execute('select * from stocks order by price')
>>> for row in c:
...     print(row)
...
('2006-01-05', 'BUY', 'RHAT', 100, 35.14)
('2006-03-28', 'BUY', 'IBM', 1000, 45.0)
('2006-04-06', 'SELL', 'IBM', 500, 53.0)
('2006-04-05', 'BUY', 'MSOFT', 1000, 72.0)
>>>
```

## 模块函数与常量

### sqlite3.version

本模块的版本号，字符形式。这不是SQLite库的版本号。

### sqlite3.version_info

本模块的版本号，元组形式。这不是SQLite库的版本号。

### sqlite3.sqlite_version

SQLite库版本号，字符形式。

### sqlite3.sqlite_version_info

SQLite库版本号，元组形式。

### sqlite3.PARSE_DECLTYPES

本常量是作为`connect()`函数中*detect_types*参数的可选择出现的，值为1。

若将*detect_types*设为这个值，则模块会根据数据表每列的类型定义，将查询返回结果集按列转为该列的声明类型。比如“integer primary key”会被转为“integer”，“number(10)”会被转为“number”，之后会在注册的类型转换函数字典里查找该类型的转换函数，并根据转换函数的结果返回查询值。

### sqlite3.PARSE_COLNAMES

本常量是作为`connect()`函数中*detect_types*参数的可选择出现的，值为2。

若将*detect_types*设为这个值，则模块会在结果集的列名中寻找一个型为“[mytype]”的字符串，然后将“mytype”作为该列的类型。同样的，模块会在注册类型转换函数字典中查找该类型的转换函数，并根据转换函数的结果返回查询值。结果集的列名保存在`Cursor.description`中，列名仅是第一个单词。比如如果你使用类似`'as "x [datetime]"'`的语句，列名会是第一个空格前的所以字符，在这里就是“x”。

关于上面两个变量的使用，参见“将SQLite返回值转为Python自定义类型”的示例。

### sqlite3.connect(database[, timeout, detect_types, isolation_level, check_same_thread, factory, cached_statements, uri])

与SQLite数据库*datebase*建立链接，也可以使用`:memory:`指定模块建立内存数据库。

当同一个数据库被多个链接访问，并且有一个链接在修改数据库时，该SQLite数据库会被锁定，直到这个修改事务完成提交。*timeout*参数指定链接在遇到数据库锁时，在抛出异常前会等待多久，默认值为5秒。

*isolation_level*参数参见`Connection`对象的`Connection.isolation_level`属性。

SQLite原生支持的类型为TEXT, INTEGER, REAL, BLOB, NULL。如果想要别的数据类型支持只能：配合*detect_types*参数，使用模块级函数`register_converter()`注册类型转换函数，将你需要的自定义类型转为SQLite支持的类型。

*detect_types*默认值为0（即无类型侦测）。你可以使用`PARSE_DECLTYPES`与`PARSE_COLNAMES`的任意组合启用想要的类型侦测。

本模块默认使用`Connection`类建立链接，不过用户也可以选择实现`Connection`的子类，并将你的类转给*factory*做参数，这样就可以使用你的类建立连接了。

参见“SQLite与Python类型”得到更多细节。

本模块内部会使用SQL语句缓存来优化语句解析开销。如果想要明确设置一个链接的语句缓存数量，可以设置*cached_statements*值。默认值为100。

如果*uri*置为真，则*database*会被当做URI解析，我们可以通过URI解析设置链接的细节选项。下面的代码可以以只读方式打开数据库：

```
db = sqlite3.connect('file:path/to/database?mode=ro', uri=True)
```

更多的URI选项参见[SQLite URI documentation](http://www.sqlite.org/uri.html)。

### sqlite3.register_converter(typename, callable)

注册一个可调用对象，将从数据库得到的字节流转为Python类型。数据库返回的所有*typename*类型的值都会调用这个*callable*。使用`connect()`中的*detect_types*参数进行类型侦测的设置。注意，查询语句中的typename和本函数中的*typename*参数匹配时时大小写敏感的。

### sqlite3.register_adapter(type, callable)

注册一个*callable*将指定的Python类型*type*转为SQLite支持的类型。*callable*必须是一个独参的可调用对象，参数为Python类型，返回值必须为int, float, str, bytes中的一个。

### sqlite3.complete_statement(sql)

如果*sql*中包含一个以上的完整的SQL语句（以分号结尾），则返回真。函数并不检查SQL语法，只是确保没有开放的字符串，语句以分号结尾。

我们可以使用这个函数为SQLite做一个shell：

```
# A minimal SQLite shell for experiments

import sqlite3

con = sqlite3.connect(":memory:")
con.isolation_level = None
cur = con.cursor()

buffer = ""

print("Enter your SQL commands to execute in sqlite3.")
print("Enter a blank line to exit.")

while True:
    line = input()
    if line == "":
        break
    buffer += line
    if sqlite3.complete_statement(buffer):
        try:
            buffer = buffer.strip()
            cur.execute(buffer)

            if buffer.lstrip().upper().startswith("SELECT"):
                print(cur.fetchall())
        except sqlite3.Error as e:
            print("An error occurred:", e.args[0])
        buffer = ""

con.close()
```

### sqlite3.enable_callback_tracebacks(flag)

默认下，在用户定义的functions, aggregates, converters, authorizer回调中不会有跟踪信息。如果仍想要调试，则可以调用这个函数，并把*flag*设为真。此后就会从`sys.stderr`中得到跟踪信息了。将*flag*置为假可以再次关闭输出调试跟踪信息。

