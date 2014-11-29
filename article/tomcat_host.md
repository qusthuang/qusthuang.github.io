###配置虚拟主机<br>

在文件server.xml中找到</Host>标签；在</Host>标签之后新建如下信息：<br>
           <Host name="mycompany1"  appBase="C:\mycompany1"

            unpackWARs="true" autoDeploy="true"

            xmlValidation="false" xmlNamespaceAware="false">

           <Context path="/jQueryTest" docBase="jQueryTest" reloadable="true"/>

	   </Host>
<br>
**说明：**
【mycompany1】为你的主机的名称或域名；如果想任意取个名字而又想在本机测试的话，请修改host文件在WINDOWS\system32\drivers\etc\host文件(首先把系统隐藏文件都显示出来，host是系统隐藏文件，host文件为只读，在改之前将host文件的只读属性前的勾去掉）添加一条：
mycompany1   127.0.0.1

如果本机已经申请了域名，那么只需要修改server.xml就可以。 
