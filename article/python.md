###系统自带的python
rpm -qa python：python-2.6.6-29.el6_2.2.x86_64
##1. 下载最新源代码包
http://www.python.org/download/releases/2.7/

[root@user ~]# ls
Python-2.5.2  Python-2.7.tgz  python

##2. 安装 <br>
[root@user ~]# mkdir /usr/local/python27         （创建安装目录）<br>
[root@user Python-2.7]# ./configure --prefix=/usr/local/python27<br>
[root@user Python-2.7]# make                        <br>            
[root@user Python-2.7]# make install<br>

##3. 创建链接<br>
[root@user ~]# mv /usr/bin/python /usr/bin/python_bak （保存原来的版本）<br>
[root@user ~]# ln -s /usr/local/python27/bin/python /usr/bin<br>
