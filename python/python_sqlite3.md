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

虽然模块中的`Cursor`类实现了此属性，但数据库引擎对“rows affected”与“rows selected”的定义各有不同。

对于`executemany()`执行的语句，更改的行数也计入`rowcount`属性。

属性服从Python DB API规范，如果当前游标上没有执行任何`executeXX()`方法，或接口没有定义最近一次操作的`rowcount`，`rowcount`值会被置为-1。这包括了`SELECT`语句，因为在结果集取完之前，我们无法确定查询一共会返回多少行记录。

SQLite 3.6.5以前的版本在执行`DELETE FROM table`语句后会把此属性置为0。

### lastrowid

这是一个只读属性，记录最后一次被修改行的ID。此属性只有在`execute()`提交`INSERT`语句时才会被设置。但是对于在`executemany()`方法中提交的`INSERT`语句，属性会被置为`None`。

### description

此属性为最近一次查询的结果集提供列名。为了保留符合Python DB API规范的兼容性，属性为每一列提供一个7元素元组，除了第一个元素外，其他的元素皆为`None`。

即使最近一次`SELECT`语句没有结果返回，列名属性依然会被设置。

## Row对象

### class sqlite3.Row

`Row`对象是一个为`Connection`的`row_factory`属性优化过的对象。此对象在实现时模仿元组的行为特性。

此对象也支持使用列名的映射访问（类似字典键-值映射）、下标索引、迭代访问、相等比较以及使用`len()`返回长度。

如果两个`Row`对象有相同的列且列中的成员也相同时，这两个`Row`对象会在相等比较时判定为真。

### keys()

此方法以元组形式返回查询的所有列名。在查询结束后，其返回结果为`Cursor.description`中每个元组 的首元素。

假设我们用如下代码新建表：

```
conn = sqlite3.connect(":memory:")
c = conn.cursor()
c.execute('''create table stocks
(date text, trans text, symbol text,
 qty real, price real)''')
c.execute("""insert into stocks
          values ('2006-01-05','BUY','RHAT',100,35.14)""")
conn.commit()
c.close()
```

接下来插入列：

```
>>> conn.row_factory = sqlite3.Row
>>> c = conn.cursor()
>>> c.execute('select * from stocks')
<sqlite3.Cursor object at 0x7f4e7dd8fa80>
>>> r = c.fetchone()
>>> type(r)
<class 'sqlite3.Row'>
>>> tuple(r)
('2006-01-05', 'BUY', 'RHAT', 100.0, 35.14)
>>> len(r)
5
>>> r[2]
'RHAT'
>>> r.keys()
['date', 'trans', 'symbol', 'qty', 'price']
>>> r['qty']
100.0
>>> for member in r:
...     print(member)
...
2006-01-05
BUY
RHAT
100.0
35.14
```

## SQLite类型与Python类型

### 概况

SQLite原生支持的类型有：NULL, INTEGER, REAL, TEXT, BLOB。

下列Python类型可以直接传给SQLite：

Python type  |SQLite type
-------------|-----------
None         |NULL
int          |INTEGER
float        |REAL
str          |TEXT
bytes        |BLOB

下列SQLite类型可以默认转为Python类型：

SQLite type  |Python type
-------------|-----------
Null         |None
INTEGER      |int
REAL         |float
TEXT         |具体类型由`text_factory`属性决定，默认为`str`
BLOB         |bytes

有两种方法扩展模块类型系统支持的类型：可以将附加的Python类通过对象适配器存入SQLite；或将SQLite类型通过转换函数存为Python类型。

### 使用适配器将附加Python类型存入SQLite数据库

像上面描述的那样，SQLite只支持很有限的数据类型。要在SQLite中使用其他的Python类型，则需要将这些附加类型适配成模块支持的数据类型：None, int, float, str, bytes。

sqlite3模块使用Python对象适配器，在[PEP246](http://www.python.org/dev/peps/pep-0246)中定义，遵守的协议为PrepareProtocol。

有两种方式使sqlite3模块能够将用户自定义Python类型适配为SQLite数据库支持的类型。

#### 使对象自动适配

当你自己编写类型代码时，这个方法很适用，示例：

```
class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y
```

现在你想要将这个“点”对象存在数据库的一个列中。首先，你需要在Python->SQLite支持的数据类型中选择一种能够表达“点”对象的类型（现在假设我们选择`str`类型，并使用分号将坐标隔开）；然后你需要在类中实现方法`__conform__(self, protocol)`，该方法必须返回“点”对象适配给数据库的结果（这里是形如"x;y"的`str`）。这个方法的参数*protocol*的值将会是`sqlite3.PrepareProtocol`，示例：

```
import sqlite3

class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __conform__(self, protocol):
        if protocol is sqlite3.PrepareProtocol:
            return "%f;%f" % (self.x, self.y)

con = sqlite3.connect(":memory:")
cur = con.cursor()

p = Point(4.0, -3.2)
cur.execute("select ?", (p,))
print(cur.fetchone()[0])
```

#### 为类型注册适配函数

另一种实现是通过模块级适配器注册函数`sqlite3.register_adapter`为特定类型注册转换函数，示例：

```
import sqlite3

class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y

def adapt_point(point):
    return "%f;%f" % (point.x, point.y)

sqlite3.register_adapter(Point, adapt_point)

con = sqlite3.connect(":memory:")
cur = con.cursor()

p = Point(4.0, -3.2)
cur.execute("select ?", (p,))
print(cur.fetchone()[0])
```

sqlite3模块为类型`datetime.date`和`datetime.datetime`提供了两种默认适配器。但现在假设我们需要将`datetime.datetime`存成一个Unix风格的时间戳，而不是通常的ISO风格，示例：

```
import sqlite3
import datetime
import time

def adapt_datetime(ts):
    return time.mktime(ts.timetuple())

sqlite3.register_adapter(datetime.datetime, adapt_datetime)

con = sqlite3.connect(":memory:")
cur = con.cursor()

now = datetime.datetime.now()
cur.execute("select ?", (now,))
print(cur.fetchone()[0])
```

### 将SQLite数据转为自定义Python对象

编写适配器可以将Python对象转为SQLite支持的类型，但如果要使得整个映射机制完整，除了要实现Python到SQLite的适配，还需要SQLite到Python的转换。

回到前面“点”对象的例子，我们将点表示为型为"x;y"的字符串存于SQLite中。

首先，我们需要写一个转换器，接受一个`str`为参数，并返回`Point`对象。

注意，不论你将什么类型存入数据库，转换函数总是接受一个`str`类型的参数。

```
def convert_point(s):
    x, y = map(float, s.split(b";"))
    return Point(x, y)
```

然后，我们需要让sqlite3模块知道我们从数据库中查询出的结果其实是一个“点”对象，有两种实现方式：

* 通过类型声明，提供默认类型；
* 查询时在列名中明确指出；

这两种方法都已经在“模块函数与常量”一节中提到，通过为参数*detect_types*设置`PARSE_DECLTYPES`和`PARSE_COLNAMES`常量来侦测返回类型。

下面的代码是这两种方法的示例：

```
import sqlite3

class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __repr__(self):
        return "(%f;%f)" % (self.x, self.y)

def adapt_point(point):
    return ("%f;%f" % (point.x, point.y)).encode('ascii')

def convert_point(s):
    x, y = list(map(float, s.split(b";")))
    return Point(x, y)

# Register the adapter
sqlite3.register_adapter(Point, adapt_point)

# Register the converter
sqlite3.register_converter("point", convert_point)

p = Point(4.0, -3.2)

#########################
# 1) Using declared types
con = sqlite3.connect(":memory:", detect_types=sqlite3.PARSE_DECLTYPES)
cur = con.cursor()
cur.execute("create table test(p point)")

cur.execute("insert into test(p) values (?)", (p,))
cur.execute("select p from test")
print("with declared types:", cur.fetchone()[0])
cur.close()
con.close()

#######################
# 1) Using column names
con = sqlite3.connect(":memory:", detect_types=sqlite3.PARSE_COLNAMES)
cur = con.cursor()
cur.execute("create table test(p)")

cur.execute("insert into test(p) values (?)", (p,))
cur.execute('select p as "p [point]" from test')
print("with column names:", cur.fetchone()[0])
cur.close()
con.close()
```

### 默认适配器与转换器

上面提到，sqlite3模块为类型`datetime.date`和`datetime.datetime`提供了默认适配器，分别将这两种类型以ISO日期和ISO时间戳的形式存入SQLite。

为`datetime.date`注册的转换器叫“date”，为`datetime.datetime`注册的转换器叫“timestamp”。

所以，你可以在大多数情况下，不需要提供任何额外代码，就直接使用date/timestamps。

示例：

```
import sqlite3
import datetime

con = sqlite3.connect(":memory:", detect_types=sqlite3.PARSE_DECLTYPES|sqlite3.PARSE_COLNAMES)
cur = con.cursor()
cur.execute("create table test(d date, ts timestamp)")

today = datetime.date.today()
now = datetime.datetime.now()

cur.execute("insert into test(d, ts) values (?, ?)", (today, now))
cur.execute("select d, ts from test")
row = cur.fetchone()
print(today, "=>", row[0], type(row[0]))
print(now, "=>", row[1], type(row[1]))

cur.execute('select current_date as "d [date]", current_timestamp as "ts [timestamp]"')
row = cur.fetchone()
print("current_date", row[0], type(row[0]))
print("current_timestamp", row[1], type(row[1]))
```

如果SQLite中的时间戳拥有超过6个元素，默认转换器会自动将精确值截断在毫秒级。

### 事务处理

在默认状态下，sqlite3模块在执行一个DML语句（`INSERT`/`UPDATE`/`DELETE`/`REPLACE`）前就已经进入了一个事务，在执行下一个非DML、非查询（`INSERT`/`UPDATE`/`DELETE`/`REPLACE`/`SELECT`）语句前提交DML语句所在事务。

也就是，如果你在一个事务中写了诸如`CREATE TABLE ...`、`VACUUM`、`PRAGMA`的语句，sqlite3模块就会在执行这条语句前自动提交事务。这种行为有两个原因：一是这种语句中，有一些不能在事务中提交；二是为了满足模块对事务状态（该事务是否处于活动状态）跟踪的需要。访问`Connection.in_transaction`属性可以查看当前事务状态。

你可以通过修改`connect()`中的*isolation_level*或`isolation_level`属性，控制`BEGIN`语句的执行方式。

如果你希望**自动提交**事务，就把`isolation_level`置为`None`。

如果使用默认，则`BEGIN`语句会控制事务。也可以设置成SQLite支持的“DEFERRED”, “IMMEDIATE” or “EXCLUSIVE”之一。

## 高效使用sqlite3

### 使用快捷方法

直接使用`Connection`对象的非标准方法：`execute()`, `executemany()`, `executescript()`，可以避免显示声明多余的`Cursor`对象，从而简化代码。事实上，在执行这些非标准方法时，游标对象被隐式的创建并返回了。通过这种方式，你可以使用仅一次调用就执行`SELECT`语句，并迭代其返回的结果集。

```
import sqlite3

persons = [
    ("Hugo", "Boss"),
    ("Calvin", "Klein")
    ]

con = sqlite3.connect(":memory:")

# Create the table
con.execute("create table person(firstname, lastname)")

# Fill the table
con.executemany("insert into person(firstname, lastname) values (?, ?)", persons)

# Print the table contents
for row in con.execute("select firstname, lastname from person"):
    print(row)

print("I just deleted", con.execute("delete from person").rowcount, "rows")
```

### 使用列名代替索引

`sqlite3.Row`类型是一个非常有用的记录封装类。

通过`Row`封装的记录既可以使用索引访问（类似元组），也可以使用列名访问（大小写敏感，类似字典）。

```
import sqlite3

con = sqlite3.connect(":memory:")
con.row_factory = sqlite3.Row

cur = con.cursor()
cur.execute("select 'John' as name, 42 as age")
for row in cur:
    assert row[0] == row["name"]
    assert row["name"] == row["nAmE"]
    assert row[1] == row["age"]
    assert row[1] == row["AgE"]
```

### 使用链接对象管理上下文

数据库链接对象可以控制事务的自动提交以及回滚操作。如果遇到异常，则回滚事务，否则提交：

```
import sqlite3

con = sqlite3.connect(":memory:")
con.execute("create table person (id integer primary key, firstname varchar unique)")

# Successful, con.commit() is called automatically afterwards
with con:
    con.execute("insert into person(firstname) values (?)", ("Joe",))

# con.rollback() is called after the with block finishes with an exception, the
# exception is still raised and must be caught
try:
    with con:
        con.execute("insert into person(firstname) values (?)", ("Joe",))
except sqlite3.IntegrityError:
    print("couldn't add Joe twice")
```

## 常见问题

### 多线程

旧版SQLite存在多线程共享一个链接的问题，这就是为什么Python模块禁止线程间共享同一个链接或游标对象。如果你执意要使用共享，会得到一个运行时异常。

只有一个例外，调用`interrupt()`方法，因为只有从另一个线程调用此方法才有意义。
