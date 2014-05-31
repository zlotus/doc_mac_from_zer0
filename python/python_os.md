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

### os.getpgid(pid)

返回ID为*pid*的进程所属进程组的ID。如果*pid*为0，则返回当前进程所属进程组的ID。

支持：Unix。

### os.getpgrp()

返回当前进程所属的进程组ID。

支持：Unix。

### os.getpid()

放回当前进程ID。

支持：Unix，Windows。

### os.getppid()

返回父进程ID。在Unix上，当父进程退出时，返回值为当前进程的一个初始化进程的ID；而在Windows上仍然返回其父进程原有的ID，不过此时这个ID可能意见被其他进程占用了。

支持：Unix，Windows。

### os.getpriority(which, who)

获得程序的调度优先级。*which*的值应是`PRIO_PROCESS`, `PRIO_PGRP`, `PRIO_USER`中的一个，*who*是相对于*which*（`PRIO_PROCESS`的进程标识符，`PRIO_PGRP`的进程组标识符，`PRIO_USER`的用户ID）的。将*who*置为0表示调用的进程，调用的进程所属的进程组，或调用进程的用户ID。

支持：Unix。

### os.PRIO_PROCESS
### os.PRIO_PGRP
### os.PRIO_USER

为`getpriority()`和`setpriority()`提供参数。

支持：Unix。

### os.getresuid()

以元组形式返回当前进程的实际组ID（rgid）、有效组ID（egid）和设置组ID（sgid）。

支持：Unix。

### os.getuid()

返回当前进程的用户ID。

支持：Unix。

### os.initgroups(username, gid)

根据用户*username*在的所有组，以及*gid*指定的组，调用系统的initgroups()函数以初始化组访问列表。

### os.putenv(key, value)

将名为*key*的环境变量值置为*value*。此修改会影响使用`os.system()`, `os.popen()`, `os.fork()`, `os.execv()`发起的子进程。

支持：多数Unix，Windows。

注意：在某些环境中（包括FreeBSD和Mac OS X），直接修改`environ`属性可能会导致内存泄露。详情参见操作系统的`putenv`命令或函数。

如果操作系统支持`putenv()`，对`os.environ`的赋值会调用`putenv()`去修改对应的环境变量。但是，直接使用`putenv()`修改环境变量不会更新`os.environ`。所以，更推荐使用`os.environ`给环境变量赋值。

### os.setegid(egid)

设置当前进程的有效组ID。

支持：Unix。

### os.seteuid(euid)

设置当前进程的有效用户ID。

支持：Unix。

### os.setgid(gid)

设置当前进程组ID。

支持：Unix。

### os.setgroups(groups)

将当前进程关联的附加组置为*groups*。*groups*必须是一个序列，其中的每个元素都是组ID的整数。通常只有超级用户可以使用此函数。

支持：Unix。

注意：对于Mac OS X，*groups*的长度可能会受到系统最大有效组ID的限制，通常为16。详情参见`getgroups()`的说明文档。

### os.setpgrp()

根据系统的支持（如果支持的话），调用系统的`setpgrp()`或`setpgrp(0, 0)`函数。语法参见`man`命令。

支持：Unix。

### os.setpgid(pid, pgrp)

调用系统的`setpgid()`函数，ID为`pid`的进程所属进程组的ID置为*pgrp*。语法参见`man`命令。

支持：Unix。

### os.setpriority(which, who, priority)

设置程序的调度优先级。*which*的值应是`PRIO_PROCESS`, `PRIO_PGRP`, `PRIO_USER`中的一个，*who*是相对于*which*（`PRIO_PROCESS`的进程标识符，`PRIO_PGRP`的进程组标识符，`PRIO_USER`的用户ID）的。将*who*置为0表示调用的进程，调用的进程所属的进程组，或调用进程的用户ID。*priority*在-20到19取值，默认为0，值越小越容易调度。

支持：Unix。

### os.setregid(rgid, egid)

设置当前进程的实际组ID和有效组ID。

支持：Unix。

### os.setresgid(rgid, egid, sgid)

设置当前进程的实际组ID、有效组ID和设置组ID。

支持：Unix。

### os.setresuid(ruid, euid, suid)

设置当前进程的实际用户ID、有效用户ID和设置用户ID。

支持：Unix。

### os.setreuid(ruid, euid)

设置当前进程的实际用户ID和有效用户ID。

支持：Unix。

### os.getsid(pid)

调用系统的`getsid()`。语法参见`man`命令。

支持：Unix。

### os.setsid()

调用系统的`setsid()`。语法参见`man`命令。

支持：Unix。

### os.setuid(uid)

设置当前进程的用户ID。

支持：Unix。

### os.strerror(code)

返回错误代码为`code`的错误信息。当提供未知错误代码时，平台的`strerror()`返回`NULL`，抛出`ValueError`异常。

支持：Unix，Windows。

### os.supports_bytes_environ

如果原生系统环境为`bytes`类型则返回`True`。（如在Windows上为`False`）

### os.umask(mask)

设置当前umask码，并返回原来的umask码。

支持：Unix，Windows。

### os.uname()

返回操作系统的标识信息。返回值有五个属性：

* `sysname` - 操作系统名；
* `nodename` - 网络节点名；
* `release` - 系统发布版本号；
* `version` - 操作系统版本；
* `machine` - 硬件标识；

为了保证向后兼容性，返回值是一个可迭代的、行为类似元组的、元素按顺序分别是`sysname`, `nodename`, `release`, `version`, `machine`的对象。

一些系统会截断`nodename`，返回8个字符或首个词组。推荐使用`socket.gethostname()`或`socket.gethostbyaddr(socket.gethostname())`获得主机名。

支持：多数Unix。

### os.unsetenv(key)

释放（删除）名为*key*的环境变量，这会影响到使用`os.system()`, `os.popen()`, `fork()`, `execv()`发起的子进程。

当系统支持`unsetenv()`时，删除`os.environ`中的项会自动调用`unsetenv()`以删除相应的环境变量。但是，直接调用`unsetenv()`删除环境变量不会更新`os.environ`。所以，更推荐删除`os.environ`中的项。

支持：多数Unix，Windows。

## 创建文件对象

此函数会创建新的文件对象（见`os.open()`，用以打开文件描述符）。

### os.fdopen(fd, *args, **kwargs)

返回一个与文件描述符*fd*关联的打开的文件对象。这是内建函数`open()`的别名，他们接受相同的参数，唯一的区别是`fdopen()`的第一个参数必须是整数。

## 文件描述符操作

以下的函数通过文件描述符操作I/O流。

文件描述符，是一个较小的整数，关联着当前进程中打开的文件。标准输入（stdin）的文件描述符通常为0，标准输出（stdout）为1，标准错误（stderr）为2，进程接下来打开的文件将获得3、4、5等文件描述符。“文件描述符”这个名称可能会误导用户，因为在Unix中，sockets和pipes也是通过文件描述符引用的。

`io.IOBase.fileno()`方法可以用来在需要的时候获取文件描述符。值得注意的是，如果绕过文件对象的方法，直接使用文件描述符，可能会忽略掉内部缓冲数据。

### os.close(fd)

关闭文件描述符*fd*。

支持：Unix，Windows。

注意：此函数用于服务底层I/O，用来关闭诸如`os.open()`或`pipe()`等函数返回的文件描述符。如果要关闭类似内建函数`open()`或`os.popen()`和`os.fdopen()`等函数打开的文件对象时，请使用文件对象自己的`close()`方法。

### os.closerange(fd_low, fd_high)

关闭介于*fd_low*（包括）与*fd_high*（不包括）之间的所有文件描述符，并忽略异常，相当于（当然，比下面的代码快的多）：

```
for fd in range(fd_low, fd_high):
    try:
        os.close(fd)
    except OSError:
        pass
```

支持：Unix，Windows。

### os.device_encoding(fd)

如果*fd*关联的设备连接到一个终端，则返回一个描述设备编码的字符串，否则返回`None`。

### os.dup(fd)

返回文件描述符*fd*的副本，新的文件描述符是不可继承的。

在Windows上，如果复制标准流（stdin:0, stdout:1, stderr:2），新的文件描述符是可继承的。

支持：Unix，Windows。

### os.dup2(fd, fd2, inheritable=True)

将文件描述符*fd*复制到*fd2*，并会根据需要关闭后者。*fd2*默认是可继承的，除非将`inheritable`置为`False`。

支持：Unix，Windows。

### os.fchmod(fd, mode)

将给定文件*fd*的模式设置为*mode*。可用的*mode*参见`os.chmod()`的说明。对于Python3.3，相当于`os.chmod(fd, mode)`。

支持：Unix。

### os.fchown(fd, uid, gid)

将文件*fd*的属主码置为*uid*，属组码置为*gid*。若不需要改变文件某个id属性，置为-1即可。参见`os.chown()`。对于Python3.3，相当于`os.chown(fd, uid, gid)`。

支持：Unix。

### os.fdatasync(fd)

强制将与文件描述符*fd*关联的文件写入磁盘，但并不强制更新文件的元数据。

支持：Unix。

注意：MacOS不支持此函数。

### os.fpathconf(fd, name)

返回与被打开文件相关的的系统配置信息。用*name*参数指定取回的配置值，参数应为系统定义值，可能是字符串形式，这些系统定义值可以在某些标准（POSIX.1, Unix 95, Unix 98等）里找到。部分平台也支持额外的这类配置值。这些操作系统相关的配置值在`pathconf_names`目录中给出。如果有配置值并不在目录的映射中，则可以给传给*name*一个整型。

如果*name*是字符串，且未定义在系统中，则会抛出`ValueError`；如果*name*不被操作系统支持，即使定义在`pathconf_names`中，也会抛出`OSError`，并附带一个`errno.EINVAL`的错误代码。

对于Python3.3，相当于`os.pathconf(fd, name)`。

支持：Unix。

### os.fstat(fd)

返回文件描述符*fd*的状态，类似`os.stat()`。对于Python3.3，相当于`os.stat(fd)`。

支持：Unix，Windows。

### os.fstatvfs(fd)

返回包含与文件描述符*fd*相关联的文件的文件系统信息，类似`os.statvfs()`。对于Python3.3，相当于`os.statvfs(fd)`。

支持：Unix。

### os.fsync(fd)

强制将与文件描述符*fd*关联的文件写入磁盘。在Unix上，会调用系统的`fsync()`，在Windows上则是`_commit()`。

如果你操作的是一个有数据在缓冲区的文件对象*f*，首先应使用`f.flush()`，而后再使用`os.fsync(f.fileno())`，这样可以保证与文件对象*f*关联的缓冲区数据都被写入磁盘。

支持：Unix，Windows。

### os.ftruncate(fd, length)

截断文件描述符*fd*关联的文件，使其不超过*length*个字节。对于Python3.3，相当于`os.truncate(fd, length`。

支持：Unix。

### os.isatty(fd)

如果文件描述符*fd*关联的文件在类tty设备中打开，返回`True`，否则返回`False`。

### os.lockf(fd, cmd, len)

在一个打开的文件描述符*fd*上添加、测试或删除一个POSIX锁。*cmd*使用`F_LOCK`, `F_TLOCK`, `F_ULOCK`, `F_TEST`中的一个指定操作类型，*len*指定锁定的部位。

支持：Unix。

### os.F_LOCK
### os.F_TLOCK
### os.F_ULOCK
### os.F_TEST

指定`lockf()`的动作。

### os.lseek(fd, pos, how)

将文件描述符*fd*的游标移动至*pos*，移动的动作设置：`SEEK_SET`或0，表示相对于文件开头移动；`SEEK_CUR`或1，表示相对于当前游标的位置移动；`SEEK_END`或2，表示相对于文件结尾移动。动作结束后以字节形式返回当前游标相对于文件开头的位置。

支持：Unix，Windows。

### os.SEEK_SET
### os.SEEK_CUR
### os.SEEK_END

函数`lseek()`的参数，其值分别为0, 1, 2。

支持：Unix，Windows。

某些操作系统还支持额外的类似`os.SEEK_HOLE`和`os.SEEK_DATA`的操作。

