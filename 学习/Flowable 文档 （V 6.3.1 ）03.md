---
title: Flowable 文档 （V 6.3.1 ）03
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

# 3.配置
## 3.1. 创建ProcessEngine
Flowable流程引擎通过名为`flowable.cfg.xml`的XML文件配置。请注意这种方式与使用Spring创建流程引擎不一样。

获取`ProcessEngine`，最简单的方式是使用`org.flowable.engine.ProcessEngines`类：
```java?linenums
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine()
```
这样会从classpath寻找`flowable.cfg.xml`，并用这个文件中的配置构造引擎。下面的代码展示了一个配置的例子。后续章节会对配置参数进行详细介绍。
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
请注意这个配置XML文件实际上是一个Spring配置文件。但这并不意味着Flowable只能用于Spring环境！我们只是简单利用Spring内部的解析与依赖注入功能来构造引擎。

也可以通过编程方式使用配置文件，来构造ProcessEngineConfiguration对象。也可以使用不同的bean id(例如第3行)。
```java?linenums
ProcessEngineConfiguration.
  createProcessEngineConfigurationFromResourceDefault();
  createProcessEngineConfigurationFromResource(String resource);
  createProcessEngineConfigurationFromResource(String resource, String beanName);
  createProcessEngineConfigurationFromInputStream(InputStream inputStream);
  createProcessEngineConfigurationFromInputStream(InputStream inputStream, String beanName);
```
也可以不使用配置文件，基于默认创建配置（参考不同的支持类获得更多信息）。
```java?linenums
ProcessEngineConfiguration.createStandaloneProcessEngineConfiguration();
ProcessEngineConfiguration.createStandaloneInMemProcessEngineConfiguration();
```
所有的`ProcessEngineConfiguration.createXXX()`方法都返回`ProcessEngineConfiguration`，并可以继续按需调整。

调用buildProcessEngine()后，生成一个ProcessEngine：
```java?linenums
ProcessEngine processEngine = ProcessEngineConfiguration.createStandaloneInMemProcessEngineConfiguration()
  .setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_FALSE)
  .setJdbcUrl("jdbc:h2:mem:my-own-db;DB_CLOSE_DELAY=1000")
  .setAsyncExecutorActivate(false)
  .buildProcessEngine();
```
## 3.2. ProcessEngineConfiguration bean
`flowable.cfg.xml`文件中必须包含一个id为`processEngineConfiguration`的bean。
```xml?linenums
<bean id="processEngineConfiguration" class="org.flowable.engine.impl.cfg.StandaloneProcessEngineConfiguration">
```
这个bean用于构建ProcessEngine。有多个类可以用于定义processEngineConfiguration。这些类用于不同的环境，并各自设置一些默认值。最佳实践是选择最匹配你环境的类，以便减少配置引擎需要的参数。下面列出目前可以使用的类：

 - `org.flowable.engine.impl.cfg.StandaloneProcessEngineConfiguration`：流程引擎独立运行。Flowable自行处理事务。在默认情况下，数据库检查只在引擎启动时进行（如果Flowable表结构不存在或表结构版本不对，会抛出异常）。
 - `org.flowable.engine.impl.cfg.StandaloneInMemProcessEngineConfiguration`：这是一个便于使用单元测试的类。Flowable自行处理事务。默认使用H2内存数据库。数据库会在引擎启动时创建，并在引擎关闭时删除。使用这个类时，很可能不需要更多的配置（除了使用任务执行器或邮件功能等时）。
 - `org.flowable.spring.SpringProcessEngineConfiguration`：在流程引擎处于Spring环境时使用。查看Spring集成章节获得更多信息。
 - `org.flowable.engine.impl.cfg.JtaProcessEngineConfiguration`：用于引擎独立运行，并使用JTA事务的情况。
 
 ## 3.3. 数据库配置
 有两种方式可以配置Flowable引擎使用的数据库。第一种方式是定义数据库的JDBC参数：
 
 - jdbcUrl：数据库的JDBC URL。
 - jdbcDriver：实现特定数据库类型的驱动程序。
 - jdbcUsername：用于连接数据库的用户名。
 - jdbcPassword：连接数据库的密码。
 
 通过提供的JDBC参数构造的数据源，使用默认的MyBatis连接池设置。可用下列属性调整这个连接池（来自MyBatis文档）：
 
 - jdbcMaxActiveConnections：连接池能够容纳的最大活动连接数量。默认值为10。
 - jdbcMaxIdleConnections：连接池能够容纳的最大空闲连接数量。
 - jdbcMaxCheckoutTime：连接从连接池“取出”后，被强制返回前的最大时间间隔，单位为毫秒。默认值为20000（20秒）。
 - jdbcMaxWaitTime：这是一个底层设置，在连接池获取连接的时间异常长时，打印日志并尝试重新获取连接（避免连接池配置错误造成的永久沉默失败。默认值为20000（20秒）。
 
 数据库配置示例：
 ```xml?linenums
 <property name="jdbcUrl" value="jdbc:h2:mem:flowable;DB_CLOSE_DELAY=1000" />
<property name="jdbcDriver" value="org.h2.Driver" />
<property name="jdbcUsername" value="sa" />
<property name="jdbcPassword" value="" />
 ```
 我们的跑分显示MyBatis连接池在处理大量并发请求时，并不是最经济或最具弹性的。因此，建议使用javax.sql.DataSource的实现，并将其注入到流程引擎配置中（例如DBCP、CP30、Hikari、Tomcat连接池，等等）：
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
 请注意Flowable发布时不包括用于定义数据源的库。需要自行把库放在你的classpath中。

无论使用JDBC还是数据源方式配置，下列参数都可以使用：

 - databaseType：通常不需要专门设置这个参数，因为它可以从数据库连接信息中自动检测得出。只有在自动检测失败时才需要设置。可用值：{h2, mysql, oracle, postgres, mssql, db2}。这个选项会决定创建、删除与查询时使用的脚本。查看“支持的数据库”章节了解我们支持哪些类型的数据库。
 - databaseSchemaUpdate：用于设置流程引擎启动关闭时使用的数据库表结构控制策略。
  - false （默认值）：当引擎启动时，检查数据库表结构的版本是否匹配库文件版本。版本不匹配时抛出异常。
  - true：构建引擎时，检查并在需要时更新表结构。表结构不存在则会创建。
  - create-drop：引擎创建时创建表结构，并在引擎关闭时删除表结构。
  
## 3.4. JNDI数据源配置
默认情况下，Flowable的数据库配置保存在每个web应用WEB-INF/classes目录下的db.properties文件中。有时这样并不合适，因为这需要用户修改Flowable源码中的db.properties文件并重新编译war包，或者在部署后解开war包并修改db.properties文件。

通过使用JNDI（Java Naming and Directory Interface，Java命名和目录接口）获取数据库连接时，连接就完全由Servlet容器管理，并可以在war部署之外管理配置。同时也提供了比db.properties中更多的控制连接的参数。
### 3.4.1. 组态
根据你使用的servlet容器应用不同，配置JNDI数据源的方式也不同。下面的介绍用于Tomcat，对于其他容器应用，请参考对应的文档。

Tomcat的JNDI资源配置在$CATALINA_BASE/conf/[enginename]/[hostname]/[warname].xml (对于Flowable UI通常会是$CATALINA_BASE/conf/Catalina/localhost/flowable-app.xml)。当应用第一次部署时，默认会从Flowable war包中复制context.xml。所以如果存在这个文件则需要替换。例如，如果需要将JNDI资源修改为应用连接MySQL而不是H2，按照下列修改文件：
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
要配置JNDI数据源，在Flowable UI的配置文件中使用下列参数：

- spring.datasource.jndi-name =：数据源的JNDI名。

- datasource.jndi.resourceRef：设置是否在J2EE容器中查找。也就是说，如果JNDI名中没有包含"java:comp/env/"前缀，是否需要添加它。默认为"true"。

## 3.5. 支持的数据库
下面列出Flowable用于引用数据库的类型（区分大小写！）。

|  数据库类型   |  示例JDBC URL   |  备注   |
| :-- | :-- | :-- |
|   h2  |  jdbc:h2:tcp://localhost/flowable   | Default configured database    |
|  mysql   |  jdbc:mysql://localhost:3306/flowable?autoReconnect=true   |  Tested using mysql-connector-java database driver   |
|   oracle  |   jdbc:oracle:thin:@localhost:1521:xe  |     |
|  postgres   | jdbc:postgresql://localhost:5432/flowable    |     |
|   db2  |     |  jdbc:db2://localhost:50000/flowable   |
|  mssql   |  jdbc:sqlserver://localhost:1433;<br>databaseName=flowable <br>(jdbc.driver=com.microsoft.sqlserver.jdbc.SQLServerDriver) <br>OR <br>jdbc:jtds:sqlserver://localhost:1433/flowable (jdbc.driver=net.sourceforge.jtds.jdbc.Driver)   |   Tested using Microsoft JDBC Driver 4.0 (sqljdbc4.jar) and JTDS Driver  |
## 3.6. 创建数据库表
为数据库创建数据库表的最简单方法是：
- 在classpath中增加flowable-engine jar

- 添加合适的数据库驱动程序

- 在classpath中增加Flowable配置文件(flowable.cfg.xml)，指向你的数据库(参考数据库配置)

- 执行DbSchemaCreate类的main方法

然而，通常只有数据库管理员可以在数据库中执行DDL语句，在生产环境中这也是最明智的选择。DDL的SQL脚本可以在Flowable下载页面或Flowable发布目录中找到，位于database子目录。引擎jar (flowable-engine-x.jar)的org/flowable/db/create包中也有一份(drop目录存放删除脚本)。SQL文件的格式为：
```
flowable.{db}.{create|drop}.{type}.sql
```
db为支持的数据库，而type为：
- engine：引擎执行所需的表，必需。

- history：存储历史与审计信息的表。当历史级别设置为none时不需要。请注意不使用这些表会导致部分使用历史数据的功能失效（如任务备注）。

MySQL用户注意事项：低于5.6.4的MySQL版本不支持timestamps或包含毫秒精度的日期。更糟的是部分版本会在创建类似的列时抛出异常，而另一些版本则不会。当使用自动创建/升级时，引擎在执行时会自动修改DDL语句。当使用DDL文件方式建表时，可以使用通用版本，或使用文件名包含mysql55的特殊版本（用于5.6.4以下的任何版本）。特殊版本的文件中不会使用毫秒精度的列类型。

具体来说，对于MySQL的版本：

- <5.6：不支持毫秒精度。可以使用DDL文件（使用包含mysql55的文件）。可以使用自动创建/升级。

- 5.6.0 - 5.6.3：不支持毫秒精度。不可以使用自动创建/升级。建议升级为较新版本的数据库。如果确实需要，可以使用包含mysql55的DDL文件。

- 5.6.4+：支持毫秒精度。可以使用DDL文件（默认的包含mysql的文件）。可以使用自动创建/升级。

请注意在Flowable表已经创建/升级后，再升级MySQL数据库，则需要手工修改列类型！

## 3.7. 数据库表名称解释
Flowable的所有数据库表都以ACT_开头。第二部分是说明表用途的两字符标示符。服务API的命名也大略符合这个规则。

- ACT_RE_ *：'RE’代表`repository`。带有这个前缀的表包含“静态”信息，例如流程定义与流程资源（图片、规则等）。

- ACT_RU_ *： 'RU’代表`runtime`。这些表存储运行时信息，例如流程实例（process instance）、用户任务（user task）、变量（variable）、作业（job）等。Flowable只在流程实例运行中保存运行时数据，并在流程实例结束时删除记录。这样保证运行时表小和快。

- ACT_HI_ *：'HI’代表`history`。这些表存储历史数据，例如已完成的流程实例、变量、任务等。

- ACT_GE_ *： 通用数据。在很多场景中使用。

## 3.8. 数据库升级
在升级前，请确保你已经（使用数据库的备份功能）备份了数据库。

默认情况下，每次流程引擎创建时会进行版本检查，通常是在你的应用或者Flowable web应用启动的时候。如果Flowable库发现库版本与Flowable数据库表版本不同，会抛出异常。

要进行升级，首先需要将下列配置参数放入你的flowable.cfg.xml配置文件：
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
同时，在classpath中加上合适的数据库驱动。升级你应用中的Flowable库，或者启动一个新版本的Flowable，并将它指向包含旧版本数据的数据库。将`databaseSchemaUpdate`设置为true。当Flowable发现库与数据库表结构不同步时，会自动将数据库表结构升级至新版本。

你还可以直接运行升级DDL语句，也可以从Flowable下载页面获取升级数据库脚本并运行。
## 3.9. 作业执行器（从6.0.0版开始）
在Flowable V6中唯一可用的作业执行器，是Flowable V5中的异步执行器(async executor)。因为它为Flowable引擎提供了性能更好，对数据库也更友好的执行异步作业的方式。Flowable V5中的作业执行器(job executor)在V6中不再可用。

此外，如果在Java EE 7下运行，容器还可以使用符合JSR-236标准的ManagedJobExecutor来管理线程。要启用这个功能，需要在配置中如下加入线程工厂：
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
如果没有设置线程工厂，ManagedJobExecutor实现会退化为默认实现（非managed版本）。
## 3.10. 作业执行器激活
 `AsyncExecutor`是管理线程池的组件，用于触发定时器与其他异步任务。也可以使用其他实现（例如使用消息队列，参见用户手册的高级章节）。

默认情况下，`AsyncExecutor`并未激活，也不会启动。用如下配置使异步执行器与Flowable引擎一同启动：
```xml?linenums
	<property name="asyncExecutorActivate" value="true" />
```
`asyncExecutorActivate`这个参数使Flowable引擎在启动同时启动异步执行器。
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
