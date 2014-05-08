# collections

推荐阅读[Advanced Data Structures in Python](http://pypix.com/python/advanced-data-structures-python/)

以及Python文档[collections — Container datatypes](https://docs.python.org/3.4/library/collections.html)

除了`dict`, `list`, `set`, `tuple`之外，Python还友善的提供了别的数据结构。

## ChainMap

`ChainMap`是3.3里才出现的，它可以把多个独立的字典“组织”成一个单一的字典。

    >>> d1 = {1: 'a', 2: 'b', 3: 'c', }
    >>> d2 = {4: 'd', 5: 'e', 3: 'f', }
    >>> import collections
    >>> cm = collections.ChainMap(d1, d2)
    >>> cm[1]
    'a'
    >>> cm[3]
    'c'
    >>> cm[5]
    'e'

注意到，对于重复的键，ChainMap会顺序查找包含的字典，找到键会立刻返回值，所以当访问`cm[3]`的时候，ChainMap返回d1的查找结果。

而访问`mc[3]`的时候会返回d2的结果：

    >>> mc = collections.ChainMap(d2, d1)
    >>> mc[3]
    'f'

使用ChainMap对多个字典进行合并，比把一个字典的内容`update()`到另一个字典中高效很多。

    >>> id(d1[1])
    4393542968
    >>> id(mc[1])
    4393542968
    >>> id(cm[1])
    4393542968

ChainMap的合并动作就说到这，对于增删改动作而言，默认是操作第一个字典。

不过对于ChainMap的这种默认动作，可以很容易通过子类实现：

    class DeepChainMap(ChainMap):
        'Variant of ChainMap that allows direct updates to inner scopes'
    
        def __setitem__(self, key, value):
            for mapping in self.maps:
                if key in mapping:
                    mapping[key] = value
                    return
            self.maps[0][key] = value
    
        def __delitem__(self, key):
            for mapping in self.maps:
                if key in mapping:
                    del mapping[key]
                    return
            raise KeyError(key)
    
    >>> d = DeepChainMap({'zebra': 'black'}, {'elephant': 'blue'}, {'lion': 'yellow'})
    >>> d['lion'] = 'orange'         # update an existing key two levels down
    >>> d['snake'] = 'red'           # new keys get added to the topmost dict
    >>> del d['elephant']            # remove an existing key one level down
    DeepChainMap({'zebra': 'black', 'snake': 'red'}, {}, {'lion': 'orange'})

## Counter

用Counter做简单的统计再合适不过了

Counter接受两种参数，iterable或mapping。

    >>> collections.Counter('gallahad')
    Counter({'a': 3, 'l': 2, 'd': 1, 'g': 1, 'h': 1})
    >>> collections.Counter({'red': 4, 'blue': 2})
    Counter({'red': 4, 'blue': 2})
    >>> collections.Counter(cats=4, dogs=8)
    Counter({'dogs': 8, 'cats': 4})

从这里看，Counter实际上就是dict的子类。

    >>> import re
    >>> words = re.findall(r'\w+', open('hamlet.txt').read().lower())
    >>> Counter(words).most_common(10)
    [('the', 1143), ('and', 966), ('to', 762), ('of', 669), ('i', 631),
     ('you', 554),  ('a', 546), ('my', 514), ('hamlet', 471), ('in', 451)]

不过，我们最常用的应该还是这个`most_common()`，用于统计出现最频繁的键。

另外，还可以用Counter作向量运算：

    >>> c = Counter(a=3, b=1)
    >>> d = Counter(a=1, b=2)
    >>> c + d                       # add two counters together:  c[x] + d[x]
    Counter({'a': 4, 'b': 3})
    >>> c - d                       # subtract (keeping only positive counts)
    Counter({'a': 2})
    >>> c & d                       # intersection:  min(c[x], d[x])
    Counter({'a': 1, 'b': 1})
    >>> c | d                       # union:  max(c[x], d[x])
    Counter({'a': 3, 'b': 2})

不过，既然是统计，Counter默认去掉了运算结果为负的键值。还好有`subtract()`可以补救一下：

    >>> c.subtract(d)
    >>> c
    Counter({'a': 2, 'b': -1})

需要注意的是，Counter被设计用来做统计的，所以很多方法都默认忽略负值。

对于Counter来说，虽然每个键的值理应存储该键的出现次数，但Counter作为dict的子类，在值的类型上并没有做限制。

## deque

deque是"double-ended queue"的缩写，发音为"deck"。

deque是线程安全的，在队列两端的操作复杂度为O(1)。

相比而言，list更适合做`pop(n)`, `append(v)`这种队列定长操作，对于`pop(0)`, `insert(0, v)`操作的复杂度会上升为O(n)。自然是deque更适合这种场景。

但是，虽然deque在队列两端操作很快，但是对于中间元素的操作却没那么高效了，所以对于随机访问队列的场景而言，还是用list吧。

deque支持list的大多数操作，如：正负向索引、迭代器、序列化、反转、深浅拷贝、`in`运算符等。不过deque **不支持** list常用的切片操作(slice)。

    >>> import timeit
    >>> timeit.timeit('for i in range(10000): d.append(i)', setup='from collections import deque; d = deque()', number=1)
    0.0010227239999949234
    >>> timeit.timeit('for i in range(10000): d.appendleft(i)', setup='from collections import deque; d = deque()', number=1)
    0.000990255999568035
    
    >>> timeit.timeit('for i in range(10000): l.append(i)', setup='l = list()', number=1)
    0.000842813999952341
    >>> timeit.timeit('for i in range(10000): l.insert(0, i)', setup='l = list()', number=1)
    0.023448073000508884
    
    >>> timeit.timeit('for i in range(10000): d.pop()', setup='from collections import deque; d = deque(range(10000))', number=1)
    0.0008286150005005766
    >>> timeit.timeit('for i in range(10000): d.popleft()', setup='from collections import deque; d = deque(range(10000))', number=1)
    0.0008930439998948714
    
    >>> timeit.timeit('for i in range(10000): l.pop()', setup='l = list(range(10000))', number=1)
    0.001184474999718077
    >>> timeit.timeit('for i in range(10000): l.pop(0)', setup='l = list(range(10000))', number=1)
    0.013785433000521152

deque中的`rotate(n)`是list中没有的，等价于把最后n个元素添加到队头，若n为负数，则是从头取元素添加至队尾。等价于`d.appendleft(d.pop())`。


## defaultdict

一个dict的子类，只是对访问不存在键时有额外的操作，通过参数`default_factory`指定。

dict在访问不存在键时(d[undef])，会`raise KeyError`；

defaultdict在被访问不存在键时(d[undef])，若未指定(`default_factory=None`)，则与dict一样`raise KeyError`，若指定(`default_factory=func`)，则利用指定函数新建键值(d[undef] = func())。

如，用来统计单词出现位置：

    >>> from collections import defaultdict
    >>> s = "the quick brown fox jumps over the lazy dog"
    >>> words = s.split()
    >>> location = defaultdict(list)
    >>> for m, n in enumerate(words):
    ...     location[n].append(m)
    >>> location
    defaultdict(<class 'list'>, {'quick': [1], 'the': [0, 6], 'fox': [3], 'jumps': [4], 'brown': [2], 'dog': [8], 'lazy': [7], 'over': [5]})

## namedtuple

namedtuple是一个Factory Function，其返回值是一个tuple的子类，类名由参数`typename`指定；初始化参数由`field_names`指定。

如果我们在读取格式化文件或访问数据库时，不想新建类来维护每一行数据，就可以用namedtuple偷个懒。

    EmployeeRecord = namedtuple('EmployeeRecord', 'name, age, title, department, paygrade')
    
    import csv
    for emp in map(EmployeeRecord._make, csv.reader(open("employees.csv", "rb"))):
        print(emp.name, emp.title)
    
    import sqlite3
    conn = sqlite3.connect('/companydata')
    cursor = conn.cursor()
    cursor.execute('SELECT name, age, title, department, paygrade FROM employees')
    for emp in map(EmployeeRecord._make, cursor.fetchall()):
        print(emp.name, emp.title)

如果打开`verbose`开关或访问`_source`属性，会得到返回类型的Python代码：

    >>> Point = collections.namedtuple('Point', ['x', 'y'], verbose=True)
    from builtins import property as _property, tuple as _tuple
    from operator import itemgetter as _itemgetter
    from collections import OrderedDict
    
    class Point(tuple):
        'Point(x, y)'
    
        __slots__ = ()
    
        _fields = ('x', 'y')
    
        def __new__(_cls, x, y):
            'Create new instance of Point(x, y)'
            return _tuple.__new__(_cls, (x, y))
    
        @classmethod
        def _make(cls, iterable, new=tuple.__new__, len=len):
            'Make a new Point object from a sequence or iterable'
            result = new(cls, iterable)
            if len(result) != 2:
                raise TypeError('Expected 2 arguments, got %d' % len(result))
            return result
    
        def _replace(_self, **kwds):
            'Return a new Point object replacing specified fields with new values'
            result = _self._make(map(kwds.pop, ('x', 'y'), _self))
            if kwds:
                raise ValueError('Got unexpected field names: %r' % list(kwds))
            return result
    
        def __repr__(self):
            'Return a nicely formatted representation string'
            return self.__class__.__name__ + '(x=%r, y=%r)' % self
    
        @property
        def __dict__(self):
            'A new OrderedDict mapping field names to their values'
            return OrderedDict(zip(self._fields, self))
    
        def _asdict(self):
            '''Return a new OrderedDict which maps field names to their values.
               This method is obsolete.  Use vars(nt) or nt.__dict__ instead.
            '''
            return self.__dict__
    
        def __getnewargs__(self):
            'Return self as a plain tuple.  Used by copy and pickle.'
            return tuple(self)
    
        def __getstate__(self):
            'Exclude the OrderedDict from pickling'
            return None
    
        x = _property(_itemgetter(0), doc='Alias for field number 0')
    
        y = _property(_itemgetter(1), doc='Alias for field number 1')

看到这个，就可以清楚明了的操作namedtuple返回的类了。

新建实例：

    >>> p = Point(2, 4)
    >>> Point(*[4, 8])
    Point(x=4, y=8)
    >>> Point(**{'x': 5, 'y': 10})
    Point(x=5, y=10)

通过内建函数`vars()`获得实例的OrderedDict：

    >>> vars(p)
    OrderedDict([('x', 2), ('y', 4)])

通过`_make()`新建实例：

    >>> Point._make([3, 6])
    Point(x=3, y=6)

通过`_replace`修改成员：

    >>> p._replace(y=8)
    Point(x=2, y=8)

通过`_fields`查看成员列表：

    >>> p._fields
    ('x', 'y')

需要注意的是，namedtuple返回的是tuple的子类，所以tuple支持的操作对于namdtuple来说是同样适用的。

## OrderedDict

看名字就能知道这个类型的功能，就是一个记录了元素插入顺序dict。

特有的方法有两个：`move_to_end(key, last=True)`, `popitem(last=True)`

    >>> d = {'banana': 3, 'apple':4, 'pear': 1, 'orange': 2}
    >>> od = OrderedDict(sorted(d.items(), key=lambda t: t[1]))    # OrderedDict([('pear', 1), ('orange', 2), ('banana', 3), ('apple', 4)])
    >>> od.move_to_end('pear')                                     # OrderedDict([('orange', 2), ('banana', 3), ('apple', 4), ('pear', 1)])
    >>> od.move_to_end('pear', last=False)                         # OrderedDict([('pear', 1), ('orange', 2), ('banana', 3), ('apple', 4)])
    >>> od.popitem()
    ('apple', 4)
    >>> od.popitem(last=False)
    ('pear', 1)

