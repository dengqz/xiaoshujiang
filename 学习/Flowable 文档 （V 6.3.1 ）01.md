---
title: Flowable 文档 （V 6.3.1 ）01
tags: Flowable,工作流,springBoot
grammar_cjkRuby: true
---

# 1. 简介
## 1.1. 许可
Flowable 在[Apache V2许可](http://www.apache.org/licenses/LICENSE-2.0.html)下分发。
## 1.2. 下载
http://www.flowable.org/downloads.html
## 1.3. 源码
该发行版包含大多数源作为JAR文件。可以在上面找到Flowable的源代码 https://github.com/flowable/flowable-engine
## 1.4. 所需软件
### 1.4.1. JDK 8+
Flowable在JDK高于或等于版本8上运行。转到[Oracle Java SE](http://www.oracle.com/technetwork/java/javase/downloads/index.html)下载并单击“下载JDK”按钮。该页面上也有安装说明。要验证安装是否成功，请java -version在命令行上运行。这应该打印已安装的JDK版本。
### 1.4.2. IDE
可以使用您选择的IDE完成flowable开发。如果您想使用Flowable Designer，那么您需要Eclipse Mars或Neon。从Eclipse下载页面下载您选择的Eclipse发行版。解压缩下载的文件，然后您应该能够使用目录中的Eclipse文件启动它eclipse。在本指南中，有一节介绍如何安装我们的eclipse设计器插件
## 1.5. 报告问题
我们希望开发人员在报告或询问任何内容之前阅读如何以智能方式提问。

完成后，您可以在用户论坛上发布有关增强功能的问题，评论或建议，并在我们的Github问题跟踪器中创建错误问题。
## 1.6. 实验功能
标有[EXPERIMENTAL]的章节不应被视为稳定。

.impl.包名中包含的所有类都是内部实现类，不能被认为是稳定的或以任何方式保证。但是，如果“用户指南”提到任何类作为配置值，则它们受支持并且可以被认为是稳定的。
## 1.7. 内部实现类
在JAR文件中，包含.impl.（例如org.flowable.engine.impl.db）名称的包中的所有类都是实现类，应该仅被视为内部使用。对实现类中的类或接口没有给出稳定性保证。
## 1.8. 版本控制策略
版本使用标准的整数三元组表示：MAJOR.MINOR.MICRO。目的是MAJOR版本用于核心引擎的演变。MINOR版本用于新功能和新API。MICRO版本用于修复和改进错误。

通常，Flowable尝试在所有非内部实现类的MINOR和MICRO版本中保持“源兼容” 。我们定义“源兼容”意味着应用程序将继续构建而没有错误，并且语义保持不变。Flowable也试图在MINOR和MICRO版本中保持“二进制兼容” 。我们定义“二进制兼容”意味着可以将这个新版本的Flowable作为jar替换放入已编译的应用程序中并继续正常运行。

如果在MINOR版本中引入了API更改，则策略是在其中保留向后兼容的版本，使用@Deprecated进行注释。这些已弃用的API将在稍后的两个MINOR版本中删除。