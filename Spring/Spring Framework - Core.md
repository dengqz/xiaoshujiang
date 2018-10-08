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

|  模式   | 说明    |
| --- | --- |
|   no  |  （默认）不自动装配。豆引用必须通过定义ref元素。更改默认设置，不建议较大规模的部署，因为指定的合作者明确地给出了更大的控制和清晰度。从某种程度上说，它记录了系统的结构。   |
|  byName   | 根据属性名自动装配。春季查找具有相同的名称，需要被装配属性的bean。例如，如果一个bean定义设置的名字自动装配，它包含一个主属性（即，它有一个 setMaster（..）方法），春季查找名为的bean定义master，并用它来设置属性。    |
|   byType  | 如果允许的属性类型的只有一个豆在容器中存在作为自动连接的属性。如果存在一个以上的，致命的异常被抛出，这表明你可能不使用byType的自动装配的bean，。如果没有匹配的豆子，什么都不会发生; 该属性未设置。    |
|   constructor  | 类似于byType的，但适用于构造函数的参数。如果没有在容器中的构造函数参数类型的只有一个豆，一个致命的错误引发。    |
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
与其他作用域相比，Spring不管理原型bean的完整生命周期：容器实例化，配置和组装原型对象，并将其交给客户端，而不再记录该原型实例。因此，尽管无论范围如何都在所有对象上调用初始化生命周期回调方法，但在原型的情况下，不会调用已配置的销毁 生命周期回调。客户端代码必须清理原型范围的对象并释放原型bean所持有的昂贵资源。要让Spring容器释放原型范围的bean所拥有的资源，请尝试使用自定义bean后处理器，它包含对需要清理的bean的引用。

在某些方面，Spring容器关于原型范围bean的角色是Java new运算符的替代品。超过该点的所有生命周期管理必须由客户端处理。（有关Spring容器中bean的生命周期的详细信息，请参阅Lifecycle回调。）
#### 1.5.3. 具有原型bean依赖关系的单例bean
当您使用具有依赖于原型bean的单例作用域bean时，请注意 在实例化时解析依赖项。因此，如果依赖项将原型范围的bean注入到单例范围的bean中，则会实例化一个新的原型bean，然后将依赖注入到单例bean中。原型实例是唯一提供给单例范围bean的实例。

但是，假设您希望单例范围的bean在运行时重复获取原型范围的bean的新实例。您不能将原型范围的bean依赖注入到您的单例bean中，因为当Spring容器实例化单例bean并解析并注入其依赖项时，该注入只发生 一次。如果您需要在运行时多次使用原型bean的新实例，请参阅方法注入
#### 1.5.4. 请求，会话，应用程序和WebSocket范围
在request，session，application，和websocket范围是仅如果使用基于web的Spring可ApplicationContext实现（如 XmlWebApplicationContext）。如果你将这些范围与常规的Spring IoC容器一起使用ClassPathXmlApplicationContext，那么IllegalStateException就会抛出一个抱怨未知bean范围的东西。

初始Web配置
为了支持豆的范围界定在request，session，application，和 websocket（即具有web作用域bean），需要做少量的初始配置定义你的豆之前。（这是初始设置不必需的标准范围，singleton和prototype）。

如何完成此初始设置取决于您的特定Servlet环境。

如果您在Spring Web MVC中访问scoped bean，实际上是在Spring处理的请求中DispatcherServlet，则不需要特殊设置： DispatcherServlet已经公开了所有相关状态。

如果您使用Servlet 2.5 Web容器，并且在Spring之外处理请求 DispatcherServlet（例如，使用JSF或Struts时），则需要注册 org.springframework.web.context.request.RequestContextListener ServletRequestListener。对于Servlet 3.0+，可以通过WebApplicationInitializer 界面以编程方式完成。或者，或者对于旧容器，将以下声明添加到Web应用程序的web.xml文件中：
```xml
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>
    ...
</web-app>
```
或者，如果您的侦听器设置存在问题，请考虑使用Spring RequestContextFilter。过滤器映射取决于周围的Web应用程序配置，因此您必须根据需要进行更改。
```xml
<web-app>
    ...
    <filter>
        <filter-name>requestContextFilter</filter-name>
        <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>requestContextFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>

```


DispatcherServlet，RequestContextListener和RequestContextFilter所有做同样的事情，即绑定的HTTP请求对象的Thread是服务该请求。这使得豆类是请求-和会话范围可进一步向下的调用链。
申请范围

考虑一个bean定义以下XML配置：
```xml


<bean id="loginAction" class="com.foo.LoginAction" scope="request"/>


```


Spring容器创建的新实例LoginAction使用bean loginAction每个bean定义和每个HTTP请求。也就是说， loginAction豆是在HTTP请求级别作用域。您可以更改为你想要的，因为从同创建其他实例太多创建实例的内部状态loginActionbean定义不会看到这些状态的变化; 它们是特定于某个请求。当请求完成处理，即可以被限制在请求中的豆被丢弃。

当使用注解驱动组件或Java配置中，@RequestScope注释可以用于一个组件分配给request范围。
```java?linenums
@RequestScope
@Component
public class LoginAction {
    // ...
}
```
会议范围

考虑一个bean定义以下XML配置：
```xml


<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>


```


Spring容器创建的新实例UserPreferences使用bean userPreferencesbean定义为一个HTTP的寿命Session。换句话说，该userPreferencesbean是有效的HTTP范围的Session水平。与request-scoped豆类，您可以更改，就像你想要创建实例的内部状态，知道其他HTTP Session，它们也使用同一创建的实例的实例userPreferencesbean定义没有看到状态这些变化，因为它们是特殊于某个HTTP Session。当HTTP Session最终废弃，范围限定于特定的HTTP豆 Session也被丢弃。

当使用注解驱动组件或Java配置中，@SessionScope注释可以用于一个组件分配给session范围。

```java?linenums
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```
应用范围

考虑一个bean定义以下XML配置：
```xml


<bean id="appPreferences" class="com.foo.AppPreferences" scope="application"/>


```


Spring容器创建的新实例AppPreferences使用bean appPreferencesbean定义一次整个Web应用程序。也就是说， appPreferences豆是在作用域ServletContext级别，存储为普通 ServletContext属性。这有点类似于Spring的singleton的bean，但在两个重要方面有所不同：它是一个单独的每个ServletContext，而不是每个春天“的ApplicationContext”（对于其中有可能在任何给定的Web应用程序数），它实际上是暴露的，因此可见作为一个ServletContext属性。

当使用注解驱动组件或Java配置中，@ApplicationScope 注释可以用于一个组件分配给application范围。
```java?linenums
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```
作用域bean与依赖

Spring IoC容器管理不仅对象（bean）的实例化，也合作者（或依赖）的接线起来。如果要注入（例如）HTTP请求范围的bean为寿命较长，范围的另外一个bean，你可以选择在地方的作用域bean注入一个AOP代理。也就是说，你需要注入暴露出一样的公共接口作用域对象的代理对象，但也可以检索相关范围的真正目标对象（如HTTP请求），并委托方法调用到真正的对象。

```


您也可以使用<aop:scoped-proxy/>该范围限定为豆之间singleton，与当时的参考通过中间代理是序列化的，因此能够重新获得反序列化的目标单豆去。

当声明<aop:scoped-proxy/>反对的范围豆prototype，在共享代理每一个方法调用都会导致创建该呼叫随后被转发到一个新的目标实例。

另外，范围代理不访问生命周期中的安全方式从较短范围豆的唯一途径。你也可以简单地声明您的注入点（即构造/ setter方法参数或自动装配领域）为ObjectFactory<MyTargetBean>，允许一个getObject()呼叫到每一个需要它时按需检索当前实例-未持有到该实例或单独存放。

作为一个扩展的变种，你可以声明ObjectProvider<MyTargetBean>它提供了一些额外的访问变体，包括getIfAvailable和getIfUnique。

这样做的JSR-330的变体被称为Provider，与使用Provider<MyTargetBean> 的声明和相应get()呼吁每一个检索的尝试。见这里对JSR-330的详细信息整体。
```
在下面的示例中的配置只有一行，但是理解“为何这么做”以及“如何做”是很重要的。
```xml


<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- an HTTP Session-scoped bean exposed as a proxy -->
    <bean id="userPreferences" class="com.foo.UserPreferences" scope="session">
        <!-- instructs the container to proxy the surrounding bean -->
        <aop:scoped-proxy/>
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.foo.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
</beans>


```


要创建这样的代理，你插入一个子<aop:scoped-proxy/>元素插入一个定义的名字（见选择代理的类型来创建和 基于XML Schema的配置）。豆类的定义为何作用域的request，session和自定义范围水平要求<aop:scoped-proxy/>元素？让我们来看看下面的单例的bean定义，并对比它与你需要定义为上述范围是什么（请注意以下 userPreferences，因为它代表bean定义是不完整的）。

```xml


<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>

<bean id="userManager" class="com.foo.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>


```


在前面的示例中，单豆userManager被注入到HTTP参考Session-scoped豆userPreferences。这里最关键的一点是， userManagerbean是一个单：它会被实例化只有一次每个集装箱，和它的依赖（在这种情况下，只有一个，在userPreferences豆）也仅注射一次。这意味着userManagerbean将只完全相同的操作上userPreferences的对象，也就是说，它最初被注入的一个。

这是不是注射时更短住作用域的bean为寿命较长的作用域的bean，例如注入HTTP你想要的行为Session-scoped合作bean作为依赖，进入单豆。相反，你需要一个userManager 对象，并为HTTP的寿命Session，你需要一个userPreferences对象，它是专用于所述HTTP Session。因此，容器创建暴露完全相同的公共接口作为一个对象UserPreferences类（理想的对象是一个 UserPreferences实例），其能够获取真实 UserPreferences从范围机制（HTTP请求，对象Session等）。容器注入此代理对象为userManager豆，这是不知道这UserPreferences参考是一个代理。在这个例子中，当一个UserManager 实例调用的依赖性注入上的方法UserPreferences的对象，但它实际上正在调用的代理的方法。代理然后取真正 UserPreferences从对象（在这种情况下）的HTTP Session，和代表所述方法调用到所检索的真实UserPreferences对象。

因此，你注射时需要以下，准确，完整，配置 request-和session-scoped豆到协作对象：
```xml


<bean id="userPreferences" class="com.foo.UserPreferences" scope="session">
    <aop:scoped-proxy/>
</bean>

<bean id="userManager" class="com.foo.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>


```
选择代理的类型来创建

默认情况下，当Spring容器创建是一个被标记用一个bean的代理<aop:scoped-proxy/>元素，创建一个基于CGLIB的类代理。


CGLIB代理仅仅拦截public方法的调用！不要叫这样的代理非公共方法; 他们不会被委派到实际范围的目标对象。


可替代地，可以配置Spring容器创造这样作用域bean标准JDK基于接口的代理，通过指定false为的值proxy-target-class的属性<aop:scoped-proxy/>元素。使用JDK基于接口的代理意味着你不需要在你的应用程序的类路径附加库来实现这种代理。然而，这也意味着该类的作用域bean必须实现至少一个接口，并且，所有到其中的范围的bean注入合作者必须通过其接口中的一个引用豆。
```xml


<!-- DefaultUserPreferences implements the UserPreferences interface -->
<bean id="userPreferences" class="com.foo.DefaultUserPreferences" scope="session">
    <aop:scoped-proxy proxy-target-class="false"/>
</bean>

<bean id="userManager" class="com.foo.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>


```
有关选择基于类还是基于接口的代理的更多详细信息，请参阅代理机制
#### 1.5.5. 定制范围


这个bean作用域机制是可以扩展的; 您可以定义自己的作用域，甚至重新定义现有的作用域，尽管后者被认为是不好的做法，你不能覆盖内置singleton和prototype范围。
创建自定义范围

为了整合您的自定义范围（S）到Spring容器中，需要实现的 org.springframework.beans.factory.config.Scope接口，这在本节中描述。对于如何实现自己的作用域的想法，看到Scope 了与Spring框架本身以及所提供实现 Scope的javadoc，这说明你需要更详细地实现的方法。

该Scope接口有四种方法来从范围获取的对象，从范围中删除，并允许将其销毁。

以下方法返回从底层范围的对象。会话范围实现中，例如，返回会话作用域的bean（如果它不存在，则该方法返回的bean的一个新实例，在已经它绑定到会话以供将来参考）。
```java
Object get(String name, ObjectFactory objectFactory)
```


下面的方法除去从底层范围的对象。例如会话范围实施，除去从底层会话的会话作用域的bean。对象应返回，但如果用指定名称的对象未找到，你可以返回null。
```java
Object remove(String name)
```


以下方法注册时，它被破坏，或者当在范围指定的对象被销毁的范围应该执行回调。参考JavaDocs或析构回调的更多信息Spring的范围实施。
```java
void registerDestructionCallback(String name, Runnable destructionCallback)
```


下述方法获得用于底层的范围会话标识符。该标识符是针对每个范围不同。对于会话范围的实施中，该标识符可以是会话标识符。
```java
String getConversationId()
```
使用自定义范围

你编写和测试一个或多个自定义后Scope实现，你需要让Spring容器知道你的新范围（或多个）。下面的方法是注册一个新的中央方法Scope与Spring容器：
```java
void registerScope(String scopeName, Scope scope);
```


此方法声明的上ConfigurableBeanFactory界面，这是适用于大部分混凝土ApplicationContext是通过BeanFactory的特性与Spring船舶实现。

第一个参数的registerScope(..)方法是用范围相关联的唯一名称; 本身在Spring容器此类名称的例子是singleton和 prototype。第二个参数的registerScope(..)方法是自定义的一个实际的例子Scope，你要注册和使用的实现。

假设你编写自定义Scope的实现，然后如下注册。



下面使用的例子SimpleThreadScope，其包含在春天，但默认情况下注册。该指令将是自己的自定义相同的Scope 实现。

```java
Scope threadScope = new SimpleThreadScope();
beanFactory.registerScope("thread", threadScope);
```


然后创建坚持您的自定义的范围规则bean定义Scope：

```xml
<bean id="..." class="..." scope="thread">
```


通过自定义Scope实现，你不局限于范围的编程注册。你也可以做的Scope登记声明，使用 CustomScopeConfigurer类：
```xml


<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
        <property name="scopes">
            <map>
                <entry key="thread">
                    <bean class="org.springframework.context.support.SimpleThreadScope"/>
                </entry>
            </map>
        </property>
    </bean>

    <bean id="bar" class="x.y.Bar" scope="thread">
        <property name="name" value="Rick"/>
        <aop:scoped-proxy/>
    </bean>

    <bean id="foo" class="x.y.Foo">
        <property name="bar" ref="bar"/>
    </bean>

</beans>


```


当您将<aop:scoped-proxy/>在FactoryBean实施，这是工厂bean本身的作用范围，而不是对象从返回getObject()。



### 1.6. 自定义bean的本质
#### 1.6.1. 生命周期回调


为了与bean的生命周期的容器的管理相结合，可以实现在春季InitializingBean和DisposableBean接口。容器调用 afterPropertiesSet()了前者destroy()为后者允许bean在bean的初始化和销毁某些行动。
```


该JSR-250 @PostConstruct和@PreDestroy注解通常被认为在现代Spring应用程序接收生命周期回调的最佳实践。使用这些注解意味着你的豆子没有耦合到Spring特定接口。有关详情请参阅@PostConstruct和@PreDestroy。

如果你不希望使用JSR-250注解，但你仍在寻找卸下联接考虑使用初始化方法和销毁法对象定义元数据。

```


在内部，Spring框架使用BeanPostProcessor实现来处理任何回调接口，它可以找到并调用相应的方法。如果您需要自定义特性或者生命周期行为不提供外的开箱，你可以实现BeanPostProcessor你自己。欲了解更多信息，请参阅 容器扩展点。

除了初始化和销毁回调，Spring管理对象还可以实现Lifecycle接口，以便通过容器的自己的生命周期驱动这些对象可以参与的启动和关闭的过程。

生命周期回调接口在本节描述。
初始化回调

该org.springframework.beans.factory.InitializingBean接口允许后对bean的所有必要属性容器设置一个bean来执行初始化的工作。的InitializingBean接口规定了一个方法：
```java
void afterPropertiesSet() throws Exception;
```


建议您不要使用该InitializingBean接口，因为这样会将夫妻代码春天。或者，使用@PostConstruct注释或指定POJO初始化方法。在基于XML的配置元数据的情况下，你可以使用init-method属性来指定具有孔隙无参数签名的方法的名称。随着Java的配置，您使用initMethod的属性@Bean，看到接收生命周期回调。例如，以下内容：
```xml


<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>


```
```java?linenums
public class ExampleBean {

    public void init() {
        // do some initialization work
    }
}
```
...是完全一样...
```xml


<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>


```
```java
public class AnotherExampleBean implements InitializingBean {

    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```
但不把代码耦合到Spring。
```

的destroy-method一个的属性<bean>元件可以被分配一个专用 (inferred)其指示Spring来自动地检测一个公共值close或 shutdown方法的特定bean类（实现任何类 java.lang.AutoCloseable或java.io.Closeable因此将匹配）。这种特殊的 (inferred)价值，也可以在设置default-destroy-method一个属性 <beans>元素，这种行为是针对整个集豆（见的 默认初始化和销毁方法）。请注意，这是与Java配置的默认行为。
```
缺省初始化和销毁方法

当你写的初始化和销毁不使用Spring的具体方法回调InitializingBean和DisposableBean回调接口，你通常写有名字，如方法init()，initialize()，dispose()，等等。理想情况下，这样的生命周期回调方法的名称在一个项目范围内标准化，使所有开发人员使用同样的方法名称，并确保一致性。

您可以配置Spring容器look名为初始化和销毁回调方法名每豆。这意味着，你作为一个应用程序开发人员，可以写你的应用程序类和使用一种称为初始化回调 init()，而无需配置的init-method="init"每个bean定义属性。Spring IoC容器调用当豆被创建（并根据先前描述的标准生命周期回调合同），该方法。此功能还强制执行初始化一致的命名约定和destroy方法回调。

假设你的初始化回调方法被命名init()和销毁回调方法被命名destroy()。您的类将类似于类在下面的例子。
```java?linenums
public class DefaultBlogService implements BlogService {

    private BlogDao blogDao;

    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }

    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }
}
```
```xml


<beans default-init-method="init">

    <bean id="blogService" class="com.foo.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>


```


所述的存在default-init-method于顶层属性<beans/>元素属性促使Spring IoC容器来识别叫做方法init上豆类作为初始化方法回调。当创建一个豆和组装，如果bean类具有这样的方法，它是在适当的时候被调用。

您配置使用destroy方法回调同样（在XML中，这是） default-destroy-method顶级的属性<beans/>元素。

如果现有的bean类已经有在与惯例有差异命名回调方法，你可以通过指定覆盖默认（XML格式，这是）使用方法名init-method和destroy-method该属性<bean/> 本身。

Spring容器保证一个bean与所有依赖供应后配置的初始化回调立即调用。因此，初始化回调被称为在生豆参考，这意味着AOP拦截器等还没有应用到豆。靶bean被完全创建第一， 然后 AOP代理（举例来说）与它的拦截器链被应用。如果目标bean和代理是分开定义了，你的代码甚至可以与原目标Bean交互，绕过代理。因此，这将是不一致的拦截器应用到init方法，因为这样做会夫妇目标bean和它的代理/拦截器的生命周期，并留下奇怪的语义，当你的代码直接交互，以原始的目标bean。
结合生命周期机制

在Spring 2.5中，你有控制bean的生命周期行为的三个选项：在 InitializingBean和 DisposableBean回调接口; 自定义 init()和destroy()方法; 和 @PostConstruct和@PreDestroy 注释。您可以结合这些机制来控制定bean。
```xml


如果多个生命周期机制被配置用于一个bean，并且每个机构被配置成与不同的方法的名称，则每个配置方法在下面列出的顺序执行。然而，如果相同的方法名称配置-例如， init()一个初始化方法-对于这些生命周期机制多于一个时，执行该方法一次，如前一节中说明。

```


配置为相同的豆，用不同的初始化方法多的生命周期机制，被称为如下：

- 方法标注了 @PostConstruct
- afterPropertiesSet()作为由定义的InitializingBean回调接口
- 自定义配置init()方法

破坏方法被调用以相同的顺序：

- 方法标注了 @PreDestroy
- destroy()作为由定义的DisposableBean回调接口
- 自定义配置destroy()方法

启动和关闭回调

该Lifecycle接口定义有自己的生命周期要求（如启动和停止一些后台进程）的任何对象的基本方法：
```java
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```


任何Spring管理对象可能实现该接口。然后，当 ApplicationContext在运行时本身接收的停止/恢复方案启动和停止的信号，例如，它会级联到所有这些调用Lifecycle该上下文中定义的实现。它通过委托给LifecycleProcessor：
```java
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}
```


请注意，LifecycleProcessor是本身的扩展Lifecycle 接口。它还增加了其他两种方法反应的背景下被刷新和关闭。
```


需要注意的是常规org.springframework.context.Lifecycle接口仅仅是明确的开始一个普通的合同/停止的通知，并不意味着在上下文刷新时间自动启动。考虑实施org.springframework.context.SmartLifecycle，而不是用于在特定的bean（包括启动阶段）的自动启动细粒度控制。另外，请注意，停止的通知，不能保证毁灭之前来：在定时关机，所有的Lifecycle豆类将首先一般销毁回调被传播之前收到停止通知; 然而，在上下文的生存期间或中止刷新尝试热刷新，只会破坏方法将被调用。

```


启动和关闭调用的顺序也很重要。如果一个“依赖式”任意两个对象之间存在关系，依赖方将开始后的依赖，并且它将停止之前的依赖。然而，有时直接依赖关系是未知的。你可能只知道某一类型的对象应该先于其它类型的对象开始。在这些情况下，SmartLifecycle接口定义的另一种选择，即getPhase()它的超接口上定义的方法， Phased。
```java

public interface Phased {

    int getPhase();
}
```
```java
public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);
}
```


在启动时，具有最低相的对象第一次启动和停止时，以相反的顺序被执行。因此，实现了一个对象SmartLifecycle，其getPhase()方法返回Integer.MIN_VALUE将率先启动，最后停之中。在光谱的另一端，的相位值 Integer.MAX_VALUE将表明该对象应最后启动和停止第一（可能是因为它依赖于其他进程中运行）。当考虑相位值，它也是重要的是知道，对于任何“正常”的默认阶段 Lifecycle目标没有实现SmartLifecycle将是0。因此，任何负面相位值将指示对象应这些标准组件开始前（后停止它们），并且对于任何正相位值，反之亦然。

正如你所看到的所定义的停止方法SmartLifecycle接受一个回调。任何实现必须调用该回调的run()后实施的关机过程完成的方法。这使异步停机，必要的，因为默认实现LifecycleProcessor接口 DefaultLifecycleProcessor，将等待其超时值的组对象的每个阶段中调用该回调。默认每相超时时间为30秒。您可以通过定义范围内名为“lifecycleProcessor”豆覆盖默认生命周期处理器实例。如果你只是想修改超时，然后定义以下就足够了：
```xml


<bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
    <!-- timeout value in milliseconds -->
    <property name="timeoutPerShutdownPhase" value="10000"/>
</bean>


```


如提到的，LifecycleProcessor接口定义的上下文的清爽和关闭回调方法为好。后者将简单地推动关机过程中仿佛stop()已经明确要求，但是当上下文被关闭它会发生。在另一方面，“刷新”回调使的另一特点 SmartLifecycle豆。当所述上下文被刷新（毕竟对象已被实例化并初始化），该回调将被调用，并且在该点的默认的生命周期处理器将检查由每个返回的布尔值 SmartLifecycle对象的isAutoStartup()方法。如果“真”，那么那个对象将在该点开始，而不是等待上下文的还是其自身的显式调用start()方法（不像背景刷新，上下文开始不会自动对标准实施背景下发生）。“相位”的值，以及任何“取决于接通”关系将确定以相同的方式启动顺序，如上所述。
在非web应用中优雅地关闭Spring IoC容器
```


本节仅适用于非Web应用程序。Spring的基于网络的 ApplicationContext实现已经到位时，相关的Web应用程序被关闭正常关闭Spring IoC容器代码。

```


如果您使用在非Web应用程序环境中，Spring的IoC容器; 例如，在一个富客户端桌面环境; 你注册到JVM关闭挂钩。这样做可以确保正常关机，并调用相关的破坏singleton bean上的方法，使所有资源被释放。当然，你还必须配置和实施这些正确销毁回调。

要注册一个关闭挂钩，您拨打的registerShutdownHook()是在上声明的方法ConfigurableApplicationContext接口：
```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();

        // app runs here...

        // main method exits, hook is called prior to the app shutting down...
    }
}
```
#### 1.6.2. ApplicationContextAware和BeanNameAware


当ApplicationContext创建了实现所述对象实例 org.springframework.context.ApplicationContextAware接口，该实例被提供有该基准ApplicationContext。
```java


public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}

```


因此豆可以在编程方式操作ApplicationContext创建它们，通过ApplicationContext接口，或通过铸造参照该接口的已知子类，如ConfigurableApplicationContext，它公开的附加功能。其中一个用途是其他豆类的纲领性检索。有时候，这种能力是非常有用的; 然而，一般来说，你应该避免，因为它情侣代码春不遵循控制的风格，合作者提供给豆类特性反演。在其他的方法 ApplicationContext提供访问文件资源，发布应用程序事件，和访问MessageSource。这些附加特征中描述 的ApplicationContext中其他功能

在Spring 2.5中，自动装配是另一种选择，以获得与参考 ApplicationContext。“传统的” constructor和byType自动装配模式（在描述的自动装配合作者）可以提供类型的依赖 ApplicationContext于构造器参数或设置器方法参数，分别。欲了解更多灵活性，包括自动装配属性和多参数方法的能力，使用新的基于注解的自动连接功能。如果这样做，将ApplicationContext被自动连接到所期待的字段，构造函数的参数，或者方法参数ApplicationContext类型，如果该域，构造或问题的方法进行了@Autowired注释。欲了解更多信息，请参见 @Autowired。

当ApplicationContext创建了实现的一类 org.springframework.beans.factory.BeanNameAware接口，类设置有在其相关联的对象定义所定义的名称的引用。
```java
public interface BeanNameAware {

    void setBeanName(String name) throws BeansException;
}
```


回调正常bean属性的人口之后但在初始化回调如之前调用InitializingBean 的afterPropertiesSet或自定义的初始化方法。
#### 1.6.4. 其他感知接口


除了ApplicationContextAware和BeanNameAware上面所讨论的，Spring提供了一系列的Aware，允许豆指示，他们需要一定的容器接口基础设施的依赖。最重要的Aware接口总结如下-作为一般规则，该名称是依赖型的一个很好的迹象：
|Name|依赖注入|解释..|
|-|-|-|
|ApplicationContextAware|声明 ApplicationContext|ApplicationContextAware和BeanNameAware|
|ApplicationEventPublisherAware|封闭的事件发布 ApplicationContext| ApplicationContext中的其他功能|
|BeanClassLoaderAware|类加载器用于加载bean类。|实例化bean|
|BeanFactoryAware|声明 BeanFactory|ApplicationContextAware和BeanNameAware|
|BeanNameAware|申报bean的名称|ApplicationContextAware和BeanNameAware|
|BootstrapContextAware|资源适配器BootstrapContext的容器中，仅在JCA知道运行。通常可用ApplicationContext小号|CCI制定|
|LoadTimeWeaverAware|定义韦弗在加载时处理类定义|加载时在Spring应用中使用AspectJ织|
|MessageSourceAware|解决消息（与参数化和国际化的支持）配置的策略|ApplicationContext中的其他功能|
|NotificationPublisherAware|春天JMX通知出版商|通知|
|ResourceLoaderAware|对于低级别的访问到资源配置装载机|资源|
|ServletConfigAware|当前ServletConfig容器中仅在基于web的Spring运行。有效 ApplicationContext|Spring MVC|
|ServletContextAware|当前ServletContext容器中仅在基于web的Spring运行。有效 ApplicationContext|Spring MVC|



再次注意，这些接口是使用您的代码与Spring的API，并且不遵循转控制的原则。因此，他们建议适用于要求容器编程访问的基础设施豆类。

### 1.7. Bean定义继承


一个bean定义可以包含大量的配置信息，包括构造器参数，属性值，以及容器的特定信息，如初始化方法，静态工厂方法名等。子bean定义从父定义继承配置数据。儿童的定义可以覆盖一些值，或者添加一些它需要的。使用父子bean定义可以节省很多的输入工作。实际上，这是模板的一种形式。

如果你有工作ApplicationContext接口编程，子bean定义所代表ChildBeanDefinition的类。大多数用户不跟他们在这个层面上的工作，而不是配置在声明类似的bean定义ClassPathXmlApplicationContext。当您使用基于XML的配置元数据，您可以指示使用一个子bean定义parent属性，指定父bean作为这个属性的值。
```xml


<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>


```


子bean定义使用bean类从父定义，如果没有指定，但也可以覆盖它。在后一种情况下，孩子bean类必须与父母兼容的，也就是说，它必须接受父的属性值。

子bean定义继承范围，构造函数参数值，属性值和方法覆盖从父，以增加新的价值的选项。任何范围，初始化方法，销毁方法，和/或static您指定将覆盖相应的父设置工厂方法设置。

其余的设置总是从子定义处得到：依赖， 自动装配模式，依赖检查，单，延迟初始化。

前面的例子中，通过使用显式地将父bean定义为抽象abstract属性。如果父定义并没有指定一个类，明确标记为父bean定义abstract是必需的，如下：
```xml


<bean id="inheritedTestBeanWithoutClass" abstract="true">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithClass" class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBeanWithoutClass" init-method="initialize">
    <property name="name" value="override"/>
    <!-- age will inherit the value of 1 from the parent bean definition-->
</bean>


```


父豆不能对自己进行实例化，因为它是不完整的，并且它也明确标记为abstract。当一个定义是abstract这样的，它仅可使用作为一个纯粹的模板bean定义，充当子定义的父定义。尝试使用这样的abstract自身父bean，参照它作为其他bean的ref属性或做一个明确的getBean()与父bean的id调用，返回一个错误。同样地，容器的内部 preInstantiateSingletons()方法忽略被定义为抽象bean定义。
```


ApplicationContext默认情况下，预实例化所有singleton。因此，重要的是（至少对于单豆），如果你有，你打算只使用一个模板（父）bean定义，这个定义说明了一个类，你必须确保所设置的抽象属性为真，否则应用上下文将会（试着）预实例abstract豆。

```

### 1.8. 集装箱扩建点


通常，应用程序开发者不需要子类ApplicationContext 实现类。相反，Spring IoC容器可以通过特殊的集成接口实现扩展。接下来的几节将介绍这些集成接口。

#### 1.8.1. 使用了BeanPostProcessor定制bean


该BeanPostProcessor接口定义回调方法，你可以实现提供自己的（或覆盖容器的默认值）的实例化逻辑，依赖解析逻辑，等等。如果你想实现一些自定义逻辑Spring容器完成实例化，配置和初始化后，豆，你可以在一个或多个插件BeanPostProcessor实现。

您可以配置多个BeanPostProcessor实例，你可以控制这些顺序BeanPostProcessorS按设定的执行order性能。您可以设置该属性只有在BeanPostProcessor实现了Ordered接口; 如果你写你自己的BeanPostProcessor，你应该考虑实施Ordered 过界面。欲知详情，请咨询的的javadoc BeanPostProcessor和 Ordered接口。另请参阅下面的注释 的纲领性注册BeanPostProcessor小号。
```


BeanPostProcessorS于豆（或对象）操作实例 ; 也就是说，Spring IoC容器实例化一个bean实例，然后 BeanPostProcessor就做他们的工作。

BeanPostProcessors的范围的每个容器。如果您所使用的容器，这只是相关的。如果定义BeanPostProcessor在一个容器，它会只处理后的该容器中的咖啡豆。换句话说，在一个容器中定义的豆不是后处理通过BeanPostProcessor在另一容器中所定义，即使这两个容器是相同的分层结构的一部分。

要改变实际的bean定义（即蓝图定义豆），而不是你需要使用BeanFactoryPostProcessor在描述 定制配置元数据与BeanFactoryPostProcessor的。

```


该org.springframework.beans.factory.config.BeanPostProcessor接口包括正好两个回调方法。当这样一类被注册为后置处理器的容器中，由容器所创建的每个bean实例中，后处理器会从容器中的回调都之前容器初始化方法（如的InitializingBean的afterPropertiesSet方法（）和任何声明init方法）被称为以及之后的任何豆初始化回调。后处理器可以利用这个bean实例的任何行动，包括完全忽略此回调。一个bean后处理器通常检查回调接口，或者可以与一个代理包裹的bean。一些Spring AOP的基础设施类，以提供代理包装逻辑实现bean后置处理器。

的ApplicationContext 自动检测，其中实施所述配置元数据中定义的任何豆BeanPostProcessor接口。该 ApplicationContext寄存器这些豆子为后置处理器，使他们能够在以后bean创建调用。bean后置处理器可以在容器中，就像任何其他豆类部署。

需要注意的是声明当BeanPostProcessor使用@Bean工厂方法上的配置类，工厂方法的返回类型应该是实现类本身或至少org.springframework.beans.factory.config.BeanPostProcessor 界面，这清楚地表明bean的后处理器性质。否则， ApplicationContext将无法按类型完全创建之前将自动检测到它。由于BeanPostProcessor需要进行早期实例，以适用于上下文中的其他bean初始化，这种早期类型的检测是至关重要的。
```
编程注册BeanPostProcessor的

而对于建议的方法BeanPostProcessor注册是通过 ApplicationContext自动检测（如上文所述），但也可以注册它们编程对一个ConfigurableBeanFactory使用 addBeanPostProcessor方法。需要在登记前，甚至用于复制bean后置处理器跨情境的层次评估条件逻辑时，这可能是有用的。然而要注意BeanPostProcessorš编程方式添加不尊重的Ordered接口。这是登记的顺序即决定执行顺序。还要注意BeanPostProcessor的注册程序是那些通过自动检测登记之前经常被加工，无论任何明确的排序。

```
```
BeanPostProcessor的和AOP自动代理

实现类BeanPostProcessor接口是特殊的，并且由容器特别对待。所有BeanPostProcessor小号和豆类，他们直接引用的实例在启动时，作为的特殊启动阶段的一部分 ApplicationContext。接下来，所有的BeanPostProcessors的注册入一个列表并应用于容器中的所有的bean。因为AOP自动代理为实现BeanPostProcessor自身，无论BeanPostProcessor小号也不是他们直接引用豆有资格自动代理，因而没有织成他们的方面。

对这些bean，你会看到一条信息日志消息：“ 豆富不符合所有的BeanPostProcessor接口（例如：不符合自动代理）得到处理 ”。

请注意，如果您有豆连线到您BeanPostProcessor使用自动装配或 @Resource（可能回落到自动装配），类型匹配的依赖候选人搜索时，弹簧会访问意外豆，因此使他们没有资格自动代理或其他类型的bean后的-处理。例如，如果你有注解的依赖@Resource，其中场/ setter方法名称并不直接对应于一个bean并没有name属性的声明的名称使用，那么Spring将访问其他豆类按类型匹配他们。

```


下面的示例显示了如何编写，注册并使用BeanPostProcessor在一个s ApplicationContext。
例如：世界您好，BeanPostProcessor的风格

这个第一实施例说明的基本用法。该示例示出了定制的 BeanPostProcessor调用执行toString()每个bean的方法，因为它是由容器中创建并打印结果字符串到系统控制台。

找到自定义以下BeanPostProcessor实现类的定义：
```java?linenums
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

    // simply return the instantiated bean as-is
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        return bean; // we could potentially return any object reference here...
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("Bean '" + beanName + "' created : " + bean.toString());
        return bean;
    }
}
```
```xml


<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:lang="http://www.springframework.org/schema/lang"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/lang
        http://www.springframework.org/schema/lang/spring-lang.xsd">

    <lang:groovy id="messenger"
            script-source="classpath:org/springframework/scripting/groovy/Messenger.groovy">
        <lang:property name="message" value="Fiona Apple Is Just So Dreamy."/>
    </lang:groovy>

    <!--
    when the above bean (messenger) is instantiated, this custom
    BeanPostProcessor implementation will output the fact to the system console
    -->
    <bean class="scripting.InstantiationTracingBeanPostProcessor"/>

</beans>


```


请注意如何InstantiationTracingBeanPostProcessor被简单地定义。它甚至没有一个名字，因为它是一个bean也可以是依赖注入就像任何其他的bean。（前面的配置也定义了一个由Groovy脚本支持的bean。春季动态语言支持在章节标题详述 动态语言支持）。

下面简单的Java应用程序执行上述代码和配置：
```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.scripting.Messenger;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("scripting/beans.xml");
        Messenger messenger = (Messenger) ctx.getBean("messenger");
        System.out.println(messenger);
    }

}
```
前面的应用程序的输出如下所示：
```


豆“信使”创建：org.springframework.scripting.groovy.GroovyMessenger@272961 
org.springframework.scripting.groovy.GroovyMessenger@272961


```
RequiredAnnotationBeanPostProcessor示例

在使用自定义结合使用回调接口或注解 BeanPostProcessor实施是扩展Spring IoC容器的常用手段。一个例子是Spring RequiredAnnotationBeanPostProcessor-一个 BeanPostProcessor实现附带这确保了对标有（任意的）注释豆JavaBean属性实际上（构造为）依赖注射用一个值Spring发行。
#### 1.8.2. 定制配置元数据用的BeanFactoryPostProcessor


接下来的扩展点，我们将看看是 org.springframework.beans.factory.config.BeanFactoryPostProcessor。此接口的语义是类似于那些的BeanPostProcessor，有一个主要的区别在于：BeanFactoryPostProcessor运行在豆配置元数据 ; 也就是说，Spring IoC容器允许BeanFactoryPostProcessor读取配置元数据，并可能修改它之前的容器实例之外的任何豆BeanFactoryPostProcessor秒。

您可以配置多BeanFactoryPostProcessor秒，就可以控制这些顺序BeanFactoryPostProcessorS按设定的执行order性能。但是，你才可以设置该属性，如果BeanFactoryPostProcessor实现了 Ordered接口。如果你写你自己的BeanFactoryPostProcessor，你应该考虑实施Ordered过界面。咨询的的javadoc BeanFactoryPostProcessor和Ordered接口的更多细节。
```


如果你想改变实际的bean 实例（即，从配置元数据创建的对象），那么你反而需要使用BeanPostProcessor （在上述定制使用的BeanPostProcessor豆）。虽然在技术上是可行内与bean实例的工作BeanFactoryPostProcessor（例如，使用 BeanFactory.getBean()），这样做会导致过早豆实例，违反标准集装箱的生命周期。这可能会导致消极的副作用，如绕过豆后处理。

此外，BeanFactoryPostProcessors的作用域每个容器。如果您所使用的容器，这只是相关的。如果定义BeanFactoryPostProcessor在一个容器中，将只适用于该容器中的bean定义。在一个容器中的Bean定义不会进行后处理由BeanFactoryPostProcessorS IN另一个容器，即使这两个容器都处在同一层次的一部分。

```


当它内部的声明的bean工厂后处理器自动执行 ApplicationContext，以更改应用到限定所述容器的结构的元数据。弹簧包括许多预定义的bean工厂后置处理器，比如PropertyOverrideConfigurer和 PropertyPlaceholderConfigurer。自定义BeanFactoryPostProcessor也可使用，例如，以注册自定义属性编辑器。

一个ApplicationContext自动检测部署在它实现了任何豆BeanFactoryPostProcessor接口。它用这些豆子bean工厂后置处理器，在适当的时候。你可以象其它bean部署这些处理器后豆。
```


至于BeanPostProcessorS，你通常不希望配置 BeanFactoryPostProcessor为延迟初始化秒。如果没有其他bean的引用 Bean(Factory)PostProcessor，即后处理器将不会在所有的实例。因此，将其标记为延迟初始化将被忽略， Bean(Factory)PostProcessor会急切地实例化，即使你设定的 default-lazy-init属性true对你的声明<beans />元素。

```
例如：类名替代PropertyPlaceholderConfigurer

您可以使用PropertyPlaceholderConfigurer从使用标准Java在一个单独的文件中的bean定义一些属性值Properties的格式。这样做使应用程序部署到自定义具体环境的属性，如数据库URL和密码的人，没有复杂性或修改主XML定义文件或文件容器的风险。

考虑以下基于XML的配置元数据片段，其中DataSource 与占位符值被定义。该示例示出了从外部配置的属性Properties文件。在运行时，PropertyPlaceholderConfigurer施加到将替换dataSource的一些性质的元数据。的值，以取代被指定为占位符形式的${property-name}随后的蚂蚁/ log4j的/ JSP EL风格。
```xml


<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations" value="classpath:com/foo/jdbc.properties"/>
</bean>

<bean id="dataSource" destroy-method="close"
        class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>


```


实际值来自于标准的Java另一种文件Properties格式：
```


jdbc.driverClassName = org.hsqldb.jdbcDriver 
jdbc.url = JDBC：HSQLDB：HSQL：//生产：9002 
jdbc.username = SA 
jdbc.password =根


```


因此，字符串${jdbc.username}替换在与值“SA”运行时，并且同样适用于匹配在属性文件密钥，其他占位符值。在PropertyPlaceholderConfigurer为大多数属性和bean定义的属性占位符检查。此外，占位符前缀和后缀可以定制。

随着context在Spring 2.5中引入的命名空间，也可以用专用的配置元素配置属性的占位符。一个或多个位置可以如逗号分隔的列表中提供location属性。
```xml


<context:property-placeholder location="classpath:com/foo/jdbc.properties"/>


```


在PropertyPlaceholderConfigurer不仅将查找在属性Properties 指定的文件。默认情况下，它还会检查在Java的System性能，如果它不能找到指定的属性文件的属性。您可以通过设置自定义此行为systemPropertiesMode有以下三种支持整数值之一的配置者的财产：

- 从来没有（0）：从不检查系统属性
- 回退（1）：如果在指定的属性文件中不能解决的检查系统属性。这是默认的。
- 覆盖（2）：首先检查系统属性，试图指定的属性文件之前。这允许系统属性来覆盖任何其他属性源。

咨询PropertyPlaceholderConfigurer了解更多信息的javadoc。
```xml


您可以使用PropertyPlaceholderConfigurer替代类名，有时它是有用时，你必须选择一个特定的实现类在运行时。例如：

<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations">
        <value>classpath:com/foo/strategy.properties</value>
    </property>
    <property name="properties">
        <value>custom.strategy.class=com.foo.DefaultStrategy</value>
    </property>
</bean>

<bean id="serviceStrategy" class="${custom.strategy.class}"/>

如果类不能在运行时被解析为一个有效的类，bean的分辨率，当它即将被创造，这是在失败preInstantiateSingletons() 的阶段ApplicationContext对非延迟实例化的bean。

```
例：PropertyOverrideConfigurer

在PropertyOverrideConfigurer另一个bean工厂后置处理器，类似 PropertyPlaceholderConfigurer，但不同的是后者，原来的定义可以有缺省值或者根本没有值的bean属性。如果一个压倒一切的 Properties文件没有某个bean属性的内容，则使用缺省的上下文定义。

需要注意的是bean定义为 没有意识到被覆盖，所以它不是从正在使用覆盖配置XML定义文件立即明显。在多的情况下，PropertyOverrideConfigurer对于相同的bean属性，最后一胜，由于覆盖机制定义不同值的实例。

Properties文件的配置采用这种格式：
```
beanName.property =值
```
例如:
```


dataSource.driverClassName = com.mysql.jdbc.Driver 
dataSource.url = JDBC：MySQL的的的：MYDB


```


这个例子文件可以包含一个名为豆的容器定义中使用 数据源，其中有驱动程序和URL的属性。

组合的属性名称也支持，只要被覆盖的路径除了最后一个属性的每个组件已非空（比如通过构造器初始化）。在这个例子中...
```
foo.fred.bob.sammy = 123
```


    在sammy该财产bob的财产fred的财产foobean被设置为标量值123。

```


指定覆盖值始终是文字值; 他们没有转化成Bean引用。当XML bean定义的原始值指定bean引用该约定也适用。
```


随着context在Spring 2.5中引入的命名空间，可以配置属性压倒一切有专用的配置元素：
```xml
<context:property-override location="classpath:override.properties"/>
```
#### 1.8.3. 自定义实例化逻辑与一个FactoryBean


落实org.springframework.beans.factory.FactoryBean为对象的接口 有自己的工厂。

该FactoryBean接口是一个点的可插入到Spring IoC容器的实例化逻辑。如果您有在Java中更好的表现，而不是XML的（可能）冗长的复杂的初始化代码，你可以创建自己的 FactoryBean，那个类中写入复杂的初始化，然后把你定制的FactoryBean容器中。

该FactoryBean接口提供了三种方法：

    Object getObject()：返回该工厂创建的对象的实例。这个实例可能依赖于该工厂是否返回singleton或原型被共享。

    boolean isSingleton()：返回true如果FactoryBean返回单身， false否则。

    Class getObjectType()：返回由返回的对象类型getObject()的方法或null，如果该类型不是预先已知的。

该FactoryBean概念和接口被一些Spring框架内的地方使用; 在超过50个实现FactoryBean接口船，Spring本身。

当你需要问一个容器实际FactoryBean实例本身，而不是它创建的bean，前言bean的ID和连字符（&调用时）getBean()的方法ApplicationContext。因此，对于一个给定的FactoryBean 用的ID myBean，调用getBean("myBean")在容器上返回的产品FactoryBean; 然而，调用getBean("&myBean")返回的 FactoryBean实例本身。

### 1.9. 基于注释的容器配置
```

注释是比XML的配置Spring更好？

引入基于注解的配置提出的这种做法是否比XML“更好”的问题。简短的答案是它依赖。长的答复是，每种方法都有其优点和缺点，通常它是由开发者决定哪种策略适合他们更好。由于他们的方式定义，注释在他们的声明中提供了很多方面，导致更短，更简洁的配置。然而，XML擅长连接最多部件而不触及他们的源代码或重新编译他们。有些人希望具有靠近源布线而其他人则认为注释类不再POJO和，此外，该结构变得分散，难以控制。

无论选择，Spring可以容纳两种风格，甚至将它们混合在一起。值得指出的是，通过其JavaConfig选项，Spring允许注释以非侵入性的方式使用，而不触及目标组件的源代码，并在模具方面，所有的配置风格被支持的 春天工具套件。
```


到XML设置的替代是通过基于注释的配置依赖于字节码元数据布线组件组成，而不是尖括号声明提供。代替使用XML来描述一个bean布线的，显影剂通过使用在相关类，方法或域声明注解移动配置到部件类本身。正如所提到RequiredAnnotationBeanPostProcessor示例，使用BeanPostProcessor结合注解是扩展Spring IoC容器的常用手段。例如，Spring 2.0引入执行与所要求的性能的可能性@Required注释。Spring 2.5的使人们有可能跟随驱动Spring的依赖注入相同的一般方法。从本质上讲，@Autowired注释提供了如所描述的相同的功能，自动装配协作者但具有更精细的控制和更广泛的适用性。弹簧2.5还增加了对JSR-250注解如 @PostConstruct，和@PreDestroy。弹簧3.0添加JSR-330（Java依赖注入）包含在所述包javax.inject注释如支持@Inject 和@Named。关于那些标注详细信息可在中找到 相关的部分。
```
注释注射执行之前 XML喷射，从而后者配置将覆盖前者用于通过这两种方法的有线特性。

```


与往常一样，你可以作为一个独立的bean定义他们注册，但是他们也可以通过包括一个基于XML的配置方式，下面的标记（注意列入被隐性登记的context命名空间）：
```xml


<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>


```


（该隐式注册的后处理器包括 AutowiredAnnotationBeanPostProcessor， CommonAnnotationBeanPostProcessor， PersistenceAnnotationBeanPostProcessor，以及前述 RequiredAnnotationBeanPostProcessor）。
```


<context:annotation-config/>只查找在同一个应用程序上下文豆定义它的注解。这意味着，如果你把 <context:annotation-config/>一个WebApplicationContext一个DispatcherServlet，它只是检查@Autowired在控制器豆类，而不是你的服务。见 DispatcherServlet的更多信息。

```
#### 1.9.1. @Required
该@Required注释适用于bean属性setter方法，如下面的例子：
```java?linenums
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```


此注释仅表示受影响的bean属性必须在配置时间在一个bean定义或通过自动装配来填充，通过一个明确的属性值。容器抛出如果受影响的bean属性还没有被填充的除外; 这使得急于和显性故障，避免NullPointerExceptions等后面。它仍然建议你把断言到bean类本身，例如，到init方法。当您使用类容器的外面否则强制执行这些要求的参考和值偶数。
#### 1.9.2. @Autowired
```
JSR 330的@Inject注释来代替Spring的的使用@Autowired注释在下面的例子。请参阅这里了解更多详情。
```
您可以将@Autowired注释构造函数：
```java?linenums
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```
```


由于Spring框架4.3的，一个@Autowired这样的构造注释不再是必要的，如果目标bean只定义了一个构造函数开始。但是，如果几个构造函数可用，必须至少有一个被标注教一个要使用的容器。
```


正如预期的那样，你还可以将@Autowired标注为“传统的” setter方法：
```java?linenums
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```
您也可以标注适用于具有任意名称和/或多个参数的方法：
```java?linenums
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```


你可以申请@Autowired到领域以及，甚至可以与构造混合：
```java?linenums
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    private MovieCatalog movieCatalog;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```
```


确保你的目标组件（例如MovieCatalog，CustomerPreferenceDao）始终得到您正在使用您的类型声明@Autowired-annotated注入点。否则注射可能因没有类型匹配发现在运行时失败。

用于通过类路径扫描发现XML定义的豆或组件类，容器通常知道具体类型的前期。然而，对于@Bean工厂方法，你需要确保的是，声明的返回类型是足够的表现力。为了实现多个接口组件或通过其实施类型可能提及的部件，考虑你的工厂方法声明最具体的返回类型（至少需要通过注射点指的是你的bean作为特定的）。

```


另外，也可以提供所有从特定类型的豆 ApplicationContext通过添加注释到期望类型的数组的字段或方法：
```java?linenums
public class MovieRecommender {

    @Autowired
    private MovieCatalog[] movieCatalogs;

    // ...
}
```
这同样适用于类型集合：
```java?linenums
public class MovieRecommender {

    private Set<MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```
```


你的目标豆可以实现org.springframework.core.Ordered接口或使用@Order或标准@Priority，如果你想在数组或列表中的项目以特定的顺序进行排序注释。否则，它们的顺序将遵循在容器中的相应目标bean定义的登记顺序。

所述@Order注释可以在目标类水平，而且也对被声明@Bean（在的多个定义的情况下用相同的bean类）的方法，有可能为每bean定义非常个体。@Order值可以在注入点影响的优先事项，但是请注意，它们不影响单启动顺序是由依赖关系，并确定正交关注@DependsOn声明。

需要注意的是标准的javax.annotation.Priority注解是不可用的 @Bean级别，因为它不能在方法声明。其语义可以通过被建模@Order组合值与@Primary每个类型的单个豆。

```


即使输入地图可只要与期望的密钥类型是自动装配String。地图值将包含预期的类型的所有豆类和按键将包含相应的bean的名字：
```java?linenums
public class MovieRecommender {

    private Map<String, MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```


默认情况下，每当自动连接失败零种候选人豆可用; 默认行为是治疗方法，构造器，字段假设为需要依赖。如下面所示可以改变这种现象。
```java?linenums
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired(required = false)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```
```


只有一个每类注释构造函数可以作为标记要求，但多个非必需的构造函数可以被注解。在这种情况下，每次的候选人中考虑和Spring使用贪婪的构造，其依赖关系可以得到满足，那就是有参数的数目最多的构造。

将所需的属性@Autowired，建议在@Required注解。将所需的属性表示该属性不需要自动装配的目的，如果它不能自动装配的属性被忽略。@Required，在另一方面，是因为它会强制通过由容器所支持的任何装置设定的属性更强。如果没有值被注入，相应的异常。

```

另外，您也可以表达通过Java 8的一个特别依赖的非必需性质java.util.Optional：
```java?linenums
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        ...
    }
}
```


由于Spring框架5.0的，你也可以使用一个@Nullable注释（任何种类的任何包装，例如javax.annotation.Nullable从JSR-305）：
```java?linenums
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        ...
    }
}
```


您还可以使用@Autowired对于那些众所周知的解析依赖接口：BeanFactory，ApplicationContext，Environment，ResourceLoader， ApplicationEventPublisher，和MessageSource。这些接口和扩展接口，如ConfigurableApplicationContext或ResourcePatternResolver，被自动解决，而且不需要特殊的设置。
```java?linenums
public class MovieRecommender {

    @Autowired
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...
}
```
```


@Autowired，@Inject，@Resource，和@Value注释由Spring处理 BeanPostProcessor实现这反过来又意味着你不能在您自己的应用这些注释BeanPostProcessor或BeanFactoryPostProcessor类型（如果有的话）。这些类型必须为“连接好”明确通过XML或使用Spring的@Bean方法。


```
#### 1.9.3. 微调标注为基础自动装配与@Primary


因为通过自动装配型可能会导致多个候选，它通常需要具有对选择过程的更多控制。做到这一点的方法之一是使用Spring的 @Primary注解。@Primary表明特定豆应优先考虑在多个豆都被装配到一个单值依赖候选人。如果只有一个“主”豆的候选人中存在，这将是自动装配Autowired值。

假设我们有以下的配置定义firstMovieCatalog为 初级 MovieCatalog。
```java?linenums
@Configuration
public class MovieConfiguration {

    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }

    // ...
}
```


通过这样的结构，下面MovieRecommender将与被装配 firstMovieCatalog。
```java?linenums
public class MovieRecommender {

    @Autowired
    private MovieCatalog movieCatalog;

    // ...
}
```
相应的bean定义如下所示。
```xml


<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog" primary="true">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>


```
#### 1.9.4. 与预选赛微调基于注解的自动连接


@Primary是通过用型几个实例以使用自动装配时，能够确定的一个一次候选的有效途径。当需要在选择过程中更多的控制权，Spring的@Qualifier可以用来注解。您可以使用特定的参数限定词值相关联，缩小集类型匹配的，这样一个特定的bean被选择为每一个参数。在最简单的情况下，这可以是一个简单的描述性值：
```java?linenums
public class MovieRecommender {

    @Autowired
    @Qualifier("main")
    private MovieCatalog movieCatalog;

    // ...
}
```


的@Qualifier注释也可以在单独的构造器参数或方法参数指定：
```java?linenums
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(@Qualifier("main")MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```
相应的bean定义如下所示。与限定值的bean的“主”是有线与有资格使用相同的值构造函数的参数。
```xml


<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="main"/>

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="action"/>

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>


```


对于一个备用的比赛，该bean的名称被认为是默认预选赛值。因此，你可以定义一个ID为“主”，而不是嵌套预选赛元素豆，导致同样的匹配结果。不过，虽然你可以使用这种约定按名称来指代特定的豆子，@Autowired从根本上说与可选的语义预选赛型驱动的注射。这意味着限定值，甚至与bean名称回退，总是有一组类型的比赛中缩小语义; 他们没有语义表达为一个唯一的ID豆的参考。良好限定的值是“主”或“EMEA”或“持久的”，表示是独立于从所述豆的特定部件的特性id，

限定符也适用于类型集合，如上所讨论的，例如，到 Set<MovieCatalog>。在这种情况下，按申报预选赛全部匹配的bean注入作为一个集合。这意味着预选赛不必是唯一的; 他们而不是简单地构成的过滤标准。例如，您可以定义多个MovieCatalog豆具有相同限定值“行动”，所有这些都将被注入Set<MovieCatalog>与注解@Qualifier("action")。

```


让预选赛值中选择针对目标bean的名字，类型匹配的候选人中，甚至不要求@Qualifier在注入点注释。如果没有其他的分辨率指示（如限定词或主要标志），用于非唯一依赖的情况下，Spring将针对目标bean名称相匹配的注入点名称（即字段名或参数名称），然后选择各项─名为候选人，如果有的话。

这就是说，如果你打算表达的名字注解驱动的注入，不主要使用@Autowired，即使是能够通过bean名称类型匹配的候选人中选择的。相反，使用JSR-250 @Resource注释，其被定义语义通过其唯一的名称来标识一个特定的目标成分，与声明的类型是无关的匹配处理。@Autowired具有相当不同的语义：由类型选择候选豆后，将指定的字符串限定符值将只有那些类型选择的候选内被考虑，例如匹配针对标以相同的限定符标签豆一个“帐户”限定符。

对于本身被定义为一个收集/地图或阵列型豆，@Resource 是一个很好的解决方案，参照由唯一名称的特定集合或数组豆。这就是说，作为4.3，收集/地图和数组类型都可以通过Spring的匹配 @Autowired类型匹配算法为好，只要元素的类型信息被保存在@Bean返回类型签名或集合继承层次。在这种情况下，限定值可以使用相同类型的集合中进行选择，如前一段概述。

对于4.3，@Autowired也考虑用于注射的自我引用，即引用回到当前注入的豆。需要注意的是自我注射是一个备用; 对其他组件的依赖定期始终具有优先权。在这个意义上，自我引用不参加定期的候补选择，因此特别为从未小学; 相反，他们总是最终成为优先级最低。在实践中，使用自引用作为最后的手段而已，例如通过bean的事务代理调用同一实例的其他方法：考虑在这样的情况下分解出受影响的方法，以一个单独的委托豆。另外，使用@Resource它可以通过其独特的名称，来获得代理回到当前Bean。

@Autowired适用于字段，构造，和多参数的方法，允许在参数级通过限定器注解缩小。与此相反，@Resource 在与一个参数字段和bean属性setter方法只支持。因此，坚持预选赛，如果你的注入目标是构造函数或多方论证方法。

```


您可以创建自己的自定义限定器注解。简单地定义一个注释，并提供@Qualifier你的定义中的注释：
```java?linenums
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {

    String value();
}
```
然后，你可以提供自定义的限定自动连接上的字段和参数：
```java?linenums
public class MovieRecommender {

    @Autowired
    @Genre("Action")
    private MovieCatalog actionCatalog;

    private MovieCatalog comedyCatalog;

    @Autowired
    public void setComedyCatalog(@Genre("Comedy") MovieCatalog comedyCatalog) {
        this.comedyCatalog = comedyCatalog;
    }

    // ...
}
```


接下来，提供对候选bean定义的信息。您可以添加 <qualifier/>标签作为的子元素<bean/>标签，然后指定type并 value匹配您的自定义限定器注解。该类型对注释的完全限定类名相匹配。或者，仿佛存在不便利的名称冲突的风险，可以使用短类名。这两种方法表明在下面的例子。
```xml


<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="Genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="example.Genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>


```


在类路径扫描和管理的组成部分，你将看到一个基于注释的替代品在XML提供限定符元数据。具体来说，请参阅提供与注释限定符元数据。

在某些情况下，它可能是足够使用的标注没有值。当注释提供一个更通用的目的，并且可以在几个不同类型的依赖关系可以应用，这可能是有用的。例如，您可以提供离线 的时候没有互联网连接可用，将搜索目录。首先定义简单的注解：
```java?linenums
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Offline {

}
```
然后注释添加到字段或作为自动连接：
```java?linenums
public class MovieRecommender {

    @Autowired
    @Offline
    private MovieCatalog offlineCatalog;

    // ...
}
```


现在，这个bean定义只需要一个限定type：

```xml


<bean class="example.SimpleMovieCatalog">
    <qualifier type="Offline"/>
    <!-- inject any dependencies required by this bean -->
</bean>


```

您也可以定义接受一个名为除了属性或代替简单的自定义限定器注解 value属性。如果在字段或参数，然后指定多个属性值作为自动连接，一个bean定义必须匹配 所有这些属性的值被认为是一个自动装配候选。作为一个例子，考虑下面的注释定义：
```java?linenums
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface MovieQualifier {

    String genre();

    Format format();
}
```


在这种情况下Format是一个枚举：
```java?linenums
public enum Format {
    VHS, DVD, BLURAY
}
```


作为自动连接这些字段将与自定义的限定，包括了每个属性值：genre和format。
```java?linenums
public class MovieRecommender {

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Action")
    private MovieCatalog actionVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Comedy")
    private MovieCatalog comedyVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.DVD, genre="Action")
    private MovieCatalog actionDvdCatalog;

    @Autowired
    @MovieQualifier(format=Format.BLURAY, genre="Comedy")
    private MovieCatalog comedyBluRayCatalog;

    // ...
}
```


最终，这个bean定义应该与限定器匹配的值。此实施例还证明豆元属性可被用来代替 <qualifier/>子元素。如果有，<qualifier/>和它的属性优先，但自动连接机制将依赖内所提供的价值 <meta/>标签，如果没有这样的限定词存在，如在下面的示例中的最后两个bean定义。
```xml


<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Action"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Comedy"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="DVD"/>
        <meta key="genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="BLURAY"/>
        <meta key="genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

</beans>
```
#### 1.9.5. 使用仿制药自动装配预选赛
除了@Qualifier注释，还可以使用Java泛型类型作为资格的隐形式。例如，假设您有以下配置：
```java?linenums
@Configuration
public class MyConfiguration {

    @Bean
    public StringStore stringStore() {
        return new StringStore();
    }

    @Bean
    public IntegerStore integerStore() {
        return new IntegerStore();
    }
}
```


假设上述bean实现一个通用接口，即Store<String>和 Store<Integer>，你可以@Autowire在Store界面和通用将作为一个限定：

```java?linenums


@Autowired
private Store<String> s1; // <String> qualifier, injects the stringStore bean

@Autowired
private Store<Integer> s2; // <Integer> qualifier, injects the integerStore bean


```
自动装配清单时，地图和阵列一般预选赛也适用：
```java?linenums
// Inject all Store beans as long as they have an <Integer> generic
// Store<String> beans will not appear in this list
@Autowired
private List<Store<Integer>> s;
```
#### 1.9.6. CustomAutowireConfigurer上


的 CustomAutowireConfigurer 是BeanFactoryPostProcessor，使您能够注册即使他们不与Spring的注解自己的自定义限定注释类型@Qualifier的注释。
```xml


<bean id="customAutowireConfigurer"
        class="org.springframework.beans.factory.annotation.CustomAutowireConfigurer">
    <property name="customQualifierTypes">
        <set>
            <value>example.CustomQualifier</value>
        </set>
    </property>
</bean>

```


在AutowireCandidateResolver由确定自动装配候选：

 - 的autowire-candidate每个bean定义的值

- 任何default-autowire-candidates图案（多个）上的可用<beans/>元件

- 存在@Qualifier注释和任何自定义注解与注册CustomAutowireConfigurer

当多个豆有资格作为候选自动装配中，“主要”的判定为执行以下操作：如果所述候选者中恰好一个bean定义具有primary 设置为属性true，它将被选中。
#### 1.9.7. @Resource


Spring还支持注射使用JSR-250 @Resource的字段或bean属性setter方法的注释。这是在JSF 1.2管理豆或JAX-WS 2.0端点的Java EE 5和6的共同图案，例如。Spring支持这种模式的Spring管理对象也是如此。

@Resource需要一个name属性，缺省时，Spring解释该值作为bean的名字将被注入。换句话说，它遵循按名称的语义，如在本实施例证实：
```java?linenums
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource(name="myMovieFinder")
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```


如果没有明确指定名称，默认名称是从字段名或者setter方法的。在现场的情况下，它需要的字段名; 在setter方法的情况下，它需要的bean属性的名称。所以，下面的例子将不得不注射到它的setter方法名“的MovieFinder”豆：
```java?linenums
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```
```

提供注解的名称解析由一个bean的名称 ApplicationContext，其中的CommonAnnotationBeanPostProcessor知道。名称可以通过JNDI，如果你配置Spring的解决 SimpleJndiBeanFactory 明确。不过，建议您依靠缺省行为与Spring的JNDI查找功能来保存间接的水平。
```


在专属情况下，@Resource不指定明确的名称，以及类似的使用@Autowired，@Resource发现的主要类型的比赛，而不是一个具体的bean并解决众所周知的解析依存关系：BeanFactory， ApplicationContext，ResourceLoader，ApplicationEventPublisher，和MessageSource 接口。

因此，在下面的例子中，customerPreferenceDao场第一查找名为一个customerPreferenceDao豆，然后回落到主类型匹配的类型 CustomerPreferenceDao。的“上下文”字段是基于所述已知的可解析的依赖性型注入ApplicationContext。
```java?linenums
public class MovieRecommender {

    @Resource
    private CustomerPreferenceDao customerPreferenceDao;

    @Resource
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...
}
```
#### 1.9.8. @PostConstruct和@PreDestroy


将CommonAnnotationBeanPostProcessor不仅承认了@Resource注解也是JSR-250 的生命周期注解。在Spring 2.5中引入的，对于这些注释的支持提供了另一种替代那些描述 初始化回调和 销毁回调。提供的 CommonAnnotationBeanPostProcessor是在Spring内注册 ApplicationContext，携带这些注释中的一个的方法，在生命周期的作为相应的弹簧生命周期接口的方法在相同的点调用或显式声明回调方法。在下面的示例中，高速缓存将被预先填充在初始化时与销毁。
```java?linenums
public class CachingMovieLister {

    @PostConstruct
    public void populateMovieCache() {
        // populates the movie cache upon initialization...
    }

    @PreDestroy
    public void clearMovieCache() {
        // clears the movie cache upon destruction...
    }
}
```
```
关于组合不同的生命周期机制的效果的详细信息，请参阅 结合生命周期机制。
```
### 1.10. 类路径扫描和托管组件


在本章中使用XML最实例来指定产生每个配置元数据BeanDefinitionSpring容器内。在上一节（基于注解的容器配置）演示了如何通过代码级注解提供大量的结构的元数据。即使是在那些例子，然而，“基地” bean定义明确的XML文件中定义，而只注释驱动的依赖注入。本节描述了一个选项，隐含地检测 候选组件通过扫描类路径。候选组件是匹配针对过滤标准，并且具有与所述容器中注册的对应bean定义的类。这样就省去了使用XML来进行注册豆; 相反，您可以使用标注（例如@Component），AspectJ的类型的表达式，或您自己的自定义过滤器标准来选择哪些类将用容器注册的bean定义。
```


与Spring 3.0开始，由Spring JavaConfig项目提供了很多功能都是核心Spring框架的一部分。这允许您定义使用Java，而不是使用传统的XML文件豆类。看看的@Configuration，@Bean， @Import，和@DependsOn注释有关如何使用这些新功能的例子。

```
#### 1.10.1. @Component和进一步典型化注解


的@Repository注释是用于满足所述角色或任何类的标记 构造型的存储库（也被称为数据访问对象或DAO）的。中该标志物的用途是如在描述的异常进行自动翻译 异常翻译。

Spring提供进一步典型化注解：@Component，@Service，和 @Controller。@Component为任何Spring管理组件的通用刻板印象。 @Repository，@Service和@Controller是的特例@Component分别更具体的使用情况，例如，在持久性，服务，和表示层。因此，你可以用你的注解组件类 @Component，但如果用注解它们@Repository，@Service或者@Controller ，你的类能更好地被工具处理，或与切面进行关联。例如，这些典型化注解可以成为理想的切入点目标。这也有可能是@Repository，@Service和@Controller可携带Spring Framework的未来版本中为更多的语义。因此，如果你正在使用之间进行选择@Component或@Service为您服务层，@Service显然是更好的选择。同样，如上所述，@Repository已经支持为持久层的自动异常转换的标志。

#### 1.10.2. 元注释


许多Spring提供的注释可作为元注释在自己的代码。甲元注释仅仅是可应用于另一注释的注释。例如，@Service上面提到的注释是间注解为 @Component：
```java?linenums
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component // Spring will see this and treat @Service in the same way as @Component
public @interface Service {

    // ....
}
```


元注释也可以结合起来，创造组成的注释。例如，@RestController从Spring MVC的注解组成的@Controller和 @ResponseBody。

另外，由注释可以任选地从元注释重新声明属性，以允许用户定制。当你想只露出元注释的属性的子集，这可以是特别有用。例如，Spring的 @SessionScope注解的硬编码范围的名字session，但仍允许的定制proxyMode。
```java?linenums
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Scope(WebApplicationContext.SCOPE_SESSION)
public @interface SessionScope {

    /**
     * Alias for {@link Scope#proxyMode}.
     * <p>Defaults to {@link ScopedProxyMode#TARGET_CLASS}.
     */
    @AliasFor(annotation = Scope.class)
    ScopedProxyMode proxyMode() default ScopedProxyMode.TARGET_CLASS;

}
```


@SessionScope然后可以在不声明被使用proxyMode如下：
```java?linenums
@Service
@SessionScope
public class SessionScopedService {
    // ...
}
```


或为一个覆盖值proxyMode，如下所示：
```java?linenums
@Service
@SessionScope(proxyMode = ScopedProxyMode.INTERFACES)
public class SessionScopedUserService implements UserService {
    // ...
}
```
有关详细信息，请参阅 春批注编程模型 的wiki页面。
#### 1.10.3. 自动地检测的类和注册bean定义


Spring可以自动检测定型类和相应的登记 BeanDefinition使用S ApplicationContext。例如，下面的两个类是满足这种自动检测：
```java?linenums
@Service
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```
```java?linenums
@Repository
public class JpaMovieFinder implements MovieFinder {
    // implementation elided for clarity
}
```


为了检测这些类并注册相应的豆，你需要添加 @ComponentScan到您的@Configuration类，这里的basePackages属性是一种常见的父包为两班。（可替换地，可以指定一个逗号/分号/空格分隔的列表，其中包括每一类的父包）。
```java?linenums
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    ...
}
```


为简明，上述可能已使用value的注释，即属性@ComponentScan("org.example")
下面是使用XML的替代
```xml


<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example"/>

</beans>

```
```


采用<context:component-scan>显式地开启的功能 <context:annotation-config>。通常没有必要列入 <context:annotation-config>当使用元素<context:component-scan>。
```
```


类路径包的扫描需要在类路径对应的目录条目的存在。当你建立的JAR用Ant，请确保你没有 激活JAR任务的唯一文件开关。此外，类路径的目录可能不会基于JDK 1.7.0_45及更高版本（这需要在你的清单“可信图书馆设置在某些环境中的安全策略，如独立应用程序暴露出来;看到 http://stackoverflow.com/questions/ 19394570 / Java的JRE-7u45-符类加载器的-getresources）。

在JDK 9的模块路径（拼图），Spring的类路径扫描通常按预期工作。但是，请确保您的组件类中导出的module-info 描述; 如果你希望春天来调用你的类的非公共成员，确保他们是“打开”（即使用一个opens声明，而不是一个的exports 在声明中module-info描述）。

```


此外，AutowiredAnnotationBeanPostProcessor和 CommonAnnotationBeanPostProcessor都隐含当您使用组件扫描元件包括在内。这意味着，这两个部件被自动检测和 连接在一起-无需在XML提供任何bean配置元数据。

```xml
可以禁用的登记AutowiredAnnotationBeanPostProcessor和 CommonAnnotationBeanPostProcessor通过包括注解的配置具有值属性false。
```
#### 1.10.4. 使用过滤器自定义扫描


默认情况下，类注有@Component，@Repository，@Service， @Controller，或者本身都标注有一个自定义的注释@Component是唯一检测到的候选组件。但是，您可以修改，并简单地通过自定义过滤扩展这一行为。将其添加为includeFilters或excludeFilters 所述的参数@ComponentScan注释（或作为包括过滤器或排除过滤器 的的子元素component-scan元素）。每个过滤器元件需要type 和expression属性。下表介绍了过滤选项。


下面的例子显示了配置忽略所有@Repository注解并用“存根”储存库来代替。
```java?linenums
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    ...
}
```
和等效采用XML
```


<beans>
    <context:component-scan base-package="org.example">
        <context:include-filter type="regex"
                expression=".*Stub.*Repository"/>
        <context:exclude-filter type="annotation"
                expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>
</beans>


```
```


您也可以通过设置禁用默认的过滤器useDefaultFilters=false上标注或提供use-default-filters="false"的的属性<component-scan/>元素。这将在关闭对使用注解的类自动检测@Component，@Repository， @Service，@Controller，或@Configuration。

```
#### 1.10.5. 定义组件中的元数据豆


Spring组件也有助于bean定义元数据的容器。此功能是通过同样@Bean用于中定义的bean元数据注解@Configuration 注解的类。下面是一个简单的例子：
```java?linenums
@Component
public class FactoryMethodComponent {

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    public void doWork() {
        // Component method implementation omitted
    }
}
```


这个类是具有包含在其应用程序特定的代码Spring组件 doWork()方法。然而，也有利于有一个工厂方法指的是方法的bean定义publicInstance()。该@Bean注释标识工厂方法和其它bean定义特性，如通过一个限定值@Qualifier注释。可以指定其他方法级别的注解是 @Scope，@Lazy和自定义限定器注解。
```


除了其对组件初始化作用下，@Lazy注释也可以被放置在标有注入点@Autowired或@Inject。在这种情况下，它会导致一个懒散分辨率代理的注入。

```


如先前所讨论的自动连接的字段和方法都被支持，具有用于自动装配的额外支持@Bean的方法：
```java?linenums

@Component
public class FactoryMethodComponent {

    private static int i;

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    // use of a custom qualifier and autowiring of method parameters
    @Bean
    protected TestBean protectedInstance(
            @Qualifier("public") TestBean spouse,
            @Value("#{privateInstance.age}") String country) {
        TestBean tb = new TestBean("protectedInstance", 1);
        tb.setSpouse(spouse);
        tb.setCountry(country);
        return tb;
    }

    @Bean
    private TestBean privateInstance() {
        return new TestBean("privateInstance", i++);
    }

    @Bean
    @RequestScope
    public TestBean requestScopedInstance() {
        return new TestBean("requestScopedInstance", 3);
    }
}
```


该示例autowires的String方法参数country的的值age 上另一个名为bean属性privateInstance。一个Spring表达式语言元素通过符号定义属性的值#{ <expression> }。对于@Value 注释，表达式解析器解析预先文字表达时要寻找bean的名字。

由于Spring框架4.3的，你可能需要声明类型的工厂方法参数 InjectionPoint（或它的更具体的子类DependencyDescriptor，以便访问请求注入点触发当前Bean的创建）。请注意，这仅适用于实际创建bean实例，而不是现有实例的注入。因此，此功能使原型范围的豆类最有意义。对于其他范围，工厂方法将只看到注射点，这引发了给定范围内一个新的bean实例的创建：例如，触发一个懒惰的singleton的bean创建的依赖。使用带有语义护理提供注射点的元数据在这样的情况下。
```java?linenums
@Component
public class FactoryMethodComponent {

    @Bean @Scope("prototype")
    public TestBean prototypeInstance(InjectionPoint injectionPoint) {
        return new TestBean("prototypeInstance for " + injectionPoint.getMember());
    }
}
```


将@Bean在普通的Spring组件方法比春天里的同行处理方式不同@Configuration类。不同的是，@Component 类不与CGLIB增强拦截的方法和字段的调用。CGLIB代理是通过该内调用方法或字段的装置@Bean的方法@Configuration的类，以创建协作对象的元数据豆引用; 这些方法都没有用通常的Java语义调用，而是经过容器，以便通过编程调用执行参考其它豆类，即使提供的Spring bean通常的生命周期管理和代理@Bean的方法。与此相反，在调用一个方法或字段@Bean的纯内方法@Component 类具有 标准的Java语义，没有特殊CGLIB处理或其他约束应用。
```


你可以宣布@Bean方法为static，允许他们不创造他们包含配置类的一个实例调用。定义当后处理器豆类，例如类型的这使得特定意义BeanFactoryPostProcessor或 BeanPostProcessor，因为这种豆将获得在容器生命周期的早期初始化，并且应避免在该点触发的配置的其他部分。

请注意，调用静态@Bean方法将不会因容器得到截获，甚至没有在@Configuration类（见上文）。这是由于技术的限制：CGLIB子类只能重写非静态方法。因此，另一种直接调用@Bean的方法将有标准的Java语义，从而导致一个独立的情况下被从工厂方法本身直返回。

Java语言的知名度@Bean方法没有在Spring的容器上所产生的bean定义产生直接影响。你认为合适的非您可以自由申报的工厂方法@Configuration的类，也为静态方法的任何地方。然而，常规@Bean的方法@Configuration类需要被重写，即它们不能被声明为private或final。

@Bean方法也将在一个给定的组件或结构类的基类发现，以及在由所述组分或结构类实现接口中声明的Java 8默认方法。这使得很多在合成复杂的配置安排的灵活性，甚至多重继承通过Java 8默认的方法为春季4.2的是可能的。

最后，请注意，一个类可以拥有多个@Bean针对同一个bean的方法，为多个工厂方法的安排取决于在运行时可依赖使用。这是相同的算法作为选择在其它配置方案的“贪婪”构造或工厂方法：用可满足的依赖性将在施工时间被拾取，类似于容器多个之间如何选择最大数量的变体@Autowired的构造。

```

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