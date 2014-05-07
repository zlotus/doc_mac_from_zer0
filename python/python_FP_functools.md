# 函数式编程支持 - `functools`模块

## 概况

这个模块主要提供了高阶函数（操作或返回函数的函数）支持。一般的，使用这个模块中的函数时，要记住“任何可调用对象理论上都可以当做函数看待”的事实。

## 提供的函数

### functools.cmp_to_key(func)

这个函数是为了继续支持曾经在Python2中使用的比较函数，它的行为就像名字起的那样，把cmp函数转为key函数给诸如`sorted()`, `min()`, `max()`, `heapq.nlargest()`, `heapq.nsmallest()`, `itertools.groupby()`这些需要用到key的函数使用。

这个cmp函数必须是一个接受两个参数的可调用对象，函数应该比较这两个参数，而后返回：负值表示小于、零表示等于、正值代表大于。

另，key函数应该是一个只接受一个参数的可调用对象，返回一个值用来表示该参数在序列中应有的位置。

`cmp_to_key()`一般这样使用：

    sorted(iterable, key=cmp_to_key(locale.strcoll))  # locale-aware sort order

功能和代码都很简练：

    def cmp_to_key(mycmp):
        """Convert a cmp= function into a key= function"""
        class K(object):
            __slots__ = ['obj']
            def __init__(self, obj):
                self.obj = obj
            def __lt__(self, other):
                return mycmp(self.obj, other.obj) < 0
            def __gt__(self, other):
                return mycmp(self.obj, other.obj) > 0
            def __eq__(self, other):
                return mycmp(self.obj, other.obj) == 0
            def __le__(self, other):
                return mycmp(self.obj, other.obj) <= 0
            def __ge__(self, other):
                return mycmp(self.obj, other.obj) >= 0
            def __ne__(self, other):
                return mycmp(self.obj, other.obj) != 0
            __hash__ = None
        return K


### @functools.lru_cache(maxsize=128, typed=False)

这个函数修饰器可以称得上“大名鼎鼎”了，在[Python修饰器的函数式编程](http://coolshell.cn/articles/11265.html)中也提到过，它是用来缓存别的函数的运行结果的。

当遇到需要经常用同样的参数调用时间代价高或是I/O绑定的函数时，这个修饰器会非常有用。

*maxsize*限制了修饰器可以缓存最多多少个不同的调用，设置为`None`时可以取消这个上限，推荐的设置是2的幂级数。

*type*设置为`True`时，若两次调用的参数分别为两个数据类型，修饰器会区别存储这次调用，比如`f(3)`和`f(3.0)`在`type=True`时会被存储两次。

修饰器还有一个`cache_info()`函数用来查看缓存命中率和缓存当前存储状态。不过在多线程环境下，这个命中率为近似值。

`cache_clear()`方法用于清理缓存。

    >>> @lru_cache(maxsize=32)
    ... def get_pep(num):
    ...     'Retrieve text of a Python Enhancement Proposal'
    ...     resource = 'http://www.python.org/dev/peps/pep-%04d/' % num
    ...     try:
    ...         with urllib.request.urlopen(resource) as s:
    ...             return s.read()
    ...     except urllib.error.HTTPError:
    ...         return 'Not Found'
    ... 
    >>> for n in 8, 290, 308, 320, 8, 218, 320, 279, 289, 320, 9991:
    ...     pep = get_pep(n)
    ...     print(n, len(pep))
    >>> get_pep.cache_info()
    CacheInfo(hits=3, misses=8, maxsize=32, currsize=8)

这就是最著名的用例，计算Fibonacci数列时使用`lru_cache()`实现动态规划：

    >>> @lru_cache(maxsize=None)
    ... def fib(n):
    ...     if n < 2:
    ...         return n
    ...     return fib(n-1) + fib(n-2)
    ... 
    >>> [fib(n) for n in range(16)]
    [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610]
    >>> fib.cache_info()
    CacheInfo(hits=28, misses=16, maxsize=None, currsize=16)

### @functools.total_ordering

一个讨巧的类修饰器，如果一个类定义了至少一个富比较排序方法（包括`__lt__()`, `__le__()`, `__gt__()`, `__ge__()`），这个修饰器会帮我们实现剩下的。另外，被修饰类应当实现`__eq__()`方法。

不过，虽然这个方法会让我们方便准确的实现所有的排序方法，但是它的执行效率是低下的，而且复杂的调用会使得调试变的困难。如果需要更高的比较效率，还是推荐手写全部的六个富比较方法。

    def _not_op(op, other):
        # "not a < b" handles "a >= b"
        # "not a <= b" handles "a > b"
        # "not a >= b" handles "a < b"
        # "not a > b" handles "a <= b"
        op_result = op(other)
        if op_result is NotImplemented:
            return NotImplemented
        return not op_result
    
    def _op_or_eq(op, self, other):
        # "a < b or a == b" handles "a <= b"
        # "a > b or a == b" handles "a >= b"
        op_result = op(other)
        if op_result is NotImplemented:
            return NotImplemented
        return op_result or self == other
    
    def _not_op_and_not_eq(op, self, other):
        # "not (a < b or a == b)" handles "a > b"
        # "not a < b and a != b" is equivalent
        # "not (a > b or a == b)" handles "a < b"
        # "not a > b and a != b" is equivalent
        op_result = op(other)
        if op_result is NotImplemented:
            return NotImplemented
        return not op_result and self != other
    
    def _not_op_or_eq(op, self, other):
        # "not a <= b or a == b" handles "a >= b"
        # "not a >= b or a == b" handles "a <= b"
        op_result = op(other)
        if op_result is NotImplemented:
            return NotImplemented
        return not op_result or self == other
    
    def _op_and_not_eq(op, self, other):
        # "a <= b and not a == b" handles "a < b"
        # "a >= b and not a == b" handles "a > b"
        op_result = op(other)
        if op_result is NotImplemented:
            return NotImplemented
        return op_result and self != other
    
    def total_ordering(cls):
        """Class decorator that fills in missing ordering methods"""
        convert = {
            '__lt__': [('__gt__', lambda self, other: _not_op_and_not_eq(self.__lt__, self, other)),
                       ('__le__', lambda self, other: _op_or_eq(self.__lt__, self, other)),
                       ('__ge__', lambda self, other: _not_op(self.__lt__, other))],
            '__le__': [('__ge__', lambda self, other: _not_op_or_eq(self.__le__, self, other)),
                       ('__lt__', lambda self, other: _op_and_not_eq(self.__le__, self, other)),
                       ('__gt__', lambda self, other: _not_op(self.__le__, other))],
            '__gt__': [('__lt__', lambda self, other: _not_op_and_not_eq(self.__gt__, self, other)),
                       ('__ge__', lambda self, other: _op_or_eq(self.__gt__, self, other)),
                       ('__le__', lambda self, other: _not_op(self.__gt__, other))],
            '__ge__': [('__le__', lambda self, other: _not_op_or_eq(self.__ge__, self, other)),
                       ('__gt__', lambda self, other: _op_and_not_eq(self.__ge__, self, other)),
                       ('__lt__', lambda self, other: _not_op(self.__ge__, other))]
        }
        # Find user-defined comparisons (not those inherited from object).
        roots = [op for op in convert if getattr(cls, op, None) is not getattr(object, op, None)]
        if not roots:
            raise ValueError('must define at least one ordering operation: < > <= >=')
        root = max(roots)       # prefer __lt__ to __le__ to __gt__ to __ge__
        for opname, opfunc in convert[root]:
            if opname not in roots:
                opfunc.__name__ = opname
                opfunc.__doc__ = getattr(int, opname).__doc__
                setattr(cls, opname, opfunc)
        return cls

这就是它效率低，调试复杂的原因。但是在简单的比较中，这个修饰器可以节省不少程序员时间。

### functools.partial(func, *args, **keywords)

这个函数体现了函数式编程的几个典型思想之一：currying，也叫partial application。它可以把一个拥有多个参数的函数拆分成想要的几部分，递进调用。

它返回一个`partial()`对象，但行为与函数无异。

    >>> from functools import partial
    >>> basetwo = partial(int, base=2)
    >>> basetwo.__doc__ = 'Convert base 2 string to an int.'
    >>> basetwo('10010')
    18

在这个例子中，`int()`函数的*base*参数被“冻结”为2后赋给变量`basetwo`，于是`basetwo('10010')`就等价于`int('10010', base=2)`

这个技术可以在[函数式编程](http://coolshell.cn/articles/10822.html)中找到详细解释。

### class functools.partialmethod(func, *args, **keywords)

Python3.4新添加。

调用这个类返回一个partialmethod的descriptor，行为和上面的`partial()`类似，主要作用也是为了分治一个函数的各个部分，与上面的`partial()`不同的是，`partialmethod()`被设计用做定义类方法。

*func*必须是一个descriptor或可调用对象。

当*func*是descriptor时（可能是一个classmethod(), staticmethod(), abstractmethod(), 或是partialmethod的一个实例），所有`__get__()`调用会委托给相应的descriptor，从而返回一个恰当的`partial`对象。

当*func*不是descriptor时，一个相应的绑定方法会被动态创建。这个过程类似一个函数被实例调用时变成了绑定方法的过程：一个`self`参数被强行插入在第一个参数的位置。

    >>> class Cell(object):
    ...     @property
    ...     def alive(self):
    ...         return self._alive
    ...     def set_state(self, state):
    ...         self._alive = bool(state)
    ...     set_alive = partialmethod(set_state, True)
    ...     set_dead = partialmethod(set_state, False)
    ...
    >>> c = Cell()
    >>> c.alive
    False
    >>> c.set_alive()
    >>> c.alive
    True

### functools.reduce(function, iterable[, initializer])

函数的大概行为：从*iterable*中取出两个值作为两个参数传给*function*算出结果，再从*iterable*中取下一个值并连同上一步的结果作为两个参数传给*function*算出结果，再从*iterable*中取值...直到*iterable*取完。

所以，`reduce(lambda x, y: x+y, [1, 2, 3, 4, 5])`的计算过程应该是`((((1+2)+3)+4)+5)`。

*initalizer*指定计算的初始值，若为`None`则初始参数为*iterable*的第一个元素。

### @functools.singledispatch(default)

这个decorator可以把一个函数变为单分派泛型函数，这个句子里的术语很多，看看例子吧。

    >>> from functools import singledispatch
    >>> @singledispatch
    ... def fun(arg, verbose=False):
    ...     if verbose:
    ...         print("Let me just say,", end=" ")
    ...     print(arg)
    ...

要注意的是，decorator只对第一个参数有效，也就这里的*arg*。现在，可以用`register()`为这个函数添加类型重载了：

    >>> @fun.register(int)
    ... def _(arg, verbose=False):
    ...     if verbose:
    ...         print("Strength in numbers, eh?", end=" ")
    ...     print(arg)
    ...
    >>> @fun.register(list)
    ... def _(arg, verbose=False):
    ...     if verbose:
    ...         print("Enumerate this:")
    ...     for i, elem in enumerate(arg):
    ...         print(i, elem)
    ...

注意，这个`register()`也是个decorator，需要一个类型作为参数，被修饰函数应该实现对这个类型的相应操作。

当然，decorator也可以直接写成函数：

    >>> def nothing(arg, verbose=False):
    ...     print("Nothing.")
    ...
    >>> fun.register(type(None), nothing)

`register()`返回未被修饰的函数对象，所以可以很容易的做调试、持久化、单元测试：

    >>> @fun.register(float)
    ... @fun.register(Decimal)
    ... def fun_num(arg, verbose=False):
    ...     if verbose:
    ...         print("Half of your number:", end=" ")
    ...     print(arg / 2)
    ...
    >>> fun_num is fun
    False

上面我们定义了对`int`, `list`, `None`, `float`, `Decimal`类型的重载，现在调用它们：

    >>> fun("Hello, world.")
    Hello, world.
    >>> fun("test.", verbose=True)
    Let me just say, test.
    >>> fun(42, verbose=True)
    Strength in numbers, eh? 42
    >>> fun(['spam', 'spam', 'eggs', 'spam'], verbose=True)
    Enumerate this:
    0 spam
    1 spam
    2 eggs
    3 spam
    >>> fun(None)
    Nothing.
    >>> fun(1.23)
    0.615

另一个需要注意的是，被`@singledispatch`修饰的函数，即这里的`fun()`，注册时使用的是`object`类型，即类型查找失败时会调用原函数。

检查泛型函数在处理不同类型时到底调用的是哪个函数对象，可以使用decorator提供的`dispatch()`：

    >>> fun.dispatch(float)
    <function fun_num at 0x1035a2840>
    >>> fun.dispatch(dict)    # note: default implementation
    <function fun at 0x103fe0000>

这里的`dict`就是会导致查找失败的类型，于是返回了原函数对象。

检查泛型函数的所有支持类型，使用decorator提供的`registey`属性，这是一个字典，所以可以使用`key()`来查看：

    >>> fun.registry.keys()
    dict_keys([<class 'NoneType'>, <class 'int'>, <class 'object'>,
              <class 'decimal.Decimal'>, <class 'list'>,
              <class 'float'>])
    >>> fun.registry[float]
    <function fun_num at 0x1035a2840>
    >>> fun.registry[object]
    <function fun at 0x103fe0000>

### functools.update_wrapper(wrapper, wrapped, assigned=WRAPPER_ASSIGNMENTS, updated=WRAPPER_UPDATES)

看名字就知道，这是一个负责包装的函数，默认情况下它可以把被封装函数的`__name__`, `__module__`,`__qualname__`, `__doc__`, `__annotations__`, `__dict__`都复制到封装函数中，使得到的封装函数更像原来的被封装函数，具体参见下面的例子。

先看看源码，很简单：

    WRAPPER_ASSIGNMENTS = ('__module__', '__name__', '__qualname__', '__doc__',
                           '__annotations__')
    WRAPPER_UPDATES = ('__dict__',)
    def update_wrapper(wrapper,
                       wrapped,
                       assigned = WRAPPER_ASSIGNMENTS,
                       updated = WRAPPER_UPDATES):
        for attr in assigned:
            try:
                value = getattr(wrapped, attr)
            except AttributeError:
                pass
            else:
                setattr(wrapper, attr, value)
        for attr in updated:
            getattr(wrapper, attr).update(getattr(wrapped, attr, {}))
        wrapper.__wrapped__ = wrapped
        return wrapper

*assigned*指定将被封装函数中的哪些属性直接复制到封装函数中，默认复制`WRAPPER_ASSIGNMENTS`指定的属性。

*updated*指定将被封装函数中的哪些属性更新到封装函数中，默认更新`WRAPPER_UPDATES`指定的属性。

封装函数默认提供`__wrapped__`属性，用来直接访问被封装函数。

示例，不使用update_wrapper时，封装函数仅仅实现功能，并不具备被封装函数的属性：

    >>> def wrapper(func):
    ...     def callable(*args, **kwargs):
    ...         return func(*args, **kwargs) + 'wrapper. '
    ...     return callable
    ... 
    >>> @wrapper
    ... def wrapped():
    ...     '''doc of wrapped function'''
    ...     return 'wrapped. '
    ... 
    >>> wrapped()
    'wrapped. wrapper. '
    >>> wrapped.__doc__
    >>>

使用update_wrapper时，被封装函数的属性也复制给了封装函数：

    >>> from functools import update_wrapper
    >>> def wrapper(func):
    ...     def callable(*args, **kwargs):
    ...         return func(*args, **kwargs) + 'wrapper. '
    ...     return update_wrapper(callable, func)
    ... 
    >>> @wrapper
    ... def wrapped():
    ...     '''doc of wrapped function'''
    ...     return 'wrapped. '
    ... 
    >>> wrapped()
    'wrapped. wrapper. '
    >>> wrapped.__doc__
    'doc of wrapped function'

### @functools.wraps(wrapped, assigned=WRAPPER_ASSIGNMENTS, updated=WRAPPER_UPDATES)

这个decorator的目的是为了更方便的调用`update_wrapper()`：

有`wrapper = partial(update_wrapper, wrapped=func, assigned=assigned, updated=updated)(wrapper)`；

即`wrapper = update_wrapper(wrapper=wrapper, wrapped=func, assigned=assigned, updated=updated)`。

所以上面的`update_wrapper()`时的代码可以改成：

    >>> from functools import wraps
    >>> def wrapper(func):
    ...     @wraps(func)
    ...     def callable(*args, **kwargs):
    ...         return func(*args, **kwargs) + 'wrapper. '
    ...     return callable
    ... 
    >>> @wrapper
    ... def wrapped():
    ...     '''doc of wrapped function'''
    ...     return 'wrapped. '
    ... 
    >>> wrapped()
    'wrapped. wrapper. '
    >>> wrapped.__doc__
    'doc of wrapped function'

更加清楚明了。

