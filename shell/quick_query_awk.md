# awk 命令速查

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

# awk 命令详解

## 调用awk

如果在命令行中使用awk，通常型为：

```
$ ￼￼awk -F field-separator 'commands' input-files
```

`-F field-separator`为可选项，用于指定域分隔符，默认为空格，如果需要处理如`','`、`':'`、`';'`之类的字符做分隔符时，可以设置该值，如使用`':'`的情形：

```
$ awk -F: 'commands' input-files
```

也可以将awk命令写在文件中，作为脚本调用，如调用名为awk-script-file的脚本文件：

```
$ awk -f awk-script-file input-files
```

`-f`选项用于指定脚本文件名。

## awk脚本

### 模式和动作

awk脚本由各种**动作**和**模式**组成。

awk使用`-F`命令指定的分隔符（不指定时使用空格）分离记录中的域，直到发现新一行。这个动作将一直持续到文件结束。

awk命令可以有许多语句，语句中的**模式**用于控制**动作**的触发条件。

**模式**可以是任何条件语句、复合语句或正则表达式。模式包括了两个特殊字段，`BEGIN`和`END`。`BEGIN`语句使用在awk的任何文本浏览动作之前，之后awk文本浏览动作依据输入文件开始执行，`END`语句用来用来在awk完成文本浏览动作后打印输出文本总数及结尾状态标识。

**动作**在`{}`内声明，通常为打印动作，或是诸如条件语句或循环语句。如果不指名**动作**，awk将打印所有浏览记录。

### 记录和域

awk会将读取的一条记录用`-F`指定的分隔符分为多个域，并依次将域命名为`$1`, `$2`, `$3`等，这些域名的使用类似于语言中变量名的使用。

`{print $1, $3}`就表示打印第1域和第3域，而`$0`表示所有的域，所以`{print $0}`表示打印所有域。

上面的在`{}`中的`print`就是一个awk动作。

假设有数据文件grade.txt，文件分隔符为`TAB`（即`'\t'`）：

```
$ cat grade.txt
M.Transley	05/99	48311	Green	8	40	44
J.Lulu	06/99	48317	green	9	24	26
P.Bunny	02/99	48	Yellow	12	35	28
J.Troll	07/99	4842	Brown-3	12	26	26
L.Transly	05/99	4712	Brown-2	12	30	28
```

### 常用的输入输出

使用命令行时通常我们将结果重定向到文件中：

```
$ awk -F\t '{print $1, $3}' grade.txt > wow
```

或是使用`tee`，在将结果输出至标准输出的同时，写入文件：

```
$ awk -F\t '{print $1, $3}' grade.txt | tee wow
```

如果是脚本，在输入上我们可以使用文件：

```
$ belt.awk grade.txt
```

或是使用标准输入：

```
$ belt.awk < grade.txt
```

或是管道：

```
$ grade.txt | belt.awk
```

### 打印头尾信息

使用`BEGIN`打印表头，为了对齐使用了制表符`\t`：

```
$ awk 'BEGIN {print "Name\tBelt\n--------------------"}
quote> {print $1"\t"$4}' grade.txt
Name	Belt
--------------------
M.Transley	Green
J.Lulu	green
P.Bunny	Yellow
J.Troll	Brown-3
L.Transly	Brown-2
```

使用`END`打印表尾：

```
$ awk 'BEGIN {print "Name\n----------"} {print $1} END {print "----------\ntotle: " NR}' grade.txt
Name
----------
M.Transley
J.Lulu
P.Bunny
J.Troll
L.Transly
----------
totle: 5
```

### 条件操作

操作符        |描述
:------------|:-----------
`<`          |小于
`<=`         |小于等于
`==`         |等于
`>`          |大于
`>=`         |大于等于
`~`          |匹配正则表达式
`!~`         |不匹配正则表达式

* 使用正则表达式匹配域时，用`~`接`/Regular_Express/`，如果使用`if`语句则需要放在`()`中。示例，为第4域匹配正则表达式，输出匹配的记录：

```
$ awk '{if($4~/Brown/) print $0}' grade.txt
J.Troll	07/99	4842	Brown-3	12	26	26
L.Transly	05/99	4712	Brown-2	12	30	28
```

上例中不使用`if`也可以，不指定**动作**时awk默认输出整条记录：

```
$ awk '$4~/Brown/' grade.txt
J.Troll	07/99	4842	Brown-3	12	26	26
L.Transly	05/99	4712	Brown-2	12	30	28
```

* 使用正则表达式模糊匹配域：

```
$ awk '{if($3~/48/) print $0}' grade.txt
M.Transley	05/99	48311	Green	8	40	44
J.Lulu	06/99	48317	green	9	24	26
P.Bunny	02/99	48	Yellow	12	35	28
J.Troll	07/99	4842	Brown-3	12	26	26
```

或

```
$ awk '$3~/48/ {print $0}' grade.txt
M.Transley	05/99	48311	Green	8	40	44
J.Lulu	06/99	48317	green	9	24	26
P.Bunny	02/99	48	Yellow	12	35	28
J.Troll	07/99	4842	Brown-3	12	26	26
```

* 使用`==`精确匹配：

```
$ awk '$3==48 {print $0}' grade.txt
P.Bunny	02/99	48	Yellow	12	35	28
```

或

```
$ awk '{if($3==48) print $0}' grade.txt
P.Bunny	02/99	48	Yellow	12	35	28
```

* 使用`!~`的正则表达式：

```
$ awk '$0 !~ /Brown/' grade.txt
M.Transley	05/99	48311	Green	8	40	44
J.Lulu	06/99	48317	green	9	24	26
P.Bunny	02/99	48	Yellow	12	35	28
```

配合`if`使用`!~`：

```
$ awk '{if($4!~/Brown/) print $1, $4}' grade.txt
M.Transley Green
J.Lulu green
P.Bunny Yellow
```

* 比较：

小于：

```
$ awk '{if($6<$7) print $1" try better at next comp"}' grade.txt
M.Transley try better at next comp
J.Lulu try better at next comp
```

小于等于：

```
$ awk '{if($6<=$7) print $1}' grade.txt
M.Transley
J.Lulu
J.Troll
```

大于：

```
$ awk '{if($6>$7) print $1}' grade.txt
P.Bunny
L.Transly
```

* 常见的正则表达式功能：

匹配大小写，使用`[]`匹配字符：

```
$ awk '/[Gg]reen/' grade.txt
M.Transley	05/99	48311	Green	8	40	44
J.Lulu	06/99	48317	green	9	24	26
```

通配符，`.`：

```
$ awk '$1 ~ /^....a/' grade.txt
M.Transley	05/99	48311	Green	8	40	44
L.Transly	05/99	4712	Brown-2	12	30	28
```

逻辑或，`|`：

```
$ awk '$4 ~ /(Yellow|Brown)/' grade.txt
P.Bunny	02/99	48	Yellow	12	35	28
J.Troll	07/99	4842	Brown-3	12	26	26
L.Transly	05/99	4712	Brown-2	12	30	28
```

行首`^`：

```
$ awk '/^J/' grade.txt
J.Lulu	06/99	48317	green	9	24	26
J.Troll	07/99	4842	Brown-3	12	26	26
```

行尾`$`:

```
$ awk '/28$/' grade.txt
P.Bunny	02/99	48	Yellow	12	35	28
L.Transly	05/99	4712	Brown-2	12	30	28
```

### 复合逻辑

* 逻辑与，`&&`：

```
$ awk '{if ($1=="P.Bunny" && $4=="Yellow") print $0}' grade.txt
P.Bunny	02/99	48	Yellow	12	35	28
```

* 逻辑或，`||`：

```
$ awk '{if ($4=="Yellow" || $4~/Brown/) print $0}' grade.txt
P.Bunny	02/99	48	Yellow	12	35	28
J.Troll	07/99	4842	Brown-3	12	26	26
L.Transly	05/99	4712	Brown-2	12	30	28
```

* 逻辑否，`!`：

```
$ awk '$4 != "Brown" {print $1, $4}' grade.txt
M.Transley Green
J.Lulu green
P.Bunny Yellow
J.Troll Brown-3
L.Transly Brown-2
```

### 内置变量的示例

* 已读取的记录数，`NR`：

```
$ awk '{print} END {print "total: "NR}' grade.txt
M.Transley	05/99	48311	Green	8	40	44
J.Lulu	06/99	48317	green	9	24	26
P.Bunny	02/99	48	Yellow	12	35	28
J.Troll	07/99	4842	Brown-3	12	26	26
L.Transly	05/99	4712	Brown-2	12	30	28
total: 5
```

使用`NR`检查保证文件记录数大于0：

```
$ awk '{if (NR>0 && $4~/Brown/) print $0}' grade.txt
J.Troll	07/99	4842	Brown-3	12	26	26
L.Transly	05/99	4712	Brown-2	12	30	28
```

* 当前记录的域数，`NF`：

```
$ awk '{print NF, NR, $0} END {print FILENAME}' grade.txt
7 1 M.Transley	05/99	48311	Green	8	40	44
7 2 J.Lulu	06/99	48317	green	9	24	26
7 3 P.Bunny	02/99	48	Yellow	12	35	28
7 4 J.Troll	07/99	4842	Brown-3	12	26	26
7 5 L.Transly	05/99	4712	Brown-2	12	30	28
grade.txt
```

显示当前目录名：

```
$ pwd
/Users/zealot
$ echo $PWD | awk -F/ '{print $NF}'
zealot
```

显示文件名：

```
$ echo "/Users/zealot/grade.txt" | awk -F/ '{print $NF}'
grade.txt
```
