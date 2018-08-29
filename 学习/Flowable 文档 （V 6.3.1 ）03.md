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
 请注意，Flowable不附带允许您定义此类数据源的库。因此，您需要确保库位于类路径中。

无论您使用的是JDBC还是数据源方法，都可以设置以下属性：

 - databaseType：通常不需要指定此属性，因为它是从数据库连接元数据中自动检测到的。只应在自动检测失败时指定。可能的值：{h2，mysql，oracle，postgres，mssql，db2}。此设置将确定将使用哪些创建/删除脚本和查询。请参阅该支持的数据库部分用于哪些类型支持的概述。
 - databaseSchemaUpdate：设置策略以在流程引擎启动和关闭时处理数据库模式。
  - false （默认值）：在创建流程引擎时检查库模式的版本，如果版本不匹配则抛出异常。
  - true：在构建流程引擎时，执行检查并在必要时执行模式更新。如果架构不存在，则创建它。
  - create-drop：在创建流程引擎时创建架构，并在关闭流程引擎时删除架构。
  
## 3.4. JNDI数据源配置
默认情况下，Flowable的数据库配置包含在每个Web应用程序的WEB-INF / classes中的db.properties文件中。这并不总是理想的，因为它要求用户修改Flowable源中的db.properties并重新编译WAR文件，或者在每次部署时分解WAR并修改db.properties。

通过使用JNDI（Java命名和目录接口）获取数据库连接，连接完全由Servlet容器管理，并且可以在WAR部署之外管理配置。与db.properties文件提供的连接参数相比，这还允许更多地控制连接参数。
### 3.4.1. 组态
根据您使用的servlet容器应用程序，JNDI数据源的配置会有所不同。以下说明适用于Tomcat，但对于其他容器应用程序，请参阅容器应用程序的文档。

如果使用Tomcat，所述JNDI资源被配置$ CATALINA_BASE内/ CONF / [引擎] / [主机名] / [warname] .XML（用于可流动UI这通常将是$ CATALINA_BASE / CONF /卡塔利娜/本地主机/可流动的应用内。 XML）。首次部署应用程序时，将从Flowable WAR文件复制默认上下文，因此如果已存在，则需要替换它。例如，要更改JNDI资源以便应用程序连接到MySQL而不是H2，请将文件更改为以下内容：
```xml?linenums
<?xml version="1.0" encoding="UTF-8"?>
    <Context antiJARLocking="true" path="/flowable-app">
        <Resource auth="Container"
            name="jdbc/flowableDB"
            type="javax.sql.DataSource"
            description="JDBC DataSource"
            url="jdbc:mysql://localhost:3306/flowable"
            driverClassName="com.mysql.jdbc.Driver"
            username="sa"
            password=""
            defaultAutoCommit="false"
            initialSize="5"
            maxWait="5000"
            maxActive="120"
            maxIdle="5"/>
        </Context>
```
### 3.4.2. JNDI属性
要配置JNDI数据源，请在Flowable UI的属性文件中使用以下属性：

- spring.datasource.jndi-name =：数据源的JNDI名称。

- datasource.jndi.resourceRef：设置是否在J2EE容器中进行查找，例如，如果JNDI名称尚未包含前缀，则需要添加前缀“java：comp / env /”。默认为“true”。

## 3.5. 支持的数据库
下面列出了Flowable用于引用数据库的类型（区分大小写！）。

|  数据库类型   |  示例JDBC URL   |  笔记   |
| :-- | :-- | :-- |
|   h2  |  jdbc:h2:tcp://localhost/flowable   | Default configured database    |
|  mysql   |  jdbc:mysql://localhost:3306/flowable?autoReconnect=true   |  Tested using mysql-connector-java database driver   |
|   oracle  |   jdbc:oracle:thin:@localhost:1521:xe  |     |
|  postgres   | jdbc:postgresql://localhost:5432/flowable    |     |
|   db2  |     |  jdbc:db2://localhost:50000/flowable   |
|  mssql   |  jdbc:sqlserver://localhost:1433;databaseName=flowable (jdbc.driver=com.microsoft.sqlserver.jdbc.SQLServerDriver) OR jdbc:jtds:sqlserver://localhost:1433/flowable (jdbc.driver=net.sourceforge.jtds.jdbc.Driver)   |   Tested using Microsoft JDBC Driver 4.0 (sqljdbc4.jar) and JTDS Driver  |


