# awk 命令

文章的全部干货都来源于**[酷壳]**(http://coolshell.cn/)的**[AWK 简明教程]**(http://coolshell.cn/articles/9070.html)。

本文纯属搬砖。

最近经常用乱七八糟的命令输出大段的东西，所以想学习一下这个`awk`命令，这有点像是一个微型语言。

## 先弄一堆乱七八糟的输出

    netstat -a > output
    cat output | more -n 6
    Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)
    tcp4       0      0  192.168.1.100.62750    114.112.202.20.http    SYN_SENT
    tcp4       0      0  192.168.1.100.62749    113.103.139.251.10684  CLOSE_WAIT
    tcp4       0      0  192.168.1.100.62748    202.150.16.5.cslistene ESTABLISHED
    tcp4       0      0  192.168.1.100.62747    114.112.202.20.http    ESTABLISHED
    tcp4       0      0  192.168.1.100.62746    114.82.234.97.12077    SYN_SENT


## 最简单的选择列，输出第一列`$1`与第四列`$4`

    awk '{print $1, $4}' output
    Proto Local-Address
    tcp4 192.168.1.100.62750
    tcp4 192.168.1.100.62749
    tcp4 192.168.1.100.62748
    tcp4 192.168.1.100.62747

## 像C语言一样格式化输出

    awk '{printf "%-8s %-8s %-8s %-18s %-22s %-15s\n", $1, $2, $3, $4, $5, $6}' output
    Proto    Recv-Q   Send-Q   Local-Address      Foreign-Address        (state)
    tcp4     0        0        192.168.1.100.62750 114.112.202.20.http    SYN_SENT
    tcp4     0        0        192.168.1.100.62749 113.103.139.251.10684  CLOSE_WAIT
    tcp4     0        0        192.168.1.100.62748 202.150.16.5.cslistene ESTABLISHED
    tcp4     0        0        192.168.1.100.62747 114.112.202.20.http    ESTABLISHED

## 过滤信息

取第三列为0且第6列位“LISTEN”的数据：

    awk '$3==0 && $6=="LISTEN"' output
    tcp4       0      0  *.oms                  *.*                    LISTEN
    tcp4       0      0  *.socks                *.*                    LISTEN
    tcp4       0      0  *.58932                *.*                    LISTEN
    tcp4       0      0  localhost.ddi-tcp-1    *.*                    LISTEN
    tcp4       0      0  localhost.51219        *.*                    LISTEN

同时用内建变量`NR`选出表头（第一行）：

    awk '$3==0 && $6=="LISTEN" || NR==1' output
    Proto Recv-Q Send-Q  Local-Address          Foreign-Address        (state)
    tcp4       0      0  *.oms                  *.*                    LISTEN
    tcp4       0      0  *.socks                *.*                    LISTEN
    tcp4       0      0  *.58932                *.*                    LISTEN
    tcp4       0      0  localhost.ddi-tcp-1    *.*                    LISTEN

取第三列大于0的数据：

    awk ' $3>0 {print $0}' output
    Proto Recv-Q Send-Q  Local-Address          Foreign-Address        (state)
    tcp4       0     68  192.168.1.100.62743    123.138.206.178.8948   ESTABLISHED
    tcp4       0     68  192.168.1.100.62720    123.138.54.160.mdqs    ESTABLISHED
    tcp4       0      9  192.168.1.100.62707    180.97.152.55.http-alt FIN_WAIT_1
    tcp4       0     68  192.168.1.100.62695    123.138.206.178.8948   FIN_WAIT_1

取第三列为0且第六列为“LISTEN”的数据，并进行格式输出：

    awk '$3==0 && $6=="LISTEN" || NR==1 {printf "%-20s %-20s %s\n",$4,$5,$6}' output
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

    awk '$3==0 && $6=="ESTABLISHED" || NR==1 {printf "%02s %s %-20s %-20s %s %s", NR, FNR, $4, $5, $6, ORS}' output
    01 1 Local-Address        Foreign-Address      (state)
    04 4 192.168.1.100.62748  202.150.16.5.cslistene ESTABLISHED
    05 5 192.168.1.100.62747  114.112.202.20.http  ESTABLISHED
    07 7 192.168.1.100.62744  114.112.202.20.http  ESTABLISHED
    11 11 192.168.1.100.62739  14.157.50.39.13566   ESTABLISHED

指定字段（列）分隔符

    awk  'BEGIN{FS=":"} {print $1,$3,$6}' /etc/passwd
    nobody -2 /var/empty
    root 0 /var/root
    daemon 1 /var/root
    _uucp 4 /var/spool/uucp
    _taskgated 13 /var/empty

也可以用`-F`指定分隔符

    awk  -F: '{print $1,$3,$6}' /etc/passwd

或指定多种分隔符

    awk  -F '[;:]' '{print $1,$3,$6}' /etc/passwd

指定输出字段（列）分隔符

    awk  -F: '{print $1,$3,$6}' OFS="\t" /etc/passwd
    nobody  -2      /var/empty
    root    0       /var/root
    daemon  1       /var/root
    _uucp   4       /var/spool/uucp
    _taskgated      13      /var/empty

