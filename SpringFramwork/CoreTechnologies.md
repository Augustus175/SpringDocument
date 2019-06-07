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