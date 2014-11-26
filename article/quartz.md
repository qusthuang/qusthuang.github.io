```Java
<bean id="schedulerFactory" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">   
        <property name="triggers">   
            <list>   
                <ref local="minCronTrigger"/>   
                <ref local="dayCronTrigger"/>
            </list>   
        </property>   
</bean> 
 
 <bean id="minCronTrigger" class="org.springframework.scheduling.quartz.CronTriggerBean" >   
         <property name="jobDetail" ref="minSchedulerJob"/>   
         <property name="cronExpression">   
             <value>0 0/5 * * * ?</value>   
         </property>   
     </bean>  
     
    <bean id="dayCronTrigger" class="org.springframework.scheduling.quartz.CronTriggerBean" >   
         <property name="jobDetail" ref="DaySchedulerJob"/>   
         <property name="cronExpression">   
             <value>0 0 1 * * ?</value>   
         </property>   
     </bean>   
     <bean id="BackupCustMinThread" class="com.huang.cacti.backup.BackupCustMinThread"></bean>
     
     <bean id="minSchedulerJob" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">   
        <property name="targetObject" ref="BackupCustMinThread"/>   
        <property name="targetMethod" value="run"/>   
        <property name="concurrent" value="false"/>   
 </bean>
 
 <bean id="BackupCustDayThread" class="com.huang.cacti.backup.BackupCustDayThread"></bean>
 
 <bean id="DaySchedulerJob" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">   
        <property name="targetObject" ref="BackupCustDayThread"/>   
        <property name="targetMethod" value="run"/>   
        <property name="concurrent" value="false"/>   
 </bean>
