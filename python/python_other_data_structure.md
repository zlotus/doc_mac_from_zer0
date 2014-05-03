# 一些别的数据结构

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

### 确保值的唯一性

尽管从上面我们可以看到，Enum是允许重复值出现的，但如果我们不希望Enum中同一个值有两个别名时，可以使用`@enum.unique`。这个decorator能够检查Enum中是否有重复的值，如果有则会`raise ValueError`：

    >>> from enum import Enum, unique
    >>> @unique
    ... class Mistake(Enum):
    ...     one = 1
    ...     two = 2
    ...     three = 3
    ...     four = 3
    ...
    Traceback (most recent call last):
    ...
    ValueError: duplicate values found in <enum 'Mistake'>: four -> three

### 迭代

Enum在迭代的时候是排除别名的，也就是对同一个值来说，最先定义在Enum中的成员才会出现，在`Shape`这个例子中，`alias_for_square`是不会出现在iteration中的：

    >>> list(Shape)
    [<Shape.square: 2>, <Shape.diamond: 1>, <Shape.circle: 3>]

`__member__`属性是一个OrderedDict，它包含了所以的Enum成员，包括同一个值出现的别名，在`Shape`中，`alias_for_square`将出现在`__member__`中：

    >>> for name, member in Shape.__members__.items():
    ...     name, member
    ...
    ('square', <Shape.square: 2>)
    ('diamond', <Shape.diamond: 1>)
    ('circle', <Shape.circle: 3>)
    ('alias_for_square', <Shape.square: 2>)

`__member__`的程序化访问：

    >>> [name for name, member in Shape.__members__.items() if member.name != name]
    ['alias_for_square']

## 比较

Enum类型可以使用`is`运算符比较成员：

    >>> Color.red is Color.red
    True
    >>> Color.red is Color.blue
    False
    >>> Color.red is not Color.blue
    True

Enum不能比较大小，因为Enum成员并不是Int类型（另有`IntEnum`类型实现类似功能）：

    >>> Color.red < Color.blue
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: unorderable types: Color() < Color()

不过Enum的成员可以做等价比较：

    >>> Color.blue == Color.red
    False
    >>> Color.blue != Color.red
    True
    >>> Color.blue == Color.blue
    True

注意，Enum成员和非Enum成员作比较时结果始终是不相等（`IntEnum`类专门用来做Int相关的操作）：

    >>> Color.blue == 2
    False

### 允许的成员及属性

上面的例子都使用Int做成员值，因为Int是常用且简便的，但这并不意味着强制使用Int。Enum并不关心值的类型，值的类型有很大的随意性。

Enum是一个Python类，所以和其他类一样可以定义方法或特殊方法：

    >>> class Mood(Enum):
    ...     funky = 1
    ...     happy = 3
    ...
    ...     def describe(self):
    ...         # self is the member here
    ...         return self.name, self.value
    ...
    ...     def __str__(self):
    ...         return 'my custom str! {0}'.format(self.value)
    ...
    ...     @classmethod
    ...     def favorite_mood(cls):
    ...         # cls here is the enumeration
    ...         return cls.happy
    ...
    >>> Mood.favorite_mood()
    <Mood.happy: 3>
    >>> Mood.happy.describe()
    ('happy', 3)
    >>> str(Mood.funky)
    'my custom str! 1'

注意，形如`_sunder_`（前后各一个下划线）的名称为Enum的保留项，不允许使用这类名称；其他所有在Enum中定义的属性都会成为Enum的成员，形如`__dunder__`（前后各两个下划线）的名称和descriptor是例外（方法也是descriptor，详见另一篇文章：[python_descriptor](https://github.com/zlotus/doc_mac_from_zer0/blob/master/python/python_descriptor.md)）

如果Enum中定义了`__new__()`或`__init__()`，不论Enum成员值是什么，这个值都会被当做参数传入这两个特殊方法：

参考下面的示例部分可以深入理解定义这两个方法后子类具有的行为。

### Enum的继承限制

只有在一个Enum类中不包含任何成员时，这个类才可以作为父类，所以这样的代码是不可以的：

    >>> class MoreColor(Color):
    ...     pink = 17
    ...
    Traceback (most recent call last):
    ...
    TypeError: Cannot extend enumerations

但可以这样：

    >>> class Foo(Enum):
    ...     def some_behavior(self):
    ...         pass
    ...
    >>> class Bar(Foo):
    ...     happy = 1
    ...     sad = 2
    ...

### Python持久化

可持久化的Enum类必须定义在模块顶层，因为解包时需要保证他们是可导入的。

    >>> from test.test_enum import Fruit
    >>> from pickle import dumps, loads
    >>> Fruit.tomato is loads(dumps(Fruit.tomato))
    True

通过定义`__reduce_ex__()`可以改变pickle/unpickle行为。

### 函数式API

Enum类是可调用的：

    >>> Animal = Enum('Animal', 'ant bee cat dog')
    >>> Animal
    <enum 'Animal'>
    >>> Animal.ant
    <Animal.ant: 1>
    >>> Animal.ant.value
    1
    >>> list(Animal)
    [<Animal.ant: 1>, <Animal.bee: 2>, <Animal.cat: 3>, <Animal.dog: 4>]

类似`namedtuple`，第一个参数是Enum的类名；

第二个参数是Enum成员的来源，这个来源可以是：

* 一个包含成员名称的字符串，各成员名间使用空格隔开；
* 一个成员名称的序列；
* 一个2元组的序列，元组形式为键-值；
* 一个映射，比如dict；

后两个来源能够给Enum成员的值进行自由赋值，其他的来源会给成员赋上从1开始自增的整数值。

调用`Enum()`返回一个新的Enum子类，上面的代码等同于：

    >>> class Animals(Enum):
    ...     ant = 1
    ...     bee = 2
    ...     cat = 3
    ...     dog = 4
    ...

之所以从1开始自增是因为从布尔值的角度看，`0==False`，而所有Enum成员应为`True`。

持久化使用函数式创建的Enum类型会因为frame stack实现的细节会尝试找到该Enum类是创建在那个模块中的（如：使用在另一个模块中的工具函数或使用IronPython和Jython会导致失败）。解决方法是明确指定模块名：

    >>> Animals = Enum('Animals', 'ant bee cat dog', module=__name__)

如果没指定模块，且Enum尝试找出模块未果时，返回的新的Enum子类将不可持久化（unpicklable）

### Enum子类：IntEnum

IntEnum也是Int的子类，所以IntEnum的成员可以与整数做比较。同样的，不同类型的IntEnum类成员可以互相做比较：

    >>> enum.IntEnum.__mro__
    (<enum 'IntEnum'>, <class 'int'>, <enum 'Enum'>, <class 'object'>)
    >>> from enum import IntEnum
    >>> class Shape(IntEnum):
    ...     circle = 1
    ...     square = 2
    ...
    >>> class Request(IntEnum):
    ...     post = 1
    ...     get = 2
    ...
    >>> Shape == 1
    False
    >>> Shape.circle == 1
    True
    >>> Shape.circle == Request.post
    True

但是，IntEnum类型成员仍然不能与Enum类型成员做比较：

    >>> class Shape(IntEnum):
    ...     circle = 1
    ...     square = 2
    ...
    >>> class Color(Enum):
    ...     red = 1
    ...     green = 2
    ...
    >>> Shape.circle == Color.red
    False

IntEnum值的行为都类似整型：

    >>> int(Shape.circle)
    1
    >>> ['a', 'b', 'c'][Shape.circle]
    'b'
    >>> [i for i in range(Shape.square)]
    [0, 1]

注意，在大多数情形下，推荐使用Enum，因为IntEnum违反了一些枚举类型语义上的协议（通过与整型的比较，可以传递给其他不相关的枚举类型）。IntEnum只用于一些别无选择的情形下，如：整型常量被替换成为枚举类型的同时，需要满足向后兼容代码仍然需要整型值。

虽然IntEnum是enum模块的一部分，但是我们可以用很简单的代码实现这个类型：

    class IntEnum(int, Enum):
        pass

代码展示了如何定义Enum衍生类，比如将int替换为str，从而得到一个strEnum类：

关于衍生类的一些规定：

1. 在父类列表中，混合类型应当出现在Enum类前，就像上面的IntEnum演示的那样。
2. 虽然Enum成员值可以是任何类型，但是一点混合了别的类型，所以成员值都必须使用该类型。这个限制不适用于只添加方法而不指定另一个数据类型的情形。
3. 一旦混合了别的类型，`value`属性就不再是enum成员本身了，虽然它们是等价的，而且比较时也相等。
4. 字符串的`%`运算符：`%s`和`%r`分别调用Enum类型的`__str__()`和`__repr__()`方法；其他的参数（如IntEnum的`%i`和`%h`）会把成员的类型看做混合类型对待。
5. `str.__format__()`或`format()`函数会调用混合类型的`__format__()`方法。如果调用Enum的`str()`和`repr()`，使用`!s`和`!r`作为参数。

### 示例

利用`__new__()`，自动为成员分配值：

    >>> class AutoNumber(Enum):
    ...     def __new__(cls):
    ...         value = len(cls.__members__) + 1
    ...         obj = object.__new__(cls)
    ...         obj._value_ = value
    ...         return obj
    ...
    >>> class Color(AutoNumber):
    ...     red = ()
    ...     green = ()
    ...     blue = ()
    ...
    >>> Color.green.value == 2
    True

如果定义了`__new__()`，则会再创建成员时调用；而后这个自定义的`__new__()`会被Enum的`__new__()`代替（Enum的`__new__()`会在新建类之后调用，用于查找已有成员）。因为要保证枚举类型应有的行为，所以没有办法直接修改Enum的`__new__()`。


利用`__init__()`实现当出现成员值重复的情况时抛出异常：

    >>> class DuplicateFreeEnum(Enum):
    ...     def __init__(self, *args):
    ...         cls = self.__class__
    ...         if any(self.value == e.value for e in cls):
    ...             a = self.name
    ...             e = cls(self.value).name
    ...             raise ValueError(
    ...                 "aliases not allowed in DuplicateFreeEnum:  %r --> %r"
    ...                 % (a, e))
    ...
    >>> class Color(DuplicateFreeEnum):
    ...     red = 1
    ...     green = 2
    ...     blue = 3
    ...     grene = 2
    ...
    Traceback (most recent call last):
    ...
    ValueError: aliases not allowed in DuplicateFreeEnum:  'grene' --> 'green'

如果定义了`__init__()`，则会在初始化子类成员时调用，从上面的代码可以看出，调用`__init__()`时子类的成员已经创建完毕了。这段代码实现了`enum.unique()`的功能，主要是示例如何自定义实现诸如“禁止成员值重复”这样的Enum子类行为。

再来看看初始化时，成员如何向`__new__()`或`__init__()`传递参数：

    >>> class Planet(Enum):
    ...     MERCURY = (3.303e+23, 2.4397e6)
    ...     VENUS   = (4.869e+24, 6.0518e6)
    ...     EARTH   = (5.976e+24, 6.37814e6)
    ...     MARS    = (6.421e+23, 3.3972e6)
    ...     JUPITER = (1.9e+27,   7.1492e7)
    ...     SATURN  = (5.688e+26, 6.0268e7)
    ...     URANUS  = (8.686e+25, 2.5559e7)
    ...     NEPTUNE = (1.024e+26, 2.4746e7)
    ...     def __init__(self, mass, radius):
    ...         self.mass = mass       # in kilograms
    ...         self.radius = radius   # in meters
    ...     @property
    ...     def surface_gravity(self):
    ...         # universal gravitational constant  (m3 kg-1 s-2)
    ...         G = 6.67300E-11
    ...         return G * self.mass / (self.radius * self.radius)
    ...
    >>> Planet.EARTH.value
    (5.976e+24, 6378140.0)
    >>> Planet.EARTH.surface_gravity
    9.802652743337129

上面提到IntEnum违反了一下枚举类型应有的行为，其实可以保持Enum的默认行为，并使之支持比较运算符，可以这样：

    >>> class OrderedEnum(Enum):
    ...     def __ge__(self, other):
    ...         if self.__class__ is other.__class__:
    ...             return self.value >= other.value
    ...         return NotImplemented
    ...     def __gt__(self, other):
    ...         if self.__class__ is other.__class__:
    ...             return self.value > other.value
    ...         return NotImplemented
    ...     def __le__(self, other):
    ...         if self.__class__ is other.__class__:
    ...             return self.value <= other.value
    ...         return NotImplemented
    ...     def __lt__(self, other):
    ...         if self.__class__ is other.__class__:
    ...             return self.value < other.value
    ...         return NotImplemented
    ...
    >>> class Grade(OrderedEnum):
    ...     A = 5
    ...     B = 4
    ...     C = 3
    ...     D = 2
    ...     F = 1
    ...
    >>> Grade.C < Grade.A
    True

始终记得Enum也是一个Python类，所以那些增强其他Python类功能的方法对Enum也同样适用。
