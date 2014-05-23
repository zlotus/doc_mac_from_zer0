# 我不知道的Python

最近按照知道创宇的技能表加天赋，开始了大坑无数的阅读Python document的路程。

缓慢填坑。

## `for`...`else`...语句是允许的

## hashable class

在 **Data Model** 一节，`object.__hash__(self)`下有这么一段说明：

一个重写了`__eq__()`的类，如果不显式定义`__hash__()`：

* 会导致该类实例 unhashable，如：不能做dict的key，不能加入set；
* `hash(instance)` 会报 TypeError；
* `isinstance(instance, collections.Hashable)` 返回 False。

所以，如果重写了 `__eq__()`，同时想保持该类hashable，就必须显式定义 `__hash__()`。

如：`__hash__ = <ParentClass>.__hash__`

如果想在不定义 `__eq__()` 的情况下，去掉类的hashable属性：`__hash__ = None` 即可。

## 用`zip()`进行简单的矩阵转置

    >>> m = [[1,2,3], [4,5,6], [7,8,9]]
    >>> mt = list(zip(*m))
    >>> mt
    [(1, 4, 7), (2, 5, 8), (3, 6, 9)]

## `re`中的`group()`可以接受多个组号

    >>> p = re.compile(r'(a(b)c)d')
    >>> m = p.match('abcd')
    >>> m.group(2,1,2)
    ('b', 'abc', 'b')

## 用`difflib`查找变动

这个库的用法和输出非常类似`diff` `-c` `-u`或`git diff`，常用语比对文件改动。基于gestalt pattern matching。

可以得到这种效果：

    >>> s1 = ['bacon\n', 'eggs\n', 'ham\n', 'guido\n']
    >>> s2 = ['python\n', 'eggy\n', 'hamster\n', 'guido\n']
    >>> for line in context_diff(s1, s2, fromfile='before.py', tofile='after.py'):
    ...     sys.stdout.write(line)
    *** before.py
    --- after.py
    ***************
    *** 1,4 ****
    ! bacon
    ! eggs
    ! ham
      guido
    --- 1,4 ----
    ! python
    ! eggy
    ! hamster
      guido

或是：

    >>> s1 = ['bacon\n', 'eggs\n', 'ham\n', 'guido\n']
    >>> s2 = ['python\n', 'eggy\n', 'hamster\n', 'guido\n']
    >>> for line in unified_diff(s1, s2, fromfile='before.py', tofile='after.py'):
    ...     sys.stdout.write(line)
    --- before.py
    +++ after.py
    @@ -1,4 +1,4 @@
    -bacon
    -eggs
    -ham
    +python
    +eggy
    +hamster
     guido

或：

    >>> diff = ndiff('one\ntwo\nthree\n'.splitlines(1),
    ...              'ore\ntree\nemu\n'.splitlines(1))
    >>> print(''.join(diff), end="")
    - one
    ?  ^
    + ore
    ?  ^
    - two
    - three
    ?  -
    + tree
    + emu

## 用`textwrap`规范段落

用这个工具规范代码很倒是很合适。它常用来调整缩进或截断行。

缩进：

    >>> s = 'hello\n\n \nworld'
    >>> textwrap.indent(s, '    ')
    '    hello\n\n \n    world'

或：

    >>> s = """this
    ... is
    ... comment"""
    >>> print(textwrap.indent(s, '# ', lambda line: True))
    # this
    # is
    # comment

截断行：

    >>> s = 'five is enough'
    >>> print(textwrap.fill(s, 5))
    five
    is en
    ough
    >>> textwrap.wrap(s, 5)
    ['five', 'is en', 'ough']

## `calendar`输出日历

输出的格式非常完美。

其中的函数都会补全周，即除了本月的日子，还包括上个月的最后几天和下个月的头几天，用于补全这个月的第一周和最后一周，很多函数提供参数`firstweekday`用于定义“周”的结构。

这种输出真是让人眼前一亮：

    >>> import calendar
    >>> calendar.TextCalendar().prmonth(2014, 4)
         April 2014
    Mo Tu We Th Fr Sa Su
        1  2  3  4  5  6
     7  8  9 10 11 12 13
    14 15 16 17 18 19 20
    21 22 23 24 25 26 27
    28 29 30

## `collections.abc`测试容器支持的功能

    size = None
    if isinstance(myvar, collections.abc.Sized):
        size = len(myvar)

## 利用`heapq`排序

    >>> def heapqsort(iterable):
    ...     h = []
    ...     for value in iterable:
    ...         heapq.heappush(h, value)
    ...     return [heapq.heappop(h) for i in range(len(h))]
    >>> l = [random.randrange(1, 10) for x in range(10)]
    >>> heapqsort(l)
    [3, 4, 6, 6, 7, 7, 8, 8, 9, 9]

## 内建函数`any()`和`all()`的应用场景：

在enum模块说明中出现的代码，`any()`函数的典型场景，没有什么比Python自己的document更经典的了：

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

## 用`zip()`对iterable对象元素进行分组

来自[30 Python Language Features and Tricks You May Not Know About](http://sahandsaba.com/thirty-python-language-features-and-tricks-you-may-not-know.html)的方法

`[iter(a)] * k`返回`[<list_iterator object at 0x106593898>, <list_iterator object at 0x106593898>]`，这样使用`zip(*iter_list)`就可以对同一个迭代器进行“分别”遍历，从而达到分组的效果。

    >>> a = [1, 2, 3, 4, 5, 6]
    >>> zip(*([iter(a)] * 2))
    [(1, 2), (3, 4), (5, 6)]
     
    >>> group_adjacent = lambda a, k: zip(*([iter(a)] * k))
    >>> group_adjacent(a, 3)
    [(1, 2, 3), (4, 5, 6)]
    >>> group_adjacent(a, 2)
    [(1, 2), (3, 4), (5, 6)]
    >>> group_adjacent(a, 1)
    [(1,), (2,), (3,), (4,), (5,), (6,)]

另一种方法是使用`slice`：
   
    >>> zip(a[::2], a[1::2])
    [(1, 2), (3, 4), (5, 6)]
    
    >>> zip(a[::3], a[1::3], a[2::3])
    [(1, 2, 3), (4, 5, 6)]
    
    >>> group_adjacent = lambda a, k: zip(*(a[i::k] for i in range(k)))
    >>> group_adjacent(a, 3)
    [(1, 2, 3), (4, 5, 6)]
    >>> group_adjacent(a, 2)
    [(1, 2), (3, 4), (5, 6)]
    >>> group_adjacent(a, 1)
    [(1,), (2,), (3,), (4,), (5,), (6,)]

或者使用`islice`：

    >>> from itertools import islice
    >>> def n_grams(a, n):
    ...     z = (islice(a, i, None) for i in range(n))
    ...     return zip(*z)
    ...
    >>> a = [1, 2, 3, 4, 5, 6]
    >>> n_grams(a, 3)
    [(1, 2, 3), (2, 3, 4), (3, 4, 5), (4, 5, 6)]
    >>> n_grams(a, 2)
    [(1, 2), (2, 3), (3, 4), (4, 5), (5, 6)]
    >>> n_grams(a, 4)
    [(1, 2, 3, 4), (2, 3, 4, 5), (3, 4, 5, 6)]

## 集合运算

使用内建类型`set`进行运算：

* `a <= b`测试a是否为b的子集，与`>`结果相反。
* `a < b`测试a是否为b的真子集，与`>=`结果相反
* `a | b`求a, b的并集
* `a & b`求a, b的交集
* `a - b`求在a中而不在b中的元素集合
* `a ^ b`求既不在a中也不在b中的元素集合

```
>>> A = {1, 2, 3, 3}
>>> A
set([1, 2, 3])
>>> B = {3, 4, 5, 6, 7}
>>> B
set([3, 4, 5, 6, 7])
>>> A | B
set([1, 2, 3, 4, 5, 6, 7])
>>> A & B
set([3])
>>> A - B
set([1, 2])
>>> B - A
set([4, 5, 6, 7])
>>> A ^ B
set([1, 2, 4, 5, 6, 7])
>>> (A ^ B) == ((A - B) | (B - A))
True
```

或者使用`collections.Counter`进行运算：

    >>> A = collections.Counter([1, 2, 2])
    >>> B = collections.Counter([2, 2, 3])
    >>> A
    Counter({2: 2, 1: 1})
    >>> B
    Counter({2: 2, 3: 1})
    >>> A | B
    Counter({2: 2, 1: 1, 3: 1})
    >>> A & B
    Counter({2: 2})
    >>> A + B
    Counter({2: 4, 1: 1, 3: 1})
    >>> A - B
    Counter({1: 1})
    >>> B - A
    Counter({3: 1})

## Python初学者容易犯的错误

[Top 10 Mistakes that Python Programmers Make](http://www.toptal.com/python/top-10-mistakes-that-python-programmers-make)

### 函数默认值的初始化问题

Python中，函数也是一个对象（Callable types），函数的默认值初始化类似class对实例的初始化（就是调用类的`__init__()`），在函数体定义时只会被eval**一次**。

问题出现在，当我使用类似mutable的对象作为默认值时，该对象会随着函数体的定义被初始化仅一次，于是有了：

```
>>> def foo(bar=[]):        # bar is optional and defaults to [] if not specified
...     bar.append("baz")   # but this line could be problematic, as we'll see...
...     return '%s, %i' % (bar, id(bar))
... 
>>> foo()
"['baz'], 4318961736"
>>> foo()
"['baz', 'baz'], 4318961736"
>>> foo()
"['baz', 'baz', 'baz'], 4318961736"
```

可见，每次调用`foo()`时，变量*bar*使用的是同一个list对象，而我们通常希望的是每次调用`foo()`都会得到一个新的*bar*。所以，通常应该这样实现：

```
>>> def foo(bar=None):
...    if bar is None:		# or if not bar:
...        bar = []
...    bar.append("baz")
...    return bar
...
>>> foo()
["baz"]
>>> foo()
["baz"]
>>> foo()
["baz"]
```

关于函数的详情，参见[Calls](https://docs.python.org/3.4/reference/expressions.html#calls), [Function definitions](https://docs.python.org/3.4/reference/compound_stmts.html#function), [The standard type hierarchy](https://docs.python.org/3.4/reference/datamodel.html#the-standard-type-hierarchy)中的**Callable types**。

### 类属性查找问题

Python的属性查找机制复杂而精巧，解释器默默的在后台做了很多工作，保证了我们这些小白每次调用`o.x`时都能返回一个其恰当的值。

比如：

```
>>> class A(object):
...     x = 1
...
>>> class B(A):
...     pass
...
>>> class C(A):
...     pass
...
>>> print(A.x, B.x, C.x)
1 1 1
```

这是我们想要的结果。

如果：

```
>>> B.x = 2
>>> print(A.x, B.x, C.x)
1 2 1
```

也没错，和预想的一样。

接下来再：

```
>>> A.x = 3
>>> print(A.x, B.x, C.x)
3 2 3
```

不对了，在修改`A.x`时`C.x`也变了。

其实，当第一次print的时候：

```
>>> print(id(A.x), id(B.x), id(C.x))
4304067520 4304067520 4304067520
```

在对B.x赋值之后，变成了：

```
>>> print(id(A.x), id(B.x), id(C.x))
4304067520 4304067552 4304067520
```

接下来对A.x赋值之后：

```
>>> print(id(A.x), id(B.x), id(C.x))
4304067584 4304067552 4304067584
```

第一次`print`时，只有`A.x`是存在的；在对`B.x`进行赋值后，`B.x`独立于父类的`x`属性存在了；而我们修改`A.x`连带了`C.x`一起变动，其实是因为类型`C`没有自己的`x`属性，于是回溯到父类的`A.x`属性并返回。

关于Python属性查找的详情，参见：[Customizing attribute access](https://docs.python.org/3.4/reference/datamodel.html#customizing-attribute-access), 
[Descriptor HowTo Guide](https://docs.python.org/3.4/howto/descriptor.html), 
[Special method lookup](https://docs.python.org/3.4/reference/datamodel.html#special-lookup), 
[class.mro](https://docs.python.org/3.4/library/stdtypes.html#class.mro), 
[Method Resolution Order](http://python-history.blogspot.com.ar/2010/06/method-resolution-order.html), 

### except子句写法错误

因为是Python新手，不经常使用Python2，所以这里只是提及：

```
>>> try:
...     l = ["a", "b"]
...     int(l[2])
... except ValueError, IndexError:  # To catch both exceptions, right?
...     pass
...
Traceback (most recent call last):
  File "<stdin>", line 3, in <module>
IndexError: list index out of range
```

捕捉不起作用是因为Python2中的语法：`except Exception, e`的意思是将*Exception*绑定到对象*e*上，于是上面的代码就是把`ValueError`绑定到了一个名为"IndexError"的对象上了。

正确的写法应该是：

```
>>> try:
...     l = ["a", "b"]
...     int(l[2])
... except (ValueError, IndexError) as e:  
...     pass
...
>>>
```

其实，第一种写法在Python3中是不允许的，会直接提示

```
  File "<input>", line 4
    except IndexError, ValueError:
                     ^
SyntaxError: invalid syntax
```

### 变量作用域问题

Python变量作用域解析是遵循[LEGB](https://blog.mozilla.org/webdev/2011/01/31/python-scoping-understanding-legb/)规则，即**L**ocal, **E**nclosing, **G**lobal, **B**uilt-in，但是：

```
>>> x = 10
>>> def foo():
...     x += 1
...     print(x)
...
>>> foo()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 2, in foo
UnboundLocalError: local variable 'x' referenced before assignment
```

注意这是一个`UnboundLocalError`，局部变量在未定义时引用。当给作用域中的变量做赋值动作时，该变量会被当做这个作用域的局部变量，从而屏蔽掉外部作用域中的同名变量。见[Why am I getting an UnboundLocalError when the variable has a value?](https://docs.python.org/3.4/faq/programming.html#why-am-i-getting-an-unboundlocalerror-when-the-variable-has-a-value)

当我们使用list做局部变量时可以更明显的看到赋值语句对作用域中变量的操作：

```
>>> lst = [1, 2, 3]
>>> def foo1():
...     lst.append(5)   # This works ok...
...
>>> foo1()
>>> lst
[1, 2, 3, 5]

>>> lst = [1, 2, 3]
>>> def foo2():
...     lst += [5]      # ... but this bombs!
...
>>> foo2()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 2, in foo
UnboundLocalError: local variable 'lst' referenced before assignment
```

foo1中调用列表的添加方法，没有出错；

foo2中的赋值语句`lst += [5]`（语句`lst = lst + [5]`的缩写）就会报错。

### 迭代进行时修改列表

这个错误是经典中的经典：

```
>>> odd = lambda x : bool(x % 2)
>>> numbers = [n for n in range(10)]
>>> for i in range(len(numbers)):
...     if odd(numbers[i]):
...         del numbers[i]  # BAD: Deleting item from a list while iterating over it
...
Traceback (most recent call last):
  	  File "<stdin>", line 2, in <module>
IndexError: list index out of range
```

每个小白程序员可能出过这种越界的错误，就是在迭代进行时修改列表。

如果要得到上面这段代码期望的结果，可以使用列表推导式（[List Comprehensions](https://docs.python.org/3.4/tutorial/datastructures.html#tut-listcomps)）：

```
>>> odd = lambda x : bool(x % 2)
>>> numbers = [n for n in range(10)]
>>> numbers[:] = [n for n in numbers if not odd(n)]  # ahh, the beauty of it all
>>> numbers
[0, 2, 4, 6, 8]
```

或者更函数式的方法，利用惰性求值在迭代的过程中剔除不符合要求的项：

```
>>> list(filter(lambda x: not (x % 2), range(10)))
[0, 2, 4, 6, 8]
```

### 闭包变量值的延迟绑定问题

闭包，经常出现在函数式编程中（其实，在使用Python的decorator、lambda是经常会出现闭包），只有在被调用时才执行相应的函数块（包括函数体和其所需的外部变量），即前面提到的惰性求值。

闭包中的外部变量会有延迟绑定行为，即调用时才进行赋值。如果这样写：

```
>>> def create_multipliers():
...     return [lambda x: i * x for i in range(5)]
...
>>> for multiplier in create_multipliers():
...     print multiplier(2)
...
```

上面的代码返回了5个8，为什么不是0, 2, 4, 6, 8呢？

因为变量*i*是lambda的闭包变量，即free variables（类似于环境中的变量，是独立于lambda之外，而x是在lambda内的参数），它是在调用时被赋值的，而调用时列表推导式已经结束，*i*停留在了4这个结果值上。

可以使用函数的`__closure__`属性看到闭包变量：

```
>>> def create_multipliers():
...     return [lambda x: i * x for i in range(5)]
... 
>>> funcs = create_multipliers()
>>> pprint.pprint(funcs)
[<function create_multipliers.<locals>.<listcomp>.<lambda> at 0x10f7e88c8>,
 <function create_multipliers.<locals>.<listcomp>.<lambda> at 0x10f7e8950>,
 <function create_multipliers.<locals>.<listcomp>.<lambda> at 0x10f7e89d8>,
 <function create_multipliers.<locals>.<listcomp>.<lambda> at 0x10f7e8a60>,
 <function create_multipliers.<locals>.<listcomp>.<lambda> at 0x10f7e8ae8>]
>>> for multiplier in funcs:
...     print(multiplier(2), multiplier.__closure__, multiplier.__closure__[0].cell_contents)
... 
8 (<cell at 0x10f5dce28: int object at 0x10f3c4c20>,) 4
8 (<cell at 0x10f5dce28: int object at 0x10f3c4c20>,) 4
8 (<cell at 0x10f5dce28: int object at 0x10f3c4c20>,) 4
8 (<cell at 0x10f5dce28: int object at 0x10f3c4c20>,) 4
8 (<cell at 0x10f5dce28: int object at 0x10f3c4c20>,) 4
```

注意到`__closure__`属性返回的int值指向同一个地址，它们的值都是4，这是调用时进行的赋值。

可以用一种技巧避免闭包变量的延迟绑定行为，即把lambda需要使用的闭包变量变为lambda的内部参数：

```
>>> def create_multipliers():
...     return [lambda x, i=i : i * x for i in range(5)]
...
>>> for multiplier in create_multipliers():
...     print multiplier(2)
...
```

这样就可以得到我们想要的0, 2, 4, 6, 8了。

看更多细节：

```
>>> def create_multipliers():
...     return [lambda x, i=i: i * x for i in range(5)]
... 
>>> ifunc = create_multipliers()
>>> pprint.pprint(ifunc)
[<function create_multipliers.<locals>.<listcomp>.<lambda> at 0x10dcf3840>,
 <function create_multipliers.<locals>.<listcomp>.<lambda> at 0x10dcf38c8>,
 <function create_multipliers.<locals>.<listcomp>.<lambda> at 0x10dcf3950>,
 <function create_multipliers.<locals>.<listcomp>.<lambda> at 0x10dcf39d8>,
 <function create_multipliers.<locals>.<listcomp>.<lambda> at 0x10dcf3a60>]
>>> for multiplier in ifunc:
...     print(multiplier(2), multiplier.__closure__)
... 
0 None
2 None
4 None
6 None
8 None
```
这个lambda函数没有闭包变量了。

### 模块的循环引用问题

做两个模块，`a.py`：

```
import b

def f():
    return b.x
	
print f()
```

和`b.py`：

```
import a

x = 1

def g():
    print a.f()
```

如果：

```
>>> import a
1
```

不会出现异常，因为解释器对同一个模块仅导入一次，所以这里看起来是循环引用了，但因为`b.py`没有函数调用，所以不会出现问题。

但是：

```
>>> import b
Traceback (most recent call last):
  	  File "<stdin>", line 1, in <module>
  	  File "b.py", line 1, in <module>
    import a
  	  File "a.py", line 6, in <module>
	print f()
  	  File "a.py", line 4, in f
	return b.x
AttributeError: 'module' object has no attribute 'x'
```

虽然是同样的代码，但是这里`import`的问题在于，`b.py`中导入`a.py`时`a.py`不会再导入`b.py`，而`a.py`产生了函数调用需要使用到`b.py`，此时`b.py`并没有被导入，所以出现了属性查找失败。

其实，只需将`b.py`的import语句放在函数内就可以了：

```
x = 1

def g():
    import a	# This will be evaluated only when g() is called
    print a.f()
```

这样，只有在函数被调用时才会导入`a.py`，于是：

```
>>> import b
>>> b.g()
1	# Printed a first time since module 'a' calls 'print f()' at the end
1	# Printed a second time, this one is our call to 'g'
```

### 自定义模块与Python标准库名冲突问题

把Python帮助文档放在枕头边，然后，不要这么干！

### Python2与Python3之间的差异问题

类似这种代码，`foo.py`：

```
import sys

def bar(i):
    if i == 1:
        raise KeyError(1)
    if i == 2:
        raise ValueError(2)

def bad():
    e = None
    try:
        bar(int(sys.argv[1]))
    except KeyError as e:
        print('key error')
    except ValueError as e:
        print('value error')
    print(e)

bad()
```
在Python2下的结果是：

```
$ python foo.py 1
key error
1
$ python foo.py 2
value error
2
```
但是Python3中会报错：

```
$ python3 foo.py 1
key error
Traceback (most recent call last):
  File "foo.py", line 19, in <module>
    bad()
  File "foo.py", line 17, in bad
    print(e)
UnboundLocalError: local variable 'e' referenced before assignment
```

问题在于Python3中，异常绑定的对象只在except代码块内部有效，参见[The try statement](https://docs.python.org/3/reference/compound_stmts.html#the-try-statement)。

于是，我们可以把异常赋给一个`except`语句外的变量，这样在`except`以外的地方调用就不会出现未定义的异常了。

```
import sys

def bar(i):
    if i == 1:
        raise KeyError(1)
    if i == 2:
        raise ValueError(2)

def good():
    exception = None
    try:
        bar(int(sys.argv[1]))
    except KeyError as e:
        exception = e
        print('key error')
    except ValueError as e:
        exception = e
        print('value error')
    print(exception)

good()
```

Python3中的结果：

```
$ python3 foo.py 1
key error
1
$ python3 foo.py 2
value error
2
```

在[Python Hiring Guide](http://www.toptal.com/python#hiring-guide)中讨论了将Python2的代码迁移到Python3的一些需要注意的地方。

### 误用`__del__`方法时的问题

做两个模块，`mod.py`：

```
import foo

class Bar(object):
   	    ...
    def __del__(self):
        foo.cleanup(self.myhandle)
```

和`another_mod.py`：

```
import mod
mybar = mod.Bar()
```

这里会得到一个`AttributeError`。

参见[bug list中的另一个AttributeError](https://mail.python.org/pipermail/python-bugs-list/2009-January/069209.html): 

编译器退出时，模块的全局变量会在模块被回收前就被置为`None`。`__del__()`方法可能在这种不可控的情况下被调用。所以，代码不应该过度依靠模块全局变量。

在这个例子中，当`__del__()`被调用时，`foo`已经被置为`None`了。

这里，我们可以使用`atexit.register()`，当代码执行完毕（正常退出）时，我们注册的析构代码会在解释器退出前执行。像这样：

```
import foo
import atexit

def cleanup(handle):
    foo.cleanup(handle)


class Bar(object):
    def __init__(self):
        ...
        atexit.register(cleanup, self.myhandle)
```

这是一种在代码正常退出时的析构方法，简洁可靠。

## sqlite3的内存数据库

在`connect()`函数中，如果*database*参数设置为字符串`:memory:`，模块就会在内存中建立一个数据库：

```
with sqlite3.connect(":memory:") as conn:
    pass
```
