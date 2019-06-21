
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
