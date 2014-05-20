# 数据持久化-`pickle`模块

* 注：下面提到的pickle/pickling可以译为序列化，unpickle/unpickling可以译为反序列化。

**首先，需要特别注意，对于错误的甚至恶意的字节流，pickle并没有安全措施。所以，不要unpickle从不受信来源得到的数据。**

## 与其他序列化相关模块的关系

`pickle`和`marshal`模块提供了把Python对象转为字节流存储或传输，从字节流还原Python对象的功能。

在大多数情况下推荐使用`pickle`模块，因为`marshal`是一个偏底层的模块，主要为Python字节码（`.pyc`）提供支持。

`pickle`与`marshal`的主要区别：

* `pickle`会持续跟踪已被序列化的对象，这样，当后文中又出现该对象的引用时，就不会被重复序列化了。而`marshal`不会这么做。

    从这里可以看出，`pickle`对**递归对象**（一个对象包含了对自身的引用）和**对象共享**（多个出现在继承树中不同层级的对同一个对象的引用）同样有效。如果你试图使用`marshal`序列化一个递归对象，你的Python解释器会因此崩溃。对于**对象共享**，`pickle`仅储存一份这样的对象，然后确保每个引用都指向这个对象。于是，即使在序列化之后，共享对象仍旧保持共享，这一点对于那些可变对象（mutable object如list）很重要。
    
* `marshal`不可以被用来序列化用户定义类及其实例。而`pickle`则没有这个限制，不过类的定义必须是可导入的，而且应该保证与它被序列化时所处的模块相同（即保持上下文环境一致）。

* `marshal`序列化的结果不保证在各Python版本中的通用性。因为`marshal`的主要任务是提供对`.pyc`的支持，当前的Python的`.pyc`是没有向后兼容性的（如果想要使用原来的`.pyc`文件，还是拿到源码，使用`compile()`在编译一次吧）。不过，如果以后真的有“使`.pyc`向后兼容”的呼声，Python团队保留实现这个功能的权力。

`pickle`与`json`的主要区别：

* JSON是文本格式的序列化（其返回值是Unicode文本，通常以`utf-8`编码），pickle是二进制格式的序列化（控制台都可以看出来，JSON返回是`'***'`，pickle是`b'***'`）。
* JSON是“人类可读”的字符串，而pickle不是。
* JSON因其强大的系统间协作能力（因为是文本嘛），也被广泛的应用在Python以外的各大生态系统中，而pickle的结果是仅限于Python的。
* 默认情况下，JSON仅能够描述一小部分Python内建类型（built-in types，如dict和list），不能用在用户定义类型中；pickle可以用来表达海量的Python类型（其中的很多巧妙利用了Python的内省机制；比较复杂的情况参考下文关于“类实例序列化”的条目）

## 数据流的格式

使用`pickle`得到的数据格式是仅能用于Python的。此特性的优势在于，不会受到其他标准的牵制（如JSON、XDR，XDR还不支持对象共享…）；不过，这也意味着，非Python程序可能无法从pickle的结果中还原Python对象。

`pickle`默认输出的对象格式，是已经进行了一定程度压缩的二进制表达，如果想要进一步优化其结果的大小，可以使用[Data Compression and Archiving](https://docs.python.org/3.4/library/archiving.html)中的模块进行处理。

`pickletools`模块为`pickle`的输出数据流提供了分析、解释工具，`pickletools`也在源码中为pickle协议使用的操作码做了大量的注释。

当前，有5种不同的pickling协议正在使用中，协议的版本越高，unpickling该协议产生数据所需的Python版本也就越高：

* 0版协议，是最初的“人类可读”协议，对早期的几个Python版本是提供向后兼容。
* 1版协议，是最初的二进制格式，同样对早期的几个Python版本提供向后兼容。
* 2版协议，最初在Python2.3中引入，提供了比原来高效多的对新类（new-style classes）的序列化。[PEP 307](http://legacy.python.org/dev/peps/pep-0307/)中有2版协议的具体改进。
* 3版协议，最初在Python3.0中引入，提供了对[bytes](https://docs.python.org/3.4/library/functions.html#bytes)对象的明确支持。同时，3版协议的输出数据不支持使用Python2.x进行unpickle。当使用Python3时，这是默认的，同时也是推荐使用的协议版本。
* 4版协议，最初在Python3.4中引用，为了提供对大型对象的支持，也提供了对更多类型的支持，同时做了些许调优。可以在[PEP 3154](http://legacy.python.org/dev/peps/pep-3154/)中查到4版协议的具体改进。

注：序列化是一个比持久化更底层的概念，虽然`pickle`模块也涉及到文件对象操作，但它并不负责持久化对象的命名问题，也不关心持久化对象的一致性操作问题。`pickle`可以把一个复杂对象转为字节流，也可以将这个字节流准确的还原为Python对象。我们可以选择把这个字节流存入文件，同样也可以选择通过网络传输或是存入数据库。`shelve`模块提供了一个pickle/unpickle对象到DBM-style数据库的接口。

## 模块接口

要序列化一个对象（包括其继承树），调用`dumps()`函数即可。同样的，要反序列化一段数据流，只需调用`loads()`即可。如果需要对序列化/反序列化过程进行更多控制，创建`Pickler`/`Unpickler`对象就可以了。

模块级常量：

### pickle.HIGHEST_PROTOCOL

一个用来表示模块提供的最高级协议的整型。可以用来为`dump()`、`dumps()`、`Pickler`构造函数提供*protocol*参数。

### pickle.DEFAULT_PROTOCOL

一个用来表示模块当前使用的协议的整型。可能会低于`HIGHEST_PROTOCOL`。当下是`3`版协议。

模块级函数：

### pickle.dump(obj, file, protocol=None, *, fix_imports=True)

对*obj*进行pickle，将结果写入[类文件对象](https://docs.python.org/3.4/glossary.html#term-file-object)*file*。等价于`Pickler(file, protocol).dump(obj)`。

*protocol*指定pickle使用的协议版本，从0到`HIGHEST_PROTOCOL`取值，默认使用`DEFAULT_PROTOCOL`。如果设为负数，则使用`HIGHEST_PROTOCOL`。

*file*对象必须有`write()`方法（一个接受`bytes`为唯一参数的方法）。该对象可以是通过`open(file, 'wb')`方式打开的磁盘物理文件，可以是[io.BytesIO](https://docs.python.org/3.4/library/io.html#io.BytesIO)实例，或其他拥有这种接口的任何自定义对象。

如果*fix_imports*为真且*protocol*小于3，函数则会试着将Python3中的新模块名映射为Python2中的旧版，这样可以使pickle的数据流在Python2中unpickle。

### pickle.dumps(obj, protocol=None, *, fix_imports=True)

除了返回值为`bytes`对象外，其他与`pickle.dump()`一致。`dump()`相当于`dumps()`后接写文件操作。

### pickle.load(file, *, fix_imports=True, encoding="ASCII", errors="strict")

从[类文件对象](https://docs.python.org/3.4/glossary.html#term-file-object)*file*中读取pickle的结果，将对象还原并返回。等价于`Unpickler(file).load()`。

*file*对象必须有`read()`方法（接受整型为唯一参数的方法）和`readline()`方法（没有参数），这两个方法都应该返回`bytes`对象。该对象可以是通过`open(file, 'rb')`方式打开的磁盘物理文件，可以是[io.BytesIO](https://docs.python.org/3.4/library/io.html#io.BytesIO)实例，或其他拥有这种接口的任何自定义对象。

*fix_imports*, *encoding*, *errors*用来为由Python生成的pickle数据流提供兼容性支持。如果*fix_imports*为真，函数会试着将Python2中的旧模块名映射为Python3中使用的新版。*encoding*和*error*指定函数如何解码由Python2生成的8-bit字符串对象，默认值分别为`ASCII`和`strict`。*encoding*可以设为`bytes`，用来把8-bit字符串当做bytes对象读取。

### pickle.loads(bytes_object, *, fix_imports=True, encoding="ASCII", errors="strict")

除了从*bytes_object*加载并还原对象以外，其他与`pickle.load()`一致。

模块级异常：

### exception pickle.PickleError

pickle时的通用异常，继承自`Exception`。

### exception pickle.PicklingError

当`Pickler`遇到不可以被pickle的对象时抛出，继承自`PickleError`。

### exception pickle.UnpicklingError

在unpickling时遇到问题抛出，如数据损坏或违反安全设置等。继承自`PickleError`。

模块级类对象：

### class pickle.Pickler(file, protocol=None, *, fix_imports=True)

获取一个二进制文件句柄，并写入pickle的数据流。

*protocol*指定pickle使用的协议版本，从0到`HIGHEST_PROTOCOL`取值，默认使用`DEFAULT_PROTOCOL`。如果设为负数，则使用`HIGHEST_PROTOCOL`。

*file*对象必须有`write()`方法（一个接受`bytes`为唯一参数的方法）。该对象可以是通过`open(file, 'wb')`方式打开的磁盘物理文件，可以是[io.BytesIO](https://docs.python.org/3.4/library/io.html#io.BytesIO)实例，或其他拥有这种接口的任何自定义对象。

如果*fix_imports*为真且*protocol*小于3，函数则会试着将Python3中的新模块名映射为Python2中的旧版，这样可以使pickle的数据流在Python2中unpickle。

#### dump(obj)

pickle对象*obj*，并将数据流写入构造函数中指定的*file*。

#### persistent_id(obj)

默认无动作，子类继承重写时使用。

如果`persistent_id()`返回`None`，*obj*会照常被pickled。其他的返回值会使`Pickler`返回这个函数的返回值（`Pickler`本应得到序列化数据流并将其写入文件，若此函数有返回值，则得到此函数的返回值并写入文件），作为持久化对象的ID。这个持久化ID的解释应当定义在`Unpickler.persistent_load()`中（利用`persistent_load()`函数从持久化对象的ID还原对象，该方法定义了还原对象的过程，并返回得到的对象）。注意，`persistent_id()`的返回值不可以拥有自己的ID。

用法参见下面的“持久化外部对象”。

#### dispatch_table

`Pickler`对象的dispatch表是`copyreg.pickle()`中用到的reduction函数的注册。dispatch表本身是一个class到其reduction函数的映射键值对。reduction函数只接受一个参数，就是其关联的class，函数行为应当遵守`__reduce__()`接口规范。

`Pickler`对象默认并没有`dispatch_table`属性，该对象默认使用`copyreg`模块中定义的全局dispatch表。如果想要特定`Pickler`类或子类执行指定的序列化函数，就可以将`dispatch_table`设置为类字典对象（dict-like object）。如果`Pickler`子类设置了`dispatch_table`，则该子类会使用这个表作为默认的dispatch表。

用法参见下面的“dispatch表示例”。

#### fast

该属性已被舍弃，设为真时，pickle使用快速模式。该模式不可以用在涉及自指（self-referential）对象，否则会引起`Pickler`的无限递归。

如果需要进一步提高pickle的压缩率，参考使用`pickletools.optimize()`。

### class pickle.Unpickler(file, *, fix_imports=True, encoding="ASCII", errors="strict")

读取pickle数据流的二进制文件，并还原其中的对象。

*file*对象必须有`read()`方法（接受整型为唯一参数的方法）和`readline()`方法（没有参数），这两个方法都应该返回`bytes`对象。该对象可以是通过`open(file, 'rb')`方式打开的磁盘物理文件，可以是[io.BytesIO](https://docs.python.org/3.4/library/io.html#io.BytesIO)实例，或其他拥有这种接口的任何自定义对象。

*fix_imports*, *encoding*, *errors*用来为由Python生成的pickle数据流提供兼容性支持。如果*fix_imports*为真，函数会试着将Python2中的旧模块名映射为Python3中使用的新版。*encoding*和*error*指定函数如何解码由Python2生成的8-bit字符串对象，默认值分别为`ASCII`和`strict`。*encoding*可以设为`bytes`，用来把8-bit字符串当做bytes对象读取。

#### load()

从构造函数指定的文件中读取pickle数据流，并返回从中还原的对象。

#### persistent_load(pid)

默认行为是`raise UnpicklingError`。

如果定义，则应该从给定的*pid*还原出pickled对象并返回。如果给定的*pid*不合法，则应当`raise UnpicklingError`

应当与`persistent_id()`方法对应。

用法参见下面的“持久化外部对象”。

#### find_class(module, name)

函数会按需导入*module*并以*name*指定的名称返回，两个参数都是字符串。不要被这个函数的名字迷惑，它同样可以用来导入函数。

子类重写这个函数可以针对特定类制定定特定的导入方式，所以存在潜在安全问题。参见"限制全局量"的例子。

## 可以被序列化/反序列化的对象

类型如下：

* `None`, `True`, `False`；
* 整型、浮点型、复数；
* `str`, `bytes`, `bytearray`；
* 只包含可序列化对象的集合，包括`tuple`, `list`, `set`, `dict`；
* 模块级的函数对象（使用`def`定义在模块顶层，`lambda`函数则**不可以**）；
* 模块级的内建函数（built-in function）；
* 模块级的类；
* 其`__dict__`，或调用其`__getstate__()`所得结果可以被序列化的类实例，参见下面的“序列化类实例”；

尝试pickle不可以被序列化的对象会`raise PicklingError`。需要注意的是，此时，不可控的结果可能已经被写入指定文件中。尝试pickle递归层级深的对象时，可能会因为超出解释器设定的最大递归层级而导致`raise RuntimeError`，可以通过设置`sys.setrecursionlimit()`调整递归层级，不过请谨慎使用这个函数，因为可能会导致解释器崩溃。

函数（内建函数或用户定义函数）在被序列化时，使用的是函数全名（参见[qualified name](https://docs.python.org/3.4/glossary.html#term-qualified-name)），这就是为什么`lambda`函数不可以被序列化——所有的匿名函数都有同一个名字：`<lambda>`。函数被序列化时，只有函数所在的模块名，与函数名会被序列化，函数体及其属性不会被序列化。因此，在反序列化时，函数所属的模块必须是可以被导入的，而且模块必须包含这个函数被序列化的名称，否则会抛出异常。

类似的，在反序列化时，类也需要反序列化的环境中，有它在被序列化时使用的全名。同样的，类体及其数据不会被序列化。所以，在下面的例子中`attr`不会存在于反序列化的环境中：

```
class Foo:
    attr = 'A class attribute'

picklestring = pickle.dumps(Foo)
```

这就是可序列化的函数或类必须定义在模块级的原因。

类似的，在序列化类实例时，其类体和类数据不会跟着实例一起被序列化，只有实例数据会被序列化。这种机制可以使得，在我们修改类定义、给类增加方法之后，仍然可以通过载入原来版本类实例的序列化数据来还原该实例。如果你准备长期使用一个对象，可能会导致同时存在较多版本的类体，可以为实例添加版本号，这样的话，就可以通过类的`__setstate__()`接口得到恰当的还原版本。

## 序列化类实例

我们来了解一下序列化/反序列化类实例的普适机制，从而可以做到定义、制定、控制序列化/反序列化类实例的过程。

通常，使一个实例可序列化不需要附加任何代码。pickle模块默认会通过Python的内省机制获得实例的类及属性。而当实例反序列化时，他的`__init__()`函数通常不会被调用，其默认动作是：先创建一个未初始化的实例，而后通过update其`__dict__`的方式还原属性，像这样：

```
def save(obj):
    return (obj.__class__, obj.__dict__)

def load(cls, attributes):
    obj = cls.__new__(cls)
    obj.__dict__.update(attributes)
    return obj
```

我们可以通过定义一下的一个或几个方法来修改序列化的默认行为：

### `object.__getnewargs_ex__()`

对于使用4版或更高版协议的pickle，实现了`__getnewargs_ex__()`的类可以控制在unpickling时传给`__new__()`方法的参数。方法必须以`(args, kwargs)`返回从而提供构建实例所需的参数，其中*args*是表示位置参数的`tuple`，而*kwargs*是表示关键字参数的`dict`。这些参数会被传给`__new__()`。

如果类的`__new__()`方法只接受关键字参数，则应当实现这个方法。如果不是，为了兼容性，更推荐实现`__getnewargs__()`方法。

### `object.__getnewargs__()`

这个方法同上一个方法类似，只不过支持到版本2或更高的协议。它要求返回的*args*是一个将要传给`__new__()`方法的`tuple`。

在版本4或更高的协议中，如果定义了`__getnewargs_ex__()`就不会再唤起`__getnewargs__()`。

### `object.__getstate__()`

Classes can further influence how their instances are pickled; if the class defines the method __getstate__(), it is called and the returned object is pickled as the contents for the instance, instead of the contents of the instance’s dictionary. If the __getstate__() method is absent, the instance’s __dict__ is pickled as usual.

类还可以进一步控制其实例的序列化过程。如果类定义了`__getstate__()`，它就会被调用，其返回的对象是被当做实例内容序列化的，而通常是当做实例的`__dict__`进行序列化的。如果`__getstate__()`未定义，实例的`__dict__`会被照常序列化。

### `object.__setstate__(state)`

当反序列化时，如果定义了`__setstate__()`，就会调用它并取回还原实例所需要的实例状态，此时不要求实例的状态对象必须是`dict`。否则，需要的实例对象必须是`dict`，取回字典后会将字典内容更新给这个新的类实例。

**注意**，如果`__getstate__()`返回值经布尔测试为假，则在反序列化时`__setstate__()`不会被调用。

参见“控制有状态的对象”了解`__getstate__()`和`__setstate__()`的使用。

**注意**，在反序列化时，实例的`__getattr__()`, `__getattribute__()`, `__setattr__()`方法可能会被调用，而这几个函数需要模型内部不变量存在，所以该类应当实现`__getnewargs__()`或`__getnewargs_ex__()`来准备这些内部不变量，否则`__new__()`和`__init__()`都不会被调用。

可以看出，其实pickle并不直接调用上面的几个函数。事实上，这几个函数是复制协议的一部分，它们实现了`__reduce__()`接口。复制协议为在序列化或复制对象的过程中取得所需数据提供了统一接口。

尽管这个协议功能很强，但是直接在类中实现`__reduce__()`接口容易产生错误。因此，设计类时应当尽可能的使用高级接口（比如`__getnewargs_ex__()`, `__getstate__()`, `__setstate__()`）。后面可以看到适合直接实现`__reduce__()`接口的状况（可能因为别无他法，或是为了获得更好的性能）。

### `object.__reduce__()`

函数不需要参数，应当返回一个字符串，或更详细的，返回一个`tuple`（返回值通常被称为“化简值”）。

如果返回字符串，该字符串会被当做`object`在全局变量中的名称。这个字符串应当是`object`在模块中的相对名称，pickle模块会搜索模块命名空间找到对象所属模块。这种行为常在单例模式使用。

如果返回`tuple`，则会包含2到5个元素，可选元素可以省略或设置为`None`：

* 必选元素，一个可调用对象，该对象会在创建最初版`object`时调用；
* 必选元素，一个`tuple`，是可调用对象的参数，如果可调用对象不接受参数，必须提供一个空`tuple`（不能因为空参而省略）；
* 可选元素，用于表示`object`的状态，将被传给`__setstat__()`函数。如果`object`没有`__setstat__()`方法，则这个元素必须是dict类型，并会被加入`object.__dict__`中。
* 可选元素，一个返回连续项的迭代器（而不是序列）。这些项会被`append()`逐个加入`object`，或被`extend()`批量加入`object`。这个元素主要用于list的子类，也可以用于那些实现了`append()`和`extend()`方法的类。（具体是使用`append()`还是`extend()`取决于pickle协议版本以及这个元素的项数，所以这两个方法必须同时被类支持）；
* 可选元素，一个返回连续键值对的迭代器（而不是序列）。这些键值对将会以`object[key] = value`的方式提供给`object`。这个元素主要用于dict子类，也可以用于那些实现了`__setitem__()`的类。

### `object.__reduce_ex__(protocol)`

除了实现上面的`__reduce__()`外，也可以选择实现`__reduce_ex__()`。这两个方法的唯一区别是`__reduce_ex__()`接受一个整型参数用于指定pickle版本。如果定义了这个函数，则会覆盖`__reduce__()`的行为。这个函数主要用于为以前的Python版本提供向后兼容性。

### 持久化外部对象

pickle模块提供了对除数据流之外的对象引用的支持，为对象持久化提供便利。这些外部对象以持久化ID的方式被pickle模块引用，这个ID应是一个整数的字符串形式（用于0版协议）或是一个任意对象（用于任意新版协议）。

pickle模块不提供对持久化ID的解析工作，这个工作应由用户分别定义在`Pickler.persistent_id()`和`Unpickler.persistent_load()`中。

要通过使用持久化ID进行对象持久化，必须在`Pickler`中实现`persistent_id()`，该方法接受需要被持久化的对象作为参数，返回一个`None`或返回该对象的持久化ID。如果返回`None`，该对象会被按照默认方式持久化为数据流；如果返回持久化ID的字符串，则会吧该对象持久化为这个ID，同时需要在`Unpickler`中定义通过这个ID还原对象的方法。

要勇敢持久化对象ID还原对象，必须在`Unpicker`中实现`persistent_load()`，该方法接受持久化ID对象并返回还原这个以持久化ID形式被引用的外部对象。

下面的例子可以加强对使用持久化ID进行外部对象持久化的理解：

```
# Simple example presenting how persistent ID can be used to pickle
# external objects by reference.

import pickle
import sqlite3
from collections import namedtuple

# Simple class representing a record in our database.
MemoRecord = namedtuple("MemoRecord", "key, task")

class DBPickler(pickle.Pickler):

    def persistent_id(self, obj):
        # Instead of pickling MemoRecord as a regular class instance, we emit a
        # persistent ID.
        if isinstance(obj, MemoRecord):
            # Here, our persistent ID is simply a tuple, containing a tag and a
            # key, which refers to a specific record in the database.
            return ("MemoRecord", obj.key)
        else:
            # If obj does not have a persistent ID, return None. This means obj
            # needs to be pickled as usual.
            return None


class DBUnpickler(pickle.Unpickler):

    def __init__(self, file, connection):
        super().__init__(file)
        self.connection = connection

    def persistent_load(self, pid):
        # This method is invoked whenever a persistent ID is encountered.
        # Here, pid is the tuple returned by DBPickler.
        cursor = self.connection.cursor()
        type_tag, key_id = pid
        if type_tag == "MemoRecord":            # Fetch the referenced record from the database and return it.
            cursor.execute("SELECT * FROM memos WHERE key=?", (str(key_id),))
            key, task = cursor.fetchone()
            return MemoRecord(key, task)
        else:
            # Always raises an error if you cannot return the correct object.
            # Otherwise, the unpickler will think None is the object referenced
            # by the persistent ID.
            raise pickle.UnpicklingError("unsupported persistent object")


def main():
    import io
    import pprint

    # Initialize and populate our database.
    conn = sqlite3.connect(":memory:")
    cursor = conn.cursor()
    cursor.execute("CREATE TABLE memos(key INTEGER PRIMARY KEY, task TEXT)")
    tasks = (
        'give food to fish',
        'prepare group meeting',
        'fight with a zebra',
        )
    for task in tasks:
        cursor.execute("INSERT INTO memos VALUES(NULL, ?)", (task,))

    # Fetch the records to be pickled.
    cursor.execute("SELECT * FROM memos")
    memos = [MemoRecord(key, task) for key, task in cursor]
    # Save the records using our custom DBPickler.
    file = io.BytesIO()
    DBPickler(file).dump(memos)

    print("Pickled records:")
    pprint.pprint(memos)

    # Update a record, just for good measure.
    cursor.execute("UPDATE memos SET task='learn italian' WHERE key=1")

    # Load the records from the pickle data stream.
    file.seek(0)
    memos = DBUnpickler(file, conn).load()

    print("Unpickled records:")
    pprint.pprint(memos)


if __name__ == '__main__':
    main()
```

### dispatch表示例

如果想要对指定的类型执行自定义的序列化函数，而又不想增加额外的代码，就可以使用`Pickler`的`dispatch_table`。

在[copyreg](https://docs.python.org/3.4/library/copyreg.html?highlight=copyreg#module-copyreg)模块的`copyreg.dispatch_table`中定义了全局dispatch表。通常应该复制这个全局表在进行修改。

比如：

```
f = io.BytesIO()
p = pickle.Pickler(f)
p.dispatch_table = copyreg.dispatch_table.copy()
p.dispatch_table[SomeClass] = reduce_SomeClass
```

上面的代码修改了`Pickler`实例的dispatch表，使其对*SomeClass*进行特殊处理。而下面的代码：

```
class MyPickler(pickle.Pickler):
    dispatch_table = copyreg.dispatch_table.copy()
    dispatch_table[SomeClass] = reduce_SomeClass
f = io.BytesIO()
p = MyPickler(f)
```

将dispatch表直接写在了`Pickler`的子类定义中，于是所有的这个子类实例都会使用这个修改过的dispatch表了。也可以用下面的代码达到同样的效果：

```
copyreg.pickle(SomeClass, reduce_SomeClass)
f = io.BytesIO()
p = pickle.Pickler(f)
```
### 控制有状态的对象

下面的代码示例了如何修改序列化的行为。代码中，`TextReader`类负责打开文件，每次调用其`readline()`方法则返回文件中的一行字符。在pickle这个类的实例时，除了文件对象，其他属性都会被保存。当unpickle实例时，需要重新打开文件，而后将`readline()`恢复到pickle前的位置。实现这些功能需要实现`__setstate__()`和`__getstate__()`方法：

```
class TextReader:
    """Print and number lines in a text file."""

    def __init__(self, filename):
        self.filename = filename
        self.file = open(filename)
        self.lineno = 0

    def readline(self):
        self.lineno += 1
        line = self.file.readline()
        if not line:
            return None
        if line.endswith('\n'):
            line = line[:-1]
        return "%i: %s" % (self.lineno, line)

    def __getstate__(self):
        # Copy the object's state from self.__dict__ which contains
        # all our instance attributes. Always use the dict.copy()
        # method to avoid modifying the original state.
        state = self.__dict__.copy()
        # Remove the unpicklable entries.
        del state['file']
        return state

    def __setstate__(self, state):
        # Restore instance attributes (i.e., filename and lineno).
        self.__dict__.update(state)
        # Restore the previously opened file's state. To do so, we need to
        # reopen it and read from it until the line count is restored.
        file = open(self.filename)
        for _ in range(self.lineno):
            file.readline()
        # Finally, save the file.
        self.file = file
```

使用时会像这样：

```
>>> reader = TextReader("hello.txt")
>>> reader.readline()
'1: Hello world!'
>>> reader.readline()
'2: I am line number two.'
>>> new_reader = pickle.loads(pickle.dumps(reader))
>>> new_reader.readline()
'3: Goodbye!'
```

## 限制全局量

pickle模块在反序列化时，会默认导入序列化数据流中涉及的任何类和函数。对于软件来说，这种行为潜在很大的安全问题，因为这种行为等同于允许解释器运行任何外部代码。比如我们手动写一段序列化数据流交给pickle模块：

```
>>> import pickle
>>> pickle.loads(b"cos\nsystem\n(S'echo hello world'\ntR.")
hello world
0
```

这个例子中，我们导入了`os.system()`函数，并让它接受了`echo hello world`这个参数，等同于直接让系统运行了`echo hello world`命令，虽然这个例子是无害的，但是不难想象如何通过pickle模块做出危险动作。

因此，我们应该通过`find_class()`控制pickle模块的导入行为。这并不只是一个“查找类”的函数，事实上`Unpickler.find_class()`会在任何需要导入全局量（比如类、函数等）的时候被调用。于是，我们可以禁止全局量，或把全局量限制在一个安全的集合内进行“导入审查”。

这是一个仅允许部分内建模块被`Unpickler`导入的例子：

```
import builtins
import io
import pickle

safe_builtins = {
    'range',
    'complex',
    'set',
    'frozenset',
    'slice',
}

class RestrictedUnpickler(pickle.Unpickler):

    def find_class(self, module, name):
        # Only allow safe classes from builtins.
        if module == "builtins" and name in safe_builtins:
            return getattr(builtins, name)
        # Forbid everything else.
        raise pickle.UnpicklingError("global '%s.%s' is forbidden" %
                                     (module, name))

def restricted_loads(s):
    """Helper function analogous to pickle.loads()."""
    return RestrictedUnpickler(io.BytesIO(s)).load()
```

使用时的效果：

```
>>> restricted_loads(pickle.dumps([1, 2, range(15)]))
[1, 2, range(0, 15)]
>>> restricted_loads(b"cos\nsystem\n(S'echo hello world'\ntR.")
Traceback (most recent call last):
  ...
pickle.UnpicklingError: global 'os.system' is forbidden
>>> restricted_loads(b'cbuiltins\neval\n'
...                  b'(S\'getattr(__import__("os"), "system")'
...                  b'("echo hello world")\'\ntR.')
Traceback (most recent call last):
  ...
pickle.UnpicklingError: global 'builtins.eval' is forbidden
```

就像例子中示范的这样，你必须要明白，代码将要反序列化的对象有怎样的潜在危险。所以，从安全的角度考虑，我们可以使用`xmlrpc.client`模块或是第三方解决方案代替pickle。

## 通用示例

使用`dump()`序列化：

```
import pickle

# An arbitrary collection of objects supported by pickle.
data = {
    'a': [1, 2.0, 3, 4+6j],
    'b': ("character string", b"byte string"),
    'c': set([None, True, False])
}

with open('data.pickle', 'wb') as f:
    # Pickle the 'data' dictionary using the highest protocol available.
    pickle.dump(data, f, pickle.HIGHEST_PROTOCOL)
```

使用`load()`反序列化：

```
import pickle

with open('data.pickle', 'rb') as f:
    # The protocol version used is detected automatically, so we do not
    # have to specify it.
    data = pickle.load(f)
```

