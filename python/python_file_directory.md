# 文件操作

写服务器脚本时，经常会遇到需要对大量文件进行读、写、拷贝、删除操作的情况。

## shutil

`shutil`是一个非常方便的文件操作模块，提供文件/目录的批量拷贝、删除操作。模块提供高级文件操作，如果需要对文件进行系统级访问的话，也可以使用`os`模块。

### shutil.copyfileobj(fsrc, fdst[, length])

这是一个比较初级的函数：

```
def copyfileobj(fsrc, fdst, length=16*1024):
    """copy data from file-like object fsrc to file-like object fdst"""
    while 1:
        buf = fsrc.read(length)
        if not buf:
            break
        fdst.write(buf)
```

打开*fsrc*文件，每次读*length*KB内容，写入*fdst*文件。若*length*为负，则没有缓冲区限制，这样有可能导致内存使用不可控。

注意，这个函数没有在使用文件后关闭文件对象。它是这个模块中的基础拷贝函数，不推荐直接使用。

### shutil.copyfile(src, dst, *, follow_symlinks=True)

这个函数会将*src*文件内容拷贝至*dst*文件。

如果*src*是软链接，且*follow_symlinks*为`False`，则会新建*dst*软链接并指向原软链接的位置，注意，此时*dst*为软链接；

如果*src*是软链接，且*follow_symlinks*为`True`时，则会将原软链接所指文件的内容拷贝至*dst*，注意，此时*dst*是文件。

特别注意：

* 这个函数不会拷贝文件的元数据；
* OSX下的替身（Alias）是文件，不是链接；
* 这个函数只能用来拷贝文件或软链接。

下面的列表中：

* 第一个文件是Finder中“制作替身”命令生成的；
* 第二个文件是原始文件；
* 第三个是`ln`命令生成的硬链接；
* 第四个是`shutil.copyfile()`复制*symlink_info.php*生成的软连接；
* 第五个是`ln`命令生成的软链接；

```
➜  firstphp  ls -lAG | grep "info\.php$"
-rw-r--r--@  1 zealot  staff  171300  5 10 20:25 alias_info.php
-rw-r--r--   2 zealot  staff     104  5  7 22:47 info.php
-rw-r--r--   2 zealot  staff     104  5  7 22:47 link_info.php
lrwxr-xr-x   1 zealot  staff       8  5 10 20:35 py_symlink_info.php -> info.php
lrwxr-xr-x   1 zealot  staff       8  5 10 20:26 symlink_info.php -> info.php
```

### shutil.copymode(src, dst, *, follow_symlinks=True)

将*src*的权限码拷贝至*dst*，不管文件内容、属主及属组。

如果*src*和*dst*都是软链接，且*follow_symlinks*为`False`，函数将尝试设置*dst*本身的权限（而不是其所指文件的权限）。

注意，这个函数并不是对所有平台生效的，如果函数不能修改该平台上软链接的权限，却被指定要求修改软链接权限时，函数将不做任何处理直接返回。

### shutil.copystat(src, dst, *, follow_symlinks=True)

函数将复制权限码、最后访问时间、最后修改时间（st_mtime）以及文件标识，不管文件内容、属主及属组。

如果*src*和*dst*都是软链接，且*follow_symlinks*为`False`，函数将读取软链接本身的信息（而不是去读取所指文件的信息），并将信息写入*dst*软链接。

注意，这个函数式平台相关的：

* 如果`os.chmod in os.supports_follow_symlinks`返回`True`，`copystat()`就可以修改软链接的权限码；
* 如果`os.utime in os.supports_follow_symlinks`返回`True`，`copystat()`就可以修改文件的最后访问时间和修改时间（st_mtime）；
* 如果`os.chflags in os.supports_follow_symlinks`返回`True`，`copystat()`就可以修改软链接的文件标识（不是所有的平台都有`os.chflags`）。

所以，如果某些拷贝遇到平台不支持的情况，`copystat()`会尽可能的拷贝能拷贝的信息，而从不返回某项功能拷贝失败的信息。

### shutil.copy(src, dst, *, follow_symlinks=True)

主要使用了`copyfile()`拷贝内容，使用`copymode()`拷贝元数据，所以这个函数复制的元数据只有文件的权限码。

如果*src*是软链接，且*follow_symlinks*是`False`，则*dst*将被创建成软链接；

如果*src*是日记里，且*follow_symlinks*是`True`，则*dst*将被创建为*src*所指向文件的副本文件。

```
def copy(src, dst, *, follow_symlinks=True):
    """Copy data and mode bits ("cp src dst"). Return the file's destination.

    The destination may be a directory.

    If follow_symlinks is false, symlinks won't be followed. This
    resembles GNU's "cp -P src dst".

    If source and destination are the same file, a SameFileError will be
    raised.

    """
    if os.path.isdir(dst):
        dst = os.path.join(dst, os.path.basename(src))
    copyfile(src, dst, follow_symlinks=follow_symlinks)
    copymode(src, dst, follow_symlinks=follow_symlinks)
    return dst
```

像注释里写的那样，这个函数类似`cp src dst`命令。

### shutil.copy2(src, dst, *, follow_symlinks=True)

主要使用了`copyfile()`拷贝内容，使用`copystat()`拷贝元数据，所以这个函数复制的元数据包含文件权限码、最后访问时间、最后修改时间（st_mtime）以及文件标识，而不复制属主及属组信息。

When follow_symlinks is false, and src is a symbolic link, copy2() attempts to copy all metadata from the src symbolic link to the newly-created dst symbolic link. However, this functionality is not available on all platforms. On platforms where some or all of this functionality is unavailable, copy2() will preserve all the metadata it can; copy2() never returns failure.

如果*src*是软链接，且*follow_symlinks*是`False`，则函数会将*dst*创建为软链接，并将*src*软链接的元数据复制给*dst*软链接本身；

如果*src*是日记里，且*follow_symlinks*是`True`，则*dst*将被创建为*src*所指向文件的副本文件。


```
def copy2(src, dst, *, follow_symlinks=True):
    """Copy data and all stat info ("cp -p src dst"). Return the file's
    destination."

    The destination may be a directory.

    If follow_symlinks is false, symlinks won't be followed. This
    resembles GNU's "cp -P src dst".

    """
    if os.path.isdir(dst):
        dst = os.path.join(dst, os.path.basename(src))
    copyfile(src, dst, follow_symlinks=follow_symlinks)
    copystat(src, dst, follow_symlinks=follow_symlinks)
```

像注释里写的那样，这个函数类似`cp src dst`命令。

### shutil.ignore_patterns(*patterns)

返回一个`set`对象，包含了需要忽略的文件名、文件夹。这个函数是配合`copytree()`使用的。

*patterns*使用的文件名称匹配模式，为`fnmatch`模块提供的函数，使用Unix文件名模式匹配。

Pattern|Meaning
-------|-------
*      |matches everything
?      |matches any single character
[seq]  |matches any character in seq
[!seq] |matches any character not in seq

### shutil.copytree(src, dst, symlinks=False, ignore=None, copy_function=copy2, ignore_dangling_symlinks=False)

这是`shutil`模块中最常用的函数之一，用来递归的拷贝路径下的所有（如果可能的话）文件及文件夹。

这个函数很简单，但是挺值得拓展的：

```
def copytree(src, dst, symlinks=False, ignore=None, copy_function=copy2,
             ignore_dangling_symlinks=False):

    names = os.listdir(src)
    if ignore is not None:
        ignored_names = ignore(src, names)
    else:
        ignored_names = set()

    os.makedirs(dst)
    errors = []
    for name in names:
        if name in ignored_names:
            continue
        srcname = os.path.join(src, name)
        dstname = os.path.join(dst, name)
        try:
            if os.path.islink(srcname):
                linkto = os.readlink(srcname)
                if symlinks:
                    os.symlink(linkto, dstname)
                    copystat(srcname, dstname, follow_symlinks=not symlinks)                
                else:
                    if not os.path.exists(linkto) and ignore_dangling_symlinks:
                        continue
                    # 1. put an file ignore filter function before the `copy_function()` 
                    # can get a more specific ignore mechanism
                    copy_function(srcname, dstname)
            elif os.path.isdir(srcname):
                copytree(srcname, dstname, symlinks, ignore, copy_function)
            else:
                copy_function(srcname, dstname)
            # XXX What about devices, sockets etc.?
        except Error as err:
            errors.extend(err.args[0])
        except OSError as why:
            errors.append((srcname, dstname, str(why)))
    try:
        copystat(src, dst)
    except OSError as why:
        if why.winerror is None:
            errors.append((src, dst, str(why)))
    if errors:
        raise Error(errors)
    return dst
```

在这个函数中注释1的位置，如果加上判断条件在执行`copy_function()`，就可以精确控制每一个文件的拷贝。可以在这里传入一个callable对象（如参数中加入`ignore_manager=func`）执行if-else判断，比如可以实现类似ACL的功能来控制拷贝的文件，这在编写服务器维护脚本的时候会比较有用。

在注释2的位置，其实可以实现很多功能，比如判断是device后做相应的操作，判断是socket做另一种动作，这样就不止限于文件协议的拷贝了。

关于`ignore_patterns`一般是这样使用的：

```
from shutil import copytree, ignore_patterns

copytree(source, destination, ignore=ignore_patterns('*.pyc', 'tmp*'))
```

一个使用*ignore*参数写拷贝的日志的例子：

```
from shutil import copytree
import logging

def _logpath(path, names):
    logging.info('Working in %s' % path)
    return []   # nothing will be ignored

copytree(source, destination, ignore=_logpath)
```

使用这个函数时需要注意：

* *dst*不能已存在，否则会`raise OSError`
* 如果*symlinks*为`True`，则源文件树中的软链接会以软链接的形式拷贝至目的文件树中，而软链接的元数据会尽可能保留（默认使用`copy2()`进行拷贝）；若为`False`则会拷贝连接所指文件到目的文件树；
* *symlinks*设置为`False`时，无效的软链接的拷贝异常信息将会写入函数的异常列表，待函数结束时一次性返回异常；在这种情况下，可以将*ignore_dangling_symlinks*设置为`True`以屏蔽这种软链接失效的异常信息；
* *ignore*参数必须是一个可调用对象，接受两个参数：一个路径字符串，一个该路径下一级子文件与文件夹列表（这个列表其实是`os.listdir()`作用在前一个参数上的结果），要求返回一个需要被忽略的文件及文件夹名称列表（因为`copytree()`是递归调用的，所以列表中的文件及文件夹名称应该是相对于当前目录的名称，即相对路径）；
* *copy_function*接受是一个可调用对象，接受两个参数：源路径，目的路径；
* 拷贝途中出现的所有异常会被添加至异常列表，待函数结束时一次性返回；
