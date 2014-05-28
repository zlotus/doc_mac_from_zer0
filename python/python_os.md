# 操作系统接口 - `os`模块

此模块可以让我们“可移植”的使用平台相关函数。如果你只是想要读写文件，可以使用`__builtins__['open']`函数；如果你想要操作文件系统路径，可以使用`os.path`模块；如果你想要在命令行模式下读取所有指定文件的内容，可以使用`fileinput`模块；如果你想要创建临时文件及文件夹，可以使用`tempfile`模块；如果需要文件操作的高级函数，可以使用`shutil`模块。

留意此模块中函数的共性：

* Python设计这个边平台相关内建模块的初衷，是为了让同一个功能在任何支持的平台上，拥有相同的接口。比如，`os.stat(path)`函数在所有支持的平台上都以相同的形式返回*path*的状态（这个返回状态恰巧来自POSIX系统）；
* `os`模块也支持一些平台特有的属性，不过，使用这些平台特有属性可能会影响到代码的可移植性；
* 模块中接受路径或文件名做参数的函数都支持`str`和`bytes`两种类型。如果该函数返回路径或文件名，返回值的类型会与参数类型相同；
* 诸如“支持：Unix”的标记表示在Unix上更常见，而并不是说仅Unix提供支持；
* 如果不特意指明，“支持：Unix”同样使用于Unix内核的Mac OS X；

注意：模块中的函数在遇到：不合法或不可访问的路径或文件名，参数正确但系统不支持时会抛出`OSError`。

### exception os.error

内建异常`OSError`的别名。

### os.name

导入此模块时基于的操作系统名称，现在注册的名称有'posix', 'nt', 'mac', 'ce', 'java'。

`sys.platform`的结果更具体，`os.uname()`的结果还包含操作系统版本等信息。

要得到更详细的系统信息，可以使用`platform`模块。

## 文件名、命令行参数、环境变量

Python中使用`str`表示文件名、命令行参数、环境变量。对于模型操作环境，有必要将这些字符串转为`bytes`再传给操作系统。Python使用文件系统默认编码做这种转换（见`sys.getfilesystemencoding()`）。

Python3.1后：在一些环境中，使用文件系统编码可能会产生错误。所以Python使用了`surrogateescape`控制编码异常，也就是把不可被解码的`bytes`在解码时记为Unicode字符“U+DCxx”，于是这种字符在编码时会被重新译为原始`bytes`。

文件系统默认编码至少要做到能够正确解码每个数值小于128的`bytes`，否则API可能会抛出`UnicodeErrors`异常。

## 进程参数

下列函数和属性提供了当前进程及用户的信息与相应操作。

### os.ctermid()¶

返回控制进程的终端所关联的文件。

```
>>> os.ctermid()
'/dev/tty'
```

支持：Unix。

### os.environ

用`str`储存当前环境变量的映射。如`environ['HOME']`返回当前的用户目录（部分系统支持），相当于C语言中的`getenv("HOME")`。

这个映射在`os`模块被第一次导入的时候就生成了（如，当Python作为`site.py`的一部分启动的时候）。在此之后被修改的环境变量不会体现在`os.environ`中，除非直接修改`os.environ`本身。

如果平台支持`putenv()`，此映射可能既可以用来查询环境变量，也可以用来修改环境变量。`putenv()`会在修改映射时被自动调用。

在Unix上，映射的键与值会使用`sys.getfilesystemencoding()`和`surrogateescape`控制编码异常。如果希望使用不同的编码，可以访问`environb`属性。

注意：直接调用`putenv()`并不会改变`os.environ`，所以还是修改`os.environ`比较好。

注意：在某些环境中（包括FreeBSD和Mac OS X），直接修改`environ`属性可能会导致内存泄露。详情参见操作系统的`putenv`命令或函数。

如果平台不支持`putenv()`，一份被修改的环境变量映射可能会被进程创建函数传给子进程，使得子进程可以使用修改了的环境变量。

如果平台支持`unsetenv()`，就可以通过删除此映射中的项来释放环境变量。当`os.environ`中的项被删除时（包括`pop()`或`clear()`等方法）会自动调用`unsetenv()`。

### os.environb

`os.environ`的`bytes`版。`environ`与`environb`是同步的，修改其中一个会导致另一个也被修改。

`environb`只有在`supports_bytes_environ`为真的时候可用。

### os.chdir(path)
### os.fchdir(fd)
### os.getcwd()

这三个函数在“文件和路径”中详细描述。

### os.fsencode(filename)

使用文件系统编码及`surrogateescape`异常控制来编码*filename*，在Windows上使用`strict`异常控制。若*filename*为`bytes`则直接返回这个字节串。

### os.fsdecode(filename)

使用文件系统编码及`surrogateescape`异常控制来解码*filename*，在Windows上使用`strict`异常控制。若*filename*为`str`则直接返回这个字符串。

### os.getenv(key, default=None)

查询环境变量，如果存在就返回*key*，不存在就返回*default*。*key*、*default*及返回值都是`str`。

在Unix上，映射的键与值会使用`sys.getfilesystemencoding()`和`surrogateescape`控制编码异常。如果希望使用不同的编码，可以调用`os.getenvb()`。

支持：多数Unix，Windows。

### os.getenvb(key, default=None)

查询环境变量，如果存在就返回*key*，不存在就返回*default*。*key*、*default*及返回值都是`bytes`。

支持：多数Unix。

### os.get_exec_path(env=None)

返回可执行对象的搜索路径列表，类似于shell在执行一个命令时的动作。如果指定*env*，则应该是能够查找到PATH的环境变量。*env*默认为空时，会使用`os.environ`。

```
>>> os.get_exec_path()
['/usr/bin', '/bin', '/usr/sbin', '/sbin', '/usr/local/bin', '/opt/X11/bin']
```

### os.getegid()

返回对当前进程有效的组ID，与当前进程中被执行文件的设置ID位有关。

支持：Unix。

### os.geteuid()

返回对当前进程有效的用户ID。

支持：Unix。

### os.getgid()

返回当前进程所属的组ID。

支持：Unix。

### os.getgrouplist(user, group)

返回*user*所属的组ID列表。如果*group*不在列表中，则会被加入。*group*通常来自*user*密码记录中的组ID域。

支持：Unix。

### os.getgroups()

返回当前进程关联的附加组ID。

支持：Unix。

注意：`getgroups()`在Mac OS X中的行为与在其他Unix中的行为有所不同。如果Python解释器编译部署在10.5及更早的版本，`getgroups()`返回与当前用户进程相关的的有效组ID列表，这时的列表的长度会被一个系统定义的整数限制，通常是16，而且列表可能会适当的权限下被`setgroups()`修改。如果是大于10.5的版本，`getgroups()`返回与当前进程有效用户ID相关的用户所在用户组的许可列表。组许可列表可以在进程生命周期的任何时间被修改，`setgroups()`不会修改列表，而且列表大小不再限制在16上。可以通过`sysconfig.get_config_var()`查询部署值`MACOSX_DEPLOYMENT_TARGET`。

### os.getlogin()

返回进程控制终端的登录用户名。通常在需要使用到环境变量`LOGNAME`或`USERNAME`时调用，以检查登录用户名，或使用`pwd.getpwuid(os.getuid())[0]`得到当前有效的用户登录名。

支持：Unix，Windows。

