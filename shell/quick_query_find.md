# find 命令速查

最近经常用到这个神奇的命令，所以做个速查记录。

## 附：UNIX中的文件类型：

`d` 目录；
`l` 符号链接(指向另一个文件)；
`s` 套接字文件；
`b` 块设备文件；
`c` 字符设备文件；
`p` 命名管道文件；
`-`或`f` 普通文件，或者更准确地说，不属于以上几种类型的文件；

## 通用参数：

### 逻辑运算符：

and: `-a`

or: `-o`

not: `!`

如，在`/usr/bin`下查找名称**不**为`yourname`的文件

    $ find /usr/bin/ \! -iname "yourname"

### 参数`n`：

`+n` 大于n的，如n+1, n+2...

`-n` 小于n的，如n-1, n-2...

`n` 表示正好是n

### 转义字符：`\`

## 文件名，文件属性相关的查找：

`-name` `“<string>”`: 查找文件名匹配`string`的所有文件，字串内可用通配符`*`, `?`, `[]`;

`-lname` `“<string>”`: 查找文件名匹配`string`的所有符号链接文件，字串内可用通配符`*`, `?`, `[]`;

`-iname`, `-ilname`: 以上两个命令的case insensitive版；

`-gid` `n`: 查找属于id为`n`的**用户组**的文件；

`-uid` `n`: 查找属于id为`n`的**用户**的文件；

`-group` `"<string>"`: 查找属于**用户组**名为所给字串的所有的文件；

`-user` `"<string>"`: 查找属于**用户名**为所给字串的所有的文件；

`-empty`: 查找大小为0的目录或文件；

`-path` `"<string>"`: 查找路径名匹配所给字串的所有文件，字串内可用通配符`*`, `?`, `[]`;

`-perm` `[+|-]` `<permission-code>` 查找具有指定权限的文件和目录，权限的表示可以如`+00775`, `-04741`; 

`-`表示被查找文件的每一位**被许可权限**都匹配`permission-code`才返回为真（即匹配比`permission-code`权限更大的权限）；

`+`表示被查找文件只要有一位许可权限匹配即可返回真（即匹配不比`permission-code`权限更大的权限）；

没有`+|-`标记的为绝对匹配（即匹配与`permission-code`权限一致的权限）；

`-size` `n[ckMGTP]` 查找指定文件大小的文件，`n`后面的字符表示单位，缺省为`b`，代表512字节的块；

`-type` `<file-type>[bcdpfls]` 查找类型为的文件，x 为下列字符之一：

如，查找`/usr/bin`下的符号链接文件（Symbolic Link）：

    $ find /usr/bin -type l

`-prune`: 指定忽略查找的目录，与`-depth`共用时会被忽略


## 时间相关的查找

以时间为条件查找

`-amin` `n`: 查找n*分钟*以前被访问过的文件；

`-atime` `n[smhdw]`: 查找`n`时间单位（默认值：天）以前被访问过的文件；

`-cmin` `n`: 查找`n`*分钟*以前**文件状态**被修改过的所有文件。

`-ctime` `n[smhdw]`: 查找`n`*时间单位*（默认天）以前**文件状态**被修改过的文件；

`-mmin` `n`: 查找n*分钟*以前**文件内容**被修改过的所有文件；

`-mtime` `n`: 查找`n`*时间单位*（默认天）以前**文件内容**被修改过的所有文件；

附：`c`表示change，如`chmod`；`m`表示modify，如`echo foo >> file`；`touch`会改变三个时间属性；`ls`的时间是`st_mtime`。

    st_atime
        Time when file data was last accessed. Changed by  the
        following   functions:   creat(),   mknod(),   pipe(),
        utime(2), and read(2).

    st_mtime
        Time when data was last modified. Changed by the  fol-
        lowing  functions:  creat(), mknod(), pipe(), utime(),
        and write(2).

    st_ctime
        Time when file status was last changed. Changed by the
        following   functions:   chmod(),   chown(),  creat(),
        link(2),  mknod(),  pipe(),  unlink(2),  utime(),  and
        write().

## 可执行的操作

`-exec` `<utility>` `[arguments]` `{}`: 对符合条件的文件执行所给的unix `utility` `arguments`命令，而不询问用户是否需要执行该命令。`{}`表示命令的参数即为所找到的每一个文件；命令的末尾必须以`\;`结束；

`-ok` `<utility>` `[arguments]`: 对符合条件的文件执行所给的unix `utility` `arguments`命令，与`exec`不同的是，它会询问用户是否需要执行该命令；

`-ls`: 详细列出所找到的所有文件。

`-print`: 在标准输出设备上显示查找出的文件名。

## find实例：

查找当前目录中所有以main开头的文件，并显示这些文件的内容：

    $ find . -name "main*" -exec more {} \;

删除当前目录下所有一周之内没有被访问过的a .out或*.o文件；

注：`.`表示当前目录；`\(`和`\)`表示括号，使用`\`做转义；`\;`必须在`-exec`后出现，用来结束`-exec`；

find命令在当前目录及其子目录下找到这佯的文件之后，再进行判断，看其最后访问时间是否在7天以前（条件`-atime +7`），若是，则对该文件执行命令`rm` `<filepath>`：

    $ find . \( -name "a.out" -o -name "*.o" \) -atime +7 -exec rm {} \;

查找并列出具有`setuid`权限与`setgid`权限的文件：

    $ find / -type f \( -perm -04000 -o -perm -02000 \) \-exec ls -lg {} \;　　

查找系统中的每一个普通文件，然后使用xargs命令来测试它们分别属于哪类文件

    $ find . -type f -print | xargs file

将找到的结果输出至文件：

    find . -name "my*" -print | xargs echo >> /temp/mylog

在`/usr`中查找文件名为`a*`的文件，并忽略路径`/usr/bin`及`/usr/share`：

    find /usr \( -path "/usr/bin" -o -path "/usr/share" \) -prune -o -iname "a*" -print

