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

在`org.springframework.beans`和`org.springframework.context`包是Spring框架的IoC容器的基础。 `BeanFactory` 接口提供了一种能够管理任何类型对象的高级配置机制。 ApplicationContext 是一个子界面BeanFactory。它增加了与Spring的AOP功能的更容易的集成; 消息资源处理（用于国际化），事件发布; 和特定于应用程序层的上下文，例如WebApplicationContext 在Web应用程序中使用的上下文。

简而言之，它BeanFactory提供了配置框架和基本功能，并ApplicationContext添加了更多企业特定的功能。它ApplicationContext是完整的超集，BeanFactory在本章中专门用于Spring的IoC容器的描述。有关使用BeanFactory而不是ApplicationContext,引用 BeanFactory的更多信息。

在Spring中，构成应用程序主干并由Spring IoC 容器管理的对象称为bean。bean是一个由Spring IoC容器实例化，组装和管理的对象。否则，bean只是应用程序中众多对象之一。Bean及其之间的依赖 关系反映在容器使用的配置元数据中。
### 1.2. 容器概述
该接口org.springframework.context.ApplicationContext代表Spring IoC容器，负责实例化，配置和组装上述bean。容器通过读取配置元数据获取有关要实例化，配置和组装的对象的指令。配置元数据以XML，Java注释或Java代码表示。它允许您表达组成应用程序的对象以及这些对象之间丰富的相互依赖性。

ApplicationContextSpring的开箱即用的几个接口实现。在独立应用程序中，通常创建ClassPathXmlApplicationContext 或的实例 FileSystemXmlApplicationContext。虽然XML是定义配置元数据的传统格式，但您可以通过提供少量XML配置来声明性地支持这些其他元数据格式，从而指示容器使用Java注释或代码作为元数据格式。

在大多数应用程序方案中，不需要显式用户代码来实例化Spring IoC容器的一个或多个实例。例如，在Web应用程序场景中，应用程序文件中的简单八行（左右）样板Web描述符XML web.xml通常就足够了（请参阅Web应用程序的便捷ApplicationContext实例化）。如果您使用的是 Spring工具套件 Eclipse驱动的开发环境，只需点击几下鼠标或按键即可轻松创建此样板文件配置。

下图是Spring工作原理的高级视图。您的应用程序类与配置元数据相结合，以便在ApplicationContext创建和初始化之后，您拥有完全配置且可执行的系统或应用程序。
#### 1.2.1 配置元数据
如上图所示，Spring IoC容器使用一种配置元数据形式 ; 此配置元数据表示您作为应用程序开发人员如何告诉Spring容器在应用程序中实例化，配置和组装对象。

传统上，配置元数据以简单直观的XML格式提供，本章大部分内容用于传达Spring IoC容器的关键概念和功能。

```
基于XML的元数据不是唯一允许的配置元数据形式。Spring IoC容器本身完全与实际编写此配置元数据的格式分离。目前，许多开发人员为其Spring应用程序选择 基于Java的配置。
```
有关在Spring容器中使用其他形式的元数据的信息，请参阅：

- 基于注释的配置：Spring 2.5引入了对基于注释的配置元数据的支持。

- 基于Java的配置：从Spring 3.0开始，Spring JavaConfig项目提供的许多功能成为核心Spring Framework的一部分。因此，您可以使用Java而不是XML文件在应用程序类外部定义bean。要使用这些新功能，请参阅@Configuration，@Bean，@Import 和@DependsOn注释。

Spring配置由容器必须管理的至少一个且通常不止一个bean定义组成。基于XML的配置元数据显示这些bean配置为<bean/>顶级元素内的<beans/>元素。Java配置通常@Bean在@Configuration类中使用带注释的方法。

这些bean定义对应于构成应用程序的实际对象。通常，您定义服务层对象，数据访问对象（DAO），表示对象（如Struts Action实例），基础结构对象（如Hibernate SessionFactories，JMS Queues等）。通常，不会在容器中配置细粒度域对象，因为DAO和业务逻辑通常负责创建和加载域对象。但是，您可以使用Spring与AspectJ的集成来配置在IoC容器控制之外创建的对象。请参阅使用AspectJ使用Spring依赖注入域对象。

以下示例显示了基于XML的配置元数据的基本结构：
```xml?linenums
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```
该id属性是一个字符串，用于标识单个bean定义。该class属性定义bean的类型并使用完全限定的类名。id属性的值指的是协作对象。本例中未显示用于引用协作对象的XML; 有关更多信息，请参阅 依赖项。
#### 1.2.2 实例化容器
实例化Spring IoC容器非常简单。提供给ApplicationContext构造函数的位置路径实际上是资源字符串，允许容器从各种外部资源（如本地文件系统，Java等）加载配置元数据CLASSPATH。
```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```
```
在了解了Spring的IoC容器之后，您可能想要了解有关Spring Resource抽象的更多信息 ，如参考资料中所述，它提供了一种从URI语法中定义的位置读取InputStream的便捷机制。特别是，Resource路径用于构建应用程序上下文，如 应用程序上下文和资源路径中所述。
```
以下示例显示了服务层对象(services.xml)配置文件：
```xml?linenums
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```
以下示例显示了数据访问对象daos.xml文件：
```xml?linenums
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```
在前面的示例中，服务层由类PetStoreServiceImpl，和类型的两个数据访问对象JpaAccountDao和JpaItemDao（基于JPA对象/关系映射标准）。该property name元素是指JavaBean属性的名称，以及ref元素指的是另一个bean定义的名称。元素id和ref元素之间的这种联系表达了协作对象之间的依赖关系。有关配置对象的依赖项的详细信息，请参阅 依赖项。

编写基于XML的配置元数据
让bean定义跨越多个XML文件会很有用。通常，每个单独的XML配置文件都代表架构中的逻辑层或模块。

您可以使用应用程序上下文构造函数从所有这些XML片段加载bean定义。此构造函数采用多个Resource位置，如上一节中所示。或者，使用一个或多个<import/>元素来从另一个或多个文件加载bean定义。例如：
```xml?linenums
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```
在前面的例子中，外部豆定义是从三个文件加载： services.xml，messageSource.xml，和themeSource.xml。所有位置路径都与执行导入的定义文件相关，因此services.xml必须与执行导入的文件位于相同的目录或类路径位置， messageSource.xml而且themeSource.xml必须位于resources导入文件位置下方的位置。正如您所看到的，忽略了一个前导斜杠，但考虑到这些路径是相对的，最好不要使用斜杠。<beans/>根据Spring Schema，要导入的文件的内容（包括顶级元素）必须是有效的XML bean定义。
```
可以（但不建议）使用相对“../”路径引用父目录中的文件。这样做会对当前应用程序之外的文件创建依赖关系。特别是，不建议将此引用用于“classpath：”URL（例如，“classpath：../ services.xml”），其中运行时解析过程选择“最近的”类路径根，然后查看其父目录。类路径配置更改可能导致选择不同的，不正确的目录。

您始终可以使用完全限定的资源位置而不是相对路径：例如，“file：C：/config/services.xml”或“c​​lasspath：/config/services.xml”。但是，请注意您将应用程序的配置与特定的绝对位置耦合。通常最好为这样的绝对位置保持间接，例如，通过在运行时针对JVM系统属性解析的“$ {...}”占位符。
```
import指令是beans命名空间本身提供的功能。除了普通bean定义之外的其他配置功能在Spring提供的一系列XML命名空间中可用，例如“context”和“util”命名空间。

Groovy Bean定义DSL
作为外部化配置元数据的另一个示例，bean定义也可以在Spring的Groovy Bean定义DSL中表示，如Grails框架中所知。通常，此类配置将存在“.groovy”文件中，其结构如下：
```java?linenums
beans {
    dataSource(BasicDataSource) {
        driverClassName = "org.hsqldb.jdbcDriver"
        url = "jdbc:hsqldb:mem:grailsDB"
        username = "sa"
        password = ""
        settings = [mynew:"setting"]
    }
    sessionFactory(SessionFactory) {
        dataSource = dataSource
    }
    myService(MyService) {
        nestedBean = { AnotherBean bean ->
            dataSource = dataSource
        }
    }
}
```
此配置样式在很大程度上等同于XML bean定义，甚至支持Spring的XML配置命名空间。它还允许通过“importBeans”指令导入XML bean定义文件。
#### 1.2.3. 使用容器
它ApplicationContext是高级工厂的接口，能够维护不同bean及其依赖项的注册表。使用该方法，T getBean(String name, Class<T> requiredType)您可以检索Bean的实例。

在ApplicationContext可以读取bean定义并访问它们，如下所示：
```java?linenums
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```
使用Groovy配置，bootstrapping看起来非常相似，只是一个不同的上下文实现类，它是Groovy感知的（但也理解XML bean定义）：
```java?linenums
ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");
```
最灵活的变体是GenericApplicationContext与读者代表结合使用，例如与XmlBeanDefinitionReaderXML文件一起使用：
```java?linenums
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```
或者GroovyBeanDefinitionReader对于Groovy文件：
```java?linenums
GenericApplicationContext context = new GenericApplicationContext();
new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
context.refresh();
```
如果需要，这些读者代表可以在ApplicationContext不同的配置源中读取相同的读取bean定义。

然后，您可以使用它getBean来检索Bean的实例。该ApplicationContext 接口还有一些其他方法可用于检索bean，但理想情况下，您的应用程序代码绝不应使用它们。实际上，您的应用程序代码根本不应该调用该 getBean()方法，因此根本不依赖于Spring API。例如，Spring与Web框架的集成为各种Web框架组件（如控制器和JSF托管bean）提供依赖注入，允许您通过元数据（例如自动装配注释）声明对特定bean的依赖性。
### 1.3. Bean概述
Spring IoC容器管理一个或多个bean。这些bean是使用您提供给容器的配置元数据创建的，例如，以XML <bean/>定义的形式 。

在容器本身内，这些bean定义表示为BeanDefinition 对象，其中包含以下元数据（以及其他信息）：

- 包限定的类名：通常是正在定义的bean的实际实现类。

- Bean行为配置元素，说明bean在容器中的行为方式（范围，生命周期回调等）。

- 引用bean执行其工作所需的其他bean; 这些引用也称为协作者或依赖项。

- 要在新创建的对象中设置的其他配置设置，例如，在管理连接池的Bean中使用的连接数，或池的大小限制。

此元数据转换为组成每个bean定义的一组属性。
表1. bean定义
| 属性    |  说明   |
| --- | --- |
|     |     |
|     |     |
|     |     |
|     |     |
|     |     |
|     |     |
|     |     |
|     |     |
|     |     |
除了包含有关如何创建特定bean的信息的bean定义之外，这些ApplicationContext实现还允许用户注册在容器外部创建的现有对象。这是通过getBeanFactory()返回BeanFactory实现的方法访问ApplicationContext的BeanFactory来完成的DefaultListableBeanFactory。DefaultListableBeanFactory 支持通过方法该登记registerSingleton(..)和 registerBeanDefinition(..)。但是，典型应用程序仅适用于通过元数据bean定义定义的bean。
```
需要尽早注册Bean元数据和手动提供的单例实例，以便容器在自动装配和其他内省步骤期间正确推理它们。虽然在某种程度上支持覆盖现有元数据和现有单例实例，但是在运行时注册新bean（与对工厂的实时访问同时）并未得到官方支持，并且可能导致bean容器中的并发访问异常和/或不一致状态。
```
#### 1.3.1. 命名bean
每个bean都有一个或多个标识符。这些标识符在托管bean的容器中必须是唯一的。bean通常只有一个标识符，但如果它需要多个标识符，则额外的标识符可以被视为别名。

在基于XML的配置元数据中，使用id和/或name属性指定bean标识符。该id属性允许您指定一个id。通常，这些名称是字母数字（'myBean'，'fooService'等），但也可能包含特殊字符。如果要向bean引入其他别名，还可以在name 属性中指定它们，用逗号（,），分号（;）或空格分隔。作为历史记录，在Spring 3.1之前的版本中，该id属性被定义为一种xsd:ID类型，它约束了可能的字符。从3.1开始，它被定义为一种xsd:string类型。请注意，id容器仍然强制执行bean 唯一性，但不再是XML解析器。

您不需要为bean提供名称或ID。如果没有显式提供名称或标识，则容器会为该bean生成唯一的名称。但是，如果要通过名称引用该bean，则必须通过使用ref元素或 Service Locator样式查找来提供名称。不提供名称的动机与使用内部bean和自动装配协作者有关。
```
Bean命名约定
惯例是在命名bean时使用标准Java约定作为实例字段名称。也就是说，bean名称以小写字母开头，从那时起就是驼峰式的。这种名称的例子将是（不带引号）'accountManager'， 'accountService'，'userDao'，'loginController'，等等。

命名bean始终使您的配置更易于阅读和理解，如果您使用的是Spring AOP，那么在将建议应用于与名称相关的一组bean时，它会有很大帮助。
```
```
通过类路径中的组件扫描，Spring按照上述规则为未命名的组件生成bean名称：实质上，采用简单的类名并将其初始字符转换为小写。但是，在（不常见的）特殊情况下，当有多个字符并且第一个和第二个字符都是大写字母时，原始外壳将被保留。这些是java.beans.Introspector.decapitalize（Spring在这里使用的）定义的相同规则。
```
在bean定义之外别名bean
在bean定义本身中，您可以为bean提供多个名称，方法是使用id属性指定的最多一个名称和属性中的任意数量的其他名称name。这些名称可以是同一个bean的等效别名，并且在某些情况下很有用，例如允许应用程序中的每个组件通过使用特定于该组件本身的bean名称来引用公共依赖项。

但是，指定实际定义bean的所有别名并不总是足够的。有时需要为其他地方定义的bean引入别名。在大型系统中通常就是这种情况，其中配置在每个子系统之间分配，每个子系统具有其自己的一组对象定义。在基于XML的配置元数据中，您可以使用该<alias/>元素来完成此任务。
```xml
<alias name="fromName" alias="toName"/>
```
在这种情况下，fromName在使用此别名定义之后，同名容器中的bean 也可以称为toName。

例如，子系统A的配置元数据可以通过名称引用数据源subsystemA-dataSource。子系统B的配置元数据可以通过名称引用数据源subsystemB-dataSource。在编写使用这两个子系统的主应用程序时，主应用程序通过名称引用DataSource myApp-dataSource。要使所有三个名称引用您添加到MyApp配置元数据的同一对象，请使用以下别名定义：
```xml
<alias name="subsystemA-dataSource" alias="subsystemB-dataSource"/>
<alias name="subsystemA-dataSource" alias="myApp-dataSource" />
```
现在，每个组件和主应用程序都可以通过一个唯一的名称引用dataSource，并保证不与任何其他定义冲突（有效地创建命名空间），但它们引用相同的bean。
```
Java的配置
如果您使用的是Java配置，则@Bean可以使用注释来提供别名，请参阅使用@Bean注释了解详细信息。
```
#### 1.3.2. 实例化bean
bean定义本质上是用于创建一个或多个对象的配方。容器在被询问时查看命名bean的配方，并使用由该bean定义封装的配置元数据来创建（或获取）实际对象。

如果使用基于XML的配置元数据，则指定要在元素的class属性中实例化的对象的类型（或类）<bean/>。此 class属性在内部是 实例Class上的属性BeanDefinition，通常是必需的。（有关例外，请参阅 使用实例工厂方法和Bean定义继承实例化。）您可以通过Class以下两种方式之一使用该属性：

- 通常，在容器本身通过反向调用其构造函数直接创建bean的情况下指定要构造的bean类，稍微等同于使用new运算符的Java代码。

- 要指定包含static将被调用以创建对象的工厂方法的实际类，在不太常见的情况下，容器在类上调用 static 工厂方法来创建bean。从调用static工厂方法返回的对象类型可以完全是同一个类或另一个类。

```
内部类名
如果要为static嵌套类配置bean定义，则必须使用嵌套类的二进制名称。

例如，如果你有一个Foo在com.example包中调用的类，并且这个 Foo类有一个static被调用的嵌套类Bar，那么'class' bean定义中的属性值将是......

com.example.Foo$Bar

请注意，使用$名称中的字符将嵌套类名与外部类名分开。
```
使用构造函数实例化
当您通过构造方法创建bean时，所有普通类都可以使用并与Spring兼容。也就是说，正在开发的类不需要实现任何特定接口或以特定方式编码。简单地指定bean类就足够了。但是，根据您为该特定bean使用的IoC类型，您可能需要一个默认（空）构造函数。

Spring IoC容器几乎可以管理您希望它管理的任何类; 它不仅限于管理真正的JavaBeans。大多数Spring用户更喜欢实际的JavaBeans，只有一个默认（无参数）构造函数，并且在容器中的属性之后建模了适当的setter和getter。您还可以在容器中拥有更多异国情调的非bean样式类。例如，如果您需要使用绝对不符合JavaBean规范的旧连接池，那么Spring也可以对其进行管理。

使用基于XML的配置元数据，您可以按如下方式指定bean类：
```xml
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```
有关为构造函数提供参数的机制（如果需要）以及在构造对象后设置对象实例属性的详细信息，请参阅 注入依赖项。

使用静态工厂方法实例化
定义使用静态工厂方法创建的bean时，可以使用该class 属性指定包含static工厂方法的类和名为的属性，以factory-method指定工厂方法本身的名称。您应该能够调用此方法（使用后面描述的可选参数）并返回一个活动对象，随后将其视为通过构造函数创建的对象。这种bean定义的一个用途是static在遗留代码中调用工厂。

以下bean定义指定通过调用factory-method创建bean。该定义未指定返回对象的类型（类），仅指定包含工厂方法的类。在此示例中，该createInstance() 方法必须是静态方法。
```xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```
```java?linenums
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```
有关在从工厂返回对象后向工厂方法提供（可选）参数并设置对象实例属性的机制的详细信息，请参阅详细信息中的依赖关系和配置。

使用实例工厂方法实例化
与通过静态工厂方法实例化类似，使用实例工厂方法进行实例化会从容器调用现有bean的非静态方法来创建新bean。要使用此机制，请将该class属性保留为空，并在factory-bean属性中指定当前（或父/祖先）容器中bean的名称，该容器包含要调用以创建对象的实例方法。使用factory-method属性设置工厂方法本身的名称。
```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```
```java?linenums
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```
一个工厂类也可以包含多个工厂方法，如下所示：
```xml?linenums
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```
```java?linenums
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```
这种方法表明可以通过依赖注入（DI）来管理和配置工厂bean本身。请参阅详细信息中的依赖关系和配置。
```
在Spring文档中，工厂bean指的是在Spring容器中配置的bean，它将通过实例或 静态工厂方法创建对象 。相反， FactoryBean（注意大写）指的是特定于Spring的 FactoryBean 。
```
### 1.4. 依赖
典型的企业应用程序不包含单个对象（或Spring用法中的bean）。即使是最简单的应用程序也有一些对象可以协同工作，以呈现最终用户所看到的连贯应用程序。下一节将介绍如何定义多个独立的bean定义，以及对象协作实现目标的完全实现的应用程序。
#### 1.4.1. 依赖注入
依赖注入（DI）是一个过程，通过这个过程，对象定义它们的依赖关系，即它们使用的其他对象，只能通过构造函数参数，工厂方法的参数或在构造或返回对象实例后在对象实例上设置的属性。从工厂方法。然后容器在创建bean时注入这些依赖项。这个过程基本上是反向的，因此名称Inversion of Control（IoC），bean本身通过使用类的直接构造或服务定位器模式来控制其依赖项的实例化或位置。

使用DI原理的代码更清晰，当对象提供其依赖项时，解耦更有效。该对象不查找其依赖项，也不知道依赖项的位置或类。因此，您的类变得更容易测试，特别是当依赖关系在接口或抽象基类上时，这允许在单元测试中使用存根或模拟实现。

DI存在两个主要变体，基于构造函数的依赖注入和基于Setter的依赖注入。

基于构造函数的依赖注入
基于构造函数的 DI由容器调用具有多个参数的构造函数来完成，每个参数表示一个依赖项。调用static具有特定参数的工厂方法来构造bean几乎是等效的，本讨论同样处理构造函数和static工厂方法的参数。以下示例显示了一个只能通过构造函数注入进行依赖注入的类。请注意，此类没有什么特别之处，它是一个POJO，它不依赖于容器特定的接口，基类或注释。
```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```
构造函数参数解析
使用参数的类型进行构造函数参数解析匹配。如果bean定义的构造函数参数中不存在潜在的歧义，那么在bean定义中定义构造函数参数的顺序是在实例化bean时将这些参数提供给适当的构造函数的顺序。考虑以下课程：
```java
package x.y;

public class Foo {

    public Foo(Bar bar, Baz baz) {
        // ...
    }
}
```
假设Bar并且Baz类与继承无关，则不存在潜在的歧义。因此，以下配置工作正常，您不需要在<constructor-arg/> 元素中显式指定构造函数参数索引和/或类型。
```xml
<beans>
    <bean id="foo" class="x.y.Foo">
        <constructor-arg ref="bar"/>
        <constructor-arg ref="baz"/>
    </bean>

    <bean id="bar" class="x.y.Bar"/>

    <bean id="baz" class="x.y.Baz"/>
</beans>
```
当引用另一个bean时，类型是已知的，并且可以进行匹配（与前面的示例一样）。当使用简单类型时，例如 <value>true</value>，Spring无法确定值的类型，因此无法在没有帮助的情况下按类型进行匹配。考虑以下课程：
```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```
构造函数参数类型匹配
在前面的场景中，如果使用属性显式指定构造函数参数的类型，则容器可以使用与简单类型匹配的类型type。例如：
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```
构造函数参数索引
使用该index属性显式指定构造函数参数的索引。例如：
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```
除了解决多个简单值的歧义之外，指定索引还可以解决构造函数具有相同类型的两个参数的歧义。请注意， 索引基于0。

构造函数参数名称
您还可以使用构造函数参数名称进行值消歧：
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```
请记住，要使这项工作开箱即用，必须在启用调试标志的情况下编译代码，以便Spring可以从构造函数中查找参数名称。如果无法使用debug标志编译代码（或者不希望），则可以使用 @ConstructorProperties JDK批注显式命名构造函数参数。然后，示例类必须如下所示：
```java
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```
基于Setter的依赖注入
基于setter的 DI是在调用无参数构造函数或无参数static工厂方法来实例化bean之后，通过容器调用bean上的setter方法来完成的。

以下示例显示了一个只能使用纯setter注入进行依赖注入的类。这个类是传统的Java。它是一个POJO，它不依赖于容器特定的接口，基类或注释。
```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```
该ApplicationContext支架构造和基于setter方法的DI为它所管理的豆类。在通过构造函数方法注入了一些依赖项之后，它还支持基于setter的DI。您可以以a的形式配置依赖项，并将BeanDefinition其与PropertyEditor实例结合使用，以将属性从一种格式转换为另一种格式。然而，大多数Spring用户不直接与这些类（即，编程），而是用XML bean 定义，注释组件（即与注释类@Component， @Controller等等），或@Bean在基于Java的方法@Configuration类。然后，这些源在内部转换为实例BeanDefinition并用于加载整个Spring IoC容器实例。
```
基于构造函数或基于setter的DI？
由于您可以混合基于构造函数和基于setter的DI，因此将构造函数用于强制依赖项和setter方法或可选依赖项的配置方法是一个很好的经验法则。请注意， 在setter方法上使用@Required注释可用于使属性成为必需的依赖项。

Spring团队通常提倡构造函数注入，因为它使应用程序组件可以实现为不可变对象，并确保不需要所需的依赖项null。此外，构造函数注入的组件始终以完全初始化的状态返回到客户端（调用）代码。作为旁注，大量的构造函数参数是一个糟糕的代码气味，暗示该类可能有太多的责任，应该重构以更好地解决关注点的正确分离。

Setter注入应主要仅用于可在类中指定合理默认值的可选依赖项。否则，必须在代码使用依赖项的任何位置执行非空检查。setter注入的一个好处是setter方法使该类的对象可以在以后重新配置或重新注入。因此，通过JMX MBean进行管理是二次注入的一个引人注目的用例。

使用对特定类最有意义的DI样式。有时，在处理您没有源的第三方类时，会选择您。例如，如果第三方类没有公开任何setter方法，那么构造函数注入可能是唯一可用的DI形式。
```
依赖性解决过程
容器执行bean依赖性解析，如下所示：

- 使用ApplicationContext描述所有bean的配置元数据创建和初始化。可以通过XML，Java代码或注释指定配置元数据。

- 对于每个bean，如果使用的是依赖于普通构造函数的，那么它的依赖关系将以属性，构造函数参数或static-factory方法的参数的形式表示。实际创建 bean 时，会将这些依赖项提供给bean 。

- 每个属性或构造函数参数都是要设置的值的实际定义，或者是对容器中另一个bean的引用。

- 作为值的每个属性或构造函数参数都从其指定格式转换为该属性或构造函数参数的实际类型。默认情况下，Spring能够转换成字符串格式提供给所有的内置类型，比如数值int， long，String，boolean，等。

Spring容器在创建容器时验证每个bean的配置。但是，在实际创建 bean之前，不会设置bean属性本身。创建容器时会创建单例作用域并设置为预先实例化（默认值）的Bean。范围在Bean范围中定义。否则，仅在请求时才创建bean。创建bean可能会导致创建bean的图形，因为bean的依赖关系及其依赖关系（依此类推）被创建和分配。请注意，这些依赖项之间的解决方案不匹配可能会显示较晚，即首次创建受影响的bean时。
```
循环依赖
如果您主要使用构造函数注入，则可以创建无法解析的循环依赖关系场景。

例如：类A通过构造函数注入需要类B的实例，而类B通过构造函数注入需要类A的实例。如果将A类和B类的bean配置为相互注入，则Spring IoC容器会在运行时检测此循环引用，并抛出a BeanCurrentlyInCreationException。

一种可能的解决方案是编辑由setter而不是构造函数配置的某些类的源代码。或者，避免构造函数注入并仅使用setter注入。换句话说，尽管不推荐使用，但您可以使用setter注入配置循环依赖关系。

与典型情况（没有循环依赖）不同，bean A和bean B之间的循环依赖强制其中一个bean在完全初始化之前被注入另一个bean（一个经典的鸡/蛋场景）。
```
你通常可以信任Spring做正确的事。它在容器加载时检测配置问题，例如对不存在的bean和循环依赖的引用。当实际创建bean时，Spring会尽可能晚地设置属性并解析依赖项。这意味着，如果在创建该对象或其依赖项之一时出现问题，则在请求对象时，正确加载的Spring容器可以在以后生成异常。例如，bean因缺少属性或无效属性而抛出异常。这可能会延迟一些配置问题的可见性ApplicationContext默认情况下实现预实例化单例bean。以实际需要之前创建这些bean的一些前期时间和内存为代价，您ApplicationContext会在创建时发现配置问题，而不是更晚。您仍然可以覆盖此默认行为，以便单例bean将延迟初始化，而不是预先实例化。

如果不存在循环依赖，当一个或多个协作bean被注入依赖bean时，每个协作bean 在被注入依赖bean之前完全配置。这意味着如果bean A依赖于bean B，Spring IoC容器在调用bean A上的setter方法之前完全配置bean B.换句话说，bean被实例化（如果不是预先实例化的单例），设置依赖项，并调用相关的生命周期方法（如配置的init方法 或InitializingBean回调方法）。

依赖注入的例子
以下示例将基于XML的配置元数据用于基于setter的DI。Spring XML配置文件的一小部分指定了一些bean定义：
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```
```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```
在前面的示例中，setter被声明为与XML文件中指定的属性匹配。以下示例使用基于构造函数的DI：
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```
```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```
bean定义中指定的构造函数参数将用作构造函数的参数ExampleBean。

现在考虑这个示例的变体，其中不是使用构造函数，而是告诉Spring调用static工厂方法来返回对象的实例：
```xml
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```
```java
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }
}
```
static工厂方法的参数是通过<constructor-arg/>元素提供的，与实际使用的构造函数完全相同。工厂方法返回的类的类型不必与包含static工厂方法的类的类型相同，尽管在此示例中它是。实例（非静态）工厂方法将以基本相同的方式使用（除了使用factory-bean属性而不是class属性），因此这里不再讨论细节。
#### 1.4.2. 依赖关系和配置详细
如上一节所述，您可以将bean属性和构造函数参数定义为对其他托管bean（协作者）的引用，或者作为内联定义的值。Spring的基于XML的配置元数据为此目的支持其元素<property/>和<constructor-arg/>元素中的子元素类型 。

直值（基元，字符串等）
在value所述的属性<property/>元素指定属性或构造器参数的人类可读的字符串表示。Spring的 转换服务用于将这些值从a转换String为属性或参数的实际类型。
```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="masterkaoli"/>
</bean>
```
以下示例使用p命名空间进行更简洁的XML配置。
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="masterkaoli"/>

</beans>
```
前面的XML更简洁; 但是，除非您在创建bean定义时使用支持自动属性完成的IntelliJ IDEA或Spring Tool Suite（STS）等IDE，否则会在运行时而不是设计时发现拼写错误。强烈建议使用此类IDE帮助。

您还可以将java.util.Properties实例配置为：
```xml
<bean id="mappings"
    class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```
Spring容器通过使用JavaBeans 机制将<value/>元素内的文本转换为 java.util.Properties实例PropertyEditor。这是一个很好的快捷方式，并且是Spring团队支持<value/>在value属性样式上使用嵌套元素的少数几个地方之一。

idref元素
该idref元素只是一种防错方法，可以将容器中另一个bean 的id（字符串值 - 而不是引用）传递给<constructor-arg/>or或<property/> element。
```xml
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```
上面的bean定义代码段与以下代码段完全等效（在运行时）：
```xml
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```
第一种形式优于第二种形式，因为使用idref标签允许容器在部署时验证引用的命名bean实际存在。在第二个变体中，不对传递给bean 的targetName属性的值执行验证client。只有在client实际实例化bean 时才会发现错别字（很可能是致命的结果）。如果client bean是原型 bean，则只能在部署容器后很长时间才能发现此错误和结果异常。
```
4.0 beans xsd不再支持local该idref元素的属性，因为它不再提供常规bean引用的值。只需将现有idref local引用更改为idref bean升级到4.0架构。
```
其中一个共同的地方（至少在早期比Spring 2.0版本）<idref/>元素带来的值在配置AOP拦截在 ProxyFactoryBeanbean定义。<idref/>指定拦截器名称时使用元素可以防止拼写错误的拦截器ID。

引用其他bean（协作者）
所述ref元件是内部的最终元件<constructor-arg/>或<property/> 定义元素。在这里，您可以将bean的指定属性的值设置为对容器管理的另一个bean（协作者）的引用。引用的bean是bean的依赖项，其属性将被设置，并且在设置属性之前根据需要按需初始化。（如果协作者是单例bean，它可能已由容器初始化。）所有引用最终都是对另一个对象的引用。划定范围和有效性取决于是否通过指定其他对象的ID /名称bean，local,或parent属性。

通过标记的bean属性指定目标bean <ref/>是最常用的形式，并允许创建对同一容器或父容器中的任何bean的引用，而不管它是否在同一XML文件中。bean属性的值 可以id与目标bean 的属性相同，或者作为目标bean的name属性中的值之一。
```xml
<ref bean="someBean"/>
```
通过该parent属性指定目标bean 会创建对当前容器的父容器中的bean的引用。parent 属性的值可以id与目标bean 的属性相同，也可以与目标bean 的属性中的一个值相同name，并且目标bean必须位于当前bean的父容器中。您主要在拥有容器层次结构并且希望将现有bean包装在父容器中并使用与父bean具有相同名称的代理时使用此Bean引用变体。
```xml
<!-- in the parent context -->
<bean id="accountService" class="com.foo.SimpleAccountService">
    <!-- insert dependencies as required as here -->
</bean>
```
```xml
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```
```
4.0 beans xsd不再支持local该ref元素的属性，因为它不再提供常规bean引用的值。只需将现有ref local引用更改为ref bean升级到4.0架构。
```
内部bean
一个<bean>元素的元件<property/>或<constructor-arg/>元件定义了一个所谓的内部bean.
```xml
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```
内部bean定义不需要定义的id或名称; 如果指定，则容器不使用这样的值作为标识符。容器还会scope在创建时忽略标志：内部bean 始终是匿名的，并且始终使用外部bean创建它们。这是不是可以内部bean注入到协作bean以外进入封闭豆或独立访问它们。

作为一个极端情况，可以从自定义范围接收销毁回调，例如，对于包含在单例bean中的请求范围的内部bean：内部bean实例的创建将绑定到其包含的bean，但是销毁回调允许它参与请求范围的生命周期。这不是常见的情况; 内部bean通常只是共享其包含bean的范围。

集合
在<list/>，<set/>，<map/>，和<props/>元素，你将Java的性能和参数Collection类型List，Set，Map，和Properties分别。
```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```
映射键或值的值或设置值也可以是以下任何元素：
```xml
bean | ref | idref | list | set | map | props | value | null
```
合并合集
Spring容器还支持集合的合并。应用程序开发人员可以定义一个父风格<list/>，<map/>，<set/>或<props/>元素，并有孩子式的<list/>，<map/>，<set/>或<props/>元素继承和父集合覆盖值。也就是说，子集合的值是合并父集合和子集合的元素的结果，子集合的元素覆盖父集合中指定的值。

关于合并的这一部分讨论了父子bean机制。不熟悉父母和子bean定义的读者可能希望在继续之前阅读 相关部分。

以下示例演示了集合合并：
```xml
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```
注意在bean定义的属性元素 merge=true上使用该属性。当容器解析并实例化bean时，生成的实例有一个集合，其中包含子集合与父集合的合并结果 。<props/>adminEmailschildchildadminEmails PropertiesadminEmailsadminEmails。
```xml
administrator=administrator@example.com 
sales=sales@example.com 
support=support@example.co.uk
```
孩子Properties集合的值设置继承父所有属性元素<props/>，和孩子的为值support值将覆盖父集合的价值。

这一合并行为同样适用于<list/>，<map/>和<set/> 集合类型。在<list/>元素的特定情况下，保持与List集合类型相关联的语义，即ordered 值集合的概念; 父级的值位于所有子级列表的值之前。在的情况下Map，Set和Properties集合类型，没有顺序存在。因此，没有排序的语义在背后的关联的集合类型的效果Map，Set以及Properties该容器内部使用实现类型。

收集合并的局限性
您不能合并不同的集合类型（例如a Map和a List），如果您尝试这样做，Exception则会抛出适当的集合类型。merge必须在较低的继承子定义上指定该属性; merge在父集合定义上指定属性是多余的，不会导致所需的合并。

强烈的收藏品
通过在Java 5中引入泛型类型，您可以使用强类型集合。也就是说，可以声明一种Collection类型，使得它只能包含 String元素（例如）。如果您使用Spring依赖注入一个强类型Collection的bean，您可以利用Spring的类型转换支持，以便强类型Collection 实例的元素在被添加到之前转换为适当的类型Collection。
```java
public class Foo {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```
```xml
<beans>
    <bean id="foo" class="x.y.Foo">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```
当为注入准备bean 的accounts属性时，通过反射可获得foo关于强类型的元素类型的泛型信息Map<String, Float>。因此，Spring的类型转换基础结构将各种值元素识别为类型Float和字符串值9.99, 2.75，并将 3.99其转换为实际Float类型。

空值和空字符串值
Spring将属性等的空参数视为空Strings。以下基于XML的配置元数据片段将email属性设置为空 String值（“”）。
```xml
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```
上面的示例等效于以下Java代码：
```java
exampleBean.setEmail("");
```
该<null/>元素处理null的值。例如：
```xml
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```
以上配置等同于以下Java代码：
```java
exampleBean.setEmail(null);
```
带有p命名空间的XML快捷方式
p-namespace使您可以使用bean元素的属性而不是嵌套 <property/>元素来描述属性值和/或协作bean。

Spring支持具有命名空间的可扩展配置格式 ，这些命名空间基于XML Schema定义。beans本章中讨论的配置格式在XML Schema文档中定义。但是，p-namespace未在XSD文件中定义，仅存在于Spring的核心中。

以下示例显示了两个解析为相同结果的XML片段：第一个使用标准XML格式，第二个使用p命名空间。
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="classic" class="com.example.ExampleBean">
        <property name="email" value="foo@bar.com"/>
    </bean>

    <bean name="p-namespace" class="com.example.ExampleBean"
        p:email="foo@bar.com"/>
</beans>
```
该示例显示了bean定义中名为email的p命名空间中的属性。这告诉Spring包含一个属性声明。如前所述，p命名空间没有架构定义，因此您可以将属性的名称设置为属性名称。

下一个示例包括另外两个bean定义，它们都引用了另一个bean：
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="john-classic" class="com.example.Person">
        <property name="name" value="John Doe"/>
        <property name="spouse" ref="jane"/>
    </bean>

    <bean name="john-modern"
        class="com.example.Person"
        p:name="John Doe"
        p:spouse-ref="jane"/>

    <bean name="jane" class="com.example.Person">
        <property name="name" value="Jane Doe"/>
    </bean>
</beans>
```
如您所见，此示例不仅包含使用p命名空间的属性值，还使用特殊格式来声明属性引用。第一个bean定义用于<property name="spouse" ref="jane"/>创建从bean john到bean 的引用 jane，而第二个bean定义p:spouse-ref="jane"用作属性来执行完全相同的操作。在这种情况下spouse是属性名称，而该-ref部分表示这不是直接值，而是对另一个bean的引用。
```
p命名空间不如标准XML格式灵活。例如，声明属性引用的格式与结束的属性冲突Ref，而标准XML格式则不然。我们建议您仔细选择您的方法并将其传达给您的团队成员，以避免生成同时使用所有三种方法的XML文档。

```
带有c-namespace的XML快捷方式
与带有p-namespace的XML快捷方式类似，Spring 3.1中新引入的c-namespace允许使用内联属性来配置构造函数参数，而不是嵌套constructor-arg元素。

让我们回顾一下使用命名空间的基于构造函数的依赖注入的示例c:：
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="bar" class="x.y.Bar"/>
    <bean id="baz" class="x.y.Baz"/>

    <!-- traditional declaration -->
    <bean id="foo" class="x.y.Foo">
        <constructor-arg ref="bar"/>
        <constructor-arg ref="baz"/>
        <constructor-arg value="foo@bar.com"/>
    </bean>

    <!-- c-namespace declaration -->
    <bean id="foo" class="x.y.Foo" c:bar-ref="bar" c:baz-ref="baz" c:email="foo@bar.com"/>

</beans>
```
该c:命名空间使用相同的约定作为p:一个（后-ref为bean引用），供他们的名字设置构造函数的参数。同样，它需要声明，即使它没有在XSD架构中定义（但它存在于Spring核心内）。

对于构造函数参数名称不可用的罕见情况（通常如果编译的字节码没有调试信息），可以使用回退到参数索引：
```xml
<!-- c-namespace index declaration -->
<bean id="foo" class="x.y.Foo" c:_0-ref="bar" c:_1-ref="baz"/>
```
```
由于XML语法，索引表示法要求存在前导，_因为XML属性名称不能以数字开头（即使某些IDE允许它）。
```
在实践中，构造函数解析 机制在匹配参数方面非常有效，因此除非确实需要，否则我们建议在整个配置中使用名称表示法。

复合属性名称
设置bean属性时，可以使用复合或嵌套属性名称，只要除最终属性名称之外的路径的所有组件都不是null。请考虑以下bean定义。
```xml
<bean id="foo" class="foo.Bar">
    <property name="fred.bob.sammy" value="123" />
</bean>
```
所述foo豆具有fred属性，该属性具有bob属性，其具有sammy 特性，并且最终sammy属性被设置为值123。为了使这一工作，fred财产foo和bob财产fred绝不能 null豆后构造，或NullPointerException抛出。
#### 1.4.3. 使用依赖
如果bean是另一个bean的依赖项，通常意味着将一个bean设置为另一个bean的属性。通常，您可以使用基于XML的配置元数据中的<ref/> 元素来完成此操作。但是，有时bean之间的依赖关系不那么直接; 例如，需要触发类中的静态初始化程序，例如数据库驱动程序注册。depends-on在初始化使用此元素的bean之前，该属性可以显式强制初始化一个或多个bean。以下示例使用该depends-on属性表示对单个bean的依赖关系：
```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```
要表示对多个bean的依赖关系，请提供bean名称列表作为depends-on属性的值，使用逗号，空格和分号作为有效分隔符：
```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```
```
depends-onbean定义中的属性既可以指定初始化时间依赖性，也可以指定单独的 bean，相应的销毁时间依赖性。depends-on在给定的bean本身被销毁之前，首先销毁定义与给定bean 的关系的从属bean 。因此depends-on也可以控制关机顺序。
```
#### 1.4.4. 懒惰初始化的bean
默认情况下，ApplicationContext实现会急切地创建和配置所有 单例 bean作为初始化过程的一部分。通常，这种预先实例化是可取的，因为配置或周围环境中的错误是立即发现的，而不是几小时甚至几天后。如果不希望出现这种情况，可以通过将bean定义标记为延迟初始化来阻止单例bean的预实例化。延迟初始化的bean告诉IoC容器在第一次请求时创建bean实例，而不是在启动时创建。

在XML中，此行为由 元素lazy-init上的属性控制<bean/>; 例如：
```xml
<bean id="lazy" class="com.foo.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.foo.AnotherBean"/>
```
当前面的配置被a使用时ApplicationContext，命名的bean lazy在ApplicationContext启动时不会急切地预先实例化，而not.lazybean被急切地预先实例化。

但是，当延迟初始化的bean是未进行延迟初始化的单例bean的依赖项时 ，ApplicationContext会在启动时创建延迟初始化的bean，因为它必须满足单例的依赖关系。惰性初始化的bean被注入到其他地方的单例bean中，而不是懒惰初始化的。

您还可以通过使用元素default-lazy-init上的属性来控制容器级别的延迟初始化 <beans/>; 例如：
```xml
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```
#### 1.4.5. 自动装配协作者
Spring容器可以自动连接协作bean之间的关系。您可以通过检查内容来允许Spring自动为您的bean解析协作者（其他bean）ApplicationContext。自动装配具有以下优点：

- 自动装配可以显着减少指定属性或构造函数参数的需要。（在本章其他地方讨论的其他机制，如bean模板 ，在这方面也很有价值。）

- 自动装配可以随着对象的发展更新配置。例如，如果需要向类添加依赖项，则可以自动满足该依赖项，而无需修改配置。因此，自动装配在开发期间尤其有用，而不会在代码库变得更稳定时否定切换到显式布线的选项。

使用基于XML的配置元数据[ 2 ]时，可以使用元素的autowire属性为bean定义指定autowire模式<bean/>。自动装配功能有四种模式。您指定每个 bean的自动装配，因此可以选择要自动装配的那些。
表2.自动装配模式

|     |     |
| --- | --- |
|     |     |
|     |     |
|     |     |
|     |     |
使用byType或构造函数自动装配模式，您可以连接数组和类型集合。 在这种情况下，提供容器内与预期类型匹配的所有autowire候选者以满足依赖性。 如果预期的键类型为String，则可以自动装配强类型映射。 自动装配的Maps值将包含与预期类型匹配的所有Bean实例，而Maps键将包含相应的bean名称。

您可以将autowire行为与依赖性检查相结合，这将在自动装配完成后执行。
自动装配的局限和缺点
自动装配在项目中一致使用时效果最佳。 如果一般不使用自动装配，那么开发人员使用它来连接一个或两个bean定义可能会让人感到困惑。
考虑自动装配的局限和缺点：
- property和constructor-arg设置中的显式依赖项始终覆盖自动装配。 您无法自动装配所谓的简单属性，例如基元，字符串和类（以及此类简单属性的数组）。 这种限制是按设计的。

- 自动装配不如显式布线精确。 虽然如上表所示，Spring会小心避免在可能产生意外结果的歧义的情况下进行猜测，但不再明确记录Spring管理对象之间的关系。

- 可能无法为可能从Spring容器生成文档的工具提供接线信息。

- 容器中的多个bean定义可能与要自动装配的setter方法或构造函数参数指定的类型匹配。 对于数组，集合或地图，这不一定是个问题。 但是，对于期望单个值的依赖关系，这种模糊性不是任意解决的。 如果没有可用的唯一bean定义，则抛出异常。

在后一种情况下，您有几种选择：
- 放弃自动装配以支持显式布线。

- 通过将autowire-candidate属性设置为false，避免对bean定义进行自动装配，如下一节所述。

- 通过将其<bean />元素的primary属性设置为true，将单个bean定义指定为主要候选者。

- 如基于注释的容器配置中所述，实现基于注释的配置可用的更精细控制。

从自动装配中排除一个bean
在每个bean的基础上，您可以从自动装配中排除bean。 在Spring的XML格式中，将<bean />元素的autowire-candidate属性设置为false; 容器使特定的bean定义对自动装配基础结构不可用（包括注释样式配置，如@Autowired）。
```
The autowire-candidate attribute is designed to only affect type-based autowiring. It does not affect explicit references by name, which will get resolved even if the specified bean is not marked as an autowire candidate. As a consequence, autowiring by name will nevertheless inject a bean if the name matches.
```
您还可以根据针对bean名称的模式匹配来限制autowire候选者。 顶级<beans />元素在其default-autowire-candidates属性中接受一个或多个模式。 例如，要将autowire候选状态限制为名称以Repository结尾的任何bean，请提供值* Repository。 要提供多个模式，请在逗号分隔的列表中定义它们。 bean定义autowire-candidate属性的显式值true或false始终优先，对于此类bean，模式匹配规则不适用。

这些技术对于您永远不希望通过自动装配注入其他bean的bean非常有用。 这并不意味着排除的bean本身不能使用自动装配进行配置。 相反，bean本身不是自动装配其他bean的候选者。
#### 1.4.6. 方法注入
在大多数应用程序场景中，容器中的大多数bean都是 单例。当单例bean需要与另一个单例bean协作，或者非单例bean需要与另一个非单例bean协作时，通常通过将一个bean定义为另一个bean的属性来处理依赖关系。当bean生命周期不同时会出现问题。假设单例bean A需要使用非单例（原型）bean B，可能是在A上的每个方法调用上。容器只创建一次单例bean A，因此只有一次机会来设置属性。每次需要时，容器都不能为bean A提供bean B的新实例。

解决方案是放弃一些控制反转。您可以通过实现接口使bean A了解容器ApplicationContextAware，并通过对容器进行getBean（“B”）调用，每次bean A需要时都要求（通常是新的）bean B实例。以下是此方法的示例：
```java
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```
前面的内容是不可取的，因为业务代码知道并耦合到Spring Framework。Method Injection是Spring IoC容器的一个高级功能，它允许以干净的方式处理这个用例。
查找方法注入
Lookup方法注入是容器覆盖容器托管bean上的方法的能力 ，以返回容器中另一个命名bean的查找结果。查找通常涉及原型bean，如上一节中描述的场景。Spring Framework通过使用CGLIB库中的字节码生成来实现此方法注入，以动态生成覆盖该方法的子类。
```
为了使这个动态子类工作，Spring bean容器将子类化的类不能final，并且要重写的方法也不能final。

对具有abstract方法的类进行单元测试需要您自己对类进行子类化并提供该abstract方法的存根实现。

组件扫描也需要具体方法，这需要具体的类别来获取。

另一个关键限制是查找方法不适用于工厂方法，特别是@Bean配置类中的方法，因为容器在这种情况下不负责创建实例，因此无法在上面创建运行时生成的子类。飞。
```
查看CommandManager前面代码片段中的类，您会看到Spring容器将动态覆盖该createCommand() 方法的实现。您的CommandManager类将不具有任何Spring依赖项，如在重新编写的示例中可以看到的：
```java
package fiona.apple;

// no more Spring imports!

public abstract class CommandManager {

    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```
在包含要注入的方法的客户端类中（CommandManager在本例中），要注入的方法需要以下形式的签名：
```xml
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```
如果方法是abstract，则动态生成的子类实现该方法。否则，动态生成的子类将覆盖原始类中定义的具体方法。例如：
```xml
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```
标识为commandManager的bean createCommand() 在需要myCommand bean 的新实例时调用自己的方法。您必须小心将myCommandbean 部署为原型，如果这实际上是需要的话。如果它是一个单例，myCommand 则每次都返回相同的bean 实例。

或者，在基于注释的组件模型中，您可以通过@Lookup注释声明查找方法：
```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}
```
或者，更具惯用性，您可以依赖于针对查找方法的声明返回类型解析目标bean：
```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        MyCommand command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract MyCommand createCommand();
}
```
请注意，您通常会使用具体的存根实现来声明这种带注释的查找方法，以使它们与Spring的组件扫描规则兼容，其中默认情况下抽象类被忽略。此限制不适用于显式注册或显式导入的bean类。
```
访问不同范围的目标bean的另一种方法是ObjectFactory/ Provider注入点。将Scoped bean视为依赖项。

感兴趣的读者也可以找到ServiceLocatorFactoryBean（在 org.springframework.beans.factory.config包中）使用。
```
任意方法更换
与查找方法注入相比，一种不太有用的方法注入形式是能够使用另一个方法实现替换托管bean中的任意方法。用户可以安全地跳过本节的其余部分，直到实际需要该功能。

使用基于XML的配置元数据，您可以使用该replaced-method元素将已存在的方法实现替换为已部署的bean。考虑以下类，使用方法computeValue，我们要覆盖它：
```java
public class MyValueCalculator {

    public String computeValue(String input) {
        // some real code...
    }

    // some other methods...
}
```
实现org.springframework.beans.factory.support.MethodReplacer 接口的类提供新的方法定义。
```java
/**
 * meant to be used to override the existing computeValue(String)
 * implementation in MyValueCalculator
 */
public class ReplacementComputeValue implements MethodReplacer {

    public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
        // get the input value, work with it, and return a computed result
        String input = (String) args[0];
        ...
        return ...;
    }
}
```
部署原始类并指定方法覆盖的bean定义如下所示：
```xml
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
    <!-- arbitrary method replacement -->
    <replaced-method name="computeValue" replacer="replacementComputeValue">
        <arg-type>String</arg-type>
    </replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```
您可以<arg-type/>在<replaced-method/> 元素中使用一个或多个包含的元素来指示被覆盖的方法的方法签名。仅当方法重载且类中存在多个变体时，才需要参数的签名。为方便起见，参数的类型字符串可以是完全限定类型名称的子字符串。例如，以下所有匹配 java.lang.String：
```
java.lang.String
String
Str
```
因为参数的数量通常足以区分每个可能的选择，所以通过允许您只键入与参数类型匹配的最短字符串，此快捷方式可以节省大量的输入。

### 1.5. Bean 范围
创建bean定义时，可以创建用于创建该bean定义所定义的类的实际实例的配方。bean定义是一个配方的想法很重要，因为它意味着，与一个类一样，您可以从一个配方创建许多对象实例。

您不仅可以控制要插入到从特定bean定义创建的对象中的各种依赖项和配置值，还可以控制从特定bean定义创建的对象的范围。这种方法功能强大且灵活，您可以选择通过配置创建的对象的范围，而不必在Java类级别烘焙对象的范围。可以将Bean定义为部署在多个范围之一中：开箱即用，Spring Framework支持六个范围，其中四个范围仅在您使用Web感知时才可用ApplicationContext。

开箱即用支持以下范围。您还可以创建 自定义范围。
表3. Bean范围

|     |     |
| --- | --- |
|     |     |
|     |     |
|     |     |
|     |     |
|     |     |
|     |     |
```
从Spring 3.0开始，线程范围可用，但默认情况下未注册。有关更多信息，请参阅相关文档 SimpleThreadScope。有关如何注册此范围或任何其他自定义范围的说明，请参阅 使用自定义范围。
```
#### 1.5.1. 单身范围
只管理单个bean的一个共享实例，并且对具有与该bean定义匹配的id或id的bean的所有请求都会导致Spring容器返回一个特定的bean实例。

换句话说，当您定义一个bean定义并且它的范围是一个单例时，Spring IoC容器只创建该bean定义定义的对象的一个实例。此单个实例存储在此类单例bean的缓存中，并且该命名Bean的所有后续请求和引用都将返回缓存对象。
图
Spring的单例bean概念不同于Gang of Four（GoF）模式书中定义的Singleton模式。GoF Singleton对对象的范围进行硬编码，使得每个ClassLoader创建一个且只有一个特定类的实例。Spring单例的范围最好按容器和每个bean描述。这意味着如果在单个Spring容器中为特定类定义一个bean，则Spring容器将创建该bean定义所定义的类的一个且仅一个实例。单例范围是Spring中的默认范围。要将bean定义为XML中的单例，您可以编写，例如：
```xml
<bean id="accountService" class="com.foo.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.foo.DefaultAccountService" scope="singleton"/>
```
#### 1.5.2. 原型范围
bean的非单例原型范围部署导致每次发出对该特定bean的请求时都会创建一个新的bean实例。也就是说，将bean注入另一个bean，或者通过getBean()对容器的方法调用来请求它。通常，对所有有状态bean使用原型范围，对无状态bean使用单例范围。

下图说明了Spring原型范围。数据访问对象（DAO）通常不配置为原型，因为典型的DAO不保持任何会话状态; 这个作者更容易重用单例图的核心。
图
以下示例将bean定义为XML中的原型：
```xml
<bean id="accountService" class="com.foo.DefaultAccountService" scope="prototype"/>
```


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
## 6. Spring AOP API
### 6.1. 介绍
### 6.2. Spring中的Pointcut API
### 6.3. Spring中的Advice API
### 6.4. Spring中的Advisor API
### 6.5. 使用ProxyFactoryBean创建AOP代理
### 6.6. 简明的代理定义
### 6.7. 使用ProxyFactory以编程方式创建AOP代理
### 6.8. 操纵Advice对象
### 6.9. 使用“自动代理”工具
### 6.10. 使用TargetSources
### 6.11. 定义新的Advice类型
## 7. 无安全性
### 7.1. 用例
### 7.2. JSR 305元注释
## 8. 数据缓冲区和编解码器
### 8.1. 介绍
### 8.2. DataBufferFactory
### 8.3. DataBuffer接口
### 编解码器
## 9. 附录
### 9.1. XML模式
### 9.2. XML Schema Authoring