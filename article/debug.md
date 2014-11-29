##1.配置环境变量<br>
   export JPDA_ADDRESS=9999（默认8000）
##2.启动服务器上的tomcat<br>
    sh catalina.sh jpda start
    netstat -tnpl|grep 9999查看是否启动tomcat
##3.打开debug configuration窗口，填入调试信息，debug<br>
![](http://dl2.iteye.com/upload/attachment/0089/8214/09934512-f8fe-3ae5-b5b2-a0ebed8459ec.png)
##4.连接成功后，在eclipse中设置断点，像在本地调试一样，在debug perspective 可以看到断点信息<br>
![](http://dl2.iteye.com/upload/attachment/0089/8212/26309336-2614-3115-831f-f93af82942b1.png)
