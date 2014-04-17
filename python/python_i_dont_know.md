# 我不知道的Python

最近按照知道创宇的技能表加天赋，开始了大坑无数的阅读Python document的路程。

缓慢填坑吧~

## hashable class

在 **Data Model** 一节，`object.__hash__(self)`下有这么一段说明：

一个重写了`__eq__()`的类，如果不显式定义`__hash__()`：

* 会导致该类实例 unhashable，如：不能做dict的key，不能加入set；
* `hash(instance)` 会报 TypeError；
* `isinstance(instance, collections.Hashable)` 返回 False。

所以，如果重写了 `__eq__()`，同时想保持该类hashable，就必须显式定义 `__hash__()`。

如：`__hash__ = <ParentClass>.__hash__`

如果想在不定义 `__eq__()` 的情况下，去掉类的hashable属性：`__hash__ = None` 即可。

矩阵转置小技巧：

    >>> m = [[1,2,3], [4,5,6], [7,8,9]]
    >>> mt = list(zip(*m))
    >>> mt
    [(1, 4, 7), (2, 5, 8), (3, 6, 9)]

