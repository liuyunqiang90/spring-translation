## 1. The IoC Container  
本章主要介绍spring的IOC容器。  
### 1.1. 介绍spring的IOC容器和Beans  
本章介绍了spring的IOC原理。IOC也被称为DI。在此过程中,对象定义他们的属性(也就是，它所需要的其他对象)只有通过构造器的参数,工厂方法的参数,以及从工厂方法返回后在对象实例上设置的属性来定义它们的依赖项。容器创建bean的时候注入这些属性。此过程从根本上讲，是bean本身的逆过程，它通过使用类或服务定位器模式等机制的直接构造来控制依赖项的实例化或位置(这就是控制反转)。  

org.springframework.beans 和 org.springframework.context是spring IOC的基本包。BeanFactory接口提供了一种高级配置机制，能够管理任何类型的对象。  
ApplicationContext是BeanFactory的子接口，它增加了如下特性：  

- 集成了spring的AOP特性；
- 消息资源处理(用于国际化);
- 事件发布;
- 应用程序层特定上下文，例如用于web应用程序的WebApplicationContext。  

简而言之，BeanFactory 提供了框架配置和基本的功能，ApplicationContext 增加了更多企业特性的功能。ApplicationContext 是 BeanFactory 的子类，完整的超集，在这章中，仅仅用于描述spring的IOC容器。使用BeanFactory的更多信息，请参考 ***1.16. The BeanFactory***  
在spring中，被Spring IoC 容器管理的应用程序的对象被称为bean(就是被IOC管理的对象叫做bean),Bean是一个对象被spring的 IOC容器实例化，组装和管理。除此之外，Bean只是应用程序中许多对象之一,容器使用配置元数据(configuration metadata)管理bean之间的依赖关系。  
### 1.2. Container Overview(容器的概览)  
***org.springframework.context.ApplicationContext*** 接口代表了Spring IoC容器，负责实例化，配置和组装Beans。容器通过读取配置元数据（也就是spring的xml配置文件）来得到对象实例化，配置和组装的指令。配置元数据的方式有XML，java注解和java code（应该是javaConfig的方式）。它涵盖了组成你应用程序的对象，以及对象之间丰富的依赖关系。  
Spring提供了ApplicationContext接口的几种实现。在常见应用程序中,ClassPathXmlApplicationContext 或者 FileSystemXmlApplicationContext 是最常用的。尽管XML是定义配置元数据的传统格式，但是通过轻量化的xml去声明使用java注解或者code去配置元数据。  
在大多数应用程序场景中，不需要显式的使用代码来实例化SpringIoC容器的一个或多个实例。例如，在web应用中，在web.xml文件中，简单的8行样板d代码通常就足够了(详情参考：1.15.4. Convenient ApplicationContext Instantiation for Web Applications)。如果你使用Eclipse开发工具，你可以轻松的通过点几下鼠标或者键盘就可以创建出样板配置文件了。  
下图显示了Spring工作原理的高级视图。ApplicationContext被创建和初始化之后，应用程序类和配置的元数据被整合，然后，你就获得了可配置或者执行的y应用程序了。  

![工作原理](./image/1.png)  
#### 1.2.1. Configuration Metadata (配置元数据)  
如上图所示，spring的IOC容器需要一个配置元数据（也就是配置文件）。作为应用程序开发人员，通过这个配置文件来告诉spring容器实例化,配置或者组装你应用程序中的哪些对象。  
传统上，配置元数据以简单直观的XML格式提供，本章中，大多数使用这种方式来解释spring IOC容器的关键概念。  
~~~
注意：xml配置不是唯一的配置元数据的方式。Spring IoC容器本身与实际写入此配置元数据的格式完全解耦。现在许多开发者使用 Java-based configuration （1.12. Java-based Container Configuration）来配置
~~~
