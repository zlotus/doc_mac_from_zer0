# 数据持久化-`sqlite3`模块

大名鼎鼎的SQLite是一个使用C语言实现的存储于磁盘上的轻量级数据库。它不像大多数数据库那样需要一个独立的服务端、非标准的SQL语句操作数据库。很多软件使用SQLite作为软件内部数据存储。也有一些团队在开发软件是使用SQLite实现原型，然后再把软件移植到诸如PostgreSQL、Oracle之类的大型数据库上。

本模块服从[PEP 249](http://legacy.python.org/dev/peps/pep-0249/)中定义的DB-API 2.0规范。（PEP 249是一份出镜率很高的文档，值得阅读，因为Python与各大数据库的接口都服从这个文档规范。）

## 概述

要使用本模块，必须先创建一个`Connection`对象，以连接数据库。下面代码中的数据会写入example.db中：

```
import sqlite3
conn = sqlite3.connect('example.db')
```

如果数据库名指定为`:memory:`时，则会建立一个内存数据库。

当有了`Connection`对象后，需要使用`Cursor`对象的`execute()`方法来执行SQL指令：

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

作为替代，推荐使用DB-API规范参数：`?`。在SQL语句中，可以使用`?`作为占位符，然后在`execute()`中将所需变量放在元组中替换这些占位符。

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

## Connection对象

### class sqlite3.Connection

SQLite数据库连接类有如下属性：

### isolation_level

获得或设置数据库的事务隔离等级。使用`None`表示自动提交，或使用“DEFERRED”, “IMMEDIATE”, “EXCLUSIVE”中的一个。详情参见“事务控制”。

### in_transaction

当有活动中的事务（有未提交的更改）时为真，否则为假。这是一项只读属性。

### cursor([cursorClass])

这个cursor函数接受一个可选参数*cursorClass*。如果提供参数，则参数必须是一个继承自`sqlite3.Cursor`的游标类。

### commit()

使用这个方法提交当前事务。如果这个方法未被调用，则其他连接到该数据库上的用户将看不到你当前（自最后一次`commit()`调用以来）的更改。如果你在疑问为什么对数据库的写操作没有生效，请检查是否忘记使用该方法提交事务。

### rollback()

这个方法会回滚自最后一次`commit()`调用以来的对数据库的所有更改。

### close()

这个方法会断开数据库连接。注意，这个方法不会自动调用`commit()`，如果没有调用`commit()`直接断开数据库连接，最后一次更改将会丢失。

### execute(sql[, parameters])

非常规的SQL语句执行捷径。这个方法会调用`cursor()`方法生成一个临时游标，然后将参数传给临时游标的`execute()`并调用。

### executemany(sql[, parameters])

非常规的SQL语句执行捷径。这个方法会调用`cursor()`方法生成一个临时游标，然后将参数传给临时游标的`executemany()`并调用。

### executescript(sql_script)

非常规的SQL语句执行捷径。这个方法会调用`cursor()`方法生成一个临时游标，然后将参数传给临时游标的`executescript()`并调用。

### create_function(name, num_params, func)

这个方法会新建一个名为*name*的用户定义函数，以便在以后的SQL语句中使用。*num_params*为该函数接受参数的数目，*func*是一个Python可调用对象，将会在SQL语句中以SQL函数的方式被调用。

函数应当返回SQLite支持的任何数据类型：bytes, str, int, float, None.

示例：

```
import sqlite3
import hashlib

def md5sum(t):
    return hashlib.md5(t).hexdigest()

con = sqlite3.connect(":memory:")
con.create_function("md5", 1, md5sum)
cur = con.cursor()
cur.execute("select md5(?)", (b"foo",))
print(cur.fetchone()[0])
```

### create_aggregate(name, num_params, aggregate_class)

新建用户定义的合计函数。

合计函数必须实现一个名为`step()`的方法，接受*num_params*个参数，以及一个`finalize()`方法用来返回合计函数的结果。

`finalize()`方法应当返回SQLite支持的任何数据类型：bytes, str, int, float, None.

示例：

```
import sqlite3

class MySum:
    def __init__(self):
        self.count = 0

    def step(self, value):
        self.count += value

    def finalize(self):
        return self.count

con = sqlite3.connect(":memory:")
con.create_aggregate("mysum", 1, MySum)
cur = con.cursor()
cur.execute("create table test(i)")
cur.execute("insert into test(i) values (1)")
cur.execute("insert into test(i) values (2)")
cur.execute("select mysum(i) from test")
print(cur.fetchone()[0])
```

### create_collation(name, callable)

新建一个名为*name*的排序规则，使用*callable*实现功能。传入的Python可调用对象会接到两个字符串参数，当第一个参数比第二个参数在序列中的：位置低时返回-1，位置相同时返回0，位置高时返回1。留意到排序规则仅影响到排序（SQL中的ORDER BY语句），因此，这个比较并不会影响到其他SQL操作。

注意，*callable*接收的字节串通常是`UTF-8`编码。

示例，逆序排列：

```
import sqlite3

def collate_reverse(string1, string2):
    if string1 == string2:
        return 0
    elif string1 < string2:
        return 1
    else:
        return -1

con = sqlite3.connect(":memory:")
con.create_collation("reverse", collate_reverse)

cur = con.cursor()
cur.execute("create table test(x)")
cur.executemany("insert into test(x) values (?)", [("a",), ("b",)])
cur.execute("select x from test order by x collate reverse")
for row in cur:
    print(row)
con.close()
```

要取消*name*排序规则的注册，只需要将*callable*置为`None`即可：

```
con.create_collation("reverse", None)
```

### interrupt()

可以从别的线程调用这个方法，以取消该连接上正在执行的任何查询。之后查询将会退出，其调用者会收到异常。

### set_authorizer(authorizer_callback)

用来注册访问控制回调函数。每当有访问数据库某列的行为时，都会触发这个回调函数。当访问应当被允许时返回`SQLITE_OK`，访问应当被终止事返回`SQLITE_DENY`，该列应当被当做NULL时返回`SQLITE_IGNORE`，这几个常量就在本模块中。

回调函数的第一个参数表示允许什么类型的操作，第二和第三个参数是常规参数还是是`None`取决于第一个参数，第四个参数是数据库表名（如“main”, “temp”），第五个参数可以是负责访问控制的触发器或视图的名称，也可以是`None`表示允许SQL语句直接访问。

### set_progress_handler(handler, n)

用来注册类似计划任务的回调函数。SQLite虚拟机每执行*n*条指令就会触发一次*handler*。当你想要在一个耗时操作（如更新GUI中的数据）中接到SQLite按计划执行特定动作时，这个方法会非常有用。

### set_trace_callback(trace_callback)

用来注册调试用回调函数。每当SQLite后台执行SQL语句时，都会触发*trace_callback*。

*trace_callback*只接受唯一参数，即当前执行的SQL语句字符串，回调函数不需要返回值。注意，数据库后台不只是执行传给`Cursor.execute()`的语句，也执行其他来源的事务（如来自Python事务管理模块，或触发了当前数据库定义的触发器）。

将`None`当做*trace_callback*传给这个方法将取消跟踪回调。

### enable_load_extension(enabled)

用来启用/停用SQLite从共享库中加载插件的功能。SQLite插件可以定义普通函数、合计函数、实现虚拟表。其中最著名的一个就是更随SQLite一起发布的全文检索插件。

加载插件的功能是默认停用的。

```
import sqlite3

con = sqlite3.connect(":memory:")

# enable extension loading
con.enable_load_extension(True)

# Load the fulltext search extension
con.execute("select load_extension('./fts3.so')")

# alternatively you can load the extension using an API call:
# con.load_extension("./fts3.so")

# disable extension laoding again
con.enable_load_extension(False)

# example from SQLite wiki
con.execute("create virtual table recipe using fts3(name, ingredients)")
con.executescript("""
    insert into recipe (name, ingredients) values ('broccoli stew', 'broccoli peppers cheese tomatoes');
    insert into recipe (name, ingredients) values ('pumpkin stew', 'pumpkin onions garlic celery');
    insert into recipe (name, ingredients) values ('broccoli pie', 'broccoli cheese onions flour');
    insert into recipe (name, ingredients) values ('pumpkin pie', 'pumpkin sugar flour butter');
    """)
for row in con.execute("select rowid, name, ingredients from recipe where name match 'pie'"):
    print(row)
```

### load_extension(path)

用来从*path*加载SQLite插件，不过，在此之前，必须调用`enable_load_extension()`启用插件加载功能。

### row_factory

你可以修改这个属性，提供另一个可调用对象，该对象接受一个包含游标及原始记录数据的元组作为参数，对象应当返回你想要的记录结果。通过修改这个方法，我们可以得到更高级的记录返回值。比如，实现返回一个对象，可以通过`row[column_name]`的形式访问记录域，像这样：

```
import sqlite3

def dict_factory(cursor, row):
    d = {}
    for idx, col in enumerate(cursor.description):
        d[col[0]] = row[idx]
    return d

con = sqlite3.connect(":memory:")
con.row_factory = dict_factory
cur = con.cursor()
cur.execute("select 1 as a")
print(cur.fetchone()["a"])
```

当然，上面的只是示例。如果你确实认为，查询记录以元组形式返回功能较弱，而且你需要记录可以通过列名访问的功能，则可以选择把`row_factory`设置为一个为此优化过的类型：`sqlite3.Row`。这个类型同时支持下标访问和大小写敏感的列名访问，而且几乎没有额外的内存开支。这可能比你用字典实现或用db_row实现更好。

### text_factory

通过这个属性，可以控制SQLite的`TEXT`类型在查询返回时会转为哪种Python类。此属性默认设为`str`，模块遇到`TEXT`会返回Unicode对象。如果你希望让其返回字节串，则可以将此属性设为`bytes`。

考虑到程序效率，可以通过将此属性设置为`sqlite3.OptimizedUnicode`来实现：当遇到non-ASCII数据时，返回`str`对象，否则返回`bytes`对象。

当然，你可以将此属性设为任意可调用对象，只需保证该对象接受一个字节流作为参数，而后返回想要的结果对象即可。

模拟代码示例：

```
import sqlite3

con = sqlite3.connect(":memory:")
cur = con.cursor()

AUSTRIA = "\xd6sterreich"

# by default, rows are returned as Unicode
cur.execute("select ?", (AUSTRIA,))
row = cur.fetchone()
assert row[0] == AUSTRIA

# but we can make sqlite3 always return bytestrings ...
con.text_factory = bytes
cur.execute("select ?", (AUSTRIA,))
row = cur.fetchone()
assert type(row[0]) is bytes
# the bytestrings will be encoded in UTF-8, unless you stored garbage in the
# database ...
assert row[0] == AUSTRIA.encode("utf-8")

# we can also implement a custom text_factory ...
# here we implement one that appends "foo" to all strings
con.text_factory = lambda x: x.decode("utf-8") + "foo"
cur.execute("select ?", ("bar",))
row = cur.fetchone()
assert row[0] == "barfoo"
```

### total_changes

此属性记录了从数据库建立连接开始，一共有多少行记录被更新、插入、删除过。

### iterdump()

返回一个用于dump数据库的迭代器，在想要保存内存数据库是会非常有用。此函数提供与SQLite的Shell中`.dump`命令相同的功能。

```
# Convert file existing_db.db to SQL dump file dump.sql
import sqlite3, os

con = sqlite3.connect('existing_db.db')
with open('dump.sql', 'w') as f:
    for line in con.iterdump():
        f.write('%s\n' % line)
```

## Cursor对象

### class sqlite3.Cursor

`Cursor`实例有下列属性和方法：

### execute(sql[, parameters])

执行一条SQL语句，语句可能是参数化的（使用占位符代替SQL字符）。sqlite3模块支持两种占位符：问号占位符和命名占位符。

关于两种占位符的示例：

```
import sqlite3

con = sqlite3.connect(":memory:")
cur = con.cursor()
cur.execute("create table people (name_last, age)")

who = "Yeltsin"
age = 72

# This is the qmark style:
cur.execute("insert into people values (?, ?)", (who, age))

# And this is the named style:
cur.execute("select * from people where name_last=:who and age=:age", {"who": who, "age": age})

print(cur.fetchone())
```

需要注意的是，`execute()`只能执行一条SQL语句，如果使用它一次执行多条语句时，函数会抛出`sqlite3.Warning`异常。如果想通过一次调用执行多条SQL语句，可以使用`executescript()`。

### executemany(sql, seq_of_parameters)

将多组参数套入同一条SQL语句执行，为SQL提供的参数可以说序列，也可以是迭代器。

使用迭代器：

```
import sqlite3

class IterChars:
    def __init__(self):
        self.count = ord('a')

    def __iter__(self):
        return self

    def __next__(self):
        if self.count > ord('z'):
            raise StopIteration
        self.count += 1
        return (chr(self.count - 1),) # this is a 1-tuple

con = sqlite3.connect(":memory:")
cur = con.cursor()
cur.execute("create table characters(c)")

theIter = IterChars()
cur.executemany("insert into characters(c) values (?)", theIter)

cur.execute("select c from characters")
print(cur.fetchall())
```

或是生成器：

```
import sqlite3
import string

def char_generator():
    for c in string.ascii_lowercase:
        yield (c,)

con = sqlite3.connect(":memory:")
cur = con.cursor()
cur.execute("create table characters(c)")

cur.executemany("insert into characters(c) values (?)", char_generator())

cur.execute("select c from characters")
print(cur.fetchall())
```

### executescript(sql_script)

用来为一次执行多条SQL语句提供方便（并未出现在[PEP249](http://legacy.python.org/dev/peps/pep-0249/)中，非标准方法）。方法会先提交一个`COMMIT`语句，而后将*sql_script*作为它的参数执行。

*sql_script*可以是`str`或`bytes`。

用法示例：

```
import sqlite3

con = sqlite3.connect(":memory:")
cur = con.cursor()
cur.executescript("""
    create table person(
        firstname,
        lastname,
        age
    );

    create table book(
        title,
        author,
        published
    );

    insert into book(title, author, published)
    values (
        'Dirk Gently''s Holistic Detective Agency',
        'Douglas Adams',
        1987
    );
    """)
```

### fetchone()

从结果集中取出下一条记录，并以元组形式返回。当没有结果时返回`None`。

### fetchmany(size=cursor.arraysize)

从及国际中取出下来*size*条记录，并以返回一个元组列表。当没有结果时返回空序列。

如果不指定*size*，则默认使用当前游标的arraysize。此方法会尽可能的满足*size*指定的记录数，当没有更多记录时，返回结果序列可能小于*size*指定的大小。

一般的，使用游标的arraysize可以提供较好的执行效率，如果用户指定了*size*属性，最好使链接中的所有`fetchmany()`的size属性都使用是这个值。

### fetchall()

取回一次查询的所有结果。应注意到游标的arraysize会影响到此方法的效率。当没有结果时返回空列表。

### rowcount

`Cursor`类实现了此属性，