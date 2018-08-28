---
title: Flowable 文档 （V 6.3.1 ）02
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


# 2. 入门
## 2.1. 什么是Flowable？
Flowable是一个用Java编写的轻量级业务流程引擎。Flowable流程引擎允许您部署BPMN 2.0流程定义（用于定义流程的行业XML标准），创建流程定义的流程实例，运行查询，访问活动或历史流程实例和相关数据等等。本节将逐步介绍各种概念和API，通过您可以在自己的开发机器上遵循的示例来实现。

Flowable在将其添加到您的应用程序/服务/体系结构时非常灵活。您可以通过包含Flowable库（可用作JAR）将引擎嵌入到您的应用程序或服务中。由于它是JAR，因此您可以轻松地将其添加到任何Java环境中：Java SE; servlet容器，如Tomcat或Jetty，Spring; Java EE服务器，例如JBoss或WebSphere等。或者，您可以使用Flowable REST API通过HTTP进行通信。还有几个Flowable应用程序（Flowable Modeler，Flowable Admin，Flowable IDM和Flowable Task），它们提供了用于处理流程和任务的开箱即用的示例UI。

设置Flowable的所有方法的共同点是核心引擎，可以将其视为公开API以管理和执行业务流程的服务集合。下面的各种教程首先介绍如何设置和使用这个核心引擎。之后的部分建立在前面部分中获得的知识之上。

 - 在第一部分展示了如何在可能的最简单的方式运行可流动：仅使用Java SE普通的Java主。这里将解释许多核心概念和API。
 - 将在可流动的REST API部分显示如何运行，并通过REST使用相同的API。
 - 将在可流动的应用部分，将指导您使用出的现成例子可流动的用户界面的基本知识。
## 2.2. Flowable和Activiti
Flowable是Activiti（Alfresco的注册商标）的一个分支。在以下所有部分中，您将注意到程序包名称，配置文件等使用flowable。
## 2.3. 构建命令行应用程序
### 2.3.1 创建流程引擎
在第一篇教程中，我们将构建一个简单的示例，演示如何创建Flowable流程引擎，介绍一些核心概念并演示如何使用API。屏幕截图显示了Eclipse，但任何IDE都可以运行。我们将使用Maven来获取Flowable依赖项并管理构建，但同样，任何替代方案也都有效（Gradle，Ivy等）。

我们要构建的示例是一个简单的假日请求过程：

 - 该员工询问了一些假期
 - 该经理批准或拒绝该请求
 - 我们将模仿在某些外部系统中注册请求，并向员工发送包含结果的电子邮件
 
 首先，我们通过File→New→Other→Maven Project创建一个新的Maven项目

![enter description here](./images/1535458711260.png)

在下一个屏幕中，我们检查创建一个简单的项目（跳过原型选择）

![enter description here](./images/1535458711261.png)

并填写一些'Group Id'和'Artifact id'：

![enter description here](./images/1535458711590.png)

我们现在有一个空的Maven项目，我们将添加两个依赖项：

 - Flowable流程引擎，它允许我们创建ProcessEngine对象并访问Flowable API。
 - 在这种情况下，内存数据库H2作为Flowable引擎需要一个数据库来存储执行和历史数据，同时运行流程实例。请注意，H2依赖项包括数据库和驱动程序。如果您使用其他数据库（例如，PostgresQL，MySQL等），则需要添加特定的数据库驱动程序依赖项。

将以下内容添加到pom.xml文件中：

```xml?linenums
<dependencies>
  <dependency>
    <groupId>org.flowable</groupId>
    <artifactId>flowable-engine</artifactId>
    <version>6.3.1</version>
  </dependency>
  <dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.3.176</version>
  </dependency>
</dependencies>
```
如果由于某种原因未自动检索从属JAR，则可以右键单击项目并选择Maven→更新项目以强制手动刷新（但通常不需要这样做）。在项目中，在' Maven依赖项下，您现在应该看到可流动引擎和各种其他（传递）依赖项。
创建一个新的Java类并添加常规的Java main方法：
```java?linenums
package org.flowable;

public class HolidayRequest {

  public static void main(String[] args) {

  }

}
```
我们需要做的第一件事是实例化ProcessEngine实例。这是一个线程安全的对象，您通常只需在应用程序中实例化一次。一个流程引擎从创建ProcessEngineConfiguration实例，它允许您配置和调整设置的流程引擎。通常，ProcessEngineConfiguration是使用配置XML文件创建的，但是（如我们所做）您也可以通过编程方式创建它。ProcessEngineConfiguration需要的最小配置是与数据库的JDBC连接：
```java?linenums
package org.flowable;

import org.flowable.engine.ProcessEngine;
import org.flowable.engine.ProcessEngineConfiguration;
import org.flowable.engine.impl.cfg.StandaloneProcessEngineConfiguration;

public class HolidayRequest {

  public static void main(String[] args) {
    ProcessEngineConfiguration cfg = new StandaloneProcessEngineConfiguration()
      .setJdbcUrl("jdbc:h2:mem:flowable;DB_CLOSE_DELAY=-1")
      .setJdbcUsername("sa")
      .setJdbcPassword("")
      .setJdbcDriver("org.h2.Driver")
      .setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_TRUE);

    ProcessEngine processEngine = cfg.buildProcessEngine();
  }

}
```
在上面的代码中，在第10行，创建了一个独立的配置对象。这里的'standalone'指的是引擎是完全自己创建和使用的（而不是，例如，在Spring环境中，你在那里使用SpringProcessEngineConfiguration类）。在第11到14行，传递到内存中H2数据库实例的JDBC连接参数。重要说明：请注意，此类数据库无法在JVM重新启动后继续存在。如果您希望数据是持久的，则需要切换到持久数据库并相应地切换连接参数。在第15行，我们将标志设置为true确保在JDBC参数指向的数据库中尚不存在数据库模式时创建数据库模式。或者，Flowable附带一组SQL文件，可用于手动创建包含所有表的数据库模式。

所述流程引擎对象然后使用此配置（第17行）创建。

你现在可以运行它。Eclipse中最简单的方法是右键单击类文件并选择Run As→Java Application：

![enter description here](./images/1535458711594.png)

应用程序运行没有问题，但是，除了显示未正确配置日志记录的消息之外，控制台中不显示任何有用信息：

![enter description here](./images/1535458711261_1.png)


## 2.4. Flowable REST API入门
