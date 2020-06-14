# 1. The IoC Container  
本章主要介绍spring的IOC容器。  
**********
## 1.1. 介绍spring的IOC容器和Beans  
本章介绍了spring的IOC原理。IOC也被称为DI。在此过程中,对象定义他们的属性(也就是，它所需要的其他对象)只有通过构造器的参数,工厂方法的参数,以及从工厂方法返回后在对象实例上设置的属性来定义它们的依赖项。容器创建bean的时候注入这些属性。此过程从根本上讲，是bean本身的逆过程，它通过使用类或服务定位器模式等机制的直接构造来控制依赖项的实例化或位置(这就是控制反转)。  

org.springframework.beans 和 org.springframework.context是spring IOC的基本包。BeanFactory接口提供了一种高级配置机制，能够管理任何类型的对象。  
ApplicationContext是BeanFactory的子接口，它增加了如下特性：  

- 集成了spring的AOP特性；
- 消息资源处理(用于国际化);
- 事件发布;
- 应用程序层特定上下文，例如用于web应用程序的WebApplicationContext。  

简而言之，BeanFactory 提供了框架配置和基本的功能，ApplicationContext 增加了更多企业特性的功能。ApplicationContext 是 BeanFactory 的子类，完整的超集，在这章中，仅仅用于描述spring的IOC容器。使用BeanFactory的更多信息，请参考 ***1.16. The BeanFactory***  
在spring中，被Spring IoC 容器管理的应用程序的对象被称为bean(就是被IOC管理的对象叫做bean),Bean是一个对象被spring的 IOC容器实例化，组装和管理。除此之外，Bean只是应用程序中许多对象之一,容器使用配置元数据(configuration metadata)管理bean之间的依赖关系。  
***********
## 1.2. Container Overview(容器的概览)  
***org.springframework.context.ApplicationContext*** 接口代表了Spring IoC容器，负责实例化，配置和组装Beans。容器通过读取配置元数据（也就是spring的xml配置文件）来得到对象实例化，配置和组装的指令。配置元数据的方式有XML，java注解和java code（应该是javaConfig的方式）。它涵盖了组成你应用程序的对象，以及对象之间丰富的依赖关系。  
Spring提供了ApplicationContext接口的几种实现。在常见应用程序中,ClassPathXmlApplicationContext 或者 FileSystemXmlApplicationContext 是最常用的。尽管XML是定义配置元数据的传统格式，但是通过轻量化的xml去声明使用java注解或者code去配置元数据。  
在大多数应用程序场景中，不需要显式的使用代码来实例化SpringIoC容器的一个或多个实例。例如，在web应用中，在web.xml文件中，简单的8行样板代码通常就足够了(详情参考：1.15.4. Convenient ApplicationContext Instantiation for Web Applications)。如果你使用Eclipse开发工具，你可以轻松的通过点几下鼠标或者键盘就可以创建出样板配置文件了。  
下图显示了Spring工作原理的高级视图。ApplicationContext被创建和初始化之后，应用程序类和配置的元数据被整合，然后，你就获得了可配置或者执行的y应用程序了。  

![工作原理](./image/1.png)  
***********
### 1.2.1. Configuration Metadata (配置元数据)  
如上图所示，spring的IOC容器需要一个配置元数据（也就是配置文件）。作为应用程序开发人员，通过这个配置文件来告诉spring容器实例化,配置或者组装你应用程序中的哪些对象。  
传统上，配置元数据以简单直观的XML格式提供，本章中，大多数使用这种方式来解释spring IOC容器的关键概念。  
~~~
注意：xml配置不是唯一的配置元数据的方式。Spring IoC容器本身与实际写入此配置元数据的格式完全解耦。现在许多开发者使用 Java-based configuration （1.12. Java-based Container Configuration）来配置
~~~
更多的配置方式，如下：

- Annotation-based configuration (参考： 1.9. Annotation-based Container Configuration) 2.5版本开始支持  
- Java-based configuration (参考： 1.12. Java-based Container Configuration) 从Spring 3.0开始，Spring JavaConfig项目提供的许多功能成为了Spring核心框架的一部分。因此，您可以使用Java而不是XML文件来为你的应用程序定义benas。使用这些新的特性，请看 @Configuration, @Bean, @Import, 和@DependsOn 注解。

容器管理的配置文件由一个或者多个bean组成，xml形式配置beans，使用\<bean>标签，嵌套在\<beans/>。java配置方式是使用@Bean注解，注释在标有@Configuration的类中。  
定义的这些beans就是你应用程序中实际使用的对象。通常，您定义服务层对象，数据访问对象（DAO），表示对象（例如Struts Action实例），框架对象（例如Hibernate SessionFactories，JMS队列）等等。通常不会在配置文件中配置fine-grained domain objects，因为DAO和业务逻辑通常负责创建和加载域对象。然而，你可以使用Spring与AspectJ的集成，来配置在IoC容器控制之外创建的对象 (参考
5.10.1. Using AspectJ to Dependency Inject Domain Objects with Spring)  
接下来的例子展示了xml配置文件的基本机构  
~~~
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  ①②
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
~~~  
① id属性是一个字符串，用于标识单个bean定义；  
② class属性定义具体的bean，使用类的全名(包含包名)；  
id的值指向协作对象。示例的xml没有展示协作对象，更多请参考(1.4.1. Dependency Injection)  
**********
### 1.2.2. Instantiating a Container (实例化容器)  
ApplicationContext 构造函数的资源路径是一种字符串形式，这些资源字符串使容器可以从各种外部资源（例如本地文件系统，Java CLASSPATH等）加载配置元数据，等等。  
~~~
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
~~~  
~~~
注意：学完IOC之后，你可能想了解更多有关Spring的抽象类Resource（参考： Resources），它提供了一种方便的机制，用于从URI中定义的位置读取InputStream。就像2.7.1. Constructing Application Contexts中所描述那样，Resource路径用于构造应用程序的上下文。
~~~  
以下示例显示了服务层对象（services.xml）配置文件：    
~~~
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
~~~   
以下示例显示了数据访问对象daos.xml文件：
~~~
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
~~~   
在上面的例子中，服务层由PetStoreServiceImpl类和两个JpaAccountDao和JpaItemDao类型的数据访问对象组成（基于JPA对象关系映射标准）。元素***property name*** 代表javaBean 的属性，元素***ref*** 代表一个bean的定义。id和ref元素之间的这种联系表达了协作对象之间的依赖性。有关配置对象的依存关系的详细信息，请参阅(1.4.1. Dependency Injection)。  
#### 1.2.2.1 Composing XML-based Configuration Metadata  
bean定义可能会跨越多个XML文件，通常，独立的XML配置文件代表代码结构中的逻辑层或模块。  
您可以使用应用程序的构造函数从这些XML文件中加载bean。 如上一节中所示，此构造函数具有多个Resource路径。
或者，使用一个或多个<import />元素从另一个文件中加载bean定义。 以下示例显示了如何执行此操作：  
~~~
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
~~~  
如上示例，扩展的bean从services.xml, messageSource.xml和 themeSource.xml配置文件中获得。相对于导入文件，所有文件的路劲都是相对的（也就是说导入文件是相对路径，相对于定义导入这个文件比如import.xml），所以services.xml必须和导入文件在同一路径下，然而messageSource.xml和themeSource.xml必须在resources路径下，如您所见，斜杠被忽略。然而，鉴于这些路径是相对的，最好不要使用任何斜线。 根据Spring Schema规定，导入的文件的内容（包括顶级\<beans />元素）必须是有效的XML bean定义。  
~~~
注意：可以但不建议使用相对的“ ../”路径引用父目录中的文件。这样做会造成对当前应用程序外部文件的依赖。特别是，不建议将此应用用于classpath:URLs (例如， classpath:../services.xml),运行时会选择最近的classpath的根目录，然后查看他父级目录。类路径（classpath）配置的更改可能导致选择其他错误的目录。  
你始终可以使用标准资源位置而不是相对路径：例如，file:C:/config/services.xml or classpath:/config/services.xml。但是请注意，您正在将应用程序的配置耦合到特定的绝对位置。通常最好为这样的绝对位置保留一个间接寻址，例如通过在运行时针对JVM系统属性解析的“ $ {…}”占位符。
~~~  
命名空间本身提供了导入指令功能。Further configuration features beyond plain bean definitions are available in a selection of XML namespaces provided by Spring — for example, the context and util namespaces.  
#### 1.2.2.1 The Groovy Bean Definition DSL   
扩展配置元数据的另一个示例，从Grails框架中得知，在Spring’s Groovy Bean Definition DSL ，bean 也能被定义。通常，在“.groovy”文件中进行配置，例如：  
~~~
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
~~~  
这种配置样式在很大程度上等同于XML bean定义，甚至支持Spring的XML配置名称空间。 它还允许通过importBeans指令导入XML文件。  
************  
### 1.2.3. Using the Container(使用容器)  
ApplicationContext 是一个接口，能够管理不同bean的注册以及管理他们。通过使用***T getBean(String name, Class<T> requiredType)*** 这个方法，你可以实例化你的bean。  
ApplicationContext 可以让你读取bean definitions 并且访问他们，如下例子：  
~~~
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
~~~  
使用Groovy配置，使用起来非常相似。 它有一个不同的实现类，该类可识别Groovy（但也了解XML Bean定义）。 以下示例显示了Groovy配置：  
~~~
ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");
~~~  
最灵活的使用是GenericApplicationContext与读取器委托结合使用，例如，与XML文件的XmlBeanDefinitionReader结合使用，如以下示例所示：  
~~~
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
~~~  
也可以使用GroovyBeanDefinitionReader 解析Groovy文件，例如：  
~~~
GenericApplicationContext context = new GenericApplicationContext();
new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
context.refresh();
~~~    
在同一ApplicationContext上混合和匹配此类阅读器委托，从不同的配置源读取Bean定义。  
你可以使用getBean来获取bean的实例。ApplicationContext 几口有几种不同获取bean的方法，但是，理想情况下，您的应用程序代码永远不要使用它们。实际上，您的应用程序代码不应该调用getBean（）方法，因此就不会依赖于Spring API。例如，Spring整合了Web框架提供了依赖注入给不同web框架组件，例如controllers 和JSF-managed Bean），使您可以通过元数据（例如自动装配注释）声明对特定Bean的依赖项。  
*************  
## 1.3. Bean Overview(bean 总览)  
IOC容器可以管理很多benas，提供给容器的这些beans被元数据(也就是配置文件)所创建(例如，例如，以XML\<bean/>定义的形式)  
在容器内，这些被定义的bean为BeanDefinition对象，其中包含（除其他信息外）以下元数据：  
- 包的全名：通常，定义了Bean的实际实现类。  
- Bean行为配置元素，用于声明Bean在容器中的行为（作用域，生命周期回调等）。
- 对其它bean的引用，称为协作者或依赖项。
- 要在新创建的对象中设置的其他配置项，例如，池的大小限制或要在管理连接池的bean中使用的连接数。    

这些元数据转换为构成每个bean的属性。 下表描述了这些属性：  
Property | Explained in…​ | 对应章节
---|---|---|
Class | Instantiating Beans |1.3.2. Instantiating Beans
Name | 	Naming Beans |1.3.1. Naming Beans
Scope| Bean Scopes |1.5. Bean Scopes
Constructor arguments| Dependency Injection|1.4.1. Dependency Injection
Properties|Dependency Injection|1.4.1. Dependency Injection
Autowiring mode | Autowiring Collaborators|1.4.5. Autowiring Collaborators
Lazy initialization mode|Lazy-initialized Beans|1.4.4. Lazy-initialized Beans
Initialization method|Initialization Callbacks|1.6.1.1.Initialization Callbacks
Destruction method|Destruction Callbacks|1.6.1.2.Destruction Callbacks  

除了包含如何创建特定Bean的信息之外，ApplicationContext实现还允许将外部创建的对象交给spring管理。通过getBeanFactory() 方法从 ApplicationContextd的BeanFactory中返回默认实现类DefaultListableBeanFactory。DefaultListableBeanFactory 提供了registerSingleton(..) 和 registerBeanDefinition(..) 方法。然而，一般的应用程序还是通过元数据方式定义一个bean。  
~~~
注意：尽早的注册bean和手工提供的单例实例,以便容器在自动装配和其检查步骤中正确地推理它们。虽然在某种程度上支持覆盖现有元数据和现有单例实例，但官方不支持正在运行时（与对工厂的实时访问同时）对新bean的注册，并且可能导致并发访问异常，bean容器中的状态不一致或者二者都有。
~~~  
### 1.3.1. Naming Beans(命名beans)  
每个bean具有一个或多个标识符。 这些标识符在Bean的容器内必须是唯一的。bean通常只有一个标识符。 但是，如果需要多个，可以将扩展的名称视为别名。
  
在基本的xml配置文件中，你可以使用id属性或者name属性，或者同时使用两者来作为bean的标识。id属性可让您精确指定一个id。 通常，这些名称是字母数字（“ myBean”，“ someService”等），但它们也可以包含特殊字符。如果要为bean引入其他别名，还可以在name属性中指定它们，并用逗号（，），分号（;）或空格分隔。作为历史记录，在Spring 3.1之前的版本中，id属性定义为xsd：ID类型，该类型限制了可能的字符。 从3.1开始，它被定义为xsd：string类型。 请注意，Bean的ID唯一性仍由容器强制执行，尽管不再由XML解析器执行。  
你不需要为bean提供一个id或者name，如果你没提供id或者name，容器会为该bean生成一个唯一的名字。但是如果你想要通过name引用该bean，，则必须使用ref元素或a Service Locator style lookup，您必须提供一个name。 不提供name的动机与使用inner beans 和 autowiring 有关。
~~~
                            bean名称的约定  
当给bean命名的时候，遵循java规范。也就是，首字母小写并且采用驼峰形式。例如:accountManager,accountService,userDao,loginController等。
命名规范使你的配置文件简单且容易理解。另外，如果您使用Spring AOP,通过名字相关的一组应用起来将会方便很多。
~~~
~~~
注意：通过在类路径中进行组件扫描，Spring会按照前面描述的规则为未命名的组件生成Bean名称：本质上，是采用类名称并将其首字母转换为小写。但是，在特殊情况下，如果有多个字母并且第一个和第二个字母均为大写字母，则会保留原始大小写。 这些规则与java.beans.Introspector.decapitalize（Spring在此使用）定义的规则相同。
~~~  
#### 1.3.1.1. Aliasing a Bean outside the Bean Definition(别名)  
在bean定义本身中，可以使用由id属性指定的一个名称和name属性中任意数量的其他名称的组合来为bean提供多个名称。 这些名称可以是同一个bean的等效别名，并且在某些情况下很有用，例如通过使用该组件本身的bean名称，让应用程序中的每个组件都引用一个公共依赖项。  
但是，在实际定义bean的地方指定所有别名并不总是充足的。 有时需要为在别处定义的bean引入别名。 这在大型系统中通常是这种情况，在大型系统中，配置文件分配在每个子系统之间，每个子系统都有自己的对象定义集。 在基于XML配置元数据中，可以使用\<alias />元素来完成此操作。 以下示例显示了如何执行此操作：
~~~
<alias name="fromName" alias="toName"/>
~~~  
在这种情况下，在使用该别名定义之后，也可以将名为fromName的bean（在同一容器中）称为toName。  
例如：子系统A通过名称subsystemA-dataSource指定数据源。子系统B通过名称subsystemB-dataSource指定数据源。当主系统需要用到这两个子系统,主系统通过myApp-dataSource指定数据源。要使三个名称都引用相同的对象，可以将以下别名定义添加到配置文件中：  
~~~
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
~~~  
现在，每个组件和主应用程序都可以通过唯一的名称引用数据源，并保证不与任何其他定义冲突（有效地创建命名空间），但它们引用的是同一bean。  
~~~
                        Java-configuration
如果你使用Javaconfiguration，通过@Bean提供别名。具体参考：1.12.3. 1.Declaring a Bean
~~~  
### 1.3.2. Instantiating Beans(实例化)  
bean可以创建不止一个对象。 当被询问时，容器通过bean的名字查找，并通过配置文件使用该bean创建（或获取）实际对象。  
如果是基于xml的配置文件，在\<bean/>的标签中class属性指定了你要实例化对象的类型。class属性(在内部是BeanDefinition实例的一个Class属性)通常是必须的。（有关异常，请参见1.3.2.3Instantiation by Using an Instance Factory Method和1.7. Bean Definition Inheritance。）可以通过以下两种方式之一使用Class属性： 

- 通常，容器指定要构造的Bean类，然后通过反射调用其构造函数直接创建Bean，这在某种程度上等同于java使用new 产生的实例。  
- 指定的实际类是包含一个静态工厂方法的，该方法被调用来创建对象，少数情况下容器通过调用一个类的静态工厂方法来创建bean。通过调用静态工厂方法返回的对象的类型可能是相同的class或者完全不同的class（这段不太理解？）  
~~~
内部类的命名：
如果要为静态内部类配置Bean定义，则必须使用内部类的二进制名称。  
例如，在com.example包中有一个SomeThing类，在类SomeThing中有一个static的内部类OtherThing，class属性的值应该配置成com.example.SomeThing$OtherThing。  
$字符来区分内部类和外部类。
~~~  
#### 1.3.2.1.Instantiation with a Constructor （通过构造函数实例化一个）  
当用构造方法来创建bean，所有普通类都可以被Spring使用并兼容。也就是说，正在开发的类不需要实现任何特定的接口或以特定的方式进行编码（就是不用提供任何方法，例如setter方法）。只需指定bean类就足够了。 但是，根据特定bean使用的IoC的类型，您可能需要一个默认（空）构造函数。  
IOC容器可以管理任何你想要它管理的类。 它不仅限于管理真正的JavaBean。大多数Spring用户更喜欢实际的（真是存在的）JavaBean，它们仅具有默认（无参数）构造函数，并且根据容器中的属性适当的提供setter和getter方法。您还可以在容器中配置更多奇特的非bean样式类。例如，如果您需要使用不符合JavaBean规范的遗留的连接池，Spring也可以对其进行管理。  
通过xml配置文件，你可以指定bean像如下的方式：
~~~
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
~~~  
有关向构造函数提供参数（如果需要）并且在构造对象之后设置对象实例属性的方法，参考：1.4.1. Dependency Injection  
#### 1.3.2.2.Instantiation with a Static Factory Method（通过静态工厂实例化）  
当使用静态工厂方法去创建一个bean，class属性代表class所属类，这个类里提供了静态工厂方法，并用factory-method来指定这个方法。您应该能够调用此方法（带有可选参数，如后面所述），并返回一个对象，该对象其实是通过构造函数创建。 这种bean定义的方式被称为静态工厂方法。  
下面的例子说明了通过静态工厂方法创建bean的方式。这种定义方式不指定返回对象的类型（类），而仅指定包含的工厂方法。在这个例子中createInstance() 方式必须是静态的。请参考下面的例子：  
~~~
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
~~~  
根据上面的bean定义，下面的代码展示了如何使用：
~~~
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
~~~  
有关为工厂方法提供（可选）参数并在工厂返回对象后设置对象实例属性方法的详细信息，请参考：1.4.2. Dependencies and Configuration in Detail  
#### 1.3.2.3. Instantiation by Using an Instance Factory Method (通过实例方法来初始化)  
类似通过静态工厂方法实例化，通过实例化工厂调用非静态方法实例化一个bean。使用此方法，请将class属性保留为空，并用factory-bean属性指定bean（or parent or ancestor）的名称，该bean包含要创建对象的实例方法的。通过factory-method属性设置要调用的工厂方法。下面的例子展示如何配置该bean：  
~~~
<!-- the factory bean, which contains a method called createClientServiceInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
~~~  
下面的例子展示了相应的类  
~~~
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
~~~  
一个工厂类可以包含不止一个工厂方法，如下：  
~~~
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
~~~  
对应的类文件：  
~~~
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
~~~  
这种方法表明，工厂Bean本身可以通过依赖项注入（DI）进行管理和配置。详细请参考：1.4.2. Dependencies and Configuration in Detail  
~~~
注意：在Spring文档中，“ factory bean”是指在Spring容器中配置并通过实例或静态工厂方法创建对象的bean。相比之下，FactoryBean（请注意大小写）是指特定的Spring的实现类。
~~~  
#### 1.3.2.4. Determining a Bean’s Runtime Type(推断bean的运行时类型)  
推断特定bean的运行时类型并非易事。Bean元数据定义的类只是初始类引用，可能与声明的工厂方法或者是FactoryBean结合使用，这可能导致Bean的运行时类型不同，或者通过实例工厂方法下完全不进行设置（通过指定的factory-bean名称解析）。此外，AOP代理可以使用基于接口的代理包装bean实例，而目标Bean的实际类型（仅是其实现的接口）的暴露程度有限。***（这段很不理解，以后理解了回来进一步说明）***  
推断特定bean的实际运行时类型的推荐方法是对指定bean名称的调用BeanFactory.getType方法。 这考虑了以上所有情况，并返回了通过调用BeanFactory.getBean将针对同一bean名称返回的对象类型。（不理解）  
## 1.4. Dependencies(依赖关系)  

 