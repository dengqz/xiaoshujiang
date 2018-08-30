---
title: Spring Framework - Overview 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


# Spring框架概述
Spring可以轻松创建Java企业应用程序。它提供了在企业环境中使用Java语言所需的一切，支持Groovy和Kotlin作为JVM上的替代语言，并可根据应用程序的需要灵活地创建多种体系结构。从Spring Framework 5.0开始，Spring需要JDK 8+（Java SE 8+），并且已经为JDK 9提供了开箱即用的支持。

Spring支持广泛的应用场景。在大型企业中，应用程序通常存在很长时间，并且必须在升级周期超出开发人员控制范围的JDK和应用程序服务器上运行。其他可以作为单个jar运行，其中嵌入了服务器，可能在云环境中。还有一些可能是不需要服务器的独立应用程序（例如批处理或集成工作负载）。

Spring是开源的。它拥有一个庞大而活跃的社区，可根据各种各样的实际用例提供持续的反馈。这有助于Spring在很长一段时间内成功发展。
## “Spring”是什么意思
术语“Spring”在不同的背景下意味着不同的东西。它可以用来引用Spring Framework项目本身，它就是一切开始的地方。随着时间的推移，其他Spring项目已经构建在Spring Framework之上。大多数情况下，当人们说“Spring”时，他们指的是整个项目系列。本参考文档侧重于基础：Spring Framework本身。

Spring框架分为几个模块。应用程序可以选择所需的模块。核心是核心容器的模块，包括配置模型和依赖注入机制。除此之外，Spring Framework还为不同的应用程序体系结构提供了基础支持，包括消息传递，事务数据和持久性以及Web。它还包括基于Servlet的Spring MVC Web框架，以及Spring WebFlux响应式Web框架。

关于模块的说明：Spring的框架jar允许部署到JDK 9的模块路径（“Jigsaw”）。为了在支持Jigsaw的应用程序中使用，Spring Framework 5 jar带有“Automatic-Module-Name”清单条目，它定义了独立于jar工件的稳定语言级模块名称（“spring.core”，“spring.context”等）名称（罐子使用相同的命名模式，而不是“。”，例如“spring-core”和“spring-context”）。当然，Spring的框架jar在JDK 8和9上的类路径上都能正常工作。
## Spring的历史和Spring框架
Spring于2003年成立，是对早期J2EE规范复杂性的回应 。虽然有些人认为Java EE和Spring处于竞争中，但Spring实际上是对Java EE的补充。Spring编程模型不包含Java EE平台规范; 相反，它集成了EE保护伞中精心挑选的个别规格：

Servlet API（JSR 340）

WebSocket API（JSR 356）

并发实用程序（JSR 236）

JSON绑定API（JSR 367）

Bean验证（JSR 303）

JPA（JSR 338）

JMS（JSR 914）

以及必要时用于事务协调的JTA / JCA设置。

Spring Framework还支持依赖注入（JSR 330）和Common Annotations（JSR 250）规范，应用程序开发人员可以选择使用这些规范，而不是Spring Framework提供的Spring特定机制。

从Spring Framework 5.0开始，Spring至少需要Java EE 7级别（例如Servlet 3.1 +，JPA 2.1+） - 同时在Java EE 8级别提供与新API的开箱即用集成（例如，Servlet 4.0，JSON绑定API）在运行时遇到。这使Spring与Tomcat 8和9，WebSphere 9和JBoss EAP 7完全兼容。

随着时间的推移，Java EE在应用程序开发中的作用也在不断发展。在Java EE和Spring的早期，创建了应用程序以部署到应用程序服务器。今天，在Spring Boot的帮助下，应用程序以devops和云友好的方式创建，Servlet容器嵌入并且变得微不足道。从Spring Framework 5开始，WebFlux应用程序甚至不直接使用Servlet API，并且可以在不是Servlet容器的服务器（例如Netty）上运行。

Spring继续创新并不断发展。除了Spring Framework之外，还有其他项目，例如Spring Boot，Spring Security，Spring Data，Spring Cloud，Spring Batch等。重要的是要记住每个项目都有自己的源代码存储库，问题跟踪器和发布节奏。有关Spring项目的完整列表，请参见spring.io/projects。
## 设计理念
当您了解框架时，重要的是不仅要知道它的作用，还要了解它遵循的原则。以下是Spring Framework的指导原则：

提供各个层面的选择。Spring允许您尽可能晚地推迟设计决策。例如，您可以通过配置切换持久性提供程序，而无需更改代码。许多其他基础架构问题以及与第三方API的集成也是如此。

适应不同的观点。Spring拥抱灵活性，并不认为应该如何做。它以不同的视角支持广泛的应用需求。

保持强大的向后兼容性。Spring的演变经过精心设计，可以在版本之间进行一些重大改变。Spring支持精心挑选的JDK版本和第三方库，以便于维护依赖于Spring的应用程序和库。

关心API设计。Spring团队投入了大量的思考和时间来制作直观的API，这些API可以在多个版本和多年中保持不变。

为代码质量设定高标准。Spring Framework强调有意义，最新且准确的Javadoc。它是极少数项目之一，可以声称干净的代码结构，包之间没有循环依赖。
## 反馈和贡献
对于操作方法问题或诊断或调试问题，我们建议使用StackOverflow，我们有一个问题页面列出了要使用的建议标签。如果您非常确定Spring Framework中存在问题或想要建议功能，请使用JIRA问题跟踪器。

如果您有解决方案或建议的修复，您可以在Github上提交拉取请求 。但是，请记住，除了最微不足道的问题之外，我们希望在问题跟踪器中提交一张票据，进行讨论并留下记录以供将来参考。

有关更多详细信息，请参阅“ 贡献 ”顶级项目页面上的指南。
## 入门
如果您刚刚开始使用Spring，您可能希望通过创建基于Spring Boot的应用程序来开始使用Spring Framework 。Spring Boot提供了一种快速（和固执己见）的方式来创建一个生产就绪的基于Spring的应用程序。它基于Spring Framework，支持约定优于配置，旨在帮助您尽快启动和运行。

您可以使用start.spring.io生成基本项目，也可以按照“入门”指南之一进行操作，例如“ 入门构建RESTful Web服务”。除了更容易理解之外，这些指南非常注重任务，而且大多数都基于Spring Boot。它们还涵盖了Spring组合中您在解决特定问题时可能需要考虑的其他项目。