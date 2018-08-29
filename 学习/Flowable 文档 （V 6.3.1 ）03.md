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
|  mssql   |  jdbc:sqlserver://localhost:1433;<br>databaseName=flowable <br>(jdbc.driver=com.microsoft.sqlserver.jdbc.SQLServerDriver) <br>OR <br>jdbc:jtds:sqlserver://localhost:1433/flowable (jdbc.driver=net.sourceforge.jtds.jdbc.Driver)   |   Tested using Microsoft JDBC Driver 4.0 (sqljdbc4.jar) and JTDS Driver  |
## 3.6. 创建数据库表
为数据库创建数据库表的最简单方法是：
- 将可流动引擎JAR添加到类路径中

- 添加合适的数据库驱动程序

- 将Flowable配置文件（flowable.cfg.xml）添加到类路径中，指向您的数据库（请参阅数据库配置部分）

- 执行DbSchemaCreate类的main方法

但是，通常只有数据库管理员才能在数据库上执行DDL语句。在生产系统上，这也是最明智的选择。SQL DDL语句可以在Flowable下载页面上或Flowable分发文件夹内的database子目录中找到。脚本也在引擎JAR（flowable-engine-x.jar）中，在org / flowable / db / create包中（drop文件夹包含drop语句）。SQL文件的形式
```
flowable.{db}.{create|drop}.{type}.sql
```
其中DB是任何的支持的数据库和类型是：
- engine：引擎执行所需的表。需要。

- history：包含历史记录和审计信息的表。可选：历史记录级别设置为none时不需要。请注意，这还将禁用将数据存储在历史数据库中的某些功能（例如注释任务）。

MySQL用户注意事项：低于5.6.4的MySQL版本不支持精度为毫秒级的时间戳或日期。更糟糕的是，某些版本在尝试创建此类列时会抛出异常，但其他版本则不会。在进行自动创建/升级时，引擎会在执行时更改DDL。使用DDL文件方法时，常规版本和包含mysql55的特殊文件都可用（这适用于低于5.6.4的任何内容）。后一个文件将具有没有毫秒精度的列类型。

具体来说，以下适用于MySQL版本：

- <5.6：没有毫秒精度。DDL文件可用（查找包含mysql55的文件）。自动创建/更新将开箱即用。

- 5.6.0 - 5.6.3：没有毫秒级精度。自动创建/更新将不起作用。建议无论如何都要升级到更新的数据库版本。如果确实需要，可以使用mysql 5.5的 DDL文件。

- 5.6.4+：可用毫秒精度。DDL文件可用（包含mysql的默认文件）。自动创建/更新开箱即用。

请注意，如果稍后升级MySQL数据库并且已经创建/升级了Flowable表，则必须手动完成列类型更改！
## 3.7. 数据库表名称解释
Flowable的数据库名称都以ACT_开头。第二部分是表的用例的双字符标识。此用例也将大致匹配服务API。

- ACT_RE_ *：RE代表repository。具有此前缀的表包含静态信息，例如流程定义和流程资源（图像，规则等）。

- ACT_RU_ *：RU代表runtime。这些是包含流程实例，用户任务，变量，作业等的运行时数据的运行时表。Flowable仅在流程实例执行期间存储运行时数据，并在流程实例结束时删除记录。这使运行时表保持小而快。

- ACT_HI_ *：HI代表history。这些是包含历史数据的表，例如过去的流程实例，变量，任务等。

- ACT_GE_ *：general数据，用于各种用例。

## 3.8. 数据库升级
在运行升级之前，请确保备份数据库（使用数据库备份功能）。

默认情况下，每次创建流程引擎时都会执行版本检查。这通常在应用程序或Flowable Web应用程序的引导时发生一次。如果Flowable库注意到库版本和Flowable数据库表的版本之间的差异，则抛出异常。

要升级，必须首先将以下配置属性放在flowable.cfg.xml配置文件中：
```xml?linenums
<beans >

  <bean id="processEngineConfiguration"
      class="org.flowable.engine.impl.cfg.StandaloneProcessEngineConfiguration">
    <!-- ... -->
    <property name="databaseSchemaUpdate" value="true" />
    <!-- ... -->
  </bean>

</beans>
```
另外，为数据库添加适合数据库驱动程序的类路径。升级应用程序中的Flowable库。或者启动Flowable的新版本并将其指向包含旧版本数据的数据库。随着databaseSchemaUpdate设置为true可流动会自动的第一次，当它注意到图书馆和数据库架构不同步升级数据库架构到最新版本。

或者，您也可以运行升级DDL语句。也可以在Flowable下载页面上运行可用的升级数据库脚本。
## 3.9. Job Executor（从6.0.0版开始）
Flowable v5的异步执行程序是Flowable V6中唯一可用的作业执行程序，因为它是一种在Flowable引擎中执行异步作业的性能更高且更加数据库友好的方式。V6中不再提供Flowable 5的旧作业执行程序。可以在用户指南的高级部分中找到更多信息。

此外，如果在Java EE 7下运行，ManagedAsyncJobExecutor则可以使用JSR-236兼容容器来管理线程。为了启用它们，应该在配置中传递线程工厂，如下所示：
```xml?linenums
<bean id="threadFactory" class="org.springframework.jndi.JndiObjectFactoryBean">
   <property name="jndiName" value="java:jboss/ee/concurrency/factory/default" />
</bean>

<bean id="customJobExecutor" class="org.flowable.engine.impl.jobexecutor.ManagedAsyncJobExecutor">
   <!-- ... -->
   <property name="threadFactory" ref="threadFactory" />
   <!-- ... -->
</bean>
```
如果未指定线程工厂，则托管实现将回退到其默认对应项。
## 3.10. 作业执行者激活
这AsyncExecutor是一个管理线程池以触发计时器和其他异步任务的组件。其他实现也是可能的（例如，使用消息队列，请参阅用户指南的高级部分）。

默认情况下，AsyncExecutor未激活且未启动。通过以下配置，可以与Flowable Engine一起启动异步执行程序。
```xml?linenums
	<property name="asyncExecutorActivate" value="true" />
```
属性asyncExecutorActivate指示Flowable引擎在启动时启动Async执行程序。
## 3.11. 邮件服务器配置
配置邮件服务器是可选的。Flowable支持在业务流程中发送电子邮件。要实际发送电子邮件，需要有效的SMTP邮件服务器配置。有关配置选项，请参阅电子邮件任务。
## 3.12. 历史配置
自定义历史存储的配置是可选的。这允许您调整影响引擎历史功能的设置。有关详细信息，请参阅历史配置
```xml?linenums
	<property name="history" value="audit" />
```
## 3.13. 异步历史记录配置
[实验]从Flowable 6.1.0开始，添加了异步历史记录功能。启用异步历史记录时，历史数据将由历史作业执行程序保留，而不是作为运行时执行持久性的一部分的同步持久性。有关详细信息，请参阅异步历史记录配置
```xml?linenums
<property name="asyncHistoryEnabled" value="true" />
```
## 3.14. 在表达式和脚本中公开配置bean
默认情况下，您在flowable.cfg.xml配置或您自己的Spring配置文件中指定的所有Bean 都可用于表达式和脚本。如果要限制配置文件中bean的可见性，可以配置beans在流程引擎配置中调用的属性。beans属性ProcessEngineConfiguration是一个地图。指定该属性时，表达式和脚本只能看到该映射中指定的bean。暴露的bean将使用您在地图中指定的名称公开。
## 3.15. 部署缓存配置
所有流程定义都被缓存（在解析之后），以避免每次需要流程定义时都访问数据库，并且流程定义数据不会更改。默认情况下，此缓存没有限制。要限制流程定义缓存，请添加以下属性：
```xml?linenums
<property name="processDefinitionCacheLimit" value="10" />
```
设置此属性会将默认哈希映射缓存与具有提供的硬限制的LRU缓存交换。当然，此属性的最佳值取决于所存储的流程定义的总量以及所有运行时流程实例在运行时实际使用的流程定义的数量。

您也可以注入自己的缓存实现。这必须是实现org.flowable.engine.impl.persistence.deploy.DeploymentCache接口的bean：
```xml?linenums
<property name="processDefinitionCache">
  <bean class="org.flowable.MyCache" />
</property>
```
有一个类似的属性knowledgeBaseCacheLimit，knowledgeBaseCache用于配置规则缓存。只有在流程中使用规则任务时才需要这样做。
## 3.16. 记录
所有日志记录（flowable，spring，mybatis，...）都通过SLF4J进行路由，并允许选择您选择的日志记录实现。
默认情况下，可流动引擎依赖项中不存在SFL4J绑定JAR，应在项目中添加此JAR以使用您选择的日志记录框架。如果没有添加实现JAR，SLF4J将使用NOP记录器，而不记录任何内容，除了警告不会记录任何内容。有关这些绑定的更多信息http://www.slf4j.org/codes.html#StaticLoggerBinder。
使用Maven，添加例如这样的依赖（这里使用log4j），请注意您仍然需要添加一个版本：
```xml?linenums
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
</dependency>
```
flowable-ui和flowable-rest webapps配置为使用Log4j绑定。运行所有flowable- *模块的测试时也使用Log4j。

在类路径中使用带有commons-logging的容器时的重要注意事项：为了通过SLF4J路由弹簧日志记录，使用了一个桥接器（请参阅http://www.slf4j.org/legacy.html#jclOverSLF4J）。如果您的容器提供了commons-logging实现，请按照此页面上的说明操作：http://www.slf4j.org/codes.html#release以确保稳定性。

使用Maven时的示例（省略版本）：
```xml?linenums
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>jcl-over-slf4j</artifactId>
</dependency>
```
## 3.17. 映射诊断上下文
Flowable支持SLF4j的Mapped Diagnostic Contexts功能。此基本信息将与将要记录的内容一起传递给基础记录器：

- processDefinition Id为mdcProcessDefinitionID

- processInstance Id为mdcProcessInstanceID

- 执行ID为mdcExecutionId

默认情况下，不记录任何此信息。可以将记录器配置为以所需格式显示它们，这是常用记录消息的补充。例如，在Log4j中，以下示例布局定义使记录器显示上述信息：
```
log4j.appender.consoleAppender.layout.ConversionPattern=ProcessDefinitionId=%X{mdcProcessDefinitionID}
executionId=%X{mdcExecutionId} mdcProcessInstanceID=%X{mdcProcessInstanceID} mdcBusinessKey=%X{mdcBusinessKey} %m%n
```
例如，当日志包含需要通过日志分析器实时检查的信息时，这非常有用。
## 3.18. 事件处理程序
Flowable引擎中的事件机制允许您在引擎内发生各种事件时收到通知。查看所有支持的事件类型，以获取可用事件的概述。

可以为某些类型的事件注册侦听器，而不是在调度任何类型的事件时收到通知。您可以通过配置添加引擎范围的事件侦听器，使用API​​在运行时添加引擎范围的事件侦听器，或者将事件侦听器添加到BPMN XML中的特定流程定义。

调度的所有事件都是子类型org.flowable.engine.common.api.delegate.event.FlowableEvent。该事件自曝（如果有的话）type，executionId，processInstanceId和processDefinitionId。某些事件包含与发生的事件相关的其他上下文，有关其他有效内容的更多信息可以在所有支持的事件类型的列表中找到。
### 3.18.1 事件监听器实现
事件监听器的唯一要求是实现org.flowable.engine.delegate.event.FlowableEventListener。下面是一个侦听器的示例实现，它将收到的所有事件输出到标准输出，但与作业执行相关的事件除外：
```java?linenums
public class MyEventListener implements FlowableEventListener {

  @Override
  public void onEvent(FlowableEvent event) {
    switch (event.getType()) {

      case JOB_EXECUTION_SUCCESS:
        System.out.println("A job well done!");
        break;

      case JOB_EXECUTION_FAILURE:
        System.out.println("A job has failed...");
        break;

      default:
        System.out.println("Event received: " + event.getType());
    }
  }

  @Override
  public boolean isFailOnException() {
    // The logic in the onEvent method of this listener is not critical, exceptions
    // can be ignored if logging fails...
    return false;
  }
}
```
该isFailOnException()方法确定onEvent(..)方法在调度事件时抛出异常时的行为。当false返回，异常被忽略。当true返回，异常不会被忽略和冒泡，未能有效地与当前进行命令。如果事件是API调用（或任何其他事务操作，例如，作业执行）的一部分，则将回滚事务。如果事件监听器中的行为不是关键业务，则建议返回false。

Flowable提供了一些基本实现，以方便事件监听器的常见用例。这些可以用作基类或示例侦听器实现：

- org.flowable.engine.delegate.event.BaseEntityEventListener：一个事件侦听器基类，可用于侦听特定类型的实体或所有实体的实体相关事件。它隐藏掉类型检查，并提供4种方法应覆盖：onCreate(..)，onUpdate(..)并onDelete(..)创建实体时，更新或删除。对于所有其他与实体相关的事件，onEntityEvent(..) 调用它。

### 3.18.2 配置和设置
如果在流程引擎配置中配置了事件侦听器，则它将在流程引擎启动时处于活动状态，并在后续重新引导引擎后保持活动状态。

该属性eventListeners需要一个org.flowable.engine.delegate.event.FlowableEventListener实例列表。像往常一样，您可以声明内联bean定义或使用ref现有bean代替。下面的代码段会向调度任何事件时通知的配置添加事件侦听器，而不管其类型如何：
```xml?linenums
<bean id="processEngineConfiguration"
    class="org.flowable.engine.impl.cfg.StandaloneProcessEngineConfiguration">
    ...
    <property name="eventListeners">
      <list>
         <bean class="org.flowable.engine.example.MyEventListener" />
      </list>
    </property>
</bean>
```
要在调度某些类型的事件时收到通知，请使用typedEventListeners需要映射的属性。map-entry的关键是以逗号分隔的事件名称列表（或单个事件名称）。map-entry的值是org.flowable.engine.delegate.event.FlowableEventListener实例列表。下面的代码片段为配置添加了一个事件监听器，在作业执行成功或失败时通知：
```xml?linenums
<bean id="processEngineConfiguration"
    class="org.flowable.engine.impl.cfg.StandaloneProcessEngineConfiguration">
    ...
    <property name="typedEventListeners">
      <map>
        <entry key="JOB_EXECUTION_SUCCESS,JOB_EXECUTION_FAILURE" >
          <list>
            <bean class="org.flowable.engine.example.MyJobEventListener" />
          </list>
        </entry>
      </map>
    </property>
</bean>
```
调度事件的顺序由添加侦听器的顺序决定。首先，所有正常的事件监听器eventListeners按照它们在中定义的顺序被调用（属性）list。之后typedEventListeners，如果调度了正确类型的事件，则调用所有类型化事件侦听器（属性）。
### 3.18.3 在运行时添加侦听器
可以使用API（RuntimeService）向引擎添加和删除其他事件侦听器：
```java?linenums
/**
 * Adds an event-listener which will be notified of ALL events by the dispatcher.
 * @param listenerToAdd the listener to add
 */
void addEventListener(FlowableEventListener listenerToAdd);

/**
 * Adds an event-listener which will only be notified when an event occurs,
 * which type is in the given types.
 * @param listenerToAdd the listener to add
 * @param types types of events the listener should be notified for
 */
void addEventListener(FlowableEventListener listenerToAdd, FlowableEventType... types);

/**
 * Removes the given listener from this dispatcher. The listener will no longer be notified,
 * regardless of the type(s) it was registered for in the first place.
 * @param listenerToRemove listener to remove
 */
 void removeEventListener(FlowableEventListener listenerToRemove);
 ```
 请注意，重新引导引擎时，不会保留在运行时添加的侦听器。
 ### 3.18.4. 将侦听器添加到流程定义中
 可以将侦听器添加到特定的流程定义中。只会调用与流程定义相关的事件以及与使用该特定流程定义启动的流程实例相关的所有事件的侦听器。可以使用完全限定的类名来定义侦听器实现，该表达式解析为实现侦听器接口的bean，或者可以配置为抛出消息/信号/错误BPMN事件。

监听器执行用户定义的逻辑
下面的代码片段为流程定义添加了2个侦听器。第一个侦听器将接收任何类型的事件，并使用基于完全限定类名的侦听器实现。第二个侦听器仅在作业成功执行或失败时通知，使用已在beans进程引擎配置属性中定义的侦听器。
```xml?linenums
<process id="testEventListeners">
  <extensionElements>
    <flowable:eventListener class="org.flowable.engine.test.MyEventListener" />
    <flowable:eventListener delegateExpression="${testEventListener}" events="JOB_EXECUTION_SUCCESS,JOB_EXECUTION_FAILURE" />
  </extensionElements>

  ...

</process>
```
对于与实体相关的事件，还可以将侦听器添加到流程定义中，该流程定义仅在特定实体类型发生实体事件时才会得到通知。下面的代码段显示了如何实现这一目标。它可以用于所有实体事件（第一个示例）或仅用于特定事件类型（第二个示例）。
```xml?linenums
<process id="testEventListeners">
  <extensionElements>
    <flowable:eventListener class="org.flowable.engine.test.MyEventListener" entityType="task" />
    <flowable:eventListener delegateExpression="${testEventListener}" events="ENTITY_CREATED" entityType="task" />
  </extensionElements>

  ...

</process>
```
为支持的值entityType是：attachment，comment，execution，identity-link，job，process-instance，process-definition，task。

听众抛出BPMN事件

处理正在调度的事件的另一种方法是抛出BPMN事件。请记住，只有使用某些Flowable事件类型抛出BPMN事件才有意义。例如，删除流程实例时抛出BPMN事件将导致错误。下面的代码片段显示了如何在进程实例中抛出信号，向外部进程（全局）抛出信号，在进程实例中抛出消息事件并在进程实例中抛出错误事件。不使用classor delegateExpression，而是使用属性throwEvent以及特定于要抛出的事件类型的附加属性。
```xml?linenums
<process id="testEventListeners">
  <extensionElements>
    <flowable:eventListener throwEvent="signal" signalName="My signal" events="TASK_ASSIGNED" />
  </extensionElements>
</process>
```
```xml?linenums
<process id="testEventListeners">
  <extensionElements>
    <flowable:eventListener throwEvent="globalSignal" signalName="My signal" events="TASK_ASSIGNED" />
  </extensionElements>
</process>
```
```xml?linenums
<process id="testEventListeners">
  <extensionElements>
    <flowable:eventListener throwEvent="message" messageName="My message" events="TASK_ASSIGNED" />
  </extensionElements>
</process>
```
```xml?linenums
<process id="testEventListeners">
  <extensionElements>
    <flowable:eventListener throwEvent="error" errorCode="123" events="TASK_ASSIGNED" />
  </extensionElements>
</process>
```
如果需要额外的逻辑来决定是否抛出BPMN事件，则可以扩展Flowable提供的侦听器类。通过覆盖isValidEvent(FlowableEvent event) in your subclass, the BPMN-event throwing can be prevented. The classes involved are +org.flowable.engine.test.api.event.SignalThrowingEventListenerTest，org.flowable.engine.impl.bpmn.helper.MessageThrowingEventListener和org.flowable.engine.impl.bpmn.helper.ErrorThrowingEventListener。
关于流程定义的监听器的注释
- 事件侦听器只能在process元素上声明，作为子元素extensionElements。无法在流程中的各个活动中定义监听器。

- 在其中使用的表达式delegateExpression不能访问执行上下文，因为其他表达式（例如，在网关中）具有。它们只能引用beans在流程引擎配置的属性中定义的bean，或者在实现侦听器接口的任何spring-bean中使用Spring（并且缺少beans属性）时。

- 使用class侦听器的属性时，只会创建该类的单个实例。确保侦听器实现不依赖于成员字段或确保从多个线程/上下文安全使用。

- 当在events属性中使用非法事件类型或使用非法throwEvent值时，将在部署流程定义时（有效地使部署失败）抛出异常。当提供class或delegateExecution提供非法值（不存在的类，不存在的bean引用或不实现侦听器接口的委托）时，将在启动进程时抛出异常（或者当该进程定义的第一个有效事件为派遣给听众）。确保引用的类在类路径上，并且表达式解析为有效的实例。

### 3.18.5. 通过API调度事件
我们通过API打开了事件派发机制，允许您将自定义事件分派给在引擎中注册的任何侦听器。建议（虽然没有强制执行）只调度FlowableEvents类型CUSTOM。可以使用以下命令调度事件RuntimeService：
```java?linenums
/**
 * Dispatches the given event to any listeners that are registered.
 * @param event event to dispatch.
 *
 * @throws FlowableException if an exception occurs when dispatching the event or
 * when the {@link FlowableEventDispatcher} is disabled.
 * @throws FlowableIllegalArgumentException when the given event is not suitable for dispatching.
 */
 void dispatchEvent(FlowableEvent event);
 ```
 ### 3.18.6. 支持的事件类型
 下面列出了引擎中可能出现的所有事件类型。每种类型对应于中的枚举值org.flowable.engine.common.api.delegate.event.FlowableEventType。
 <!--StartFragment-->

<table id="eventTypes" class="tableblock frame-all grid-all spread">
  <caption class="title">Table 1. Supported events</caption>
  <colgroup>
    <col>
      <col>
        <col>
  </colgroup>
  <thead>
    <tr>
      <th class="tableblock halign-left valign-top">Event name</th>
      <th class="tableblock halign-left valign-top">Description</th>
      <th class="tableblock halign-left valign-top">Event classes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">ENGINE_CREATED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">The process-engine this listener is attached to has been created and is
          ready for API-calls.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">ENGINE_CLOSED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">The process-engine this listener is attached to has been closed. API-calls
          to the engine are no longer possible.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">ENTITY_CREATED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A new entity is created. The new entity is contained in the event.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEntityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">ENTITY_INITIALIZED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A new entity has been created and is fully initialized. If any children
          are created as part of the creation of an entity, this event will be
          fired AFTER the create/initialisation of the child entities as opposed
          to the
          <code>ENTITY_CREATE</code> event.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEntityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">ENTITY_UPDATED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">An existing entity is updated. The updated entity is contained in the event.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEntityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">ENTITY_DELETED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">An existing entity is deleted. The deleted entity is contained in the event.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEntityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">ENTITY_SUSPENDED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">An existing entity is suspended. The suspended entity is contained in the
          event. Will be dispatched for ProcessDefinitions, ProcessInstances and
          Tasks.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEntityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">ENTITY_ACTIVATED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">An existing entity is activated. The activated entity is contained in the
          event. Will be dispatched for ProcessDefinitions, ProcessInstances and
          Tasks.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEntityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">JOB_EXECUTION_SUCCESS</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A job has been executed successfully. The event contains the job that was
          executed.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEntityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">JOB_EXECUTION_FAILURE</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">The execution of a job has failed. The event contains the job that was
          executed and the exception.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEntityEvent</code> and
          <code>org.flowable…​FlowableExceptionEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">JOB_RETRIES_DECREMENTED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">The number of job retries have been decremented due to a failed job. The
          event contains the job that was updated.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEntityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">TIMER_SCHEDULED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A timer job has been created and is scheduled for being executed at a future
          point in time.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEntityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">TIMER_FIRED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A timer has been fired. The event contains the job that was executed.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEntityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">JOB_CANCELED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A job has been canceled. The event contains the job that was canceled.
          Job can be canceled by API call, task was completed and associated boundary
          timer was canceled, on the new process definition deployment.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEntityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">ACTIVITY_STARTED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">An activity is starting to execute</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableActivityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">ACTIVITY_COMPLETED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">An activity is completed successfully</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableActivityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">ACTIVITY_CANCELLED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">An activity is going to be canceled. There can be three reasons for activity
          cancellation (MessageEventSubscriptionEntity, SignalEventSubscriptionEntity,
          TimerEntity).</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableActivityCancelledEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">ACTIVITY_SIGNALED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">An activity received a signal</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableSignalEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">ACTIVITY_MESSAGE_RECEIVED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">An activity received a message. Dispatched before the activity receives
          the message. When received, a
          <code>ACTIVITY_SIGNAL</code> or
          <code>ACTIVITY_STARTED</code> will be dispatched for this activity, depending
          on the type (boundary-event or event-subprocess start-event)</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableMessageEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">ACTIVITY_MESSAGE_WAITING</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">An activity has created a message event subscription and is waiting to
          receive.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableMessageEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">ACTIVITY_MESSAGE_CANCELLED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">An activity for which a message event subscription has been created is
          canceled and thus receiving the message will not trigger this particular
          message anymore.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableMessageEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">ACTIVITY_ERROR_RECEIVED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">An activity has received an error event. Dispatched before the actual error
          has been handled by the activity. The event’s
          <code>activityId</code> contains a reference to the error-handling activity.
          This event will be either followed by a
          <code>ACTIVITY_SIGNALLED</code> event or
          <code>ACTIVITY_COMPLETE</code> for the involved activity, if the error was
          delivered successfully.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableErrorEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">UNCAUGHT_BPMN_ERROR</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">An uncaught BPMN error has been thrown. The process did not have any handlers
          for that specific error. The event’s
          <code>activityId</code> will be empty.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableErrorEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">ACTIVITY_COMPENSATE</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">An activity is about to be compensated. The event contains the id of the
          activity that is will be executed for compensation.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableActivityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">MULTI_INSTANCE_ACTIVITY_STARTED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A multi-instance activity is starting to execute</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableMultiInstanceActivityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">MULTI_INSTANCE_ACTIVITY_COMPLETED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A multi-instance activity completed successfully</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableMultiInstanceActivityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">MULTI_INSTANCE_ACTIVITY_CANCELLED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A multi-instance activity is going to be canceled. There can be three reasons
          for activity cancellation (MessageEventSubscriptionEntity, SignalEventSubscriptionEntity,
          TimerEntity).</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableMultiInstanceActivityCancelledEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">VARIABLE_CREATED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A variable has been created. The event contains the variable name, value
          and related execution and task (if any).</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableVariableEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">VARIABLE_UPDATED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">An existing variable has been updated. The event contains the variable
          name, updated value and related execution and task (if any).</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableVariableEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">VARIABLE_DELETED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">An existing variable has been deleted. The event contains the variable
          name, last known value and related execution and task (if any).</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableVariableEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">TASK_ASSIGNED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A task has been assigned to a user. The event contains the task</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEntityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">TASK_CREATED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A task has been created. This is dispatched after the
          <code>ENTITY_CREATE</code> event. If the task is part of a process, this event
          will be fired before the task listeners are executed.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEntityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">TASK_COMPLETED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A task has been completed. This is dispatched before the
          <code>ENTITY_DELETE</code> event. If the task is part of a process, this event
          will be fired before the process has moved on and will be followed by
          a
          <code>ACTIVITY_COMPLETE</code> event, targeting the activity that represents
          the completed task.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEntityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">PROCESS_CREATED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A process instance has been created. All basic properties have been set,
          but variables not yet.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEntityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">PROCESS_STARTED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A process instance has been started. Dispatched when starting a process
          instance previously created. The event PROCESS_STARTED is dispatched
          after the associated event ENTITY_INITIALIZED and after the variables
          have been set.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEntityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">PROCESS_COMPLETED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A process has been completed, meaning all executions have stopped for the
          process instance. Dispatched after the last activity
          <code>ACTIVITY_COMPLETED </code> event. Process is completed when it reaches
          state in which process instance does not have any transition to take.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableEntityEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">PROCESS_COMPLETED_WITH_TERMINATE_END_EVENT</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A process has been completed by arriving at a terminate end event.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableProcessTerminatedEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">PROCESS_CANCELLED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A process has been canceled. Dispatched before the process instance is
          deleted from runtime. A process instance can for example be canceled
          by the API call
          <code>RuntimeService.deleteProcessInstance</code>, by an interrupting boundary
          event on a call activity, …​</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableCancelledEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">MEMBERSHIP_CREATED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A user has been added to a group. The event contains the ids of the user
          and group involved.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableMembershipEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">MEMBERSHIP_DELETED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">A user has been removed from a group. The event contains the ids of the
          user and group involved.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableMembershipEvent</code>
        </p>
      </td>
    </tr>
    <tr>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">MEMBERSHIPS_DELETED</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">All members will be removed from a group. The event is thrown before the
          members are removed, so they are still accessible. No individual
          <code>MEMBERSHIP_DELETED</code> events will be thrown if all members are deleted
          at once, for performance reasons.</p>
      </td>
      <td class="tableblock halign-left valign-top">
        <p class="tableblock">
          <code>org.flowable…​FlowableMembershipEvent</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>
<!--EndFragment-->
所有ENTITY_\*事件都与引擎内的实体相关。下面的列表显示了为哪些实体分派了哪些实体事件的概述：

- ENTITY_CREATED, ENTITY_INITIALIZED, ENTITY_DELETED：附件，注释，部署，执行，组，IdentityLink，作业，模型，ProcessDefinition，ProcessInstance，任务，用户。

- ENTITY_UPDATED：附件，部署，执行，组，IdentityLink，作业，模型，ProcessDefinition，ProcessInstance，任务，用户。

- ENTITY_SUSPENDED, ENTITY_ACTIVATED：ProcessDefinition，ProcessInstance / Execution，Task。

### 3.18.7 补充说明
只有通过他们注册的引擎发送的事件才会通知监听器。因此，如果您有不同的引擎 - 针对同一个数据库运行 - 只会将侦听器注册到的引擎中发起的事件调度到该侦听器。无论是否在同一JVM中运行，其他引擎中发生的事件都不会分派给侦听器。

某些事件类型（与实体相关）公开目标实体。根据类型或事件，这些实体不能再更新（例如，删除实体时）。如果可能，使用EngineServices事件公开的内容以安全的方式与引擎进行交互。即便如此，您还需要对调度事件中涉及的实体的更新/操作保持谨慎。

没有调度与历史相关的实体事件，因为它们都具有调度其事件的运行时对应事件。
