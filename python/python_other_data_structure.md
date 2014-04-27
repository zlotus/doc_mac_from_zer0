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

