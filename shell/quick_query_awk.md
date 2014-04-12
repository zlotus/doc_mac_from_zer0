# awk 命令

文章的全部干货都来源于**[酷壳]**(http://coolshell.cn/)的**[AWK 简明教程]**(http://coolshell.cn/articles/9070.html)。

本文纯属搬砖。

最近经常用乱七八糟的命令输出大段的东西，所以想学习一下这个`awk`命令，这有点像是一个微型语言。

## 先弄一堆乱七八糟的输出

    $ netstat -a > output
    $ cat output | more -n 6
    $ Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)
    tcp4       0      0  192.168.1.100.62750    114.112.202.20.http    SYN_SENT
    tcp4       0      0  192.168.1.100.62749    113.103.139.251.10684  CLOSE_WAIT
    tcp4       0      0  192.168.1.100.62748    202.150.16.5.cslistene ESTABLISHED
    tcp4       0      0  192.168.1.100.62747    114.112.202.20.http    ESTABLISHED
    tcp4       0      0  192.168.1.100.62746    114.82.234.97.12077    SYN_SENT


## 最简单的选择列，输出第一列`$1`与第四列`$4`

    $ awk '{print $1, $4}' output
    Proto Local-Address
    tcp4 192.168.1.100.62750
    tcp4 192.168.1.100.62749
    tcp4 192.168.1.100.62748
    tcp4 192.168.1.100.62747

## 像C语言一样格式化输出

    $ awk '{printf "%-8s %-8s %-8s %-18s %-22s %-15s\n", $1, $2, $3, $4, $5, $6}' output
    Proto    Recv-Q   Send-Q   Local-Address      Foreign-Address        (state)
    tcp4     0        0        192.168.1.100.62750 114.112.202.20.http    SYN_SENT
    tcp4     0        0        192.168.1.100.62749 113.103.139.251.10684  CLOSE_WAIT
    tcp4     0        0        192.168.1.100.62748 202.150.16.5.cslistene ESTABLISHED
    tcp4     0        0        192.168.1.100.62747 114.112.202.20.http    ESTABLISHED

## 过滤条目

取第三列为0且第6列位“LISTEN”的数据：

    $ awk '$3==0 && $6=="LISTEN"' output
    tcp4       0      0  *.oms                  *.*                    LISTEN
    tcp4       0      0  *.socks                *.*                    LISTEN
    tcp4       0      0  *.58932                *.*                    LISTEN
    tcp4       0      0  localhost.ddi-tcp-1    *.*                    LISTEN
    tcp4       0      0  localhost.51219        *.*                    LISTEN

同时用内建变量`NR`选出表头（第一行）：

    $ awk '$3==0 && $6=="LISTEN" || NR==1' output
    Proto Recv-Q Send-Q  Local-Address          Foreign-Address        (state)
    tcp4       0      0  *.oms                  *.*                    LISTEN
    tcp4       0      0  *.socks                *.*                    LISTEN
    tcp4       0      0  *.58932                *.*                    LISTEN
    tcp4       0      0  localhost.ddi-tcp-1    *.*                    LISTEN

取第三列大于0的数据：

    $ awk ' $3>0 {print $0}' output
    Proto Recv-Q Send-Q  Local-Address          Foreign-Address        (state)
    tcp4       0     68  192.168.1.100.62743    123.138.206.178.8948   ESTABLISHED
    tcp4       0     68  192.168.1.100.62720    123.138.54.160.mdqs    ESTABLISHED
    tcp4       0      9  192.168.1.100.62707    180.97.152.55.http-alt FIN_WAIT_1
    tcp4       0     68  192.168.1.100.62695    123.138.206.178.8948   FIN_WAIT_1

取第三列为0且第六列为“LISTEN”的数据，并进行格式输出：

    $ awk '$3==0 && $6=="LISTEN" || NR==1 {printf "%-20s %-20s %s\n",$4,$5,$6}' output
    Local-Address        Foreign-Address      (state)
    *.oms                *.*                  LISTEN
    *.socks              *.*                  LISTEN
    *.58932              *.*                  LISTEN
    localhost.ddi-tcp-1
    *.*                  LISTEN

Special Names|Descriptions
:------------|:-----------
$0           |当前记录（行）
$1~$n        |当前行的第n个字段（列）
FS           |输入字段（列）分隔符，默认为space或tab
NF           |当前记录（行）的字段（列）数
NR           |总记录（行）数，多文件记录（行）数累加，不清零
FNR          |当前文件记录（行）数，每文件清零
RS           |输入换行符，默认系统换行符
OFS          |输出字段（列）分隔符，默认为space
ORS          |输出换行符，默认系统换行符
FILENAME     |当前文件名

输出行号

    $ awk '$3==0 && $6=="ESTABLISHED" || NR==1 {printf "%02s %s %-20s %-20s %s %s", NR, FNR, $4, $5, $6, ORS}' output
    01 1 Local-Address        Foreign-Address      (state)
    04 4 192.168.1.100.62748  202.150.16.5.cslistene ESTABLISHED
    05 5 192.168.1.100.62747  114.112.202.20.http  ESTABLISHED
    07 7 192.168.1.100.62744  114.112.202.20.http  ESTABLISHED
    11 11 192.168.1.100.62739  14.157.50.39.13566   ESTABLISHED

指定输入字段（列）分隔符

    $ awk  'BEGIN{FS=":"} {print $1,$3,$6}' /etc/passwd
    nobody -2 /var/empty
    root 0 /var/root
    daemon 1 /var/root
    _uucp 4 /var/spool/uucp
    _taskgated 13 /var/empty

也可以用`-F`指定

    $ awk  -F: '{print $1,$3,$6}' /etc/passwd

或指定多种分隔符

    $ awk  -F '[;:]' '{print $1,$3,$6}' /etc/passwd

指定输出字段（列）分隔符

    $ awk  -F: '{print $1,$3,$6}' OFS="\t" /etc/passwd
    nobody  -2      /var/empty
    root    0       /var/root
    daemon  1       /var/root
    _uucp   4       /var/spool/uucp
    _taskgated      13      /var/empty

## 字符串匹配

`~`表示匹配开始，`//`中是模式，类似正则表达式

匹配第六个字段（列）有“FIN”的记录（行）

    $ awk '$6 ~ /FIN/ || NR==1 {printf "%-4s %-20s %-30s %-10s\n", NR, $4, $5, $6}' output
    1    Local-Address        Foreign-Address                (state)
    21   192.168.1.100.62707  180.97.152.55.http-alt         FIN_WAIT_1
    22   192.168.1.100.62695  123.138.206.178.8948           FIN_WAIT_1
    24   192.168.1.100.62690  218.86.133.247.8368            FIN_WAIT_1
    25   192.168.1.100.62689  14.157.50.39.13566             FIN_WAIT_1

或是有“WAIT”的记录

    $ awk '$6 ~ /WAIT/ || NR==1 {printf "%-4s %-20s %-30s %-10s\n", NR, $4, $5, $6}' output
    1    Local-Address        Foreign-Address                (state)
    3    192.168.1.100.62749  113.103.139.251.10684          CLOSE_WAIT
    21   192.168.1.100.62707  180.97.152.55.http-alt         FIN_WAIT_1
    22   192.168.1.100.62695  123.138.206.178.8948           FIN_WAIT_1
    24   192.168.1.100.62690  218.86.133.247.8368            FIN_WAIT_1

或是直接匹配一整条记录（行）

    $ awk '/LISTEN/' output
    tcp4       0      0  *.oms                  *.*                    LISTEN
    tcp4       0      0  *.socks                *.*                    LISTEN
    tcp4       0      0  *.58932                *.*                    LISTEN
    tcp4       0      0  localhost.ddi-tcp-1    *.*                    LISTEN
    tcp4       0      0  localhost.51219        *.*                    LISTEN

像正则表达式一样，使用`|`表示“或”

    $ awk '$6 ~ /CLOSE|LAST/ {printf "%-4s %-20s %-30s %-10s\n", NR, $4, $5, $6}' output
    3    192.168.1.100.62749  113.103.139.251.10684          CLOSE_WAIT
    10   192.168.1.100.62741  114.82.234.97.12077            LAST_ACK
    13   192.168.1.100.62729  114.82.234.97.12077            LAST_ACK
    16   192.168.1.100.62722  114.82.234.97.12077            LAST_ACK
    19   192.168.1.100.62711  113.103.139.251.10684          LAST_ACK

用`!`取反

    $ awk '$6 !~ /WAIT/ {printf "%-4s %-20s %-30s %-10s\n", NR, $4, $5, $6}' output
    1    Local-Address        Foreign-Address                (state)
    2    192.168.1.100.62750  114.112.202.20.http            SYN_SENT
    4    192.168.1.100.62748  202.150.16.5.cslistene         ESTABLISHED
    5    192.168.1.100.62747  114.112.202.20.http            ESTABLISHED
    6    192.168.1.100.62746  114.82.234.97.12077            SYN_SENT

也可以

    $ awk '!/WAIT/' output
    Proto Recv-Q Send-Q  Local-Address          Foreign-Address        (state)
    tcp4       0      0  192.168.1.100.62750    114.112.202.20.http    SYN_SENT
    tcp4       0      0  192.168.1.100.62748    202.150.16.5.cslistene ESTABLISHED
    tcp4       0      0  192.168.1.100.62747    114.112.202.20.http    ESTABLISHED
    tcp4       0      0  192.168.1.100.62746    114.82.234.97.12077    SYN_SENT

## 拆分文件

按照第六个字段（列）拆分文件

    $ awk 'NR!=1 {print > $6}' output
    $ ls
    CLOSE_WAIT  ESTABLISHED FIN_WAIT_1  FIN_WAIT_2  LAST_ACK    LISTEN      SYN_SENT    output

也可以指定要输出的字段（列）

    $ awk 'NR!=1 {print $4, $5 > $6}' output

或是更加复杂，注意`quote>`说明了`awk`是一个脚本解释器

    $ awk 'NR!=1{if($6 ~ /TIME|ESTABLISHED/) print > "1.txt";
    quote> else if($6 ~ /LISTEN/) print > "2.txt";
    quote> else print > "3.txt" }' output

    $ ls ?.txt
    1.txt  2.txt  3.txt

## 统计

统计当前目录下.py, .pyc, .pyo文件的总大小

    $ ls -l *.py *.pyc *.pyo | awk '{sum+=$5} END {print sum}'
    11609730

或是按照第六个字段（列）统计，类似GROUP BY，其中`a`类似Python中的dict（键-值）

    $ awk 'NR!=1 {a[$6]++;} END {for (i in a) print i ", " a[i];}' output
    LISTEN, 12
    FIN_WAIT_1, 51
    FIN_WAIT_2, 23
    SYN_SENT, 3
    LAST_ACK, 21
    CLOSE_WAIT, 5
    ESTABLISHED, 20

统计每个用户的进程所占用的内存信息

    $ aux | awk 'NR!=1 {a[$1]+=$6;} END { for(i in a) print i ", " a[i]"KB";}'
    root, 718624KB
    _usbmuxd, 2768KB
    _locationd, 7780KB
    _mdnsresponder, 3760KB
    zealot, 3800464KB



