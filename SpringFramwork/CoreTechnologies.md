
#核心技术
#####5.2.0.M2 版本

这部分文档涵盖了spring框架必备的所有技术
其中最为重要的是spring框架的控制反转（IOC）容器。spring对IOC容器的完善处理，涵盖了紧随其后的spring面向切面编程（AOP）的所有部分。
spring 框架有着自己的AOP框架，并且简单易懂，同时成功实现了JavaEE企业级开发中的80%的最佳实践。

同时Spring提供了覆盖与AspectJ（Java企业领域最成熟的功能最为丰富的AOP实现）集成的所有范围。

## 1.IoC容器
本章节将介绍Spring的控制反转（IoC）容器。
### 1.1SpringIoC容器与Bean简介
本小节主要接受Spring框架IoC的实现原理。IoC也称作依赖注入（DI）。这是一个当一个对象需要调用另一个目标依赖对象的时候只能通过工厂方法来获取目标对象的过程，在使用工厂方法的时候，可以传递传递带参构造器，或者使用无参构造器然后再调用目标对象的set方法。然后容器在创建bean的时候会注入这些目标依赖。这个过程相比与bean本身控制实例化或者通过直接构造类或使用诸如服务定位器模式之类的机制来解决其依赖项来说其顺序是相反的（因此被叫作控制反转）  
**org.springframework.beans**和**org.springframework.context**这两个包是Spring框架的的IoC容器的基本包。BeanFactory接口提供了一种能够管理任何类型对象的高级配置机制。 ApplicationContext是BeanFactory的子接口。它增加了如下的功能：
- 与Spring AOP便捷的集成
- 信息资源的处理（用于国际化）
- 事件发布
- 应用程序的特定上下文，例如用于Web应用程序的WebApplicationContext
简而言之，BeanFactory提供了配置框架和基本功能，而ApplicationContext则添加了更多企业开发中特定的功能。ApplicationContext是BeanFactory的完整超集，在本章中专门用于Spring的IoC容器的描述。针对BeanFactory（非ApplicationContext）的描述请参阅这里**链接**。
在Spring中构成应用程序的主干并由Spring IoC管理的对象背称作Bean。Bean是一个由Spring IoC容器实例化，组装，管理的对象。否则Bean只是应用程序中的一个简单的对象。Bean及其他们之间的依赖关系反映在容器所使用的配置元数据之中。
### 1.2容器概览
org.springframework.context.ApplicationContext接口即Spring IoC容器，负责实例化、配置、组装bean。容器通过读取配置元数据来获取有关需要实例化，配置，组装的对象的相关指令。配置元数据可以是xml文件，java注解或者是java代码。因此可以表达应用程序的对象，以及对象之间的丰富依赖关系。
Spring提供了ApplicationContext的若干接口实现。在独立的应用中，通常会使用ClassPathXmlApplicationContext或者FileSystemXmlApplicationContext实例。由于xml是定义元数据的传统方式，因此可以使用配置少量的xml文件来声明容器使用java注解或者java代码作为元数据定义的方式，以开启对这种两种定义方式的支持。
在大多数的应用程序中，不需要用户在代码中显式的实例化Spirng IoC容器的一个或多个实例。例如，在web应用场景中，在项目的web.xml文件中只需要模板化的简单的8行代码就足够了（具体请参考web应用程序的ApplicationContext实例）。如果你使用Spirng Tool Suit（一个基于Eclipse的开发工具），只需简单的点击几下鼠标就可以轻松的创建文件模板。
以下图从高层次概括的展示了Spirng是如何工作的。用户的应用程序与配置元数据相互结合，一边ApplicationContext创建和初始化后，用户拥有完全配置并且可执行的系统或者程序。
 #### 1.2.1配置元数据
如上图所示，Sring IoC使用配置的方式作为元数据。作为开发人员，这个配置元数据告诉Spring容器实例化、配置和组装应用程序中的对象。
通常，配置文件以简单直观的xml文件格式给出，本章的大部分内容会表述Spring IoC的关键概念已经功能。  
*基于xml的元数据并不是唯一允许的配置元数据的方式。Spring IoC容器本身完全与实际编写此配置元数据的方式解隅。目前，许多开发人员为其Spring应用程序选择基于Java的配置。*
有关在Spring容器中使用其他形式的元数据的信息可以参考一下内容：
- 基于注解的配置：Spring2.5引入了对基于注解的配置方式的支持
- 基于Java的配置：从Spring3.0开始，Spring JavaConfig项目中提供的许多功能都成为了Spring Framework的一部分。因此，开发人员可以使用Java而不是XML文件在应用程序类外部定义bean，参考@config，@bean，@Import，和@Dependency注解。   

Spring配置包含容器所要管理的一个或多个bean的定义。基于xml配置的bean方式为：<beans/> 为元素的顶级标识，<bean/>作为内部标识。而Java配置方式是则通常是在@Configuration 中使用bean注解方式（@Bean-annotated）。    
这些bean的定义对应于构成应用程序的实际对象。通常，开发人员可以定义服务层对象（service），数据访问层（data access objects），视图层（presentation，类似于Struts的Action对象），以及像Hibernate、SessionFactory、JMS队列等基础结构对象。通常不会在容器中配置细粒度的域对象。因为域对象往往由DAO层或者业务逻辑层负责加载。当然也可以使用Spring与AspectJ的集成功能来控制在IoC容器之外的对象的创建。（参见以下链接）。    
以下实例显示了基于xml配置元数据的基本结构。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">   <!--1--> <!--2-->
        <!-- collaborators and configuration for this bean go here -->
    </bean>
    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>
    <!-- more bean definitions go here -->
</beans>
```
其中，  
**1** 处的id属性是一个字符串，用来标识单个bean的定义。  
**2** 处的class属性是bean的类性，值必须使用类的权限定名。  
id属性的值指向对应的对象。在这个xml实例中没有显示各个bean对应的对象。需要了解更多内容请阅读参考依赖连接。
#### 1.2.1 实例化容器
实例化试，传递给ApplicationContext带参构造方法的字符串参数是资源路径。容器允许从外部资源（如本地文件系统，JAVACLASSPATH等）加载元数据。
````java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
````
**在了解了Spring IoC容器之后，或许你想了解更多关于Spring的资源抽象的知识（具体可以参见参考资料**链接**）。spring 提供了一种符合url定义的资源路径描述。特别是，如应用程序与资源路径中<链接>所描述的资源路径用于构建应用程序的context。**    
以下事例中显示了服务层对象（service.xml）配置文件：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```
一下的代码显示了数据访问层（daos.xml）的配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

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
在前面的例子中，服务层（以下统称为service层）由PetStoreServiceImpl 类和两个名为JpaAccountDao和JpaItemDao的数据访问层。petStore中的各个属性，其中name元素代表PetStoreServiceImpl类中的对应属性名，ref元素代表对应类的bean的id。id和ref元素之间的关系体现了对象之间的依赖关系。有关配置文件的依赖关系的详细信息参阅<链接>
#### 编写基于xml配置文件的元数据
将把阿匼定义到多个而非一个xml文件中会很有用。通常，每一个单独的xml配置文件都代表程序架构中的一个个逻辑层或者模块。用户可以使用应用程序的上下文构造器从所有的xml文件内容中加载bean的定义。构造器采用多个不同的资源路径，资源路径见上一节<链接>。或者使用一个或多个**<import/>**元素来从一个或多个文件中加载bean定义。下面是一个例子。    
```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```
在上面的示例中，外部bean定义从三个文件加载：services.xml，messageSource.xml和themeSource.xml。所有位置路径都相对于执行导入操作的文件，所以service.xml必须和导入文件处以同一路径或者与导入文件处以同一类路径下。而messageSource.xml和themeSource.xml必须在负责导入文件的resources路径下。如上所示路径开头的"/"是不起作用的。但是，鉴于这些都是相对路径，因此不要使用"/"开头。根据Spring Schema，导入的文件的内容（包括顶级<beans/>元素）必须是有效的XML bean定义。

```
在配置文件中可以但是不建议使用“../”的形式表示父级目录。这么做导致当前的应用对该程序以外的文件产生依赖。特别是在classpath:URLs中使用（例如：classpath:../services.xml）运行是解析类过程中会选择最近一次加载的类路径，然后查找其父路径。类路径配置的更改可能会导致选择的路径不同，从而导致路径错误。
用户可以使用全限定绝对路径而不是相对路径，例如：file:C:/config/services.xml或者classpath:/config/services.xml。但是，一定要注意将应用程序的配置与特定的绝对路径像吻合。通常的比较合理的方法是保持绝对路径的参数化。————例如通过使用${...}在运行是针对jvm系统属性来解析占位符。
```
命名空间本身提供了导入指令功能。Spring提供的一系列xml命名空间中提供了除普通bean定义外的其他配置功能。————如context和util的命名空间。
#### Groovy Bean定义DSL
作为外部化配置元数据的示例，bean的定义也可以使用Spring的Groovy Bean定义DSL，就像Grails框架那样。通常，此类配置位于“.groovy”文件中，其结构如下例所示：
```java
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
此配置样式在很大程度上等同于XML bean定义，甚至支持Spring的XML配置命名空间。它还允许通过importBeans指令导入XML bean定义文件。
### 1.2.3 容器的使用
ApplicationContext 是高级的工厂接口，能够维护不同的bean以及它们的依赖的注册表。使用T getBean(String name, Class<T> requiredType)，用户就能够获得对应的bean实例。ApplicationContext允许你获取bean的定义并访问这些bean，具体示例如下：
```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```
使用Groovy配置，相关的引导也非常类似。它是一个不同的context的实现类，符合Groovy-aware模式（同时也能识别xml bean的定义）。以下示例展示的是基于Groovy配置的对应使用。
```java
ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");
```
最灵活的变体是GenericApplicationContext与读代理相结合。 例如，使用XML文件的XmlBeanDefinitionReader，具体如以下示例所示：
```java
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```
也可以使用GroovyBeanDefinitionReader用于加载Groovy文件，具体示例如下：
```java
GenericApplicationContext context = new GenericApplicationContext();
new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
context.refresh();
```
用户可以在同一个ApplicationContext中混合使用读代理模式，用来加载不同的配置文件中的bean。
用户可以使用getBean获取bean的实例。ApplicationContext中提供了一些其他获取bean实例的方法，但是理想情况下在应用程序中都不应该被使用。实际上在应用程序中更本不应该调用getBean()方法，因此就没有必要依赖Spring的API。例如：Spring和Web框架组件（如控制器和JSF管理的bean）提供依赖注入，允许用户通过元数据来声明对特定bean的依赖。
## 1.3 bean概述
Spring的IoC容器管理着一个或多个bean。这些bean是用户提供给容器的配置元数据文件来创建的（例如在xml文件中的<bean/>标签中定义的）。    
在容器内部，这些bean的定义表示为BeanDefinition对象，其中包括但不限于以下元数据：
- 全限定类名，通常是声明bean实际实现类。
- bean的行为配置元素，用以说明bean在容器内的行为方式（作用范围，生命周期，回调等）；
- 引用其字啊运行时所依赖的其他bean。这里的引用也可以称为协作或者依赖。
- 要在新建的元素中设置的其他属性，例如资源池的大小限制或者在管理连接池的bean中设置连接数。
这些元数据转换成构成每一个bean得以的一组属性。以下的表格描述了这些属性：
表1.bean定义
|--属性|--定义|
|:-----|:-----|
|Class|bean对应的实体类|
|Name|Bean名称 |
|Scope|Bean的作用域|
|Constructor arguments|依赖注入|
|Properties|依赖注入|
|Autowiring mode|Autoriring 模式|
|Lazy initialization mode|懒加载模式|
|Initialization method|初始化方法调用|
|Destruction method|销毁调用|
除了包含有关如何创建特定bean的信息的bean定义之外，ApplicationsContext实现还允许注册在外部（由用户）创建的现有对象。这是通过通过getBeanFactory()方法访问ApplicationContext的BeanFactory来完成的，该方法返回BeanFactory接口的默认实现类DefaultListableBeanFactory。DefaultListableBeanFactory通过registerSingleton()和registerBeanDefinition()方法来支持注册。但是，典型的应用程序仅使用通过常规bean定义的元数据方式定义的bean。
```
元数据定义的bean和手动提供的单例实例需要尽可能早的注册，以便容器在自动装配和其他自我校验过程中进行正确的类型推断。虽然在某种程度上支持覆盖现有元数据和现有的单例实例，但是运行时注册新bean（与工厂的实时访问同时进行）并没有得到正式支持，可能会导致并发访问异常、bean容器中的状态不一致或两者都不一致。
```
### 1.3.1 Bean的命名
每一个bean都有一个或者多个标识符。这些标识符在管理它的容器中必须是全局唯一的。通常bean只有一个标识符。但是，如果它需要多个标识符，那么这个bean的其他标识符，可以视为别名。   
在基于xml的配置文件中，使用id属性，name属性，或者两者来指定bean的标识符。id属性允许用户指点一个id。通常这些名字是字母或者数字（比如myBean，someService等），同时也可以包含特殊字符。如果想要为bean引入其他别名，还可以在name属性中指定它们，用逗号（,），分号（;）或者空格来分割。在Spring的历史版本中，在3.1之前的版本，id属性定义为xsd:ID类型，它约束了可能的字符。从3.1开始，它被定义为xsd:string类型。注意，bean ID唯一性仍旧由容器强制执行，但不再由xml解析器强制执行。    
对于Spring来说bean的name和id并不是必须的。如果没有明确的提供name和id的值，容器就会为该Bean生成唯一name。但是，如果希望通过名称来使用该bean，通过使用ref元素或者Serivice Locator来查找，就必须为Bean提供一个唯一的name值。匿名的原因与使用内部bean和自动装配有关。（待议）。
```
Bean的命名规范
在为Bean命名时通常是使用标准的Java约定作为实例字段的名称。也就是说bean名称以小写字母开头，并遵循驼峰的命名规则。例如：accountManager, accountService, userDao, loginController。
为bean命名可以使配置更易于阅读和理解。另外，如果使用Spring AOP，那么为bean设置一个与功能相关的名称会很有帮助。
```
```
Spring 通过组件扫描器扫描类路径，然后按照上述的规则为匿名的类命名，实际上Spring在自动命名时会将类的简单类名的首字母变为小写作为该bean的名字。但是，在（少数）特殊情况下，当类的简单类名的前两个字母都为大写时，那么Spring就直接使用原始的简单类名为bean命名，而不在将首字母小写。这些规则与java.beans.Introspector.decapitalize（Spring在此处使用）中定义的规则相同。
```
#### 为Bean设置别名
在Bean定义中，用户可以通过唯一的id属性为bean指定多个名称。这些名称可以是同一个bean的等效别名，这在某些情况很有用，例如让应用程序中的每个组件通过使用特定于该组件本身的bean名称来引用公共依赖项。
有时候在bean的定义阶段为bean指定别名并不能满足所有的需求。有时候还需要额外为其他地方定义的bean设置别名。在大型系统中通常会将配置文件分布在各个子系统之中，每个子系统具有其自己的一组对象定义。在基于XML的配置元数据中，用户可以使用<alias/>元素来完成此任务。以下示例显示了别名的配置示例：
```xml
<alias name="fromName" alias="toName"/>
```
在上面的示例中，同一个容器中的bean既可以叫做fromName，又可以叫做toName。
例如，子系统A的配置元数据可以通过subsystemA-dataSource名称来引用DataSource数据。子系统B的配置元数据可以通过subsystemB-dataSource的名称引用DataSource。主系统在使用这两个子系统时，主系统可以通过myApp-dataSource来引用DataSource。要使以上的三个名称都同事指向一个对象需要在元数据中如下定义别名：
```xml
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```  
现在，每个组件和主应用程序都可以通过一个唯一的名称引用dataSource，并且在有效地创建命名空间后能确保不与任何其他定义冲突，但它们引用相同的bean。
```
Java 配置
如果使用Javaconfiguration，则可以使用@Bean注解来声明别名。有关详细信息，请参阅使用@Bean注解
```
### 1.3.2 实例化Beans

Bean本质上是用来创建一个或多个java对象的清单。当被访问并使用又该bean定义封装的配置元数据来创建（或获取）实际对象时，容器就会查看具名bean的清单。
如果使用基于XML的配置元数据，则指定要在<bean />元素的class属性中实例化的对象的类型（或类）
class属性通常是必须的(class是BeanDefinition实例上的Class属性)。（一些例外情况：请参阅使用实例工厂方法和Bean定义继承进行实例化<链接>）。以下的两种方式会使用class属性。
- 一般，当容器需要以反射的形式来实例化一个bean对象时就需要bean所设置的class属性，这一过程类似与Java使用new关键字来创建一个对象。
- 在少数情况下，容器调用静态工厂方法来创建bean对象，需要指定包含创建对象时调用的静态工厂方法的实际类。从静态工厂方法的调用返回的对象类型可以是同一个类或另一个类。
内部类名
如果要为静态嵌套类配置bean定义，则必须使用嵌套类的二进制名称。
例如，如果在com.example包中有一个名为SomeThing的类，并且此SomeThing类具有一个名为OtherThing的静态嵌套类，则bean定义上的class属性值将为com.example.SomeThing $ OtherThing。
请注意，在名称中使用$字符可以将嵌套类名与外部类名分开。

使用构造器实例化

当使用构造方法创建bean时，所有普通类都可以与使用并且能与Spring兼容。也就是说，正在开发的类不需要实现任何特定接口或以特定方式编码。只要简单的指定bean类就可以了。但是，根据为该特定bean使用的IoC类型，可能需要一个默认（空）构造函数。Spring IOC容器几乎可以管理任何类。它不仅限于管理真正的JavaBeans。大部分的Spring使用人员在使用实际的JavaBeans时候通常喜欢给该Javabean设置一个无参的构造器，同时为在容器中该bean配置的属性设置getter和setter方法。你也可以在容器包含更多的非bean格式的类。例如即使你使用绝对不符合JavaBeans规范的旧的连接池，Spring也能够对其进行管理。   

在使用基于XML的配置文件中，你可以参照如下的形式声明一个bean类。
```xml
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```
有关带参的构造方法以及在实例化对象之后再为对象设置相关属性的详细机制，请参阅依赖注入（链接）。
#### 使用静态工厂方法实例化
定义使用静态工厂方法创建的bean时，请使用class属性指定包含静态工厂方法的类和名为factory-method的属性，以指定工厂方法本身的名称。你应该能够调用这个方法 (就像后面提到的，该方法带有可选参数 )并返回一个活动对象，随后将其视为通过构造函数创建的。这种bean定义的一个用途是在遗留代码中调用静态工厂。以下bean定义指定通过调用工厂方法来创建bean。该定义未指定返回对象的类型（类），仅指定包含工厂方法的类。在此示例中，createInstance()方法必须是静态方法。以下示例显示如何指定工厂方法:
````xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
````
以下示例显示了一个可以使用前面的bean定义的类:
```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```
有关向工厂方法提供(可选)参数并在对象从工厂返回后设置对象实例属性的机制的详细信息，请参阅依赖项和配置。<链接>
#### 使用实例工厂方法实例化
与通过静态工厂方法实例化类似，使用实例工厂方法进行实例化会从容器调用现有bean的非静态方法来创建新bean。如果使用该机制，需要将class属性保留为空，并在factory-bean属性中指定当前（父级或祖先）容器中bean的名称，该容器包含要调用以创建对象的实例方法。使用factory-method属性设置工厂方法本身的名称。以下示例显示如何配置此类bean：
```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance" > 
```
下示例显示了相应的Java类：
```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
``` 
个工厂类也可以包含多个工厂方法，如以下示例所示： 
```xml
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
以下示例显示了相应的Java类：
```java
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
这种方法表明工厂bean本身可以通过依赖注入（DI）进行管理和配置。请参阅详细信息中的依赖关系和配置。<链接>
```
Spring文档中，“工厂bean”指的是在Spring容器中配置的bean，它通过实例或静态工厂方法创建对象。相反，FactoryBean(注意大小写)指的是特定于spring的FactoryBean。 
```
###1.4依赖
典型的企业应用程序不仅仅包含单个的对象（或Spring中的单个bean）。即使是最简单的应用程序也有一些对象可以协同工作，以呈现最终用户所看到的连贯应用程序。 这一节将介绍如何定义多个独立的bean定义，以及对象之间相互协作实现的应用程序的既定目标。 
####1.4.1依赖注入
赖项注入(Dependency injection, DI)是一个过程，在这个过程中，对象仅通过构造函数参数、到工厂方法的参数或从工厂方法构造或返回对象实例后,在对象实例上设置的属性来定义它们的依赖项(即与它们一起工作的其他对象)。 这个过程基本上是bean本身的反向（因此名称，控制反转），它通过使用类的直接构造或服务定位器模式来控制其依赖项的实例化或位置。    
使用依赖注入（DI） 这种方式使代码更为清晰，也更容易实现对象和它的依赖之间的解耦。该对象不需要查找其依赖，也不需知道依赖的位置和具体的类。因此，代码更容易测试特别是当依赖的使接口或者抽象类时，因此在单元测试中可以使用存根或模拟实现。  
DI存在两个主要形式：基于构造函数的依赖注入和基于Setter的依赖注入。
##### 基于构造器的依赖注入
基于构造函数的DI是由容器调用一个构造函数来完成的，该构造函数有许多参数，每个参数表示一个依赖项。调用具有特定参数的静态工厂方法来构造bean几乎是等效的，本讨论同样处理构造函数和静态工厂方法的参数。以下示例显示了一个只能通过构造函数注入进行依赖注入的类。
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
