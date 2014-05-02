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

用这个小工具调整代码格式很方便。常用来调整缩进或截断行。

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

这个工具是又好用又逗，输出的格式非常完美。

其中的函数都会补全周，即除了本月的日子，还包括上个月的最后几天和下个月的头几天，用于补全这个月的第一周和最后一周，很多函数提供参数`firstweekday`用于定义“周”的结构。

这种输出真是让人开眼界^_^：

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

## 内建函数`any()`和`all()`的用法：

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

