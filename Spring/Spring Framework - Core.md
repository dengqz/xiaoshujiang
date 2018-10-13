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
#### 1.10.6. 命名自动检测组件


当组件被自动检测作为扫描过程的一部分，由所产生的bean名称BeanNameGenerator已知的是扫描仪的策略。默认情况下，任何Spring刻板印象注释（@Component，@Repository，@Service，和 @Controller包含）的名字 value将因此提供的名字相应的bean定义。

如果这样的注释不包含名称 value或用于任何其他检测到的成分（如那些由自定义过滤器发现），默认bean名称发生器返回小写形式非限定类名。例如，如果检测到以下的组件类，名称将是myMovieLister与movieFinderImpl：
```java?linenums
@Service("myMovieLister")
public class SimpleMovieLister {
    // ...
}
```
```java?linenums
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```


如果你不想依赖默认bean命名策略，可以提供一个自定义的bean命名策略。首先，实现 BeanNameGenerator 接口，并确保包括默认的无参数的构造函数。然后在配置扫描器时提供全限定类名：
```java?linenums
@Configuration
@ComponentScan(basePackages = "org.example", nameGenerator = MyNameGenerator.class)
public class AppConfig {
    ...
}
```
```xml


<beans>
    <context:component-scan base-package="org.example"
        name-generator="org.example.MyNameGenerator" />
</beans>


```


作为一般规则，考虑每当其他部件可以给它做明确提到注解指定名称。在另一方面，自动生成的名称就足够每当容器负责接线。
#### 1.10.7. 提供自动检测的组件提供一个作用域


与一般的Spring管理组件，自动检测组件的默认和最常用的范围singleton。不过，有时你需要，可以通过指定一个不同的范围@Scope注释。只需提供注释中范围的名称：
```java?linenums
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

```
@Scope注释仅内省在混凝土豆类（注释组件）或工厂方法（@Bean方法）。与此相反，以XML bean定义，没有概念bean定义继承和继承层次在类级别无关元数据的目的。
```


有关特定网络范围的细节，如在Spring上下文“请求” /“会话”，见请求，会话，应用和WebSocket的范围。就像那些范围预建的注解，你也可以使用Spring的元注释的方法创作自己的作用域注释：如自定义注解元标注有@Scope("prototype")，也可能宣布自定义范围的代理模式。
```
为了提供范围的分辨率，而不是依赖于基于注解的方法定制策略，实现 接口，并且一定要包括一个默认的无参数的构造函数。然后在配置扫描器时提供全限定类名：ScopeMetadataResolver
```
```java?linenums
@Configuration
@ComponentScan(basePackages = "org.example", scopeResolver = MyScopeResolver.class)
public class AppConfig {
    ...
}
```
```xml


<beans>
    <context:component-scan base-package="org.example" scope-resolver="org.example.MyScopeResolver"/>
</beans>


```


当使用某些非单范围，可能有必要以产生用于所述范围内的对象的代理。推理中描述作用域bean与依赖。为了这个目的，一个范围的代理属性是可用的组件的扫描元件上。三个可能的值是：无，接口和targetClass。例如，下面的配置会导致标准JDK动态代理：
```java?linenums
@Configuration
@ComponentScan(basePackages = "org.example", scopedProxy = ScopedProxyMode.INTERFACES)
public class AppConfig {
    ...
}
```
```xml


<beans>
    <context:component-scan base-package="org.example" scoped-proxy="interfaces"/>
</beans>


```
#### 1.10.8. 提供与注释限定符元数据


在@Qualifier注释中讨论微调基于注解的自动连接与预选赛。在这一节中的示例演示了如何使用的@Qualifier注释和自定义限定器注解提供细粒度的控制，当你解决自动装配的候选者。因为这些实施例是基于XML bean定义，限定符元数据用的是设置在候选bean定义qualifier或meta 所述的子元素bean中的XML元素。当classpath扫描来自动检测组件，你提供的候选类型级别的注解限定符元数据。下面的三个例子演示了这种技术：
```java?linenums
@Component
@Qualifier("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```
```java?linenums
@Component
@Genre("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```
```java?linenums
@Component
@Offline
public class CachingMovieCatalog implements MovieCatalog {
    // ...
}
```
```


与大多数基于注解的替代品，记住注解元数据是绑定到类定义本身，而使用XML允许多个豆 同一类型的，以提供其限定符元数据的变化，因为元数据是由每个提供-instance而不是每个类。

```
#### 1.10.9. 产生候选组件的索引


虽然classpath中扫描速度非常快，可以通过在编译时产生候选人的静态列表，以改善大型应用程序的启动性能。在这种模式下，所有模块的应用程序必须使用此机制中，当 ApplicationContext检测到这样的指数，它会自动使用它，而不是在扫描类路径。

生成索引，简单地添加一个额外的依赖于包含对于成分扫描指示靶成分的每个模块：
```xml


<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-indexer</artifactId>
        <version>5.0.7.RELEASE</version>
        <optional>true</optional>
    </dependency>
</dependencies>

```
或者，使用摇篮：
```
dependencies {
    compileOnly("org.springframework:spring-context-indexer:5.0.7.RELEASE")
}
```
这个过程中会产生META-INF/spring.components那将被包含在jar文件
```xml

当这种模式在IDE中工作时，spring-context-indexer必须注册为注释处理器，以确保该指数是最新的，当候选组件进行更新。
```
```xml

当该指数自动启用META-INF/spring.components在类路径中。如果索引部分可用一些库（或用例），但整个应用程序无法建立，你可以回退到普通类路径列（如没有索引存在的话）通过设置spring.index.ignore到 true，无论是作为一个系统属性或以spring.properties在类路径的根文件。

```
### 1.11. 使用JSR 330标准注释


与Spring 3.0开始，Spring提供的JSR-330标准的标注（依赖注入）的支持。这些注释都以同样的方式，Spring的注解扫描。你只需要有相关的罐子在classpath。
```


如果您使用的是Maven，javax.inject神器是在标准Maven仓库（可 http://repo1.maven.org/maven2/javax/inject/javax.inject/1/）。您可以添加以下依赖于你的文件的pom.xml：

<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>


```
#### 1.11.1. 依赖注入与@Inject和@Named


相反的@Autowired，@javax.inject.Inject也可以使用如下：
```java?linenums
import javax.inject.Inject;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.findMovies(...);
        ...
    }
}
```


如@Autowired，可以使用@Inject在现场级，方法水平和构造参数的水平。此外，你可以宣布你的注入点的 Provider，允许按需访问，以更短的范围，或者通过一个懒惰访问其他豆豆Provider.get()调用。如上面的例子的变体：
```java?linenums
import javax.inject.Inject;
import javax.inject.Provider;

public class SimpleMovieLister {

    private Provider<MovieFinder> movieFinder;

    @Inject
    public void setMovieFinder(Provider<MovieFinder> movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.get().findMovies(...);
        ...
    }
}
```


如果你想使用一个合格的名称应注入的依赖，你应该使用@Named如下注解：
```java?linenums
import javax.inject.Inject;
import javax.inject.Named;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(@Named("main") MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```


像@Autowired，@Inject也可与使用java.util.Optional或 @Nullable。这是更适用在这里，因为@Inject没有一个required属性。
```java?linenums
public class SimpleMovieLister {

    @Inject
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        ...
    }
}
```
```java?linenums
public class SimpleMovieLister {

    @Inject
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        ...
    }
}
```
#### 1.11.2. @Named和@ManagedBean：标准等同于@Component注解


相反的@Component，@javax.inject.Named或javax.annotation.ManagedBean可以被使用如下：
```java?linenums
import javax.inject.Inject;
import javax.inject.Named;

@Named("movieListener")  // @ManagedBean("movieListener") could be used as well
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```


这是很常见的使用@Component而无需为组件指定名称。 @Named可以以类似的方式使用：
```java?linenums
import javax.inject.Inject;
import javax.inject.Named;

@Named
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```


当使用@Named或@ManagedBean，可以使用组件扫描以完全相同的方式使用Spring注释时，为：
```java?linenums
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    ...
}
```
```

在对比@Component中，JSR-330 @Named和JSR-250 ManagedBean 注解不是组合的。请使用Spring的原型模型构建自定义组件的注解。
```
#### 1.11.3. 标准的JSR-330注解的局限性
当与标准的标注工作，是要知道，如下图所示的表中的一些显着功能不可用是很重要的：

### 1.12. 基于Java的容器配置
#### 1.12.1. 基本概念：@Bean和@Configuration


在Spring的Java的新配置支持，中央文物是 @Configuration-annotated类和@Bean-annotated方法。

该@Bean注释被用于指示一个方法实例，配置和初始化为通过Spring IoC容器进行管理的新对象。对于那些熟悉Spring的<beans/>XML配置中的@Bean注释扮演了同样的角色<bean/>元素。您可以使用@Bean注解的方法与任何春天 @Component，然而，它们经常与使用@Configuration豆类。

注释与类@Configuration表示其主要目的是作为bean定义的来源。此外，@Configuration类允许豆间的依赖关系，以通过简单地调用其他被定义@Bean在相同的类中的方法。可能最简单的@Configuration类内容如下：
```java?linenums
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```


在AppConfig上面的类将等同于翌年春季<beans/>XML：
```xml


<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>

```
```
全@Configuration VS '精简版' @Bean模式？

当@Bean方法被类中声明的不带注释 @Configuration它们被称为在一个“精简版”模式正在处理中。在声明的bean的方法@Component或甚至在一个普通的旧类将被视为“精简版”，与含类的不同主要目的和@Bean只是被一种奖金的方法存在。例如，服务的部件可以通过一个附加暴露管理视图到所述容器@Bean上的每个适用组件类方法。在这样的情况下，@Bean方法是一个简单的通用工厂方法机构。

与完全@Configuration，精简版@Bean的方法不能声明bean间的依赖关系。相反，他们对自己的包含组件的内部状态，并可选的，他们可以声明的参数上操作。这种@Bean因此方法不应该调用其他 @Bean方法。每一个这样的方法是从字面上只是用于特定bean引用一个工厂的方法，无需任何特殊的运行时语义。这里正的副作用是，没有CGLIB子类具有在运行时被施加，所以有在类设计的术语（即含类可能仍然是没有限制final等）。

在常见的情况，@Bean方法是将内声明的@Configuration类，确保“全”模式一直使用，因此交叉方法引用将重定向到容器的生命周期管理。这将防止同样的 @Bean意外正通过一个普通Java调用这有助于减少微妙的错误在“精简版”模式下运行时可能很难追查调用的方法。
```


的@Bean和@Configuration注解将在深度在下面的章节中讨论。但是，首先，我们将讨论创建一个使用基于Java的配置的Spring容器的各种方式。
#### 1.12.2. 实例化使用AnnotationConfigApplicationContext Spring容器


下面的文档Spring的各部分AnnotationConfigApplicationContext，新的春天3.0。这种多用途的ApplicationContext实现是能够接受不仅 @Configuration类作为输入，而且纯@Component的类和类与JSR-330的元数据注释。

当@Configuration类作为输入提供的@Configuration类本身作为一个bean定义，并宣布所有@Bean的类中的方法也被注册为bean定义。

当@Component被提供和JSR-330类，它们被登记为bean定义，并且假定DI元数据，例如@Autowired或者@Inject是这些类中使用的必要。
结构简单

以几乎相同的方式，实例化时，Spring的XML文件用作输入 ClassPathXmlApplicationContext，@Configuration类可以实例化时可用作输入AnnotationConfigApplicationContext。这允许Spring容器的完全免费的XML的使用情况：
```java?linenums
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```


正如上面提到的，AnnotationConfigApplicationContext不限于只与工作@Configuration类。任何@Component或JSR-330注解的类可作为输入提供给构造提供。例如：
```java?linenums
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```


以上假设MyServiceImpl，Dependency1并Dependency2使用Spring依赖注入注解，例如@Autowired。
构建容器编程方式使用寄存器（类<？> ...）

一个AnnotationConfigApplicationContext可以使用一个无参数的构造被实例化，然后使用配置的register()方法。这种方法是非常有用的，当编程的方式构建的AnnotationConfigApplicationContext。
```java?linenums
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```
启用组件扫描与扫描（字符串...）

为了使组件扫描，只需标注你的@Configuration类，如下所示：
```java?linenums
@Configuration
@ComponentScan(basePackages = "com.acme")
public class AppConfig  {
    ...
}
```
```


经验丰富的Spring用户将熟悉XML声明相当于从Spring的的context:命名空间

<beans>
    <context:component-scan base-package="com.acme"/>
</beans>

```


另外，在上述的例子中，com.acme包将被扫描，寻找任何 @Component-annotated类和这些类将被注册为在容器内的Spring bean定义。AnnotationConfigApplicationContext暴露 scan(String…​)，以允许相同的分量扫描功能的方法：
```java?linenums


另外，在上述的例子中，com.acme包将被扫描，寻找任何 @Component-annotated类和这些类将被注册为在容器内的Spring bean定义。AnnotationConfigApplicationContext暴露 scan(String…​)，以允许相同的分量扫描功能的方法：
```
```java?linenums
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```
```


请记住，@Configuration类是元注释 用@Component，所以他们是组件扫描的候选人！在上面的例子中，假设AppConfig在中声明的com.acme封装（或任何包装的下面），它将被调用期间拾取scan()，并且在refresh()其所有的@Bean方法将被处理，并注册为在容器内豆的定义。

```


请记住，@Configuration类是元注释 用@Component，所以他们是组件扫描的候选人！在上面的例子中，假设AppConfig在中声明的com.acme封装（或任何包装的下面），它将被调用期间拾取scan()，并且在refresh()其所有的@Bean方法将被处理，并注册为在容器内豆的定义。
```xml


<web-app>
    <!-- Configure ContextLoaderListener to use AnnotationConfigWebApplicationContext
        instead of the default XmlWebApplicationContext -->
    <context-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </context-param>

    <!-- Configuration locations must consist of one or more comma- or space-delimited
        fully-qualified @Configuration classes. Fully-qualified packages may also be
        specified for component-scanning -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.acme.AppConfig</param-value>
    </context-param>

    <!-- Bootstrap the root application context as usual using ContextLoaderListener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Declare a Spring MVC DispatcherServlet as usual -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- Configure DispatcherServlet to use AnnotationConfigWebApplicationContext
            instead of the default XmlWebApplicationContext -->
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>
                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
            </param-value>
        </init-param>
        <!-- Again, config locations must consist of one or more comma- or space-delimited
            and fully-qualified @Configuration classes -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.acme.web.MvcConfig</param-value>
        </init-param>
    </servlet>

    <!-- map all requests for /app/* to the dispatcher servlet -->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>

```
#### 1.12.3. 使用@Bean注解


@Bean是一个方法级注释和XML的直接模拟<bean/>元件。注释支持一些所提供的属性<bean/>，如： 初始化方法， 破坏法， 自动装配和name。

你可以使用@Bean一个注释@Configuration-annotated或在 @Component-annotated类。
声明一个bean

要声明一个bean，只需用注释的方法@Bean的注释。您可以使用此方法内注册一个bean定义ApplicationContext指定为方法的返回值的类型。默认情况下，bean名称将是相同的方法名。下面是一个的一个简单的例子@Bean方法声明：
```java?linenums
@Configuration
public class AppConfig {

    @Bean
    public TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}
```
上述配置是完全等同于下面的Spring XML：
```xml


<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>

```


双方的声明作出的bean transferService中可用 ApplicationContext，势必类的对象实例TransferServiceImpl：


双方的声明作出的bean transferService中可用 ApplicationContext，势必类的对象实例TransferServiceImpl：
```
transferService  - > com.acme.TransferServiceImpl
```

您也可以宣布你@Bean用的接口（或基类）的返回类型的方法：
```java?linenums
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```


然而，这限制了用于事先预测类型的可见性，以指定的接口类型（TransferService）然后，用全类型（TransferServiceImpl）仅一次受影响的单豆已经被实例化已知的容器。非懒单豆获得根据自己的声明顺序初始化，所以这取决于当其他组件试图通过非声明的类型相匹配的（比如你可能会看到不同类型的匹配结果@Autowired TransferServiceImpl 这一次的“transferService”豆已经将只解决实例化）。
```


如果你一直被申报服务接口是指你的类型，你的 @Bean返回类型可以放心地加入该设计决策。然而，对于实现多个接口组件或通过其实施类型可能提及的部件，是比较安全的声明最具体的返回类型可能（至少需要通过注射点指的是你的bean作为特定的）。

```
豆依赖

甲@Bean注解的方法可以具有描述构建豆所需要的依赖关系的参数的任意数量。例如，如果我们TransferService 需要AccountRepository，我们可以通过物化的方法参数的依赖：
```java?linenums
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}
```


该解决机制是几乎等同于基于构造函数的依赖注入，见相关章节了解更多详情。
接收生命周期回调

与定义的任何类@Bean注释支持定时生命周期回调，可以使用@PostConstruct并@PreDestroy注解从JSR-250，见 JSR-250注解进一步的细节。

常规春天的生命周期回调的充分支持，以及。如果一个bean实现InitializingBean，DisposableBean或者Lifecycle，它们各自的方法是由容器调用。

一套标准的*Aware如接口实现BeanFactoryAware， BeanNameAware， MessageSourceAware， 了ApplicationContextAware等也完全支持。

该@Bean注释支持指定任意初始化和销毁回调方法，就像春天XML的init-method，并destroy-method在属性上的bean元素：
```java?linenums
public class Foo {

    public void init() {
        // initialization logic
    }
}

public class Bar {

    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public Foo foo() {
        return new Foo();
    }

    @Bean(destroyMethod = "cleanup")
    public Bar bar() {
        return new Bar();
    }
}
```
```


默认情况下，豆类使用具有公共Java的配置定义close或shutdown 方法自动销毁回调入伍。如果你有一个公共 close或shutdown方法，你不希望它被称为当容器关闭时，只需添加@Bean(destroyMethod="")到您的bean定义禁用默认(inferred)模式。

你可能要为你通过JNDI获得其生命周期的应用程序之外管理的资源做的，默认情况下。特别是，确保始终为做到这一点DataSource，因为它被称为是对Java EE应用服务器问题。

@Bean(destroyMethod="")
public DataSource dataSource() throws NamingException {
    return (DataSource) jndiTemplate.lookup("MyDS");
}

此外，有@Bean方法，你通常会选择使用程序JNDI查找：可以使用Spring提供JndiTemplate/ JndiLocatorDelegate佣工或直JNDI InitialContext的使用，而不是JndiObjectFactoryBean这将迫使你声明的返回类型的变异FactoryBean类型，而不是实际的目标类型，使得它很难用在其他交叉引用调用@Bean的方法是打算指所提供的资源在这里。

```


当然，在的情况下Foo的上方，这将是同样是有效的调用init() 施工期间直接方法：
```java?linenums
@Configuration
public class AppConfig {

    @Bean
    public Foo foo() {
        Foo foo = new Foo();
        foo.init();
        return foo;
    }

    // ...
}
```
当您直接在Java中工作，你可以做任何你与你喜欢的对象，并不总是需要依赖于容器的生命周期！
指定bean的作用域
使用@Scope注解

您可以指定与定义你的bean @Bean注释应该有一个具体的范围。你可以使用任何在规定的标准范围的 Bean的作用域部分。

默认范围singleton，但你可以用覆盖此@Scope注释：
```java?linenums
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("prototype")
    public Encryptor encryptor() {
        // ...
    }
}
```
@Scope和作用域代理

Spring提供了通过范围的相关性问题的一种方便的方式 范围代理。使用XML配置时建立这样一个代理最简单的方法是<aop:scoped-proxy/>元件。用@Scope注解配置在Java中的豆类提供了与proxyMode属性相当的支持。默认为无代理（ScopedProxyMode.NO），但您可以指定ScopedProxyMode.TARGET_CLASS或ScopedProxyMode.INTERFACES。

如果您端口从XML参考文档范围代理的例子（见前面的链接），我们@Bean使用Java，它看起来像下面这样：
```java?linenums
// an HTTP Session-scoped bean exposed as a proxy
@Bean
@SessionScope
public UserPreferences userPreferences() {
    return new UserPreferences();
}

@Bean
public Service userService() {
    UserService service = new SimpleUserService();
    // a reference to the proxied userPreferences bean
    service.setUserPreferences(userPreferences());
    return service;
}
```
定制豆命名

默认情况下，配置类使用@Bean方法的名称所产生的bean的名字。该功能可覆盖然而，与name属性。
```java?linenums
@Configuration
public class AppConfig {

    @Bean(name = "myFoo")
    public Foo foo() {
        return new Foo();
    }
}
```
豆走样

正如所讨论的命名豆，有时希望以给出单个豆多个名称，否则称为豆混叠。所述name的的属性@Bean 注释接受用于此目的的字符串数组。
```java?linenums
豆走样

正如所讨论的命名豆，有时希望以给出单个豆多个名称，否则称为豆混叠。所述name的的属性@Bean 注释接受用于此目的的字符串数组。
```
豆说明

有时它是有帮助的，以提供一个bean的更详细的文字描述。当豆暴露（可能通过JMX）用于监测目的这可以是特别有用的。

为了说明添加到@Bean的 @Description 注释，可以使用：
```java?linenums
@Configuration
public class AppConfig {

    @Bean
    @Description("Provides a basic example of a bean")
    public Foo foo() {
        return new Foo();
    }
}
```
#### 1.12.4. 使用@Configuration注解


@Configuration是指示对象是bean定义源的类级注释。@Configuration班宣布通过公共豆@Bean注解的方法。调用@Bean的方法@Configuration类也可以用于定义bean间的依赖关系。请参阅基本概念：@Bean和@Configuration了总体介绍。
注射bean间的依赖关系

当@Bean■找上彼此依存关系，表达该依赖性是为具有一个bean方法调用另一个简单：
```java?linenums
@Configuration
public class AppConfig {

    @Bean
    public Foo foo() {
        return new Foo(bar());
    }

    @Bean
    public Bar bar() {
        return new Bar();
    }
}
```


在上面的例子中，foo豆接收参考到bar经由构造器注入。
```
 	

声明bean间的依赖关系的这种方法只有在工作@Bean方法是内声明的@Configuration类。你不能声明使用普通的bean间的依赖关系@Component类。
```
查找方法注射

正如前面提到的，查询方法注入是一种先进的功能，你应该很少使用。它是在一个单作用域的bean对prototype作用域的bean的依赖情况下非常有用。使用Java进行这种类型的配置提供了一种实现这种图案的自然方式。
```java?linenums
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


使用Java的配置支持，您可以创建一个子类CommandManager，其中的抽象createCommand()方法，以这样一种方式，它看起来了一个新的（原型）命令对象覆盖：
```java?linenums
@Bean
@Scope("prototype")
public AsyncCommand asyncCommand() {
    AsyncCommand command = new AsyncCommand();
    // inject dependencies here as required
    return command;
}

@Bean
public CommandManager commandManager() {
    // return new anonymous implementation of CommandManager with command() overridden
    // to return a new prototype Command object
    return new CommandManager() {
        protected Command createCommand() {
            return asyncCommand();
        }
    }
}
```
有关基于Java的配置内部是如何工作的更多信息

下面的例子显示了一个@Bean注释的方法被调用两次：
```java?linenums
@Configuration
public class AppConfig {

    @Bean
    public ClientService clientService1() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientService clientService2() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientDao clientDao() {
        return new ClientDaoImpl();
    }
}
```


clientDao()一旦已经在叫clientService1()，一次在clientService2()。由于此方法创建一个新的实例ClientDaoImpl并返回它，你通常会期望有2个实例（每个服务）。那肯定是有问题的：在春季，实例化豆的singleton默认范围。这是魔术进来：所有的@Configuration类都在启动时使用的子类CGLIB。在子类中，子方法检查容器第一对任何缓存的（作用域）豆，它调用父类的方法，并创建一个新的实例之前。注意，春季3.2，它是不再需要CGLIB添加到您的类路径，因为CGLIB类已下重新包装org.springframework.cglib和弹簧芯JAR中直接包含。



该行为可以根据你的bean的范围是不同的。我们在这里谈论单身。
```


还有，由于这样的事实，CGLIB动态在启动时增加了功能，有一些限制，特别是配置类绝不能是最终。然而，如为4.3，任何构造允许上配置类，包括使用的 @Autowired或用于默认注射单个非默认构造函数声明。

如果你喜欢，以避免任何CGLIB强加的限制，考虑宣布你@Bean 对非方法@Configuration类，例如在普通@Component类来代替。跨方法调用之间的@Bean方法将不会拦截的话，所以你必须在构造函数或方法级别有完全依赖于依赖注入。
```
#### 1.12.5. 撰写基于Java的配置
使用@Import注解

多为<import/>元素用于Spring的XML文件中的模块化配置，以帮助时，@Import注释允许加载@Bean从另一个配置类定义：
```java?linenums
@Configuration
public class ConfigA {

    @Bean
    public A a() {
        return new A();
    }
}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

    @Bean
    public B b() {
        return new B();
    }
}
```


现在，而不需要同时指定ConfigA.class和ConfigB.class实例化上下文时，只ConfigB需要明确地提供：
```java?linenums
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

    // now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
```


这种方法简化了容器实例化，因为只有一个类需要加以处理，而不是要求开发商要记住潜在的大量的 @Configuration施工过程中的类。
```


由于Spring框架4.2，@Import还支持常规组件类，类似的参考AnnotationConfigApplicationContext.register方法。这是特别有用，如果你想避免组件扫描，使用一些配置类为切入点，明确界定所有组件。

```
进口@Bean定义注入依赖

上面的例子中工作，但过于简单。在大多数实际情况下，豆类将有跨越配置类相互依存关系。当使用XML，这不是问题本身，因为没有涉及到的编译器，以及一个可以简单地声明 ref="someBean"，并相信Spring将容器初始化期间做出来。当然，在使用的时候@Configuration类，Java编译器放置在配置模型的约束，在向其他豆类引用必须是有效的Java语法。

幸运的是，解决这个问题很简单。正如我们已经讨论的， @Bean方法可以具有描述豆依赖性参数的任意数量。让我们考虑一个更真实的场景与几个@Configuration 班，每个取决于其他声明豆：
```java?linenums
@Configuration
public class ServiceConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```


还有另一种方式来达到同样的效果。请记住，@Configuration类基本上只是在容器中另一个bean：这意味着他们可以利用 @Autowired和@Value注射等，就像任何其他的豆！
```


请确保你注入这种方式的依赖是最简单的一种而已。@Configuration 类上下文的初始化过程中很早就处理过，迫使依赖被注入这种方式可能会导致意想不到的早期初始化。只要有可能，诉诸于基于参数的喷射在上面的例子。

此外，特别小心BeanPostProcessor，并BeanFactoryPostProcessor通过定义@Bean。那些通常应该声明为static @Bean方法，不触发其包含配置类的实例化。否则，@Autowired并且@Value不会对配置类本身，因为它是被作为一个bean实例创建太早上班。

```
```java?linenums
Configuration
public class ServiceConfig {

    @Autowired
    private AccountRepository accountRepository;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    private final DataSource dataSource;

    @Autowired
    public RepositoryConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```
```


在构造器注入@Configuration类仅支持为Spring框架4.3。还需要注意的是，没有必要指定@Autowired，如果目标bean定义只有一个构造函数; 在上面的例子中，@Autowired不进行必要的RepositoryConfig构造函数。
```
完全合格的进口豆轻松导航

在上述情况下，使用@Autowired效果良好，并提供所需的模块，但究竟确定在何处声明的自动装配bean定义仍然有些模糊。例如，作为一个开发者在看ServiceConfig，你怎么确切地知道该@Autowired AccountRepository豆声明？这不是在代码中明确，这可能只是罚款。请记住， 春天工具套件提供了工具，可以渲染图显示一切是如何连接起来-这可能是你所需要的。此外，你的Java IDE可以很容易地找到的所有声明和使用AccountRepository类型，很快就会显示出你的位置，@Bean即返回类型的方法。

在情况下，这种模糊性是不能接受的，你想有直接的导航功能IDE内从一@Configuration类到另一个，可以考虑自动装配配置类本身：
```java?linenums
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        // navigate 'through' the config class to the @Bean method!
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}
```


在上述情况下，它就是完全明确的AccountRepository定义。然而，ServiceConfig现在紧耦合RepositoryConfig; 这就是权衡。这种紧密耦合可以通过使用基于接口的或抽象的基于类的可有所减轻@Configuration类。考虑以下：
```java?linenums
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}

@Configuration
public interface RepositoryConfig {

    @Bean
    AccountRepository accountRepository();
}

@Configuration
public class DefaultRepositoryConfig implements RepositoryConfig {

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(...);
    }
}

@Configuration
@Import({ServiceConfig.class, DefaultRepositoryConfig.class})  // import the concrete config!
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return DataSource
    }

}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```


现在ServiceConfig是松耦合相对于混凝土 DefaultRepositoryConfig，并内置IDE工具仍然是有用的：它会很容易为开发者得到的类型层次RepositoryConfig的实现。以这种方式，导航@Configuration类和它们的依赖变得不大于导航基于接口的码的常规工艺不同。
```


如果你想影响某些豆类的启动创建顺序，考虑宣布一些为@Lazy（对于第一次访问，而不是在启动时创建）或@DependsOn某些其它豆类（确保特定的其他豆将在当前之前创建豆，超越了后者的直接依赖意味着什么）。
```
有条件包括@Configuration类或@Bean方法

它往往是有用的有条件地启用或禁用一个完整的@Configuration类，甚至有个别@Bean方法的基础上，一些任意的系统状态。这方面的一个常见的例子是使用@Profile注释来激活豆只有当特定的个人资料已经在Spring中启用Environment（见Bean定义的配置文件 的详细信息）。

该@Profile注释是使用所谓的更灵活的注释实际执行@Conditional。该@Conditional注释指示特定 org.springframework.context.annotation.Condition前应谘询的实施@Bean是注册。

所述的实施方式中Condition接口简单地提供一个matches(…​) 返回的方法true或false。例如，下面是实际 Condition用于实施@Profile：
```java?linenums
@Override
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    if (context.getEnvironment() != null) {
        // Read the @Profile annotation attributes
        MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
        if (attrs != null) {
            for (Object value : attrs.get("value")) {
                if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
                    return true;
                }
            }
            return false;
        }
    }
    return true;
}
```


请参阅的javadoc更多细节。 @Conditional
结合Java和XML配置

Spring的@Configuration类支持的目标并不是要成为的Spring XML 100％的完全替代。例如Spring XML命名空间的一些设施仍然配置容器的理想方式。无论实例化容器中使用的“XML为中心”的方式，例如：在XML是方便或必要的情况下，你有一个选择 ClassPathXmlApplicationContext，或者在“Java为中心”的方式使用 AnnotationConfigApplicationContext和@ImportResource根据需要导入XML注释。
XML为中心的使用@Configuration类

它可以优选地从引导XML Spring容器和包括 @Configuration在ad-hoc方式类。例如，在使用Spring XML大现有的代码库，这将是更容易地创建@Configuration一个为需要的基础上的类和它们包括从现有的XML文件。下面你会发现使用的选项@Configuration在这种“以XML为中心”的局面类。
声明@Configuration类为纯春季<bean/>元素

请记住，@Configuration类容器最终只是bean定义。在这个例子中，我们创建了一个@Configuration名为类AppConfig，包括其内system-test-config.xml的<bean/>定义。因为 <context:annotation-config/>开机时，容器将识别 @Configuration注释和处理@Bean中声明的方法AppConfig 正确。
```java?linenums
@Configuration
public class AppConfig {

    @Autowired
    private DataSource dataSource;

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }

    @Bean
    public TransferService transferService() {
        return new TransferService(accountRepository());
    }
}
```


系统测试-config.xml文件：
```xml


<beans>
    <!-- enable processing of annotations such as @Autowired and @Configuration -->
    <context:annotation-config/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="com.acme.AppConfig"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>


```
jdbc.properties:
```


jdbc.url = JDBC：HSQLDB：HSQL：//本地主机/ XDB
jdbc.username = SA
jdbc.password =

```
```java?linenums
public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:/com/acme/system-test-config.xml");
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```


在system-test-config.xml上文中，AppConfig <bean/>不声明一个id 元件。虽然这是可以接受的话，没有必要因为没有其他bean将以往任何时候都提到它，这是不可能的，这将是由名容器明确牵强。同样与DataSource豆-它永远只能按类型自动连接，所以一个bean的id要求不严格。
使用<上下文：组分扫描/>拿起@Configuration类

因为@Configuration与间注解@Component，@Configuration-annotated类自动对于组件扫描的候选者。使用相同的方案如上述，我们可以重新定义system-test-config.xml，以利用组件扫描的。请注意，在这种情况下，我们并不需要显式声明 <context:annotation-config/>，因为<context:component-scan/>启用相同的功能。

系统测试-config.xml文件：
```xml


<beans>
    <!-- picks up and registers AppConfig as a bean definition -->
    <context:component-scan base-package="com.acme"/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>

```
@Configuration类为中心的XML的使用与@ImportResource

在应用程序@Configuration类是用于配置容器的主要机制，它仍可能需要至少使用一些XML。在这些情况下，简单地使用@ImportResource和需要只定义尽可能多的XML。这样做实现了“以Java为中心”的方法来配置容器，并保持XML到最低限度。
```java?linenums
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig {

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(url, username, password);
    }
}
```
```
properties-config.xml
<beans>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>
</beans>
```
```
jdbc.properties
jdbc.url = JDBC：HSQLDB：HSQL：//本地主机/ XDB
jdbc.username = SA
jdbc.password =

```

```java?linenums
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

### 1.13. 环境抽象
这Environment 是一个集成在容器中的抽象，它模拟了应用程序环境的两个关键方面：配置文件 和属性。

一个轮廓是bean定义一个命名的逻辑组，只有当指定的配置文件是活动的容器进行登记。可以将Bean分配给配置文件，无论是以XML还是通过注释定义。Environment与配置文件相关的对象的作用是确定哪些配置文件（如果有）当前处于活动状态，以及默认情况下哪些配置文件（如果有）应处于活动状态。

属性在几乎所有应用程序中都发挥着重要作用，并且可能源自各种源：属性文件，JVM系统属性，系统环境变量，JNDI，servlet上下文参数，ad-hoc属性对象，映射等。Environment与属性相关的对象的作用是为用户提供方便的服务接口，用于配置属性源和从中解析属性。
#### 1.13.1. Bean定义配置文件
Bean定义配置文件是核心容器中的一种机制，允许在不同环境中注册不同的bean。单词环境 对不同的用户来说意味着不同的东西，这个功能可以帮助许多用例，包括：

- 在开发中使用内存数据源，在QA或生产环境中查找来自JNDI的相同数据源

- 仅在将应用程序部署到性能环境时注册监视基础结构

- 为客户A和客户B部署注册bean的自定义实现

让我们考虑实际应用中的第一个用例，它需要一个 DataSource。在测试环境中，配置可能如下所示：
```java?linenums
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.HSQL)
        .addScript("my-schema.sql")
        .addScript("my-test-data.sql")
        .build();
}
```
现在让我们考虑如何将此应用程序部署到QA或生产环境中，假设应用程序的数据源将在生产应用程序服务器的JNDI目录中注册。我们的dataSourcebean现在看起来像这样：
```java?linenums
@Bean(destroyMethod="")
public DataSource dataSource() throws Exception {
    Context ctx = new InitialContext();
    return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
}
```
问题是如何根据当前环境在使用这两种变体之间切换。随着时间的推移，Spring用户已经设计了许多方法来完成这项工作，通常依赖于系统环境变量和<import/>包含${placeholder}令牌的XML 语句的组合，这些令牌根据环境变量的值解析为正确的配置文件路径。Bean定义配置文件是核心容器功能，可为此问题提供解决方案。

如果我们概括上面特定于环境的bean定义的示例用例，我们最终需要在某些上下文中注册某些bean定义，而不是在其他上下文中。您可以说您希望在情境A中注册某个bean定义的配置文件，在情况B中注册不同的配置文件。让我们首先看看如何更新我们的配置以反映这种需求。

@profile
该@Profile 注释允许你表明组件有资格登记时的一个或多个指定的简档是活动的。使用上面的示例，我们可以dataSource按如下方式重写配置：

```java?linenums
@Configuration
@Profile("development")
public class StandaloneDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }
}
```
```java?linenums
@Configuration
@Profile("production")
public class JndiDataConfig {

    @Bean(destroyMethod="")
    public DataSource dataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```
```
如前所述，使用@Bean方法，您通常会选择使用编程JNDI查找：使用Spring的JndiTemplate/ JndiLocatorDelegatehelper或InitialContext上面显示的直接JNDI 用法，但不会JndiObjectFactoryBean 强制您将返回类型声明为FactoryBean类型。
```
@Profile可以用作元注释，以创建自定义组合注释。以下示例定义了一个自定义@Production注释，可用作以下内容的 替代 @Profile("production")：
```java?linenums
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Profile("production")
public @interface Production {
}
```
```
如果@Configuration标记了类，则将绕过与该类关联的@Profile所有@Bean方法和 @Import注释，除非一个或多个指定的配置文件处于活动状态。如果a @Component或@Configurationclass被标记@Profile({"p1", "p2"})，则除非已激活配置文件'p1'和/或'p2'，否则不会注册/处理该类。如果给定的配置文件以NOT运算符（!）作为前缀，则如果配置文件未 处于活动状态，则将注册带注释的元素。例如，@Profile({"p1", "!p2"})如果配置文件“p1”处于活动状态或配置文件“p2”未激活，则会发生注册。
```
@Profile 也可以在方法级别声明只包含配置类的一个特定bean，例如，对于特定bean的替代变体：
```java?linenums
@Configuration
public class AppConfig {

    @Bean("dataSource")
    @Profile("development")
    public DataSource standaloneDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }

    @Bean("dataSource")
    @Profile("production")
    public DataSource jndiDataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```
```
使用@Profileon @Bean方法，可能会应用特殊方案：对于@Bean相同Java方法名称的重载方法（类似于构造函数重载），@Profile需要在所有重载方法上一致地声明条件。如果条件不一致，则只有重载方法中第一个声明的条件才重要。@Profile因此，不能用于选择具有特定参数签名的重载方法而不是另一个; 同一个bean的所有工厂方法之间的分辨率遵循Spring的构造函数解析算法在创建时。

如果要定义具有不同配置文件条件的备用Bean，请使用通过@Beanname属性指向同一bean名称的不同Java方法名称，如上例所示。如果参数签名都是相同的（例如，所有变体都具有no-arg工厂方法），这是首先在有效的Java类中表示这种排列的唯一方法（因为只有一种方法可以特定的名称和参数签名）。
```
XML bean定义配置文件
XML对应物是元素的profile属性<beans>。我们上面的示例配置可以在两个XML文件中重写，如下所示：

```xml
<beans profile="development"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xsi:schemaLocation="...">

    <jdbc:embedded-database id="dataSource">
        <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
        <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
    </jdbc:embedded-database>
</beans>
```
```xml
<beans profile="production"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
</beans>
```
也可以避免<beans/>在同一文件中使用split和nest 元素：

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <!-- other bean definitions -->

    <beans profile="development">
        <jdbc:embedded-database id="dataSource">
            <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
            <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
        </jdbc:embedded-database>
    </beans>

    <beans profile="production">
        <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
    </beans>
</beans>
```
在spring-bean.xsd受到了制约，使这些元素只能作为文件中的最后一个人。这应该有助于提供灵活性，而不会在XML文件中引起混乱。

激活个人资料
现在我们已经更新了配置，我们仍然需要指示Spring哪个配置文件处于活动状态。如果我们现在开始我们的示例应用程序，我们会看到NoSuchBeanDefinitionException抛出，因为容器找不到名为的Spring bean dataSource。

激活配置文件可以通过多种方式完成，但最直接的方法是通过以下方式对EnvironmentAPI进行 编程ApplicationContext：
```java?linenums
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("development");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();

```
此外，还可以通过spring.profiles.active属性以声明方式激活配置文件，该 属性可以通过系统环境变量，JVM系统属性，servlet上下文参数web.xml或甚至作为JNDI中的条目来指定（请参阅PropertySource抽象）。在集成测试中，可以通过模块中的@ActiveProfiles注释声明活动配置文件spring-test（请参阅使用环境配置文件的上下文配置）。

请注意，配置文件不是“任何 - 或”命题; 可以一次激活多个配置文件。以编程方式，只需为setActiveProfiles()方法提供多个配置文件名称，该 方法接受String…​varargs：
```java
ctx.getEnvironment().setActiveProfiles("profile1", "profile2");
```
声明性地，spring.profiles.active可以接受以逗号分隔的配置文件名称列表：
```
-Dspring.profiles.active="profile1,profile2"
```
默认配置文件
在默认的配置文件表示默认启用的配置文件。考虑以下：
```java?linenums
@Configuration
@Profile("default")
public class DefaultDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .build();
    }
}
```
如果没有激活配置文件，dataSource将创建上述配置文件; 这可以看作是为一个或多个bean 提供默认定义的一种方法。如果启用了任何配置文件，则默认配置文件将不适用。

默认的配置文件的名称可以使用改变setDefaultProfiles()的Environment或声明使用的spring.profiles.default属性。
#### 1.13.2. PropertySource抽象
Spring的Environment抽象提供了对可配置的属性源层次结构的搜索操作。要完整解释，请考虑以下事项：

```java?linenums
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment();
boolean containsFoo = env.containsProperty("foo");
System.out.println("Does my environment contain the 'foo' property? " + containsFoo);
```
在上面的代码片段中，我们看到了一种向Spring询问是否foo为当前环境定义属性的高级方法。要回答此问题，Environment对象将对一组对象执行搜索PropertySource 。A PropertySource是对任何键值对源的简单抽象，Spring StandardEnvironment 配置有两个PropertySource对象 - 一个表示JVM系统属性集（a la System.getProperties()），另一个表示系统环境变量集（a la System.getenv()）。
```
这些默认属性源StandardEnvironment适用于独立应用程序。StandardServletEnvironment 填充了其他默认属性源，包括servlet配置和servlet上下文参数。它可以选择启用a JndiPropertySource。有关详细信息，请参阅javadocs。
```
具体来说，在使用时StandardEnvironment，env.containsProperty("foo") 如果在运行时存在foo系统属性或foo环境变量，则调用将返回true 。
```
执行的搜索是分层的。默认情况下，系统属性优先于环境变量，因此如果foo在调用期间恰好在两个位置都设置了属性env.getProperty("foo")，则系统属性值将“获胜”并优先于环境变量返回。请注意，属性值不会被合并，而是被前面的条目完全覆盖。

对于公共StandardServletEnvironment层次结构，完整层次结构如下所示，最高优先级条目位于顶部：

ServletConfig参数（如果适用，例如在DispatcherServlet上下文的情况下）

ServletContext参数（web.xml context-param条目）

JNDI环境变量（“java：comp / env /”条目）

JVM系统属性（“-D”命令行参数）

JVM系统环境（操作系统环境变量）
```
最重要的是，整个机制是可配置的。也许您有自定义的属性源，您希望将其集成到此搜索中。没问题 - 只需实现并实例化您自己的PropertySource并将其添加到PropertySources当前的集合中Environment：

```java?linenums
ConfigurableApplicationContext ctx = new GenericApplicationContext();
MutablePropertySources sources = ctx.getEnvironment().getPropertySources();
sources.addFirst(new MyPropertySource());
```
在上面的代码中，MyPropertySource在搜索中添加了最高优先级。如果它包含一个foo属性，它将被检测并foo在任何其他属性之前返回PropertySource。所述 MutablePropertySources API公开了大量的，其允许该组的属性源的精确操作方法。

#### 1.13.3 @PropertySource
该@PropertySource 注解提供便利和声明的机制添加PropertySource 到Spring的Environment。

给定包含键/值对的文件“app.properties” testbean.name=myTestBean，以下@Configuration类@PropertySource以这样的方式使用，即调用testBean.getName()将返回“myTestBean”。
```java?linenums
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```
资源位置中${…​}存在的任何占位符@PropertySource将根据已针对环境注册的属性源集进行解析。例如：

```java?linenums
@Configuration
@PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```
假设“my.placeholder”存在于已注册的其中一个属性源中，例如系统属性或环境变量，则占位符将被解析为相应的值。如果没有，则“default / path”将用作默认值。如果未指定默认值且无法解析属性，IllegalArgumentException则将抛出an 。
```
@PropertySource根据Java 8约定，注释是可重复的。但是，所有这些@PropertySource注释都需要在同一级别声明：直接在配置类上或在同一自定义注释中的元注释。不建议混合直接注释和元注释，因为直接注释将有效地覆盖元注释。
```
#### 1.13.4. 占位符决议在陈述中
从历史上看，元素中占位符的值只能针对JVM系统属性或环境变量进行解析。情况不再如此。因为环境抽象集成在整个容器中，所以很容易通过它来解决占位符的分辨率。这意味着您可以以任何您喜欢的方式配置解析过程：更改搜索系统属性和环境变量的优先级，或者完全删除它们; 根据需要将您自己的属性源添加到混合中。

具体而言，无论customer 属性在何处定义，以下语句都可以工作，只要它在以下位置可用Environment：

```xml
<beans>
    <import resource="com/bank/service/${customer}-config.xml"/>
</beans>
```

### 1.14. 注册LoadTimeWeaver
在LoadTimeWeaver用于由Spring动态变换的类，因为它们被装载到Java虚拟机（JVM）。

要启用加载时编织，请添加@EnableLoadTimeWeaving到其中一个 @Configuration类：
```java?linenums
@Configuration
@EnableLoadTimeWeaving
public class AppConfig {
}
```
或者对于XML配置使用context:load-time-weaver元素：
```xml
<beans>
    <context:load-time-weaver/>
</beans>
```
一旦配置为ApplicationContext。其中的任何bean都ApplicationContext 可以实现LoadTimeWeaverAware，从而接收对load-time weaver实例的引用。这与Spring的JPA支持结合使用特别有用， 其中JPA类转换可能需要加载时编织。有关LocalContainerEntityManagerFactoryBean更多详细信息，请参阅javadocs。有关AspectJ加载时编织的更多信息，请参阅Spring Framework中使用AspectJ进行加载时编织。

### 1.15. ApplicationContext的其他功能
正如章节介绍中所讨论的，该org.springframework.beans.factory 包提供了管理和操作bean的基本功能，包括以编程方式。除了扩展其他接口以外 ，该org.springframework.context软件包还添加了 ApplicationContext扩展BeanFactory接口的接口，以更加面向应用程序框架的方式提供附加功能。许多人ApplicationContext以完全声明的方式使用它，甚至不以编程方式创建它，而是依赖于支持类，例如ContextLoader自动实例化 ApplicationContext作为Java EE Web应用程序的正常启动过程的一部分。

为了BeanFactory以更加面向框架的样式增强功能，上下文包还提供以下功能：

- 通过MessageSource界面访问i18n风格的消息。

- 通过ResourceLoader界面访问 URL和文件等资源。

- 事件发布即ApplicationListener通过使用接口实现接口的bean ApplicationEventPublisher。

- 加载多个（分层）上下文，允许每个上下文通过HierarchicalBeanFactory接口聚焦在一个特定层上，例如应用程序的Web层 。

#### 1.15.1. 使用MessageSource进行国际化
该ApplicationContext接口扩展了一个名为的接口MessageSource，因此提供了国际化（i18n）功能。Spring还提供了接口HierarchicalMessageSource，可以分层次地解析消息。这些接口共同提供了Spring影响消息解析的基础。这些接口上定义的方法包括：

- String getMessage(String code, Object[] args, String default, Locale loc)：用于从中检索消息的基本方法MessageSource。如果未找到指定区域设置的消息，则使用默认消息。传入的任何参数都使用MessageFormat标准库提供的功能成为替换值。

- String getMessage(String code, Object[] args, Locale loc)：与前一个方法基本相同，但有一点不同：不能指定默认消息; 如果找不到该消息，NoSuchMessageException则抛出a。

- String getMessage(MessageSourceResolvable resolvable, Locale locale)：前面方法中使用的所有属性也包装在一个名为的类中 MessageSourceResolvable，您可以使用此方法。

当ApplicationContext被加载时，它自动搜索MessageSource 在上下文中定义的bean。bean必须具有名称messageSource。如果找到这样的bean，则对前面方法的所有调用都被委托给消息源。如果未找到任何消息源，则ApplicationContext尝试查找包含具有相同名称的bean的父级。如果是，它将使用该bean作为MessageSource。如果 ApplicationContext找不到任何消息源，DelegatingMessageSource则实例化为空 以便能够接受对上面定义的方法的调用。

春天提供了两种MessageSource实现方式，ResourceBundleMessageSource和 StaticMessageSource。两者都是HierarchicalMessageSource为了进行嵌套消息传递而实现的。在StaticMessageSource很少使用，但提供了编程的方式向消息源添加消息。在ResourceBundleMessageSource被示出在下面的例子：
```xml
<beans>
    <bean id="messageSource"
            class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>format</value>
                <value>exceptions</value>
                <value>windows</value>
            </list>
        </property>
    </bean>
</beans>
```
在这个例子中，假设你在你的类路径称为定义了三种资源包format，exceptions和windows。任何解决消息的请求都将以JDK标准方式处理，通过ResourceBundles解析消息。出于示例的目的，假设上述两个资源包文件的内容是......
```
# in format.properties
message=Alligators rock!
```
```
# in exceptions.properties
argument.required=The {0} argument is required.
```
MessageSource下一个示例中显示了执行该功能的程序。请记住，所有ApplicationContext实现都是MessageSource 实现，因此可以强制转换为MessageSource接口。

```java?linenums
public static void main(String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("message", null, "Default", null);
    System.out.println(message);
}
```
上述程序产生的结果将是......
```
鳄鱼摇滚！
```
总而言之，它MessageSource是在一个名为的文件中定义的，该文件beans.xml存在于类路径的根目录中。该messageSourcebean定义是指通过它的一些资源包的basenames属性。这是在列表中传递的三个文件basenames属性存在于你的classpath根目录的文件，被称为format.properties，exceptions.properties和 windows.properties分别。

下一个示例显示传递给消息查找的参数; 这些参数将转换为字符串并插入到查找消息中的占位符中。
```xml
<beans>

    <!-- this MessageSource is being used in a web application -->
    <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basename" value="exceptions"/>
    </bean>

    <!-- lets inject the above MessageSource into this POJO -->
    <bean id="example" class="com.foo.Example">
        <property name="messages" ref="messageSource"/>
    </bean>

</beans>
```
```java?linenums
public class Example {

    private MessageSource messages;

    public void setMessages(MessageSource messages) {
        this.messages = messages;
    }

    public void execute() {
        String message = this.messages.getMessage("argument.required",
            new Object [] {"userDao"}, "Required", null);
        System.out.println(message);
    }
}
```
调用该execute()方法产生的结果将是......
```
userDao参数是必需的。
```
关于国际化（i18n），Spring的各种MessageSource 实现遵循与标准JDK相同的区域设置解析和回退规则 ResourceBundle。总之，和继续该示例messageSource先前定义的，如果你想解析British（消息en-GB）语言环境中，您将创建文件名为format_en_GB.properties，exceptions_en_GB.properties和 windows_en_GB.properties分别。

通常，区域设置解析由应用程序的周围环境管理。在此示例中，将手动指定将解析（英国）消息的区域设置。
```
＃在exceptions_en_GB.properties中
argument.required = Ebagum小伙子，我说，{0}参数是必需的。
```
```java?linenums
public static void main(final String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("argument.required",
        new Object [] {"userDao"}, "Required", Locale.UK);
    System.out.println(message);
}
```
运行上述程序产生的结果将是......
```
Ebagum小伙子，我说，'userDao'论证是必需的。
```
您还可以使用该MessageSourceAware界面获取对MessageSource已定义的任何内容的引用 。在创建和配置bean时，将使用应用程序上下文注入ApplicationContext实现MessageSourceAware接口的任何bean MessageSource。
```
作为替代ResourceBundleMessageSource，Spring提供了一个 ReloadableResourceBundleMessageSource类。此变体支持相同的包文件格式，但比基于标准JDK的ResourceBundleMessageSource实现更灵活 。特别是，它允许从任何Spring资源位置（不仅仅是从类路径）读取文件，并支持bundle属性文件的热重新加载（同时在它们之间有效地缓存它们）。查看ReloadableResourceBundleMessageSourcejavadocs了解详细信息。
```
#### 1.15.2. 标准和自定义活动
ApplicationContext通过ApplicationEvent 类和ApplicationListener接口提供事件处理。如果将实现ApplicationListener接口的bean 部署到上下文中，则每次 ApplicationEvent将其发布到该ApplicationContextbean时，都会通知该bean。从本质上讲，这是标准的Observer设计模式。
```
从Spring 4.2开始，事件基础结构得到了显着改进，并提供了基于注释的模型以及发布任意事件的能力，这是一个不一定从中扩展的对象ApplicationEvent。当这样的对象发布时，我们将它包装在一个事件中。
```
Spring提供以下标准事件：

表7.内置事件
您还可以创建和发布自己的自定义事件。这个例子展示了一个扩展Spring的ApplicationEvent基类的简单
```java?linenums
public class BlackListEvent extends ApplicationEvent {

    private final String address;
    private final String test;

    public BlackListEvent(Object source, String address, String test) {
        super(source);
        this.address = address;
        this.test = test;
    }

    // accessor and other methods...
}
```
要发布自定义ApplicationEvent，请在publishEvent()方法上调用该方法 ApplicationEventPublisher。通常，这是通过创建一个实现ApplicationEventPublisherAware并将其注册为Spring bean 的类来完成的 。以下示例演示了这样一个类：

```java?linenums
public class EmailService implements ApplicationEventPublisherAware {

    private List<String> blackList;
    private ApplicationEventPublisher publisher;

    public void setBlackList(List<String> blackList) {
        this.blackList = blackList;
    }

    public void setApplicationEventPublisher(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    public void sendEmail(String address, String text) {
        if (blackList.contains(address)) {
            BlackListEvent event = new BlackListEvent(this, address, text);
            publisher.publishEvent(event);
            return;
        }
        // send email...
    }
}
```
在配置时，Spring容器将检测到该EmailService实现 ApplicationEventPublisherAware并将自动调用 setApplicationEventPublisher()。实际上，传入的参数将是Spring容器本身; 您只需通过其ApplicationEventPublisher界面与应用程序上下文进行 交互。

要接收自定义ApplicationEvent，请创建一个实现 ApplicationListener并将其注册为Spring bean的类。以下示例演示了这样一个类：
```java?linenums
public class BlackListNotifier implements ApplicationListener<BlackListEvent> {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    public void onApplicationEvent(BlackListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```
请注意，ApplicationListener通常使用自定义事件的类型进行参数化BlackListEvent。这意味着该onApplicationEvent()方法可以保持类型安全，避免任何向下转换的需要。您可以根据需要注册任意数量的事件侦听器，但请注意，默认情况下，事件侦听器会同步接收事件。这意味着该publishEvent()方法将阻塞，直到所有侦听器都已完成对事件的处理。这种同步和单线程方法的一个优点是，当侦听器接收到事件时，如果事务上下文可用，它将在发布者的事务上下文内运行。如果需要另一个事件发布策略，请参阅Spring ApplicationEventMulticaster接口的javadoc 。

以下示例显示了用于注册和配置上述每个类的bean定义：
```xml
<bean id="emailService" class="example.EmailService">
    <property name="blackList">
        <list>
            <value>known.spammer@example.org</value>
            <value>known.hacker@example.org</value>
            <value>john.doe@example.org</value>
        </list>
    </property>
</bean>

<bean id="blackListNotifier" class="example.BlackListNotifier">
    <property name="notificationAddress" value="blacklist@example.org"/>
</bean>
```
总而言之，当调用bean 的sendEmail()方法时emailService，如果有任何应该被列入黑名单的电子邮件，BlackListEvent则会发布类型的自定义事件 。所述blackListNotifier豆被注册为一个 ApplicationListener，从而接收到BlackListEvent，在该点它可以通知适当方。
```
Spring的事件机制是为在同一应用程序上下文中的Spring bean之间的简单通信而设计的。但是，对于更复杂的企业集成需求，单独维护的 Spring Integration项目为构建基于众所周知的Spring编程模型的轻量级，面向模式，事件驱动的体系结构提供了完整的支持 。
```
基于注释的事件侦听器
从Spring 4.2开始，可以通过EventListener注释在托管bean的任何公共方法上注册事件侦听器。该BlackListNotifier可改写如下：
```java?linenums
public class BlackListNotifier {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    @EventListener
    public void processBlackListEvent(BlackListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```
如您所见，方法签名再次声明它侦听的事件类型，但这次使用灵活的名称并且没有实现特定的侦听器接口。只要实际事件类型在其实现层次结构中解析通用参数，也可以通过泛型缩小事件类型。

如果您的方法应该监听多个事件，或者您想要根据任何参数进行定义，那么也可以在注释本身上指定事件类型：

```java?linenums
@EventListener({ContextStartedEvent.class, ContextRefreshedEvent.class})
public void handleContextStart() {
    ...
}
```
还可以通过condition注释的属性添加额外的运行时过滤，该注释定义应该匹配的SpEL表达式以实际调用特定事件的方法。

例如，如果test事件的属性等于foo：我们的通知程序可以被重写为仅被调用：
```java?linenums
@EventListener(condition = "#blEvent.test == 'foo'")
public void processBlackListEvent(BlackListEvent blEvent) {
    // notify appropriate parties via notificationAddress...
}
```
每个SpEL表达式再次评估专用上下文。下一个表列出了可用于上下文的项目，因此可以将它们用于条件事件处理：

表8.事件SpEL可用元数据

请注意#root.event，即使您的方法签名实际引用已发布的任意对象，也允许您访问基础事件。

如果您需要发布一个事件作为处理另一个事件的结果，只需更改方法签名以返回应该发布的事件，例如
```java?linenums
@EventListener
public ListUpdateEvent handleBlackListEvent(BlackListEvent event) {
    // notify appropriate parties via notificationAddress and
    // then publish a ListUpdateEvent...
}
```
异步侦听 器不支持此功能。
这个新方法将为上述方法处理的ListUpdateEvent每个方法发布一个新BlackListEvent的方法。如果您需要发布多个事件，请返回一个Collection事件。

异步监听器
如果您希望特定侦听器异步处理事件，只需重用 常规@Async支持：
```java?linenums
@EventListener
@Async
public void processBlackListEvent(BlackListEvent event) {
    // BlackListEvent is processed in a separate thread
}
```
使用异步事件时请注意以下限制：

如果事件侦听器抛出Exception它将不会传播给调用者，请检查AsyncUncaughtExceptionHandler更多详细信息。

此类事件监听器无法发送回复。如果您需要作为处理结果发送另一个事件，请注入ApplicationEventPublisher以手动发送事件。

订购听众
如果需要在另一个侦听器之前调用侦听器，只需将@Order 注释添加到方法声明中：
```java?linenums
@EventListener
@Order(42)
public void processBlackListEvent(BlackListEvent event) {
    // notify appropriate parties via notificationAddress...
}
```
通用事件
您还可以使用泛型来进一步定义事件的结构。考虑在 EntityCreatedEvent<T>哪里T创建实际实体的类型。您可以创建以下侦听器定义只接收EntityCreatedEvent了 Person：

```java?linenums
@EventListener
public void onPersonCreated(EntityCreatedEvent<Person> event) {
    ...
}
```
由于类型擦除，这仅在被触发的事件解析事件侦听器在其上过滤的泛型参数（类似于class PersonCreatedEvent extends EntityCreatedEvent<Person> { …​ }）时才有效 。

在某些情况下，如果所有事件都遵循相同的结构（这应该是上述事件的情况），这可能会变得相当繁琐。在这种情况下，可以实现ResolvableTypeProvider对引导超出原来的运行环境提供了框架：
```java?linenums
public class EntityCreatedEvent<T>
        extends ApplicationEvent implements ResolvableTypeProvider {

    public EntityCreatedEvent(T entity) {
        super(entity);
    }

    @Override
    public ResolvableType getResolvableType() {
        return ResolvableType.forClassWithGenerics(getClass(),
                ResolvableType.forInstance(getSource()));
    }
}
```
这不仅适用于ApplicationEvent您作为事件发送的任何对象。
#### 1.15.3. 方便地访问低级资源
为了最佳地使用和理解应用程序上下文，用户通常应该熟悉Spring的Resource抽象，如资源一章所述 。

应用程序上下文是a ResourceLoader，可用于加载Resources。A Resource本质上是一个功能更丰富的JDK类版本java.net.URL，实际上，在适当Resource的java.net.URL地方包装一个实例的实现。A Resource可以透明方式从几乎任何位置获取低级资源，包括类路径，文件系统位置，可用标准URL描述的任何位置，以及一些其他变体。如果资源位置字符串是没有任何特殊前缀的简单路径，那么这些资源来自特定且适合于实际应用程序上下文类型。

您可以配置部署到应用程序上下文中的bean来实现特殊的回调接口，ResourceLoaderAware在初始化时自动回调，应用程序上下文本身作为传入 ResourceLoader。您还可以公开Resource用于访问静态资源的类型属性; 它们将像任何其他属性一样注入其中。您可以将这些Resource属性指定为简单的String路径，并依赖于PropertyEditor上下文自动注册的特殊JavaBean ，以便Resource在部署Bean时将这些文本字符串转换为实际对象。

提供给ApplicationContext构造函数的位置路径实际上是资源字符串，并且以简单的形式适当地处理特定的上下文实现。ClassPathXmlApplicationContext将简单的位置路径视为类路径位置。您还可以使用具有特殊前缀的位置路径（资源字符串）来强制从类路径或URL加载定义，而不管实际的上下文类型如何。
#### 1.15.4. 方便的Web应用程序的ApplicationContext实例化
您可以ApplicationContext使用例如a以声明方式创建实例 ContextLoader。当然，您也可以ApplicationContext使用其中一个ApplicationContext实现以编程方式创建实例。

您可以ApplicationContext使用ContextLoaderListener以下注册：
```xml

<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/daoContext.xml /WEB-INF/applicationContext.xml</param-value>
</context-param>

<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```
监听器检查contextConfigLocation参数。如果参数不存在，则侦听器将/WEB-INF/applicationContext.xml默认使用。当参数确实存在时，侦听器使用预定义的分隔符（逗号，分号和空格）分隔String，并将值用作将搜索应用程序上下文的位置。还支持Ant样式的路径模式。示例/WEB-INF/*Context.xml适用于名称以“Context.xml”结尾的所有文件，驻留在“WEB-INF”目录中，以及/WEB-INF/**/*Context.xml“WEB-INF”的任何子目录中的所有此类文件。

#### 1.15.5. 将Spring ApplicationContext部署为Java EE RAR文件

可以将Spring ApplicationContext部署为RAR文件，将上下文及其所有必需的bean类和库JAR封装在Java EE RAR部署单元中。这相当于引导一个独立的ApplicationContext，它只是托管在Java EE环境中，能够访问Java EE服务器设施。RAR部署是部署无头WAR文件的场景的更自然的替代方案，实际上是没有任何HTTP入口点的WAR文件，仅用于在Java EE环境中引导Spring ApplicationContext。

RAR部署非常适用于不需要HTTP入口点但仅包含消息端点和预定作业的应用程序上下文。在这样的上下文中的Bean可以使用应用程序服务器资源，例如JTA事务管理器和JNDI绑定的JDBC DataSources和JMS ConnectionFactory实例，也可以通过Spring的标准事务管理以及JNDI和JMX支持工具向平台的JMX服务器注册。应用程序组件还可以通过Spring的TaskExecutor抽象与应用程序服务器的JCA WorkManager交互。

查看SpringContextResourceAdapter 类的javadoc以 获取RAR部署中涉及的配置详细信息。

对于将Spring ApplicationContext简单部署为Java EE RAR文件：将所有应用程序类打包到RAR文件中，该文件是具有不同文件扩展名的标准JAR文件。将所有必需的库JAR添加到RAR存档的根目录中。添加“META-INF / ra.xml”部署描述符（如SpringContextResourceAdapters javadoc 所示）和相应的Spring XML bean定义文件（通常为“META-INF / applicationContext.xml”），并删除生成的RAR文件进入应用程序服务器的部署目录。


```
这种RAR部署单元通常是独立的; 它们不会将组件暴露给外部世界，甚至不会暴露给同一应用程序的其他模块。与基于RAR的ApplicationContext的交互通常通过与其他模块共享的JMS目标进行。例如，基于RAR的ApplicationContext还可以调度一些作业，对文件系统中的新文件（或类似物）作出反应。如果它需要允许来自外部的同步访问，它可以例如导出RMI端点，当然可以由同一机器上的其他应用程序模块使用。
```

### 1.16. BeanFactory
它BeanFactory提供了Spring的IoC功能的底层基础，但它只是直接用于与其他第三方框架的集成，现在对于大多数Spring用户来说现在基本上是历史性的。在BeanFactory与相关的接口，如BeanFactoryAware，InitializingBean，DisposableBean，仍然存在于Spring为向后兼容性与大量与Spring集成第三方框架的目的。通常是第三方组件，它们不能使用更现代的等价物，@PostConstruct或者@PreDestroy为了避免依赖JSR-250。

本节提供了额外的背景入之间的差异 BeanFactory以及ApplicationContext如何一个可以直接通过经典单查询访问IoC容器。
#### 1.16.1. BeanFactory或ApplicationContext？
使用，ApplicationContext除非你有充分的理由不这样做。

因为ApplicationContext包含了BeanFactory它的所有功能，所以通常建议使用它BeanFactory，除了少数情况，例如在资源受限的设备上运行的嵌入式应用程序中，内存消耗可能是关键的，而一些额外的千字节可能会产生影响。但是，对于大多数典型的企业应用程序和系统，ApplicationContext您将要使用它。Spring 大量使用BeanPostProcessor扩展点（以实现代理等）。如果您只使用普通货物BeanFactory，交易和AOP等相当数量的支持将不会生效，至少在您没有采取额外措施的情况下也是如此。这种情况可能会令人困惑，因为配置实际上没有任何问题。

下表列出了提供的功能BeanFactory和 ApplicationContext接口和实现。

表9.特征矩阵

要使用实现显式注册bean后处理器BeanFactory，您需要编写如下代码：
```java?linenums
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
// populate the factory with bean definitions

// now register any needed BeanPostProcessor instances
MyBeanPostProcessor postProcessor = new MyBeanPostProcessor();
factory.addBeanPostProcessor(postProcessor);

// now start using the factory

```
要BeanFactoryPostProcessor在使用BeanFactory 实现时显式注册，必须编写如下代码：
```java?linenums
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
reader.loadBeanDefinitions(new FileSystemResource("beans.xml"));

// bring in some property values from a Properties file
PropertyPlaceholderConfigurer cfg = new PropertyPlaceholderConfigurer();
cfg.setLocation(new FileSystemResource("jdbc.properties"));

// now actually do the replacement
cfg.postProcessBeanFactory(factory);
```
在这两种情况下，显式注册步骤都不方便，这是为什么各种 实现在绝大多数Spring支持的应用程序ApplicationContext中优先于普通BeanFactory实现的一个原因，特别是在使用BeanFactoryPostProcessors和BeanPostProcessors时。这些机制实现了重要的功能，例如属性占位符替换和AOP。

## 2. 资源
### 2.1. 介绍
java.net.URL遗憾的是，Java的各种URL前缀的标准类和标准处理程序不足以完全访问低级资源。例如，没有标准化的URL实现可用于访问需要从类路径或相对于a获取的资源 ServletContext。虽然可以为专用URL 前缀注册新的处理程序（类似于现有的前缀处理程序http:），但这通常非常复杂，并且URL接口仍然缺少一些理想的功能，例如检查资源是否存在的方法指着。
### 2.2. 资源界面
Spring的Resource接口是一个更强大的接口，用于抽象对低级资源的访问。
```java?linenums
public interface Resource extends InputStreamSource {

    boolean exists();

    boolean isOpen();

    URL getURL() throws IOException;

    File getFile() throws IOException;

    Resource createRelative(String relativePath) throws IOException;

    String getFilename();

    String getDescription();

}
```
```java?linenums
public interface InputStreamSource {

    InputStream getInputStream() throws IOException;

}
```
Resource界面中一些最重要的方法是：

- getInputStream()：找到并打开资源，返回InputStream从资源中读取的内容。预计每次调用都会返回一个新的 InputStream。呼叫者有责任关闭流。

- exists()：返回一个boolean指示此资源是否实际存在的物理形式。

- isOpen()：返回一个boolean指示此资源是否表示具有打开流的句柄的a。如果true，InputStream不能多次读取，必须只读一次然后关闭以避免资源泄漏。将false用于所有常规资源实现，但不包括InputStreamResource。

- getDescription()：返回此资源的描述，用于处理资源时的错误输出。这通常是完全限定的文件名或资源的实际URL。

其他方法允许您获取表示资源的实际URL或File对象（如果底层实现兼容，并支持该功能）。

该Resource抽象需要资源时Spring自身广泛使用，在许多方法签名的参数类型。某些Spring API中的其他方法（例如各种ApplicationContext实现的构造函数）采用以 String简单或简单的形式创建Resource适合于该上下文实现的方法，或者通过String路径上的特殊前缀，允许调用者指定Resource必须创建和使用特定的实现。

虽然ResourceSpring和Spring都使用了很多接口，但是在你自己的代码中使用它作为通用实用程序类非常有用，用于访问资源，即使你的代码不知道或不关心任何其他部分春天 虽然这会将您的代码耦合到Spring，但它实际上只将它耦合到这一小组实用程序类，这些实用程序类作为更有能力的替代品URL，并且可以被认为等同于您将用于此目的的任何其他库。

重要的是要注意Resource抽象不会取代功能：它尽可能地包装它。例如，一个UrlResource包装URL，并使用包装URL来完成它的工作。

### 2.3. 内置资源实现
有许多Resource实现在Spring中直接提供：
#### 2.3.1. UrlResource对象
所述UrlResource包裹一个java.net.URL，并且可以被用于访问任何对象，该对象是通过URL正常访问，如文件，一个HTTP靶，FTP对象等的所有URL具有标准化的String表示，以使得适当的标准化的前缀被用来指示另一个URL类型。这包括file:访问文件系统路径，http:通过HTTP协议 ftp:访问资源，通过FTP访问资源等。

A UrlResource由Java代码使用UrlResource构造函数显式创建，但通常在调用API方法时隐式创建，该方法接受一个String 表示路径的参数。对于后一种情况，JavaBeans PropertyEditor最终将决定Resource要创建哪种类型。如果路径字符串包含一些众所周知的（对它来说）前缀，例如classpath:，它将Resource为该前缀创建一个合适的专用。但是，如果它不识别前缀，它将假设这只是一个标准的URL字符串，并将创建一个UrlResource。
#### 2.3.2. 使用ClassPathResource
此类表示应从类路径获取的资源。这使用线程上下文类加载器，给定的类加载器或给定的类来加载资源。

此Resource实现支持解决方案，就java.io.File好像类路径资源驻留在文件系统中一样，但不支持驻留在jar中且尚未（通过servlet引擎或任何环境）扩展到文件系统的类路径资源。为了解决这个问题，各种Resource实现总是支持解决方案java.net.URL。

A ClassPathResource由Java代码使用ClassPathResource 构造函数显式创建，但通常在调用API方法时隐式创建，该方法接受一个String表示路径的参数。对于后一种情况，JavaBeans PropertyEditor将识别classpath:字符串路径上的特殊前缀，并ClassPathResource在此情况下创建。
#### 2.3.3. 的FileSystemResource
这是句柄的Resource实现java.io.File。它显然支持解决方案作为一个File和一个URL。
#### 2.3.4. ServletContextResource
这是资源的Resource实现ServletContext，解释相关Web应用程序根目录中的相对路径。

这始终支持流访问和URL访问，但仅允许java.io.File在扩展Web应用程序归档并且资源实际位于文件系统上时进行访问。它是否在这样的文件系统上展开，或直接从JAR或其他地方（如DB）（可以想象）访问，实际上是依赖于Servlet容器。
#### 2.3.5. 的InputStreamResource
Resource给定的实现InputStream。只有在没有Resource适用的具体实施时才应使用此选项。特别地，在可能ByteArrayResource的Resource情况下，优选 或任何基于文件的实现。

相对于其他Resource的实现，这是一个描述符已经 打开资源-因此返回true的isOpen()。如果需要将资源描述符保留在某处，或者需要多次读取流，请不要使用它。
#### 2.3.6. 使用ByteArrayResource
这是Resource给定字节数组的实现。它ByteArrayInputStream为给定的字节数组创建一个 。

它对于从任何给定的字节数组加载内容非常有用，而不必求助于单次使用InputStreamResource。
### 2.4. ResourceLoader
该ResourceLoader接口旨在由可以返回（即加载）Resource实例的对象实现。
```java?linenums
public interface ResourceLoader {

    Resource getResource(String location);

}
```
所有应用程序上下文都实现了ResourceLoader接口，因此可以使用所有应用程序上下文来获取Resource实例。

当您调用getResource()特定的应用程序上下文，并且指定的位置路径没有特定的前缀时，您将返回一个Resource适合该特定应用程序上下文的类型。例如，假设针对ClassPathXmlApplicationContext实例执行了以下代码片段：
```java?linenums
Resource template = ctx.getResource("some/resource/path/myTemplate.txt");
```
将返回的是一个ClassPathResource; 如果对一个FileSystemXmlApplicationContext实例执行相同的方法，你会得到一个 FileSystemResource。对于a WebApplicationContext，你会得到一个 ServletContextResource，等等。

因此，您可以以适合特定应用程序上下文的方式加载资源。

另一方面，您也ClassPathResource可以通过指定特殊classpath:前缀强制使用，而不管应用程序上下文类型如何：
```java?linenums
Resource template = ctx.getResource("classpath:some/resource/path/myTemplate.txt");

```
类似地，可以UrlResource通过指定任何标准 java.net.URL前缀来强制使用a ：
```java?linenums
Resource template = ctx.getResource("file:///some/resource/path/myTemplate.txt");
```
```java?linenums
Resource template = ctx.getResource("http://myhost.com/resource/path/myTemplate.txt");
```
下表总结了将Strings 转换为Resources 的策略：

表10.资源字符串

### 2.5. ResourceLoaderAware接口
该ResourceLoaderAware接口是一个特殊的标记接口，它希望被提供有对象ResourceLoader参考。
```java?linenums
public interface ResourceLoaderAware {

    void setResourceLoader(ResourceLoader resourceLoader);
}
```
当一个类实现ResourceLoaderAware并部署到应用程序上下文中时（作为Spring管理的bean），它被ResourceLoaderAware应用程序上下文识别。然后，应用程序上下文将调用 setResourceLoader(ResourceLoader)，将自身作为参数提供（请记住，Spring中的所有应用程序上下文都实现了ResourceLoader接口）。

当然，由于a ApplicationContext是a ResourceLoader，bean也可以实现ApplicationContextAware接口并直接使用提供的应用程序上下文来加载资源，但一般情况下，ResourceLoader如果需要的话，最好使用专用 接口。代码只会耦合到资源加载接口，可以将其视为实用程序接口，而不是整个Spring ApplicationContext接口。

从Spring 2.5开始，您可以依靠自动装配ResourceLoader作为实现ResourceLoaderAware接口的替代方案。“传统” constructor和 byType自动装配模式（如自动装配协作者中所述）现在能够分别为ResourceLoader构造函数参数或setter方法参数提供类型的依赖性。为了获得更大的灵活性（包括自动装配字段和多参数方法的能力），请考虑使用新的基于注释的自动装配功能。在这种情况下，只要有问题的字段，构造函数或方法带有 注释，ResourceLoader就会将自动装入一个字段，构造函数参数或方法参数，这些参数期望该ResourceLoader类型@Autowired。有关更多信息，请参阅@Autowired。
### 2.6. 资源作为依赖项
如果bean本身将通过某种动态过程确定并提供资源路径，那么bean使用ResourceLoader 接口加载资源可能是有意义的。以某种模板的加载为例，其中所需的特定资源取决于用户的角色。如果资源是静态的，那么ResourceLoader 完全消除接口的使用是有意义的，只需让bean公开Resource它需要的属性，并期望它们被注入其中。

然后注入这些属性变得微不足道的是，所有应用程序上下文都注册并使用PropertyEditor可以将String路径转换为Resource对象的特殊JavaBean 。因此，如果myBean具有类型的模板属性Resource，则可以使用该资源的简单字符串进行配置，如下所示：
```xml
<bean id="myBean" class="...">
    <property name="template" value="some/resource/path/myTemplate.txt"/>
</bean>
```
请注意，资源路径没有前缀，因为应用程序上下文本身将用作ResourceLoader，资源本身将通过，或（根据情况）加载 ClassPathResource，具体取决于上下文的确切类型。FileSystemResourceServletContextResource

如果需要强制使用特定Resource类型，则可以使用前缀。以下两个示例显示了如何强制a ClassPathResource和a UrlResource（后者用于访问文件系统文件）。
```xml
<property name="template" value="classpath:some/resource/path/myTemplate.txt">
```
```xml
<property name="template" value="file:///some/resource/path/myTemplate.txt"/>
```

### 2.7. 应用程序上下文和资源路径
#### 2.7.1. 构建应用程序上下文
应用程序上下文构造函数（对于特定的应用程序上下文类型）通常将字符串或字符串数​​组作为资源的位置路径（例如构成上下文定义的XML文件）。

当这样的位置路径没有前缀时，Resource从该路径构建并用于加载bean定义的特定类型取决于并且适合于特定的应用程序上下文。例如，如果您创建 ClassPathXmlApplicationContext如下：
```java?linenums
ApplicationContext ctx = new ClassPathXmlApplicationContext("conf/appContext.xml");
```
bean定义将从类路径加载，ClassPathResource将被使用。但是如果你创建FileSystemXmlApplicationContext如下：
```java?linenums
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("conf/appContext.xml");
```
bean定义将从文件系统位置加载，在这种情况下相对于当前工作目录。

请注意，在位置路径上使用特殊类路径前缀或标准URL前缀将覆盖Resource为加载定义而创建的默认类型。所以这FileSystemXmlApplicationContext......
```java?linenums
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("classpath:conf/appContext.xml");
```
实际上将从类路径加载其bean定义。但是，它仍然是一个 FileSystemXmlApplicationContext。如果它随后用作a ResourceLoader，则任何未加前缀的路径仍将被视为文件系统路径。

构造ClassPathXmlApplicationContext实例 - 快捷方式
在ClassPathXmlApplicationContext提供了多种构造方法以便于实例。的基本思想是，提供一个仅包含XML的只是文件名的字符串数组文件本身（无前导路径信息），以及一个也供给Class; 在ClassPathXmlApplicationContext 将从给定类的路径信息。

一个例子有望清楚地表明这一点。考虑一个如下所示的目录布局：
```
com/
  foo/
    services.xml
    daos.xml
    MessengerService.class
```
一个ClassPathXmlApplicationContext在规定的豆组成的实例 'services.xml'，并'daos.xml'可以被实例化是这样的...
```java?linenums
ApplicationContext ctx = new ClassPathXmlApplicationContext(
    new String[] {"services.xml", "daos.xml"}, MessengerService.class);
```
ClassPathXmlApplicationContext有关各种构造函数的详细信息，请参阅javadocs。
#### 2.7.2. 应用程序上下文构造函数资源路径中的通配符
应用程序上下文构造函数值中的资源路径可以是一个简单的路径（如上所示），它具有与目标资源的一对一映射，或者可以包含特殊的“classpath *：”前缀和/或内部Ant-样式正则表达式（使用Spring的PathMatcher实用程序匹配）。后者都是有效的通配符

此机制的一个用途是进行组件样式的应用程序组装。所有组件都可以将上下文定义片段“发布”到一个众所周知的位置路径，并且当使用前缀为via的相同路径创建最终应用程序上下文时 classpath*:，将自动拾取所有组件片段。

请注意，此通配符特定于在应用程序上下文构造函数中使用资源路径（或PathMatcher直接使用实用程序类层次结构时），并在构造时解析。它与Resource类型本身无关。使用classpath*:前缀来构造实际Resource的资源是不可能的，因为资源一次只指向一个资源。

蚂蚁风格的图案
当路径位置包含Ant样式模式时，例如：
```
/WEB-INF/*-context.xml
com/mycompany/**/applicationContext.xml
file:C:/some/path/*-context.xml
classpath:com/mycompany/**/applicationContext.xml
```
解析器遵循更复杂但定义的过程来尝试解析通配符。它为直到最后一个非通配符段的路径生成一个Resource，并从中获取一个URL。如果此URL不是jar:URL或特定zip:于容器的变体（例如，在WebLogic中，wsjar在WebSphere中等），则java.io.File从中获取a 并用于通过遍历文件系统来解析通配符。对于jar URL，解析器要么从中获取java.net.JarURLConnection，要么手动解析jar URL，然后遍历jar文件的内容以解析通配符。

对可移植性的影响
如果指定的路径已经是文件URL（显式或隐式，因为基础ResourceLoader是文件系统），那么通配符保证以完全可移植的方式工作。

如果指定的路径是类路径位置，则解析程序必须通过Classloader.getResource()调用获取最后一个非通配符路径段URL 。由于这只是路径的一个节点（不是最后的文件），因此实际上未定义（在 ClassLoaderjavadocs中）在这种情况下返回的URL究竟是什么类型。实际上，它总是java.io.File表示目录，类路径资源解析为文件系统位置，或某种类型的jar URL，其中类路径资源解析为jar位置。尽管如此，这种操作还是存在可移植性问题。

如果为最后一个非通配符段获取了jar URL，则解析器必须能够从中获取java.net.JarURLConnection，或者手动解析jar URL，以便能够遍历jar的内容，并解析通配符。这适用于大多数环境，但在其他环境中会失败，强烈建议在依赖它之前，在特定环境中对来自jar的资源的通配符解析进行全面测试。

classpath *：前缀
构建基于XML的应用程序上下文时，位置字符串可以使用特殊classpath*:前缀：
```java?linenums
ApplicationContext ctx =
    new ClassPathXmlApplicationContext("classpath*:conf/appContext.xml");

```
此特殊前缀指定必须获取与给定名称匹配的所有类路径资源（内部，这通常通过ClassLoader.getResources(…​)调用发生 ），然后合并以形成最终的应用程序上下文定义。
```
通配符类路径依赖于getResources()底层类加载器的方法。由于现在大多数应用程序服务器都提供了自己的类加载器实现，因此行为可能会有所不同，尤其是在处理jar文件时。检查是否classpath*有效的简单测试是使用类加载器从类路径中的jar中加载文件： getClass().getClassLoader().getResources("<someFileInsideTheJar>")。尝试使用具有相同名称但放在两个不同位置的文件进行此测试。如果返回了不适当的结果，请检查应用程序服务器文档以获取可能影响类加载器行为的设置。
```
例如，classpath*:前缀也可以与PathMatcher位置路径的其余部分中的模式组合classpath*:META-INF/*-beans.xml。在这种情况下，解析策略非常简单：ClassLoader.getResources()在最后一个非通配符路径段上使用调用来获取类加载器层次结构中的所有匹配资源，然后关闭每个资源，使用上述相同的PathMatcher解析策略通配符子路径。

与通配符有关的其他说明
请注意，classpath*:与Ant样式组合使用时，只有在模式启动之前，至少有一个根目录才能可靠地工作，除非实际目标文件驻留在文件系统中。这意味着类似的模式 classpath*:*.xml可能无法从jar文件的根目录中检索文件，而只能从扩展目录的根目录中检索文件。

Spring检索类路径条目的能力来自JDK的 ClassLoader.getResources()方法，该方法仅返回传入的空字符串的文件系统位置（指示搜索的潜在根）。Spring还评估 URLClassLoader运行时配置和jar文件中的“java.class.path”清单，但不保证这会导致可移植行为。
```
扫描类路径包需要在类路径中存在相应的目录条目。使用Ant构建JAR时，请确保不要 激活JAR任务的仅文件开关。此外，在某些环境中，类路径目录可能不会基于安全策略公开，例如JDK 1.7.0_45及更高版本上的独立应用程序（需要在清单中设置“Trusted-Library”;请参阅 http://stackoverflow.com/questions/ 19394570 / java-jre-7u45-breaks-classloader-getresources）。

在JDK 9的模块路径（Jigsaw）上，Spring的类路径扫描通常按预期工作。此处强烈建议将资源放入专用目录，避免上述搜索jar文件根级别的可移植性问题。

```
classpath:如果要搜索的根包在多个类路径位置中可用，则不保证具有资源的Ant样式模式可以找到匹配的资源。这是因为资源如
```
com/mycompany/package1/service-context.xml
```
可能只在一个位置，但当一个路径，如
```
classpath:com/mycompany/**/service-context.xml
```
用于尝试解决它，解析器将处理由getResource("com/mycompany"); 返回的（第一个）URL 。如果此基本包节点存在于多个类加载器位置中，则实际的最终资源可能不在下面。因此，最好在这种情况下使用具有相同Ant样式模式的“`classpath *：`”，它将搜索包含根包的所有类路径位置。
#### 2.7.3. FileSystemResource需要注意

一个FileSystemResource未连接到FileSystemApplicationContext（即FileSystemApplicationContext不实际的ResourceLoader）将把绝对和相对路径如你所愿。相对路径相对于当前工作目录，而绝对路径相对于文件系统的根目录。

为了向后兼容（历史）的原因然而，这改变时 FileSystemApplicationContext是ResourceLoader。在 FileSystemApplicationContext简单地让所有绑定的FileSystemResource情况下，把所有的位置路径为相对的，他们是否开始与斜线与否。实际上，这意味着以下内容是等效的：
```java?linenums
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("conf/context.xml");

```
```java?linenums
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("/conf/context.xml");
```
如下所示:(即使它们有所不同，因为一个案例是相对的而另一个案例是绝对的。）
```java?linenums
FileSystemXmlApplicationContext ctx = ...;
ctx.getResource("some/resource/path/myTemplate.txt");
```
```java?linenums
FileSystemXmlApplicationContext ctx = ...;
ctx.getResource("/some/resource/path/myTemplate.txt");
```
实际上，如果需要真正的绝对文件系统路径，最好放弃使用FileSystemResource/ 的绝对路径FileSystemXmlApplicationContext，并且只UrlResource使用file:URL前缀强制使用a 。
```java?linenums
// actual context type doesn't matter, the Resource will always be UrlResource
ctx.getResource("file:///some/resource/path/myTemplate.txt");
```
```java?linenums
// force this FileSystemXmlApplicationContext to load its definition via a UrlResource
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("file:///conf/context.xml");
```

## 3. 验证，数据绑定和类型转换
### 3.1. 介绍
```
JSR-303 / JSR-349 Bean验证
Spring Framework 4.0在设置支持方面支持Bean Validation 1.0（JSR-303）和Bean Validation 1.1（JSR-349），并使其适应Spring的Validator界面。

应用程序可以选择全局启用Bean Validation一次，如 Spring Validation中所述，并专门用于所有验证需求。

应用程序还可以为Validator每个 DataBinder实例注册其他Spring 实例，如配置DataBinder中所述。这可以用于在不使用注释的情况下插入验证逻辑。
```
将验证视为业务逻辑有利有弊，而Spring提供了一种不排除其中任何一种的验证（和数据绑定）设计。特别是验证不应该与Web层绑定，应该易于本地化，并且应该可以插入任何可用的验证器。考虑到上述情况，Spring提出了一个Validator基本的界面，在应用程序的每一层都非常有用。

数据绑定对于允许用户输入动态绑定到应用程序的域模型（或用于处理用户输入的任何对象）非常有用。Spring提供了所谓的DataBinder那样做。在Validator和 DataBinder补validation包，它主要在使用，但不限于MVC框架。

这BeanWrapper是Spring Framework中的一个基本概念，并且在很多地方使用。但是，您可能不需要BeanWrapper 直接使用。因为这是参考文献，我们觉得可能会有一些解释。我们将BeanWrapper在本章中解释，因为如果你要使用它，你很可能在尝试将数据绑定到对象时这样做。

Spring的DataBinder和较低级别的BeanWrapper都使用PropertyEditors来解析和格式化属性值。该PropertyEditor概念是JavaBeans规范的一部分，本章也对此进行了解释。Spring 3引入了一个“core.convert”包，它提供了一般类型转换工具，以及用于格式化UI字段值的更高级“格式”包。这些新包可以用作PropertyEditors的更简单的替代方法，本章也将对此进行讨论。
### 3.2. 使用Spring的Validator接口进行验证
Spring提供了一个Validator可用于验证对象的接口。该 Validator接口的工作方式使用Errors对象，以便在验证，验证器可以报告验证失败的Errors对象。

让我们考虑一个小数据对象：
```java?linenums
public class Person {

    private String name;
    private int age;

    // the usual getters and setters...
}
```
我们将Person通过实现org.springframework.validation.Validator接口的以下两种方法来为类提供验证行为：

- supports(Class)- 这可以Validator验证提供的实例Class吗？

- validate(Object, org.springframework.validation.Errors)- 验证给定对象，如果验证错误，则注册具有给定Errors对象的对象

实现a Validator非常简单，特别是当您知道ValidationUtilsSpring Framework提供的 帮助程序类时。
```java?linenums
public class PersonValidator implements Validator {

    /**
     * This Validator validates *just* Person instances
     */
    public boolean supports(Class clazz) {
        return Person.class.equals(clazz);
    }

    public void validate(Object obj, Errors e) {
        ValidationUtils.rejectIfEmpty(e, "name", "name.empty");
        Person p = (Person) obj;
        if (p.getAge() < 0) {
            e.rejectValue("age", "negativevalue");
        } else if (p.getAge() > 110) {
            e.rejectValue("age", "too.darn.old");
        }
    }
}
```
如您所见，类的static rejectIfEmpty(..)方法ValidationUtils用于拒绝'name'属性，如果它是null空字符串。看看ValidationUtilsjavadocs，除了前面的例子之外，看看它提供了什么功能。

虽然可以实现单个Validator类来验证富对象中的每个嵌套对象，但最好将每个嵌套对象类的验证逻辑封装在自己的Validator实现中。“富”对象的一个简单示例Customer是由两个String 属性（第一个和第二个名称）和一个复杂Address对象组成。Address对象可以独立于Customer对象使用，因此实现了不同的对象AddressValidator 。如果您希望CustomerValidator重用AddressValidator类中包含的逻辑而不需要复制和粘贴，则可以AddressValidator在您的内部依赖注入或实例化CustomerValidator，并使用它，如下所示：
```java?linenums
public class CustomerValidator implements Validator {

    private final Validator addressValidator;

    public CustomerValidator(Validator addressValidator) {
        if (addressValidator == null) {
            throw new IllegalArgumentException("The supplied [Validator] is " +
                "required and must not be null.");
        }
        if (!addressValidator.supports(Address.class)) {
            throw new IllegalArgumentException("The supplied [Validator] must " +
                "support the validation of [Address] instances.");
        }
        this.addressValidator = addressValidator;
    }

    /**
     * This Validator validates Customer instances, and any subclasses of Customer too
     */
    public boolean supports(Class clazz) {
        return Customer.class.isAssignableFrom(clazz);
    }

    public void validate(Object target, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "firstName", "field.required");
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "surname", "field.required");
        Customer customer = (Customer) target;
        try {
            errors.pushNestedPath("address");
            ValidationUtils.invokeValidator(this.addressValidator, customer.getAddress(), errors);
        } finally {
            errors.popNestedPath();
        }
    }
}
```
验证错误将报告给Errors传递给验证程序的对象。对于Spring Web MVC，您可以使用<spring:bind/>标记来检查错误消息，但当然您也可以自己检查错误对象。有关它提供的方法的更多信息可以在javadocs中找到。

### 3.3. 解决错误消息的代码
我们已经讨论了数据绑定和验证。输出与验证错误相对应的消息是我们需要讨论的最后一件事。在我们上面显示的示例中，我们拒绝了name该age字段。如果我们要使用a输出错误消息MessageSource，我们将使用我们在拒绝字段时给出的错误代码（在这种情况下为'name'和'age'）。当你从接口调用（直接或间接地，使用例如ValidationUtils类）rejectValue或其他reject方法之一时Errors，底层实现不仅会注册你传入的代码，还会注册一些额外的错误代码。它注册的错误代码由MessageCodesResolver使用的错误代码决定。默认情况下DefaultMessageCodesResolver使用，例如，不仅使用您提供的代码注册消息，还包含您传递给reject方法的字段名称的消息。因此，如果您拒绝使用字段 rejectValue("age", "too.darn.old")，除了too.darn.old代码之外，Spring还会注册too.darn.old.age并且too.darn.old.age.int（因此第一个将包含字段名称，第二个将包括字段的类型）; 这样做是为了方便开发人员定位错误消息等。

有关MessageCodesResolver和默认策略的更多信息可以分别在javadocs MessageCodesResolver 和 DefaultMessageCodesResolver。

### 3.4. Bean操作和BeanWrapper
该org.springframework.beans软件包遵循Oracle提供的JavaBeans标准。JavaBean只是一个带有默认无参数构造函数的类，它遵循命名约定，其中（作为示例）一个名为的属性bingoMadness将具有setter方法setBingoMadness(..)和getter方法getBingoMadness()。有关JavaBeans和规范的更多信息，请参阅Oracle的网站（ javabeans）。

beans包中一个非常重要的类是BeanWrapper接口及其相应的实现（BeanWrapperImpl）。引自javadocs， BeanWrapper提供设置和获取属性值（单独或批量），获取属性描述符以及查询属性以确定它们是可读还是可写的功能。此外，这些商品BeanWrapper支持嵌套属性，可以将子属性的属性设置为无限深度。然后， BeanWrapper支持标准JavaBean的能力PropertyChangeListeners 和VetoableChangeListeners，而不需要在辅助代码。最后但并非最不重要的是，它BeanWrapper提供了对索引属性设置的支持。在BeanWrapper通常不使用应用程序代码直接的，而是由DataBinder和BeanFactory。

工作的方式BeanWrapper部分由其名称表示：它包装bean以对该bean执行操作，如设置和检索属性。
#### 3.4.1. 设置和获取基本和嵌套属性
设置和获取属性是使用setPropertyValue(s)和 getPropertyValue(s)两个带有几个重载变体的方法完成的。它们都在Spring附带的javadocs中有更详细的描述。重要的是要知道有一些用于指示对象属性的约定。几个例子：
下面你会找到一些使用BeanWrapperto get和set属性的例子。

（下一节如果你不打算与合作是不是对你非常重要的BeanWrapper。如果您只是使用了DataBinder和BeanFactory 他们外的扩展实现，你可以跳过约一节 PropertyEditors。）

考虑以下两个类：

```java?linenums
public class Company {

    private String name;
    private Employee managingDirector;

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Employee getManagingDirector() {
        return this.managingDirector;
    }

    public void setManagingDirector(Employee managingDirector) {
        this.managingDirector = managingDirector;
    }
}

```
```java?linenums
public class Employee {

    private String name;

    private float salary;

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public float getSalary() {
        return salary;
    }

    public void setSalary(float salary) {
        this.salary = salary;
    }
}
```
下面的代码片断展示了如何检索和操作的一些实例化属性的一些例子Companies和Employees：

```java?linenums
BeanWrapper company = new BeanWrapperImpl(new Company());
// setting the company name..
company.setPropertyValue("name", "Some Company Inc.");
// ... can also be done like this:
PropertyValue value = new PropertyValue("name", "Some Company Inc.");
company.setPropertyValue(value);

// ok, let's create the director and tie it to the company:
BeanWrapper jim = new BeanWrapperImpl(new Employee());
jim.setPropertyValue("name", "Jim Stravinsky");
company.setPropertyValue("managingDirector", jim.getWrappedInstance());

// retrieving the salary of the managingDirector through the company
Float salary = (Float) company.getPropertyValue("managingDirector.salary");
```
#### 3.4.2. 内置PropertyEditor实现
Spring使用概念PropertyEditors来实现Objecta和a 之间的转换 String。如果您考虑一下，有时可能会以与对象本身不同的方式表示属性。例如，a Date 可以用人类可读的方式表示（如String '2007-14-09'），而我们仍然能够将人类可读的表单转换回原始日期（甚至更好：转换以人类可读形式输入的任何日期，返回到Date对象）。通过注册类型的 自定义编辑器可以实现此行为java.beans.PropertyEditor。BeanWrapper如上一章所述，在特定的IoC容器中或在一个特定的IoC容器中注册自定义编辑器，使其了解如何将属性转换为所需的类型。了解更多 PropertyEditors在java.beansOracle提供的包的javadoc中。

在Spring中使用属性编辑的几个示例：

- 在bean上设置属性是使用PropertyEditors。当提到 java.lang.String你在XML文件中声明的某个bean的属性的值时，Spring将（如果相应属性的setter具有 Class-parameter）使用它ClassEditor来尝试将参数解析为Class 对象。

- 在Spring的MVC框架中解析HTTP请求参数是使用PropertyEditors您可以在所有子类中手动绑定的各种类型完成的 CommandController。

Spring有许多内置功能，PropertyEditors让生活变得轻松。下面列出了每一个，它们都位于org.springframework.beans.propertyeditors 包装中。大多数（但不是全部）（如下所示）默认注册为 BeanWrapperImpl。如果属性编辑器可以某种方式配置，您当然可以注册自己的变体来覆盖默认变量：
Spring使用它java.beans.PropertyEditorManager来设置可能需要的属性编辑器的搜索路径。搜索路径还包括sun.bean.editors，其包括PropertyEditor实现为类型，例如Font，Color和最原始类型。另请注意，标准JavaBeans基础结构将自动发现PropertyEditor类（无需显式注册它们），如果它们与它们处理的类位于同一个包中，并且与该类具有相同的名称，并'Editor'附加; 例如，可以使用以下类和包结构，这足以使FooEditor类被识别并用作PropertyEditorfor Foo-typed属性。
```
com
  chank
    pop
      Foo
      FooEditor // the PropertyEditor for the Foo class
```
请注意，您也可以在BeanInfo此处使用标准JavaBeans机制（ 在此处未详细描述）。下面是一个使用该BeanInfo机制的示例，该机制使用PropertyEditor关联类的属性显式注册一个或多个实例。
```
com
  chank
    pop
      Foo
      FooBeanInfo // the BeanInfo for the Foo class
```
这是引用FooBeanInfo类的Java源代码。这会将a CustomNumberEditor与类的age属性相关联Foo。
```java?linenums
public class FooBeanInfo extends SimpleBeanInfo {

    public PropertyDescriptor[] getPropertyDescriptors() {
        try {
            final PropertyEditor numberPE = new CustomNumberEditor(Integer.class, true);
            PropertyDescriptor ageDescriptor = new PropertyDescriptor("age", Foo.class) {
                public PropertyEditor createPropertyEditor(Object bean) {
                    return numberPE;
                };
            };
            return new PropertyDescriptor[] { ageDescriptor };
        }
        catch (IntrospectionException ex) {
            throw new Error(ex.toString());
        }
    }
}
```
注册其他自定义PropertyEditors
将bean属性设置为字符串值时，Spring IoC容器最终使用标准JavaBeans PropertyEditors将这些字符串转换为属性的复杂类型。Spring预先注册了许多自定义PropertyEditors（例如，将表示为字符串的类名转换为实际Class对象）。此外，Java的标准JavaBeans PropertyEditor查找机制允许PropertyEditor 简单地将类简单地命名，并将其放置在与其提供支持的类相同的包中，以便自动找到。

如果需要注册其他自定义PropertyEditors，可以使用多种机制。假设您有参考，最通常不方便或不推荐的手动方法是简单地使用界面的registerCustomEditor()方法 。另一个稍微方便的机制是使用一个特殊的bean工厂后处理器调用。尽管bean工厂后处理器可以与实现一起使用，但是它具有嵌套属性设置，因此强烈建议将它与其一起使用 ，可以以类似的方式部署到任何其他bean，并自动检测和应用。ConfigurableBeanFactoryBeanFactoryCustomEditorConfigurerBeanFactoryCustomEditorConfigurerApplicationContext

请注意，所有bean工厂和应用程序上下文都会自动使用许多内置属性编辑器，通过使用称为a BeanWrapper来处理属性转换的东西。BeanWrapper 寄存器在上一节中列出的标准属性编辑器。此外， ApplicationContexts还可以覆盖或添加其他数量的编辑器，以适合特定应用程序上下文类型的方式处理资源查找。

标准JavaBeans PropertyEditor实例用于将表示为字符串的属性值转换为属性的实际复杂类型。 CustomEditorConfigurer一个bean工厂后处理器，可以用来方便地添加对其他PropertyEditor实例的支持ApplicationContext。

考虑一个用户类ExoticType，以及另一个DependsOnExoticType需要 ExoticType设置为属性的类：
```java?linenums

package example;

public class ExoticType {

    private String name;

    public ExoticType(String name) {
        this.name = name;
    }
}

public class DependsOnExoticType {

    private ExoticType type;

    public void setType(ExoticType type) {
        this.type = type;
    }
}```
正确设置之后，我们希望能够将type属性指定为字符串，PropertyEditor后台会将其转换为实际 ExoticType实例：
```xml
<bean id="sample" class="example.DependsOnExoticType">
    <property name="type" value="aNameForExoticType"/>
</bean>
```
的PropertyEditor实现看上去就像这样：
```java?linenums
// converts string representation to ExoticType object
package example;

public class ExoticTypeEditor extends PropertyEditorSupport {

    public void setAsText(String text) {
        setValue(new ExoticType(text.toUpperCase()));
    }
}
```
最后，我们使用CustomEditorConfigurer注册new PropertyEditor， ApplicationContext然后根据需要使用它：
```xml
<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
    <property name="customEditors">
        <map>
            <entry key="example.ExoticType" value="example.ExoticTypeEditor"/>
        </map>
    </property>
</bean>
```
使用PropertyEditorRegistrars
使用Spring容器注册属性编辑器的另一种机制是创建和使用a PropertyEditorRegistrar。当您需要在几种不同的情况下使用同一组属性编辑器时，此接口特别有用：编写相应的注册器并在每种情况下重用它。PropertyEditorRegistrars与被称为PropertyEditorRegistry接口的接口一起工作，该接口由Spring BeanWrapper（和DataBinder）实现。PropertyEditorRegistrars 当与CustomEditorConfigurer （在此介绍）一起使用时特别方便，它暴露了一个名为的属性setPropertyEditorRegistrars(..)：PropertyEditorRegistrars以CustomEditorConfigurer这种方式添加到一个 可以很容易地与DataBinderSpring MVC 共享Controllers。此外，它避免了在自定义编辑器上进行同步的需要：PropertyEditorRegistrar期望PropertyEditor 为每个bean创建尝试创建新的实例。

使用a PropertyEditorRegistrar可能最好用一个例子说明。首先，您需要创建自己的PropertyEditorRegistrar实现：
```java?linenums
package com.foo.editors.spring;

public final class CustomPropertyEditorRegistrar implements PropertyEditorRegistrar {

    public void registerCustomEditors(PropertyEditorRegistry registry) {

        // it is expected that new PropertyEditor instances are created
        registry.registerCustomEditor(ExoticType.class, new ExoticTypeEditor());

        // you could register as many custom property editors as are required here...
    }
}
```
另请参见org.springframework.beans.support.ResourceEditorRegistrar示例 PropertyEditorRegistrar实现。请注意，在它的registerCustomEditors(..)方法实现中， 它如何创建每个属性编辑器的新实例。

接下来我们配置一个CustomEditorConfigurer并将一个实例 CustomPropertyEditorRegistrar注入其中：
```xml
<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
    <property name="propertyEditorRegistrars">
        <list>
            <ref bean="customPropertyEditorRegistrar"/>
        </list>
    </property>
</bean>

<bean id="customPropertyEditorRegistrar"
    class="com.foo.editors.spring.CustomPropertyEditorRegistrar"/>
```
最后，与本章的重点有所不同，对于那些使用Spring的MVC Web框架的人来说，PropertyEditorRegistrars结合使用数据绑定Controllers（如SimpleFormController）可以非常方便。以下是PropertyEditorRegistrar在initBinder(..)方法实现中使用a的示例：
```java?linenums
public final class RegisterUserController extends SimpleFormController {

    private final PropertyEditorRegistrar customPropertyEditorRegistrar;

    public RegisterUserController(PropertyEditorRegistrar propertyEditorRegistrar) {
        this.customPropertyEditorRegistrar = propertyEditorRegistrar;
    }

    protected void initBinder(HttpServletRequest request,
            ServletRequestDataBinder binder) throws Exception {
        this.customPropertyEditorRegistrar.registerCustomEditors(binder);
    }

    // other methods to do with registering a User
}
```
这种PropertyEditor注册方式可以导致简洁的代码（实现initBinder(..)只需一行！），并允许将通用PropertyEditor 注册代码封装在一个类中，然后Controllers根据需要共享 。

### 3.5. Spring类型转换
Spring 3引入了一个core.convert提供通用类型转换系统的包。系统定义了一个SPI来实现类型转换逻辑，以及一个在运行时执行类型转换的API。在Spring容器中，此系统可用作PropertyEditors的替代方法，以将外部化的bean属性值字符串转换为必需的属性类型。公共API也可以在您的应用程序中需要进行类型转换的任何地方使用。
#### 3.5.1. 转换器SPI
用于实现类型转换逻辑的SPI简单且强类型化：
```java?linenums
package org.springframework.core.convert.converter;

public interface Converter<S, T> {

    T convert(S source);

}
```
要创建自己的转换器，只需实现上面的接口即可。参数S 化为您要转换的类型，以及T您要转换为的类型。如果S需要将集合或阵列转换为阵列或集合，则也可以透明地应用这种转换器T，前提是已经注册了委托阵列/集合转换器（DefaultConversionService默认情况下）。

对于每次调用convert(S)，source参数保证为NOT null。如果转换失败，您的Converter可能会抛出任何未经检查的异常; 具体而言， IllegalArgumentException应该抛出一个报告无效的源值。注意确保您的Converter实现是线程安全的。

core.convert.support为方便起见，在包中提供了几种转换器实现。这些包括从字符串到数字的转换器和其他常见类型。考虑StringToInteger作为典型Converter实现的示例：
```java?linenums
package org.springframework.core.convert.support;

final class StringToInteger implements Converter<String, Integer> {

    public Integer convert(String source) {
        return Integer.valueOf(source);
    }

}
```
#### 3.5.2. ConverterFactory
当您需要集中整个类层次结构的转换逻辑时，例如，从String转换为java.lang.Enum对象时，请实现 ConverterFactory：
```java?linenums
package org.springframework.core.convert.converter;

public interface ConverterFactory<S, R> {

    <T extends R> Converter<S, T> getConverter(Class<T> targetType);

}
```
参数化S为您要转换的类型，R为定义您可以转换为的类范围的基本类型。然后实现getConverter（Class <T>），其中T是R的子类。

以StringToEnumConverterFactory为例：
```java?linenums
package org.springframework.core.convert.support;

final class StringToEnumConverterFactory implements ConverterFactory<String, Enum> {

    public <T extends Enum> Converter<String, T> getConverter(Class<T> targetType) {
        return new StringToEnumConverter(targetType);
    }

    private final class StringToEnumConverter<T extends Enum> implements Converter<String, T> {

        private Class<T> enumType;

        public StringToEnumConverter(Class<T> enumType) {
            this.enumType = enumType;
        }

        public T convert(String source) {
            return (T) Enum.valueOf(this.enumType, source.trim());
        }
    }
}
```
#### 3.5.3. GenericConverter
当您需要复杂的Converter实现时，请考虑GenericConverter接口。通过更灵活但不太强类型的签名，GenericConverter支持在多种源和目标类型之间进行转换。此外，GenericConverter提供了在实现转换逻辑时可以使用的源和目标字段上下文。这样的上下文允许类型转换由字段注释或在字段签名上声明的通用信息驱动。

```java?linenums

package org.springframework.core.convert.converter;

public interface GenericConverter {

    public Set<ConvertiblePair> getConvertibleTypes();

    Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);

}
```
要实现GenericConverter，请让getConvertibleTypes（）返回支持的源→目标类型对。然后实现convert（Object，TypeDescriptor，TypeDescriptor）来实现转换逻辑。源TypeDescriptor提供对包含要转换的值的源字段的访问。目标TypeDescriptor提供对将设置转换值的目标字段的访问。

GenericConverter的一个很好的例子是在Java Array和Collection之间进行转换的转换器。这样的ArrayToCollectionConverter会对声明目标Collection类型的字段进行内省，以解析Collection的元素类型。这允许在目标字段上设置Collection之前，将源数组中的每个元素转换为Collection元素类型。
```
因为GenericConverter是一个更复杂的SPI接口，所以只在需要时使用它。喜欢Converter或ConverterFactory以满足基本的类型转换需求。
```
ConditionalGenericConverter
有时您只想在Converter特定条件成立时执行。例如，Converter如果目标字段上存在特定注释，则可能只想执行a 。或者，Converter如果static valueOf在目标类上定义了特定方法（如方法），则可能只想执行a 。 ConditionalGenericConverter是GenericConverter和 ConditionalConverter接口的联合，允许您定义这样的自定义匹配条件：

```java?linenums
public interface ConditionalConverter {

    boolean matches(TypeDescriptor sourceType, TypeDescriptor targetType);

}

public interface ConditionalGenericConverter
    extends GenericConverter, ConditionalConverter {

}

```
a的一个很好的例子ConditionalGenericConverter是EntityConverter，它在持久实体标识符和实体引用之间进行转换。如果目标实体类型声明静态查找器方法，则这样的EntityConverter可能仅匹配 findAccount(Long)。你会在执行中执行这样的finder方法检查 matches(TypeDescriptor, TypeDescriptor)。
#### 3.5.4. ConversionService API
ConversionService定义了一个统一的API，用于在运行时执行类型转换逻辑。转换器通常在此Facade接口后面执行：
```java?linenums
package org.springframework.core.convert;

public interface ConversionService {

    boolean canConvert(Class<?> sourceType, Class<?> targetType);

    <T> T convert(Object source, Class<T> targetType);

    boolean canConvert(TypeDescriptor sourceType, TypeDescriptor targetType);

    Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);

}
```
大多数ConversionService实现也实现ConverterRegistry，它提供用于注册转换器的SPI。在内部，ConversionService实现委托其注册的转换器执行类型转换逻辑。

core.convert.support 包中提供了强大的ConversionService实现。GenericConversionService是适用于大多数环境的通用实现。ConversionServiceFactory提供了一个方便的工厂来创建常见的ConversionService配置。

#### 3.5.5. 配置ConversionService
ConversionService是一个无状态对象，旨在在应用程序启动时实例化，然后在多个线程之间共享。在Spring应用程序中，通常为每个Spring容器（或ApplicationContext）配置一个ConversionService实例。那个ConversionService将被Spring选中，然后在框架需要执行类型转换时使用。您也可以将此ConversionService注入任何bean并直接调用它。

```
如果没有向Spring注册ConversionService，则使用基于PropertyEditor的原始系统。
```
要使用Spring注册默认的ConversionService，请使用id添加以下bean定义conversionService：

```xml
<bean id="conversionService"
    class="org.springframework.context.support.ConversionServiceFactoryBean"/>
```
默认的ConversionService可以在字符串，数字，枚举，集合，映射和其他常见类型之间进行转换。要使用您自己的自定义转换器补充或覆盖默认转换器，请设置该converters属性。属性值可以实现Converter，ConverterFactory或GenericConverter接口。

```xml
<bean id="conversionService"
        class="org.springframework.context.support.ConversionServiceFactoryBean">
    <property name="converters">
        <set>
            <bean class="example.MyCustomConverter"/>
        </set>
    </property>
</bean>
```
在Spring MVC应用程序中使用ConversionService也很常见。请参阅 Spring MVC章节中的转换和格式化。

在某些情况下，您可能希望在转换期间应用格式。有关使用的详细信息， 请参阅 FormatterRegistry SPIFormattingConversionServiceFactoryBean。
#### 3.5.6. 以编程方式使用ConversionService
要以编程方式使用ConversionService实例，只需像对任何其他bean一样注入对它的引用：

```java?linenums
@Service
public class MyService {

    @Autowired
    public MyService(ConversionService conversionService) {
        this.conversionService = conversionService;
    }

    public void doIt() {
        this.conversionService.convert(...)
    }
}
```
对于大多数用例，可以使用convert指定targetType的方法，但它不适用于更复杂的类型，例如参数化元素的集合。如果你想转换List的Integer到List的String程序，例如，你需要提供的源和目标类型的正式定义。

幸运的是，TypeDescriptor提供了各种选项来简化：
```
DefaultConversionService cs = new DefaultConversionService();

List<Integer> input = ....
cs.convert(input,
    TypeDescriptor.forObject(input), // List<Integer> type descriptor
    TypeDescriptor.collection(List.class, TypeDescriptor.valueOf(String.class)));

```
请注意，DefaultConversionService自动寄存器转换器适用于大多数环境。这包括收集器，标转换器，也基本Object到String转换器。可以ConverterRegistry使用类 上的静态 addDefaultConverters方法向任何注册相同的转换器DefaultConversionService。

值类型转换器将被重新用于数组和集合，所以没有必要创建一个特定的转换器从转换Collection的S到 Collection的T，假设标准收集处理是适当的。

### 3.6. Spring Field格式
如前一节所述，core.convert是一种通用类型转换系统。它提供了统一的ConversionService API以及强类型的Converter SPI，用于实现从一种类型到另一种类型的转换逻辑。Spring Container使用此系统绑定bean属性值。此外，Spring Expression Language（SpEL）和DataBinder都使用此系统绑定字段值。例如，当SpEL需要强制a Short到a Long来完成expression.setValue(Object bean, Object value)尝试时，core.convert系统会执行强制。

现在考虑典型客户端环境（如Web或桌面应用程序）的类型转换要求。在这样的环境中，您通常从String转换 为支持客户端回发过程，以及返回String以支持视图呈现过程。此外，您经常需要本地化String值。更通用的core.convert Converter SPI不直接解决此类格式化要求。为了直接解决这些问题，Spring 3引入了一个方便的Formatter SPI，为客户端环境提供了一个简单而强大的PropertyEditors替代方案。

通常，在需要实现通用类型转换逻辑时使用Converter SPI; 例如，用于在java.util.Date和java.lang.Long之间进行转换。当您在客户端环境（例如Web应用程序）中工作时，请使用Formatter SPI，并且需要解析和打印本地化的字段值。ConversionService为两个SPI提供统一的类型转换API。
#### 3.6.1. 格式化SPI
用于实现字段格式化逻辑的Formatter SPI简单且强类型化：
```java?linenums
package org.springframework.format;

public interface Formatter<T> extends Printer<T>, Parser<T> {
}
```
Formatter从Printer和Parser构建块接口扩展的位置：
```java?linenums
public interface Printer<T> {
    String print(T fieldValue, Locale locale);
}

```
```java?linenums
import java.text.ParseException;

public interface Parser<T> {
    T parse(String clientValue, Locale locale) throws ParseException;
}
```
要创建自己的Formatter，只需实现上面的Formatter接口即可。将T参数化为您要格式化的对象类型，例如 java.util.Date。实现print()操作以打印T的实例以在客户端区域设置中显示。实现parse()操作以从客户端语言环境返回的格式化表示中解析T的实例。如果解析尝试失败，您的Formatter应抛出ParseException或IllegalArgumentException。请注意确保您的Formatter实现是线程安全的。

format为方便起见，在子包中提供了几种Formatter实现。该number软件包使用a 提供a NumberFormatter，CurrencyFormatter和 PercentFormatter格式化java.lang.Number对象java.text.NumberFormat。该datetime包提供了一个DateFormatter格式化java.util.Date对象的方法java.text.DateFormat。该datetime.joda软件包基于Joda-Time库提供全面的日期时间格式支持。

考虑DateFormatter作为一个示例Formatter实现：
```java?linenums
public final class DateFormatter implements Formatter<Date> {

    private String pattern;

    public DateFormatter(String pattern) {
        this.pattern = pattern;
    }

    public String print(Date date, Locale locale) {
        if (date == null) {
            return "";
        }
        return getDateFormat(locale).format(date);
    }

    public Date parse(String formatted, Locale locale) throws ParseException {
        if (formatted.length() == 0) {
            return null;
        }
        return getDateFormat(locale).parse(formatted);
    }

    protected DateFormat getDateFormat(Locale locale) {
        DateFormat dateFormat = new SimpleDateFormat(this.pattern, locale);
        dateFormat.setLenient(false);
        return dateFormat;
    }

}
```
Spring团队欢迎社区推动的Formatter贡献; 请参阅 jira.spring.io来做出贡献。
#### 3.6.2. 注释驱动的格式
如您所见，字段格式可以通过字段类型或注释进行配置。要将Annotation绑定到格式化程序，请实现AnnotationFormatterFactory：

```java?linenums
package org.springframework.format;

public interface AnnotationFormatterFactory<A extends Annotation> {

    Set<Class<?>> getFieldTypes();

    Printer<?> getPrinter(A annotation, Class<?> fieldType);

    Parser<?> getParser(A annotation, Class<?> fieldType);

}
```
例如，将参数化A设置为您希望将格式化逻辑与之关联的字段annotationType org.springframework.format.annotation.DateTimeFormat。已经 getFieldTypes()返回类型字段中的注释，可以使用上。已经 getPrinter()返回打印机打印的注释字段的值。已经 getParser()返回解析器解析一个clientValue一个注释字段。

下面的示例AnnotationFormatterFactory实现将@NumberFormat Annotation绑定到格式化程序。此注释允许指定数字样式或模式：

```java?linenums
public final class NumberFormatAnnotationFormatterFactory
        implements AnnotationFormatterFactory<NumberFormat> {

    public Set<Class<?>> getFieldTypes() {
        return new HashSet<Class<?>>(asList(new Class<?>[] {
            Short.class, Integer.class, Long.class, Float.class,
            Double.class, BigDecimal.class, BigInteger.class }));
    }

    public Printer<Number> getPrinter(NumberFormat annotation, Class<?> fieldType) {
        return configureFormatterFrom(annotation, fieldType);
    }

    public Parser<Number> getParser(NumberFormat annotation, Class<?> fieldType) {
        return configureFormatterFrom(annotation, fieldType);
    }

    private Formatter<Number> configureFormatterFrom(NumberFormat annotation,
            Class<?> fieldType) {
        if (!annotation.pattern().isEmpty()) {
            return new NumberFormatter(annotation.pattern());
        } else {
            Style style = annotation.style();
            if (style == Style.PERCENT) {
                return new PercentFormatter();
            } else if (style == Style.CURRENCY) {
                return new CurrencyFormatter();
            } else {
                return new NumberFormatter();
            }
        }
    }
}
```
要触发格式化，只需使用@NumberFormat注释字段：
```java?linenums
public class MyModel {

    @NumberFormat(style=Style.CURRENCY)
    private BigDecimal decimal;

}

```
格式注释API
org.springframework.format.annotation 包中存在可移植格式注释API 。使用@NumberFormat格式化java.lang.Number字段。使用@DateTimeFormat格式化java.util.Date，java.util.Calendar，java.util.Long或Joda-Time字段。

下面的示例使用@DateTimeFormat将java.util.Date格式化为ISO Date（yyyy-MM-dd）：
```java?linenums
public class MyModel {

    @DateTimeFormat(iso=ISO.DATE)
    private Date date;

}
```
#### 3.6.3. FormatterRegistry SPI
FormatterRegistry是一个用于注册格式化程序和转换器的SPI。 FormattingConversionService是适用于大多数环境的FormatterRegistry的实现。可以通过编程方式或声明方式将此实现配置为Spring bean FormattingConversionServiceFactoryBean。由于此实现也实现了ConversionService，因此可以直接配置它以与Spring的DataBinder和Spring Expression Language（SpEL）一起使用。

查看下面的FormatterRegistry SPI：
```java?linenums
package org.springframework.format;

public interface FormatterRegistry extends ConverterRegistry {

    void addFormatterForFieldType(Class<?> fieldType, Printer<?> printer, Parser<?> parser);

    void addFormatterForFieldType(Class<?> fieldType, Formatter<?> formatter);

    void addFormatterForFieldType(Formatter<?> formatter);

    void addFormatterForAnnotation(AnnotationFormatterFactory<?, ?> factory);

}
```
如上所示，可以通过fieldType或注释注册Formatters。

FormatterRegistry SPI允许您集中配置格式化规则，而不是在控制器之间复制此类配置。例如，您可能希望强制所有Date字段以某种方式格式化，或者具有特定注释的字段以某种方式格式化。使用共享的FormatterRegistry，您可以定义一次这些规则，并在需要格式化时应用它们。
#### 3.6.4. FormatterRegistrar SPI
FormatterRegistrar是一个SPI，用于通过FormatterRegistry注册格式化程序和转换器：
```java?linenums
package org.springframework.format;

public interface FormatterRegistrar {

    void registerFormatters(FormatterRegistry registry);

}
```
在为给定格式类别注册多个相关转换器和格式化程序时，FormatterRegistrar非常有用，例如日期格式。在声明性注册不足的情况下，它也很有用。例如，格式化程序需要在与其自己的<T>不同的特定字段类型下或在注册打印机/分析程序对时编制索引。下一节提供有关转换器和格式化程序注册的更多信息。
#### 3.6.5. 在Spring MVC中配置格式化
请参阅Spring MVC章节中的转换和格式化。

### 3.7. 配置全局日期和时间格式
默认情况下，未@DateTimeFormat使用该DateFormat.SHORT样式转换的字符串将转换未注释的日期和时间字段。如果您愿意，可以通过定义自己的全局格式来更改此设置。

您需要确保Spring不会注册默认格式化程序，而是应该手动注册所有格式化程序。根据您是否使用Joda-Time库，使用 org.springframework.format.datetime.joda.JodaTimeFormatterRegistrar或 org.springframework.format.datetime.DateFormatterRegistrar类。

例如，以下Java配置将注册全局“yyyyMMdd”格式。此示例不依赖于Joda-Time库：

```java?linenums
@Configuration
public class AppConfig {

    @Bean
    public FormattingConversionService conversionService() {

        // Use the DefaultFormattingConversionService but do not register defaults
        DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService(false);

        // Ensure @NumberFormat is still supported
        conversionService.addFormatterForFieldAnnotation(new NumberFormatAnnotationFormatterFactory());

        // Register date conversion with a specific global format
        DateFormatterRegistrar registrar = new DateFormatterRegistrar();
        registrar.setFormatter(new DateFormatter("yyyyMMdd"));
        registrar.registerFormatters(conversionService);

        return conversionService;
    }
}
```
如果您更喜欢基于XML的配置，则可以使用 FormattingConversionServiceFactoryBean。这是相同的例子，这次使用Joda Time：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd>

    <bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="registerDefaultFormatters" value="false" />
        <property name="formatters">
            <set>
                <bean class="org.springframework.format.number.NumberFormatAnnotationFormatterFactory" />
            </set>
        </property>
        <property name="formatterRegistrars">
            <set>
                <bean class="org.springframework.format.datetime.joda.JodaTimeFormatterRegistrar">
                    <property name="dateFormatter">
                        <bean class="org.springframework.format.datetime.joda.DateTimeFormatterFactoryBean">
                            <property name="pattern" value="yyyyMMdd"/>
                        </bean>
                    </property>
                </bean>
            </set>
        </property>
    </bean>
</beans>
```
```
乔达时提供单独的不同类型来表示date，time和date-time 的值。的dateFormatter，timeFormatter和dateTimeFormatter的性质 JodaTimeFormatterRegistrar，应使用来配置不同的格式为每种类型。它DateTimeFormatterFactoryBean提供了一种创建格式化程序的便捷方法。


```
如果您正在使用Spring MVC，请记住明确配置所使用的转换服务。对于基于Java的，@Configuration这意味着扩展 WebMvcConfigurationSupport类并重写mvcConversionService()方法。对于XML，您应该使用元素的'conversion-service'属性 mvc:annotation-driven。有关详细信息，请参阅转换和格式。

### 3.8. Spring验证
Spring 3为其验证支持引入了一些增强功能。首先，现在完全支持JSR-303 Bean Validation API。其次，当以编程方式使用时，Spring的DataBinder现在可以验证对象以及绑定它们。第三，Spring MVC现在支持声明性地验证@Controller输入。

#### 3.8.1. JSR-303 Bean Validation API概述
JSR-303标准化了Java平台的验证约束声明和元数据。使用此API，您可以使用声明性验证约束来注释域模型属性，并且运行时会强制执行它们。您可以利用许多内置约束。您还可以定义自己的自定义约束。

为了说明，请考虑一个具有两个属性的简单PersonForm模型：
```java?linenums
public class PersonForm {
    private String name;
    private int age;
}
```
JSR-303允许您为这些属性定义声明性验证约束：
```java?linenums
public class PersonForm {

    @NotNull
    @Size(max=64)
    private String name;

    @Min(0)
    private int age;

}
```
当JSR-303 Validator验证此类的实例时，将强制执行这些约束。

有关JSR-303 / JSR-349的一般信息，请参阅Bean Validation网站。有关默认参考实现的特定功能的信息，请参阅Hibernate Validator文档。要了解如何将Bean Validation提供程序设置为Spring bean，请继续阅读。
#### 3.8.2. 配置Bean验证提供程序
Spring提供对Bean Validation API的完全支持。这包括方便地支持将JSR-303 / JSR-349 Bean Validation提供程序作为Spring bean引导。这允许在您的应用程序中需要验证的地方注入javax.validation.ValidatorFactory或javax.validation.Validator注入。

使用将LocalValidatorFactoryBean默认Validator配置为Spring bean：

```xml
<bean id="validator"
    class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean"/>
```
上面的基本配置将触发Bean Validation使用其默认引导机制进行初始化。JSR-303 / JSR-349提供程序（如Hibernate Validator）预计将出现在类路径中，并将自动检测。

注入验证器
LocalValidatorFactoryBean同时实现了javax.validation.ValidatorFactory和 javax.validation.Validator，以及Spring的 org.springframework.validation.Validator。您可以将这些接口中的任何一个引用注入到需要调用验证逻辑的bean中。

javax.validation.Validator如果您希望直接使用Bean Validation API，请引用一个引用：
```java?linenums
import javax.validation.Validator;

@Service
public class MyService {

    @Autowired
    private Validator validator;
```
org.springframework.validation.Validator如果您的bean需要Spring Validation API，请引用一个引用：
```java?linenums
import org.springframework.validation.Validator;

@Service
public class MyService {

    @Autowired
    private Validator validator;

}
```
配置自定义约束
每个Bean Validation约束由两部分组成。首先，@Constraint声明约束及其可配置属性的注释。第二，实现javax.validation.ConstraintValidator约束行为的接口的实现。要将声明与实现相关联，每个@Constraint注释都引用相应的ValidationConstraint实现类。在运行时，ConstraintValidatorFactory在域模型中遇到约束注释时， 实例化引用的实现。

默认情况下，LocalValidatorFactoryBean配置SpringConstraintValidatorFactory 使用Spring创建ConstraintValidator实例的a。这允许您的自定义ConstraintValidators像其他任何Spring bean一样受益于依赖注入。

下面显示的是自定义@Constraint声明的示例，后跟一个ConstraintValidator使用Spring进行依赖项注入的关联 实现：

```java?linenums
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy=MyConstraintValidator.class)
public @interface MyConstraint {
}
```
```java?linenums
import javax.validation.ConstraintValidator;

public class MyConstraintValidator implements ConstraintValidator {

    @Autowired;
    private Foo aDependency;

    ...
}
```
正如您所看到的，ConstraintValidator实现可能与其他任何Spring bean一样具有@Autowired的依赖关系。

弹簧驱动的方法验证
Bean Validation 1.1支持的方法验证功能，以及Hibernate Validator 4.3的自定义扩展，可以通过MethodValidationPostProcessorbean定义集成到Spring上下文中：
```xml
<bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor"/>
```
为了有资格进行Spring驱动的方法验证，所有目标类都需要使用Spring的@Validated注释进行注释，可以选择声明要使用的验证组。MethodValidationPostProcessor使用Hibernate Validator和Bean Validation 1.1提供程序查看javadocs的设置详细信息。

其他配置选项
LocalValidatorFactoryBean对于大多数情况，默认配置应该足够了。从消息插值到遍历解析，各种Bean Validation构造有许多配置选项。有关LocalValidatorFactoryBean这些选项的更多信息，请参阅 javadocs。
#### 3.8.3. 配置DataBinder
从Spring 3开始，可以使用Validator配置DataBinder实例。配置完成后，可以通过调用来调用Validator binder.validate()。任何验证错误都会自动添加到活页夹的BindingResult中。

以编程方式使用DataBinder时，可以在绑定到目标对象后调用验证逻辑：
```
Foo target = new Foo();
DataBinder binder = new DataBinder(target);
binder.setValidator(new FooValidator());

// bind to the target object
binder.bind(propertyValues);

// validate the target object
binder.validate();

// get BindingResult that includes any validation errors
BindingResult results = binder.getBindingResult();
```
DataBinder也可以Validator通过dataBinder.addValidators和配置多个实例 dataBinder.replaceValidators。将全局配置的Bean验证与Validator在DataBinder实例上本地配置的Spring组合时，这非常有用。请参阅[validation-mvc-configurations]。

#### 3.8.4. Spring MVC 3验证
请参阅Spring MVC章节中的验证。
## 4. Spring表达语言（SpEL）
### 4.1. 介绍
Spring Expression Language（简称SpEL）是一种强大的表达式语言，支持在运行时查询和操作对象图。语言语法类似于Unified EL，但提供了其他功能，最着名的是方法调用和基本字符串模板功能。

虽然有几种其他Java表达式语言可用--OGNL，MVEL和JBoss EL，仅举几例 - 创建Spring表达式语言是为了向Spring社区提供一种支持良好的表达式语言，可以在所有产品中使用。春季组合。其语言功能由Spring产品组合中项目的需求驱动，包括基于Eclipse的Spring Tool Suite中代码完成支持的工具要求。也就是说，SpEL基于技术不可知的API，允许在需要时集成其他表达式语言实现。

虽然SpEL是Spring组合中表达式评估的基础，但它并不直接与Spring绑定，可以单独使用。为了自包含，本章中的许多示例都使用SpEL，就像它是一种独立的表达式语言一样。这需要创建一些引导基础结构类，例如解析器。大多数Spring用户不需要处理此基础结构，而只会创建表达式字符串以进行评估。这种典型用法的一个示例是将SpEL集成到创建XML或基于注释的bean定义中，如表达式支持定义bean定义一节所示。

本章介绍表达式语言的功能，API及其语言语法。在几个地方，一个Inventor和Inventor的Society类被用作表达式评估的目标对象。这些类声明和用于填充它们的数据列在本章末尾。

表达式语言支持以下功能：
- 文字表达

- 布尔和关系运算符

- 常用表达

- 类表达式

- 访问属性，数组，列表，映射

- 方法调用

- 关系运算符

- 分配

- 调用构造函数

- Bean引用

- 阵列构造

- 内联列表

- 内联地图

- 三元运算符

- 变量

- 用户定义的功能

- 收集投影

- 收藏品选择

- 模板化的表达
### 4.2. 评估
本节介绍SpEL接口及其表达式语言的简单使用。完整的语言参考可以在语言参考一节中找到 。

以下代码介绍了用于评估文字字符串表达式“Hello World”的SpEL API。
```java?linenums
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'");
String message = (String) exp.getValue();
```
消息变量的值只是“Hello World”。

您最有可能使用的SpEL类和接口位于包 org.springframework.expression及其子包中，例如spel.support。

该接口ExpressionParser负责解析表达式字符串。在此示例中，表达式字符串是由周围的单引号表示的字符串文字。该接口Expression负责评估先前定义的表达式字符串。有两个例外，可以被抛出，ParseException而 EvaluationException打电话时parser.parseExpression和exp.getValue 分别。

SpEL支持广泛的功能，例如调用方法，访问属性和调用构造函数。

作为方法调用的示例，我们concat在字符串文字上调用该方法。
```java?linenums
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'.concat('!')");
String message = (String) exp.getValue();
```
消息的价值现在是'Hello World！'。

作为调用JavaBean属性的示例，可以调用String属性Bytes，如下所示。

```java?linenums
ExpressionParser parser = new SpelExpressionParser();

// invokes 'getBytes()'
Expression exp = parser.parseExpression("'Hello World'.bytes");
byte[] bytes = (byte[]) exp.getValue();
```
SpEL还支持使用标准点表示法的嵌套属性，即 prop1.prop2.prop3属性值的设置

也可以访问公共字段。
```java?linenums
ExpressionParser parser = new SpelExpressionParser();

// invokes 'getBytes().length'
Expression exp = parser.parseExpression("'Hello World'.bytes.length");
int length = (Integer) exp.getValue();
```
可以调用String的构造函数而不是使用字符串文字。
```java?linenums
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("new String('hello world').toUpperCase()");
String message = exp.getValue(String.class);
```
请注意泛型方法的使用public <T> T getValue(Class<T> desiredResultType)。使用此方法无需将表达式的值强制转换为所需的结果类型。一个EvaluationException如果该值不能被转换到该类型将被抛出T或使用注册的类型转换器转换。

SpEL的更常见用法是提供针对特定对象实例（称为根对象）计算的表达式字符串。该示例显示如何name从Inventor类的实例检索属性或创建布尔条件：
```java?linenums
// Create and set a calendar
GregorianCalendar c = new GregorianCalendar();
c.set(1856, 7, 9);

// The constructor arguments are name, birthday, and nationality.
Inventor tesla = new Inventor("Nikola Tesla", c.getTime(), "Serbian");

ExpressionParser parser = new SpelExpressionParser();

Expression exp = parser.parseExpression("name");
String name = (String) exp.getValue(tesla);
// name == "Nikola Tesla"

exp = parser.parseExpression("name == 'Nikola Tesla'");
boolean result = exp.getValue(tesla, Boolean.class);
// result == true
```
#### 4.2.1. EvaluationContext
EvaluationContext在评估表达式以解析属性，方法或字段以及帮助执行类型转换时，将使用该接口。有两个开箱即用的实现。

- SimpleEvaluationContext - 为不需要SpEL语言语法的完整范围的表达式类别公开必要的SpEL语言特性和配置选项的子集，并且应该进行有意义的限制。示例包括但不限于数据绑定表达式，基于属性的过滤器等。

- StandardEvaluationContext - 公开全套SpEL语言功能和配置选项。您可以使用它来指定默认根对象并配置每个可用的与评估相关的策略。

SimpleEvaluationContext旨在仅支持SpEL语言语法的子集。它排除了 Java类型引用，构造函数和bean引用。它还要求明确选择表达式中属性和方法的支持级别。默认情况下，create()静态工厂方法仅启用对属性的读访问权限。您还可以获取构建器以配置所需的确切支持级别，定位以下一个或多个组合：

- PropertyAccessor仅限定制（无反射）

- 只读访问的数据绑定属性

- 读写的数据绑定属性

类型转换
默认情况下，SpEL使用Spring core（org.springframework.core.convert.ConversionService）中提供的转换服务。此转换服务附带了许多内置的转换器，可用于常见转换，但也可完全扩展，因此可以添加类型之间的自定义转换。此外，它具有通用感知的关键功能。这意味着当在表达式中使用泛型类型时，SpEL将尝试转换以维护它遇到的任何对象的类型正确性。

这在实践中意味着什么？假设使用赋值setValue()来设置List属性。实际上是属性的类型List<Boolean>。SpEL将识别列表中的元素Boolean在放入其中之前需要转换为。一个简单的例子：

```java?linenums
class Simple {
    public List<Boolean> booleanList = new ArrayList<Boolean>();
}

Simple simple = new Simple();
simple.booleanList.add(true);

EvaluationContext context = SimpleEvaluationContext().forReadOnlyDataBinding().build();

// false is passed in here as a string. SpEL and the conversion service will
// correctly recognize that it needs to be a Boolean and convert it
parser.parseExpression("booleanList[0]").setValue(context, simple, "false");

// b will be false
Boolean b = simple.booleanList.get(0);
```
#### 4.2.2. 解析器配置
可以使用解析器配置对象（org.springframework.expression.spel.SpelParserConfiguration）配置SpEL表达式解析器。配置对象控制某些表达式组件的行为。例如，如果索引到数组或集合以及指定索引处的元素，null 则可以自动创建元素。当使用由一系列属性引用组成的表达式时，这很有用。如果索引到数组或列表并指定超出数组或列表当前大小末尾的索引，则可以自动增大数组或列表以适应该索引。
```java?linenums
class Demo {
    public List<String> list;
}

// Turn on:
// - auto null reference initialization
// - auto collection growing
SpelParserConfiguration config = new SpelParserConfiguration(true,true);

ExpressionParser parser = new SpelExpressionParser(config);

Expression expression = parser.parseExpression("list[3]");

Demo demo = new Demo();

Object o = expression.getValue(demo);

// demo.list will now be a real collection of 4 entries
// Each entry is a new empty String
```
也可以配置SpEL表达式编译器的行为。
#### 4.2.3. SpEL编译
Spring Framework 4.1包含一个基本的表达式编译器。表达式通常被解释为在评估期间提供大量动态灵活性但不提供最佳性能。对于偶尔的表达式使用，这很好，但是当其他组件（如Spring Integration）使用时，性能可能非常重要，并且不需要动态。

SpEL编译器旨在满足此需求。编译器将在评估期间动态生成一个真正的Java类，它体现了表达式行为并使用它来实现更快的表达式评估。由于缺少表达式类型，编译器使用在执行编译时对表达式的解释评估期间收集的信息。例如，它不完全从表达式中知道属性引用的类型，但在第一次解释的求值期间，它将找出它是什么。当然，如果各种表达元素的类型随着时间的推移而变化，那么基于此信息的编译可能会引起麻烦。因此，编译最适合于在重复评估时类型信息不会改变的表达式。

对于这样的基本表达式：

someArray[0].someProperty.someOtherProperty < 0.1

它涉及数组访问，一些属性derefencing和数字操作，性能增益可以非常明显。在50000次迭代的微基准测试示例中，仅使用解释器评估75ms，使用表达式的编译版本仅需3ms。

编译器配置
默认情况下，编译器未打开，但有两种方法可以打开它。可以使用前面讨论的解析器配置过程打开它，或者在SpEL使用嵌入另一个组件内时通过系统属性打开它。本节讨论这两个选项。

重要的是要理解编译器可以在enum（org.springframework.expression.spel.SpelCompilerMode）中捕获的几种模式。模式如下：
- OFF - 关闭编译器; 这是默认值。

- IMMEDIATE - 在立即模式下，表达式会尽快编译。这通常是在第一次解释评估之后。如果编译的表达式失败（通常由于类型更改，如上所述），则表达式求值的调用者将收到异常。

- MIXED - 在混合模式下，表达式随着时间的推移在解释和编译模式之间静默切换。经过一些解释后的运行，它们将切换到编译形式，如果编译后的表单出现问题（如类型更改，如上所述），表达式将自动再次切换回解释形式。稍后它可能会生成另一个已编译的表单并切换到它。基本上，用户进入IMMEDIATE模式的异常是在内部处理。

IMMEDIATE模式存在，因为MIXED模式可能会导致具有副作用的表达式出现问题。如果编译后的表达式在部分成功后爆炸，则可能已经完成了影响系统状态的事情。如果发生这种情况，调用者可能不希望它以解释模式静默重新运行，因为表达式的一部分可能正在运行两次。

选择模式后，使用SpelParserConfiguration配置解析器：

```java?linenums
SpelParserConfiguration config = new SpelParserConfiguration(SpelCompilerMode.IMMEDIATE,
    this.getClass().getClassLoader());

SpelExpressionParser parser = new SpelExpressionParser(config);

Expression expr = parser.parseExpression("payload");

MyMessage message = new MyMessage();

Object payload = expr.getValue(message);
```
指定编译器模式时，也可以指定类加载器（允许传递null）。编译表达式将在任何提供的子类加载器中定义。重要的是要确保是否指定了类加载器，它可以看到表达式评估过程中涉及的所有类型。如果未指定，则将使用默认的类加载器（通常是表达式求值期间运行的线程的上下文类加载器）。

配置编译器的第二种方法是在SpEL嵌入到某个其他组件中时使用，并且可能无法通过配置对象进行配置。在这些情况下，可以使用系统属性。该属性 spring.expression.compiler.mode可被设置为一个SpelCompilerMode 枚举值（off，immediate，或mixed）。

编译器限制
从Spring Framework 4.1开始，基本的编译框架已经到位。但是，该框架尚不支持编译各种表达式。最初的重点是可能在性能关键环境中使用的常见表达式。目前无法编译以下类型的表达式：
- 涉及转让的表达

- 表达式依赖于转换服务

- 使用自定义解析器或访问器的表达式

- 使用选择或投影的表达式

将来可以编写越来越多的表达类型。
### 4.3. bean定义中的表达式
SpEL表达式可以与XML或基于注释的配置元数据一起用于定义BeanDefinitions。在这两种情况下，定义表达式的语法都是表单#{ <expression string> }。
#### 4.3.1. XML配置
可以使用表达式设置属性或构造函数-arg值，如下所示。

```xml
<bean id="numberGuess" class="org.spring.samples.NumberGuess">
    <property name="randomNumber" value="#{ T(java.lang.Math).random() * 100.0 }"/>

    <!-- other properties -->
</bean>
```
变量systemProperties是预定义的，因此您可以在表达式中使用它，如下所示。请注意，您不必# 在此上下文中使用符号为预定义变量添加前缀。

```xml
<bean id="taxCalculator" class="org.spring.samples.TaxCalculator">
    <property name="defaultLocale" value="#{ systemProperties['user.region'] }"/>

    <!-- other properties -->
</bean>

```
例如，您还可以通过名称引用其他bean属性。
```xml
<bean id="numberGuess" class="org.spring.samples.NumberGuess">
    <property name="randomNumber" value="#{ T(java.lang.Math).random() * 100.0 }"/>

    <!-- other properties -->
</bean>

<bean id="shapeGuess" class="org.spring.samples.ShapeGuess">
    <property name="initialShapeSeed" value="#{ numberGuess.randomNumber }"/>

    <!-- other properties -->
</bean>
```
#### 4.3.2. 注释配置
该@Value注释可被放置在字段，方法和方法/构造函数的参数来指定一个缺省值。

以下是设置字段变量的默认值的示例。

```java?linenums
public static class FieldValueTestBean

    @Value("#{ systemProperties['user.region'] }")
    private String defaultLocale;

    public void setDefaultLocale(String defaultLocale) {
        this.defaultLocale = defaultLocale;
    }

    public String getDefaultLocale() {
        return this.defaultLocale;
    }

}
```
等价但在属性setter方法上如下所示。
```java?linenums
public static class PropertyValueTestBean

    private String defaultLocale;

    @Value("#{ systemProperties['user.region'] }")
    public void setDefaultLocale(String defaultLocale) {
        this.defaultLocale = defaultLocale;
    }

    public String getDefaultLocale() {
        return this.defaultLocale;
    }

}
```
自动化方法和构造函数也可以使用@Value注释。
```java?linenums
public class SimpleMovieLister {

    private MovieFinder movieFinder;
    private String defaultLocale;

    @Autowired
    public void configure(MovieFinder movieFinder,
            @Value("#{ systemProperties['user.region'] }") String defaultLocale) {
        this.movieFinder = movieFinder;
        this.defaultLocale = defaultLocale;
    }

    // ...
}

```
```java?linenums
public class MovieRecommender {

    private String defaultLocale;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao,
            @Value("#{systemProperties['user.country']}") String defaultLocale) {
        this.customerPreferenceDao = customerPreferenceDao;
        this.defaultLocale = defaultLocale;
    }

    // ...
}
```
### 4.4. 语言参考
####  4.4.1. 文字表达
支持的文字表达式的类型是字符串，数值（int，real，hex），boolean和null。字符串由单引号分隔。要将单引号本身放在字符串中，请使用两个单引号字符。

以下清单显示了文字的简单用法。通常，它们不会像这样单独使用，而是作为更复杂表达式的一部分使用，例如在逻辑比较运算符的一侧使用文字。
```java?linenums
ExpressionParser parser = new SpelExpressionParser();

// evals to "Hello World"
String helloWorld = (String) parser.parseExpression("'Hello World'").getValue();

double avogadrosNumber = (Double) parser.parseExpression("6.0221415E+23").getValue();

// evals to 2147483647
int maxValue = (Integer) parser.parseExpression("0x7FFFFFFF").getValue();

boolean trueValue = (Boolean) parser.parseExpression("true").getValue();

Object nullValue = parser.parseExpression("null").getValue();
```
数字支持使用负号，指数表示法和小数点。默认情况下，使用Double.parseDouble（）解析实数。
#### 4.4.2. 属性，数组，列表，地图，索引器

使用属性引用进行导航很简单：只需使用句点来指示嵌套属性值。Inventor类，pupin和tesla 的实例填充了示例中使用的类中列出的数据。为了“向下”导航并获得特斯拉的出生年份和Pupin的出生城市，使用以下表达方式.
```java?linenums
// evals to 1856
int year = (Integer) parser.parseExpression("Birthdate.Year + 1900").getValue(context);

String city = (String) parser.parseExpression("placeOfBirth.City").getValue(context);
```
属性名称的第一个字母允许不区分大小写。使用方括号表示法获得数组和列表的内容。

```java?linenums
ExpressionParser parser = new SpelExpressionParser();
EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();

// Inventions Array

// evaluates to "Induction motor"
String invention = parser.parseExpression("inventions[3]").getValue(
        context, tesla, String.class);

// Members List

// evaluates to "Nikola Tesla"
String name = parser.parseExpression("Members[0].Name").getValue(
        context, ieee, String.class);

// List and Array navigation
// evaluates to "Wireless communication"
String invention = parser.parseExpression("Members[0].Inventions[6]").getValue(
        context, ieee, String.class);
```
通过指定括号内的文字键值来获取映射的内容。在这种情况下，因为人员映射的键是字符串，我们可以指定字符串文字。
```java?linenums
// Officer's Dictionary

Inventor pupin = parser.parseExpression("Officers['president']").getValue(
        societyContext, Inventor.class);

// evaluates to "Idvor"
String city = parser.parseExpression("Officers['president'].PlaceOfBirth.City").getValue(
        societyContext, String.class);

// setting values
parser.parseExpression("Officers['advisors'][0].PlaceOfBirth.Country").setValue(
        societyContext, "Croatia");
```
#### 4.4.3. 内联列表
列表可以使用{}符号直接表达在表达式中。

```java?linenums
// evaluates to a Java list containing the four numbers
List numbers = (List) parser.parseExpression("{1,2,3,4}").getValue(context);

List listOfLists = (List) parser.parseExpression("{{'a','b'},{'x','y'}}").getValue(context);
```
{}本身就是一个空列表。出于性能原因，如果列表本身完全由固定文字组成，则创建常量列表以表示表达式，而不是在每个评估上构建新列表。

#### 4.4.4. 内联地图
地图也可以使用{key:value}符号直接在表达式中表达。
```java?linenums
// evaluates to a Java map containing the two entries
Map inventorInfo = (Map) parser.parseExpression("{name:'Nikola',dob:'10-July-1856'}").getValue(context);

Map mapOfMaps = (Map) parser.parseExpression("{name:{first:'Nikola',last:'Tesla'},dob:{day:10,month:'July',year:1856}}").getValue(context);
```
{:}本身就是一张空地图。出于性能原因，如果地图本身由固定文字或其他嵌套常量结构（列表或地图）组成，则会创建一个常量地图来表示表达式，而不是在每次评估时构建新地图。引用地图键是可选的，上面的示例不使用引用键。
#### 4.4.5. 阵列构造
可以使用熟悉的Java语法构建数组，可选地提供初始化程序以在构造时填充数组。
```java?linenums
int[] numbers1 = (int[]) parser.parseExpression("new int[4]").getValue(context);

// Array with initializer
int[] numbers2 = (int[]) parser.parseExpression("new int[]{1,2,3}").getValue(context);

// Multi dimensional array
int[][] numbers3 = (int[][]) parser.parseExpression("new int[4][5]").getValue(context);
```
在构造多维阵列时，目前不允许提供初始化器。
#### 4.4.6. 方法
使用典型的Java编程语法调用方法。您也可以在文字上调用方法。Varargs也受到支持。
```java?linenums
// string literal, evaluates to "bc"
String bc = parser.parseExpression("'abc'.substring(1, 3)").getValue(String.class);

// evaluates to true
boolean isMember = parser.parseExpression("isMember('Mihajlo Pupin')").getValue(
        societyContext, Boolean.class);
```
#### 4.4.7. 运营商
关系运算符
关系运算符; 使用标准运算符表示法支持等于，不等于，小于，小于或等于，大于等于或等于。
```java?linenums
// evaluates to true
boolean trueValue = parser.parseExpression("2 == 2").getValue(Boolean.class);

// evaluates to false
boolean falseValue = parser.parseExpression("2 < -5.0").getValue(Boolean.class);

// evaluates to true
boolean trueValue = parser.parseExpression("'black' < 'block'").getValue(Boolean.class);
```
```
大于/小于比较null遵循一个简单的规则：null在这里被视为没有（即不为零）。因此，任何其他值总是大于null（X > null总是true），并且没有其他值永远小于任何值（X < null总是如此false）。

如果您更喜欢数字比较，请避免基于数字的比较，null以支持与零进行比较（例如X > 0或X < 0）。
```
除标准关系运算符外，SpEL还支持instanceof基于正则表达式的matches运算符。
```java?linenums
// evaluates to false
boolean falseValue = parser.parseExpression(
        "'xyz' instanceof T(Integer)").getValue(Boolean.class);

// evaluates to true
boolean trueValue = parser.parseExpression(
        "'5.00' matches '^-?\\d+(\\.\\d{2})?$'").getValue(Boolean.class);

//evaluates to false
boolean falseValue = parser.parseExpression(
        "'5.0067' matches '^-?\\d+(\\.\\d{2})?$'").getValue(Boolean.class);

```
注意原始类型，因为它们立即被装箱到包装类型，因此在1 instanceof T(int)评估false时1 instanceof T(Integer) 评估为true，如预期的那样。
每个符号运算符也可以指定为纯字母等价运算符。这避免了所使用的符号对于嵌入表达式的文档类型具有特殊含义的问题（例如，XML文档）。文本等价物如下所示：lt（<），gt（>），le（<=），ge（>=），eq（==）， ne（!=），div（/），mod（%），not（!）。这些不区分大小写。

逻辑运算符
支持的逻辑运算符是和，或，而不是。它们的用途如下所示。
```java?linenums
// -- AND --

// evaluates to false
boolean falseValue = parser.parseExpression("true and false").getValue(Boolean.class);

// evaluates to true
String expression = "isMember('Nikola Tesla') and isMember('Mihajlo Pupin')";
boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

// -- OR --

// evaluates to true
boolean trueValue = parser.parseExpression("true or false").getValue(Boolean.class);

// evaluates to true
String expression = "isMember('Nikola Tesla') or isMember('Albert Einstein')";
boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

// -- NOT --

// evaluates to false
boolean falseValue = parser.parseExpression("!true").getValue(Boolean.class);

// -- AND and NOT --
String expression = "isMember('Nikola Tesla') and !isMember('Mihajlo Pupin')";
boolean falseValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);
```
数学运算符
加法运算符可用于数字和字符串。减法，乘法和除法只能用于数字。支持的其他数学运算符是模数（％）和指数幂（^）。强制执行标准运算符优先级。这些运算符如下所示。
```java?linenums
// Addition
int two = parser.parseExpression("1 + 1").getValue(Integer.class);  // 2

String testString = parser.parseExpression(
        "'test' + ' ' + 'string'").getValue(String.class);  // 'test string'

// Subtraction
int four = parser.parseExpression("1 - -3").getValue(Integer.class);  // 4

double d = parser.parseExpression("1000.00 - 1e4").getValue(Double.class);  // -9000

// Multiplication
int six = parser.parseExpression("-2 * -3").getValue(Integer.class);  // 6

double twentyFour = parser.parseExpression("2.0 * 3e0 * 4").getValue(Double.class);  // 24.0

// Division
int minusTwo = parser.parseExpression("6 / -3").getValue(Integer.class);  // -2

double one = parser.parseExpression("8.0 / 4e0 / 2").getValue(Double.class);  // 1.0

// Modulus
int three = parser.parseExpression("7 % 4").getValue(Integer.class);  // 3

int one = parser.parseExpression("8 / 5 % 2").getValue(Integer.class);  // 1

// Operator precedence
int minusTwentyOne = parser.parseExpression("1+2-3*8").getValue(Integer.class);  // -21
```
#### 4.4.8. 分配
通过使用赋值运算符来设置属性。这通常在调用期间完成，setValue但也可以在调用内完成getValue。
```java?linenums
Inventor inventor = new Inventor();
EvaluationContext context = SimpleEvaluationContext.forReadWriteDataBinding().build();

parser.parseExpression("Name").setValue(context, inventor, "Aleksandar Seovic");

// alternatively
String aleks = parser.parseExpression(
        "Name = 'Aleksandar Seovic'").getValue(context, inventor, String.class);
```
#### 4.4.9. 类型
特殊T运算符可用于指定java.lang.Class（类型）的实例 。也使用此运算符调用静态方法。在 StandardEvaluationContext使用TypeLocator查找类型和 StandardTypeLocator（可替换）是建立与java.lang包的理解。这意味着对java.lang中的类型的T（）引用不需要完全限定，但所有其他类型引用必须是。
```java?linenums
Class dateClass = parser.parseExpression("T(java.util.Date)").getValue(Class.class);

Class stringClass = parser.parseExpression("T(String)").getValue(Class.class);

boolean trueValue = parser.parseExpression(
        "T(java.math.RoundingMode).CEILING < T(java.math.RoundingMode).FLOOR")
        .getValue(Boolean.class);
```
#### 4.4.10 构造函数
可以使用new运算符调用构造函数。除了基本类型和String（可以使用int，float等）之外，应该使用全限定类名。

```java?linenums

Inventor einstein = p.parseExpression(
        "new org.spring.samples.spel.inventor.Inventor('Albert Einstein', 'German')")
        .getValue(Inventor.class);

//create new inventor instance within add method of List
p.parseExpression(
        "Members.add(new org.spring.samples.spel.inventor.Inventor(
            'Albert Einstein', 'German'))").getValue(societyContext);
```
#### 4.4.11. 变量
可以使用语法在表达式中引用变量#variableName。变量是使用方法设置setVariable上EvaluationContext实现。

```java?linenums
Inventor tesla = new Inventor("Nikola Tesla", "Serbian");

EvaluationContext context = SimpleEvaluationContext.forReadWriteDataBinding().build();
context.setVariable("newName", "Mike Tesla");

parser.parseExpression("Name = #newName").getValue(context, tesla);
System.out.println(tesla.getName())  // "Mike Tesla"
```
#this和#root变量
#this始终定义变量并引用当前评估对象（解析非限定引用）。#root始终定义变量并引用根上下文对象。尽管#this可能会在评估表达式的组件时发生变化，但#root始终引用根。
```java?linenums
// create an array of integers
List<Integer> primes = new ArrayList<Integer>();
primes.addAll(Arrays.asList(2,3,5,7,11,13,17));

// create parser and set variable 'primes' as the array of integers
ExpressionParser parser = new SpelExpressionParser();
EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataAccess();
context.setVariable("primes", primes);

// all prime numbers > 10 from the list (using selection ?{...})
// evaluates to [11, 13, 17]
List<Integer> primesGreaterThanTen = (List<Integer>) parser.parseExpression(
        "#primes.?[#this>10]").getValue(context);
```
#### 4.4.12. 功能
您可以通过注册可在表达式字符串中调用的用户定义函数来扩展SpEL。该功能通过注册EvaluationContext。

```java?linenums
Method method = ...;

EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();
context.setVariable("myFunction", method);
```
例如，给定一个反转字符串的实用方法如下所示：
```java?linenums
public abstract class StringUtils {

    public static String reverseString(String input) {
        StringBuilder backwards = new StringBuilder(input.length());
        for (int i = 0; i < input.length(); i++)
            backwards.append(input.charAt(input.length() - 1 - i));
        }
        return backwards.toString();
    }
}
```
然后可以如下注册和使用上述方法：
```java?linenums
ExpressionParser parser = new SpelExpressionParser();

EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();
context.setVariable("reverseString",
        StringUtils.class.getDeclaredMethod("reverseString", String.class));

String helloWorldReversed = parser.parseExpression(
        "#reverseString('hello')").getValue(context, String.class);
```
#### 4.4.13. Bean引用
如果已使用bean解析器配置了评估上下文，则可以使用该@符号从表达式中查找bean 。

```java?linenums
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();
context.setBeanResolver(new MyBeanResolver());

// This will end up calling resolve(context,"foo") on MyBeanResolver during evaluation
Object bean = parser.parseExpression("@foo").getValue(context);
```
要访问工厂bean本身，bean名称应该以&符号为前缀。
```java?linenums
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();
context.setBeanResolver(new MyBeanResolver());

// This will end up calling resolve(context,"&foo") on MyBeanResolver during evaluation
Object bean = parser.parseExpression("&foo").getValue(context);
```
#### 4.4.14. 三元运算符（If-Then-Else）
您可以使用三元运算符在表达式中执行if-then-else条件逻辑。一个最小的例子是：

```java?linenums
String falseString = parser.parseExpression(
        "false ? 'trueExp' : 'falseExp'").getValue(String.class);

```
在这种情况下，boolean false导致返回字符串值'falseExp'。一个更现实的例子如下所示。
```java?linenums
parser.parseExpression("Name").setValue(societyContext, "IEEE");
societyContext.setVariable("queryName", "Nikola Tesla");

expression = "isMember(#queryName)? #queryName + ' is a member of the ' " +
        "+ Name + ' Society' : #queryName + ' is not a member of the ' + Name + ' Society'";

String queryResultString = parser.parseExpression(expression)
        .getValue(societyContext, String.class);
// queryResultString = "Nikola Tesla is a member of the IEEE Society"
```
另请参阅Elvis运算符的下一节，了解三元运算符的更短语法。

#### 4.4.15. 猫王运营商
Elvis运算符是三元运算符语法的缩写，用于 Groovy语言。使用三元运算符语法，您通常必须重复两次变量，例如：

```java?linenums
String name = "Elvis Presley";
String displayName = (name != null ? name : "Unknown");
```
相反，您可以使用Elvis运算符，该运算符的名称与Elvis的发型相似。

```java?linenums
ExpressionParser parser = new SpelExpressionParser();

String name = parser.parseExpression("name?:'Unknown'").getValue(String.class);
System.out.println(name);  // 'Unknown'
```
这是一个更复杂的例子。

```java?linenums
ExpressionParser parser = new SpelExpressionParser();
EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();

Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
String name = parser.parseExpression("Name?:'Elvis Presley'").getValue(context, tesla, String.class);
System.out.println(name);  // Nikola Tesla

tesla.setName(null);
name = parser.parseExpression("Name?:'Elvis Presley'").getValue(context, tesla, String.class);
System.out.println(name);  // Elvis Presley
```
#### 4.4.16. 安全导航操作员
航行安全的操作使用，以避免NullPointerException与来自Groovy的 语言。通常，在引用对象时，可能需要在访问对象的方法或属性之前验证它是否为null。为避免这种情况，安全导航操作符将只返回null而不是抛出异常。
```java?linenums
ExpressionParser parser = new SpelExpressionParser();
EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();

Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
tesla.setPlaceOfBirth(new PlaceOfBirth("Smiljan"));

String city = parser.parseExpression("PlaceOfBirth?.City").getValue(context, tesla, String.class);
System.out.println(city);  // Smiljan

tesla.setPlaceOfBirth(null);
city = parser.parseExpression("PlaceOfBirth?.City").getValue(context, tesla, String.class);
System.out.println(city);  // null - does not throw NullPointerException!!!

```
```
Elvis运算符可用于在表达式中应用默认值，例如在 @Value表达式中：

@Value("#{systemProperties['pop3.port'] ?: 25}")
pop3.port如果已定义，则将注入系统属性，否则注入25。


```
#### 4.4.17. 收藏选择

Selection是一种强大的表达式语言功能，允许您通过从其条目中进行选择将某些源集合转换为另一个源集合。

选择使用语法.?[selectionExpression]。这将过滤集合并返回包含原始元素子集的新集合。例如，选择将使我们能够轻松获得塞尔维亚发明家的名单：
```java?linenums
List<Inventor> list = (List<Inventor>) parser.parseExpression(
        "Members.?[Nationality == 'Serbian']").getValue(societyContext);
```
可以在列表和地图上进行选择。在前一种情况下，针对每个单独的列表元素评估选择标准，而针对映射，针对每个映射条目（Java类型的对象）评估选择标准 Map.Entry。映射条目的键和值可作为选择中使用的属性进行访问。

此表达式将返回一个新映射，其中包含原始映射中条目值小于27的那些元素。
```java?linenums
Map newMap = parser.parseExpression("map.?[value<27]").getValue();
```
除了返回所有选定元素外，还可以只检索第一个或最后一个值。为了获得与选择匹配的第一条目，语法是.^[selectionExpression]同时获得语法 的最后匹配选择 .$[selectionExpression]。
#### 4.4.18. 收集投影
投影允许集合驱动子表达式的评估，结果是新集合。投影的语法是.![projectionExpression]。最容易理解的例子是，假设我们有一个发明者名单，但想要他们出生的城市名单。实际上，我们想要为发明人列表中的每个条目评估“placeOfBirth.city”。使用投影：

```java?linenums
// returns ['Smiljan', 'Idvor' ]
List placesOfBirth = (List)parser.parseExpression("Members.![placeOfBirth.city]");
```
地图也可用于驱动投影，在这种情况下，投影表达式将根据地图中的每个条目进行评估（表示为Java Map.Entry）。跨地图投影的结果是一个列表，其中包含对每个地图条目的投影表达式的评估。
#### 4.4.19. 表达模板
表达模板允许将文字文本与一个或多个评估块混合。每个评估块都用您可以定义的前缀和后缀字符分隔，一个常见的选择是#{ }用作分隔符。例如，
```java?linenums
String randomPhrase = parser.parseExpression(
        "random number is #{T(java.lang.Math).random()}",
        new TemplateParserContext()).getValue(String.class);

// evaluates to "random number is 0.7038186818312008"

```
通过将文本文本'random number is '与在#{ }分隔符内部计算表达式的结果连接来计算字符串，在本例中是调用该random()方法的结果。该方法的第二个参数parseExpression() 是类型ParserContext。该ParserContext接口用于影响表达式的解析方式，以支持表达式模板功能。定义TemplateParserContext如下所示。

```java?linenums
public class TemplateParserContext implements ParserContext {

    public String getExpressionPrefix() {
        return "#{";
    }

    public String getExpressionSuffix() {
        return "}";
    }

    public boolean isTemplate() {
        return true;
    }
}
```
### 4.5. 示例中使用的类
Inventor.java
```java?linenums
package org.spring.samples.spel.inventor;

import java.util.Date;
import java.util.GregorianCalendar;

public class Inventor {

    private String name;
    private String nationality;
    private String[] inventions;
    private Date birthdate;
    private PlaceOfBirth placeOfBirth;

    public Inventor(String name, String nationality) {
        GregorianCalendar c= new GregorianCalendar();
        this.name = name;
        this.nationality = nationality;
        this.birthdate = c.getTime();
    }

    public Inventor(String name, Date birthdate, String nationality) {
        this.name = name;
        this.nationality = nationality;
        this.birthdate = birthdate;
    }

    public Inventor() {
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getNationality() {
        return nationality;
    }

    public void setNationality(String nationality) {
        this.nationality = nationality;
    }

    public Date getBirthdate() {
        return birthdate;
    }

    public void setBirthdate(Date birthdate) {
        this.birthdate = birthdate;
    }

    public PlaceOfBirth getPlaceOfBirth() {
        return placeOfBirth;
    }

    public void setPlaceOfBirth(PlaceOfBirth placeOfBirth) {
        this.placeOfBirth = placeOfBirth;
    }

    public void setInventions(String[] inventions) {
        this.inventions = inventions;
    }

    public String[] getInventions() {
        return inventions;
    }
}
```
PlaceOfBirth.java
```java?linenums
package org.spring.samples.spel.inventor;

public class PlaceOfBirth {

    private String city;
    private String country;

    public PlaceOfBirth(String city) {
        this.city=city;
    }

    public PlaceOfBirth(String city, String country) {
        this(city);
        this.country = country;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String s) {
        this.city = s;
    }

    public String getCountry() {
        return country;
    }

    public void setCountry(String country) {
        this.country = country;
    }

}
```
Society.java
```java?linenums
package org.spring.samples.spel.inventor;

import java.util.*;

public class Society {

    private String name;

    public static String Advisors = "advisors";
    public static String President = "president";

    private List<Inventor> members = new ArrayList<Inventor>();
    private Map officers = new HashMap();

    public List getMembers() {
        return members;
    }

    public Map getOfficers() {
        return officers;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public boolean isMember(String name) {
        for (Inventor inventor : members) {
            if (inventor.getName().equals(name)) {
                return true;
            }
        }
        return false;
    }

}
```
## 5. 使用Spring进行面向方面的编程
### 5.1. 介绍
面向方面编程（AOP）通过提供另一种思考程序结构的方式来补充面向对象编程（OOP）。OOP中模块化的关键单元是类，而在AOP中，模块化单元是方面。方面可以实现关注的模块化，例如跨越多种类型和对象的事务管理。（这种担忧通常被称为AOP文献中的横切关注点。）

Spring的一个关键组件是AOP框架。虽然Spring IoC容器不依赖于AOP，但是如果您不想使用AOP，则意味着您不需要使用AOP，AOP补充了Spring IoC以提供非常强大的中间件解决方案。
```
Spring 2.0+ AOP
Spring 2.0引入了一种使用基于模式的方法或@AspectJ注释样式编写自定义方面的更简单，更强大的方法。这两种样式都提供完全类型的建议和AspectJ切入点语言的使用，同时仍然使用Spring AOP进行编织。

本章将讨论Spring 2.0+模式和基于@ AspectJ的AOP支持。下一章将讨论较低级别的AOP支持，如Spring 1.2应用程序中常见的那样。
```
AOP在Spring Framework中用于......

- ...提供声明性企业服务，尤其是作为EJB声明性服务的替代品。最重要的此类服务是 声明式事务管理。

- ...允许用户实现自定义方面，补充他们使用AOP的OOP。

如果您只对通用声明性服务或其他预先打包的声明性中间件服务（如池）感兴趣，则无需直接使用Spring AOP，并且可以跳过本章的大部分内容。
#### 5.1.1. AOP概念
让我们从定义一些中心AOP概念和术语开始。这些术语不是特定于Spring的......不幸的是，AOP术语不是特别直观; 但是，如果Spring使用自己的术语，那将更加令人困惑。

- 方面：跨越多个类别的关注点的模块化。事务管理是企业Java应用程序中横切关注点的一个很好的例子。在Spring AOP中，方面是使用常规类（基于模式的方法）或使用@Aspect注释（@AspectJ样式）注释的常规类来实现的 。

- 连接点：程序执行期间的一个点，例如方法的执行或异常的处理。在Spring AOP中，连接点始终 表示方法执行。

- 建议：某个方面在特定连接点采取的操作。不同类型的建议包括“周围”，“之前”和“之后”建议。（建议类型将在下面讨论。）许多AOP框架（包括Spring）将建议建模为拦截器，在连接点周围维护一系列拦截器。

- 切入点：匹配连接点的谓词。建议与切入点表达式相关联，并在切入点匹配的任何连接点处运行（例如，执行具有特定名称的方法）。由切入点表达式匹配的连接点的概念是AOP的核心，而Spring默认使用AspectJ切入点表达式语言。

- 简介：代表类型声明其他方法或字段。Spring AOP允许您向任何建议的对象引入新接口（和相应的实现）。例如，您可以使用简介使bean实现 IsModified接口，以简化缓存。（引言在AspectJ社区中称为类型间声明。）

- 目标对象：由一个或多个方面建议的对象。也称为建议对象。由于Spring AOP是使用运行时代理实现的，因此该对象始终是代理对象。

- AOP代理：由AOP框架创建的对象，用于实现方面契约（建议方法执行等）。在Spring Framework中，AOP代理将是JDK动态代理或CGLIB代理。

- 编织：将方面与其他应用程序类型或对象链接以创建建议对象。这可以在编译时（例如使用AspectJ编译器），加载时间或在运行时完成。与其他纯Java AOP框架一样，Spring AOP在运行时执行编织。

建议类型：

- 建议之前：在连接点之前执行但不能阻止执行流程进入连接点的建议（除非它抛出异常）。

- 返回建议后：在连接点正常完成后执行的建议：例如，如果方法返回而不抛出异常。

- 抛出建议后：如果方法通过抛出异常退出，则执行建议。

- （最终）建议之后：无论连接点退出的方式（正常或异常返回），都要执行建议。

- 围绕建议：围绕连接点的建议，例如方法调用。这是最有力的建议。around通知可以在方法调用之前和之后执行自定义行为。它还负责选择是继续加入点还是通过返回自己的返回值或抛出异常来快速建议的方法执行。

围绕建议是最一般的建议。由于Spring AOP（如AspectJ）提供了全方位的建议类型，因此我们建议您使用可以实现所需行为的最不强大的建议类型。例如，如果您只需要使用方法的返回值更新缓存，那么最好在实现返回后的建议而不是周围的建议，尽管周围的建议可以完成同样的事情。使用最具体的建议类型可以提供更简单的编程模型，减少错误的可能性。例如，您不需要proceed() 在JoinPointused for around advice 上调用该方法，因此无法调用它。

在Spring 2.0中，所有通知参数都是静态类型的，因此您可以使用相应类型的建议参数（例如，方法执行的返回值的类型）而不是Object数组。

通过切入点匹配的连接点的概念是AOP的关键，它将其与仅提供拦截的旧技术区分开来。切入点使建议能够独立于面向对象的层次结构进行定位。例如，提供声明式事务管理的around建议可以应用于跨越多个对象的一组方法（例如服务层中的所有业务操作）。

#### 5.1.2. Spring AOP的功能和目标
Spring AOP是用纯Java实现的。不需要特殊的编译过程。Spring AOP不需要控制类加载器层次结构，因此适合在Servlet容器或应用程序服务器中使用。

Spring AOP目前仅支持方法执行连接点（建议在Spring bean上执行方法）。虽然可以在不破坏核心Spring AOP API的情况下添加对字段拦截的支持，但未实现字段拦截。如果您需要建议字段访问和更新连接点，请考虑使用AspectJ等语言。

Spring AOP的AOP方法与大多数其他AOP框架的方法不同。目的不是提供最完整的AOP实现（尽管Spring AOP非常强大）; 它是在AOP实现和Spring IoC之间提供紧密集成，以帮助解决企业应用程序中的常见问题。

因此，例如，Spring Framework的AOP功能通常与Spring IoC容器一起使用。使用普通bean定义语法配置方面（尽管这允许强大的“自动代理”功能）：这是与其他AOP实现的重要区别。使用Spring AOP可以轻松或高效地执行某些操作，例如建议非常细粒度的对象（例如域对象）：在这种情况下，AspectJ是最佳选择。但是，我们的经验是，Spring AOP为适合AOP的企业Java应用程序中的大多数问题提供了出色的解决方案。

Spring AOP永远不会努力与AspectJ竞争，以提供全面的AOP解决方案。我们相信像Spring AOP这样的基于代理的框架和像AspectJ这样的完整框架都很有价值，而且它们是互补的，而不是竞争。Spring将Spring AOP和IoC与AspectJ无缝集成，以便在一致的基于Spring的应用程序架构中满足AOP的所有使用需求。此集成不会影响Spring AOP API或AOP Alliance API：Spring AOP保持向后兼容。有关Spring AOP API的讨论，请参阅以下章节。
```
Spring框架的核心原则之一是非侵入性 ; 这是因为您不应该被迫在您的业务/域模型中引入特定于框架的类和接口。但是，在某些地方，Spring Framework确实为您提供了将Spring Framework特定的依赖项引入代码库的选项：为您提供此类选项的基本原理是因为在某些情况下，它可能更容易阅读或编写某些特定的部分以这种方式的功能。Spring Framework（几乎）总是为您提供选择：您可以自由决定哪种选项最适合您的特定用例或场景。

与本章相关的一个选择是选择哪种AOP框架（以及哪种AOP样式）。您可以选择AspectJ和/或Spring AOP，也可以选择@AspectJ注释样式方法或Spring XML配置样式方法。本章首先选择引入@ AspectJ风格的方法，这一事实不应被视为Spring团队倾向于采用Spring XML配置风格的@AspectJ注释风格方法。

请参阅选择使用哪种AOP声明样式，以更全面地讨论每种样式的原因和原因。
```
#### 5.1.3. AOP代理
Spring AOP默认使用AOP代理的标准JDK 动态代理。这使得任何接口（或接口集）都可以被代理。

Spring AOP也可以使用CGLIB代理。这是代理类而不是接口所必需的。如果业务对象未实现接口，则默认使用CGLIB。因为优良的做法是编程接口而不是类; 业务类通常会实现一个或多个业务接口。可以 强制使用CGLIB，在那些需要建议未在接口上声明的方法或需要将代理对象作为具体类型传递给方法的情况下（希望很少见）。

掌握Spring AOP是基于代理的这一事实非常重要。请参阅 了解AOP代理，以全面了解此实现细节的实际含义。
### 5.2. @AspectJ支持
@AspectJ指的是将方面声明为使用注释注释的常规Java类的样式。作为AspectJ 5版本的一部分，AspectJ项目引入了@AspectJ样式 。Spring使用AspectJ提供的库来解释与AspectJ 5相同的注释，用于切入点解析和匹配。AOP运行时仍然是纯Spring AOP，并且不依赖于AspectJ编译器或weaver。
```
使用AspectJ编译器和weaver可以使用完整的AspectJ语言，并在使用AspectJ和Spring应用程序时进行了讨论。
```
#### 5.2.1. 启用@AspectJ支持
要在Spring配置需要启用配置基于@AspectJ方面的Spring AOP和Spring支持使用@AspectJ切面，自动代理基于它们是否被那些方面建议豆。通过autoproxying，我们的意思是如果Spring确定bean被一个或多个方面建议，它将自动生成该bean的代理以拦截方法调用并确保根据需要执行建议。

可以使用XML或Java样式配置启用@AspectJ支持。在任何一种情况下，您还需要确保AspectJ的aspectjweaver.jar库位于应用程序的类路径中（版本1.8或更高版本）。此库可在'lib'AspectJ分发的 目录中或通过Maven Central存储库获得。

使用Java配置启用@AspectJ支持
要使用Java启用@AspectJ支持，请@Configuration添加@EnableAspectJAutoProxy 注释：
```java?linenums
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}
```
使用XML配置启用@AspectJ支持
要使用基于XML的配置启用@AspectJ支持，请使用以下aop:aspectj-autoproxy 元素：

```xml
<aop:aspectj-autoproxy/>
```
这假设您正在使用基于XML架构的配置中描述的架构支持 。有关如何在 命名空间中导入标记，请参阅 AOP架构aop。
#### 5.2.2. 宣布一个方面
在启用@AspectJ支持的情况下，在应用程序上下文中定义的具有@AspectJ方面（具有@Aspect注释）的类的任何bean 将由Spring自动检测并用于配置Spring AOP。以下示例显示了非常有用的方面所需的最小定义：

应用程序上下文中的常规bean定义，指向具有@Aspect注释的bean类：
```xml
<bean id="myAspect" class="org.xyz.NotVeryUsefulAspect">
    <!-- configure properties of aspect here as normal -->
</bean>
```
和NotVeryUsefulAspect类定义，注释 org.aspectj.lang.annotation.Aspect注释;
```java?linenums
package org.xyz;
import org.aspectj.lang.annotation.Aspect;

@Aspect
public class NotVeryUsefulAspect {

}
```
方面（带有注释的类@Aspect）可能具有与任何其他类一样的方法和字段。它们还可能包含切入点，建议和引入（类型间）声明。
```
通过组件扫描自动检测方面
您可以在Spring XML配置中将方面类注册为常规bean，或者通过类路径扫描自动检测它们 - 就像任何其他Spring管理的bean一样。但是，请注意，@Aspect注解是不足够的classpath中自动检测：为了这个目的，你需要添加一个单独的@Component注释（或可选择地有资格，按照Spring的组件扫描仪的规则自定义构造型注解）。
```
```
与其他方面的方面建议？
在Spring AOP，它是不是可以有自己的方面从其他方面意见的目标。类上的@Aspect注释将其标记为方面，因此将其从自动代理中排除。
```
#### 5.2.3. 宣布切入点
回想一下，切入点确定了感兴趣的连接点，从而使我们能够控制建议何时执行。Spring AOP仅支持Spring bean的方法执行连接点，因此您可以将切入点视为匹配Spring bean上方法的执行。切入点声明有两个部分：一个包含名称和任何参数的签名，以及一个精确确定我们感兴趣的方法执行的切入点表达式。在AOP的@AspectJ注释样式中，切入点签名由常规方法提供定义，并使用@Pointcut注释指示切入点表达式（用作切入点签名的方法 必须具有void返回类型）。

一个示例将有助于区分切入点签名和切入点表达式。以下示例定义了一个名为的切入点'anyOldTransfer'，它将匹配任何名为的方法的执行'transfer'：
```java?linenums
@Pointcut("execution(* transfer(..))")// the pointcut expression
private void anyOldTransfer() {}// the pointcut signature
```
形成@Pointcut注释值的切入点表达式是常规的AspectJ 5切入点表达式。有关AspectJ的切入点语言的完整讨论，请参阅AspectJ编程指南（和扩展， AspectJ 5开发人员笔记本）或AspectJ上的一本书，例如Colyer等的“Eclipse AspectJ”。人。或Ramnivas Laddad的“AspectJ in Action”。

支持的切入点指示符
Spring AOP支持以下AspectJ切入点指示符（PCD）用于切入点表达式：

```
其他切入点类型
完整的AspectJ切入点语言支持Spring中不支持的其他切入点指示符。这些是：call, get, set, preinitialization, staticinitialization, initialization, handler, adviceexecution, withincode, cflow, cflowbelow, if, @this和@withincode。在Spring AOP解释的切入点表达式中使用这些切入点指示符将导致IllegalArgumentException抛出。

Spring AOP支持的切入点指示符集可以在将来的版本中进行扩展，以支持更多的AspectJ切入点指示符。
```
- 执行 - 对于匹配方法执行连接点，这是您在使用Spring AOP时将使用的主要切入点指示符

- within - 限制匹配某些类型中的连接点（只是在使用Spring AOP时执行在匹配类型中声明的方法）

- this - 限制匹配连接点（使用Spring AOP时执行方法），其中bean引用（Spring AOP代理）是给定类型的实例

- target - 限制匹配到连接点（使用Spring AOP时执行方法），其中目标对象（被代理的应用程序对象）是给定类型的实例

- args - 限制匹配连接点（使用Spring AOP时执行方法），其中参数是给定类型的实例

- @target - 限制匹配连接点（使用Spring AOP时执行方法），其中执行对象的类具有给定类型的注释

- @args - 限制匹配到连接点（使用Spring AOP时执行方法），其中传递的实际参数的运行时类型具有给定类型的注释

- @within - 限制匹配以连接具有给定注释的类型中的点（使用Spring AOP时在具有给定注释的类型中声明的方法的执行）

- @annotation - 限制匹配到连接点的主题，其中连接点的主题（在Spring AOP中执行的方法）具有给定的注释

由于Spring AOP仅限制与方法执行连接点的匹配，因此上面对切入点指示符的讨论给出了比在AspectJ编程指南中找到的更窄的定义。除此之外，AspectJ本身具有基于类型的语义，在执行的连接点this和target指代相同的对象-对象执行方法。Spring AOP是一个基于代理的系统，它区分代理对象本身（绑定到this）和代理后面的目标对象（绑定到target）。
```
由于Spring的AOP框架基于代理的特性，目标对象内的调用根据定义不会被截获。对于JDK代理，只能拦截代理上的公共接口方法调用。使用CGLIB，代理上的公共和受保护方法调用将被拦截，甚至包括必要的包可见方法。但是，通过代理进行的常见交互应始终通过公共签名进行设计。

请注意，切入点定义通常与任何截获的方法匹配。如果切入点严格意义上是公开的，即使在通过代理进行潜在非公共交互的CGLIB代理方案中，也需要相应地定义切入点。

如果您的拦截需要包括目标类中的方法调用甚至构造函数，请考虑使用Spring驱动的本机AspectJ编织而不是Spring的基于代理的AOP框架。这构成了具有不同特征的不同AOP使用模式，因此在做出决定之前一定要先熟悉编织。
```
Spring AOP还支持另一个名为的PCD bean。此PCD允许您将连接点的匹配限制为特定的命名Spring bean，或限制为一组命名的Spring bean（使用通配符时）。该beanPCD具有下列形式：

```java?linenums
bean(idOrNameOfBean)
```
该idOrNameOfBean令牌可以是任何Spring bean的名字：使用限定通配符*提供的性格，所以如果你建立一些命名约定，你的Spring豆你可以很容易地编写一个beanPCD表达挑出来。与其他切入点指示符的情况一样，beanPCD可以被&&'，||'和！（否定）也是。
```
请注意，beanPCD 仅在Spring AOP中受支持 - 而不是在原生AspectJ编织中。它是AspectJ定义的标准PCD的Spring特定扩展，因此不适用于@Aspect模型中声明的方面。

该beanPCD的操作实例级别（建设于Spring bean的概念），而不是仅在类型级别（这是基于什么编织的AOP仅限于）。基于实例的切入点指示符是Spring基于代理的AOP框架的一种特殊功能，它与Spring bean工厂紧密集成，通过名称可以自然而直接地识别特定的bean。
```
结合切入点表达式
可以使用'&&'，'||'组合切入点表达式 和'！'。也可以通过名称引用切入点表达式。以下示例显示了三个切入点表达式:( anyPublicOperation如果方法执行连接点表示任何公共方法的执行，则匹配）; inTrading（如果方法执行在交易模块中，tradingOperation则匹配），以及（如果方法执行表示交易模块中的任何公共方法，则匹配）。

```java?linenums
@Pointcut("execution(public * *(..))")
private void anyPublicOperation() {}

@Pointcut("within(com.xyz.someapp.trading..*)")
private void inTrading() {}

@Pointcut("anyPublicOperation() && inTrading()")
private void tradingOperation() {}
```
如上所示，最佳实践是从较小的命名组件构建更复杂的切入点表达式。当按名称引用切入点时，适用普通的Java可见性规则（您可以看到相同类型的私有切入点，层次结构中的受保护切入点，任何地方的公共切入点等等）。可见性不会影响切入点 匹配。

共享通用切入点定义
使用企业应用程序时，您经常需要从几个方面引用应用程序的模块和特定的操作集。我们建议定义一个“SystemArchitecture”方面，为此目的捕获常见的切入点表达式。典型的这种方面看起来如下：

```java?linenums
package com.xyz.someapp;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class SystemArchitecture {

    /**
     * A join point is in the web layer if the method is defined
     * in a type in the com.xyz.someapp.web package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.someapp.web..*)")
    public void inWebLayer() {}

    /**
     * A join point is in the service layer if the method is defined
     * in a type in the com.xyz.someapp.service package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.someapp.service..*)")
    public void inServiceLayer() {}

    /**
     * A join point is in the data access layer if the method is defined
     * in a type in the com.xyz.someapp.dao package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.someapp.dao..*)")
    public void inDataAccessLayer() {}

    /**
     * A business service is the execution of any method defined on a service
     * interface. This definition assumes that interfaces are placed in the
     * "service" package, and that implementation types are in sub-packages.
     *
     * If you group service interfaces by functional area (for example,
     * in packages com.xyz.someapp.abc.service and com.xyz.someapp.def.service) then
     * the pointcut expression "execution(* com.xyz.someapp..service.*.*(..))"
     * could be used instead.
     *
     * Alternatively, you can write the expression using the 'bean'
     * PCD, like so "bean(*Service)". (This assumes that you have
     * named your Spring service beans in a consistent fashion.)
     */
    @Pointcut("execution(* com.xyz.someapp..service.*.*(..))")
    public void businessService() {}

    /**
     * A data access operation is the execution of any method defined on a
     * dao interface. This definition assumes that interfaces are placed in the
     * "dao" package, and that implementation types are in sub-packages.
     */
    @Pointcut("execution(* com.xyz.someapp.dao.*.*(..))")
    public void dataAccessOperation() {}

}
```
在这样的方面定义的切入点可以被引用到您需要切入点表达式的任何地方。例如，要使服务层具有事务性，您可以编写：
```xml
<aop:config>
    <aop:advisor
        pointcut="com.xyz.someapp.SystemArchitecture.businessService()"
        advice-ref="tx-advice"/>
</aop:config>

<tx:advice id="tx-advice">
    <tx:attributes>
        <tx:method name="*" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>
```
的<aop:config>和<aop:advisor>元件在讨论基于模式的AOP支持。事务管理中讨论了事务元素。

例子
Spring AOP用户可能execution最常使用切入点指示符。执行表达式的格式是：

```java?linenums
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern)
            throws-pattern?)
```
除返回类型模式（上面的代码段中的ret-type-pattern），名称模式和参数模式之外的所有部分都是可选的。返回类型模式确定方法的返回类型必须是什么才能匹配连接点。最常用的*是作为返回类型模式，它匹配任何返回类型。仅当方法返回给定类型时，完全限定类型名称才匹配。名称模式与方法名称匹配。您可以将*通配符用作名称模式的全部或部分。如果指定声明类型模式，则包括尾部.以将其连接到名称模式组件。参数模式稍微复杂一些：()匹配一个不带参数的方法，而(..)匹配任意数量的参数（零或更多）。模式(*)匹配采用任何类型的一个参数的(*,String)方法， 匹配采用两个参数的方法，第一个可以是任何类型，第二个必须是String。有关更多信息，请参阅AspectJ编程指南的 语言语义部分。

下面给出了常见切入点表达式的一些示例。

- 执行任何公共方法：
```java?linenums
execution(public * *(..))
```
- 名称以“set”开头的任何方法的执行：
```java?linenums
execution(* set*(..))
```
- 执行AccountService接口定义的任何方法：
```java?linenums
execution(* com.xyz.service.AccountService.*(..))
```
- 执行服务包中定义的任何方法：
```java?linenums
execution(* com.xyz.service.*.*(..))
```
- 执行服务包或子包中定义的任何方法：
```java?linenums
execution(* com.xyz.service..*.*(..))
```
- 服务包中的任何连接点（仅在Spring AOP中执行方法）：
```java?linenums
within(com.xyz.service.*)
```
- 服务包或子包中的任何连接点（仅在Spring AOP中执行方法）：
```java?linenums
within(com.xyz.service..*)
```
- 代理实现AccountService接口的任何连接点（仅在Spring AOP中执行方法） ：
```java?linenums
this(com.xyz.service.AccountService)
```
```
'this'更常用于绑定形式： - 请参阅以下有关如何在建议体中提供代理对象的建议。
```
- 目标对象实现AccountService接口的任何连接点（仅在Spring AOP中执行方法）：
```java?linenums
target(com.xyz.service.AccountService)
```
```
	
'target'更常用于绑定形式： - 请参阅以下有关如何在建议体中提供目标对象的建议。
```
- 任何连接点（仅在Spring AOP中执行的方法），它接受一个参数，并且在运行时传递的参数是Serializable：
```java?linenums
args(java.io.Serializable)
```
```
'args'更常用于绑定形式： - 请参阅以下有关如何在建议体中提供方法参数的建议。
```
请注意，此示例中给出的切入点不同于execution(* *(java.io.Serializable))：如果在运行时传递的参数是Serializable，则args版本匹配，如果方法签名声明了单个参数类型，则执行版本匹配Serializable。

- 目标对象具有@Transactional注释的任何连接点（仅在Spring AOP中执行方法） ：
```java?linenums
@target(org.springframework.transaction.annotation.Transactional)
```
```
'@target'也可以用于绑定形式： - 请参阅以下有关如何在建议体中提供注释对象的建议。
```
- 任何连接点（仅在Spring AOP中执行方法），其中目标对象的声明类型具有@Transactional注释：
```java?linenums
@within(org.springframework.transaction.annotation.Transactional)
```
```
'@within'也可以用于绑定形式： - 请参阅以下有关如何在建议体中提供注释对象的建议。
```
-任何连接点（仅在Spring AOP中执行方法），其中执行方法具有 @Transactional注释：
```java?linenums
@annotation(org.springframework.transaction.annotation.Transactional)
```
```
'@annotation'也可以用于绑定形式： - 请参阅以下有关如何在建议体中提供注释对象的建议。
```

- 任何连接点（仅在Spring AOP中执行的方法），它接受一个参数，并且传递的参数的运行时类型具有@Classified注释：
```java?linenums
@args(com.xyz.security.Classified)
```
```
'@ args'也可以用于绑定形式： - 请参阅以下有关如何在建议体中提供注释对象的建议。
```
- 名为的Spring bean上的任何连接点（仅在Spring AOP中执行方法） tradeService：

```java?linenums
bean(tradeService)
```
- 具有与通配符表达式匹配的名称的Spring bean上的任何连接点（仅在Spring AOP中执行方法）*Service：
```java?linenums
bean(*Service)
```

写出好的切入点
在编译期间，AspectJ处理切入点以尝试和优化匹配性能。检查代码并确定每个连接点是否（静态地或动态地）匹配给定切入点是一个代价高昂的过程。（动态匹配意味着无法通过静态分析完全确定匹配，并且将在代码中放置测试以确定代码运行时是否存在实际匹配）。在第一次遇到切入点声明时，AspectJ会将其重写为匹配过程的最佳形式。这是什么意思？基本上，切入点在DNF（析取范式）中重写，并且切入点的组件被排序，以便首先检查那些评估成本更低的组件。

但是，AspectJ只能处理它所说的内容，并且为了获得最佳匹配性能，您应该考虑它们要实现的目标，并在定义中尽可能缩小匹配的搜索空间。现有的指示符自然分为三组：kinded，scoping和context：

- Kinded指示符是选择特定类型的连接点的指示符。例如：执行，获取，设置，调用，处理程序

- 范围界定指示符是选择一组感兴趣的连接点（可能是多种类型）的指示符。例如：内部，内部代码

- 上下文指示符是基于上下文匹配（并且可选地绑定）的指示符。例如：this，target，@ annotation

一个写得很好的切入点应该尝试包括至少前两种类型（kinded和scoping），而如果希望基于连接点上下文匹配，则可以包括上下文指示符，或者绑定该上下文以在建议中使用。只提供一个kinded指示符或仅提供一个上下文指示符将起作用，但由于所有额外的处理和分析，可能会影响编织性能（使用的时间和内存）。范围界定指示符非常快速匹配，它们的使用意味着AspectJ可以非常快速地消除不应该进一步处理的连接点组 - ​​这就是为什么一个好的切入点应该总是包括一个如果可能的原因。
#### 5.2.4. 宣布建议
建议与切入点表达式相关联，并在切入点匹配的方法执行之前，之后或周围运行。切入点表达式可以是对命名切入点的简单引用，也可以是在适当位置声明的切入点表达式。

在建议之前
在使用@Before注释在方面声明建议之前：
```java?linenums
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

    @Before("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }

}

```
如果使用就地切入点表达式，我们可以将上面的示例重写为：
```java?linenums
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

    @Before("execution(* com.xyz.myapp.dao.*.*(..))")
    public void doAccessCheck() {
        // ...
    }

}
```
回复建议后
返回建议后，匹配的方法执行正常返回。它使用@AfterReturning注释声明：
```java?linenums
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class AfterReturningExample {

    @AfterReturning("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }

}
```
```
注意：当然可以在同一方面内有多个建议声明和其他成员。我们只是在这些例子中展示了一条建议声明，专注于当时正在讨论的问题。

```
```java?linenums
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class AfterReturningExample {

    @AfterReturning(
        pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
        returning="retVal")
    public void doAccessCheck(Object retVal) {
        // ...
    }

}

```
returning属性中使用的名称必须与advice方法中的参数名称相对应。当方法执行返回时，返回值将作为相应的参数值传递给advice方法。甲returning子句也限制了只能匹配到返回指定类型的值（这些方法执行Object在这种情况下，这将匹配任何返回值）。

请注意，这是没有可能返回一个完全不同的参考使用后置通知时。

投掷建议后
抛出建议运行时，匹配的方法执行通过抛出异常退出。它使用@AfterThrowing注释声明：
```java?linenums
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;

@Aspect
public class AfterThrowingExample {

    @AfterThrowing("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doRecoveryActions() {
        // ...
    }

}
```
通常，您希望仅在抛出给定类型的异常时才运行建议，并且您还经常需要访问建议体中的抛出异常。使用该 throwing属性来限制匹配（如果需要，Throwable否则，将其用作异常类型）并将抛出的异常绑定到advice参数。


```java?linenums
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;

@Aspect
public class AfterThrowingExample {

    @AfterThrowing(
        pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
        throwing="ex")
    public void doRecoveryActions(DataAccessException ex) {
        // ...
    }

}
```
throwing属性中使用的名称必须与advice方法中的参数名称相对应。当通过抛出异常退出方法时，异常将作为相应的参数值传递给advice方法。甲throwing 子句也限制了只能匹配到抛出指定类型的异常（那些方法执行DataAccessException在这种情况下）。

经过（终于）建议
在（最终）建议运行之后，匹配的方法执行退出。它是使用@After注释声明的。在建议之后必须准备好处理正常和异常返回条件。它通常用于释放资源等。
```java?linenums
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.After;

@Aspect
public class AfterFinallyExample {

    @After("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doReleaseLock() {
        // ...
    }

}
```
围绕建议
最后一种建议是建议。周围的建议围绕匹配的方法执行运行。它有机会在方法执行之前和之后完成工作，并确定方法实际上何时，如何以及甚至是否实际执行。如果您需要以线程安全的方式（例如，启动和停止计时器）在方法执行之前和之后共享状态，则通常会使用around建议。始终使用满足您要求的最不强大的建议形式（即如果在建议之前，请不要使用简单的建议）。

使用@Around注释声明around建议。advice方法的第一个参数必须是type ProceedingJoinPoint。在建议的主体内，调用导致底层方法执行proceed()的ProceedingJoinPoint原因。该proceed方法也可以被称为传入Object[]- 数组中的值将在进行时用作方法执行的参数。
```
使用Object []调用时，proceed的行为与由AspectJ编译器编译的around建议的行为略有不同。对于使用传统AspectJ语言编写的周围建议，传递给proceed的参数数量必须与传递给around建议的参数数量（不是底层连接点所采用的参数数量）相匹配，并且传递给的值继续给定的参数位置取代了值绑定到的实体的连接点的原始值（如果现在没有意义，请不要担心！）。Spring采用的方法更简单，更好地匹配其基于代理的，仅执行语义。如果要编译为Spring编写的@AspectJ方面并使用带有AspectJ编译器和weaver的参数继续，则只需要了解这种差异。有一种方法可以编写在Spring AOP和AspectJ上100％兼容的方面，这将在下面的建议参数部分中讨论。
```
```java?linenums
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.ProceedingJoinPoint;

@Aspect
public class AroundExample {

    @Around("com.xyz.myapp.SystemArchitecture.businessService()")
    public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
        // start stopwatch
        Object retVal = pjp.proceed();
        // stop stopwatch
        return retVal;
    }

}

```
around通知返回的值将是方法调用者看到的返回值。例如，一个简单的缓存方面可以从缓存中返回一个值（如果有的话），如果没有，则调用proceed（）。请注意，可以在周围建议的主体内调用一次，多次或根本不调用，所有这些都是非常合法的。

建议参数
Spring提供完全类型的建议 - 这意味着您在建议签名中声明了所需的参数（正如我们在上面看到的返回和抛出示例所示），而不是一直使用Object[]数组。我们将看到如何在一瞬间为建议主体提供参数和其他上下文值。首先让我们看看如何编写通用建议，以便了解建议目前建议的方法。

访问当前的JoinPoint
任何通知方法都可以声明为它的第一个参数，一个类型的参数 org.aspectj.lang.JoinPoint（请注意，周围的建议需要声明类型的第一个参数ProceedingJoinPoint，它是一个子类JoinPoint。 JoinPoint接口提供了许多有用的方法，如getArgs()（返回方法）参数），getThis()（返回代理对象），getTarget()（返回目标对象），getSignature()（返回正在建议的方法的描述）和toString()（打印出被建议的方法的有用描述）。请查阅javadocs以获取完整的详细信息。

将参数传递给建议
我们已经看到了如何绑定返回的值或异常值（在返回之后和抛出建议之后使用）。要使参数值可用于建议体，您可以使用绑定形式args。如果在args表达式中使用参数名称代替类型名称，则在调用通知时，相应参数的值将作为参数值传递。一个例子应该使这更清楚。假设您要建议执行以Account对象作为第一个参数的dao操作，并且您需要访问建议体中的帐户。你可以写下面的内容：
```java?linenums

@Before("com.xyz.myapp.SystemArchitecture.dataAccessOperation() && args(account,..)")
public void validateAccount(Account account) {
    // ...
}
```
args(account,..)切入点表达式的一部分有两个目的：首先，它将匹配仅限于那些方法至少接受一个参数的方法执行，而传递给该参数的参数是一个实例Account; 其次，它Account通过account 参数使实际对象可用于建议。

另一种编写方法是声明一个切入点，Account 当它与连接点匹配时“提供” 对象值，然后从建议中引用指定的切入点。这看起来如下：

```java?linenums
@Pointcut("com.xyz.myapp.SystemArchitecture.dataAccessOperation() && args(account,..)")
private void accountDataAccessOperation(Account account) {}

@Before("accountDataAccessOperation(account)")
public void validateAccount(Account account) {
    // ...
}
```
感兴趣的读者再次参考AspectJ编程指南以获取更多详细信息。

代理对象（this），目标对象（target）和注释（@within, @target, @annotation, @args）都可以以类似的方式绑定。以下示例显示了如何匹配带@Auditable注释的注释方法的执行 ，并提取审计代码。

首先是@Auditable注释的定义：
```java?linenums
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Auditable {
    AuditCode value();
}
```
然后是与@Auditable方法执行相匹配的建议：

```java?linenums
@Before("com.xyz.lib.Pointcuts.anyPublicMethod() && @annotation(auditable)")
public void audit(Auditable auditable) {
    AuditCode code = auditable.value();
    // ...
}
```
建议参数和泛型
Spring AOP可以处理类声明和方法参数中使用的泛型。假设您有这样的泛型类型：


```java?linenumspublic interface Sample<T> {
    void sampleGenericMethod(T param);
    void sampleGenericCollectionMethod(Collection<T> param);
}
```
您可以通过简单地将advice参数键入要拦截方法的参数类型来限制方法类型拦截到某些参数类型：

```java?linenums
@Before("execution(* ..Sample+.sampleGenericMethod(*)) && args(param)")
public void beforeSampleMethod(MyType param) {
    // Advice implementation
}
```
正如我们上面已经讨论的那样，这是非常明显的。但是，值得指出的是，这不适用于通用集合。所以你不能像这样定义一个切入点：

```java?linenums
@Before("execution(* ..Sample+.sampleGenericCollectionMethod(*)) && args(param)")
public void beforeSampleMethod(Collection<MyType> param) {
    // Advice implementation
}
```
为了完成这项工作，我们必须检查集合中的每个元素，这是不合理的，因为我们也无法决定如何处理null一般的值。要实现与此类似的功能，您必须键入参数Collection<?>并手动检查元素的类型。

确定参数名称
通知调用中的参数绑定依赖于切入点表达式中使用的匹配名称与（advice和pointcut）方法签名中声明的参数名称。参数名无法通过Java反射来获取，所以Spring AOP使用如下的策略来确定参数名字：

- 如果用户明确指定了参数名称，则使用指定的参数名称：通知和切入点注释都有一个可选的“argNames”属性，可用于指定带注释的方法的参数名称 - 这些参数名字是在运行时可用。例如：
```java?linenums
@Before(value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && @annotation(auditable)",
        argNames="bean,auditable")
public void audit(Object bean, Auditable auditable) {
    AuditCode code = auditable.value();
    // ... use code and bean
}
```
如果第一个参数是的JoinPoint，ProceedingJoinPoint或 JoinPoint.StaticPart类型，你可以在“argNames”属性的值中省去参数的名字。例如，如果修改前面的建议以接收连接点对象，则“argNames”属性不需要包含它：

```java?linenums
@Before(value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && @annotation(auditable)",
        argNames="bean,auditable")
public void audit(JoinPoint jp, Object bean, Auditable auditable) {
    AuditCode code = auditable.value();
    // ... use code, bean, and jp
}
```
给出的第一个参数的特殊待遇JoinPoint， ProceedingJoinPoint和JoinPoint.StaticPart类型是不收取任何其它连接上下文的通知特别方便。在这种情况下，您可以简单地省略“argNames”属性。例如，以下建议无需声明“argNames”属性：
```java?linenums
@Before("com.xyz.lib.Pointcuts.anyPublicMethod()")
public void audit(JoinPoint jp) {
    // ... use jp
}
```
- 使用该'argNames'属性有点笨拙，因此如果'argNames'未指定该属性，则Spring AOP将查看该类的调试信息，并尝试从局部变量表中确定参数名称。只要使用调试信息（'-g:vars'至少）编译了类，就会出现此信息。使用此标志进行编译的后果是：（1）您的代码将更容易理解（逆向工程），（2）类文件大小将略微更大（通常无关紧要），（3）要删除的优化编译器不会应用未使用的局部变量。换句话说，使用此标志构建时不会遇到任何困难。

```
如果没有调试信息，AspectJ编译器（ajc）编译了@AspectJ方面，那么就不需要添加argNames属性，因为编译器将保留所需的信息。
```
- 如果代码编译时没有必要的调试信息，则Spring AOP将尝试推断绑定变量与参数的配对（例如，如果只有一个变量绑定在切入点表达式中，并且advice方法只接受一个参数，配对很明显！）。如果给定可用信息，变量的绑定是不明确的，那么AmbiguousBindingException将抛出一个。

- 如果所有上述策略都失败，那么IllegalArgumentException将抛出一个。

```java?linenums
@Around("execution(List<Account> find*(..)) && " +
        "com.xyz.myapp.SystemArchitecture.inDataAccessLayer() && " +
        "args(accountHolderNamePattern)")
public Object preProcessQueryPattern(ProceedingJoinPoint pjp,
        String accountHolderNamePattern) throws Throwable {
    String newPattern = preProcess(accountHolderNamePattern);
    return pjp.proceed(new Object[] {newPattern});
}
```
在许多情况下，无论如何你都会做这个绑定（如上例所示）。

建议订购
当多条建议都想在同一个连接点运行时会发生什么？Spring AOP遵循与AspectJ相同的优先级规则来确定建议执行的顺序。最高优先级的建议首先“在路上”（因此，给出两条之前的建议，优先级最高的建议首先运行）。从连接点“出路”，最高优先级建议最后运行（因此，给出两条后建议，具有最高优先级的建议将运行第二）。

当在不同方面定义的两条建议都需要在同一个连接点运行时，除非您另行指定，否则执行顺序是未定义的。您可以通过指定优先级来控制执行顺序。这是通过在方法类中实现org.springframework.core.Ordered接口或使用注释对其进行Order注释来以常规Spring方式完成的。给定两个方面，从Ordered.getValue()（或注释值）返回较低值的方面具有较高的优先级。

当在同一方面中定义的两条建议都需要在同一个连接点上运行时，排序是未定义的（因为没有办法通过反射为javac编译的类检索声明顺序）。考虑将这些建议方法折叠到每个方面类中每个连接点的一个建议方法中，或者将这些建议重构为单独的方面类 - 可以在方面级别进行排序。
#### 5.2.5. 简介
简介（在AspectJ中称为类型间声明）使方面能够声明建议对象实现给定接口，并代表这些对象提供该接口的实现。

使用@DeclareParents注释进行介绍。此批注用于声明匹配类型具有新父级（因此名称）。例如，给定接口UsageTracked和该接口的实现DefaultUsageTracked，以下方面声明服务接口的所有实现者也实现UsageTracked接口。（例如，为了通过JMX公开统计信息。）
```java?linenums
@Aspect
public class UsageTracking {

    @DeclareParents(value="com.xzy.myapp.service.*+", defaultImpl=DefaultUsageTracked.class)
    public static UsageTracked mixin;

    @Before("com.xyz.myapp.SystemArchitecture.businessService() && this(usageTracked)")
    public void recordUsage(UsageTracked usageTracked) {
        usageTracked.incrementUseCount();
    }

}
```
要实现的接口由注释字段的类型确定。注释的 value属性@DeclareParents是AspectJ类型模式： - 任何匹配类型的bean都将实现UsageTracked接口。请注意，在上面示例的before advice中，服务bean可以直接用作UsageTracked接口的实现。如果以编程方式访问bean，您将编写以下内容：
```java?linenums
UsageTracked usageTracked = (UsageTracked) context.getBean("myService");
```
#### 5.2.6. Aspect实例化模型
```
（这是一个高级主题，所以如果你刚刚开始使用AOP，你可以安全地跳过它直到以后。）
```
默认情况下，应用程序上下文中的每个方面都有一个实例。AspectJ将其称为单例实例化模型。它可以与其他的生命周期定义方面： - Spring支持AspectJ的perthis和pertarget 实例化模型（percflow, percflowbelow,和pertypewithin目前不支持）。

通过perthis在@Aspect 注释中指定子句来声明“perthis”方面。让我们看一个例子，然后我们将解释它是如何工作的。

```java?linenums
@Aspect("perthis(com.xyz.myapp.SystemArchitecture.businessService())")
public class MyAspect {

    private int someState;

    @Before(com.xyz.myapp.SystemArchitecture.businessService())
    public void recordServiceUsage() {
        // ...
    }

}
```
该'perthis'子句的作用是为执行业务服务的每个唯一服务对象创建一个方面实例（每个唯一对象在由切入点表达式匹配的连接点处绑定到'this'）。方法实例是在第一次在服务对象上调用方法时创建的。当服务对象超出范围时，该方面超出范围。在创建方面实例之前，其中没有任何建议执行。一旦创建了方面实例，在其中声明的通知将在匹配的连接点处执行，但仅在服务对象是与此方面相关联的服务对象时执行。有关per子句的更多信息，请参阅AspectJ编程指南。

该'pertarget'实例化样板工程完全相同的方式perthis，但在匹配的连接点，为每个唯一目标对象的一个方面的实例。
#### 5.2.7. 例
既然你已经看到了所有组成部分的工作方式，那就让我们把它们放在一起做一些有用的事情吧！

由于并发问题（例如，死锁失败者），业务服务的执行有时会失败。如果重试该操作，则很可能在下一轮成功。对于适合在这种情况下重试的业务服务（幂等操作不需要回到用户进行冲突解决），我们希望透明地重试操作以避免客户端看到 PessimisticLockingFailureException。这是明确跨越服务层中的多个服务的要求，因此是通过方面实现的理想选择。

因为我们想要重试操作，我们需要使用around建议，以便我们可以多次调用proceed。以下是基本方面实现的外观：

```java?linenums
@Aspect
public class ConcurrentOperationExecutor implements Ordered {

    private static final int DEFAULT_MAX_RETRIES = 2;

    private int maxRetries = DEFAULT_MAX_RETRIES;
    private int order = 1;

    public void setMaxRetries(int maxRetries) {
        this.maxRetries = maxRetries;
    }

    public int getOrder() {
        return this.order;
    }

    public void setOrder(int order) {
        this.order = order;
    }

    @Around("com.xyz.myapp.SystemArchitecture.businessService()")
    public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
        int numAttempts = 0;
        PessimisticLockingFailureException lockFailureException;
        do {
            numAttempts++;
            try {
                return pjp.proceed();
            }
            catch(PessimisticLockingFailureException ex) {
                lockFailureException = ex;
            }
        } while(numAttempts <= this.maxRetries);
        throw lockFailureException;
    }

}
```
请注意，方面实现了Ordered接口，因此我们可以将方面的优先级设置为高于事务通知（我们每次重试时都需要一个新的事务）。在maxRetries和order属性都可以在Spring中配置。主要行动发生在doConcurrentOperation周围的建议中。请注意，目前我们正在将重试逻辑应用于所有人businessService()s。我们试图继续进行，如果我们失败了，PessimisticLockingFailureException我们只需再试一次，除非我们已经用尽所有的重试尝试。

相应的Spring配置是：
```xml
<aop:aspectj-autoproxy/>

<bean id="concurrentOperationExecutor" class="com.xyz.myapp.service.impl.ConcurrentOperationExecutor">
    <property name="maxRetries" value="3"/>
    <property name="order" value="100"/>
</bean>
```
为了优化方面以便它只重试幂等操作，我们可以定义一个 Idempotent注释：

```java?linenums
@Retention(RetentionPolicy.RUNTIME)
public @interface Idempotent {
    // marker annotation
}
```
并使用注释来注释服务操作的实现。对方面的更改仅重试幂等操作只涉及改进切入点表达式，以便只有@Idempotent操作匹配：

```java?linenums
@Around("com.xyz.myapp.SystemArchitecture.businessService() && " +
        "@annotation(com.xyz.myapp.service.Idempotent)")
public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
    ...
}
```

### 5.3. 基于模式的AOP支持
如果您更喜欢基于XML的格式，那么Spring还支持使用新的“aop”命名空间标记定义方面。使用@AspectJ样式时，支持完全相同的切入点表达式和建议类型，因此在本节中我们将重点介绍新语法，并引用读者阅读上一节（@AspectJ支持）中的讨论以了解写作切入点表达式和建议参数的绑定。

要使用本节中描述的aop命名空间标记，您需要按照基于XML模式的配置中的spring-aop描述导入 模式。有关 如何在命名空间中导入标记，请参阅AOP架构。aop

在Spring配置中，所有aspect和advisor元素必须放在一个<aop:config>元素中（<aop:config>在应用程序上下文配置中可以有多个元素）。一个<aop:config>元素可以包含切入点，顾问和纵横元件（注意这些必须按照这个顺序进行声明）。
```
该<aop:config>风格的配置使得大量使用Spring的 自动代理机制。如果您已经通过使用BeanNameAutoProxyCreator或类似的方式使用显式自动代理，这可能会导致问题（例如建议不被编织） 。建议的使用模式是仅使用<aop:config>样式，或仅使用AutoProxyCreator样式。
```
#### 5.3.1. 宣布一个方面
使用模式支持，方面只是在Spring应用程序上下文中定义为bean的常规Java对象。状态和行为在对象的字段和方法中捕获，切入点和建议信息在XML中捕获。

使用<aop：aspect>元素声明方面，并使用以下ref属性引用辅助bean ：
```xml
<aop:config>
    <aop:aspect id="myAspect" ref="aBean">
        ...
    </aop:aspect>
</aop:config>

<bean id="aBean" class="...">
    ...
</bean>
```
支持方面（"aBean"在这种情况下）的bean 当然可以配置和依赖注入，就像任何其他Spring bean一样。

#### 5.3.2. 宣布切入点
可以在<aop：config>元素内声明命名切入点，使切入点定义能够跨多个方面和顾问程序共享。

表示服务层中任何业务服务执行的切入点可以定义如下：
```xml
<aop:config>

    <aop:pointcut id="businessService"
        expression="execution(* com.xyz.myapp.service.*.*(..))"/>

</aop:config>
```
请注意，切入点表达式本身使用与@AspectJ支持中描述的相同的AspectJ切入点表达式语言。如果使用基于模式的声明样式，则可以引用切入点表达式中类型（@Aspects）中定义的命名切入点。定义上述切入点的另一种方法是：

```xml

<aop:config>

    <aop:pointcut id="businessService"
        expression="com.xyz.myapp.SystemArchitecture.businessService()"/>

</aop:config>
```
假设您有共享公共切入点定义中SystemArchitecture描述的方面。

在方面内部声明切入点与声明顶级切入点非常相似：
```xml
<aop:config>

    <aop:aspect id="myAspect" ref="aBean">

        <aop:pointcut id="businessService"
            expression="execution(* com.xyz.myapp.service.*.*(..))"/>

        ...

    </aop:aspect>

</aop:config>
```
在@AspectJ方面大致相同的方式，使用基于模式的定义样式声明的切入点可以收集连接点上下文。例如，以下切入点将'this'对象收集为连接点上下文并将其传递给建议：
```xml
<aop:config>

    <aop:aspect id="myAspect" ref="aBean">

        <aop:pointcut id="businessService"
            expression="execution(* com.xyz.myapp.service.*.*(..)) &amp;&amp; this(service)"/>

        <aop:before pointcut-ref="businessService" method="monitor"/>

        ...

    </aop:aspect>

</aop:config>
```
必须通过包含匹配名称的参数来声明建议以接收收集的连接点上下文：
```java?linenums
public void monitor(Object service) {
    ...
}
```
当需要连接子表达式，&&是尴尬的XML文档中，所以关键字and，or和not可以代替使用&&，||和! 分别。例如，之前的切入点可能更好地写为：
```xml
<aop:config>

    <aop:aspect id="myAspect" ref="aBean">

        <aop:pointcut id="businessService"
            expression="execution(* com.xyz.myapp.service..(..)) and this(service)"/>

        <aop:before pointcut-ref="businessService" method="monitor"/>

        ...
    </aop:aspect>
</aop:config>
```
请注意，以这种方式定义的切入点由其XML ID引用，不能用作命名切入点来形成复合切入点。因此，基于模式的定义样式中的命名切入点支持比@AspectJ样式提供的更有限。
#### 5.3.3. 宣布建议
对于@AspectJ样式，支持相同的五种建议类型，它们具有完全相同的语义。

在建议之前
在匹配的方法执行之前运行建议之前。它在<aop:aspect>使用<aop：before>元素内部声明 。

```xml
<aop:aspect id="beforeExample" ref="aBean">

    <aop:before
        pointcut-ref="dataAccessOperation"
        method="doAccessCheck"/>

    ...

</aop:aspect>

```
这dataAccessOperation是在top（<aop:config>）级别定义的切入点的id 。要改为内联定义切入点，请使用pointcut-ref属性替换该pointcut属性：

```xml
<aop:aspect id="beforeExample" ref="aBean">

    <aop:before
        pointcut="execution(* com.xyz.myapp.dao.*.*(..))"
        method="doAccessCheck"/>

    ...

</aop:aspect>
```
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