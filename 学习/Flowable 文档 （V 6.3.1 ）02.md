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
 ![enter description here](./images/getting.started.new.maven.png)




## 2.4. Flowable REST API入门
