# Python Descriptor

遇到了这个descriptor，Google了下。
干货都来自[Descriptor HowTo Guide](https://docs.python.org/3.3/howto/descriptor.html) 和 [Python Attributes and Methods](http://www.cafepy.com/article/python_attributes_and_methods/python_attributes_and_methods.html)。

## 0. `obj.x`

Python如何在复杂的环境（属性、方法、继承）中查找恰当的属性，是每个pyer早晚都得搞明白的问题，这牵扯到一个**变量名**优先级查找机制。

### attribute lookup order：

1. 调用`obj.__getattribute__()`，若找到x（可能在继承链中递归查找，类似`-R`），进入2，未找到进入6；
2. 如果找到的x是一个descriptor，进入3；如果不是descriptor，则返回x；
3. 如果obj是object，进入4；如果obj是class，进入5；
4. 用obj代表object，则obj.x会被`object.__getattribute__()`翻译成`type(b).__dict__['x'].__get__(b, type(b))`，按照如下优先级查找：

    **data descriptors** > instance variables > **non-data descriptors**
    
    找到返回x，没找到进入6；
5. 用OBJ代表class，则OBJ.x会被`type.__getattribute__()`翻译成`B.__dict__['x'].__get__(None, B)`，找到返回，没找到进入n6. 
6. 以上查找全部落空，如果obj定义有`__getattr__()`则调用并返回x，若未定义则进入7；
7. `raise AttributeError`

#### 继承链查找：

1. 进入`obj.__dict__`，找到返回x，未找到进入2；
2. 进入`obj.__class__.__dict__`，找到返回x，未找到进入3；
3. 遍历`obj.__class__.__bases__` (in `__mro__` order) 的 `__dict__`，找到返回x，未找到进入4；
4. `raise AttributeError`

## 1. `__getattribute__()`

这个方法会拦截所有的变量查找，即便是特殊方法(Special method)也 [**很难**](https://docs.python.org/3.4/reference/datamodel.html?highlight=data model#special-method-lookup)(通过语法特性和内建函数的不明确调用) 绕过`__getattribute__()`

## 2. Descriptor Protocal

只要定义了下面的方法的对象，就成了descriptor：

    __get__(self, obj, type=None)
    __set__(self, obj, value)
    __delete__(self, obj)

如果同时定义了`__get__()`, `__set__()`，就被称为 **data descriptor**。

只定义`__get__()`的对象被称为 **non-data descriptor**。

如果想要 **read-only data descriptor**，就同时定义`__get__()`, `__set__()`，并让`__set__()` `raise AttributeError`即可。

经常用到的`staticmethod()`, `classmethod()`, `property()`都是基于这个协议。

## 应用

主要（我见过的）用于拦截 attribute access、给对象绑定函数、静态/类方法，从上面的attribute lookup order就可以看出。

### 拦截属性访问

实现private variable access。使用property()返回一个**data descriptor**，控制所有对`__x`的 get/set/del。

    class C(object):
        def getx(self): return self.__x
        def setx(self, value): self.__x = value
        def delx(self): del self.__x
        x = property(getx, setx, delx, "I'm the 'x' property.")

或是，假设我们曾经用`Cell('b10').value`访问变量，现在我们想让每次访问得到的结果被重新计算，但是又懒得改以前的代码。那么，保持`Cell('b10').value`不变的情况下，用data descriptor拦截属性访问即可。

    class Cell(object):
        def getvalue(self, obj):
            "Recalculate cell before returning value"
            self.recalc()
            return obj._value
        value = property(getvalue)

### 绑定函数

以前遇到过这种情况，没多想就放过了，现在可以弄清楚了。给实例绑定方法用的就是 **non-data descriptor**。

    >>> class D(object):
    ...     def f(self, x):
    ...         return x

    >>> d = D()
    >>> D.__dict__['f'] # Stored internally as a function
    <function f at 0x00C45070>
    >>> D.f             # Get from a class becomes an unbound method
    <unbound method D.f>
    >>> d.f             # Get from an instance becomes a bound method
    <bound method D.f of <__main__.D object at 0x00B18C90>>

用Python模拟C语言实现，便于理解，C语言实现更加复杂，见 Objects/funcobject.c 中的`func_descr_get()`：

    class Function(object):
        def __get__(self, obj, objtype=None):
            return types.MethodType(self, obj, objtype)

函数可以看成是 **non-data descriptors**，调用时返回 bound 或 unbound 取决于调用者。

像上面 attribute lookup order 中描述的那样：

**object** 调用时，使用`object.__getattribute__()`将`d.f`转为`type(d).__dict__['x'].__get__(d, type(d))`，

**class** 调用时，使用`type.__getattribute__()`将`D.f`转为`D.__dict__['x'].__get__(None, D)`

注意`__get__()`第一个参数的不同。

### 静态方法和类方法

静态方法没有状态，自然就不需要`self`这个参数了，对object调用和class调用返回一致。

    >>> class E(object):
    ...     def f(x):
    ...         print x
    ...     f = staticmethod(f)

    >>> print E.f(3)
    3
    >>> print E().f(3)
    3

用Python模拟C语言实现，便于理解，C语言实现更加复杂，见 Objects/funcobject.c 中的`PyStaticMethod_Type()`：

    class StaticMethod(object):
        def __init__(self, f):
            self.f = f
        def __get__(self, obj, objtype=None):
            return self.f


将`staticmethod()`看成 **non-data descriptors**：不论返回是`__get__(None, E)`还是`__get__(E(), type(E()))`，调用时只返回`f`，与`obj`及`objtype`无关。

类方法需要class做第一个参数，同样对于object调用和class调用返回一致。

    >>> class E(object):
    ...     def f(klass, x):
    ...         return klass.__name__, x
            f = classmethod(f)

    >>> print E.f(3)
    ('E', 3)
    >>> print E().f(3)
    ('E', 3)

用Python模拟C语言实现，便于理解，C语言实现更加复杂，见 Objects/funcobject.c 中的`PyClassMethod_Type()`：

    class ClassMethod(object):
        def __init__(self, f):
            self.f = f
        def __get__(self, obj, klass=None):
            if klass is None:
                klass = type(obj)
            def newfunc(*args):
                return self.f(klass, *args)
            return newfunc

将`classmethod()`看成 **non-data descriptors**：不论返回是`__get__(None, E)`还是`__get__(E(), type(E()))`，调用时返回`f`，是一个以calss（在例中为`E`）为第一个参数，并且可以接受后续参数（在例中为`3`）的函数，与`obj`无关。