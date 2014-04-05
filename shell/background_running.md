# 关于后台运行命令

系统运维经常会遇到类似计划任务的东西……

## cron和crontab

### cron

这是一个系统进程，通常我们靠cron执行无人值守的脚本。

### crontab

`crontab`命令相当于cron进程的用户界面，我们可以用`crontab`命令向cron进程中加入我们想要执行的命令、脚本、程序等。

`crontab`是由很多条目组成的，每一个`crontab`条目都有如下结构：

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


