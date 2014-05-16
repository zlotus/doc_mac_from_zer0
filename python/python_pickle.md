# 数据持久化-`pickle`模块

* 注：下面提到的pickle/pickling可以译为序列化，unpickle/unpickling可以译为反序列化。

**首先，需要特别注意，对于错误的甚至恶意的字节流，pickle并没有安全措施。所以，不要unpickle从不受信来源得到的数据。**

## 与别的序列化相关模块的关系

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

### 模块接口

要序列化一个对象（包括其继承树），调用`dumps()`函数即可。同样的，要反序列化一段数据流，只需调用`loads()`即可。如果需要对序列化/反序列化过程进行更多控制，创建`Pickler`/`Unpickler`对象就可以了。

模块级常量：

#### pickle.HIGHEST_PROTOCOL

一个用来表示模块提供的最高级协议的整型。可以用来为`dump()`、`dumps()`、`Pickler`构造函数提供*protocol*参数。

#### pickle.DEFAULT_PROTOCOL

一个用来表示模块当前使用的协议的整型。可能会低于`HIGHEST_PROTOCOL`。当下是`3`版协议。

模块级函数：

#### pickle.dump(obj, file, protocol=None, *, fix_imports=True)

#### pickle.dumps(obj, protocol=None, *, fix_imports=True)

#### pickle.load(file, *, fix_imports=True, encoding="ASCII", errors="strict")

#### pickle.loads(bytes_object, *, fix_imports=True, encoding="ASCII", errors="strict")
