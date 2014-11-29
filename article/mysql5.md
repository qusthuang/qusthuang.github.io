     makedir /usr/local/mysql
     shell> groupadd mysql
     shell> useradd -g mysql mysql
     shell> tar -xvzf mysql-5.0.22.tar.gz
     shell> cd mysql-5.0.22
     shell> ./configure --prefix=/usr/local/mysql
     shell> make
     shell> make install
     shell> make clean      
     
     shell> cd /usr/local/mysql
     shell> bin/mysql_install_db --user=mysql 
     shell> chown -R root  .    
     shell> chown -R mysql var
     shell> chgrp -R mysql .
     shell> cp share/mysql/my-medium.cnf  /etc/my.cnf
     shell> bin/mysqld_safe --user=mysql & 启动服务端
     shell>bin /mysqladmin -u root password 123 修改root的密码
 

     # cp share/mysql/mysql.server /etc/rc.d/init.d/mysqld //开机自动启动 mysql。
     # chmod 755 /etc/rc.d/init.d/mysqld
     # chkconfig --add mysqld
     # service mysqld start
