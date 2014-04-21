# 我不知道的Python

最近按照知道创宇的技能表加天赋，开始了大坑无数的阅读Python document的路程。

缓慢填坑。

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

