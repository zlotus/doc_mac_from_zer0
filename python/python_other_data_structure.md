# Other Data Structure

[Advanced Data Structure](http://pypix.com/python/advanced-data-structures-python/)

## array

array是一种擅长存储同种类型数据的容器，它会非常节省内存空间。

    >>> a = array.array("i", [1,2,3,4,5])
    >>> b = array.array(a.typecode, (2*x for x in a))

不过，array对元素的操作比list慢。由于array节省空间的特性，所以对array的操作也应该配合使用节省空间的算法，如in-place等。

    >>> a = array.array("i", [1,2,3,4,5])
    >>> for i, x in enumerate(a):
    ...     a[i] = 2*x

可以测试一下时间开销：

    >>> generatortest = """
    ... def generatortest():
    ...     a = array.array("i", [1, 2, 3, 4, 5])
    ...     b = array.array(a.typecode, (2 * x for x in a))
    ... """
    >>> inplacetest = """
    ... def inplacetest():
    ...     a = array.array("i", [1, 2, 3, 4, 5])
    ...     for i, x in enumerate(a):
    ...         a[i] = 2 * x
    ... """
    >>> importtest = """
    ... import array
    ... import timeit
    ... """
    >>> import timeit
    >>> timeit.timeit("generatortest()", importtest+generatortest)
    2.306710820999797
    >>> timeit.timeit("inplacetest()", importtest+inplacetest)
    1.5715019919998667

## heapq

heapq就是heap queue的缩写。

常用来排序：

    >>> import heapq
    >>> for value in range(10):
    ...     heapq.heappush(heap, random.randrange(10))
    ... 
    >>> heap
    [0, 1, 1, 5, 1, 8, 6, 7, 8, 3]

`nlargest(n, iterable, key=None)`和`nlargest(n, iterable, key=None)`是很有意思的两个函数。

比如用来根据指定的字段找出字典中的最值：

    >>> portfolio = [
    ... {'name': 'IBM', 'shares': 100, 'price': 91.1},
    ... {'name': 'AAPL', 'shares': 50, 'price': 543.22},
    ... {'name': 'FB', 'shares': 200, 'price': 21.09},
    ... {'name': 'HPQ', 'shares': 35, 'price': 31.75},
    ... {'name': 'YHOO', 'shares': 45, 'price': 16.35},
    ... {'name': 'ACME', 'shares': 75, 'price': 115.65}
    ... ]
    ... 
    cheap = heapq.nsmallest(3, portfolio, key=lambda s: s['price'])    # [{'price': 16.35, 'shares': 45, 'name': 'YHOO'}, {'price': 21.09, 'shares': 200, 'name': 'FB'}, {'price': 31.75, 'shares': 35, 'name': 'HPQ'}]
    
    expensive = heapq.nlargest(3, portfolio, key=lambda s: s['price']) # [{'price': 543.22, 'shares': 50, 'name': 'AAPL'}, {'price': 115.65, 'shares': 75, 'name': 'ACME'}, {'price': 91.1, 'shares': 100, 'name': 'IBM'}]

堆元素可以是tuple，这对于带有优先级的实例很有用：

    >>> h = []
    >>> heappush(h, (5, 'write code'))
    >>> heappush(h, (7, 'release product'))
    >>> heappush(h, (1, 'write spec'))
    >>> heappush(h, (3, 'create tests'))
    >>> heappop(h)
    (1, 'write spec')

## bisect

bisect是bisection的缩写，也就是它使用二分法，经常被用来在保持列表有序的情况下插入元素。

用`insort_right()`从尾端插入元素（假设列表是有序的），它的行为类似`a.insert(bisect.bisect_right(a, x), x)`：

    >>> import bisect
    >>> a = [(0, 100), (150, 220), (500, 1000)]
    >>> bisect.insort_right(a, (250,400)        # [(0, 100), (150, 220), (250, 400), (500, 1000)]

用`bisect.bisect()`查找插入点（）：

    >>> bisect.insort_right(a, (399, 450))      # [(0, 100), (150, 220), (250, 400), (399, 450), (500, 1000)]
    >>> bisect.bisect(a, (550, 1200))
    5

然后插入：

    >>> bisect.insort_right(a, (550, 1200))     # [(0, 100), (150, 220), (250, 400), (399, 450), (500, 1000), (550, 1200)]

## weakref

先看看什么是strong reference：

    >>> class Foo(object):
    ...     def __init__(self):
    ...         self.obj = None
    ...         print('created')
    ...  
    ...     def __del__(self):
    ...         print ('destroyed')
    ...  
    ...     def show(self):
    ...         print(self.obj)
    ...  
    ...     def store(self, obj):
    ...         self.obj = obj
    ... 
    >>> a = Foo()
    created
    >>> id(a)
    4429650072
    >>> b = a
    >>> id(b)
    4429650072
    >>> del b
    >>> del a
    destroyed

这里的a, b是Foo()的同一个实例的strong reference，只删除b并不会导致这个实例对象被删除，只有当该对象的所有strong reference都被删除时，这个对象才会被回收。

现代语言的垃圾回收会判定一个对象的引用类型，若该对象仅剩weak reference时，对象会被回收。

可以通过weakref创建对象的weak reference：

    >>> import weakref
    >>> a = Foo()           # <Foo object at 0x10808a630>
    created
    >>> b = weakref.ref(a)  # <weakref at 0x108674138; to 'Foo' at 0x10808a630>

通过`b()`可以得到该对象的临时strong reference：

    >>> a == b()
    True
    >>> b().show()
    None

此时删除a，即该对象的strong reference，对象会被立刻删除：

    >>> del a
    destroyed

这时，weak reference不再起作用：

    >>> b() is None
    True

使用`weakref.proxy()`也可以做同样的工作，看起来更顺眼：

    >>> a = Foo()
    created
    >>> b = weakref.proxy(a)
    >>> b.store('fish')
    >>> b.show()
    fish
    >>> del a
    destroyed
    >>> b.show()
    Traceback (most recent call last):
    ReferenceError: weakly-referenced object no longer exists

## types

除了提供动态创建类的函数`types.new_class(name, bases=(), kwds=None, exec_body=None)`以外，还提定义了一些被Python解释器使用，但是没有在内建类型中显示定义（如int, str）的类型：

    >>> pprint.pprint([t for t in dir(types) if not '_' in t])
    ['BuiltinFunctionType',
     'BuiltinMethodType',
     'CodeType',
     'DynamicClassAttribute',
     'FrameType',
     'FunctionType',
     'GeneratorType',
     'GetSetDescriptorType',
     'LambdaType',
     'MappingProxyType',
     'MemberDescriptorType',
     'MethodType',
     'ModuleType',
     'SimpleNamespace',
     'TracebackType']


## copy

Python中的赋值操作只会把对象绑定到一个名字上（*引用*），不会复制该对象。

有时我们想要复制一个可变容器，或是容器中包含可变元素时，我们是想要得到一个全新的容器，这样就可以修改新容器中的元素而不对原来的容器及其对象产生影响了。这是我们需要使用到copy模块。

shallow copy与deep copy只在compound objects(一个包含了其他对象的对象，如list、class、instance)上体现差异：

* shallow copy会创建一个新的compound objects，并将原对象中的其他对象以*引用*的方式插入这个新的compound objects。

* deep copy会创建一个新的compound object，并将源对象中的其他对象以*复制*的方式递归的插入这个新的 compound object。

试验一下：

    >>> import copy
    >>> a = [1, 2, 3]
    >>> b = [4, 5, 6]
    >>> c = [a, b]            # compound object
    >>> d = c                 # assignment, nothing is new
    >>> id(c) == id(d)
    True
    >>> id(c[0]) == id(d[0])
    True
    >>> e = copy.copy(c)      # shallow copy, compound object itself is new
    >>> id(c) == id(e)
    False
    >>>  id(c[0]) == id(e[0])
    >>> id(c[0]) == id(e[0])
    True
    >>> f = copy.deepcopy(c)  # deep copy, everything is new
    >>> id(c) == id(f)
    False
    >>> id(c[0]) == id(f[0])
    False

## pprint

让输出变得更好看。

主要是用于当list、dict等集合递归包含了很多其他的容器类型时，用缩进表示递归层级，使得输出错落有致。

    import requests
    r = requests.get('http://pypi.python.org/pypi/Twisted/json')
    pprint.pprint(r.text)
    >>> pprint.pprint(project_info)
    {'info': {'_pypi_hidden': False,
              '_pypi_ordering': 125,
              'author': 'Glyph Lefkowitz',
              'author_email': 'glyph@twistedmatrix.com',
              'bugtrack_url': '',
              'cheesecake_code_kwalitee_id': None,
              'cheesecake_documentation_id': None,
              'cheesecake_installability_id': None,
              'classifiers': ['Programming Language :: Python :: 2.6',
                              'Programming Language :: Python :: 2.7',
                              'Programming Language :: Python :: 2 :: Only'],
              'description': 'An extensible framework for Python programming, '
                             'with special focus\r\n'
                             'on event-based network programming and '
                             'multiprotocol integration.',
              'docs_url': '',
              'download_url': 'UNKNOWN',
              'home_page': 'http://twistedmatrix.com/',
              'keywords': '',
              'license': 'MIT',
              'maintainer': '',
              'maintainer_email': '',
              'name': 'Twisted',
              'package_url': 'http://pypi.python.org/pypi/Twisted',
              'platform': 'UNKNOWN',
              'release_url': 'http://pypi.python.org/pypi/Twisted/12.3.0',
              'requires_python': None,
              'stable_version': None,
              'summary': 'An asynchronous networking framework written in Python',
              'version': '12.3.0'},
     'urls': [{'comment_text': '',
               'downloads': 71844,
               'filename': 'Twisted-12.3.0.tar.bz2',
               'has_sig': False,
               'md5_digest': '6e289825f3bf5591cfd670874cc0862d',
               'packagetype': 'sdist',
               'python_version': 'source',
               'size': 2615733,
               'upload_time': '2012-12-26T12:47:03',
               'url': 'https://pypi.python.org/packages/source/T/Twisted/Twisted-12.3.0.tar.bz2'},
              {'comment_text': '',
               'downloads': 5224,
               'filename': 'Twisted-12.3.0.win32-py2.7.msi',
               'has_sig': False,
               'md5_digest': '6b778f5201b622a5519a2aca1a2fe512',
               'packagetype': 'bdist_msi',
               'python_version': '2.7',
               'size': 2916352,
               'upload_time': '2012-12-26T12:48:15',
               'url': 'https://pypi.python.org/packages/2.7/T/Twisted/Twisted-12.3.0.win32-py2.7.msi'}]}

也可以指定递归层数：

    >>> pprint.pprint(project_info, depth=2)
    {'info': {'_pypi_hidden': False,
              '_pypi_ordering': 125,
              'author': 'Glyph Lefkowitz',
              'author_email': 'glyph@twistedmatrix.com',
              'bugtrack_url': '',
              'cheesecake_code_kwalitee_id': None,
              'cheesecake_documentation_id': None,
              'cheesecake_installability_id': None,
              'classifiers': [...],
              'description': 'An extensible framework for Python programming, '
                             'with special focus\r\n'
                             'on event-based network programming and '
                             'multiprotocol integration.',
              'docs_url': '',
              'download_url': 'UNKNOWN',
              'home_page': 'http://twistedmatrix.com/',
              'keywords': '',
              'license': 'MIT',
              'maintainer': '',
              'maintainer_email': '',
              'name': 'Twisted',
              'package_url': 'http://pypi.python.org/pypi/Twisted',
              'platform': 'UNKNOWN',
              'release_url': 'http://pypi.python.org/pypi/Twisted/12.3.0',
              'requires_python': None,
              'stable_version': None,
              'summary': 'An asynchronous networking framework written in Python',
              'version': '12.3.0'},
     'urls': [{...}, {...}]}

或是设置每行长度最大值，但是当遇到超过设置长度的不可分割对象出现时，设置的长度会被超出：

    >>> pprint.pprint(project_info, depth=2, width=50)
    {'info': {'_pypi_hidden': False,
              '_pypi_ordering': 125,
              'author': 'Glyph Lefkowitz',
              'author_email': 'glyph@twistedmatrix.com',
              'bugtrack_url': '',
              'cheesecake_code_kwalitee_id': None,
              'cheesecake_documentation_id': None,
              'cheesecake_installability_id': None,
              'classifiers': [...],
              'description': 'An extensible '
                             'framework for '
                             'Python programming, '
                             'with special '
                             'focus\r\n'
                             'on event-based '
                             'network programming '
                             'and multiprotocol '
                             'integration.',
              'docs_url': '',
              'download_url': 'UNKNOWN',
              'home_page': 'http://twistedmatrix.com/',
              'keywords': '',
              'license': 'MIT',
              'maintainer': '',
              'maintainer_email': '',
              'name': 'Twisted',
              'package_url': 'http://pypi.python.org/pypi/Twisted',
              'platform': 'UNKNOWN',
              'release_url': 'http://pypi.python.org/pypi/Twisted/12.3.0',
              'requires_python': None,
              'stable_version': None,
              'summary': 'An asynchronous '
                         'networking framework '
                         'written in Python',
              'version': '12.3.0'},
     'urls': [{...}, {...}]}

## enum

这是Python3.4新加入的模块。

其中提供了两个class：`Enum`, `IntEnum`，和一个decorator：`unique()`。

### 新建Enum

新建Enum与新建类一样，可读性强：

    >>> from enum import Enum
    >>> class Color(Enum):
    ...     red = 1
    ...     green = 2
    ...     blue = 3
    ...

同样具有可读性的输出表达：

    >>> print(Color.red)
    Color.red

使用`repr`得到更多信息：

    >>> print(repr(Color.red))
    <Color.red: 1>

使用`type`检视Enum的成员，成员的type是其所属的Enum类型：

    >>> type(Color.red)
    <enum 'Color'>
    >>> isinstance(Color.green, Color)
    True

Enum成员都具有一个名为`name`的属性：

    >>> print(Color.red.name)
    red

Enum自然支持iteration：

    >>> class Shake(Enum):
    ...     vanilla = 7
    ...     chocolate = 4
    ...     cookies = 9
    ...     mint = 3
    ...
    >>> for shake in Shake:
    ...     print(shake)
    ...
    Shake.vanilla
    Shake.chocolate
    Shake.cookies
    Shake.mint

Enum成员是hashable的，所以可以作为dict的键或是成为set成员：

    >>> apples = {}
    >>> apples[Color.red] = 'red delicious'
    >>> apples[Color.green] = 'granny smith'
    >>> apples == {Color.red: 'red delicious', Color.green: 'granny smith'}
    True

### 程序化访问Enum成员

比如当我们不知道一个Enum类中有那些成员，所以无法使用诸如`Color.red`这种写法时：

    >>> Color(1)
    <Color.red: 1>
    >>> Color(3)
    <Color.blue: 3>

也可以通过成员名称访问，就像使用dict一样：

    >>> Color['red']
    <Color.red: 1>
    >>> Color['green']
    <Color.green: 2>

或是直接访问成员的`name`和`value`属性：

    >>> member = Color.red
    >>> member.name
    'red'
    >>> member.value
    1

### 重复的成员和重复的值

Enum不允许定义重复的成员：

    >>> class Shape(Enum):
    ...     square = 2
    ...     square = 3
    ...
    Traceback (most recent call last):
    ...
    TypeError: Attempted to reuse key: 'square'

不过Enum是允许定义重复的值的，只是在访问时需要注意，不论是通过成员名称访问还是通过成员值访问，Enum只返回先定义的那个成员：

    >>> class Shape(Enum):
    ...     square = 2
    ...     diamond = 1
    ...     circle = 3
    ...     alias_for_square = 2
    ...
    >>> Shape.square
    <Shape.square: 2>
    >>> Shape.alias_for_square
    <Shape.square: 2>
    >>> Shape(2)
    <Shape.square: 2>


