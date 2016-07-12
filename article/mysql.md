

一、MySQL复制概述

   ⑴、MySQL数据的复制的基本介绍

   目前MySQL数据库已经占去数据库市场上很大的份额，其一是由于MySQL数据的开源性和高性能，当然还有重要的一条就是免费~不过不知道还能免费多久，不容乐观的未来，但是我们还是要能熟练掌握MySQL数据的架构和安全备份等功能，毕竟现在它还算是开源界的老大吧！

   MySQL数据库支持同步复制、单向、异步复制，在复制的过程中一个服务器充当主服务，而一个或多个服务器充当从服务器。主服务器将更新写入二进制日志文件，并维护文件的一个索引以跟踪日志循环。这些日志可以记录发送到从服务器的更新。当一个从服务器连接主服务器时，它通知主服务器从服务器在日志中读取的最后一次成功更新的位置。从服务器接收从那时起发生的任何更新，然后封锁并等待主服务器通知新的更新。

请注意当你进行复制时，所有对复制中的表的更新必须在主服务器上进行。否则，你必须要小心，以避免用户对主服务器上的表进行的更新与对从服务器上的表所进行的更新之间的冲突。
    单向复制有利于健壮性、速度和系统管理：

   健壮性：主服务器/从服务器设置增加了健壮性。主服务器出现问题时，你可以切换到从服务器作为备份。

   速度快：通过在主服务器和从服务器之间切分处理客户查询的负荷，可以得到更好的客户响应时间。SELECT查询可以发送到从服务器以降低主服务器的查询处理负荷。但修改数据的语句仍然应发送到主服务器，以便主服务器和从服务器保持同步。如果非更新查询为主，该负载均衡策略很有效，但一般是更新查询。

   系统管理：使用复制的另一个好处是可以使用一个从服务器执行备份，而不会干扰主服务器。在备份过程中主服务器可以继续处理更新。

   ⑵、MySQL数据复制的原理

   MySQL复制基于主服务器在二进制日志中跟踪所有对数据库的更改(更新、删除等等)。因此，要进行复制，必须在主服务器上启用二进制日志。

   每个从服务器从主服务器接收主服务器已经记录到其二进制日志的保存的更新，以便从服务器可以对其数据拷贝执行相同的更新。

   认识到二进制日志只是一个从启用二进制日志的固定时间点开始的记录非常重要。任何设置的从服务器需要主服务器上的在主服务器上启用二进制日志时的数据库拷贝。如果启动从服务器时，其数据库与主服务器上的启动二进制日志时的状态不相同，从服务器很可能失败。

   将主服务器的数据拷贝到从服务器的一个途径是使用LOAD DATA FROM MASTER语句。请注意LOAD DATA FROM MASTER目前只在所有表使用MyISAM存储引擎的主服务器上工作。并且，该语句将获得全局读锁定，因此当表正复制到从服务器上时，不可能在主服务器上进行更新。当我们执行表的无锁热备份时，则不再需要全局读锁定。

   MySQL数据复制的原理图大致如下：

[![](http://img1.51cto.com/attachment/201305/082139520.png "图像 1111.png")](http://img1.51cto.com/attachment/201305/082139520.png)

从上图我们可以看出MySQL数据库的复制需要启动三个线程来实现：

   其中1个在主服务器上，另两个在从服务器上。当发出START SLAVE时，从服务器创建一个I/O线程，以连接主服务器并让它发送记录在其二进制日志中的语句。主服务器创建一个线程将二进制日志中的内容发送到从服务器。该线程可以识别为主服务器上SHOW PROCESSLIST的输出中的Binlog Dump线程。从服务器I/O线程读取主服务器Binlog Dump线程发送的内容并将该数据拷贝到从服务器数据目录中的本地文件中，即中继日志。第3个线程是SQL线程，是从服务器创建用于读取中继日志并执行日志中包含的更新。

   在前面的描述中，每个从服务器有3个线程。有多个从服务器的主服务器创建为每个当前连接的从服务器创建一个线程；每个从服务器有自己的I/O和SQL线程。

   这样读取和执行语句被分成两个独立的任务。如果语句执行较慢则语句读取任务没有慢下来。例如，如果从服务器有一段时间没有运行了，当从服务器启动时，其I/O线程可以很快地从主服务器索取所有二进制日志内容，即使SQL线程远远滞后。如果从服务器在SQL线程执行完所有索取的语句前停止，I/O 线程至少已经索取了所有内容，以便语句的安全拷贝保存到本地从服务器的中继日志中，供从服务器下次启动时执行。这样允许清空主服务器上的二进制日志，因为不再需要等候从服务器来索取其内容。

二、实列说明MySQL的主从复制架构和实现详细过程

     主从架构数据库的复制图如下：![](http://img1.51cto.com/attachment/201305/232014198.png "图像 4.png")

其配置详细过程如下：

   1、环境架构：

       RedHat Linux Enterprise 5.8         mysql-5.5.28-linux2.6-i686.tar

       Master：172.16.7.1/16                 Slave：172.16.7.2/16

   2 、安装mysql-5.5.28，需要在主节点和备节点上安装mysql

       Master：

       安装环境准备：

<div>

<div id="highlighter_931438" class="syntaxhighlighter  as3">

| 

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

<div class="line number6 index5 alt1">6</div>

<div class="line number7 index6 alt2">7</div>

<div class="line number8 index7 alt1">8</div>

<div class="line number9 index8 alt2">9</div>

<div class="line number10 index9 alt1">10</div>

<div class="line number11 index10 alt2">11</div>

<div class="line number12 index11 alt1">12</div>

<div class="line number13 index12 alt2">13</div>

<div class="line number14 index13 alt1">14</div>

<div class="line number15 index14 alt2">15</div>

<div class="line number16 index15 alt1">16</div>

<div class="line number17 index16 alt2">17</div>

<div class="line number18 index17 alt1">18</div>

<div class="line number19 index18 alt2">19</div>

<div class="line number20 index19 alt1">20</div>

<div class="line number21 index20 alt2">21</div>

<div class="line number22 index21 alt1">22</div>

<div class="line number23 index22 alt2">23</div>

<div class="line number24 index23 alt1">24</div>

<div class="line number25 index24 alt2">25</div>

<div class="line number26 index25 alt1">26</div>

<div class="line number27 index26 alt2">27</div>

<div class="line number28 index27 alt1">28</div>

<div class="line number29 index28 alt2">29</div>

<div class="line number30 index29 alt1">30</div>

<div class="line number31 index30 alt2">31</div>

<div class="line number32 index31 alt1">32</div>

<div class="line number33 index32 alt2">33</div>

<div class="line number34 index33 alt1">34</div>

<div class="line number35 index34 alt2">35</div>

<div class="line number36 index35 alt1">36</div>

<div class="line number37 index36 alt2">37</div>

<div class="line number38 index37 alt1">38</div>

<div class="line number39 index38 alt2">39</div>

<div class="line number40 index39 alt1">40</div>

<div class="line number41 index40 alt2">41</div>

 | 

<div class="container">

<div class="line number1 index0 alt2">`为mysql的安装提供前提环境和初始化安装mysql`</div>

<div class="line number2 index1 alt1">`创建数据库目录`</div>

<div class="line number3 index2 alt2">`# mkdir /mydata/data –pv`</div>

<div class="line number4 index3 alt1">`创建mysq用户`</div>

<div class="line number5 index4 alt2">`# useradd -r mysql`</div>

<div class="line number6 index5 alt1">`修改权限`</div>

<div class="line number7 index6 alt2">`# chown -R mysql.mysql /mydata/data/`</div>

<div class="line number8 index7 alt1">`使用mysql-``5.5``通用二进制包安装`</div>

<div class="line number9 index8 alt2">`解压mysql软件包`</div>

<div class="line number10 index9 alt1">`# tar xf mysql-``5.5``.``28``-linux2.``6``-i686.tar.gz-C /usr/local/`</div>

<div class="line number11 index10 alt2">`创建连接，为了方便查看mysql的版本等信息`</div>

<div class="line number12 index11 alt1">`# cd /usr/local/`</div>

<div class="line number13 index12 alt2">`#ln –sv mysql-``5.5``.``28``-linux2.``6``-i686.tar.gzmysql`</div>

<div class="line number14 index13 alt1">`修改属主属组`</div>

<div class="line number15 index14 alt2">`# cd mysql`</div>

<div class="line number16 index15 alt1">`# chown -R root.mysql ./*`</div>

<div class="line number17 index16 alt2">`初始化数据库`</div>

<div class="line number18 index17 alt1">`# scripts/mysql_install_db –user=mysql --datadir=/mydata/data/`</div>

<div class="line number19 index18 alt2">`提供配置文件`</div>

<div class="line number20 index19 alt1">`# cp support-files/my-large.cnf /etc/my.cnf`</div>

<div class="line number21 index20 alt2">`提供服务脚本`</div>

<div class="line number22 index21 alt1">`# cp support-files/mysql.server/etc/rc.d/init.d/mysqld`</div>

<div class="line number23 index22 alt2">`添加至服务列表`</div>

<div class="line number24 index23 alt1">`# chkconfig --add mysqld`</div>

<div class="line number25 index24 alt2">`# chkconfig --list mysqld`</div>

<div class="line number26 index25 alt1">`# chkconfig mysqld on`</div>

<div class="line number27 index26 alt2">`编辑配置文件，提供数据目录`</div>

<div class="line number28 index27 alt1">`# vim /etc/my.cnf`</div>

<div class="line number29 index28 alt2">`# The MySQL server  修改mysqld服务器端的内容`</div>

<div class="line number30 index29 alt1">`log-bin=master-bin 主服务器二进制日志文件前缀名`</div>

<div class="line number31 index30 alt2">`log-bin-index=master-bin.index  索引文件`</div>

<div class="line number32 index31 alt1">`innodb_file_per_table=` `1`     `开启innodb的一表一个文件的设置`</div>

<div class="line number33 index32 alt2">`server-id       =` `1`          `必须是唯一的`</div>

<div class="line number34 index33 alt1">`datadir =/mydata/data        数据目录路径`</div>

<div class="line number35 index34 alt2">`启动mysql服务`</div>

<div class="line number36 index35 alt1">`# servicemysqld start`</div>

<div class="line number37 index36 alt2">`为了便于下面的测试，设置环境变量`</div>

<div class="line number38 index37 alt1">`# vim/etc/profile.d/mysql.sh`</div>

<div class="line number39 index38 alt2">`export PATH=$PATH:/usr/local/mysql/bin`</div>

<div class="line number40 index39 alt1">`执行环境变量脚本，使其立即生效`</div>

<div class="line number41 index40 alt2">`# . /etc/profile.d/mysql.sh`</div>

</div>

 |

</div>

</div>

![](http://img1.51cto.com/attachment/201305/001648734.png "图像 5.png")

![](http://img1.51cto.com/attachment/201305/001649448.png "图像 6.png")

![](http://img1.51cto.com/attachment/201305/001649384.png "图像 7.png")

![](http://img1.51cto.com/attachment/201305/002242528.png "图像 8.png")

 启动服务并进行相关的测试：

![](http://img1.51cto.com/attachment/201305/002520642.png "图像 9.png")

 mysql的安装配置完成，下面增加一个用于同步数据的账户并设置相关的权限吧！

<div>

<div id="highlighter_326447" class="syntaxhighlighter  as3">

| 

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

 | 

<div class="container">

<div class="line number1 index0 alt2">`建立用户账户`</div>

<div class="line number2 index1 alt1">`mysql> grant replication slave on *.* to` `'chris'``@``'172.16.%.%'` `identified by` `'work'``;`</div>

<div class="line number3 index2 alt2">`刷新数据使其生效`</div>

<div class="line number4 index3 alt1">`mysql> flush privileges;`</div>

</div>

 |

</div>

</div>

![](http://img1.51cto.com/attachment/201305/003223318.png "图像 10.png")

   至此我们mysql的Master设置完成，下面进行slave端的设置吧！

   Slave：

   安装环境配置：

<div>

<div id="highlighter_447392" class="syntaxhighlighter  as3">

| 

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

<div class="line number6 index5 alt1">6</div>

<div class="line number7 index6 alt2">7</div>

<div class="line number8 index7 alt1">8</div>

<div class="line number9 index8 alt2">9</div>

<div class="line number10 index9 alt1">10</div>

<div class="line number11 index10 alt2">11</div>

<div class="line number12 index11 alt1">12</div>

<div class="line number13 index12 alt2">13</div>

<div class="line number14 index13 alt1">14</div>

<div class="line number15 index14 alt2">15</div>

<div class="line number16 index15 alt1">16</div>

<div class="line number17 index16 alt2">17</div>

<div class="line number18 index17 alt1">18</div>

<div class="line number19 index18 alt2">19</div>

<div class="line number20 index19 alt1">20</div>

<div class="line number21 index20 alt2">21</div>

<div class="line number22 index21 alt1">22</div>

<div class="line number23 index22 alt2">23</div>

<div class="line number24 index23 alt1">24</div>

<div class="line number25 index24 alt2">25</div>

<div class="line number26 index25 alt1">26</div>

<div class="line number27 index26 alt2">27</div>

<div class="line number28 index27 alt1">28</div>

<div class="line number29 index28 alt2">29</div>

<div class="line number30 index29 alt1">30</div>

<div class="line number31 index30 alt2">31</div>

<div class="line number32 index31 alt1">32</div>

<div class="line number33 index32 alt2">33</div>

<div class="line number34 index33 alt1">34</div>

<div class="line number35 index34 alt2">35</div>

<div class="line number36 index35 alt1">36</div>

<div class="line number37 index36 alt2">37</div>

<div class="line number38 index37 alt1">38</div>

<div class="line number39 index38 alt2">39</div>

 | 

<div class="container">

<div class="line number1 index0 alt2">`创建mysql数据库目录`</div>

<div class="line number2 index1 alt1">`# mkdir /mydata/data –pv`</div>

<div class="line number3 index2 alt2">`创建mysql用户`</div>

<div class="line number4 index3 alt1">`# useradd -r mysql`</div>

<div class="line number5 index4 alt2">`修改数据目录权限`</div>

<div class="line number6 index5 alt1">`# chown -R mysql.mysql /mydata/data/`</div>

<div class="line number7 index6 alt2">`使用mysql-``5.5``通用二进制包安装mysql`</div>

<div class="line number8 index7 alt1">`解压mysql软件包`</div>

<div class="line number9 index8 alt2">`# tar xf mysql-``5.5``.``28``-linux2.``6``-i686.tar.gz-C /usr/local/`</div>

<div class="line number10 index9 alt1">`创建连接，便于查看mysql的版本等信息`</div>

<div class="line number11 index10 alt2">`# cd /usr/local/`</div>

<div class="line number12 index11 alt1">`# ln –sv mysql-``5.5``.``28``-linux2.``6``-i686.tar.gzmysql`</div>

<div class="line number13 index12 alt2">`修改mysql属主属组`</div>

<div class="line number14 index13 alt1">`# cd mysql`</div>

<div class="line number15 index14 alt2">`# chown -R root.mysql ./*`</div>

<div class="line number16 index15 alt1">`初始化mysql数据库`</div>

<div class="line number17 index16 alt2">`# scripts/mysql_install_db –user=mysql--datadir=/mydata/data/`</div>

<div class="line number18 index17 alt1">`提供mysql配置文件`</div>

<div class="line number19 index18 alt2">`# cp support-files/my-large.cnf /etc/my.cnf`</div>

<div class="line number20 index19 alt1">`提供服务脚本`</div>

<div class="line number21 index20 alt2">`# cp support-files/mysql.server /etc/init.d/mysqld`</div>

<div class="line number22 index21 alt1">`添加至服务列表`</div>

<div class="line number23 index22 alt2">`# chkconfig --add mysqld`</div>

<div class="line number24 index23 alt1">`编辑配置文件`</div>

<div class="line number25 index24 alt2">`# vim /etc/my.cnf`</div>

<div class="line number26 index25 alt1">`# The MySQL server`</div>

<div class="line number27 index26 alt2">`#log-bin=mysql-bin      禁用二进制日志，从服务器不需要二进制日志文件`</div>

<div class="line number28 index27 alt1">`datadir = /mydata/data  mysql的数据目录`</div>

<div class="line number29 index28 alt2">`relay-log = relay-log   设置中继日志`</div>

<div class="line number30 index29 alt1">`relay-log-index = relay-log.index  中继日志索引`</div>

<div class="line number31 index30 alt2">`innodb_file_per_table =` `1`</div>

<div class="line number32 index31 alt1">`server-id       =` `2`    `id不要和主服务器的一样`</div>

<div class="line number33 index32 alt2">`设置环境变量`</div>

<div class="line number34 index33 alt1">`# vim/etc/profile.d/mysql.sh`</div>

<div class="line number35 index34 alt2">`export PATH=$PATH:/usr/local/mysql/bin`</div>

<div class="line number36 index35 alt1">`执行此脚本（导出环境变量）`</div>

<div class="line number37 index36 alt2">`# . /etc/profile.d/mysql.sh`</div>

<div class="line number38 index37 alt1">`启动服务`</div>

<div class="line number39 index38 alt2">`# service mysqld start`</div>

</div>

 |

</div>

</div>

![](http://img1.51cto.com/attachment/201305/005508870.png "图像 11.png")

![](http://img1.51cto.com/attachment/201305/005508798.png "图像 12.png")

![](http://img1.51cto.com/attachment/201305/005508700.png "图像 13.png")

![](http://img1.51cto.com/attachment/201305/005508530.png "图像 14.png")

  到这slave服务的mysql安装和配置完成，下面启动slave复制吧，开启之前先查看下从服务上的二进制文件吧

<div>

<div id="highlighter_861332" class="syntaxhighlighter  as3">

| 

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

 | 

<div class="container">

<div class="line number1 index0 alt2">`mysql> show master status; #在Master上执行查看二进制文件`</div>

<div class="line number2 index1 alt1">`在从服务器上开启复制功能`</div>

<div class="line number3 index2 alt2">`change master to master_host=``'172.16.7.1'``,master_user=``'chris'``,master_password=``'work'``,master_log_file=``'master-bin.000001'``,master_log_pos=``407``;`</div>

<div class="line number4 index3 alt1">`开启复制功能`</div>

<div class="line number5 index4 alt2">`mysql>start slave;`</div>

</div>

 |

</div>

</div>

![](http://img1.51cto.com/attachment/201305/142642166.png "图像 16.png")

![](http://img1.51cto.com/attachment/201305/135014618.png "图像 15.png")

![](http://img1.51cto.com/attachment/201305/135318274.png "图像 17.png")

至此我们的mysql服务器的主从复制架构已经基本完成，下面开启服务并测试测试吧~

在从服务器开启复制进程：mysql>start slave;

![](http://img1.51cto.com/attachment/201305/213221190.png "图像 14.png")

![](http://img1.51cto.com/attachment/201305/213221323.png "图像 15.png")

   至此我们mysql服务器的主从复制架构已经完成，但是我们现在的主从架构并不完善，因为我们的从服务上还可以进行数据库的写入操作，一旦用户把数据写入到从服务器的数据库内，然后从服务器从主服务器上同步数据库的时候，会造成数据的错乱，从而会造成数据的损坏，所以我们需要把从服务器设置成只读~方法如下：

注意：read-only = ON ，这项功能只对非管理员组以为的用户有效！

![](http://img1.51cto.com/attachment/201305/213339289.png "图像 17.png")

![](http://img1.51cto.com/attachment/201305/213339209.png "图像 18.png")

OK，此致我们的mysql基于主从架构的复制功能已经搭建全部完成~下面介绍下关于mysql数据目录下面各个文件的功能和作用！

![](http://img1.51cto.com/attachment/201305/213412138.png "图像 19.png")

   由于二进制文件的缓冲区内，当我们的服务器宕机的时候，缓存区内的数据并没有同步到二进制日志文件内的时候，那就悲剧了，缓冲区内的数据就无法找回了，为了防止这种情况的发送，我们通过设置mysql直接把二进制文件记录到二进制文件而不再缓冲区内停留。

sync-binlog = ON 在主服务器上进行设置，用于事务安全

  从上面我们可以看到从服务器启动的时候其Slave_IO_Running: Yes和Slave_SQL_Running: Yes是自动启动的，但是有时候我们在主服务上进行的误操作等，也会直接同步到从服务器上的，要想恢复那就难了，所以我们需要关闭其自动执行功能，让其能够停止，skip-slave-start = 1 ,让其不开启自动同步，但是遗憾的是mysql5.28上已经没有了，我们可以通过停止相关线程来实现：

mysql>STOP SLAVE 或STOP SLAVE  IO_THREAF或STOP SLAVE SQL_THREAD

<span style="color:#ff0000;">注意：从服务器的所有操作日志都会被记录到数据目录下的错误日志中！</span>

<span style="color:#000000;">三、MySQL的半同步复制</span>

<span style="color:#000000;"></span>

   实现半同步复制的功能很简单，只需在mysql的主服务器和从服务器上安装个google提供的插件即可实现，

   主服务上使用semisync_master.，从服务器上使用sosemisync_slave.so插件即可实现，插件在mysql通用二进制的mysql/lib/plugin目录内。

其配置步骤如下

1、分别在主从节点上安装相关的插件

master：

<div>

<div id="highlighter_482476" class="syntaxhighlighter  as3">

| 

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

 | 

<div class="container">

<div class="line number1 index0 alt2">`安装插件：mysql> INSTALL PLUGIN rpl_semi_sync_master SONAME` `'semisync_master.so'``;`</div>

<div class="line number2 index1 alt1">`启动模块：mysql> SET GLOBAL rpl_semi_sync_master_enabled =` `1``;`</div>

<div class="line number3 index2 alt2">`设置超时时间：mysql> SET GLOBAL rpl_semi_sync_master_timeout =` `1000``；`</div>

</div>

 |

</div>

</div>

<span style="color:#000000;"></span>![](http://img1.51cto.com/attachment/201305/213628525.png "图像 23.png")

<div>

<div id="highlighter_570177" class="syntaxhighlighter  as3">

| 

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

 | 

<div class="container">

<div class="line number1 index0 alt2">`slave：`</div>

<div class="line number2 index1 alt1">`安装插件：msyql> INSTALL PLUGIN rpl_semi_sync_slave SONAME` `'semisync_slave.so'``;`</div>

<div class="line number3 index2 alt2">`启动模块：mysql> SET GLOBAL rpl_semi_sync_slave_enabled =` `1``;`</div>

<div class="line number4 index3 alt1">`重启进程使其模块生效：mysql> STOP SLAVE IO_THREAD; START SLAVE IO_THREAD;`</div>

</div>

 |

</div>

</div>

![](http://img1.51cto.com/attachment/201305/213801612.png "图像 24.png")
  上面的设置时在mysql进程内动态设定了，会立即生效但是重启服务以后就会失效，为了保证永久有效，需要把相关配置写到主、从服务器的配置文件my.cnf内：

<div>

<div id="highlighter_459914" class="syntaxhighlighter  as3">

| 

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

<div class="line number6 index5 alt1">6</div>

<div class="line number7 index6 alt2">7</div>

<div class="line number8 index7 alt1">8</div>

 | 

<div class="container">

<div class="line number1 index0 alt2">`在Master和Slave的my.cnf中编辑：`</div>

<div class="line number2 index1 alt1">`# On Master`</div>

<div class="line number3 index2 alt2">`[mysqld]`</div>

<div class="line number4 index3 alt1">`rpl_semi_sync_master_enabled=``1`</div>

<div class="line number5 index4 alt2">`rpl_semi_sync_master_timeout=``1000`   `#此单位是毫秒`</div>

<div class="line number6 index5 alt1">`# On Slave`</div>

<div class="line number7 index6 alt2">`[mysqld]`</div>

<div class="line number8 index7 alt1">`rpl_semi_sync_slave_enabled=``1`</div>

</div>

 |

</div>

</div>

  确认半同步功能已经启用，通过下面的操作进行查看

<div>

<div id="highlighter_606465" class="syntaxhighlighter  as3">

| 

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

 | 

<div class="container">

<div class="line number1 index0 alt2">`master：`</div>

<div class="line number2 index1 alt1">`mysql> CREATE DATABASE asyncdb;`</div>

<div class="line number3 index2 alt2">`master> SHOW STATUS LIKE` `'Rpl_semi_sync_master_yes_tx'``;`</div>

<div class="line number4 index3 alt1">`slave> SHOW DATABASES;`</div>

<div class="line number5 index4 alt2">`其测试过程如下`</div>

</div>

 |

</div>

</div>

![](http://img1.51cto.com/attachment/201305/214144972.png "图像 25.png")

然后把从服务器上的复制进程开启，

![](http://img1.51cto.com/attachment/201305/214201559.png "图像 26.png")

  我们至此已经实现了mysql数据库复制的半同步方式的架构，并且通过测试查看了复制功能，下面我们进行双主模型架构吧。

四、MySQL设置主-主复制：master<-->slave 
1、在两台服务器上各自建立一个具有复制权限的用户；让两个数据库互为主从的关系

2、修改配置文件：

把上面的连个数据库的配置文件重新配置，其配置如下

<div>

<div id="highlighter_847245" class="syntaxhighlighter  as3">

| 

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

<div class="line number6 index5 alt1">6</div>

<div class="line number7 index6 alt2">7</div>

<div class="line number8 index7 alt1">8</div>

 | 

<div class="container">

<div class="line number1 index0 alt2">`# 主服务器上`</div>

<div class="line number2 index1 alt1">`[mysqld]`</div>

<div class="line number3 index2 alt2">`server-id =` `1`</div>

<div class="line number4 index3 alt1">`log-bin = mysql-bin`</div>

<div class="line number5 index4 alt2">`relay-log = relay-mysql`</div>

<div class="line number6 index5 alt1">`relay-log-index = relay-mysql.index`</div>

<div class="line number7 index6 alt2">`auto-increment-increment =` `2`           `#每次跳两个数。`</div>

<div class="line number8 index7 alt1">`auto-increment-offset =` `1`              `#从``1``开始。`</div>

</div>

 |

</div>

</div>

![](http://img1.51cto.com/attachment/201305/214826105.png "图像 27.png")

<div>

<div id="highlighter_523502" class="syntaxhighlighter  as3">

| 

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

<div class="line number6 index5 alt1">6</div>

<div class="line number7 index6 alt2">7</div>

 | 

<div class="container">

<div class="line number1 index0 alt2">`[mysqld]`</div>

<div class="line number2 index1 alt1">`server-id =` `2`</div>

<div class="line number3 index2 alt2">`log-bin = mysql-bin`</div>

<div class="line number4 index3 alt1">`relay-log = relay-mysql`</div>

<div class="line number5 index4 alt2">`relay-log-index = relay-mysql.index`</div>

<div class="line number6 index5 alt1">`auto-increment-increment =` `2`</div>

<div class="line number7 index6 alt2">`auto-increment-offset =` `2`</div>

</div>

 |

</div>

</div>

![](http://img1.51cto.com/attachment/201305/214932101.png "图像 28.png")

  如果此时两台服务器均为新建立，且无其它写入操作，各服务器只需记录当前自己二进制日志文件及事件位置，以之作为另外的服务器复制起始位置即可

<div>

<div id="highlighter_544247" class="syntaxhighlighter  as3">

| 

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

<div class="line number4 index3 alt1">4</div>

<div class="line number5 index4 alt2">5</div>

<div class="line number6 index5 alt1">6</div>

<div class="line number7 index6 alt2">7</div>

<div class="line number8 index7 alt1">8</div>

<div class="line number9 index8 alt2">9</div>

<div class="line number10 index9 alt1">10</div>

<div class="line number11 index10 alt2">11</div>

<div class="line number12 index11 alt1">12</div>

<div class="line number13 index12 alt2">13</div>

<div class="line number14 index13 alt1">14</div>

<div class="line number15 index14 alt2">15</div>

 | 

<div class="container">

<div class="line number1 index0 alt2">`master：查看日志文件信息`</div>

<div class="line number2 index1 alt1">`mysql> show master status;`</div>

<div class="line number3 index2 alt2">`+------------------+----------+--------------+------------------+`</div>

<div class="line number4 index3 alt1">`| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |`</div>

<div class="line number5 index4 alt2">`+------------------+----------+--------------+------------------+`</div>

<div class="line number6 index5 alt1">`| mysql-bin.``000001` `|     ` `107` `|              |                  |`</div>

<div class="line number7 index6 alt2">`+------------------+----------+--------------+------------------+`</div>

<div class="line number8 index7 alt1">`Slave:查看服务器日志文件信息`</div>

<div class="line number9 index8 alt2">`mysql> show master status;`</div>

<div class="line number10 index9 alt1">`+------------------+----------+--------------+------------------+`</div>

<div class="line number11 index10 alt2">`| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |`</div>

<div class="line number12 index11 alt1">`+------------------+----------+--------------+------------------+`</div>

<div class="line number13 index12 alt2">`| mysql-bin.``000001` `|     ` `107` `|              |                  |`</div>

<div class="line number14 index13 alt1">`+------------------+----------+--------------+------------------+`</div>

<div class="line number15 index14 alt2">`1` `row` `in` `set` `(``0.00` `sec)`</div>

</div>

 |

</div>

</div>

 在各个服务器上建立账号和权限，来进行同步设置

<div>

<div id="highlighter_573959" class="syntaxhighlighter  as3">

| 

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

 | 

<div class="container">

<div class="line number1 index0 alt2">`master：`</div>

<div class="line number2 index1 alt1">`mysql> GRANT REPLICATION SLAVE ON *.* TO` `'chrislee'``@``'172.16.%.%'` `IDENTIFIED BY` `'work'``;`</div>

<div class="line number3 index2 alt2">`mysql> flush privileges;`</div>

</div>

 |

</div>

</div>

![](http://img1.51cto.com/attachment/201305/215119511.png "图像 29.png")

<div>

<div id="highlighter_158583" class="syntaxhighlighter  as3">

| 

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

<div class="line number3 index2 alt2">3</div>

 | 

<div class="container">

<div class="line number1 index0 alt2">`slave：`</div>

<div class="line number2 index1 alt1">`mysql> GRANT REPLICATION SLAVE ON *.* TO` `'chrisli'``@``'172.16.%.%'` `IDENTIFIED BY` `'work'``;`</div>

<div class="line number3 index2 alt2">`mysql> flush privileges`</div>

</div>

 |

</div>

</div>

![](http://img1.51cto.com/attachment/201305/215153660.png "图像 30.png")

在各服务器上指定对另一台服务器为自己的主服务器即可：

<div>

<div id="highlighter_502069" class="syntaxhighlighter  as3">

| 

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

 | 

<div class="container">

<div class="line number1 index0 alt2">`server1`</div>

<div class="line number2 index1 alt1">`mysql> CHANGE MASTER TO MASTER_HOST=``'172.16.7.2'``,MASTER_USER=``'chrisli'``,MASTER_PASSWORD=``'work'``,MASTER_LOG_FILE=``'mysql-bin.000001'``,MASTER_LOG_POS=``344``;`</div>

</div>

 |

</div>

</div>

![](http://img1.51cto.com/attachment/201305/215313264.png "图像 31.png")

<div>

<div id="highlighter_664936" class="syntaxhighlighter  as3">

| 

<div class="line number1 index0 alt2">1</div>

<div class="line number2 index1 alt1">2</div>

 | 

<div class="container">

<div class="line number1 index0 alt2">`server2：`</div>

<div class="line number2 index1 alt1">`mysql> CHANGE MASTER TO MASTER_HOST=``'172.16.7.1'``,MASTER_USER=``'chrislee'``,MASTER_PASSWORD=``'work'``,MASTER_LOG_FILE=``'mysql-bin.000001'``,MASTER_LOG_POS=``345``;`</div>

</div>

 |

</div>

</div>

![](http://img1.51cto.com/attachment/201305/215339601.png "图像 33.png")

双主架构配置基本完成，下面在各自上面启动复制进程吧~并进行测试：

![](http://img1.51cto.com/attachment/201305/215419531.png "图像 34.png")

![](http://img1.51cto.com/attachment/201305/215419915.png "图像 35.png")

   至此我们通过上面的测试，可以看出已经实现了主主复制的功能~到此我们关于mysql数据库的主从复制、半同步复制和主主复制的架构都已经实现，东西较多~整理的不好，还望包涵~其中的错误还望各位大神指出~

                                                                     chrinux-chris linux

本文出自 “[Chris On the way](http://chrinux.blog.51cto.com)” 博客，请务必保留此出处[http://chrinux.blog.51cto.com/6466723/1204586](http://chrinux.blog.51cto.com/6466723/1204586)

