---
title: Flowable 文档 （V 6.3.1 ）03
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

# 3.配置
## 3.1. 创建ProcessEngine
Flowable流程引擎通过名为的XML文件配置flowable.cfg.xml。请注意，如果您使用Spring样式构建流程引擎，则不适用。

获得a的最简单方法ProcessEngine是使用org.flowable.engine.ProcessEngines该类：
```java?linenums
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine()
```
这将flowable.cfg.xml在类路径中查找文件，并根据该文件中的配置构造引擎。以下代码段显示了一个示例配置。以下部分将详细介绍配置属性。
```xml?linenums
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="processEngineConfiguration" class="org.flowable.engine.impl.cfg.StandaloneProcessEngineConfiguration">

    <property name="jdbcUrl" value="jdbc:h2:mem:flowable;DB_CLOSE_DELAY=1000" />
    <property name="jdbcDriver" value="org.h2.Driver" />
    <property name="jdbcUsername" value="sa" />
    <property name="jdbcPassword" value="" />

    <property name="databaseSchemaUpdate" value="true" />

    <property name="asyncExecutorActivate" value="false" />

    <property name="mailServerHost" value="mail.my-corp.com" />
    <property name="mailServerPort" value="5025" />
  </bean>

</beans>
```
请注意，配置XML实际上是一个Spring配置。这并不意味着Flowable只能在Spring环境中使用！我们只是在内部利用Spring的解析和依赖注入功能来构建引擎。

ProcessEngineConfiguration对象也可以使用配置文件以编程方式创建。也可以使用不同的bean id（例如，参见第3行）。
```java?linenums
ProcessEngineConfiguration.
  createProcessEngineConfigurationFromResourceDefault();
  createProcessEngineConfigurationFromResource(String resource);
  createProcessEngineConfigurationFromResource(String resource, String beanName);
  createProcessEngineConfigurationFromInputStream(InputStream inputStream);
  createProcessEngineConfigurationFromInputStream(InputStream inputStream, String beanName);
```
也可以不使用配置文件，并根据默认值创建配置（有关更多信息，请参阅不同的受支持类）。
```java?linenums
ProcessEngineConfiguration.createStandaloneProcessEngineConfiguration();
ProcessEngineConfiguration.createStandaloneInMemProcessEngineConfiguration();
```
如果需要，所有这些ProcessEngineConfiguration.createXXX()方法都返回ProcessEngineConfiguration可以进一步调整的方法。调用buildProcessEngine()操作后，ProcessEngine创建一个：
```java?linenums
ProcessEngine processEngine = ProcessEngineConfiguration.createStandaloneInMemProcessEngineConfiguration()
  .setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_FALSE)
  .setJdbcUrl("jdbc:h2:mem:my-own-db;DB_CLOSE_DELAY=1000")
  .setAsyncExecutorActivate(false)
  .buildProcessEngine();
```
## 3.2. ProcessEngineConfiguration bean
在flowable.cfg.xml必须包含有ID的Bean 'processEngineConfiguration'。
```xml?linenums
<bean id="processEngineConfiguration" class="org.flowable.engine.impl.cfg.StandaloneProcessEngineConfiguration">
```
然后使用这个bean来构造ProcessEngine。有多个类可用于定义processEngineConfiguration。这些类表示不同的环境，并相应地设置默认值。最佳做法是选择最适合您环境的类，以最小化配置引擎所需的属性数。目前提供以下课程：

 - org.flowable.engine.impl.cfg.StandaloneProcessEngineConfiguration：流程引擎以独立方式使用。Flowable将负责所有交易。默认情况下，仅在引擎引导时检查数据库（如果没有Flowable架构或架构版本不正确，则抛出异常）。
 - org.flowable.engine.impl.cfg.StandaloneInMemProcessEngineConfiguration：这是一个用于单元测试的便利类。Flowable将负责所有交易。默认情况下使用H2内存数据库。引擎启动和关闭时将创建和删除数据库。使用此功能时，可能不需要其他配置（例如，使用作业执行程序或邮件功能时除外）。
 - org.flowable.spring.SpringProcessEngineConfiguration：在Spring环境中使用流程引擎时使用。有关更多信息，请参阅Spring集成部分。
 - org.flowable.engine.impl.cfg.JtaProcessEngineConfiguration：在引擎以独立模式运行时使用JTA事务。
 
 ## 3.3. 数据库配置
 有两种方法可以配置Flowable引擎将使用的数据库。第一个选项是定义数据库的JDBC属性：
 
 - jdbcUrl：数据库的JDBC URL。
 - jdbcDriver：实现特定数据库类型的驱动程序。
 - jdbcDriver：实现特定数据库类型的驱动程序。
 - jdbcPassword：连接数据库的密码。
 
 基于提供的JDBC属性构造的数据源将具有默认的MyBatis连接池设置。可以选择设置以下属性来调整该连接池（取自MyBatis文档）：
 
 - jdbcMaxActiveConnections：任何时候连接池最多可以包含的活动连接数。默认值为10。
 - jdbcMaxIdleConnections：任何时候连接池最多可以包含的空闲连接数。
 - jdbcMaxCheckoutTime：在强制返回连接之前，可以从连接池中检出连接的时间量（以毫秒为单位）。默认值为20000（20秒）。
 - jdbcMaxWaitTime：这是一个低级别设置，它使池有机会打印日志状态，并在它花费异常时间的情况下重新尝试获取连接（以避免在池配置错误时永远无声地失败）默认值是20000（20秒）。
 
 示例数据库配置：
 
 ```xml?linenums
 <property name="jdbcUrl" value="jdbc:h2:mem:flowable;DB_CLOSE_DELAY=1000" />
<property name="jdbcDriver" value="org.h2.Driver" />
<property name="jdbcUsername" value="sa" />
<property name="jdbcPassword" value="" />
 ```
 我们的基准测试表明，在处理大量并发请求时，MyBatis连接池不是最有效或最有弹性的。因此，建议我们使用一个javax.sql.DataSource实现并将其注入流程引擎配置（例如HikariCP，Tomcat JDBC连接池等）：
 ```xml?linenums
 <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" >
  <property name="driverClassName" value="com.mysql.jdbc.Driver" />
  <property name="url" value="jdbc:mysql://localhost:3306/flowable" />
  <property name="username" value="flowable" />
  <property name="password" value="flowable" />
  <property name="defaultAutoCommit" value="false" />
</bean>

<bean id="processEngineConfiguration" class="org.flowable.engine.impl.cfg.StandaloneProcessEngineConfiguration">

  <property name="dataSource" ref="dataSource" />
  ...
 ```

