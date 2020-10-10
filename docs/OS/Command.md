## Linux 体系结构

- 体系结构主要分为**用户态**(用户上层活动)和**内核态**
- 内核（Kernel）：**本质**上是一段管理计算机硬件设备的**程序**
- 系统调用：内核的访问接口，是一种能再简化的操作。
- 公用函数库：系统调用的组合拳
- **`Shell`: 命令解释器，可编程**

![Linux 体系结构](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWctbXkuY3Nkbi5uZXQvdXBsb2Fkcy8yMDEyMTAvMjUvMTM1MTE1OTQ2NF83ODE0LnBuZw?x-oss-process=image/format,png)

## 常用命令

> [39条常见的Linux系统简单面试题](https://zhuanlan.zhihu.com/p/32250942)
>
> [Linux 工具参考](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/ps.html)

### 物理 CPU 及其核数

```shell
$ cat /proc/cpuinfo | grep -c 'physical id'
$ cat /proc/cpuinfo | grep -c 'processor
```

### 系统负载

> load average：平均负载，三个数值分别表示 1min，5min，15min 内系统的平均负载，及平均任务数。

```shell
$ w

 18:48:49 up 22:19,  3 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    210.30.106.82    18:46    1.00s  0.45s  0.00s w

$ uptime
 18:49:36 up 22:20,  3 users,  load average: 0.00, 0.01, 0.05

$ top # 实时监控系统状态

$ iostat -x 1 10 # 监控IO, -x表示显示所有参数信息, 1表示每秒监控一次, 10 表示工监控10次

$ vmstat # 获得有关进程、虚存、页面交换空间及CPU活动的信息

$ free -m # 查看系统内存的使用情况, -m参数表示按照兆字节展示

$ sar -n DEV 1 # 查看网络设备的吞吐率

$ mpstat -P ALL 1 # 显示每个CPU的占用情况, 如果CPU占用率特别高, 可能是单线程应用程序引起

$ dmesg | tail # 输出系统日志的最后10行
```

### vmstat

> Report virtual memory statistics

```shell
$ vmstat
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0      0 1151012  58008 580968    0    0     6     4  193  484  0  0 99  0  0
 
r: running，正在运行的任务数
b: blocked，被阻塞的任务数
si: swap in，从交换分区读入内存的数据
so: swap out，从内存换出到交换分区的数据
bi: block in，从磁盘读入内存的数据量
bo: block out，从内存写入磁盘的数据量
```

### top

> 动态实时查看系统进程信息

```shell
$ top
top - 19:02:54 up 22:33,  3 users,  load average: 0.01, 0.02, 0.05
Tasks:  70 total,   2 running,  68 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.3 us,  0.3 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1882028 total,  1150528 free,    92380 used,   639120 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  1622908 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
20257 root      10 -10  130448  13148  10032 S  0.3  0.7   5:46.85 AliYunDun
    1 root      20   0   51776   3920   2612 S  0.0  0.2   0:02.77 systemd
```

- 第一行：
  - 19:02:54：当前时间；
  - up 22:33：系统已运行时间；
  - 3 users：在线用户；
  - load average：平均负载，与 `w, uptime` 解释相同；
- 第二行：Tasks；
  - 70 total：总进程；
  - 2 running：正在运行进程；
  - 68 sleeping：正在休眠进程；
  - 0 stopped：已停止进程；
  - 0 zombie：僵尸进程；
- 第三行：%Cpu(s)；
  - 0.3 us：用户进程 CPU 占用率；
  - 0.3 sy：系统占用率；
  - 0.0 ni：用户进程空间内改变过优先级的进程占用率；
  - 99.3 id：CPU 空闲率；
  - 0.0 wa：等待 IO 时间占用率；
  - 0.0 hi：硬中断比率；
  - 0.0 si：软中断比率；
- 第四行：KiB Mem；
  - 1882028 total：内存总量；
  - 1150528 free：空闲内存；
  - 92380 used：已用内存；
  - 639120 buff/cache：缓冲/缓存所占内存
- 第五行：KiB Swap
  - 0 total：交换区总量；
  - 0 free：交换区空闲量；
  - 0 used：交换区已使用量；
  - 1622908 avail Mem：可用内存；

- 第六行：
  - PID：Process ID，进程 ID；
  - USER：进程所属用户；
  - PR：Process Priority，进程优先级；
  - NI：NICE 值，越小优先级越高，最小 $-20$，最大 $20$（用户设置最大 $19$）；
  - VIRT：进程锁使用的虚拟内存总量，KB。VIRT = SWAP + RES；
  - RES：进程使用的、未被换出的物理内存大小，KB。RES=CODE + DATA；
  - SHR：SHARE，共享内存；
  - S：STATUS，进程状态。D=不可中断的睡眠状态；R=运行；S=睡眠；T=跟踪/停止；Z=僵尸进程；
  - %CPU：CPU 占用率；
  - %MEM：内存占用率；
  - TIME+：进程运行时间；
  - COMMAND：进程名称。

### 查看网卡流量

```shell
$ sar -n DEV # 查看网卡流量，默认 10 分钟更新一次
$ sar -n DEV 1 10 # 一秒显示一次，一共显示 10 次
$ sar -n DEV -f /var/log/sa/sa22 # 查看指定日期的流量日志
```

### 查看系统进程

```shell
$ ps -aux # a: all
$ ps -ef # e: every
$ ps -ejH # 查看进程树

UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 10月09 ?      00:00:03 /usr/lib/systemd/systemd --switched-ro
root         2     0  0 10月09 ?      00:00:00 [kthreadd]
```

### 查看系统开放端口

> Netstat  prints  information about the Linux networking subsystem.  The type
>        of information printed is controlled by the first argument, as follows:

```shell
$ netstat -lnp

Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1707/sshd
udp        0      0 0.0.0.0:68              0.0.0.0:*                           699/dhclient

Active UNIX domain sockets (only servers)
Proto RefCnt Flags       Type       State         I-Node   PID/Program name     Path
unix  2      [ ACC ]     STREAM     LISTENING     7459     1/systemd            /run/systemd/journal/stdout
```

### 查看网络连接状态

```shell
$ netstat -an

Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 172.16.139.15:47044     100.100.45.106:443      TIME_WAIT
tcp        0      0 172.16.139.15:34908     100.100.30.25:80        ESTABLISHED

Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  3      [ ]         DGRAM                    7449     /run/systemd/notify
unix  2      [ ]         DGRAM                    7451     /run/systemd/cgroups-agent
unix  2      [ ACC ]     STREAM     LISTENING     7459     /run/systemd/journal/stdout
```

### 修改 IP 重启网卡

```shell
$ vi /etc/sysconfig/network-scripts/ifcft-eth0
# 内容类似下面这种
DEVICE=eth0
HWADDR=00:0C:29:06:37:BA
TYPE=Ethernet
UUID=0eea1820-1fe8-4a80-a6f0-39b3d314f8da
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=static
IPADDR=192.168.147.130
NETMASK=255.255.255.0
GATEWAY=192.168.147.2
DNS1=192.168.147.2
DNS2=8.8.8.8

# 重启网卡
$ ifdown eth0
$ ifup eth0

# 或者重启网络服务
$ service network restart
```

### 一网卡配置多 IP

```shell
# 查看eth0的配置
$ cat /etc/sysconfig/network-scripts/ifcfg-eth0

# 1. 新建 ifcfg-eth0:1 文件
$ cp /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-eth0:1

# 2. 修改其内容如下
$ vim /etc/sysconfig/network-scripts/ifcfg-eth0:1
# 只是示例
DEVICE=eth0:1
HWADDR=00:0C:29:06:37:BA
TYPE=Ethernet
UUID=0eea1820-1fe8-4a80-a6f0-39b3d314f8da
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=static
IPADDR=192.168.147.133 # IP address
NETMASK=255.255.255.0
GATEWAY=192.168.147.2
DNS1=192.168.147.2
DNS2=8.8.8.8

# 重启网络服务
$ service network restart
```

### 修改主机名

```shell
# 查看主机名
$ hostname

# 修改主机名
$ hostname CentOS7.3

# 永久生效修改 添加 HOSTNAME=CentOS7.3
$ vim /etc/sysconfig/network

```

### 设置 DNS

```shell
# 需要配置以下两个文件
$ cat /etc/resolv.conf
options timeout:2 attempts:3 rotate single-request-reopen
; generated by /usr/sbin/dhclient-script
nameserver 100.100.2.136
nameserver 100.100.2.138

$ cat /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
BOOTPROTO=dhcp
ONBOOT=yes
```

### ip-tables

```shell
# 拒绝来源 IP 为 192.168.1.101 访问本机 80 端口的包
$ iptables -I INPUT -s 192.168.1.101 -p tcp --dport 80 -j REJECT

# 把 iptables 的规则保存到文件中？如何恢复？
## 使用 iptables-save 重定向 到文件中
$ iptables-save > 1.ipt
## 使用 iptables-restore 反重定向
$ iptables-restore < 1.ipt
```

### 关闭系统服务

```shell
$ chkconfig servicename off

# 开启 nginx 3,5 级别，关闭其他级别。
## 关闭 nginx
$ chkconfig nginx off
$ chkconfig --level 35 nginx on
```

### 查看网卡或硬盘

```shell
$ dmesg
```

### xargs 和 exec

```shell
# 当前目录下所有后缀名为 .txt 的文件的权限修改为 777
$ find ./ --type f -name "*.txt" | args chmod 777
# OR
$ find ./ --type f -name "*.txt" exec chmod 777 {};
```

### 域名解析

```shell
# 命令行访问某网站，并且该网站域名还没有解析，如何做？
# 在 /etc/hosts 文件中增加一条从该网站域名到其 IP 的解析记录，或者用 curl -x

# 自定义域名解析: 可以一个 IP 对应多个域名，反之不然。
$ vi /etc/hosts

# 指定 DNS 服务器来解析域名？
$ dig @DNSip domain_name
$ dig @8.8.8.8 www.baidu.com # 使用 Google DNS 解析百度
```

### 问题解决思路

```shell
# 发现公司网站访问速度变的很慢如何解决？
# 1. 分析系统负载，使用 w 或 uptime 查看系统负载
# 2. 如果负载很高，使用 top 查看 CPU，MEM 等占用情况
# 3. 如果二者都正常，使用 sar 分析网卡流量
# 4. 根据分析结果采取对应措施
```

## 文件操作

### 检索文件名  find

```shell
find path [options] params
# 作用：在指定目录下查找文件
# 不加 path 作用域时的默认路径为当前目录及其子目录
find -name "helloworld.java"
# linux 系统中 / 代表 root 根目录 ~ 代表 home 目录
find / -name "helloworld.java"
# 支持模糊查询
find ~ -name "hello*"
# -iname 忽略大小写
find ~ -iname "hello*"
```

### 检索文件内容 grep

```shell
# grep : global regular expression print
# 用于查找文件里符合条件的字符串 可以使用正则表达式 最终输出到终端
grep [options] pattern file

# 从以 hello 开头的文件中寻找含 hello 的字符串
grep "hello" "hello*"
```

### 管道操作符  | 

```shell
# 可将指令连接起来 前一个指令的输出(stdout)作为后一个指令的输入(stdin)
# 只处理前一个命令的正确输出 不处理错误输出
# 右边的命令必须能够接收标准输入流 否则传递过程中会被抛弃
find ~ | grep "hello"
# grep -o
# grep -v
```

### 对文件内容做统计 awk

```shell
awk [options] 'cmd' file
# 一次读取一行文本 按输入分隔符进行切片 切成多个组成部分
# 将切片直接保存在内建的变量中 $1, $2... ($0 表示行的全部)
# 支持对耽搁切片的判断，支持循环判断，默认分隔符为空格。

# 打印 netstat.txt 文件中第一列，第四列的内容
awk '{print $1, $4}' netstat.txt

# 打印 netstat.txt 文件中 第一列中为 tcp 第二列为 1 的所有内容
# NR
awk '$1=="tcp" && $2=1 {print $0}' netstat.txt

# enginearr[] 为变量名
# 可以先用 grep 过滤数据，然后用 awk 来整理
awk '{enginearr[$1]++}END{for(i in enginearr) print i "\t" enginearr[i]}'
```

### 批量替换掉文档中的内容 sed

```java
// replace.java
Str a = "The girl's boyfriend is Jack".
Str b = "The girl often chats with Jack and Jack is Jack".
Str c = "The girl loves Jack so much".
    
Integer bf = new Integer(2);
```

```shell
sed [option] 'sed command' filename
# 全名 stream editor 流编辑器
# 适合用于对文本的 行内容 进行处理

# 将 replace.java 中所有 Str 字符串替换为 String
# 第一个 / 之前的 s 代表对字符串进行操作 
# 第一个 / 与第二个 / 之间的代表要操作的内容
# 第二个 / 与第三个 / 之间的内容表示要操作后的内容
sed 's/^Str/String/' replace.java

# 上面的命令指示将修改后的内容输出到 终端
# 如果需要输出到文件中需要加 -i
sed -i 's/^Str/String/' replace.java

# \ 为转义字符
# 将 replace.java 中所有以 . 作为结尾的变为以 ; 结尾的
sed -i 's/\.$/\;/' replace.java

# 默认只替换每行文件中第一个出现等于 Jack 的字符串
sed -i 's/Jack/me/' replace.java
# 全局替换
sed -i 's/Jack/me/g' replace.java

# 删除 3-4 行的空行 注意 ^ 与 *$ 之间的空格代表空行 /d 代表删除
sed -i '/^ *$/d' replace.java

# 删除含有 Integer 的行
sed -i '/Integer/d' replace.java
```

### 查看或者编辑文件

```shell
$ cat 猫一眼, 可以合并文件
$ head -n N 头几行
$ tail -n N 尾几行
$ more 较大文件, 分页显示, 提示文件显示百分比
$ less  less 可以随意浏览文件, 而 more 仅能向前移动, 而且 less 在查看之前不会加载整个文件
$ vi 
$ vim

# 如何查看大文件
# 就跟所有操作大文件的方法一样: 分治
# vim 默认加载整个文件, 但是可以按需取用, 使用 less 最好, 可以把命令直接加进去. 
```

