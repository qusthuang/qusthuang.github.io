
\<Resource name="jdbc/cbb_connection" auth="Container"
   type="javax.sql.DataSource"
   driverClassName="com.microsoft.sqlserver.jdbc.SQLServerDriver"
   url="jdbc:sqlserver://localhost:1433;DatabaseName=XXXXX"
   username="XX"
   password="XXXXXX"
   maxActive="100"
   maxIdle= "2"
   maxWait="1000"
   poolPreparedStatements="true"
   defaultAutoCommit="true" /><br>

###最后解决方案是将上面的配置放到tomcat 的context.xml里配置，而不要放到server.xml里配置。另外驱动是放置到tomcat lib下的。
