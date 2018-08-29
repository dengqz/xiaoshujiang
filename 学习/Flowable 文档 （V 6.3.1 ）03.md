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