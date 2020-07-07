<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [文件操作](#文件操作)
- [root相关操作](#root相关操作)
- [本地文件上传下载](#本地文件上传下载)
- [打包解压缩命令](#打包解压缩命令)
- [获取当前路径](#获取当前路径)
- [ps操作](#ps操作)
- [CentOS SSH密钥登陆改为密码登陆](#centos-ssh密钥登陆改为密码登陆)
- [文件查看](#文件查看)
- [添加新用户并授权](#添加新用户并授权)
- [查看端口状态](#查看端口状态)

<!-- /TOC -->

# 文件操作

```bash
LS命令－作用：显示目录内容，类似DOS下的DIR－格式：LS【options】【filename】－常用参数：  
>-a:all，不隐藏任何以"."字符开始的文件  
>-l：使用较长的格式列出信息  
>-r:按照文件名的逆序打印输出  
>-F:加上文件类型的指示符  ls -lF | grep /  过滤  man ls          
查询ls的帮助文件  cat命令－作用：显示文件内容，concatenate的缩写，类似dos的type命令。－格式：cat【options】【fielname】－常用参数：  
>-n：显示文件内容的行号。  
>-b：类似-n，但是不对空白行进行编号。  
>-s：当遇到有连续两行以上的空白行时，就代换为一行的空白行。

mv命令－作用：更改文件或者目录的名字。－格式：mv[options]source destination－常用参数：  
>-f：强制模式，覆盖文件不提示。  
>-i：交互模式，当要覆盖文件的时候给提示。

rm命令－作用：删除文件命令，类似dos的del命令－格式：rm【options】filenames－常用参数： 
>-f：强制模式，不给提示。 
>-r,-R：删除目录，recursive

mkdir命令－作用：创建目录，类似dos的md命令－格式：mkdir【options】directory－常用参数： 
>-p：创建目录和它的父目录。 
>-m：指定模式，类似chmod。

more命令－作用：分屏显示输出结果，同DOS下的more命令。－格式：more【options】【filename】－常用参数： 
>-p：通过清屏而不是滚动来显示信息 
>-＋num：从第num行开始显示 
>-s：把连续的多行空白行压缩成一行

cat aa.txt | more  通过管道的作用连接两个命令

grep命令－作用：在文件中搜索特定的字符串。       
 Global Regular Expression Print－格式：grep【options】PATTERN【filename】－常用参数： 
>-i：不区分大小写 
>-n：显示序号 
>-v：显示不匹配的内容

-多条件查询

grep -E "exe|dll" aa.txt

find命令－作用：搜索指定目录下的文件－格式：find【path】【options】【expression】－常用参数： 
>-name：搜索指定文件名字的文件，支持通配符 
>-atime n：搜索过去n天之内访问的文件 
>-ctime n：搜索过去n天之内修改的文件 
>-group gname：搜索指定组属的文件

file命令－作用：判断文件的类型－格式：file【options】filename－常用参数： 
>-z：检测压缩过的文件类型说明：     
file命令可以检测某个文件是否是目录，shell脚本，英文文本，二进制可执行文件，c语言源文件，文本文件，dos的可执行文件。

chmod命令－作用：改变文件存取权限。－格式：chmod【options】 mode filename－常用参数： 
>-R:对目录下的文件进行递归操作 
>+r：增加读权限 
>-W：删除写权限 
>-x：增加执行权限

```

[toTop](#jump)

# root相关操作

```bash

更改密码：sudo passwd root

获取root权限：sudo su -
```

[toTop](#jump)


# 本地文件上传下载

```bash

下载安装
sudo yum install lrzsz

sz命令发送文件到本地：
sz filename

rz命令本地上传文件到服务器：
rz
```

[toTop](#jump)

# 打包解压缩命令

```bash
.tar
压缩：tar cvf FileName.tar FileName
解压：tar xvf FileName.tar
--------------------------------------------- 
.gz
解压1：gunzip FileName.gz 
解压2：gzip -d FileName.gz 
压缩：gzip FileName 
.tar.gz 
解压：tar zxvf FileName.tar.gz 
压缩：tar zcvf FileName.tar.gz DirName 
--------------------------------------------- 
.bz2 
解压1：bzip2 -d FileName.bz2 
解压2：bunzip2 FileName.bz2 
压缩： bzip2 -z FileName 
.tar.bz2 
解压：tar jxvf FileName.tar.bz2 
压缩：tar jcvf FileName.tar.bz2 DirName 
--------------------------------------------- 
.bz 
解压1：bzip2 -d FileName.bz 
解压2：bunzip2 FileName.bz 
压缩：未知 
.tar.bz 
解压：tar jxvf FileName.tar.bz 
压缩：未知 
--------------------------------------------- 
.Z 
解压：uncompress FileName.Z 
压缩：compress FileName 
.tar.Z 
解压：tar Zxvf FileName.tar.Z 
压缩：tar Zcvf FileName.tar.Z DirName 
--------------------------------------------- 
.tgz 
解压：tar zxvf FileName.tgz 
压缩：未知 
.tar.tgz 
解压：tar zxvf FileName.tar.tgz 
压缩：tar zcvf FileName.tar.tgz FileName 
--------------------------------------------- 
.zip 
解压：unzip FileName.zip 
压缩：zip FileName.zip DirName 
--------------------------------------------- 
.rar 
解压：rar a FileName.rar 
压缩：rar e FileName.rar 


```

[toTop](#jump)

# 获取当前路径

```bash

pwd
```

[toTop](#jump)

# ps操作

```bash
PID: 运行着的命令(CMD)的进程编号    TTY: 命令所运行的位置（终端）    TIME: 运行着的该命令所占用的CPU处理时间    CMD: 该进程所运行的命令这些信息在显示时未排序。
1.显示所有当前进程
使用 -a 参数。-a 代表 all。同时加上x参数会显示没有控制终端的进程。
$ ps -ax
这个命令的结果或许会很长。为了便于查看，可以结合less命令和管道来使用。
$ ps -ax | less

2.根据用户过滤进程
在需要查看特定用户进程的情况下，我们可以使用 -u 参数。比如我们要查看用户'pungki'的进程，可以通过下面的命令：
$ ps -u pungki
通过cpu和内存使用来过滤进程
也许你希望把结果按照 CPU 或者内存用量来筛选，这样你就找到哪个进程占用了你的资源。要做到这一点，我们可以使用 aux 参数，来显示全面的信息:
$ ps -aux | less
当结果很长时，我们可以使用管道和less命令来筛选。
默认的结果集是未排好序的。可以通过 --sort命令来排序。

3.根据 CPU 使用来升序排序
$ ps -aux --sort -pcpu | less

4.根据 内存使用 来升序排序
$ ps -aux --sort -pmem | less
我们也可以将它们合并到一个命令，并通过管道显示前10个结果：
$ ps -aux --sort -pcpu,+pmem | head -n 10

5.通过进程名和PID过滤
使用 -C 参数，后面跟你要找的进程的名字。比如想显示一个名为getty的进程的信息，就可以使用下面的命令：
$ ps -C getty
如果想要看到更多的细节，我们可以使用-f参数来查看格式化的信息列表：
$ ps -f -C getty

6.根据线程来过滤进程
如果我们想知道特定进程的线程，可以使用-L 参数，后面加上特定的PID。
$ ps -L 1213

7.树形显示进程
有时候我们希望以树形结构显示进程，可以使用 -axjf 参数。
$ps -axjf
或者可以使用另一个命令。
$ pstree

8.显示安全信息
如果想要查看现在有谁登入了你的服务器。可以使用ps命令加上相关参数:
$ ps -eo pid,user,args
参数 -e 显示所有进程信息，-o 参数控制输出。Pid,User 和 Args参数显示PID，运行应用的用户和该应用。
能够与-e 参数 一起使用的关键字是args, cmd, comm, command, fname, ucmd, ucomm, lstart, bsdstart 和 start。

9.格式化输出root用户（真实的或有效的UID）创建的进程
系统管理员想要查看由root用户运行的进程和这个进程的其他相关信息时，可以通过下面的命令:
$ ps -U root -u root u
-U 参数按真实用户ID(RUID)筛选进程，它会从用户列表中选择真实用户名或 ID。真实用户即实际创建该进程的用户。
-u 参数用来筛选有效用户ID（EUID）。
最后的u参数用来决定以针对用户的格式输出，由User, PID, %CPU, %MEM, VSZ, RSS, TTY, STAT, START, TIME 和 COMMAND这几列组成。

10.使用PS实时监控进程状态
ps 命令会显示你系统当前的进程状态，但是这个结果是静态的。
当有一种情况，我们需要像上面第四点中提到的通过CPU和内存的使用率来筛选进程，并且我们希望结果能够每秒刷新一次。
为此，我们可以将ps命令和watch命令结合起来。
$ watch -n 1 ‘ps -aux --sort -pmem, -pcpu’
如果输出太长，我们也可以限制它，比如前20条，我们可以使用head命令来做到。
$ watch -n 1 ‘ps -aux --sort -pmem, -pcpu | head 20’
这里的动态查看并不像top或者htop命令一样。但是使用ps的好处是你能够定义显示的字段，你能够选择你想查看的字段。
举个例子，如果你只需要看名为'pungki'用户的信息，你可以使用下面的命令：
$ watch -n 1 ‘ps -aux -U pungki u --sort -pmem, -pcpu | head 20’

```

[toTop](#jump)

# CentOS SSH密钥登陆改为密码登陆


```bash

1.检查安装系统时是否已经安装SSH服务端软件包:   rpm -qa|grep openssh 

若显示结果中包含openssh-server-*,则说明已经安装,直接启动   sshd服务就可以了(service sshd start).(其中*的内容是该包的版本,一般为3.5p1-6)

netstat -a | more看有没正常启动

如果出现：

tcp   0      0 *:ssh         *:*         LISTEN

就说明正常启动了

vi /etc/ssh/sshd_config

2.设置为密码登陆方式

查找

PermitRootLogin yes

删除前面的#注释

查找

PasswordAuthentication no

改为

PasswordAuthentication yes


保存

3.重启ssh服务或重启服务器

service sshd restart

```

[toTop](#jump)

# 文件查看


```bash
tail -f      等同于--follow=descriptor，根据文件描述符进行追踪，当文件改名或被删除，追踪停止
 
tail -F     等同于--follow=name  --retry，根据文件名进行追踪，并保持重试，即该文件被删除或改名后，如果再次创建相同的文件名，会继续追踪
 
tailf        等同于tail -f -n 10（貌似tail -f或-F默认也是打印最后10行，然后追踪文件），与tail -f不同的是，如果文件不增长，它不会去访问磁盘文件，所以tailf特别适合那些便携机上跟踪日志文件，因为它减少了磁盘访问，可以省电

```
[toTop](#jump)

# 添加新用户并授权

```bash
一）创建一个admin用户
 
 
adduser admin
 
 
（二）为admin用户设置密码
 
passwd admin
 
 
（四） 查找sudoers文件所在
 
whereis sudoers
 
（五）查看一下文件的权限
 
 
ls -l /etc/sudoers
 
可以看到只有只读的权限，这时候我们要加入一个可写的（w）的权限
 
（六）加入可写的权限
 
 
chmod -v u+w /etc/sudoers
 
（七）把admin用户添加到sudoers中
 
vi /etc/sudoers1
 
 
 
## Allow root to run any commands anywher  
root    ALL=(ALL)       ALL  
admin ALL=(ALL)       ALL  #新增admin用户123
 
（八）wq保存，然后把sudoers文件权限改回去，毕竟这是一个重要的文件
 
chmod -v u-w /etc/sudoers
 
 
（九）测试新用户admin进行登陆，使用su  admin操作：

su admin

```

[toTop](#jump)


# 查看端口状态

```bash
netstat命令各个参数说明如下：
 
　　-t : 指明显示TCP端口
 
　　-u : 指明显示UDP端口
 
　　-l : 仅显示监听套接字(所谓套接字就是使应用程序能够读写与收发通讯协议(protocol)与资料的程序)
 
　　-p : 显示进程标识符和程序名称，每一个套接字/端口都属于一个程序。
 
　　-n : 不进行DNS轮询，显示IP(可以加速操作)
 
即可显示当前服务器上所有端口及进程服务，于grep结合可查看某个具体端口及服务情况··
 
netstat -ntlp   //查看当前所有tcp端口·
 
netstat -ntulp |grep 80   //查看所有80端口使用情况·
 
netstat -an | grep 3306   //查看所有3306端口使用情况·
 
查看一台服务器上面哪些服务及端口
 
netstat  -lanp
 
查看一个服务有几个端口。比如要查看mysqld
 
ps -ef |grep mysqld
 
查看某一端口的连接数量,比如3306端口
 
netstat -pnt |grep :3306 |wc
 
查看某一端口的连接客户端IP 比如3306端口
 
netstat -anp |grep 3306
 
netstat -an 查看网络端口 
 
lsof -i :port，使用lsof -i :port就能看见所指定端口运行的程序，同时还有当前连接。 
 
nmap 端口扫描
netstat -nupl  (UDP类型的端口)
netstat -ntpl  (TCP类型的端口)
netstat -anp 显示系统端口使用情况

```

[toTop](#jump)