<div id="article_details" class="details">
    <div class="article_title">   
         <span class="ico ico_type_Repost"></span>

# 
        <span class="link_title">[
        MySQL数据的主从复制、半同步复制和主主复制详解            
        ](/goustzhu/article/details/9339621)</span>

</div>

        <div class="article_manage clearfix">
        <div class="article_r">
            <span class="link_postdate">2013-07-16 10:08</span>
            <span class="link_view" title="阅读次数">14062人阅读</span>
            <span class="link_comments" title="评论次数"> [评论](#comments)(0)</span>
            <span class="link_collect tracking-ad" data-mod="popu_171"> [收藏](javascript:void(0); "收藏")</span>
             <span class="link_report"> [举报](#report "举报")</span>

        </div>
    </div>
    <div class="embody" style="display:none" id="embody">
        <span class="embody_t">本文章已收录于：</span>
        <div class="embody_c" id="lib" value="{&quot;err&quot;:0,&quot;msg&quot;:&quot;ok&quot;,&quot;data&quot;:[]}"></div>
    </div>
    <style type="text/css">        
            .embody{
                padding:10px 10px 10px;
                margin:0 -20px;
                border-bottom:solid 1px #ededed;                
            }
            .embody_b{
                margin:0 ;
                padding:10px 0;
            }
            .embody .embody_t,.embody .embody_c{
                display: inline-block;
                margin-right:10px;
            }
            .embody_t{
                font-size: 12px;
                color:#999;
            }
            .embody_c{
                font-size: 12px;
            }
            .embody_c img,.embody_c em{
                display: inline-block;
                vertical-align: middle;               
            }
             .embody_c img{               
                width:30px;
                height:30px;
            }
            .embody_c em{
                margin: 0 20px 0 10px;
                color:#333;
                font-style: normal;
            }
    </style>
    <script type="text/javascript">
        $(function () {
            try
            {
                var lib = eval("("+$("#lib").attr("value")+")");
                var html = "";
                if (lib.err == 0) {
                    $.each(lib.data, function (i) {
                        var obj = lib.data[i];
                        //html += '![]()' + obj.name + "&nbsp;&nbsp;";
                        html += ' [';
                        html += ' ![]()';
                        html += ' _**' + obj.name + '**_';
                        html += ' ]()';
                    });
                    if (html != "") {
                        setTimeout(function () {
                            $("#lib").html(html);                      
                            $("#embody").show();
                        }, 100);
                    }
                }      
            } catch (err)
            { }

        });
    </script>
    <script type="text/javascript" src="http://static.blog.csdn.net/scripts/category.js"></script>  

<div id="article_content" class="article_content">

一、[MySQL](http://lib.csdn.net/base/14 "MySQL知识库")复制概述

&nbsp; &nbsp;⑴、MySQL数据的复制的基本介绍

&nbsp; &nbsp;目前MySQL[数据库](http://lib.csdn.net/base/14 "MySQL知识库")已经占去数据库市场上很大的份额，其一是由于MySQL数据的开源性和高性能，当然还有重要的一条就是免费~不过不知道还能免费多久，不容乐观的未来，但是我们还是要能熟练掌握MySQL数据的[架构](http://lib.csdn.net/base/16 "大型网站架构知识库")和安全备份等功能，毕竟现在它还算是开源界的老大吧！

&nbsp; &nbsp;MySQL数据库支持同步复制、单向、异步复制，在复制的过程中一个服务器充当主服务，而一个或多个服务器充当从服务器。主服务器将更新写入二进制日志文件，并维护文件的一个索引以跟踪日志循环。这些日志可以记录发送到从服务器的更新。当一个从服务器连接主服务器时，它通知主服务器从服务器在日志中读取的最后一次成功更新的位置。从服务器接收从那时起发生的任何更新，然后封锁并等待主服务器通知新的更新。

请注意当你进行复制时，所有对复制中的表的更新必须在主服务器上进行。否则，你必须要小心，以避免用户对主服务器上的表进行的更新与对从服务器上的表所进行的更新之间的冲突。

&nbsp; &nbsp;单向复制有利于健壮性、速度和系统管理：

&nbsp; &nbsp;健壮性：主服务器/从服务器设置增加了健壮性。主服务器出现问题时，你可以切换到从服务器作为备份。

&nbsp; &nbsp;速度快：通过在主服务器和从服务器之间切分处理客户查询的负荷，可以得到更好的客户响应时间。SELECT查询可以发送到从服务器以降低主服务器的查询处理负荷。但修改数据的语句仍然应发送到主服务器，以便主服务器和从服务器保持同步。如果非更新查询为主，该负载均衡策略很有效，但一般是更新查询。

&nbsp; &nbsp;系统管理：使用复制的另一个好处是可以使用一个从服务器执行备份，而不会干扰主服务器。在备份过程中主服务器可以继续处理更新。

&nbsp; &nbsp;⑵、MySQL数据复制的原理

&nbsp; &nbsp;MySQL复制基于主服务器在二进制日志中跟踪所有对数据库的更改(更新、删除等等)。因此，要进行复制，必须在主服务器上启用二进制日志。

&nbsp; &nbsp;每个从服务器从主服务器接收主服务器已经记录到其二进制日志的保存的更新，以便从服务器可以对其数据拷贝执行相同的更新。

&nbsp; &nbsp;认识到二进制日志只是一个从启用二进制日志的固定时间点开始的记录非常重要。任何设置的从服务器需要主服务器上的在主服务器上启用二进制日志时的数据库拷贝。如果启动从服务器时，其数据库与主服务器上的启动二进制日志时的状态不相同，从服务器很可能失败。

&nbsp; &nbsp;将主服务器的数据拷贝到从服务器的一个途径是使用LOAD DATA FROM MASTER语句。请注意LOAD DATA FROM MASTER目前只在所有表使用MyISAM存储引擎的主服务器上工作。并且，该语句将获得全局读锁定，因此当表正复制到从服务器上时，不可能在主服务器上进行更新。当我们执行表的无锁热备份时，则不再需要全局读锁定。

&nbsp; &nbsp;MySQL数据复制的原理图大致如下：

[![](http://img1.51cto.com/attachment/201305/082139520.png "图像 1111.png")](http://img1.51cto.com/attachment/201305/082139520.png)

从上图我们可以看出MySQL数据库的复制需要启动三个线程来实现：

&nbsp; &nbsp;其中1个在主服务器上，另两个在从服务器上。当发出START SLAVE时，从服务器创建一个I/O线程，以连接主服务器并让它发送记录在其二进制日志中的语句。主服务器创建一个线程将二进制日志中的内容发送到从服务器。该线程可以识别为主服务器上SHOW PROCESSLIST的输出中的Binlog Dump线程。从服务器I/O线程读取主服务器Binlog Dump线程发送的内容并将该数据拷贝到从服务器数据目录中的本地文件中，即中继日志。第3个线程是SQL线程，是从服务器创建用于读取中继日志并执行日志中包含的更新。

&nbsp; &nbsp;在前面的描述中，每个从服务器有3个线程。有多个从服务器的主服务器创建为每个当前连接的从服务器创建一个线程；每个从服务器有自己的I/O和SQL线程。

&nbsp; &nbsp;这样读取和执行语句被分成两个独立的任务。如果语句执行较慢则语句读取任务没有慢下来。例如，如果从服务器有一段时间没有运行了，当从服务器启动时，其I/O线程可以很快地从主服务器索取所有二进制日志内容，即使SQL线程远远滞后。如果从服务器在SQL线程执行完所有索取的语句前停止，I/O 线程至少已经索取了所有内容，以便语句的安全拷贝保存到本地从服务器的中继日志中，供从服务器下次启动时执行。这样允许清空主服务器上的二进制日志，因为不再需要等候从服务器来索取其内容。

二、实列说明MySQL的主从复制架构和实现详细过程

&nbsp; &nbsp; &nbsp;主从架构数据库的复制图如下：![](http://img1.51cto.com/attachment/201305/232014198.png "图像 4.png")

其配置详细过程如下：

&nbsp; &nbsp;1、环境架构：

&nbsp; &nbsp; &nbsp; &nbsp;RedHat Linux Enterprise 5.8 &nbsp; &nbsp; &nbsp; &nbsp; mysql-5.5.28-linux2.6-i686.tar

&nbsp; &nbsp; &nbsp; &nbsp;Master：172.16.7.1/16 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Slave：172.16.7.2/16

&nbsp; &nbsp;2 、安装mysql-5.5.28，需要在主节点和备节点上安装mysql

&nbsp; &nbsp; &nbsp; &nbsp;Master：

&nbsp; &nbsp; &nbsp; &nbsp;安装环境准备：

<div class="dp-highlighter bg_sql"><div class="bar"><div class="tools">**[sql]** [view plain](# "view plain")<span data-mod="popu_168"> [copy](# "copy")<div style="position: absolute; left: 751px; top: 2609px; width: 27px; height: 15px; z-index: 99;"><embed id="ZeroClipboardMovie_1" src="http://static.blog.csdn.net/scripts/ZeroClipboard/ZeroClipboard.swf" loop="false" menu="false" quality="best" bgcolor="#ffffff" width="27" height="15" name="ZeroClipboardMovie_1" align="middle" allowscriptaccess="always" allowfullscreen="false" type="application/x-shockwave-flash" pluginspage="http://www.macromedia.com/go/getflashplayer" flashvars="id=1&amp;width=27&amp;height=15" wmode="transparent"></div></span><span data-mod="popu_169"> [print](# "print")</span>[?](# "?")</div></div>

1.  <span><span>为mysql的安装提供前提环境和初始化安装mysql&nbsp;&nbsp;</span></span>
2.  <span>创建数据库目录&nbsp;&nbsp;</span>
3.  <span>#&nbsp;mkdir&nbsp;/mydata/data&nbsp;–pv&nbsp;&nbsp;</span>
4.  <span>创建mysq用户&nbsp;&nbsp;</span>
5.  <span>#&nbsp;useradd&nbsp;-r&nbsp;mysql&nbsp;&nbsp;</span>
6.  <span>修改权限&nbsp;&nbsp;</span>
7.  <span>#&nbsp;chown&nbsp;-R&nbsp;mysql.mysql&nbsp;/mydata/data/&nbsp;&nbsp;</span>
8.  <span>使用mysql-5.5通用二进制包安装&nbsp;&nbsp;</span>
9.  <span>解压mysql软件包&nbsp;&nbsp;</span>
10.  <span>#&nbsp;tar&nbsp;xf&nbsp;mysql-5.5.28-linux2.6-i686.tar.gz-C&nbsp;/usr/<span class="keyword">local</span><span>/&nbsp;&nbsp;</span></span>
11.  <span>创建连接，为了方便查看mysql的版本等信息&nbsp;&nbsp;</span>
12.  <span>#&nbsp;cd&nbsp;/usr/<span class="keyword">local</span><span>/&nbsp;&nbsp;</span></span>
13.  <span>#ln&nbsp;–sv&nbsp;mysql-5.5.28-linux2.6-i686.tar.gzmysql&nbsp;&nbsp;</span>
14.  <span>修改属主属组&nbsp;&nbsp;</span>
15.  <span>#&nbsp;cd&nbsp;mysql&nbsp;&nbsp;</span>
16.  <span>#&nbsp;chown&nbsp;-R&nbsp;root.mysql&nbsp;./*&nbsp;&nbsp;</span>
17.  <span>初始化数据库&nbsp;&nbsp;</span>
18.  <span>#&nbsp;scripts/mysql_install_db&nbsp;–<span class="func">user</span><span>=mysql&nbsp;</span><span class="comment">--datadir=/mydata/data/</span><span>&nbsp;&nbsp;</span></span>
19.  <span>提供配置文件&nbsp;&nbsp;</span>
20.  <span>#&nbsp;cp&nbsp;support-files/my-large.cnf&nbsp;/etc/my.cnf&nbsp;&nbsp;</span>
21.  <span>提供服务脚本&nbsp;&nbsp;</span>
22.  <span>#&nbsp;cp&nbsp;support-files/mysql.server/etc/rc.d/init.d/mysqld&nbsp;&nbsp;</span>
23.  <span>添加至服务列表&nbsp;&nbsp;</span>
24.  <span>#&nbsp;chkconfig&nbsp;<span class="comment">--add&nbsp;mysqld</span><span>&nbsp;&nbsp;</span></span>
25.  <span>#&nbsp;chkconfig&nbsp;<span class="comment">--list&nbsp;mysqld</span><span>&nbsp;&nbsp;</span></span>
26.  <span>#&nbsp;chkconfig&nbsp;mysqld&nbsp;<span class="keyword">on</span><span>&nbsp;&nbsp;</span></span>
27.  <span>编辑配置文件，提供数据目录&nbsp;&nbsp;</span>
28.  <span>#&nbsp;vim&nbsp;/etc/my.cnf&nbsp;&nbsp;</span>
29.  <span>#&nbsp;The&nbsp;MySQL&nbsp;server&nbsp;&nbsp;修改mysqld服务器端的内容&nbsp;&nbsp;</span>
30.  <span>log-bin=master-bin&nbsp;主服务器二进制日志文件前缀名&nbsp;&nbsp;</span>
31.  <span>log-bin-<span class="keyword">index</span><span>=master-bin.</span><span class="keyword">index</span><span>&nbsp;&nbsp;索引文件&nbsp;&nbsp;</span></span>
32.  <span>innodb_file_per_table=&nbsp;1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;开启innodb的一表一个文件的设置&nbsp;&nbsp;</span>
33.  <span>server-id&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;必须是唯一的&nbsp;&nbsp;</span>
34.  <span>datadir&nbsp;=/mydata/data&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;数据目录路径&nbsp;&nbsp;</span>
35.  <span>启动mysql服务&nbsp;&nbsp;</span>
36.  <span>#&nbsp;servicemysqld&nbsp;start&nbsp;&nbsp;</span>
37.  <span>为了便于下面的测试，设置环境变量&nbsp;&nbsp;</span>
38.  <span>#&nbsp;vim/etc/profile.d/mysql.sh&nbsp;&nbsp;</span>
39.  <span>export&nbsp;PATH=$PATH:/usr/<span class="keyword">local</span><span>/mysql/bin&nbsp;&nbsp;</span></span>
40.  <span>执行环境变量脚本，使其立即生效&nbsp;&nbsp;</span>
41.  <span>#&nbsp;.&nbsp;/etc/profile.d/mysql.sh&nbsp;&nbsp;</span><div class="save_code tracking-ad" data-mod="popu_249" style="display: none;">[![](http://static.blog.csdn.net/images/save_snippets.png)](javascript:;)</div></div><pre name="code" class="sql" style="display: none;">为mysql的安装提供前提环境和初始化安装mysql
创建数据库目录
# mkdir /mydata/data –pv
创建mysq用户
# useradd -r mysql
修改权限
# chown -R mysql.mysql /mydata/data/
使用mysql-5.5通用二进制包安装
解压mysql软件包
# tar xf mysql-5.5.28-linux2.6-i686.tar.gz-C /usr/local/
创建连接，为了方便查看mysql的版本等信息
# cd /usr/local/
#ln –sv mysql-5.5.28-linux2.6-i686.tar.gzmysql
修改属主属组
# cd mysql
# chown -R root.mysql ./*
初始化数据库
# scripts/mysql_install_db –user=mysql --datadir=/mydata/data/
提供配置文件
# cp support-files/my-large.cnf /etc/my.cnf
提供服务脚本
# cp support-files/mysql.server/etc/rc.d/init.d/mysqld
添加至服务列表
# chkconfig --add mysqld
# chkconfig --list mysqld
# chkconfig mysqld on
编辑配置文件，提供数据目录
# vim /etc/my.cnf
# The MySQL server  修改mysqld服务器端的内容
log-bin=master-bin 主服务器二进制日志文件前缀名
log-bin-index=master-bin.index  索引文件
innodb_file_per_table= 1     开启innodb的一表一个文件的设置
server-id       = 1          必须是唯一的
datadir =/mydata/data        数据目录路径
启动mysql服务
# servicemysqld start
为了便于下面的测试，设置环境变量
# vim/etc/profile.d/mysql.sh
export PATH=$PATH:/usr/local/mysql/bin
执行环境变量脚本，使其立即生效
# . /etc/profile.d/mysql.sh</pre>

![](http://img1.51cto.com/attachment/201305/001648734.png "图像 5.png")

![](http://img1.51cto.com/attachment/201305/001649448.png "图像 6.png")

![](http://img1.51cto.com/attachment/201305/001649384.png "图像 7.png")

![](http://img1.51cto.com/attachment/201305/002242528.png "图像 8.png")

&nbsp;启动服务并进行相关的测试：

![](http://img1.51cto.com/attachment/201305/002520642.png "图像 9.png")

&nbsp;mysql的安装配置完成，下面增加一个用于同步数据的账户并设置相关的权限吧！

<div style="padding:0px; margin:0px; color:rgb(85,85,85); font-family:宋体,'Arial Narrow',arial,serif; font-size:14px; line-height:28px">
<div id="highlighter_389095" class="syntaxhighlighter  as3" style="width:720px; padding:0px!important; margin:0.3em 0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:relative!important; right:auto!important; top:auto!important; vertical-align:baseline!important; font-family:Consolas,'Bitstream Vera Sans Mono','Courier New',Courier,monospace!important; font-size:1em!important; min-height:auto!important">
</div>
</div>

<div class="dp-highlighter bg_sql"><div class="bar"><div class="tools">**[sql]** [view plain](# "view plain")<span data-mod="popu_168"> [copy](# "copy")<div style="position: absolute; left: 751px; top: 5310px; width: 27px; height: 15px; z-index: 99;"><embed id="ZeroClipboardMovie_2" src="http://static.blog.csdn.net/scripts/ZeroClipboard/ZeroClipboard.swf" loop="false" menu="false" quality="best" bgcolor="#ffffff" width="27" height="15" name="ZeroClipboardMovie_2" align="middle" allowscriptaccess="always" allowfullscreen="false" type="application/x-shockwave-flash" pluginspage="http://www.macromedia.com/go/getflashplayer" flashvars="id=2&amp;width=27&amp;height=15" wmode="transparent"></div></span><span data-mod="popu_169"> [print](# "print")</span>[?](# "?")</div></div>

1.  <span><span>建立用户账户&nbsp;&nbsp;</span></span>
2.  <span>mysql&gt;&nbsp;<span class="keyword">grant</span><span>&nbsp;replication&nbsp;slave&nbsp;</span><span class="keyword">on</span><span>&nbsp;*.*&nbsp;</span><span class="keyword">to</span><span>&nbsp;</span><span class="string">'chris'</span><span>@</span><span class="string">'172.16.%.%'</span><span>&nbsp;identified&nbsp;</span><span class="keyword">by</span><span>&nbsp;</span><span class="string">'work'</span><span>;&nbsp;&nbsp;</span></span>
3.  <span>刷新数据使其生效&nbsp;&nbsp;</span>
4.  <span>mysql&gt;&nbsp;flush&nbsp;<span class="keyword">privileges</span><span>;&nbsp;&nbsp;</span></span><div class="save_code tracking-ad" data-mod="popu_249" style="display: none;">[![](http://static.blog.csdn.net/images/save_snippets.png)](javascript:;)</div></div><pre name="code" class="sql" style="display: none;">建立用户账户
mysql&gt; grant replication slave on *.* to 'chris'@'172.16.%.%' identified by 'work';
刷新数据使其生效
mysql&gt; flush privileges;</pre>

![](http://img1.51cto.com/attachment/201305/003223318.png "图像 10.png")

&nbsp; &nbsp;至此我们mysql的Master设置完成，下面进行slave端的设置吧！

&nbsp; &nbsp;Slave：

&nbsp; &nbsp;安装环境配置：

<div style="padding:0px; margin:0px; color:rgb(85,85,85); font-family:宋体,'Arial Narrow',arial,serif; font-size:14px; line-height:28px">
<div id="highlighter_38648" class="syntaxhighlighter  as3" style="width:720px; padding:0px!important; margin:0.3em 0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:relative!important; right:auto!important; top:auto!important; vertical-align:baseline!important; font-family:Consolas,'Bitstream Vera Sans Mono','Courier New',Courier,monospace!important; font-size:1em!important; min-height:auto!important">
</div>
</div>

<div class="dp-highlighter bg_sql"><div class="bar"><div class="tools">**[sql]** [view plain](# "view plain")<span data-mod="popu_168"> [copy](# "copy")<div style="position: absolute; left: 751px; top: 5656px; width: 27px; height: 15px; z-index: 99;"><embed id="ZeroClipboardMovie_3" src="http://static.blog.csdn.net/scripts/ZeroClipboard/ZeroClipboard.swf" loop="false" menu="false" quality="best" bgcolor="#ffffff" width="27" height="15" name="ZeroClipboardMovie_3" align="middle" allowscriptaccess="always" allowfullscreen="false" type="application/x-shockwave-flash" pluginspage="http://www.macromedia.com/go/getflashplayer" flashvars="id=3&amp;width=27&amp;height=15" wmode="transparent"></div></span><span data-mod="popu_169"> [print](# "print")</span>[?](# "?")</div></div>

1.  <span><span>创建mysql数据库目录&nbsp;&nbsp;</span></span>
2.  <span>#&nbsp;mkdir&nbsp;/mydata/data&nbsp;–pv&nbsp;&nbsp;</span>
3.  <span>创建mysql用户&nbsp;&nbsp;</span>
4.  <span>#&nbsp;useradd&nbsp;-r&nbsp;mysql&nbsp;&nbsp;</span>
5.  <span>修改数据目录权限&nbsp;&nbsp;</span>
6.  <span>#&nbsp;chown&nbsp;-R&nbsp;mysql.mysql&nbsp;/mydata/data/&nbsp;&nbsp;</span>
7.  <span>使用mysql-5.5通用二进制包安装mysql&nbsp;&nbsp;</span>
8.  <span>解压mysql软件包&nbsp;&nbsp;</span>
9.  <span>#&nbsp;tar&nbsp;xf&nbsp;mysql-5.5.28-linux2.6-i686.tar.gz-C&nbsp;/usr/<span class="keyword">local</span><span>/&nbsp;&nbsp;</span></span>
10.  <span>创建连接，便于查看mysql的版本等信息&nbsp;&nbsp;</span>
11.  <span>#&nbsp;cd&nbsp;/usr/<span class="keyword">local</span><span>/&nbsp;&nbsp;</span></span>
12.  <span>#&nbsp;ln&nbsp;–sv&nbsp;mysql-5.5.28-linux2.6-i686.tar.gzmysql&nbsp;&nbsp;</span>
13.  <span>修改mysql属主属组&nbsp;&nbsp;</span>
14.  <span>#&nbsp;cd&nbsp;mysql&nbsp;&nbsp;</span>
15.  <span>#&nbsp;chown&nbsp;-R&nbsp;root.mysql&nbsp;./*&nbsp;&nbsp;</span>
16.  <span>初始化mysql数据库&nbsp;&nbsp;</span>
17.  <span>#&nbsp;scripts/mysql_install_db&nbsp;–<span class="func">user</span><span>=mysql</span><span class="comment">--datadir=/mydata/data/</span><span>&nbsp;&nbsp;</span></span>
18.  <span>提供mysql配置文件&nbsp;&nbsp;</span>
19.  <span>#&nbsp;cp&nbsp;support-files/my-large.cnf&nbsp;/etc/my.cnf&nbsp;&nbsp;</span>
20.  <span>提供服务脚本&nbsp;&nbsp;</span>
21.  <span>#&nbsp;cp&nbsp;support-files/mysql.server&nbsp;/etc/init.d/mysqld&nbsp;&nbsp;</span>
22.  <span>添加至服务列表&nbsp;&nbsp;</span>
23.  <span>#&nbsp;chkconfig&nbsp;<span class="comment">--add&nbsp;mysqld</span><span>&nbsp;&nbsp;</span></span>
24.  <span>编辑配置文件&nbsp;&nbsp;</span>
25.  <span>#&nbsp;vim&nbsp;/etc/my.cnf&nbsp;&nbsp;</span>
26.  <span>#&nbsp;The&nbsp;MySQL&nbsp;server&nbsp;&nbsp;</span>
27.  <span>#log-bin=mysql-bin&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;禁用二进制日志，从服务器不需要二进制日志文件&nbsp;&nbsp;</span>
28.  <span>datadir&nbsp;=&nbsp;/mydata/data&nbsp;&nbsp;mysql的数据目录&nbsp;&nbsp;</span>
29.  <span>relay-log&nbsp;=&nbsp;relay-log&nbsp;&nbsp;&nbsp;设置中继日志&nbsp;&nbsp;</span>
30.  <span>relay-log-<span class="keyword">index</span><span>&nbsp;=&nbsp;relay-log.</span><span class="keyword">index</span><span>&nbsp;&nbsp;中继日志索引&nbsp;&nbsp;</span></span>
31.  <span>innodb_file_per_table&nbsp;=&nbsp;1&nbsp;&nbsp;</span>
32.  <span>server-id&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;2&nbsp;&nbsp;&nbsp;&nbsp;id不要和主服务器的一样&nbsp;&nbsp;</span>
33.  <span>设置环境变量&nbsp;&nbsp;</span>
34.  <span>#&nbsp;vim/etc/profile.d/mysql.sh&nbsp;&nbsp;</span>
35.  <span>export&nbsp;PATH=$PATH:/usr/<span class="keyword">local</span><span>/mysql/bin&nbsp;&nbsp;</span></span>
36.  <span>执行此脚本（导出环境变量）&nbsp;&nbsp;</span>
37.  <span>#&nbsp;.&nbsp;/etc/profile.d/mysql.sh&nbsp;&nbsp;</span>
38.  <span>启动服务&nbsp;&nbsp;</span>
39.  <span>#&nbsp;service&nbsp;mysqld&nbsp;start&nbsp;&nbsp;</span><div class="save_code tracking-ad" data-mod="popu_249" style="display: none;">[![](http://static.blog.csdn.net/images/save_snippets.png)](javascript:;)</div></div><pre name="code" class="sql" style="display: none;">创建mysql数据库目录
# mkdir /mydata/data –pv
创建mysql用户
# useradd -r mysql
修改数据目录权限
# chown -R mysql.mysql /mydata/data/
使用mysql-5.5通用二进制包安装mysql
解压mysql软件包
# tar xf mysql-5.5.28-linux2.6-i686.tar.gz-C /usr/local/
创建连接，便于查看mysql的版本等信息
# cd /usr/local/
# ln –sv mysql-5.5.28-linux2.6-i686.tar.gzmysql
修改mysql属主属组
# cd mysql
# chown -R root.mysql ./*
初始化mysql数据库
# scripts/mysql_install_db –user=mysql--datadir=/mydata/data/
提供mysql配置文件
# cp support-files/my-large.cnf /etc/my.cnf
提供服务脚本
# cp support-files/mysql.server /etc/init.d/mysqld
添加至服务列表
# chkconfig --add mysqld
编辑配置文件
# vim /etc/my.cnf
# The MySQL server
#log-bin=mysql-bin      禁用二进制日志，从服务器不需要二进制日志文件
datadir = /mydata/data  mysql的数据目录
relay-log = relay-log   设置中继日志
relay-log-index = relay-log.index  中继日志索引
innodb_file_per_table = 1
server-id       = 2    id不要和主服务器的一样
设置环境变量
# vim/etc/profile.d/mysql.sh
export PATH=$PATH:/usr/local/mysql/bin
执行此脚本（导出环境变量）
# . /etc/profile.d/mysql.sh
启动服务
# service mysqld start</pre>

![](http://img1.51cto.com/attachment/201305/005508870.png "图像 11.png")

![](http://img1.51cto.com/attachment/201305/005508798.png "图像 12.png")

![](http://img1.51cto.com/attachment/201305/005508700.png "图像 13.png")

![](http://img1.51cto.com/attachment/201305/005508530.png "图像 14.png")

&nbsp; 到这slave服务的mysql安装和配置完成，下面启动slave复制吧，开启之前先查看下从服务上的二进制文件吧

<div style="padding:0px; margin:0px; color:rgb(85,85,85); font-family:宋体,'Arial Narrow',arial,serif; font-size:14px; line-height:28px">
<div id="highlighter_323752" class="syntaxhighlighter  as3" style="width:720px; padding:0px!important; margin:0.3em 0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:relative!important; right:auto!important; top:auto!important; vertical-align:baseline!important; font-family:Consolas,'Bitstream Vera Sans Mono','Courier New',Courier,monospace!important; font-size:1em!important; min-height:auto!important">
<div class="dp-highlighter bg_sql"><div class="bar"><div class="tools">**[sql]** [view plain](# "view plain")<span data-mod="popu_168"> [copy](# "copy")<div style="position: absolute; left: 751px; top: 8452px; width: 27px; height: 15px; z-index: 99;"><embed id="ZeroClipboardMovie_4" src="http://static.blog.csdn.net/scripts/ZeroClipboard/ZeroClipboard.swf" loop="false" menu="false" quality="best" bgcolor="#ffffff" width="27" height="15" name="ZeroClipboardMovie_4" align="middle" allowscriptaccess="always" allowfullscreen="false" type="application/x-shockwave-flash" pluginspage="http://www.macromedia.com/go/getflashplayer" flashvars="id=4&amp;width=27&amp;height=15" wmode="transparent"></div></span><span data-mod="popu_169"> [print](# "print")</span>[?](# "?")</div></div>

1.  <span><span>mysql&gt;&nbsp;show&nbsp;master&nbsp;status;&nbsp;#在Master上执行查看二进制文件&nbsp;&nbsp;</span></span>
2.  <span>在从服务器上开启复制功能&nbsp;&nbsp;</span>
3.  <span>change&nbsp;master&nbsp;<span class="keyword">to</span><span>&nbsp;master_host=</span><span class="string">'172.16.7.1'</span><span>,master_user=</span><span class="string">'chris'</span><span>,master_password=</span><span class="string">'work'</span><span>,master_log_file=</span><span class="string">'master-bin.000001'</span><span>,master_log_pos=407;&nbsp;&nbsp;</span></span>
4.  <span>开启复制功能&nbsp;&nbsp;</span>
5.  <span>mysql&gt;start&nbsp;slave;&nbsp;&nbsp;</span><div class="save_code tracking-ad" data-mod="popu_249">[![](http://static.blog.csdn.net/images/save_snippets.png)](javascript:;)</div></div><pre name="code" class="sql" style="display: none;">mysql&gt; show master status; #在Master上执行查看二进制文件
在从服务器上开启复制功能
change master to master_host='172.16.7.1',master_user='chris',master_password='work',master_log_file='master-bin.000001',master_log_pos=407;
开启复制功能
mysql&gt;start slave;</pre>

![](http://img1.51cto.com/attachment/201305/142642166.png "图像 16.png")</div>
</div>

![](http://img1.51cto.com/attachment/201305/135014618.png "图像 15.png")

![](http://img1.51cto.com/attachment/201305/135318274.png "图像 17.png")

至此我们的mysql服务器的主从复制架构已经基本完成，下面开启服务并测试测试吧~

在从服务器开启复制进程：mysql&gt;start slave;

![](http://img1.51cto.com/attachment/201305/213221190.png "图像 14.png")

![](http://img1.51cto.com/attachment/201305/213221323.png "图像 15.png")

&nbsp; &nbsp;至此我们mysql服务器的主从复制架构已经完成，但是我们现在的主从架构并不完善，因为我们的从服务上还可以进行数据库的写入操作，一旦用户把数据写入到从服务器的数据库内，然后从服务器从主服务器上同步数据库的时候，会造成数据的错乱，从而会造成数据的损坏，所以我们需要把从服务器设置成只读~方法如下：

注意：read-only = ON ，这项功能只对非管理员组以为的用户有效！

![](http://img1.51cto.com/attachment/201305/213339289.png "图像 17.png")

![](http://img1.51cto.com/attachment/201305/213339209.png "图像 18.png")

OK，此致我们的mysql基于主从架构的复制功能已经搭建全部完成~下面介绍下关于mysql数据目录下面各个文件的功能和作用！

![](http://img1.51cto.com/attachment/201305/213412138.png "图像 19.png")

&nbsp; &nbsp;由于二进制文件的缓冲区内，当我们的服务器宕机的时候，缓存区内的数据并没有同步到二进制日志文件内的时候，那就悲剧了，缓冲区内的数据就无法找回了，为了防止这种情况的发送，我们通过设置mysql直接把二进制文件记录到二进制文件而不再缓冲区内停留。

sync-binlog = ON 在主服务器上进行设置，用于事务安全

&nbsp; 从上面我们可以看到从服务器启动的时候其Slave_IO_Running: Yes和Slave_SQL_Running: Yes是自动启动的，但是有时候我们在主服务上进行的误操作等，也会直接同步到从服务器上的，要想恢复那就难了，所以我们需要关闭其自动执行功能，让其能够停止，skip-slave-start = 1 ,让其不开启自动同步，但是遗憾的是mysql5.28上已经没有了，我们可以通过停止相关线程来实现：

mysql&gt;STOP SLAVE 或STOP SLAVE &nbsp;IO_THREAF或STOP SLAVE SQL_THREAD

<span style="padding:0px; margin:0px; color:rgb(255,0,0)">注意：从服务器的所有操作日志都会被记录到数据目录下的错误日志中！</span>

<span style="padding:0px; margin:0px; color:rgb(0,0,0)">三、MySQL的半同步复制</span>

<span style="padding:0px; margin:0px; color:rgb(0,0,0)"></span>

&nbsp; &nbsp;实现半同步复制的功能很简单，只需在mysql的主服务器和从服务器上安装个google提供的插件即可实现，

&nbsp; &nbsp;主服务上使用semisync_master.，从服务器上使用sosemisync_slave.so插件即可实现，插件在mysql通用二进制的mysql/lib/plugin目录内。

其配置步骤如下

1、分别在主从节点上安装相关的插件

master：

<div style="padding:0px; margin:0px; color:rgb(85,85,85); font-family:宋体,'Arial Narrow',arial,serif; font-size:14px; line-height:28px">
<div id="highlighter_869437" class="syntaxhighlighter  as3" style="width:720px; padding:0px!important; margin:0.3em 0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:relative!important; right:auto!important; top:auto!important; vertical-align:baseline!important; font-family:Consolas,'Bitstream Vera Sans Mono','Courier New',Courier,monospace!important; font-size:1em!important; min-height:auto!important">
</div>
</div>

<div class="dp-highlighter bg_sql"><div class="bar"><div class="tools">**[sql]** [view plain](# "view plain")<span data-mod="popu_168"> [copy](# "copy")<div style="position: absolute; left: 751px; top: 12517px; width: 27px; height: 15px; z-index: 99;"><embed id="ZeroClipboardMovie_5" src="http://static.blog.csdn.net/scripts/ZeroClipboard/ZeroClipboard.swf" loop="false" menu="false" quality="best" bgcolor="#ffffff" width="27" height="15" name="ZeroClipboardMovie_5" align="middle" allowscriptaccess="always" allowfullscreen="false" type="application/x-shockwave-flash" pluginspage="http://www.macromedia.com/go/getflashplayer" flashvars="id=5&amp;width=27&amp;height=15" wmode="transparent"></div></span><span data-mod="popu_169"> [print](# "print")</span>[?](# "?")</div></div>

1.  <span><span>安装插件：mysql&gt;&nbsp;INSTALL&nbsp;PLUGIN&nbsp;rpl_semi_sync_master&nbsp;SONAME&nbsp;</span><span class="string">'semisync_master.so'</span><span>;&nbsp;&nbsp;</span></span>
2.  <span>启动模块：mysql&gt;&nbsp;<span class="keyword">SET</span><span>&nbsp;</span><span class="keyword">GLOBAL</span><span>&nbsp;rpl_semi_sync_master_enabled&nbsp;=&nbsp;1;&nbsp;&nbsp;</span></span>
3.  <span>设置超时时间：mysql&gt;&nbsp;<span class="keyword">SET</span><span>&nbsp;</span><span class="keyword">GLOBAL</span><span>&nbsp;rpl_semi_sync_master_timeout&nbsp;=&nbsp;1000；&nbsp;&nbsp;</span></span><div class="save_code tracking-ad" data-mod="popu_249" style="display: none;">[![](http://static.blog.csdn.net/images/save_snippets.png)](javascript:;)</div></div><pre name="code" class="sql" style="display: none;">安装插件：mysql&gt; INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
启动模块：mysql&gt; SET GLOBAL rpl_semi_sync_master_enabled = 1;
设置超时时间：mysql&gt; SET GLOBAL rpl_semi_sync_master_timeout = 1000；</pre>

<span style="padding:0px; margin:0px; color:rgb(0,0,0)"></span>![](http://img1.51cto.com/attachment/201305/213628525.png "图像 23.png")

<div style="padding:0px; margin:0px; color:rgb(85,85,85); font-family:宋体,'Arial Narrow',arial,serif; font-size:14px; line-height:28px">
<div id="highlighter_468720" class="syntaxhighlighter  as3" style="width:720px; padding:0px!important; margin:0.3em 0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:relative!important; right:auto!important; top:auto!important; vertical-align:baseline!important; font-family:Consolas,'Bitstream Vera Sans Mono','Courier New',Courier,monospace!important; font-size:1em!important; min-height:auto!important">
<table border="0" cellpadding="0" cellspacing="0" style="padding:0px; margin:0px auto 10px; font-size:12px; border-collapse:collapse">
<tbody style="padding:0px!important; margin:0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-size:1em!important; min-height:auto!important">
<tr style="padding:0px!important; margin:0px!important; font-size:1em!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; top:auto!important; vertical-align:baseline!important; width:auto!important; min-height:auto!important">
<td class="gutter" style="padding:0px!important; margin:0px!important; font-size:1em!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-family:Consolas,'Bitstream Vera Sans Mono','Courier New',Courier,monospace!important; min-height:auto!important; color:rgb(175,175,175)!important">
<div class="line number1 index0 alt2" style="word-wrap:normal; padding:0px 0.5em 0px 1em!important; margin:0px!important; border-width:0px 3px 0px 0px!important; border-right-style:solid!important; border-right-color:rgb(108,226,108)!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; text-align:right!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-size:1em!important; min-height:auto!important; white-space:pre!important">
1</div>
<div class="line number2 index1 alt1" style="word-wrap:normal; padding:0px 0.5em 0px 1em!important; margin:0px!important; border-width:0px 3px 0px 0px!important; border-right-style:solid!important; border-right-color:rgb(108,226,108)!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; text-align:right!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-size:1em!important; min-height:auto!important; white-space:pre!important">
2</div>
<div class="line number3 index2 alt2" style="word-wrap:normal; padding:0px 0.5em 0px 1em!important; margin:0px!important; border-width:0px 3px 0px 0px!important; border-right-style:solid!important; border-right-color:rgb(108,226,108)!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; text-align:right!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-size:1em!important; min-height:auto!important; white-space:pre!important">
3</div>
<div class="line number4 index3 alt1" style="word-wrap:normal; padding:0px 0.5em 0px 1em!important; margin:0px!important; border-width:0px 3px 0px 0px!important; border-right-style:solid!important; border-right-color:rgb(108,226,108)!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; text-align:right!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-size:1em!important; min-height:auto!important; white-space:pre!important">
4</div>
</td>
<td class="code" style="height:26px; width:692px; padding:0px!important; margin:0px!important; font-size:1em!important; line-height:1.1em!important; float:none!important; border:0px!important; bottom:auto!important; left:auto!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; top:auto!important; vertical-align:baseline!important; font-family:Consolas,'Bitstream Vera Sans Mono','Courier New',Courier,monospace!important; min-height:auto!important">
<div class="container" style="padding:0px!important; margin:0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:relative!important; right:auto!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-size:1em!important; min-height:auto!important">
<div class="line number1 index0 alt2" style="padding:0px 1em!important; margin:0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-size:1em!important; min-height:auto!important">
`slave：`</div>
<div class="line number2 index1 alt1" style="padding:0px 1em!important; margin:0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-size:1em!important; min-height:auto!important">
`安装插件：msyql&gt;
 INSTALL PLUGIN rpl_semi_sync_slave SONAME&nbsp;``'semisync_slave.so'``;`</div>
<div class="line number3 index2 alt2" style="padding:0px 1em!important; margin:0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-size:1em!important; min-height:auto!important">
`启动模块：mysql&gt;
 SET GLOBAL rpl_semi_sync_slave_enabled =&nbsp;``1``;`</div>
<div class="line number4 index3 alt1" style="padding:0px 1em!important; margin:0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-size:1em!important; min-height:auto!important">
`重启进程使其模块生效：mysql&gt;
 STOP SLAVE IO_THREAD; START SLAVE IO_THREAD;`</div>
</div>
</td>
</tr>
</tbody>
</table>
</div>
</div>

![](http://img1.51cto.com/attachment/201305/213801612.png "图像 24.png")

&nbsp; 上面的设置时在mysql进程内动态设定了，会立即生效但是重启服务以后就会失效，为了保证永久有效，需要把相关配置写到主、从服务器的配置文件my.cnf内：

<div style="padding:0px; margin:0px; color:rgb(85,85,85); font-family:宋体,'Arial Narrow',arial,serif; font-size:14px; line-height:28px">
<div id="highlighter_718547" class="syntaxhighlighter  as3" style="width:720px; padding:0px!important; margin:0.3em 0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:relative!important; right:auto!important; top:auto!important; vertical-align:baseline!important; font-family:Consolas,'Bitstream Vera Sans Mono','Courier New',Courier,monospace!important; font-size:1em!important; min-height:auto!important">
</div>
</div>

<div class="dp-highlighter bg_sql"><div class="bar"><div class="tools">**[sql]** [view plain](# "view plain")<span data-mod="popu_168"> [copy](# "copy")<div style="position: absolute; left: 751px; top: 13862px; width: 27px; height: 15px; z-index: 99;"><embed id="ZeroClipboardMovie_6" src="http://static.blog.csdn.net/scripts/ZeroClipboard/ZeroClipboard.swf" loop="false" menu="false" quality="best" bgcolor="#ffffff" width="27" height="15" name="ZeroClipboardMovie_6" align="middle" allowscriptaccess="always" allowfullscreen="false" type="application/x-shockwave-flash" pluginspage="http://www.macromedia.com/go/getflashplayer" flashvars="id=6&amp;width=27&amp;height=15" wmode="transparent"></div></span><span data-mod="popu_169"> [print](# "print")</span>[?](# "?")</div></div>

1.  <span><span>在Master和Slave的my.cnf中编辑：&nbsp;&nbsp;</span></span>
2.  <span>#&nbsp;<span class="keyword">On</span><span>&nbsp;Master&nbsp;&nbsp;</span></span>
3.  <span>[mysqld]&nbsp;&nbsp;</span>
4.  <span>rpl_semi_sync_master_enabled=1&nbsp;&nbsp;</span>
5.  <span>rpl_semi_sync_master_timeout=1000&nbsp;&nbsp;&nbsp;#此单位是毫秒&nbsp;&nbsp;</span>
6.  <span>#&nbsp;<span class="keyword">On</span><span>&nbsp;Slave&nbsp;&nbsp;</span></span>
7.  <span>[mysqld]&nbsp;&nbsp;</span>
8.  <span>rpl_semi_sync_slave_enabled=1&nbsp;&nbsp;</span><div class="save_code tracking-ad" data-mod="popu_249" style="display: none;">[![](http://static.blog.csdn.net/images/save_snippets.png)](javascript:;)</div></div><pre name="code" class="sql" style="display: none;">在Master和Slave的my.cnf中编辑：
# On Master
[mysqld]
rpl_semi_sync_master_enabled=1
rpl_semi_sync_master_timeout=1000   #此单位是毫秒
# On Slave
[mysqld]
rpl_semi_sync_slave_enabled=1</pre>

&nbsp; 确认半同步功能已经启用，通过下面的操作进行查看

<div style="padding:0px; margin:0px; color:rgb(85,85,85); font-family:宋体,'Arial Narrow',arial,serif; font-size:14px; line-height:28px">
<div id="highlighter_748842" class="syntaxhighlighter  as3" style="width:720px; padding:0px!important; margin:0.3em 0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:relative!important; right:auto!important; top:auto!important; vertical-align:baseline!important; font-family:Consolas,'Bitstream Vera Sans Mono','Courier New',Courier,monospace!important; font-size:1em!important; min-height:auto!important">
<div class="dp-highlighter bg_sql"><div class="bar"><div class="tools">**[sql]** [view plain](# "view plain")<span data-mod="popu_168"> [copy](# "copy")<div style="position: absolute; left: 751px; top: 14126px; width: 27px; height: 15px; z-index: 99;"><embed id="ZeroClipboardMovie_7" src="http://static.blog.csdn.net/scripts/ZeroClipboard/ZeroClipboard.swf" loop="false" menu="false" quality="best" bgcolor="#ffffff" width="27" height="15" name="ZeroClipboardMovie_7" align="middle" allowscriptaccess="always" allowfullscreen="false" type="application/x-shockwave-flash" pluginspage="http://www.macromedia.com/go/getflashplayer" flashvars="id=7&amp;width=27&amp;height=15" wmode="transparent"></div></span><span data-mod="popu_169"> [print](# "print")</span>[?](# "?")</div></div>

1.  <span><span>master：&nbsp;&nbsp;</span></span>
2.  <span>mysql&gt;&nbsp;<span class="keyword">CREATE</span><span>&nbsp;</span><span class="keyword">DATABASE</span><span>&nbsp;asyncdb;&nbsp;&nbsp;</span></span>
3.  <span>master&gt;&nbsp;SHOW&nbsp;STATUS&nbsp;<span class="op">LIKE</span><span>&nbsp;</span><span class="string">'Rpl_semi_sync_master_yes_tx'</span><span>;&nbsp;&nbsp;</span></span>
4.  <span>slave&gt;&nbsp;SHOW&nbsp;DATABASES;&nbsp;&nbsp;</span>
5.  <span>其测试过程如下&nbsp;&nbsp;</span><div class="save_code tracking-ad" data-mod="popu_249">[![](http://static.blog.csdn.net/images/save_snippets.png)](javascript:;)</div></div><pre name="code" class="sql" style="display: none;">master：
mysql&gt; CREATE DATABASE asyncdb;
master&gt; SHOW STATUS LIKE 'Rpl_semi_sync_master_yes_tx';
slave&gt; SHOW DATABASES;
其测试过程如下</pre>

![](http://img1.51cto.com/attachment/201305/214144972.png "图像 25.png")</div>
</div>

然后把从服务器上的复制进程开启，

![](http://img1.51cto.com/attachment/201305/214201559.png "图像 26.png")

&nbsp; 我们至此已经实现了mysql数据库复制的半同步方式的架构，并且通过测试查看了复制功能，下面我们进行双主模型架构吧。

四、MySQL设置主-主复制：master&lt;--&gt;slave&nbsp;

1、在两台服务器上各自建立一个具有复制权限的用户；让两个数据库互为主从的关系

2、修改配置文件：

把上面的连个数据库的配置文件重新配置，其配置如下&nbsp;

<div style="padding:0px; margin:0px; color:rgb(85,85,85); font-family:宋体,'Arial Narrow',arial,serif; font-size:14px; line-height:28px">
<div id="highlighter_911208" class="syntaxhighlighter  as3" style="width:720px; padding:0px!important; margin:0.3em 0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:relative!important; right:auto!important; top:auto!important; vertical-align:baseline!important; font-family:Consolas,'Bitstream Vera Sans Mono','Courier New',Courier,monospace!important; font-size:1em!important; min-height:auto!important">
</div>
</div>

<div class="dp-highlighter bg_sql"><div class="bar"><div class="tools">**[sql]** [view plain](# "view plain")<span data-mod="popu_168"> [copy](# "copy")<div style="position: absolute; left: 751px; top: 15249px; width: 27px; height: 15px; z-index: 99;"><embed id="ZeroClipboardMovie_8" src="http://static.blog.csdn.net/scripts/ZeroClipboard/ZeroClipboard.swf" loop="false" menu="false" quality="best" bgcolor="#ffffff" width="27" height="15" name="ZeroClipboardMovie_8" align="middle" allowscriptaccess="always" allowfullscreen="false" type="application/x-shockwave-flash" pluginspage="http://www.macromedia.com/go/getflashplayer" flashvars="id=8&amp;width=27&amp;height=15" wmode="transparent"></div></span><span data-mod="popu_169"> [print](# "print")</span>[?](# "?")</div></div>

1.  <span><span>#&nbsp;主服务器上&nbsp;&nbsp;</span></span>
2.  <span>[mysqld]&nbsp;&nbsp;</span>
3.  <span>server-id&nbsp;=&nbsp;1&nbsp;&nbsp;</span>
4.  <span>log-bin&nbsp;=&nbsp;mysql-bin&nbsp;&nbsp;</span>
5.  <span>relay-log&nbsp;=&nbsp;relay-mysql&nbsp;&nbsp;</span>
6.  <span>relay-log-<span class="keyword">index</span><span>&nbsp;=&nbsp;relay-mysql.</span><span class="keyword">index</span><span>&nbsp;&nbsp;</span></span>
7.  <span>auto-increment-increment&nbsp;=&nbsp;2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#每次跳两个数。&nbsp;&nbsp;</span>
8.  <span>auto-increment-offset&nbsp;=&nbsp;1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#从1开始。&nbsp;&nbsp;</span><div class="save_code tracking-ad" data-mod="popu_249" style="display: none;">[![](http://static.blog.csdn.net/images/save_snippets.png)](javascript:;)</div></div><pre name="code" class="sql" style="display: none;"># 主服务器上
[mysqld]
server-id = 1
log-bin = mysql-bin
relay-log = relay-mysql
relay-log-index = relay-mysql.index
auto-increment-increment = 2           #每次跳两个数。
auto-increment-offset = 1              #从1开始。</pre>

![](http://img1.51cto.com/attachment/201305/214826105.png "图像 27.png")

<div style="padding:0px; margin:0px; color:rgb(85,85,85); font-family:宋体,'Arial Narrow',arial,serif; font-size:14px; line-height:28px">
<div id="highlighter_494882" class="syntaxhighlighter  as3" style="width:720px; padding:0px!important; margin:0.3em 0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:relative!important; right:auto!important; top:auto!important; vertical-align:baseline!important; font-family:Consolas,'Bitstream Vera Sans Mono','Courier New',Courier,monospace!important; font-size:1em!important; min-height:auto!important">
</div>
</div>

<div class="dp-highlighter bg_sql"><div class="bar"><div class="tools">**[sql]** [view plain](# "view plain")<span data-mod="popu_168"> [copy](# "copy")<div style="position: absolute; left: 751px; top: 16092px; width: 27px; height: 15px; z-index: 99;"><embed id="ZeroClipboardMovie_9" src="http://static.blog.csdn.net/scripts/ZeroClipboard/ZeroClipboard.swf" loop="false" menu="false" quality="best" bgcolor="#ffffff" width="27" height="15" name="ZeroClipboardMovie_9" align="middle" allowscriptaccess="always" allowfullscreen="false" type="application/x-shockwave-flash" pluginspage="http://www.macromedia.com/go/getflashplayer" flashvars="id=9&amp;width=27&amp;height=15" wmode="transparent"></div></span><span data-mod="popu_169"> [print](# "print")</span>[?](# "?")</div></div>

1.  <span><span>[mysqld]&nbsp;&nbsp;</span></span>
2.  <span>server-id&nbsp;=&nbsp;2&nbsp;&nbsp;</span>
3.  <span>log-bin&nbsp;=&nbsp;mysql-bin&nbsp;&nbsp;</span>
4.  <span>relay-log&nbsp;=&nbsp;relay-mysql&nbsp;&nbsp;</span>
5.  <span>relay-log-<span class="keyword">index</span><span>&nbsp;=&nbsp;relay-mysql.</span><span class="keyword">index</span><span>&nbsp;&nbsp;</span></span>
6.  <span>auto-increment-increment&nbsp;=&nbsp;2&nbsp;&nbsp;</span>
7.  <span>auto-increment-offset&nbsp;=&nbsp;2&nbsp;&nbsp;</span><div class="save_code tracking-ad" data-mod="popu_249" style="display: none;">[![](http://static.blog.csdn.net/images/save_snippets.png)](javascript:;)</div></div><pre name="code" class="sql" style="display: none;">[mysqld]
server-id = 2
log-bin = mysql-bin
relay-log = relay-mysql
relay-log-index = relay-mysql.index
auto-increment-increment = 2
auto-increment-offset = 2</pre>

![](http://img1.51cto.com/attachment/201305/214932101.png "图像 28.png")

&nbsp; 如果此时两台服务器均为新建立，且无其它写入操作，各服务器只需记录当前自己二进制日志文件及事件位置，以之作为另外的服务器复制起始位置即可

<div style="padding:0px; margin:0px; color:rgb(85,85,85); font-family:宋体,'Arial Narrow',arial,serif; font-size:14px; line-height:28px">
<div id="highlighter_191916" class="syntaxhighlighter  as3" style="width:720px; padding:0px!important; margin:0.3em 0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:relative!important; right:auto!important; top:auto!important; vertical-align:baseline!important; font-family:Consolas,'Bitstream Vera Sans Mono','Courier New',Courier,monospace!important; font-size:1em!important; min-height:auto!important">
</div>
</div>

<div class="dp-highlighter bg_sql"><div class="bar"><div class="tools">**[sql]** [view plain](# "view plain")<span data-mod="popu_168"> [copy](# "copy")<div style="position: absolute; left: 751px; top: 16975px; width: 27px; height: 15px; z-index: 99;"><embed id="ZeroClipboardMovie_10" src="http://static.blog.csdn.net/scripts/ZeroClipboard/ZeroClipboard.swf" loop="false" menu="false" quality="best" bgcolor="#ffffff" width="27" height="15" name="ZeroClipboardMovie_10" align="middle" allowscriptaccess="always" allowfullscreen="false" type="application/x-shockwave-flash" pluginspage="http://www.macromedia.com/go/getflashplayer" flashvars="id=10&amp;width=27&amp;height=15" wmode="transparent"></div></span><span data-mod="popu_169"> [print](# "print")</span>[?](# "?")</div></div>

1.  <span><span>master：查看日志文件信息&nbsp;&nbsp;</span></span>
2.  <span>mysql&gt;&nbsp;show&nbsp;master&nbsp;status;&nbsp;&nbsp;</span>
3.  <span>+<span class="comment">------------------+----------+--------------+------------------+</span><span>&nbsp;&nbsp;</span></span>
4.  <span>|&nbsp;File&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;Position&nbsp;|&nbsp;Binlog_Do_DB&nbsp;|&nbsp;Binlog_Ignore_DB&nbsp;|&nbsp;&nbsp;</span>
5.  <span>+<span class="comment">------------------+----------+--------------+------------------+</span><span>&nbsp;&nbsp;</span></span>
6.  <span>|&nbsp;mysql-bin.000001&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;107&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;</span>
7.  <span>+<span class="comment">------------------+----------+--------------+------------------+</span><span>&nbsp;&nbsp;</span></span>
8.  <span>Slave:查看服务器日志文件信息&nbsp;&nbsp;</span>
9.  <span>mysql&gt;&nbsp;show&nbsp;master&nbsp;status;&nbsp;&nbsp;</span>
10.  <span>+<span class="comment">------------------+----------+--------------+------------------+</span><span>&nbsp;&nbsp;</span></span>
11.  <span>|&nbsp;File&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;Position&nbsp;|&nbsp;Binlog_Do_DB&nbsp;|&nbsp;Binlog_Ignore_DB&nbsp;|&nbsp;&nbsp;</span>
12.  <span>+<span class="comment">------------------+----------+--------------+------------------+</span><span>&nbsp;&nbsp;</span></span>
13.  <span>|&nbsp;mysql-bin.000001&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;107&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;</span>
14.  <span>+<span class="comment">------------------+----------+--------------+------------------+</span><span>&nbsp;&nbsp;</span></span>
15.  <span>1&nbsp;row&nbsp;<span class="op">in</span><span>&nbsp;</span><span class="keyword">set</span><span>&nbsp;(0.00&nbsp;sec)&nbsp;&nbsp;</span></span><div class="save_code tracking-ad" data-mod="popu_249" style="display: none;">[![](http://static.blog.csdn.net/images/save_snippets.png)](javascript:;)</div></div><pre name="code" class="sql" style="display: none;">master：查看日志文件信息
mysql&gt; show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |      107 |              |                  |
+------------------+----------+--------------+------------------+
Slave:查看服务器日志文件信息
mysql&gt; show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |      107 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)</pre>

&nbsp;在各个服务器上建立账号和权限，来进行同步设置

<div style="padding:0px; margin:0px; color:rgb(85,85,85); font-family:宋体,'Arial Narrow',arial,serif; font-size:14px; line-height:28px">
<div id="highlighter_487350" class="syntaxhighlighter  as3" style="width:720px; padding:0px!important; margin:0.3em 0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:relative!important; right:auto!important; top:auto!important; vertical-align:baseline!important; font-family:Consolas,'Bitstream Vera Sans Mono','Courier New',Courier,monospace!important; font-size:1em!important; min-height:auto!important">
<table border="0" cellpadding="0" cellspacing="0" style="padding:0px; margin:0px auto 10px; font-size:12px; border-collapse:collapse">
<tbody style="padding:0px!important; margin:0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-size:1em!important; min-height:auto!important">
<tr style="padding:0px!important; margin:0px!important; font-size:1em!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; top:auto!important; vertical-align:baseline!important; width:auto!important; min-height:auto!important">
<td class="gutter" style="padding:0px!important; margin:0px!important; font-size:1em!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-family:Consolas,'Bitstream Vera Sans Mono','Courier New',Courier,monospace!important; min-height:auto!important; color:rgb(175,175,175)!important">
<div class="line number1 index0 alt2" style="word-wrap:normal; padding:0px 0.5em 0px 1em!important; margin:0px!important; border-width:0px 3px 0px 0px!important; border-right-style:solid!important; border-right-color:rgb(108,226,108)!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; text-align:right!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-size:1em!important; min-height:auto!important; white-space:pre!important">
1</div>
<div class="line number2 index1 alt1" style="word-wrap:normal; padding:0px 0.5em 0px 1em!important; margin:0px!important; border-width:0px 3px 0px 0px!important; border-right-style:solid!important; border-right-color:rgb(108,226,108)!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; text-align:right!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-size:1em!important; min-height:auto!important; white-space:pre!important">
2</div>
<div class="line number3 index2 alt2" style="word-wrap:normal; padding:0px 0.5em 0px 1em!important; margin:0px!important; border-width:0px 3px 0px 0px!important; border-right-style:solid!important; border-right-color:rgb(108,226,108)!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; text-align:right!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-size:1em!important; min-height:auto!important; white-space:pre!important">
3</div>
</td>
<td class="code" style="height:26px; width:692px; padding:0px!important; margin:0px!important; font-size:1em!important; line-height:1.1em!important; float:none!important; border:0px!important; bottom:auto!important; left:auto!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; top:auto!important; vertical-align:baseline!important; font-family:Consolas,'Bitstream Vera Sans Mono','Courier New',Courier,monospace!important; min-height:auto!important">
<div class="container" style="padding:0px!important; margin:0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:relative!important; right:auto!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-size:1em!important; min-height:auto!important">
<div class="line number1 index0 alt2" style="padding:0px 1em!important; margin:0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-size:1em!important; min-height:auto!important">
`master：`</div>
<div class="line number2 index1 alt1" style="padding:0px 1em!important; margin:0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-size:1em!important; min-height:auto!important">
`mysql&gt;
 GRANT REPLICATION SLAVE ON *.* TO&nbsp;``'chrislee'``@``'172.16.%.%'`&nbsp;`IDENTIFIED
 BY&nbsp;``'work'``;`</div>
<div class="line number3 index2 alt2" style="padding:0px 1em!important; margin:0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:static!important; right:auto!important; top:auto!important; vertical-align:baseline!important; width:auto!important; font-size:1em!important; min-height:auto!important">
`mysql&gt;
 flush privileges;`</div>
</div>
</td>
</tr>
</tbody>
</table>
</div>
</div>

![](http://img1.51cto.com/attachment/201305/215119511.png "图像 29.png")

<div style="padding:0px; margin:0px; color:rgb(85,85,85); font-family:宋体,'Arial Narrow',arial,serif; font-size:14px; line-height:28px">
<div id="highlighter_810002" class="syntaxhighlighter  as3" style="width:720px; padding:0px!important; margin:0.3em 0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:relative!important; right:auto!important; top:auto!important; vertical-align:baseline!important; font-family:Consolas,'Bitstream Vera Sans Mono','Courier New',Courier,monospace!important; font-size:1em!important; min-height:auto!important">
</div>
</div>

<div class="dp-highlighter bg_sql"><div class="bar"><div class="tools">**[sql]** [view plain](# "view plain")<span data-mod="popu_168"> [copy](# "copy")<div style="position: absolute; left: 751px; top: 17580px; width: 27px; height: 15px; z-index: 99;"><embed id="ZeroClipboardMovie_11" src="http://static.blog.csdn.net/scripts/ZeroClipboard/ZeroClipboard.swf" loop="false" menu="false" quality="best" bgcolor="#ffffff" width="27" height="15" name="ZeroClipboardMovie_11" align="middle" allowscriptaccess="always" allowfullscreen="false" type="application/x-shockwave-flash" pluginspage="http://www.macromedia.com/go/getflashplayer" flashvars="id=11&amp;width=27&amp;height=15" wmode="transparent"></div></span><span data-mod="popu_169"> [print](# "print")</span>[?](# "?")</div></div>

1.  <span><span>slave：&nbsp;&nbsp;</span></span>
2.  <span>mysql&gt;&nbsp;<span class="keyword">GRANT</span><span>&nbsp;REPLICATION&nbsp;SLAVE&nbsp;</span><span class="keyword">ON</span><span>&nbsp;*.*&nbsp;</span><span class="keyword">TO</span><span>&nbsp;</span><span class="string">'chrisli'</span><span>@</span><span class="string">'172.16.%.%'</span><span>&nbsp;IDENTIFIED&nbsp;</span><span class="keyword">BY</span><span>&nbsp;</span><span class="string">'work'</span><span>;&nbsp;&nbsp;</span></span>
3.  <span>mysql&gt;&nbsp;flush&nbsp;<span class="keyword">privileges</span><span>&nbsp;&nbsp;</span></span><div class="save_code tracking-ad" data-mod="popu_249">[![](http://static.blog.csdn.net/images/save_snippets.png)](javascript:;)</div></div><pre name="code" class="sql" style="display: none;">slave：
mysql&gt; GRANT REPLICATION SLAVE ON *.* TO 'chrisli'@'172.16.%.%' IDENTIFIED BY 'work';
mysql&gt; flush privileges</pre>

![](http://img1.51cto.com/attachment/201305/215153660.png "图像 30.png")

在各服务器上指定对另一台服务器为自己的主服务器即可：

<div class="dp-highlighter bg_sql"><div class="bar"><div class="tools">**[sql]** [view plain](# "view plain")<span data-mod="popu_168"> [copy](# "copy")<div style="position: absolute; left: 751px; top: 17900px; width: 27px; height: 15px; z-index: 99;"><embed id="ZeroClipboardMovie_12" src="http://static.blog.csdn.net/scripts/ZeroClipboard/ZeroClipboard.swf" loop="false" menu="false" quality="best" bgcolor="#ffffff" width="27" height="15" name="ZeroClipboardMovie_12" align="middle" allowscriptaccess="always" allowfullscreen="false" type="application/x-shockwave-flash" pluginspage="http://www.macromedia.com/go/getflashplayer" flashvars="id=12&amp;width=27&amp;height=15" wmode="transparent"></div></span><span data-mod="popu_169"> [print](# "print")</span>[?](# "?")</div></div>

1.  <span><span>server1&nbsp;&nbsp;</span></span>
2.  <span>mysql&gt;&nbsp;CHANGE&nbsp;MASTER&nbsp;<span class="keyword">TO</span><span>&nbsp;MASTER_HOST=</span><span class="string">'172.16.7.2'</span><span>,MASTER_USER=</span><span class="string">'chrisli'</span><span>,MASTER_PASSWORD=</span><span class="string">'work'</span><span>,MASTER_LOG_FILE=</span><span class="string">'mysql-bin.000001'</span><span>,MASTER_LOG_POS=344;&nbsp;&nbsp;</span></span><div class="save_code tracking-ad" data-mod="popu_249" style="display: none;">[![](http://static.blog.csdn.net/images/save_snippets.png)](javascript:;)</div></div><pre name="code" class="sql" style="display: none;">server1
mysql&gt; CHANGE MASTER TO MASTER_HOST='172.16.7.2',MASTER_USER='chrisli',MASTER_PASSWORD='work',MASTER_LOG_FILE='mysql-bin.000001',MASTER_LOG_POS=344;</pre>

![](http://img1.51cto.com/attachment/201305/215313264.png "图像 31.png")

<div style="padding:0px; margin:0px; color:rgb(85,85,85); font-family:宋体,'Arial Narrow',arial,serif; font-size:14px; line-height:28px">
<div id="highlighter_504398" class="syntaxhighlighter  as3" style="width:720px; padding:0px!important; margin:0.3em 0px!important; border:0px!important; bottom:auto!important; float:none!important; left:auto!important; line-height:1.1em!important; outline:0px!important; overflow:visible!important; position:relative!important; right:auto!important; top:auto!important; vertical-align:baseline!important; font-family:Consolas,'Bitstream Vera Sans Mono','Courier New',Courier,monospace!important; font-size:1em!important; min-height:auto!important">
</div>
</div>

<div class="dp-highlighter bg_sql"><div class="bar"><div class="tools">**[sql]** [view plain](# "view plain")<span data-mod="popu_168"> [copy](# "copy")<div style="position: absolute; left: 751px; top: 18388px; width: 27px; height: 15px; z-index: 99;"><embed id="ZeroClipboardMovie_13" src="http://static.blog.csdn.net/scripts/ZeroClipboard/ZeroClipboard.swf" loop="false" menu="false" quality="best" bgcolor="#ffffff" width="27" height="15" name="ZeroClipboardMovie_13" align="middle" allowscriptaccess="always" allowfullscreen="false" type="application/x-shockwave-flash" pluginspage="http://www.macromedia.com/go/getflashplayer" flashvars="id=13&amp;width=27&amp;height=15" wmode="transparent"></div></span><span data-mod="popu_169"> [print](# "print")</span>[?](# "?")</div></div>

1.  <span><span>server2：&nbsp;&nbsp;</span></span>
2.  <span>mysql&gt;&nbsp;CHANGE&nbsp;MASTER&nbsp;<span class="keyword">TO</span><span>&nbsp;MASTER_HOST=</span><span class="string">'172.16.7.1'</span><span>,MASTER_USER=</span><span class="string">'chrislee'</span><span>,MASTER_PASSWORD=</span><span class="string">'work'</span><span>,MASTER_LOG_FILE=</span><span class="string">'mysql-bin.000001'</span><span>,MASTER_LOG_POS=345;&nbsp;&nbsp;</span></span><div class="save_code tracking-ad" data-mod="popu_249">[![](http://static.blog.csdn.net/images/save_snippets.png)](javascript:;)</div></div><pre name="code" class="sql" style="display: none;">server2：
mysql&gt; CHANGE MASTER TO MASTER_HOST='172.16.7.1',MASTER_USER='chrislee',MASTER_PASSWORD='work',MASTER_LOG_FILE='mysql-bin.000001',MASTER_LOG_POS=345;</pre>

![](http://img1.51cto.com/attachment/201305/215339601.png "图像 33.png")

双主架构配置基本完成，下面在各自上面启动复制进程吧~并进行测试：

![](http://img1.51cto.com/attachment/201305/215419531.png "图像 34.png")

![](http://img1.51cto.com/attachment/201305/215419915.png "图像 35.png")

转载自：http://chrinux.blog.51cto.com/6466723/1204586

</div>

<!-- Baidu Button BEGIN -->

<div class="bdsharebuttonbox tracking-ad bdshare-button-style0-16" style="float: right;" data-mod="popu_172" data-bd-bind="1468314371856">
[](#)
[](# "分享到QQ空间")
[](# "分享到新浪微博")
[](# "分享到腾讯微博")
[](# "分享到人人网")
[](# "分享到微信")
</div>
<script>window._bd_share_config = { "common": { "bdSnsKey": {}, "bdText": "", "bdMini": "1", "bdMiniList": false, "bdPic": "", "bdStyle": "0", "bdSize": "16" }, "share": {} }; with (document) 0[(getElementsByTagName('head')[0] || body).appendChild(createElement('script')).src = 'http://bdimg.share.baidu.com/static/api/js/share.js?v=89860593.js?cdnversion=' + ~(-new Date() / 36e5)];</script>
<!-- Baidu Button END -->

   <link rel="stylesheet" href="http://static.blog.csdn.net/css/blog_detail.css">

<!--172.16.140.11-->

<!-- Baidu Button BEGIN -->
<script type="text/javascript" id="bdshare_js" data="type=tools&amp;uid=1536434" src="http://bdimg.share.baidu.com/static/js/bds_s_v2.js?cdnversion=407866"></script>

<script type="text/javascript">
    document.getElementById("bdshell_js").src = "http://bdimg.share.baidu.com/static/js/shell_v2.js?cdnversion=" + Math.ceil(new Date()/3600000)
</script>
<!-- Baidu Button END -->

        <div id="digg" articleid="9339621">
            <dl id="btnDigg" class="digg digg_disable" onclick="btndigga();">

                 <dt>顶</dt>
                <dd>1</dd>
            </dl>

            <dl id="btnBury" class="digg digg_disable" onclick="btnburya();">

                  <dt>踩</dt>
                <dd>0</dd>               
            </dl>

        </div>
     <div class="tracking-ad" data-mod="popu_222">[&nbsp;](javascript:void(0);)   </div>
    <div class="tracking-ad" data-mod="popu_223"> [&nbsp;](javascript:void(0);)</div>
    <script type="text/javascript">
                function btndigga() {
                    $(".tracking-ad[data-mod='popu_222'] a").click();
                }
                function btnburya() {
                    $(".tracking-ad[data-mod='popu_223'] a").click();
                }
            </script>

*   <span onclick="_gaq.push(['_trackEvent','function', 'onclick', 'blog_articles_shangyipian']);location.href='/goustzhu/article/details/8784870';">上一篇</span>[经典计算机书籍](/goustzhu/article/details/8784870)
*   <span onclick="_gaq.push(['_trackEvent','function', 'onclick', 'blog_articles_xiayipian']);location.href='/goustzhu/article/details/9820393';">下一篇</span>[特殊二叉树程序](/goustzhu/article/details/9820393)

    <div style="clear:both; height:10px;"></div>

</div>
