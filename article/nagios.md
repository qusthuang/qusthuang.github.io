###1.添加用户与组：<br>
   useradd nagios<br>
   groupadd nagios<br>
   useradd –G nagios nagios<br>
###2.安装nagios：<br>
tar zxvf nagios-3.5.0.tar.gz<br>
cd nagios<br>
 ./configure --prefix=/usr/local/nagios --with-nagios-user=nagios --with-nagios-group=nagios<br>
Make all<br>
make install<br>
Make install-init<br>
Make install-commmode<br>
Make-install-config<br>
Make install-webconf（失败，没有找到httpd.conf<br>
手动添加apache支持  ）<br>
 vi httpd.conf，添加以下内容<br>
ScriptAlias /nagios/cgi-bin /usr/local/nagios/sbin<br>
<Directory "/usr/local/nagios/sbin">
    Options ExecCGI
    AllowOverride None
    Order allow,deny
    Allow from all
    AuthName "Nagios Access"
    AuthType Basic
    AuthUserFile /usr/local/nagios/etc/htpasswd
    Require valid-user
</Directory>
 
Alias /nagios /usr/local/nagios/share<br>
 
<Directory "/usr/local/nagios/share">
    Options None
    AllowOverride None
    Order allow,deny
    Allow from all
    AuthName "Nagios Access"
    AuthType Basic
    AuthUserFile /usr/local/nagios/etc/htpasswd
    Require valid-user
</Directory><br>
添加用户密码<br>
/usr/local/apache2/bin/htpasswd  ‐ c  /usr/local/nagios/etc/htpasswd  nagios <br>
 
New   password:  (输入12345)  <br>
Re‐ type   new   password:  (再输入12345）<br>
 
###3.  启动nagios  启动项<br><br>
把Nagios 加入到服务列表中以使之在系统启动时自动启动 <br>
chkconfig  ‐‐ add   nagios  <br>
chkconfig   nagios   on <br>
验证Nagios 的样例配置文件 <br>
/usr/local/nagios/bin/nagios  ‐ v  /usr/local/nagios/etc/nagios.cfg  <br>
如果没有报错，可以启动Nagios 服务 <br>
service   nagios   start  <br>
 
###4.服务端安装插件nagios-plugins<br><br>
tar zxvf nagios-plugins-1.4.16.tar.gz<br>
cd nagios-plugins-1.4.16<br>
./configure<br>
make <br>
make  install  <br>
 
chown   nagios.nagios  /usr/local/nagios  <br>
chown  ‐ R  nagios.nagios  /usr/local/nagios/libexec <br>
 
###5.服务端安装nrpe插件<br><br>
 
tar ‐ zxvf   nrpe ‐***.tar.gz  <br>
cd   nrpe <br>
./configure  <br>
make  all  <br>
make  install -plugin <br>
测试连接 /usr/local/nagios/libexec/check_nrpe -H localhost<br>
 
###6.客户端安装plugin和nrpe<br><br>
 
useradd nagios<br>
groupadd nagios<br>
 
 
tar zxvf nagios-plugins-1.4.16.tar.gz<br>
cd nagios-plugins-1.4.16<br>
./configure<br>
 
make <br>
make  install  <br>
 
chown   nagios.nagios  /usr/local/nagios  <br>
chown  ‐ R  nagios.nagios  /usr/local/nagios/libexec <br>
 
 
tar ‐ zxvf   nrpe ‐***.tar.gz  <br>
cd   nrpe <br>
./configure  <br>
make  all  <br>
make  install -plugin <br>
make  install -daemon  <br>
make  install -daemon- config<br>
 
###7.客户端配置check_nrpe（被动方式）<br>
vi /usr/local/nagios/etc/objects/command.cfg<br>
 
define command{<br>
        command_name check_nrpe<br>
        command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$<br>
        }<br>
vi /usr/local/nagios/etc/nagios.cfg<br>
cfg_file=/usr/local/nagios/etc/objects/mylinux.cfg(添加被监控机器的配置文件）<br>
vi /usr/local/nagios/etc/objects/mylinux.cfg<br>
define host{<br>
   use linux-server <br>
   host_name oracle<br>
   alias oracle 10g<br>
   address 192.168.1.132 被监控机器ip<br>
}<br>
define service{<br>
   use generic-service<br>
   host_name oracle<br>
   service_description proc<br>
   check_command check_nrpe!check_total_procs!20!10<br>
 
 
}<br>
###8.客户端修改配置文件，启动nrpe<br><br>
vi /usr/local/nagios/etc/nrpe.cfg<br>
allowed_hosts=127.0.0.1,192.168.1.165<br>
/usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d<br>
tcp      0     0 0.0.0.0:5666            0.0.0.0:*               LISTEN      27753/nrpe<br>
 
###9.访问 http:/165主机ip/nagios,查看132（oracle）服务<br>
![](http://dl.iteye.com/upload/attachment/0084/4052/486a92dc-be16-30e5-9bc1-183eea62fa9e.png)
