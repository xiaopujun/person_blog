```xmind preview
./xmind/Spring.xmind
```

## Autowired和Resource区别

#### 注入顺序不同

- @Autowired：匹配时类型优先，其次根据名称匹配
- @Resource：匹配时名称优先，其次根据类型匹配

#### 作用范围

- @Autowired：构造器、字段、setter方法, 仅Spring容器内支持
- @Resource：字段、setter方法,所有IOC容器都应该支持

#### 默认行为

- @Autowired：默认情况下，找不到Bean，则抛出异常。
- @Resource：默认情况下，找不到Bean，则使用null。

## Spring循环依赖

#### 什么是循环依赖

如下代码所示，A和B都是都标注了@Component注解，因此他们都是Spring组件，并且A依赖B，B依赖A，他们相互依赖。相互依赖导致的问题是，当创建A实例的时候他引用了B，因此要创建B。但同时创建B时候发现引用了A。因此他需要创建A。如此反复，形成了死循环

```java

@Component
public class A {
    @Autowired
    private B b;

    public void setB(B b) {
        this.b = b;
    }
}

@Component
public class B {
    @Autowired
    private A a;

    public void setA(A a) {
        this.a = a;
    }
}

```

#### 如何解决循环依赖

介绍三级缓存需要先了解我们通常创建Java对象的过程。

```java 
public static void main(String[] args) {
        A a = new A();
        B b = new B();
        a.setB(b);
        b.setA(a);
    }
```

你会发现实例化一个对象的过程差不多分为两步，创建一个对象，给对象属性赋值。在Spring实例化的过程中，这2步加在一起完成算作实例化Bean完成。因此解决循环依赖的时候可以使用这个思路。假设现在要实例化A

1. 首先创建A实例，使用反射也好，new关键字也罢，先给他申请一块内存空间，把空对象先创建出来，
2. 此时发现他依赖于B，按照相同的方式去实例化B，然后发现B又依赖于A。又去实例化A，似乎陷入了死循环中。
3. 但其实我们第一步已经创建A对象了，只是还没有给他的字段属性赋值。因此如果有一个办法，将没有完全创建好的A对象先存起来，在实例化B要注入A的时候把这个提前存好的A先注入进去。B就可以完整的实例化。
4. 最后B已经是一个完整实例化好的对象，再把之前的半成品A拿出来设置好其中的b属性，这样A和B就都完整了。

这其中涉及到了A的半成品，他提前存了起来。这个存放他的地方就是缓存。（本质是一个Map容器而已）

##### 看似只需要一层缓存就行了，为什么是三级缓存？

孩子，你本身创建出来的这些单例Bean是不是就需要缓存？
不然难道每次都重新创建嘛？那就不叫单例Bean了。因此就算没有循环依赖，创建出来的Bean也需要缓存一份。而确实，二级缓存就可以解决循环依赖的问题。
但是还有一些特殊型情况。如下代码，加入A依赖的不是纯正的B对象，而是B对象的代理对象，又当如何？为了注入b的代理对象，需要先创建b对象，然后创建b的代理对象，最终注入到A的proxyb属性中。
你会说，这用二级缓存不也行嘛？只不过A、B的依赖之间加了一个代理对象。没错，也可以解决。那么为什么不使用二级缓存呢？原因在于Spring的AOP代理。在Spring中，一个Bean可能会被AOP框架代理。
在创建代理对象时，需要获取到完全初始化后的Bean。然而，如果我们只使用二级缓存，那么在Bean初始化的早期阶段，我们无法获取到需要被代理的完全初始化后的Bean。
因此，Spring引入了第三级缓存。第三级缓存存储的是创建Bean的工厂对象，这个工厂对象能够在Bean完全初始化之前返回Bean的引用。这样，即使Bean还没有完全初始化，AOP框架也可以创建对应的代理对象。

```java 

@Component
public class A {
    @Autowired
    private B proxyb;
}

@Component
public class B {
    @Autowired
    private A a;
}

```

三级缓存如下：

- 一级缓存（singletonObjects）：存放完全实例化后的Bean
- 二级缓存（earlySingletonObjects）：存放半成品Bean
- 三级缓存（singletonFactories）：保存单例Bean的创建工厂，工厂模式可以是一个lambda接口，它给出了自定义创造bean的空间；判断当前bean是否需要代理，然后创建代理对象

#### 解决循环依赖的限制

##### 为什么只支持单例作用域的bean之间的循环依赖

单例说明全局只有一个对象，比如A对象，那么整个应用的声明周期理论上只有一个A对象，因此创建出来的半成品A是可以缓存的。我从缓存里拿到的一直都是这个全局的，之后使用的也是这个全局的A对象。
如果是非单例的，那每次使用都是创建新的A对象。缓存起来有何意义？你缓存起来我也不会用它，我需要完全重新创建一个A，这就又卡死了，因为A的创建又需要一个全新的B。
这是通常的说法，我觉得倒也未必，如果每次都暂时缓存一下这个新创建的A对象，等这次循环依赖解决后立移除缓存中的半成品，应该也能解决。但可能Spring框架太庞大，这种方式可能对其他地方造成影响，所以只支持了单例

##### 为什么解决不了构造器注入的循环依赖

```java 
@Component
public class A {
    private B b;

    @Autowired
    public A(B b) {

    }
}
@Component
public class B {
    private A a;

    @Autowired
    public B(A a) {
        this.a = a;
    }
}

```

如果循环依赖的形式是上面这样的，这就有意思了，我们知道创建一个实例对象的时候，构造器是要先执行的，只有构造器执行完毕了，一个对象的内存空间才申请完毕，这个对象才可以使用。同时他才有了内存的地址引用。如果你在
构造器上注入，那么A创建的时候要求B一定要现有一个内存引用，但是B创建的时候A也是如此。此时的困境在于，他们两个连半成品的自己都造不出来，因此没有继续往下走的能力

## Spring事务的传播机制

Spring事务传播机制定义了当一个事务方法被另一个事务方法调用时，事务如何在这两个方法之间传播。Spring提供了7种事务传播行为：

1. **PROPAGATION_REQUIRED**：如果当前没有事务，就新建一个事务；如果当前存在事务，就加入该事务。这是默认的传播行为。
2. **PROPAGATION_SUPPORTS**：如果当前存在事务，就加入该事务；如果当前没有事务，就以非事务方式执行。
3. **PROPAGATION_MANDATORY**：如果当前存在事务，就加入该事务；如果当前没有事务，就抛出异常。
4. **PROPAGATION_REQUIRES_NEW**：新建一个事务，并暂停当前的事务（如果存在）。
5. **PROPAGATION_NOT_SUPPORTED**：以非事务方式执行，并暂停当前的事务（如果存在）。
6. **PROPAGATION_NEVER**：以非事务方式执行，如果当前存在事务，就抛出异常。
7. **PROPAGATION_NESTED**：如果当前存在事务，则在嵌套事务中执行；如果当前没有事务，就新建一个事务。

可以通过在@Transactional注解中指定propagation属性来设置方法的传播行为。例如：

```java
class DemoService {
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void someMethod() {
        // ...
    }
}
```

以上代码表示someMethod方法总是在一个新的事务中执行。


> 假设我现在有两个方法A和B。 A方法调用B方法。请就上面7种事务传播机制。分别模拟下A调用B方法时事务传播出现的各种情况

下面是模拟A方法调用B方法时，根据不同的事务传播行为，事务传播可能出现的各种情况：

PROPAGATION_REQUIRED：

如果A方法在事务中执行，那么B方法会加入A方法的事务中执行。 如果A方法不在事务中执行，那么B方法会新建一个事务并在其中执行。
PROPAGATION_SUPPORTS：

如果A方法在事务中执行，那么B方法会加入A方法的事务中执行。 如果A方法不在事务中执行，那么B方法也不在事务中执行。
PROPAGATION_MANDATORY：

如果A方法在事务中执行，那么B方法会加入A方法的事务中执行。 如果A方法不在事务中执行，那么B方法会抛出异常。
PROPAGATION_REQUIRES_NEW：

无论A方法是否在事务中执行，B方法都会新建一个事务并在其中执行。如果A方法在事务中执行，那么A方法的事务会被暂停，直到B方法的事务完成。
PROPAGATION_NOT_SUPPORTED：

无论A方法是否在事务中执行，B方法都不在事务中执行。如果A方法在事务中执行，那么A方法的事务会被暂停，直到B方法完成。
PROPAGATION_NEVER：

如果A方法不在事务中执行，那么B方法也不在事务中执行。 如果A方法在事务中执行，那么B方法会抛出异常。 PROPAGATION_NESTED：

如果A方法在事务中执行，那么B方法会在一个嵌套事务中执行。嵌套事务是一个子事务，它可以独立于父事务进行提交或回滚。
如果A方法不在事务中执行，那么B方法会新建一个事务并在其中执行。 以上就是根据不同的传播行为，A调用B时可能出现的各种情况。

## Spring事务的失效场景

Spring事务依赖于AOP，AOP本质上是动态代理，因此如果代理对象出现问题，就可能会导致事务失效。

#### 类方法中的直接调用

```java 
@Component
public class A {

    @Transactional
    public void test1() {
        test2();
    }

    @Transactional
    public void test2() {

    }
}
```

上面的代码中test2方法中的事务不会生效，A中是直接调用的test2方法，本质上是this.test2()。此时的this是纯粹的A对象，而不是代理对象。

#### final、static方法调用

代理对象、代理对象是对象级别的代理。一般JDK动态代理要实现接口要实现复写。final类型的无法进行复写，因此无法支持代理。static更不用说，不是对象级别的方法。无法代理

#### @Transactional参数设置错误

由于@Transactional可以控制事务的传播机制，因此设置不同的参数可以影响事务的实际行为，某些参数会要求方法在非事务环境下执行。
此外@Transactional(rollbackFor = RuntimeException.class)可以指定方法在什么异常下会导致事务回滚，如果异常类型指定错误，即使抛出异常，事务也不会回滚

#### 使用的@Transactional注解来源错误

并不是只有Spring实现了@Transactional注解，但是Spring体系下应该只有Spring实现的@Transactional注解才能让事务生效。如果使用了其他来源的@Transactional也会导致事务失效

#### 异常在方法内部被处理了

正常情况下，一个处于事务中的方式，如果抛出异常，事务回滚。但是如果抛出的异常被内部消化了，则不会导致事务触发回滚

#### 事务中使用了多线程

Spring中的声明式事务产生的上下文是绑定在ThreadLocal上的，而ThreadLocal是线程隔离的。每个线程都有自己的ThreadLocal。因此多线程环境下声明式事务不会生效。
但是可以使用编程式事务进行手动控制。

#### 使用的数据库本身就不支持事务

在数据库中，事务是数据库级别的，因此如果数据库本身不支持事务，则事务也无法生效。比如MySql的MyISAM引擎就不支持事务

## Spring Bean的生命周期

Spring Bean的生命周期指的是从Bean的创建、初始化、使用到销毁的整个过程。以下是Spring Bean生命周期的详细步骤：

1. **实例化（Instantiation）**：
    - Spring容器使用Bean的构造器或静态工厂方法来创建Bean实例。

2. **设置属性值（Populate properties）**：
    - Spring容器根据配置（XML、注解等）为Bean的属性赋值。这包括字段注入、setter方法注入以及构造器注入。

3. **处理BeanNameAware和BeanFactoryAware**：
    - 如果Bean实现了`BeanNameAware`接口，Spring容器会调用它的`setBeanName()`方法，传入Bean的ID。
    - 如果Bean实现了`BeanFactoryAware`接口，Spring容器会调用它的`setBeanFactory()`方法，传入BeanFactory容器实例。

4. **BeanPostProcessor前置处理（BeanPostProcessor pre-processing）**：
    - 在Bean的初始化方法调用之前，Spring容器会调用所有注册的`BeanPostProcessor`的`postProcessBeforeInitialization()`方法。

5. **初始化（Initialization）**：
    - 如果Bean实现了`InitializingBean`接口，Spring容器会调用它的`afterPropertiesSet()`方法。
    - 如果Bean在配置中指定了`init-method`，Spring容器会调用此方法。

6. **BeanPostProcessor后置处理（BeanPostProcessor post-processing）**：
    - 在Bean初始化完成之后，Spring容器会调用所有注册的`BeanPostProcessor`的`postProcessAfterInitialization()`方法。

7. **使用Bean**：
    - 此时Bean已经准备好被应用程序使用了，Spring容器会管理这个Bean的生命周期，直到容器关闭。

8. **处理DisposableBean和destroy-method**：
    - 当Spring容器关闭时，如果Bean实现了`DisposableBean`接口，Spring容器会调用它的`destroy()`方法。
    - 如果Bean在配置中指定了`destroy-method`，Spring容器也会调用此方法。

9. **销毁Bean（Destruction）**：
    - 最终，当Spring容器销毁时，Bean的生命周期结束。

### 扩展生命周期的控制

除了上述基本的生命周期步骤，Spring还提供了多种方式来扩展或自定义Bean的生命周期：

- **BeanPostProcessor**：允许你在Bean初始化前后执行自定义逻辑。
- **BeanFactoryPostProcessor**：允许你在BeanDefinition加载完成后、实例化之前修改BeanDefinition。
- **自定义初始化和销毁方法**：通过`init-method`和`destroy-method`属性指定自定义的初始化和销毁方法。
- **@PostConstruct和@PreDestroy注解**：在JSR-250标准中定义的注解，分别用于标记Bean初始化后和销毁前的方法。

通过这些机制，开发者可以灵活地控制和管理Spring Bean的生命周期，以满足不同的业务需求。


# 三、BeanFactory和FactroyBean的关系？

BeanFactory：

BeanFactory是Spring框架中的核心接口之一，它是用于管理和获取Bean实例的工厂接口。
BeanFactory负责创建、配置和管理Bean对象，提供了基本的IOC容器功能。它是Spring的底层基础设施，提供了对依赖注入和控制反转的支持。
BeanFactory的实现类有多种，如XmlBeanFactory、DefaultListableBeanFactory等。

FactoryBean：

FactoryBean是一个特殊的Bean，也是一个接口，它用于创建和管理特定类型的Bean实例。
实现FactoryBean接口的类是一种特殊的Bean工厂，用于生产其他类型的Bean对象。这意味着FactoryBean本身也是一个Bean。
通过FactoryBean创建的Bean可以具有更高的灵活性，可以在Bean创建过程中进行一些自定义的逻辑处理。 联系和区别：

BeanFactory是Spring框架的核心接口之一，用于管理和获取Bean实例，提供了基本的IOC容器功能。
FactoryBean是一个特殊的Bean，实现了FactoryBean接口，用于创建和管理特定类型的Bean实例，它本身也是一个Bean。
BeanFactory是Spring框架的一部分，负责整个Bean的生命周期和管理。而FactoryBean是BeanFactory的一种实现方式，它是在BeanFactory的基础上提供了更高级别的功能和扩展性。
BeanFactory可以管理各种类型的Bean，而FactoryBean专门用于创建某一特定类型的Bean。
当从容器中获取FactoryBean类型的Bean时，会得到FactoryBean的实例本身，而不是通过FactoryBean创建的Bean实例。如果想获取FactoryBean创建的Bean实例，需要在Bean名称前加上"&"
前缀。
例如，考虑一个名为"MyServiceFactoryBean"的FactoryBean实现，它创建类型为"MyService"的Bean实例。当从容器中获取"MyService"
时，实际上会得到"MyServiceFactoryBean"
的实例。要获取由"MyServiceFactoryBean"创建的"MyService"实例，需要使用"&myService"来获取。

总之，BeanFactory是Spring的核心接口，负责管理Bean的创建和生命周期。而FactoryBean是BeanFactory的一种特殊实现，用于创建和管理特定类型的Bean实例，提供了更高级别的功能和自定义逻辑的能力。

# SpringMVC路由的全过程

路由地址到Controller的方法底层数据结构使用HashMap，key为RequestMappingInfo，value为AbstractHandlerMethodMapping$MappingRegistration内部类对象。

使用反射调用到最终的Controller方法。

在Spring MVC中，当请求经过`javax.servlet.http.HttpServlet#service`方法，到通过反射调用具体Controller方法的整个过程中，Spring
MVC会执行以下主要步骤：

1. **请求进入Servlet容器**
   ：当客户端发起HTTP请求时，请求首先进入Servlet容器（如Tomcat），然后Servlet容器根据URL找到匹配的`DispatcherServlet`。

2. **DispatcherServlet处理请求**：`DispatcherServlet`是Spring MVC的前端控制器，所有请求都经过它的处理。`DispatcherServlet`
   会根据URL映射和HandlerAdapter的选择来决定将请求分派给哪个Controller处理。

3. **HandlerMapping的选择**：`DispatcherServlet`通过`HandlerMapping`来选择合适的Handler(Controller)
   处理请求。`HandlerMapping`是Spring
   MVC的接口，可以通过多种方式来配置URL和Handler之间的映射关系，例如`RequestMappingHandlerMapping`、`SimpleUrlHandlerMapping`
   等。

4. **HandlerAdapter的选择**：`DispatcherServlet`根据请求的处理方式，选择合适的`HandlerAdapter`来处理请求。`HandlerAdapter`
   是Spring
   MVC的接口，不同的Handler类型（如基于类的控制器和基于方法的控制器）会有不同的`HandlerAdapter`实现。

5. **参数解析和数据绑定**：一旦选择了合适的Handler(Controller)，`DispatcherServlet`
   会根据请求的参数和路径变量，通过参数解析器（`HandlerMethodArgumentResolver`
   ）将请求数据绑定到Controller的方法参数上。

6. **HandlerInterceptor的执行**：在调用Controller方法之前，`DispatcherServlet`会执行注册的`HandlerInterceptor`
   ，它可以在Controller方法执行前后进行一些预处理和后处理。

7. **Controller方法执行**：`DispatcherServlet`通过反射调用具体的Controller方法，传递之前解析的方法参数。

8. **返回逻辑视图名**：Controller方法处理完请求后，返回一个逻辑视图名（比如"index"、"viewPage"等）。

9. **视**图解析和渲染**：`DispatcherServlet`通过`ViewResolver`将逻辑视图名解析为具体的视图（如JSP、Thymeleaf模板等），然后将视图渲染为HTML。

10. **返回响应**：`DispatcherServlet`将渲染后的HTML作为响应返回给客户端。

需要注意的是，Spring MVC的整个**处理过程是高度可配置的，你可以通过`@RequestMapping`
注解、`HandlerMapping`、`HandlerAdapter`、`HandlerInterceptor`
、`ViewResolver`等来定制请求的处理过程，以满足不同的业务需求。这也是Spring MVC被广泛使用的原因之一，因为它提供了灵活的扩展点，可以方便地进行定制和拓展。

**浅谈对spring框架的理解：**

​
spring框架是一整个生态链工程，其中衍生出来很多具体型的框架，但是纠其核心内容，一个就是Di（依赖注入），一个是AOP（面向切面）。而我的理解，这两个核心又来自于Java语言自身的特性，由于Java支撑接口等解耦式的编程模型，这为spring的强大功能提供了保障。

​
对于spring的学习，首先应该有良好的Java基础，在学习spring的DI内容时，应该对Java的接口和继承等知识了解透彻，而学习AOP的时候，除了DI的基础知识，Java的代理也是一个需要掌握的重点，我认为，将这两点把握住，对学习spring框架有着极其重要的影响。

前言：经过学习之后，在spring的诸多配置中，我更加喜欢基于Java配置类的配置，和spring自动装配，这两着简洁明了。让我彻底不再想用原来的XML配置文件配置，所以，本笔记中暂时对基于XML的配置不进行记录，如果后面时间多，会补充上来。

# spring容器

spring容器简单理解就是一个大的盆，或者一个大缸，里面装满了Java对象，并且建立维护好了这些对象之间的关系，而在我们需要使用这些对象的时候，就不用再用new关键字来创建了，而实直接从这个大缸里面取，然后直接用就行了。

spring的容器有很多，并不止一个，spring框架有很多的容器实现，但是就分类而言，可以归纳为两种不同的类型，bean工厂就是最简单的容器，提供了基本的依赖注入（DI）功能，
**应用上下文**是基于bean工厂而来的，提供了框架级别的服务。

所以简单来讲，spring的容器有两种

1. bean工厂
2. 基于bean工厂的应用上下文（更加常用和高级）

spring中的应用上下文又有多种，下面是可能遇见的

| 上下文                                   | 解释                            |
|:--------------------------------------|-------------------------------|
| AnnotationConfigApplicationContext    | 从Java配置类中加载spring应用上下文（容器）    |
| AnnotationConfigWebApplicationContext | 从Java配置类中加载springWeb应用上下文（容器） |
| ClassPathXmlApplicationContext        | 从类路径下的xml配置文件中加载应用上下文（容器）     |
| FileSystemXmlApplicationContext       | 从文件系统下的xml配置文件中加载应用上下文（容器）    |
| XmlWebApplicationContext              | 从web应用下的xml配置文件中加载应用上下文（容器）   |

不管是从哪里加载spring上下文（容器）将对象bean加载到bean工厂的过程基本都是一致的。

```java
public class ContextTest {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("application.xml");
        System.out.println(applicationContext);
    }
}
```

如上代码，通过在类路径下加载spring容器，其他方式的都是类似的。

在成功加载了容器之后就可以使用getBean()方法获取容器中的对象了

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <beans>
        <bean id="person" class="com.ctbu.spring_demo1.Person">
            <property name="name" value="李白"/>
        </bean>
    </beans>
</beans>
```

```java
public class ContextTest {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("application.xml");
        Person person = (Person) applicationContext.getBean("person");
        System.out.println(person.name);
    }
}
```

如上代码，就是在xml中配置了类型之后，在Java中通过ClassPathXmlApplicationContext获取容器，并通过getBean()获取容器中的对象

# spring模块

spring框架发展至今，已经形成了多个模块，开发者可以自由选择，同时spring可以很友好的支撑集成第三方框架等。

就spring模块而言基本分为下面6个模块

| 模块              | 说明                    |
|-----------------|-----------------------|
| 数据访问与集成         | 用于持久层的操作，如访问数据库，关系映射等 |
| Web与远程调用        | 主要用于web应用的开发          |
| 面向切面编程          | 切面                    |
| instrumentation | 为jvm添加代理              |
| spring核心容器      | 依赖注入、控制反转             |
| 测试              | spring测试              |

# 如何装配Bean

​ 要装配bean，首先应该知道bean应该要装到哪里去，在spring中容纳bean（也就是对象实例）的东西叫容器。容器是spring的核心，容器里面装着各种各样的对象实例Bean。

​ spring的容器总体来讲有两个类型，一个是bean工厂，另一个是应用上下文（或者叫应用环境）ApplicationContext。我们常用的是应用上下文这个容器。

### 装配Bean的三种方式

1. 在XML配置文件中进行显示配置（最次选择）
2. 在Java配置类中进行显示配置（次优选择）
3. 使用注解的spring隐式自动发现机制和自动装配(最优选择)

#### 自动装配

*第一步：创建基本显示配置类，开启组件扫描*

实现自动装配仍然需要一定的显示配置，首先我们仍然需要一个配置类，这个配置类中什么都不需要写，但是需要加上必要的注解。

```java
@Configuration
@ComponentScan(value = {"aop"})
public class AopConfig {
}
```

##### @Configuration

@Configuration注解会告诉spring这是一个配置类，这是必须的注解

##### @ComponentScan

@ComponentScan注解指明需要扫描哪些包下的类，并为他们创建实例放入spring容器中，也就是开启组件扫描。

###### @ComponentScan的基本使用：

如果直接在配置类上贴上@ComponentScan注解，表示只扫描被贴上注解的配置类所在的包以及所在包的子包。

```java
@Configuration
@ComponentScan
public class AopConfig {
}
```

但是我们很多时候需要将配置类单独放到一个包中。这样就他的默认扫描范围就不满足我们的要求。于是我们可以这样修改。可以指定需要扫描的包，值是一个string类型的数组，值的具体元素为包得类路径，所以可以指定多个扫描的范围。

```java
@Configuration
@ComponentScan(value = {"aop"})
public class AopConfig {
}
```

明确的指明所设置的包是基础包

```java
@Configuration
@ComponentScan(basePackages = {"com.ctbu.spring_demo1.config"})
public class AppConfig {
}
```

扫描的基础包可以有多个，支持接收集合

```java
@Configuration
@ComponentScan(basePackages = {"com.ctbu.spring_demo1.config", "com.ctbu.spring_demo1.controller"})
public class AppConfig {
}
```

支持类作为扫描标记，使用这种方式的话对重构代码很友好，spring会自动去寻找这个类，并把这个类所在的包作为扫描的基础包。如果需要的时候，可以创建一个空的接口为组件扫描提供标记。

```java
@Configuration
@ComponentScan(basePackageClasses = Person.class)
public class AppConfig {
}	
```

*第二步：标注需要加入容器的类*

```java
@Component
public class Dance implements Performance {
}
```

##### @Component

@Component注解会将贴上注解的类作为spring的组件扫描进spring容器中。

###### @Component详解

spring应用上下文会给所有的bean都分配一个id，作为这个对象实例的唯一标识，当我们没有明确的指定这个id的时候，spring会自动为其分配一个id

| 情况                    | 措施                       |
|-----------------------|--------------------------|
| 在xml配置文件中没有明确指定beanID | spring会以全限定类名写作为BeanID   |
| 在组件扫描中没有明确指定BeanID    | spring会以类名的首字母小写作为BeanID |
| 在Java配置类中没有明确指定BeanID | spring会以方法名作为BeanID      |

**如果想自定义beanID**

| 情况        | 措施                                  |
|-----------|-------------------------------------|
| 在组件扫描中    | 指定@Component的值：@Component("person") |
| 在xml配置文件中 | 指定bean的id属性 ：id="person"            |
| 在Java配置类中 | 指定@Bean注解的值：@Bean（name=“person”）    |

spring还支持使用@Name来为bean（对象实例）命名，用法：@Name（“person”），但是此方法基本不使用。

*第三步：使用容器中的组件*

```java
public class MagicTest {
    @Autowired
    private Performance magic;
    
    @Test
    public void perform() {
        magic.perform(20);
        SpecialService service = (SpecialService) magic;
        service.specialService();
    }
}
```

##### @Autowired

@Autowired注解表示spring会对被标注的变量使用自动装配，spring会从容器中找出适合的bean实例，并将他赋值给这个变量。

###### @Autowired注解的使用

@Autowired注解不仅可以使用在一般的变量上，还可以使用在构造器方法，和属性的setter方法上。实际上@Autowired注解可以用在类中的任何方法上，spring会尽力去满足方法上所声明的依赖。

在自动注入的过程中，如果spring没有在spring容器中找到合适的bean进行注入，spring就会抛出异常，为了避免这个异常的发生，我们以及将@Autowired注解的required属性设置为false

```java
@Autowired(required = false)
```

但此时也要注意空指针异常的情况。

同样@Autowired注解也有替代方案@Inject，但是我们基本不使用这个注解，@Autowired更加适合spring，也更加让人容易接收。

#### 使用Java配置类进行显示配置

为什么要使用Java配置类的方式？尽管自动装配已经很强大了，对于我们自己写的bean，自动装配完全可以满足要求，但是，对于第三方的bean，比如datesource等，我们无法在其源码上贴上@Component注解，因此必须通过new其对象来创建对应的bean，而基于xml配置文件的方式也可以达到这样的效果，但是xml实在是太繁琐了。

*第一步：创建配置类*

```java
@Configuration
public class AopConfig {
}
```

如上代码是一个完整的基于Java配置类的组件装配设置。

如果完全使用Java配置类来为spring容器中添加bean，那么就可以不用开启组件扫描。

##### @Configuration

@Configuration注解表示被标注的类是一个配置类，spring容器对象可以从这个配置类中加载需要装入容器的bean实例。

*第二步：声明Bean*

```java
@Configuration
public class AopConfig {
    @Bean
    public Dance dance(){
        return new Dance();
    }
}
```

如上代码，当spring加载这个配置类的时候就会把创建的对象实例，作为组件，放入到spring容器中。默认情况下，这里创建的bean的名称ID为方法名，但是我们也可以自己指定名称ID。
**（在上面的表格中有详细归纳）**

```
@Bean(name = "dance")
```

##### @Bean

@Bean注解会告诉spring这个被标注的方法会返回一个对象，而这个返回的对象要注册为spring容器中的bean

第三步：使用组件

这一步和自动装配是一样的，同样可以使用@Autowired进行注入。

##### Java配置类装配存在引用的bean

很多时候，我们在装配一个bean的时候，这个bean可能引用了其他的bean实例，这样的配置在Java配置类中同样可以实现。下面的代码展示如何装配存在对象引用的bean

```java
public class Man {

    Food food;

    public Man(Food food){
        this.food = food;
    }

    public void eat() {
        System.out.println("吃：" + this. food);
    }
}
```

在上面的代码中，可以知道，如果要通过Java配置类的方式将Man这个bean放入spring容器中，那么newMan对象的时候需要有一个Food对象的引用。可以这样做。

```java
@Configuration
@ComponentScan(basePackageClasses = Person.class)
public class AppConfig {

    @Bean
    public Food food(){
        return new Food();
    }

    @Bean
    public Man man(){
        return new Man(food());
    }
}
```

直接在man这个方法中调用food()方法，以为food方法一定会产生一个Food类型的实例bean。

还有一种更加简单的当时来实现有引用的装配。

```java
@Configuration
@ComponentScan(basePackageClasses = Person.class)
public class AppConfig {

    @Bean
    public Food food(){
        return new Food();
    }

    @Bean
    public Man man(Food food){
        return new Man(food);
    }
}
```

如上Java配置类，直接将要引用的对象作为方法的形参传入，在创建对应bean的时候可以直接使用这个形参，spring会自动满足这个依赖。而且，要强调的是，这个要传入的形参food并不一定要求和Man在同一个配置类中，它可以是通过自动装配实现的，也可以在其他Java配置类中，甚至可以是在xml配置文件中，只要spring能在容器中发现他的存在。

#### 使用XML配置文件进行装配

使用XML配置文件进行bean的装配之前，首先需要创建一个xml配置文件，这一步相当于Java配置类中的@Configuration注解的作用。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

##### 声明一个简单的Bean

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="com.ctbu.xmlconfigtest.domain.Person"/>

</beans>
```

如上就声明了一个最简单的，以XML配置文件形式装配的bean实例

这里没有明确指定bean的id，所以默认情况下，会以类的全限定名称，也就是：com.ctbu.xmlconfigtest.domain.Person
作为bean的默认id，但是这太麻烦了，每一次引用难道都要写这么大一串？

因此，可以使用id属性指定bean的id；

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="com.ctbu.xmlconfigtest.domain.Person"/>

</beans>
```

##### XML中构造器参数的注入

XML配置文件中也会存在要配置对象引用的时候，这个使用有两种方式

1. <constructor-arg/>元素
2. c-命名空间：spring3.0后引入的功能

###### 使用<constructor-arg / >元素

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="com.ctbu.xmlconfigtest.domain.Person"/>

    <bean id="work" class="com.ctbu.xmlconfigtest.domain.Work">
        <constructor-arg ref="person"/>
    </bean>

</beans>
```

可以看到，创建work对象的时候需要注入一个person对象的引用，使用<constructor-arg ref="person"/>元素实现了这一需求。

###### 使用c-命名空间

这里的c：后面使用了构造器参数的名称

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="com.ctbu.xmlconfigtest.domain.Person"/>

    <bean id="work" class="com.ctbu.xmlconfigtest.domain.Work" c:person-ref="person"/>

</beans>
```

使用位置标志

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="com.ctbu.xmlconfigtest.domain.Person"/>

    <bean id="work" class="com.ctbu.xmlconfigtest.domain.Work" c:_0-ref="person"/>

</beans>
```

当构造器只有一个参数的时候，可以直接使用下划线（这种方式在IDEA集成环境下会报错，但是并不影响使用，这是集成环境的检查机制）所以选择一个最合适的即可

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="com.ctbu.xmlconfigtest.domain.Person"/>

    <bean id="work" class="com.ctbu.xmlconfigtest.domain.Work" c:_-ref="person"/>

</beans>
```

###### 构造器中注入字面量值

使用xml元素注入（此时的constructor-arg后不再用ref进行引用，而直接是value属性）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="com.ctbu.xmlconfigtest.domain.Person">
        <constructor-arg value="李白"/>
    </bean>

</beans>
```

使用c-命名空间

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="com.ctbu.xmlconfigtest.domain.Person" c:name="李白"/>

</beans>
```

###### 构造器装配集合

这种装配在XML中只有<constructor-arg/>属性能办到

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="com.ctbu.xmlconfigtest.domain.Person">
        <constructor-arg>
            <list>
                <value>篮球</value>
                <value>足球</value>
                <value>游戏</value>
            </list>
        </constructor-arg>
    </bean>

</beans>
```

这里person对象的构造器需要一个string数据类型的数据，同样的如果实际上你需要的是一个其他对象类型的引用，你可以把value改成ref即可完成bean引用列表的装配。

对于集合类型的变化也可以自行选择，使用list还是set，他们的差别和Java语法中，list和set集合的差别是一样的。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="com.ctbu.xmlconfigtest.domain.Person">
        <constructor-arg>
            <set>
                <value>篮球</value>
                <value>足球</value>
                <value>游戏</value>
            </set>
        </constructor-arg>
    </bean>

</beans>
```

##### XML中属性值的注入

属性值的注入方式也有2种，**都需要提供属性的setter方法**

- XML元素注入
- p命名空间注入

###### 字面量属性注入——XML元素

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="com.ctbu.xmlconfigtest.domain.Person">
        <property name="name" value="李白"/>
    </bean>

</beans>
```

###### p-命名空间的方式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:c="http://www.springframework.org/schema/c" xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="com.ctbu.xmlconfigtest.domain.Person" p:name="李白"/>

</beans>
```

若属性值为对象引用，同样使用ref的方式进行引用

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:c="http://www.springframework.org/schema/c" xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="com.ctbu.xmlconfigtest.domain.Person">
        <property name="work" ref="work"/>
    </bean>

    <bean id="work" class="com.ctbu.xmlconfigtest.domain.Work"/>

</beans>
```

###### 属性中集合的装配

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:c="http://www.springframework.org/schema/c" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/util https://www.springframework.org/schema/util/spring-util.xsd">

    <bean id="person" class="com.ctbu.xmlconfigtest.domain.Person">
        <property name="hobby">
            <list>
                <value>唱</value>
                <value>鬼畜</value>
            </list>
        </property>
    </bean>

</beans>
```

使用util命名空间完成属性集合的装配，如果使用util命名空间则需要在头部引入util命名空间规范。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:c="http://www.springframework.org/schema/c" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/util https://www.springframework.org/schema/util/spring-util.xsd">

    <bean id="person" class="com.ctbu.xmlconfigtest.domain.Person">
        <property name="hobby" ref="hobby"/>
    </bean>

    <util:list id="hobby">
        <value>唱</value>
        <value>跳</value>
        <value>rap</value>
    </util:list>

</beans>
```

# spring中容器配置的导入和混合使用

在使用spring装配bean的时候，可以有三种方式，这三种方式在实际开发中是可以混用的，spring并不排斥这种做法。

所以在使用Java配置类的时候仍然可以使用XML来做为辅助配置，同理自动装配也能与另外两种混用。

### 在Java配置类中引入XML配置

如果我现在有一个XML配置文件：application.xml，也有一个Java配置类AppConfig.java，现在我需要把XML中的配置引入到Java配置类中，参数最好是注明类路径。这样，在整个spring工程中，使用AppConfig这个配置类作为总的配置来加载，就能获取所有的bean，包括XML中的。

```java
@Configuration
@ImportResource("classpath:application.xml")
public class AppConfig {

    @Bean
    public Person person(){
        return new Person();
    }

}
```

#### @ImportResource

@ImportResource注解会告诉spring，去指定的文件中加载bean：如 @ImportResource("classpath:application.xml")

### 在Java配置类中导入Java配置类

```java
@Configuration
@Import(WebConfig.class)
public class AppConfig {

    @Bean
    public Person person(){
        return new Person();
    }

}
```

#### @Import

@Import注解会告诉spring，去那个配置类中导入配置

### 在XML中导入其他XML配置文件

同样使用XML的import标签

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <import resource="application2.xml"/>

</beans>
```

### 在XML中导入Java配置类

在XML中导入Java配置，其实就像是在XML中配置Bean一样。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="com.ctbu.source_merge.config.AppConfig"/>
</beans>
```

### 建议使用的方式

不管是使用XML配置，还是使用Java配置，实际开发中都建议配置一个总的配置类或者XML，用于导入其他配置。这样会显示的层次分明

# spring测试

spring的测试仍然需要junit的支持，所以需要引入依赖

```xml

<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

### @RunWith(SpringJUnit4ClassRunner.class)

此注解会在测试的时候自动创建一个spring的容器

### @ContextConfiguration(classes = AppConfig.class)

此注解会告诉spring容器去哪里加载配置，示例就是从AppConfig这个类加载bean和其他配置

完整示例

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = AppConfig.class)
public class PersonTest {

    @Autowired
    private Person person;

    @Test
    public void test1(){
        person.work();
    }

}
```

# spring—profile：环境快速迁移

spring-profile简单来说就是spring所提供的能快速切换开发模式的功能。很贴近开发的一个例子，我们在编写代码的时候可能用的本地数据库，自然需要配置一个本地数据库的链接，包括用户名，密码等，而项目完成之后将会使用的是云服务器上的数据库，这个时候怎么让测试时候的数据库一下子就切换到云数据库上去？spring-profile就是干这个事儿的。

当我们使用profile的时候首先需要配置profile，也就是决定哪些Bean应该在哪些环境下才允许被创建。当配置完了之后还不够，需要激活profile，这个时候就是程序真正运行的时候，它需要通过你的配置，加载你指定的环境下的Bean，而非指定环境的Bean将不会被创建。

## 配置profile

配置profile可以有多种方式

- @Profile注解
- XML配置文件

### @Profile注解

贴上了@Profile注解的配置类中的Bean，只有在选择了prod这种环境的时候才会被创建。

```java
@Configuration
@Profile("prod")
public class SeverDataSourceConfig {
    @Bean
    public ServeConnect serveConnect(){
        return new ServeConnect();
    }
}
```

现在的@Profile注解不仅支持类上，也支持在方法上进行标注。这样可以把不同环境的配置放入同一个配置类中

```java
@Configuration
public class LocalDataSourceConfig {

    @Bean
    @Profile("dev")
    public LocalConnect localConnect() {
        return new LocalConnect();
    }

    @Bean
    @Profile("prod")
    public ServeConnect serveConnect(){
        return new ServeConnect();
    }
}
```

### 在XML中配置profile

在XML中配置profile也很简单，只需在beans标签中配置profile属性即可，这样，这个配置文件下的所有bean只会在对应环境下才会创建

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd" profile="dev">

    <bean id="localConnect" class="com.ctbu.spring.profile.model.LocalConnect"/>

</beans>
```

当然，在XML中也能将多个环境配置在一个XML文件中，只需在beans标签中嵌套beans标签，并定义profile属性。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <beans profile="dev">
        <bean id="localConnect" class="com.ctbu.spring.profile.model.LocalConnect"/>
    </beans>

    <beans profile="prod">
        <bean id="serveConnect" class="com.ctbu.spring.profile.model.ServeConnect"/>
    </beans>

</beans>
```

## 激活profile

当profile配置完成了之后，实际在测试或者运行程序的时候，我们需要指定激活哪一种环境。

激活profile依赖于两个独立参数

```java
spring.profiles.active
spring.profiles.default
```

active的优先级高于default，当配置了active的时候，会优先使用，若没有配置则使用default的默认配置，若两个都没有配置，那就没有激活的profile，因此就值会创建没有定义profile的Bean。

有多种方式来设置这两个参数

1. DispatcherServlet的初始化参数
2. Web应用的上下文参数
3. JNDI条目
4. 环境变量
5. JVM的系统属性
6. 测试时，使用@ActiveProfiles注解

在web应用中使用web.xml进行激活

```xml

<context-param>
    <param-name>spring.profiles.active</param-name>
    <param-value>dev</param-value>
</context-param>
```

或者使用代码配置的方式

```java
AnnotationConfigApplicationContext acac = new AnnotationConfigApplicationContext(AppConfig.class);
acac.getEnvironment().setActiveProfiles("dev");
```

在测试时，使用@ActiveProfiles注解

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = AppConfig.class)
@ActiveProfiles({"dev","prod"})
public class LocalConnectTest {

    @Autowired
    private LocalConnect localConnect;

    @Autowired
    private ServeConnect serveConnect;

    @Test
    public void test(){
        System.out.println(localConnect.connect);
        System.out.println(serveConnect.connect);
    }

}
```

# Spring-AOP

spring aop功能是spring框架的核心功能之一，他能在不改变源代码的情况下在某一个步骤之前或者之后直接插入一个新的功能，极大的实现了程序的解耦性。

## 术语解释

- 通知：简单理解就是要插入的功能。通知分为5

  | 通知类型 | 说明                                                         |
                                                                                                                                                                                                                                  | -------- | ------------------------------------------------------------ |
  | 前置通知 | 在目标方法被调用之前调用通知方法                             |
  | 后置通知 | 在目标方法被调用之后调用通知方法                             |
  | 返回通知 | 在目标方法成功执行之后调用通知方法                           |
  | 异常通知 | 在目标方法抛出异常之后调用通知方法                           |
  | 环绕通知 | 将目标方法包含到通知方法之中，在目标方法执行的前后执行自定义的操作 |

- 连接点：程序中有很多地方都可能被插入通知方法，这些方法每一个都是一个点，称为连接点

- 切点：也就是通知方法具体要插入的那个点（就是连接点中的某一个点），也就是通知的目标方法

- 切面：通知和切点的结合就是切面

- 引入：允许在现有的类中添加新的方法或者属性

- 织入：将切面应用到目标方法上的过程，织入有3个时机，编译期、类加载期、运行期。

## spring对AOP的支持

spring框架提供了4种类型的AOP支持

1. 基于代理的经典spring aop
2. 纯POJO切面
3. @AspectJ注解驱动的切面
4. 注入式AspectJ切面（适用于spring的各个版本）

需要注意的一点是：spring sop构建在动态代理的基础上，因此，spring aop的支持局限于对方法的拦截。

## spring支持的切点指示器

通过指示器，我们可以自定义spring中的切面

| AspectJ指示器  | 说明                                                  |
|-------------|-----------------------------------------------------|
| arg()       | 限制连接点匹配参数为指定类型的执行方法                                 |
| @args()     | 限制连接点匹配参数由指定注解标注的执行方法                               |
| execution   | 用于匹配是连接点的执行方法                                       |
| this()      | 限制连接点匹配AOP代理的bean引用为指定类型的类                          |
| target      | 限制连接点匹配目标对象为指定类型的类                                  |
| @target()   | 限制连接点匹配特定的执行对象。这些对象对应的类要具有指定类型的注解。                  |
| within()    | 限制连接点匹配指定的类型。                                       |
| @within()   | 限制连接点匹配指定注解所标注的类型。当使用spring AOP时，方法定义在由指定的注解所标注的类里。 |
| @annotation | 限定匹配带有指定注解的连接点。                                     |

以上的匹配器中，只有execution指示器是用类执行匹配工作的，其他指示器都是用来限制匹配的。

## 切点表达式详解

```java
execution(* com.ctbu.spring.aop.test.impl.AopImpl.pay(..))
```

execution表示在目标方法执行时出发此切面

“ * ” 表示任意类型的返回值，也就是不关心返回值类型

后面是全限定类名和方法名

“ .. " 两点表示使用任意参数类型

```java
execution(* com.ctbu.spring.aop.test.impl.AopImpl.pay(..)) && within(test.*)
```

若限制条件不够，可以使用”&&“来连接其他的表达式，比如这里就限定了匹配的范围为 test包下的所有能匹配的方法。同样可以使用”||“来表示或，用”
！“表示非的关系。在XML中由于&有特殊的含义，因此链接表达式可以使用”and“、”or“、”not“来替换。

除了以上指示器，我们还有一种方法，可以具体到Bean的Id上，以此匹配到特定的Bean上。

```java
execution(* com.ctbu.spring.aop.test.impl.AopImpl.pay(..))&&bean(aopImpl)
```

如上，将通知方法的匹配范围限定到只有Bean的Id为aopImpl的这个类上。

还可以使用”！“在匹配范围中除去哪些类

```java
execution(* com.ctbu.spring.aop.test.impl.AopImpl.pay(..))&& !bean(aopImpl)
```

## 使用注解声明切面

| 注解              | 通知          |
|-----------------|-------------|
| @After          | 后置通知        |
| @Before         | 前置通知        |
| @AfterReturning | 目标方法返回后调用   |
| @AfterThrowing  | 目标方法抛出异常后调用 |
| @Around         | 环绕通知        |

### @Aspect

使用此注解标注的类说明此类是一个切面类，所有的切面都可以定义在此类中

使用此注解除了spring aop的依赖还需要引入aspect项目中的依赖

```xml

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>5.2.2.RELEASE</version>
</dependency>
```

也可以单独引入aspect项目的包

以下代码展示了一个完整的切面类的定义，定义切面类的时候注意要将切面类作为组件注入到spring容器中，可以使用自动扫描，也可以使用Javaconfig显示配置

```java
@Aspect
@Component
public class PerformanceAspect {

    @Before("execution(* com.ctbu.spring.aop.test.service.impl.Dance.dance(..))")
    public void audienceSitting(){
        System.out.println("观众就坐");
    }

    @After("execution(* com.ctbu.spring.aop.test.service.impl.Dance.dance(..))")
    public void applaud(){
        System.out.println("鼓掌");
    }
}
```

其他类型的切面同理

如果接的很多切面写同一个表达式比较麻烦，也可以使用如下方法来精简此过程

### @Pointcut

使用@Pointcut注解表示一个公共的切点，如果很多通知方法都是指向同一个切点的，那么这样定义可以极大的提升开发效率。

```java
@Aspect
@Component
public class PerformanceAspect {

    @Pointcut("execution(* com.ctbu.spring.aop.test.service.impl.Dance.dance(..))")
    public void performance(){};

    @Before("performance()")
    public void audienceSitting(){
        System.out.println("观众就坐");
    }

    @After("performance()")
    public void applaud(){
        System.out.println("鼓掌");
    }
}
```

### @EnableAspectJAutoProxy

如果就此止步的话，标注有@Aspect注解的类，虽然我们知道它是一个注解类，但实际上他和普通的Java类没什么区别。所以首先应该把他交给spring容器，使用@Component的自动扫描或者显示配置。其次，使用@EnableAspectJAutoProxy注解开启自动代理功能，这样一个切面的配置才算是真正完成

需要注意的一点是，在使用AOP的时候，注入的对象类型最好是接口（父类）类型。否则容易代理错误。

## 环绕通知

环绕通知时最为强大的通知类型，他能让你所编写的逻辑将被通知的目标方法完全包裹起来，实际上就像是在一个通知方法中同时编写前置和后置通知。

```java
@Aspect
@Component
public class PerformanceAspect {

    @Pointcut("execution(* com.ctbu.spring.aop.test.service.impl.Dance.dance(..))")
    public void performance(){};

    @Around("performance()")
    public void readToWork(ProceedingJoinPoint pj){
        try {
            System.out.println("观众就坐");
            pj.proceed();
            System.out.println("鼓掌");
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
    }
}
```

ProceedingJoinPoint参数时必须的，通过这个类型的对象的proceed()方法来调用被通知方法。

## 通知中的参数

```java
@Aspect
@Component
public class PerformanceAspect {

    @Before("execution(* com.ctbu.spring.aop.test.service.impl.DancePerform.dance(int)) && args(num)")
    public int sit(int num) {
        System.out.println("就坐" + num);
        return num;
    }
}
```

表达式中的int表示目标方法接收的参数类型，num表示将目标参数的类型绑定到num属性上，并传递给sit方法的num参数。从而实现了通知方法获取目标方法的参数

若使用porint的方式也可以这样实现

```java
@Aspect
@Component
public class PerformanceAspect {

    @Pointcut("execution(* com.ctbu.spring.aop.test.service.impl.DancePerform.dance(int)) && args(num)")
    public void performance(int num) {
    }

    @Before("performance(num)")
    public int sit(int num) {
        System.out.println("就坐" + num);
        num++;
        return num;
    }
}
```

# 向实体中注入数据

### 第一种：配置文件批量注入

###### 需要导入的依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
</dependency>
```

###### 实体类清单

**学生实体类**

```
@Data
@Component
@ConfigurationProperties("student")
public class Student {
    String name;
    Integer age;
    String grade;
    String[] hobby;
    Dog dog;
}
```

**dog类实体**

```
@Data
@Component
@ConfigurationProperties(prefix = "dog")
public class Dog {
    String name;
    String color;
}
```

###### **注解说明：**

@Data为lombok插件注解，作用是给实体类自动装配get、set、tostring方法

@Component：作用是将实体类交给spring容器

@ConfigurationProperties(prefix = "dog")
：作用是让springboot将拥有本注解的实体类和配置文件中的相关属性相互绑定，从而实现数据注入，prefix属性的作用就是指明将配置文件中的哪些数据进行绑定，默认从全局配置文件中读取数据

###### **配置文件数据清单**

```
student:
  name: "黎明"
  age: 34
  grade: 五年级三班
  hobby:
    - 唱歌
    - 跳舞
    - 游戏
  dog:
    name: 小黑
    color: 黑色
```

使用测试类，自动注入一个学生实体，即可将配置文件中的数据注入到实体类中

```
@SpringBootTest
public class StudentTest {

    @Autowired
    Student student;

    @Test
    public void test01(){
        System.out.println(student);
    }
}
```

说明：此测试是springboot的单元测试

### 第二种：使用@value单个注入

```
@Data
@Component
public class Student {
    @Value("${student.name}")
    String name;
    @Value("23")
    Integer age;
    @Value("大学一年级")
    String grade;
    String[] hobby;
    Dog dog;
}
```

如上，使用value只能每个属性一个一个的注入

使用@value注入配置文件中的值使用***@Value("${student.name}")***这样的写法（配置文件就是上面的清单）

@value支持SpEL表达式的写法：***@Value("#{2*7}")**，在里面可以进行计算运算

### 两种注入方式的比较

|            | @ConfigurationProperties | @Value |
|------------|--------------------------|--------|
| 功能         | 批量注入配置文件中的属性             | 一个个指定  |
| 松散绑定（松散语法） | 支持                       | 不支持    |
| SpEL       | 不支持                      | 支持     |
| JSR303数据校验 | 支持                       | 不支持    |
| 复杂类型封装     | 支持                       | 不支持    |

说明：松散绑定指的是说语法不严格，比如大小写不敏感，或者命名不严格要求，复杂类型封装指的是Value的方式无法封装对象，数组等类型的数据

#### JSR303数据校验

使用注解：@Validated

```
@Data
@Component
@ConfigurationProperties(prefix = "student")
@Validated
public class Student {
    @Email
    String name;
    Integer age;
    String grade;
    String[] hobby;
    Dog dog;
}
```

如上清单所示，使用@Validated注解后可以对实体的属性值进行数据校验，如果不符合数据类型会报错，但此种校验仅支持@ConfigurationProperties批量注入的形式

### 第三种：加载指定配置文件中的数据

需要的注解：@PropertySource，默认支持.properties类型的配置文件，不支持yml，需要自己设置

在需要出入的实体类上贴上此注解即可，读取流程和上面一样

另一个注解：

@**ImportResource**：导入Spring的配置文件，让配置文件里面的内容生效；

Spring Boot里面没有Spring的配置文件，我们自己编写的配置文件，也不能自动识别；

想让Spring的配置文件生效，加载进来；@**ImportResource**标注在一个配置类上

```java
@ImportResource(locations = {"classpath:beans.xml"})
导入Spring的配置文件让其生效
```



