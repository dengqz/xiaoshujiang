---
title: Spring Framework - Core 
tags: IoC container, Events, Resources, i18n, Validation, Data Binding, Type Conversion, SpEL, AOP
grammar_cjkRuby: true
---


# 核心技术 Core Technologies
**这部分参考文档涵盖了Spring Framework绝对不可或缺的所有技术**。

其中最重要的是Spring Framework的控制反转（IoC）容器。对Spring框架的IoC容器进行彻底的处理后，我们将全面介绍Spring的面向方面编程（AOP）技术。Spring Framework有自己的AOP框架，概念上易于理解，并且成功地解决了Java企业编程中AOP要求的80％最佳点。

还提供了Spring与AspectJ集成的覆盖范围（目前最丰富的 - 在功能方面 - 当然也是Java企业领域中最成熟的AOP实现）。
## 1. IoC容器 The IoC container
### 1.1 Spring IoC容器和bean简介
本章介绍了Spring Framework实现的控制反转（IoC）[ 1 ]原理。IoC也称为依赖注入（DI）。这是一个过程，通过这个过程，对象定义它们的依赖关系，即它们使用的其他对象，只能通过构造函数参数，工厂方法的参数，或者在构造或从工厂方法返回后在对象实例上设置的属性。 。然后容器 在创建bean时注入这些依赖项。这个过程基本上是相反的，因此称为控制反转（IoC），bean本身通过使用类的直接构造或诸如服务定位器模式之类的机制来控制其依赖关系的实例化或位置。

在`org.springframework.beans`和`org.springframework.context`包是Spring框架的IoC容器的基础。该 BeanFactory 接口提供了一种能够管理任何类型对象的高级配置机制。 ApplicationContext 是一个子界面BeanFactory。它增加了与Spring的AOP功能的更容易的集成; 消息资源处理（用于国际化），事件发布; 和特定于应用程序层的上下文，例如WebApplicationContext 在Web应用程序中使用的上下文。

简而言之，它BeanFactory提供了配置框架和基本功能，并ApplicationContext添加了更多企业特定的功能。它ApplicationContext是完整的超集，BeanFactory在本章中专门用于Spring的IoC容器的描述。有关使用BeanFactory而不是ApplicationContext,引用 BeanFactory的更多信息。

在Spring中，构成应用程序主干并由Spring IoC 容器管理的对象称为bean。bean是一个由Spring IoC容器实例化，组装和管理的对象。否则，bean只是应用程序中众多对象之一。Bean及其之间的依赖 关系反映在容器使用的配置元数据中。
### 1.2. 容器概述
### 1.3. Bean概述
### 1.4. 依赖
### 1.5. Bean 范围
### 1.6. 自定义bean的本质
### 1.7. Bean定义继承
### 1.8. 集装箱扩建点
### 1.9. 基于注释的容器配置
### 1.10. 类路径扫描和托管组件
### 1.11. 使用JSR 330标准注释
### 1.12. 基于Java的容器配置
### 1.13. 环境抽象
### 1.13. 注册LoadTimeWeaver
### 1.15. ApplicationContext的其他功能
### 1.16. BeanFactory
## 2. 资源
### 2.1. 介绍
### 2.2. 资源界面
### 2.3. 内置资源实现
### 2.4. ResourceLoader
### 2.5. ResourceLoaderAware接口
### 2.6. 资源作为依赖项
### 2.7. 应用程序上下文和资源路径
## 3. 验证，数据绑定和类型转换
### 3.1. 介绍
### 3.2. 使用Spring的Validator接口进行验证
### 3.3. 解决错误消息的代码
### 3.4. Bean操作和BeanWrapper
### 3.5. Spring类型转换
### 3.6. Spring Field格式
### 3.7. 配置全局日期和时间格式
### 3.8. Spring验证
## 4. Spring表达语言（SpEL）
### 4.1. 介绍
### 4.2. 评估
### 4.3. bean定义中的表达式
### 4.4. 语言参考
### 4.5. 示例中使用的类
## 5. 使用Spring进行面向方面的编程
### 5.1. 介绍
### 5.2. @AspectJ支持
### 5.3. 基于模式的AOP支持
### 5.4. 选择要使用的AOP声明样式
### 5.5. 混合方面类型
### 5.6. 代理机制
### 5.7. 程序化创建@AspectJ Proxies
### 5.8. 在Spring应用程序中使用AspectJ
### 5.9. 更多资源
