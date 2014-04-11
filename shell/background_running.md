# 关于后台运行命令

系统运维经常会遇到类似计划任务的东西……

## 1. cron和crontab

### 1.1 cron

这是一个系统进程，通常我们靠cron执行无人值守的脚本。

### 1.2 crontab

`crontab`命令相当于cron进程的用户界面，我们可以用`crontab`命令向cron进程中加入我们想要执行的命令、脚本、程序等。

每一个用户都有一个自己的crontab计划。

`crontab`是由很多条目组成的，每一个`crontab`条目都有如下结构（共六部分）：

<minutes>[0-59] <hours>[0-23] <days>[1-31] <months>[1-12] <weeks>[0-6] <command>

比如：

    # execute myscript.sh at 12:30 everyday:
    30 12 * * * ~/scripts/myscript.sh

    # execute at 0:00 in day 1 and day 15 of every month:
    0 0 1,15 * * ~/scripts/myscript.sh

    # check file permission in every weekends at 3:15:
    3 15 * * 6,0 /bin/find / -type f \( -perm -04000 -o -perm -02000 \) \-exec ls -lAG {} \;

    # execute at every 0 and half between 18 - 23 o'clock:
    0,30 18-23 * * * ~/scripts/myscript.sh

注意，要在命令中使用绝对路径，在脚本中设置环境变量，cron进程仅仅知道全局设置，并不自动读取用户设置。

`crontab` 命令很简单：
`crontab` [`-u` `<username>`] 后接 `-e`、`-l`、`-r`。

`-u`: 可以指定在那个用户的cron列表里执行操作，不加`-u`就是默认当前用户；
`-e`: 编辑指定用户的crontab计划；
`-l`: 列出指定用户的crontab计划内容；
`-r`: 清空指定用户的crontab计划；

注意，用`-e`选项删除所有内容并保存，**不等于**`-r`选项。
使用`-r`后用`-l`检查会发现两者的不同。
求助，在OSX下没找到“crontab文件”在哪…

通常推荐把crontab条目在文件里编辑，完成后再用crontab提交所有条目：

1. 新建文件，起个能够区分用户的名字，如zltcron；
2. 用vi之类的编辑器完成编辑；
3. 使用命令`crontab` `zltcron`完成提交；

这样可以避免在使用`crontab`时`-e`和`-r`离的太近摁错。

## 2. at

这个命令相对`crontab`更简便（与`crontab`不同，`at`执行时会使用当前shell的环境变量，但它仅执行一次），分为两种情况：
提示符模式和命令行模式。

### 2.1 提示符模式：

提示符下时使用control+D结束提示符模式：

    at 12:15 +1day
    echo Hello | mail zealot
    ^D
    job 2 at Mon Apr  7 12:15:00 2014

`at`输出的内容包括挂起任务的id和时间；

`atq`: 查看当前用户挂起的at命令，若用root运行`atq`则查看所有未运行的`atq`命令，输出内容包括挂起任务的id和任务执行时间。

`atrm` `<id>`或`at` `-r` `<id>`: 根据id删除计划（配合`atq`很方便）；

`at`接受的时间格式非常丰富：

    at 1230
    at 12:30
    at 04.15.2014
    at 04.15.14
    at 04/15/2014
    at 04/15/14
    at 04152014
    at 041514
    at now + 15 minutes

### 2.2 脚本模式：

`at` `<datetime>` `-f` `<filepath>`: 执行shell脚本；

## 3. &

当我们执行`find /`之类的命令时，会长时间占据终端，这是我们可以使用`&`命令把它放在后台执行。

`&`命令并不会脱离终端运行，即终端断开时这些程序会默认会被结束。


    find / -iname "*.deny" -print >~/output 2>&1 &

提交后会显示一个进程id，可以用`ps -x`或`ps -ef`命令查看进程，并结合`kill`终止进程。

    ps -x | grep 1437
    1437 ttys001    0:01.26 find / -iname *.deny -print
    1684 ttys001    0:00.00 grep 1437
    ps -ef | grep 1437
    501   770   586   0 11:45上午 ttys001    0:00.00 xargs ls -lAG
    501  1671   586   0  1:47下午 ttys001    0:00.00 grep 770
    kill 1437

## 4. nohup

`nohup`与`&`的区别在于，使用`nohup`的命令不会在会话结束时被结束。

    nohup find / -iname "*.deny" -print >~/output 2>&1 &

如果不手动重定向输出：

    nohup find / -iname "*.deny" -print &
    appending output to nohup.out

`nohup`重定向输出到默认的nohup.out，就在`~/`目录下。

## 5. 附：关于重定向

`0`   stdin

`1`   stdout

`2`   stderr

`>`   truncate

`>>`  append

`/dev/null`带有让一切重定向消失的魔法。

`2>&1` 将错误信息重定向到标准输出。

