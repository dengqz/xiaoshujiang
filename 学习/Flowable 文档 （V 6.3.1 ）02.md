---
title: Flowable 文档 （V 6.3.1 ）02
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


# 2. 入门
## 2.1. 什么是Flowable？
Flowable是一个使用Java编写的轻量级业务流程引擎。Flowable流程引擎让你可以部署BPMN 2.0流程定义（用于定义流程的行业XML标准），创建这些流程定义的流程实例，进行查询，访问运行中或历史的流程实例和相关数据等等。本章节将用一个可以在你自己的开发环境中使用的例子，逐步介绍各种概念与API。

Flowable可以十分灵活地加入你的应用/服务/构架。可以将JAR形式发布的Flowable库加入应用或服务，来嵌入引擎。以JAR形式发布使Flowable可以轻易加入任何Java环境：Java SE；Tomcat、Jetty或Spring之类的servlet容器；JBoss或WebSphere之类的Java EE服务器，等等。另外，也可以使用Flowable REST API通过HTTP通信。也有许多Flowable应用（Flowable Modeler, Flowable Admin, Flowable IDM 与 Flowable Task），提供了直接可用的UI示例，可以使用流程与任务。

所有设置Flowable的方法的共同点是核心引擎，核心引擎可被看做是一组服务的集合，并暴露了管理与执行业务流程的API。下面的各种教程都以设置与使用核心引擎的介绍开始。后续章节都建立在之前章节中获取的知识之上。
## 2.2. Flowable和Activiti
Flowable是Activiti(Alfresco的注册商标)的分支。在下面的章节中，你会注意到包名，配置文件等等，都使用flowable。
## 2.3. 构建命令行应用程序
### 2.3.1 创建一个流程引擎
在本章节中，将构建一个简单的例子，以展示如何创建一个Flowable流程引擎，介绍一些核心概念，并展示如何使用API。截图使用Eclipse，但实际上可以使用任何IDE。我们使用Maven获取Flowable依赖及管理构建，但是类似的任何其它方法也都可以使用（Gradle，Ivy，等等）。

我们要构建的示例是一个简单的请假流程：

 - 雇员申请几天的假期
 - 经理批准或驳回申请
 - 模拟将申请注册到某个外部系统，并给雇员发送结果邮件
 
 首先，我们通过File→New→Other→Maven Project创建一个新的Maven项目

![enter description here](./images/1535512648411.png)

在下一个界面中，选中‘创建一个简单的项目’（跳过原型选择）

![enter description here](./images/1535512648413.png)

填入'Group Id'和'Artifact id'：

![enter description here](./images/1535512648941.png)

我们现在有一个空的Maven项目，然后添加两个依赖：

 - Flowable流程引擎，使我们可以创建一个ProcessEngine流程引擎对象并访问Flowable API。
 - 本例中为H2，因为Flowable引擎在运行流程实例时，需要使用数据库来存储执行与历史数据。请注意H2依赖包含了数据库及驱动。如果使用其他数据库（例如PostgreSQL，MySQL等），需要添加对应的数据库驱动依赖。

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
如果由于某些原因，依赖JAR无法自动获取，可以右键点击项目，并选择'Maven → Update Project'以强制手动刷新（一般不会需要这么操作）。在这个项目中的'Maven Dependencies'下，可以看到flowable-engine与许多其他依赖。
创建一个新的Java类，并添加标准的Java main方法：
```java?linenums
package org.flowable;

public class HolidayRequest {

  public static void main(String[] args) {

  }

}
```
首先要做的是初始化一个ProcessEngine流程引擎实例。这是一个线程安全的对象，因此通常只需要在一个应用中初始化一次。ProcessEngine由一个ProcessEngineConfiguration实例创建。该实例可以配置与调整流程引擎的设置。通常使用一个配置XML文件创建ProcessEngineConfiguration，但是（像在这里做的一样）也可以编程方式创建它。ProcessEngineConfiguration最少需要配置一个数据库的JDBC连接：
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
在上面的代码中，第10行创建了一个独立(standalone)配置对象。这里的'独立'指的是引擎是完全独立创建及使用的（而不是在例如Spring环境中使用，这时需要使用SpringProcessEngineConfiguration类代替）。第11至14行中，传递了一个内存H2数据库实例的JDBC连接参数。重要：请注意这样的数据库在JVM重启后会消失。如果需要永久保存数据，需要切换为持久化数据库，并相应切换连接参数。第15行中，设置了一个true标志，确保在JDBC参数连接的数据库中，数据库表结构不存在时，会创建相应的表结构。另外，Flowable也提供了一组SQL文件，可用于手动创建所有表的数据库表结构。

然后使用这个配置创建ProcessEngine对象（第17行）。

这样就可以运行了。在Eclipse中最简单的方法是右键点击类文件，选择Run As → Java Application

![enter description here](./images/1535512648941_1.png)

应用运行没有问题，但也没有在控制台提供有用的信息，只有一条消息提示日志没有正确配置：

![enter description here](./images/1535512648413_1.png)

Flowable使用SLF4J作为内部日志框架。在这个例子中，我们在SLF4J之上使用log4j。因此在pom.xml文件中添加下列依赖：
```xml?linenums
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>1.7.21</version>
</dependency>
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
  <version>1.7.21</version>
</dependency>
```
Log4j需要一个配置文件。在src/main/resources文件夹下添加log4j.properties文件，并写入下列内容：
```xml?linenums
log4j.rootLogger = DEBUG，CA

log4j.appender.CA = org.apache.log4j.ConsoleAppender
log4j.appender.CA.layout = org.apache.log4j.PatternLayout
log4j.appender.CA.layout.ConversionPattern =％d {hh：mm：ss，SSS} [％t]％-5p％c％x  - ％m％n
```
重新运行该应用程序。您现在应该可以看到关于引擎启动与创建数据库表结构的提示日志

![enter description here](./images/1535512648938.png)

### 2.3.2. 部署一个流程定义

我们要构建的流程是一个非常简单的请假流程。Flowable引擎需要流程定义为BPMN 2.0格式，这是一个业界广泛接受的XML标准。在Flowable术语中，我们将其称为一个流程定义(process definition)。一个流程定义可以启动多个流程实例(process instance)。流程定义可以看做是重复执行流程的蓝图。在这个例子中，流程定义定义了请假的各个步骤，而一个流程实例对应某个雇员提出的一个请假申请。

BPMN 2.0存储为XML，包含可视化的部分：使用标准方式定义了每个步骤类型（人工任务，自动服务调用，等等）如何呈现，以及如何互相连接。这样BPMN 2.0标准使技术人员与业务人员能用双方都能理解的方式交流业务流程。

我们要使用的流程定义如下：

![enter description here](./images/1535512648433.png)

这个流程应该是不言自明的了，但为了清楚起见，描述一下不同的部分：

 - 我们假定启动流程需要提供一些信息，例如雇员名字、请假时长以及描述。当然，这些可以单独建模为流程中的第一步。但是如果将它们作为流程的“输入信息”，就能保证只有在实际请求时才会建立一个流程实例。否则（将提交作为流程的第一步），用户可能在提交之前改变主意并取消，但流程实例已经创建了。在某些场景中，就可能影响重要的指标（例如启动了多少申请，但还未完成），取决于业务目标。
 - 左侧的圆圈叫做启动事件(start event)。这是一个流程实例的起点。
 - 第一个矩形是一个用户任务(user task)。这是流程中人类用户操作的步骤。在这个例子中，经理需要批准或驳回申请。
 - 取决于经理的决定，排他网关(exclusive gateway) (带叉的菱形)会将流程实例路由至批准或驳回路径。
 - 如果批准，则需要将申请注册至某个外部系统，并跟着另一个用户任务，将经理的决定通知给申请人。当然也可以替换为邮件。
 - 如果驳回，则为雇员发送一封邮件通知他。

一般来说，这样的流程定义使用可视化建模工具建立，如Flowable Designer(Eclipse)或Flowable Web Modeler(Web应用)。

然而，在这里，我们将直接编写XML以熟悉BPMN 2.0及其概念。

与上面展示的流程图对应的BPMN 2.0 XML在下面显示。请注意这只包含了“流程部分”。如果使用图形化建模工具，实际的XML文件还将包含“可视化部分”，用于描述图形信息，如流程定义中各个元素的坐标（所有的图形化信息包含在XML的BPMNDiagram标签中，作为definitions标签的子元素）。

将以下XML保存在src / main / resources文件夹下名为holiday-request.bpmn20.xml的文件中。

```xml?linenums
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="http://www.w3.org/2001/XMLSchema"
  xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI"
  xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC"
  xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI"
  xmlns:flowable="http://flowable.org/bpmn"
  typeLanguage="http://www.w3.org/2001/XMLSchema"
  expressionLanguage="http://www.w3.org/1999/XPath"
  targetNamespace="http://www.flowable.org/processdef">

  <process id="holidayRequest" name="Holiday Request" isExecutable="true">

    <startEvent id="startEvent"/>
    <sequenceFlow sourceRef="startEvent" targetRef="approveTask"/>

    <userTask id="approveTask" name="Approve or reject request"/>
    <sequenceFlow sourceRef="approveTask" targetRef="decision"/>

    <exclusiveGateway id="decision"/>
	<sequenceFlow sourceRef="decision" targetRef="externalSystemCall">
      <conditionExpression xsi:type="tFormalExpression">
        <![CDATA[
          ${approved}
        ]]>
      </conditionExpression>
    </sequenceFlow>
    <sequenceFlow  sourceRef="decision" targetRef="sendRejectionMail">
      <conditionExpression xsi:type="tFormalExpression">
        <![CDATA[
          ${!approved}
        ]]>
      </conditionExpression>
    </sequenceFlow>

    <serviceTask id="externalSystemCall" name="Enter holidays in external system"
        flowable:class="org.flowable.CallExternalSystemDelegate"/>
    <sequenceFlow sourceRef="externalSystemCall" targetRef="holidayApprovedTask"/>
	<userTask id="holidayApprovedTask" name="Holiday approved"/>
    <sequenceFlow sourceRef="holidayApprovedTask" targetRef="approveEnd"/>

    <serviceTask id="sendRejectionMail" name="Send out rejection email"
        flowable:class="org.flowable.SendRejectionMail"/>
    <sequenceFlow sourceRef="sendRejectionMail" targetRef="rejectEnd"/>

    <endEvent id="approveEnd"/>

    <endEvent id="rejectEnd"/>

  </process>

</definitions>
```
第2行到第11行看起来有点令人生畏，但其实在大多数的流程定义中都是一样的。这是一种样板文件，需要与BPMN 2.0标准规范完全一致。

每一个步骤（在BPMN 2.0术语中称作活动 activity）都有一个id属性，为其提供一个在XML文件中唯一的标识符。所有的活动都可选有一个名字，以提高流程图的可读性。

活动之间通过顺序流(sequence flow)连接，在流程图中是一个有向箭头。在执行流程实例时，执行(execution)会从启动事件沿着顺序流流向下一个活动。

离开排他网关(带有X的菱形)的顺序流很特别：都以表达式(expression)的形式定义了条件(condition) （见第25至32行）。当流程实例的执行到达这个网关时，会计算条件，并使用第一个计算为true的顺序流。这就是排他的含义：只选择一个。当然如果需要不同的路由策略，可以使用其他类型的网关。

这里用作条件的表达式格式为${approved}，这是${approved == true}的简写。变量’approved’被称作流程变量(process variable)。流程变量是一个持久化数据，与流程实例存储在一起，并可以在流程实例的生命周期中使用。在这个例子里，我们需要在特定的地方（当经理用户任务提交时，或者以Flowable的术语来说，完成(complete)时）设置这个流程变量，因为这不是流程实例启动时就能获取的数据。

现在我们有了流程BPMN 2.0 XML文件，接下来我们需要将它部署到引擎中。部署流程定义意味着：

 - 流程引擎将XML文件存储在数据库中，因此可以在需要的时候获取它。
 - 流程定义被解析为内部可执行对象模型，这样使用它就可以启动流程实例。

将流程定义部署至Flowable引擎，需要使用RepositoryService，其可以从ProcessEngine对象获取。使用RepositoryService，可以通过传递XML文件的地址创建一个新的部署(Deployment)，并调用deploy()方法实际执行它：

```java?linenums
RepositoryService repositoryService = processEngine.getRepositoryService();
Deployment deployment = repositoryService.createDeployment()
  .addClasspathResource("holiday-request.bpmn20.xml")
  .deploy();
```

我们现在可以通过API查询验证流程定义已经部署在引擎中（并学习一些API）。通过RepositoryService创建的ProcessDefinitionQuery对象实现。
```java?linenums
ProcessDefinition processDefinition = repositoryService.createProcessDefinitionQuery()
  .deploymentId(deployment.getId())
  .singleResult();
System.out.println("Found process definition : " + processDefinition.getName());
```
### 2.3.3 启动一个流程实例

现在已经在流程引擎中部署了流程定义，因此可以使用这个流程定义作为蓝图启动流程实例。

要启动流程实例，需要提供一些初始化流程变量。一般来说，可以通过呈现给用户的表单，或者在流程由其他系统自动触发时通过REST API，来获取这些变量。在这个例子里，我们简化为使用java.util.Scanner类在命令行输入一些数据：
```java?linenums
Scanner scanner= new Scanner(System.in);

System.out.println("Who are you?");
String employee = scanner.nextLine();

System.out.println("How many holidays do you want to request?");
Integer nrOfHolidays = Integer.valueOf(scanner.nextLine());

System.out.println("Why do you need them?");
String description = scanner.nextLine();
```
 接下来，我们使用RuntimeService启动一个流程实例。收集的数据作为一个java.util.Map实例传递，其中的键就是之后用于获取变量的标识符。这个流程实例使用key启动。这个key就是BPMN 2.0 XML文件中设置的id属性，在这个例子里是holidayRequest。

（请注意：除了使用key之外，在后面你还会看到有很多其他方式启动一个流程实例）
```xml?linenums
<process id="holidayRequest" name="Holiday Request" isExecutable="true">
```

```java?linenums
RuntimeService runtimeService = processEngine.getRuntimeService();

Map<String, Object> variables = new HashMap<String, Object>();
variables.put("employee", employee);
variables.put("nrOfHolidays", nrOfHolidays);
variables.put("description", description);
ProcessInstance processInstance =
  runtimeService.startProcessInstanceByKey("holidayRequest", variables);
```
 在流程实例启动后，会创建一个执行(execution)，并将其放在启动事件上。从这里开始，这个执行沿着顺序流移动到经理审批的用户任务，并执行用户任务行为。这个行为将在数据库中创建一个任务，该任务可以之后使用查询找到。用户任务是一个等待状态(wait state)，引擎会停止继续执行，返回API调用。

### 2.3.4 另一个话题：事务性
在Flowable中，数据库事务扮演了关键角色，保证了数据一致性，并解决并发问题。当调用Flowable API时，默认情况下，所有操作都是同步的，并处于同一个事务下。这意味着，当方法调用返回时，会启动并提交一个事务。

流程启动后，会有一个数据库事务从流程实例启动时持续到下一个等待状态。在这个例子里，指的是第一个用户任务。当引擎到达这个用户任务时，状态会持久化至数据库，提交事务，并返回API调用。

在Flowable中，当一个流程实例运行时，总会有一个数据库事务从前一个等待状态持续到下一个等待状态。数据持久化之后，可能在数据库中保存很长时间，甚至几年，直到某个API调用使流程实例继续执行。请注意当流程处在等待状态时，不会消耗任何计算或内存资源。

在这个例子中，当第一个用户任务完成时，会启动一个数据库事务，从用户任务开始，经过排他网关（自动逻辑），直到第二个用户任务。或通过另一条路径直接到达结束。
### 2.3.5 查询和完成事务
在更实际的应用中，会为雇员及经理提供用户界面，让他们可以登入并查看任务列表。其中可以看到作为流程变量存储的流程实例数据，并决定如何操作任务。在这个例子中，我们通过执行API调用模拟任务列表，通常这些API都是由UI驱动的服务在后台调用的。

我们还没有为用户任务配置办理人。我们想将第一个任务指派给"经理(managers)"组，而第二个用户任务指派给请假申请的提交人。这需要为第一个任务添加candidateGroups属性：
```xml?linenums
<userTask id="approveTask" name="Approve or reject request" flowable:candidateGroups="managers"/>
```
 并如下所示为第二个任务添加assignee属性。请注意我们没有像上面的’managers’一样使用静态值，而是使用一个流程变量动态指派。这个流程变量是在流程实例启动时传递的：
```xml?linenums
<userTask id="holidayApprovedTask" name="Holiday approved" flowable:assignee="${employee}"/>
```
要获得实际的任务列表，需要通过TaskService创建一个TaskQuery。我们配置这个查询只返回’managers’组的任务：
```java?linenums
TaskService taskService = processEngine.getTaskService();
List<Task> tasks = taskService.createTaskQuery().taskCandidateGroup("managers").list();
System.out.println("You have " + tasks.size() + " tasks:");
for (int i=0; i<tasks.size(); i++) {
  System.out.println((i+1) + ") " + tasks.get(i).getName());
}
```
可以使用任务Id获取特定流程实例的变量，并在屏幕上显示实际的申请：
```java?linenums
System.out.println("Which task would you like to complete?");
int taskIndex = Integer.valueOf(scanner.nextLine());
Task task = tasks.get(taskIndex - 1);
Map<String, Object> processVariables = taskService.getVariables(task.getId());
System.out.println(processVariables.get("employee") + " wants " +
    processVariables.get("nrOfHolidays") + " of holidays. Do you approve this?");
```
如果你运行它，应该看起来像这样：

![enter description here](./images/1535512648581.png)

经理现在就可以完成任务了。在现实中，这通常意味着由用户提交一个表单。表单中的数据作为流程变量传递。在这里，我们在完成任务时传递带有’approved’变量（这个名字很重要，因为之后会在顺序流的条件中使用！）的map来模拟
```java?linenums
boolean approved = scanner.nextLine().toLowerCase().equals("y");
variables = new HashMap<String, Object>();
variables.put("approved", approved);
taskService.complete(task.getId(), variables);
```
现在任务完成，并会在离开排他网关的两条路径中，基于’approved’流程变量选择一条。

### 2.3.6. 编写一个JavaDelegate

 拼图还缺了一块：我们还没有实现在申请通过后执行的自动逻辑。在BPMN 2.0 XML中，这是一个服务任务(service task)，长这样：
```xml?linenums
<serviceTask id="externalSystemCall" name="Enter holidays in external system"
    flowable:class="org.flowable.CallExternalSystemDelegate"/>
```
在现实中，这个逻辑可以做任何事情：向某个系统发起一个HTTP REST服务调用，或调用某个使用了好几十年的系统中的遗留代码。我们不会在这里实现实际的逻辑，而只是简单的日志记录流程。

创建一个新的类(在Eclipse里File → New → Class)，填入org.flowable作为包名，CallExternalSystemDelegate作为类名。让这个类实现org.flowable.engine.delegate.JavaDelegate接口，并实现execute方法：
```java?linenums
package org.flowable;

import org.flowable.engine.delegate.DelegateExecution;
import org.flowable.engine.delegate.JavaDelegate;

public class CallExternalSystemDelegate implements JavaDelegate {

    public void execute(DelegateExecution execution) {
        System.out.println("Calling the external system for employee "
            + execution.getVariable("employee"));
    }

}
```
当执行到达服务任务时，会初始化并调用BPMN 2.0 XML中引用的类。

现在执行这个例子的时候，日志信息就会显示出，已经执行了自定义逻辑：

![enter description here](./images/1535512648581_1.png)

### 2.3.7 使用历史数据
选择使用Flowable这样的流程引擎的原因之一，是它可以自动存储所有流程实例的审计数据或历史数据。这些数据可以用于创建报告，深入展现组织运行的情况，瓶颈在哪里，等等。

例如，如果希望显示流程实例已经执行的时间，就可以从ProcessEngine获取HistoryService，并创建历史活动(historical activities)的查询。在下面的代码片段中，可以看到我们添加了一些额外的过滤：

 - 只选择一个特定流程实例的活动
 - 只选择已完成的活动

结果也按结束时间排序，这意味着我们我们可以获取其执行顺序。
```java?linenums
HistoryService historyService = processEngine.getHistoryService();
List<HistoricActivityInstance> activities =
  historyService.createHistoricActivityInstanceQuery()
   .processInstanceId(processInstance.getId())
   .finished()
   .orderByHistoricActivityInstanceEndTime().asc()
   .list();

for (HistoricActivityInstance activity : activities) {
  System.out.println(activity.getActivityId() + " took "
    + activity.getDurationInMillis() + " milliseconds");
}
```
再次运行示例，我们现在在控制台中看到类似的内容：
```xml
startEvent花了1毫秒
approveTask花了2638毫秒
决定花了3毫秒
externalSystemCall花了1毫秒
```
### 2.3.8 小结
这个教程介绍了各种Flowable与BPMN 2.0的概念与术语，也展示了如何编程使用Flowable API。

当然，这只是旅程的开始。下面的章节会更深入介绍许多Flowable引擎支持的选项与特性。其他章节介绍安装与使用Flowable引擎的不同方法，并详细介绍了可用的所有BPMN 2.0结构。
## 2.4. Flowable REST API入门
 这个章节展示了与上一节相同的例子：部署一个流程定义，启动一个流程实例，获取任务列表并完成一个任务。最好先快速浏览上一章节以了解所做的事情。
 
 这一次，将使用Flowable REST API而不是Java API。你很快就会意识到REST API与Java API紧密关联。只要了解一个，就能很快学会另一个。

### 2.4.1. 安装REST应用
 从flowable.org网站下载.zip文件后，可以在wars目录下找到REST应用。要运行这个WAR文件，需要一个servlet容器，例如Tomcat、Jetty等。

使用Tomcat的步骤如下：

 - 下载并解压缩最新的Tomcat zip文件（在Tomcat网站中选择’Core’发行版）。
 - 将flowable-rest.war文件从解压的Flowable发行版目录中复制到解压的Tomcat目录下的webapps文件夹下。
 - 使用命令行，转到Tomcat目录下的bin文件夹。
 - 执行'./catalina run'启动Tomcat服务器。

在服务启动过程中，会显示一些Flowable日志信息。在最后显示的一条类似'INFO [main] org.apache.catalina.startup.Catalina.start Server startup in xyz ms'的消息标志着服务器已经启动，可以接受请求。请注意默认情况下，使用H2内存数据库，这意味着数据在服务器重启后会丢失。

在以下部分中，我们将使用cURL演示各种REST调用。默认情况下，所有REST调用都受基本身份验证保护。所有调用的用户都是rest-admin，密码都是test。

在启动后，通过执行下列命令验证应用运行正常
```
curl --user rest-admin:test http://localhost:8080/flowable-rest/service/management/engine
```
如果您返回正确的json响应，则REST API已启动并正在运行。

### 2.4.2. 部署流程定义
第一步是部署一个流程定义。使用REST API时，需要将一个.bpmn20.xml文件（或对于多个流程引擎，一个.zip文件）作为’multipart/formdata’上传：
```
curl --user rest-admin:test -F "file=@holiday-request.bpmn20.xml" http://localhost:8080/flowable-rest/service/repository/deployments
```
要验证是否正确部署了流程定义，可以请求流程定义列表：
```
curl --user rest-admin:test http://localhost:8080/flowable-rest/service/repository/process-definitions
```
它返回当前部署到引擎的所有流程定义的列表。
### 2.4.3. 启动流程实例
使用REST API启动一个流程实例与使用Java API很像：提供key作为流程定义的标识，并使用一个map作为初始化流程变量：
```
curl --user rest-admin:test -H "Content-Type: application/json" -X POST -d '{ "processDefinitionKey":"holidayRequest", "variables": [ { "name":"employee", "value": "John Doe" }, { "name":"nrOfHolidays", "value": 7 }]}' http://localhost:8080/flowable-rest/service/runtime/process-instances
```
这将返回类似：
```json?linenums
{"id":"43","url":"http://localhost:8080/flowable-rest/service/runtime/process-instances/43","businessKey":null,"suspended":false,"ended":false,"processDefinitionId":"holidayRequest:1:42","processDefinitionUrl":"http://localhost:8080/flowable-rest/service/repository/process-definitions/holidayRequest:1:42","activityId":null,"variables":[],"tenantId":"","completed":false}
```
### 2.4.4. 任务列表并完成任务
当流程实例启动后，第一个任务会指派给’managers’组。要获取这个组的所有任务，可以通过REST API进行任务查询：
```
curl --user rest-admin:test -H "Content-Type: application/json" -X POST -d '{ "candidateGroup" : "managers" }' http://localhost:8080/flowable-rest/service/query/tasks
```
这将返回’manager’组的所有任务的列表。

这样的任务可以如此完成：
```shell
curl --user rest-admin:test -H "Content-Type: application/json" -X POST -d '{ "action" : "complete", "variables" : [ { "name" : "approved", "value" : true} ]  }' http://localhost:8080/flowable-rest/service/runtime/tasks/25
```
但是，您很可能会收到如下错误：
```shell
{"message":"Internal server error","exception":"couldn't instantiate class org.flowable.CallExternalSystemDelegate"}
```
这意味着引擎无法找到服务任务引用的CallExternalSystemDelegate类。要解决这个错误，需要将该类放在应用的classpath下（并需要重启应用）。按照《Flowable基础二 Flowable是什么》的介绍创建该类，并将其打包为JAR，放在Tomcat的webapps目录下的flowable-rest目录下的WEB-INF/lib目录下。
