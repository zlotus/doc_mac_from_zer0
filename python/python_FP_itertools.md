# 函数式编程支持 - `itertools`模块

## 概况

这个模块主要包含了使用Python重写的来自APL、Haskell、SML等语言的类似迭代器，速度快而且节省内存。

## 无限迭代器

### itertools.count(start=0, step=1) --> start, start+step, start+2*step, ...

返回一个无限迭代器（不会raise StopIteration），结果就是一个首项为*start*公差为*step*的等差数列。

    # count(10) --> 10 11 12 13 14 ...
    # count(2.5, 0.5) -> 2.5 3.0 3.5 ...

需要注意的是，如果是操作浮点数，可以使用`(start + step * i for i in count())`保证精度。

### itertools.cycle(iterable)

返回一个无限迭代器，这个函数的大概行为是：把*iterable*逐个输出的同时将每一个元素加入一个临时列表，再循环输出这个临时列表。

    # cycle('ABCD') --> A B C D A B C D A B C D ...

### itertools.repeat(object[, times])

如果不指定*times*的话，会返回无限迭代器。

    # repeat(10, 3) --> 10 10 10

典型场景，为`map()`或`zip()`持续提供参数：

    >>> list(map(pow, range(10), repeat(2)))
    [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

## 有限迭代器

### itertools.accumulate(iterable[, func])

函数的大概行为：每次的计算结果都是用上一步的结果做*func*的第一个参数，用*iterable*的下一个元素做*func*的第二个参数进行计算。
*func*默认行为是`operator.add()`，这个函数类似`functools.reduce()`，只不过`functools.reduce()`只返回最后一步的结果。

    # accumulate([1,2,3,4,5]) --> 1 3 6 10 15
    # accumulate([1,2,3,4,5], operator.mul) --> 1 2 6 24 120

典型示例：

    >>> data = [3, 4, 6, 2, 1, 9, 0, 7, 5, 8]
    >>> list(accumulate(data, operator.mul))     # running product
    [3, 12, 72, 144, 144, 1296, 0, 0, 0, 0]
    >>> list(accumulate(data, max))              # running maximum
    [3, 4, 6, 6, 6, 9, 9, 9, 9, 9]
    
    # Amortize a 5% loan of 1000 with 4 annual payments of 90
    >>> cashflows = [1000, -90, -90, -90, -90]
    >>> list(accumulate(cashflows, lambda bal, pmt: bal*1.05 + pmt))
    [1000, 960.0, 918.0, 873.9000000000001, 827.5950000000001]
    
    # Chaotic recurrence relation http://en.wikipedia.org/wiki/Logistic_map
    >>> logistic_map = lambda x, _:  r * x * (1 - x)
    >>> r = 3.8
    >>> x0 = 0.4
    >>> inputs = repeat(x0, 36)     # only the initial value is used
    >>> [format(x, '.2f') for x in accumulate(inputs, logistic_map)]
    ['0.40', '0.91', '0.30', '0.81', '0.60', '0.92', '0.29', '0.79', '0.63',
     '0.88', '0.39', '0.90', '0.33', '0.84', '0.52', '0.95', '0.18', '0.57',
     '0.93', '0.25', '0.71', '0.79', '0.63', '0.88', '0.39', '0.91', '0.32',
     '0.83', '0.54', '0.95', '0.20', '0.60', '0.91', '0.30', '0.80', '0.60']

### itertools.chain(*iterables)

函数的大概行为：从*iterables*中取第一个可迭代对象，逐个返回直到结束，然后再从*iterables*中取第二个可迭代对象...直到*iterables*中没有可迭代对象为止。

    # chain('ABC', 'DEF') --> A B C D E F
    # chain.from_iterable(['ABC', 'DEF']) --> A B C D E F

典型场景是将数个可迭代对象合起来当做一个可迭代对象返回。

### classmethod chain.from_iterable(iterable)

同上面的`chain()`一样，只是欢乐一种调用方法：

    # chain.from_iterable(['ABC', 'DEF']) --> A B C D E F

### itertools.compress(data, selectors)

函数的大概行为：使用*selectors*过滤*data*中的数据，其实就是逻辑与运算。

    # compress('ABCDEF', [1,0,1,0,1,1]) --> A C E F

### itertools.dropwhile(predicate, iterable)

函数的大概行为：从*iterable*中取值，传给*predicate*进行测试，若结果为**真**，则丢掉这个值不做返回的的取下一个值，直到首次返回**假**时，返回该值及*iterable*中剩下的元素。

    # dropwhile(lambda x: x<5, [1,4,6,4,1]) --> 6 4 1

### itertools.filterfalse(predicate, iterable)

函数的大概行为：从*iterable*中取值，传给*predicate*进行测试，若结果为**假**，则返回该值，否则丢弃这个值并继续取下一个值进行测试。

如果*predicate*为`None`，则默认对*iterable*使用`bool()`函数进行测试。

    # filterfalse(lambda x: x%2, range(10)) --> 0 2 4 6 8

### itertools.groupby(iterable, key=None)

函数的大概行为：如果指定*key*函数，则会使用这个函数为每一个*iterable*中的值计算一个键值；若不指定*key*，则会转到一个默认函数，该函数将每一个*iterable*中的值原封不动的返回作为键值。有了键值，函数会按顺序将拥有同样键值的值归在一组，在遍历的过程中，一旦键值函数的返回值发生改变，则会新建一个组，存储带有该键值的值，直到*iterable*耗尽。
整个过程类似Unix中的`uniq`过滤器，而不同于SQL中的`GROUP BY`字句，因为`GROUP BY`会将所有同样键值的元素找出来放在一起，而uniq是按顺序将同样的元素放在一起，每当`key`函数返回值发生变化，都将创建一个新的组，看下面的例子更好理解：

    # [(k, list(g)) for k, g in itertools.groupby('AAAABBBCCD')] --> [('A', ['A', 'A', 'A', 'A']), ('B', ['B', 'B', 'B']), ('C', ['C', 'C']), ('D', ['D'])]
    # [(k, list(g)) for k, g in itertools.groupby('AAAABBBCCDAABBB')] --> [('A', ['A', 'A', 'A', 'A']), ('B', ['B', 'B', 'B']), ('C', ['C', 'C']), ('D', ['D']), ('A', ['A', 'A']), ('B', ['B', 'B', 'B'])]

因为函数的这个特性，所以我们通常有必要将*iterable*中的元素按照`key`函数的返回值进行排列，然后在使用本函数进行分组。

### itertools.islice(iterable, stop) & itertools.islice(iterable, start, stop[, step])

与built-in函数`slice()`的行为类似，不同之处在于：返回值不同，`islice()`返回迭代器而`slice()`直接返回结果；`islice()`不支持参数为负数的索引，即*start*、*stop*、*step*不能为负。

    # islice('ABCDEFG', 2) --> A B
    # islice('ABCDEFG', 2, 4) --> C D
    # islice('ABCDEFG', 2, None) --> C D E F G
    # islice('ABCDEFG', 0, None, 2) --> A C E G

### itertools.starmap(function, iterable)

与built-in函数`map()`的行为类似，不同之处在于：`map()`与`starmap()`传递参数的区别类似于`function(a, b)`与`function(*c)`的区别，`map()`有一个对参数列表中的参数有一个类似zip的动作，会把相应的参数从后面的迭代对象列表中取出来放在一起使用；而`starmap()`则有一个类似unpack的动作，把tuple直接传递给函数。

    >>> list(map(pow, [2, 3, 10], [5, 2, 3]))
    [32, 9, 1000]
    >>> list(itertools.starmap(pow, [(2,5), (3,2), (10,3)]))
    [32, 9, 1000]

### itertools.takewhile(predicate, iterable)

这个函数的行为与`itertools.dropwhile(predicate, iterable)`正好相反，它会返回被*predicate*测试为**真**的值，并在第一个**假**出现时终止。

    # takewhile(lambda x: x<5, [1,4,6,4,1]) --> 1 4

### itertools.tee(iterable, n=2)

函数的大概行为：假设我们调用tee([1, 2, 3, 4, 5], 4)，它将返回一个tuple，其中包含了4个generator（简称g1, g2, g3, g4），每个gen维护一个deque（相应的称为d1, d2, d3, d4），当任一个generator被调用时，假设现在使用g1：

* g1被唤醒，从*iterable*中取值`1`，向d1中push值`1`，此时d2、d3、d4同样被push了值`1`，而后g1将d1中的值`1`pop给调用函数，而d2、d3、d4不变；--> ([], [1], [1], [1])
* g1取值`2`，向d1中push值`2`，此时d2、d3、d4也得到值`2`，而后g1仍会将d1中的值`2`pop给调用函数，同样的，d2、d3、d4不变；--> ([], [1, 2], [1, 2], [1, 2])
* 后面的情形同样如此，于是调用函数依次得到了返回值1, 2, 3, 4, 5，而d1, d2, d3, d4在每一步中的变化为：
* ([], [1], [1], [1]) -> 
* ([], [1, 2], [1, 2], [1, 2]) -> 
* ([], [1, 2, 3], [1, 2, 3], [1, 2, 3]) -> 
* ([], [1, 2, 3, 4], [1, 2, 3, 4], [1, 2, 3, 4]) -> 
* ([], [1, 2, 3, 4, 5], [1, 2, 3, 4, 5], [1, 2, 3, 4, 5])
* 此时*iterable*中的值被取完，`raise StopIteration`，而后g2, g3, g4会依次弹出自己的deque给调用函数。
* 所以有：

```
>>> for i in itertools.tee([1, 2, 3, 4, 5], 4):
...     print(list(i))
...
[1, 2, 3, 4, 5]
[1, 2, 3, 4, 5]
[1, 2, 3, 4, 5]
[1, 2, 3, 4, 5]
```

* 其实是将来源给了g1并输出的同时，复制给了g2, g3, g4，在讲g2, g3, g4输出，所以这个行为看起来与Unix中的`tee`命令相似。

### itertools.zip_longest(*iterables, fillvalue=None)

与built-in函数`zip()`的行为类似，不同之处在于：`zip()`的*iterables*中只要有一个迭代器耗尽，`zip`就会停止，而`zip_longest()`则会按照最长的迭代器处理，并为短的迭代器使用*fillvalue*指定的值补位。

    >>> list(itertools.zip_longest('ABCD', 'xy', fillvalue='-'))
    [('A', 'x'), ('B', 'y'), ('C', '-'), ('D', '-')]
    >>> list(zip('ABCD', 'xy', ))
    [('A', 'x'), ('B', 'y')]

## 组合数学相关函数

简单的排列组合函数。

### itertools.product(*iterables, repeat=1)

这个函数返回的是笛卡尔积，product(A, B)的返回结果类似((x, y) for x in A for y in B)。函数的行为类似汽车里程表，变化最大的值在最右边，最右迭代完成后次最右取下一个值（类似进位）。所以，如果*iterables*中的几个可迭代对象是有顺序的，则`product()`函数的返回结果会是有序的。

*repeat*参数zhid每一个*iterables*中的迭代器重复出现几次。

    # product('ABCD', 'xy') --> Ax Ay Bx By Cx Cy Dx Dy
    # product(range(2), repeat=3) --> 000 001 010 011 100 101 110 111

### itertools.permutations(iterable, r=None)

若不指定*r*则返回*iterable*的全排列，即`P(len(iterable), len(iterable))`；若指定*r*则返回`P(r, len(iterable))`。

    # permutations('ABCD', 2) --> AB AC AD BA BC BD CA CB CD DA DB DC
    # permutations(range(3)) --> 012 021 102 120 201 210

排列是笛卡尔积的子集，所以可以借助上面的`product()`产生结果集，而后从中滤掉有重复元素的项：

    def permutations(iterable, r=None):
        pool = tuple(iterable)
        n = len(pool)
        r = n if r is None else r
        for indices in product(range(n), repeat=r):
            if len(set(indices)) == r:
                yield tuple(pool[i] for i in indices)

### itertools.combinations(iterable, r)

函数返回指定*iterable*的组合结果，即`C(r, len(iterable))`。

    # combinations('ABCD', 2) --> AB AC AD BC BD CD
    # combinations(range(4), 3) --> 012 013 023 123

组合是排列的子集，所以可以借助上面的`permutations()`产生结果集，而后从中滤掉未排序的项：

    def combinations(iterable, r):
        pool = tuple(iterable)
        n = len(pool)
        for indices in permutations(range(n), r):
            if sorted(indices) == list(indices):
                yield tuple(pool[i] for i in indices)

### itertools.combinations_with_replacement(iterable, r)

函数返回指定*iterable*的组合，但*iterable*中的元素可以重复*r*次。

    # combinations_with_replacement('ABC', 2) --> AA AB AC BB BC CC

元素可重复的组合是笛卡尔积的自己，所以可以借助上面的`product()`产生结果集，而后从中滤掉未排序的项：

    def combinations_with_replacement(iterable, r):
        pool = tuple(iterable)
        n = len(pool)
        for indices in product(range(n), repeat=r):
            if sorted(indices) == list(indices):
                yield tuple(pool[i] for i in indices)

## 一些示例

使用itertools函数的优势在于，需要值的时候才进行计算，否则挂起。这种惰性求值相对于“一次性把所以值加进内存整体求值并返回”而言更加节省资源。使用itertools完成工作的代码风格更函数式，像管道一样把每个工具连接起来，这样可以帮助消除计算中出现的大量的临时变量。“向量化”的代码执行效率通常高于使用for循环和生成器，因为这样会有解释器开销。

    def take(n, iterable):
        "Return first n items of the iterable as a list"
        return list(islice(iterable, n))

    def tabulate(function, start=0):
        "Return function(0), function(1), ..."
        return map(function, count(start))
    
    def consume(iterator, n):
        "Advance the iterator n-steps ahead. If n is none, consume entirely."
        # Use functions that consume iterators at C speed.
        if n is None:
            # feed the entire iterator into a zero-length deque
            collections.deque(iterator, maxlen=0)
        else:
            # advance to the empty slice starting at position n
            next(islice(iterator, n, n), None)
    
    def nth(iterable, n, default=None):
        "Returns the nth item or a default value"
        return next(islice(iterable, n, None), default)
    
    def quantify(iterable, pred=bool):
        "Count how many times the predicate is true"
        return sum(map(pred, iterable))
    
    def padnone(iterable):
        """Returns the sequence elements and then returns None indefinitely.
    
        Useful for emulating the behavior of the built-in map() function.
        """
        return chain(iterable, repeat(None))
    
    def ncycles(iterable, n):
        "Returns the sequence elements n times"
        return chain.from_iterable(repeat(tuple(iterable), n))

    def dotproduct(vec1, vec2):
        return sum(map(operator.mul, vec1, vec2))
    
    def flatten(listOfLists):
        "Flatten one level of nesting"
        return chain.from_iterable(listOfLists)
    
    def repeatfunc(func, times=None, *args):
        """Repeat calls to func with specified arguments.
    
        Example:  repeatfunc(random.random)
        """
        if times is None:
            return starmap(func, repeat(args))
        return starmap(func, repeat(args, times))
    
    def pairwise(iterable):
        "s -> (s0,s1), (s1,s2), (s2, s3), ..."
        a, b = tee(iterable)
        next(b, None)
        return zip(a, b)
    
    def grouper(iterable, n, fillvalue=None):
        "Collect data into fixed-length chunks or blocks"
        # grouper('ABCDEFG', 3, 'x') --> ABC DEF Gxx"
        args = [iter(iterable)] * n
        return zip_longest(*args, fillvalue=fillvalue)
    
    def roundrobin(*iterables):
        "roundrobin('ABC', 'D', 'EF') --> A D E B F C"
        # Recipe credited to George Sakkis
        pending = len(iterables)
        nexts = cycle(iter(it).__next__ for it in iterables)
        while pending:
            try:
                for next in nexts:
                    yield next()
            except StopIteration:
                pending -= 1
                nexts = cycle(islice(nexts, pending))

    def partition(pred, iterable):
        'Use a predicate to partition entries into false entries and true entries'
        # partition(is_odd, range(10)) --> 0 2 4 6 8   and  1 3 5 7 9
        t1, t2 = tee(iterable)
        return filterfalse(pred, t1), filter(pred, t2)
    
    def powerset(iterable):
        "powerset([1,2,3]) --> () (1,) (2,) (3,) (1,2) (1,3) (2,3) (1,2,3)"
        s = list(iterable)
        return chain.from_iterable(combinations(s, r) for r in range(len(s)+1))
    
    def unique_everseen(iterable, key=None):
        "List unique elements, preserving order. Remember all elements ever seen."
        # unique_everseen('AAAABBBCCDAABBB') --> A B C D
        # unique_everseen('ABBCcAD', str.lower) --> A B C D
        seen = set()
        seen_add = seen.add
        if key is None:
            for element in filterfalse(seen.__contains__, iterable):
                seen_add(element)
                yield element
        else:
            for element in iterable:
                k = key(element)
                if k not in seen:
                    seen_add(k)
                    yield element

    def partition(pred, iterable):
        'Use a predicate to partition entries into false entries and true entries'
        # partition(is_odd, range(10)) --> 0 2 4 6 8   and  1 3 5 7 9
        t1, t2 = tee(iterable)
        return filterfalse(pred, t1), filter(pred, t2)
    
    def powerset(iterable):
        "powerset([1,2,3]) --> () (1,) (2,) (3,) (1,2) (1,3) (2,3) (1,2,3)"
        s = list(iterable)
        return chain.from_iterable(combinations(s, r) for r in range(len(s)+1))
    
    def unique_everseen(iterable, key=None):
        "List unique elements, preserving order. Remember all elements ever seen."
        # unique_everseen('AAAABBBCCDAABBB') --> A B C D
        # unique_everseen('ABBCcAD', str.lower) --> A B C D
        seen = set()
        seen_add = seen.add
        if key is None:
            for element in filterfalse(seen.__contains__, iterable):
                seen_add(element)
                yield element
        else:
            for element in iterable:
                k = key(element)
                if k not in seen:
                    seen_add(k)
                    yield element

    def random_combination(iterable, r):
        "Random selection from itertools.combinations(iterable, r)"
        pool = tuple(iterable)
        n = len(pool)
        indices = sorted(random.sample(range(n), r))
        return tuple(pool[i] for i in indices)
    
    def random_combination_with_replacement(iterable, r):
        "Random selection from itertools.combinations_with_replacement(iterable, r)"
        pool = tuple(iterable)
        n = len(pool)
        indices = sorted(random.randrange(n) for i in range(r))
        return tuple(pool[i] for i in indices)

