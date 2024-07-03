# 一、spring启动的整个过程

核心代码位置 org.springframework.context.support.AbstractApplicationContext#refresh

```java
class AbstractApplicationContext {
    public void refresh() throws BeansException, IllegalStateException {
        synchronized (this.startupShutdownMonitor) {
            //...
            try {
                //...
                // 扫描项目中所有的bean定义
                invokeBeanFactoryPostProcessors(beanFactory);
                //...
                // 实例化所有非懒加载的bean
                finishBeanFactoryInitialization(beanFactory);
                //...
            } catch (BeansException ex) {
                //...
            } finally {
                //...
            }
        }
    }
}
```

## 1.1 启动应用：

在Spring应用中，应用的启动一般由应用的入口类完成，例如使用SpringBoot时，可以通过@SpringBootApplication注解标记的类作为入口类。启动应用时，会触发Spring的初始化和上下文创建过程。

## 1.2. 创建Spring上下文：

创建Spring上下文是通过选择适当的ApplicationContext类型来完成的。一般有两种类型的上下文：XML文件配置的上下文和基于注解配置的上下文。

XML文件配置的上下文：
如果应用使用XML文件进行配置，可以使用ClassPathXmlApplicationContext或FileSystemXmlApplicationContext等类来创建上下文。通过提供XML配置文件的路径，上下文会读取并解析配置文件，创建相应的Bean定义和容器。

基于注解配置的上下文：
如果应用使用基于注解的配置方式，通常会使用AnnotationConfigApplicationContext来创建上下文。您可以通过传递配置类（带有@Configuration注解的类）或扫描基础包来指定要扫描的组件和配置信息。

在创建上下文时，还会判断应用是否为Web应用。如果是Web应用，则会初始化Web相关的服务和Servlet。

## 1.3. 创建Bean的扫描器：

在基于注解的配置中，Spring需要一个Bean扫描器来扫描项目中的类，确定哪些类应该被识别为Bean并进行处理。扫描器需要配置以下信息：

包的扫描路径：指定需要扫描的包路径，Spring会在这些路径下寻找带有特定注解的类。

扫描的注解：指定需要扫描的注解，带有这些注解的类会被认为是Bean。

在基于注解的配置中，Spring的扫描器是通过ClassPathBeanDefinitionScanner类来实现的。在创建AnnotationConfigApplicationContext时，会创建一个ClassPathBeanDefinitionScanner实例，并根据配置扫描指定的包路径和注解。对于XML配置的上下文，Bean扫描器是通过XmlBeanDefinitionReader类来实现的。

## 1.4. 创建Bean工厂：

在Spring中，Bean工厂负责管理Bean的创建、依赖注入和生命周期等任务。在上下文创建过程中，会根据上一步中的扫描器信息，创建一个对应的Bean工厂。

对于XML配置的上下文，常用的Bean工厂是DefaultListableBeanFactory。

对于基于注解的上下文，常用的Bean工厂是DefaultListableBeanFactory或AnnotationConfigApplicationContext。

在创建Bean工厂时，还会创建一个BeanPostProcessor列表，用于在Bean实例化过程中执行一些自定义的操作，例如AOP代理、事务管理等。

## 1.5. 扫描项目中的Bean定义：

在上下文创建完成后，根据扫描器的配置，开始扫描项目中定义的Bean。这个过程会扫描指定包路径下的类，并解析类上的注解信息，确定哪些类需要被创建为Bean。

对于XML配置的上下文，会解析XML配置文件中定义的Bean，并将它们转化为Bean定义。

对于基于注解的上下文，会扫描指定包路径下的类，并解析带有特定注解的类，将它们转化为Bean定义。例如@Component、@Service、@Repository等。这些带有注解的类会被认为是Bean，并被转化为Bean定义。

在扫描过程中，解析出的Bean定义将被注册到Bean工厂中。这些定义包含了Bean的相关信息，如名称、类型、依赖关系等。
在扫描过程中，对于基于注解的配置，还会解析类中的其他注解，例如@Autowired、@Qualifier等，这些注解会影响Bean的依赖注入过程。

## 1.6. 创建所有非懒加载的单例Bean实例：

在Bean定义阶段完成后，Spring会根据定义创建所有非懒加载的单例Bean实例。这意味着这些Bean会在上下文创建时被实例化，并且在整个应用的生命周期中只有一个实例。

对于XML配置的上下文，会根据XML配置文件中的Bean定义，使用反射机制实例化Bean对象。

对于基于注解的上下文，会根据扫描到的带有特定注解的类，使用反射机制实例化Bean对象。

在实例化Bean时，如果Bean存在依赖关系，则会先实例化依赖的Bean，并将其注入到目标Bean中。

在创建Bean实例时，如果Bean实现了FactoryBean接口，那么会调用FactoryBean的getObject()方法来创建Bean实例。另外，在实例化Bean时，还会处理Bean的循环依赖问题，例如A依赖B，B依赖A。

## 1.7. 执行Bean的初始化方法：

在实例化Bean之后，Spring会执行Bean的初始化方法。这些方法可以使用注解@PostConstruct标注或实现InitializingBean接口的afterPropertiesSet()
方法。在初始化方法中，您可以执行一些自定义的逻辑，例如初始化资源、建立连接等。

在执行Bean的初始化方法之前，还会执行BeanPostProcessor的postProcessBeforeInitialization()
方法，这些方法可以对Bean实例进行一些预处理操作。在执行初始化方法之后，还会执行BeanPostProcessor的postProcessAfterInitialization()方法，这些方法可以对Bean实例进行一些后处理操作。

## 1.8. 完成Spring上下文的创建：

完成所有非懒加载单例Bean的实例化和初始化后，Spring上下文的创建过程就完成了。此时，您可以通过上下文获取和使用Bean实例。在上下文创建完成后，还会触发一些事件，例如ContextRefreshedEvent。您可以通过实现ApplicationListener接口并监听相应的事件来执行一些自定义的操作。

AbstractAutowireCapableBeanFactory.java:进行实例的初始化，包括依赖注入，初始化方法调用等

# 问题解答

```java

@Component
public class A {
    @Autowired
    private B b;

    public void print() {
        System.out.println("A");
    }
}

@Component
public class B {
    @Autowired
    private A a;

    public void print() {
        System.out.println("B");
        a.print();
    }
}
```

> 这是我定义的两个bean， spring是如何将他们实例化的？依赖注入的步骤是怎么样的？

在您提供的代码中，A和B类都使用了@Component注解，这意味着它们都会被Spring框架扫描并注册为bean。此外，两个类都使用了@Autowired注解来注入对方的实例。

当Spring容器启动时，它会读取配置元数据并扫描应用程序中的所有类。当它发现带有@Component注解的类时，它会将这些类注册为bean。在这个过程中，Spring容器会创建一个BeanDefinition对象来存储有关bean的信息。

接下来，当您第一次调用getBean()方法或者当ApplicationContext启动时，IoC容器会触发依赖注入并将bean实例放入缓存中。在这个过程中，Spring容器会实例化bean并根据配置元数据进行依赖注入。

在您提供的代码中，当Spring容器实例化A类时，它会发现A类依赖于B类。因此，它会先实例化B类，并将其注入到A类中。同样，当Spring容器实例化B类时，它会发现B类依赖于A类。因此，它会先实例化A类，并将其注入到B类中。

总之，在您提供的代码中，Spring容器会扫描应用程序中的所有类，并将带有@Component注解的类注册为bean。当您第一次调用getBean()
方法或者当ApplicationContext启动时，IoC容器会触发依赖注入并将bean实例放入缓存中。在这个过程中，Spring容器会根据配置元数据进行依赖注入。


> 这个过程中a依赖于B，他想先实例化B。但是必要依赖于a。他要先实例化a。这样就会有一个循环依赖的问题。 Spring是如何解决的？

您提到的问题被称为循环依赖。Spring框架可以处理单例作用域的bean之间的循环依赖，但不能处理原型作用域的bean之间的循环依赖。

当Spring容器检测到单例作用域的bean之间存在循环依赖时，它会使用一种称为三级缓存的技术来解决这个问题。具体来说，Spring容器会在实例化bean之后、初始化bean之前将bean的引用存储在一个缓存中。当另一个bean需要注入这个bean时，Spring容器会从缓存中获取这个bean的引用并注入它。

在您提供的代码中，当Spring容器实例化A类时，它会发现A类依赖于B类。因此，它会先实例化B类，并将其引用存储在缓存中。然后，它会继续实例化A类，并将B类的引用注入到A类中。同样，当Spring容器实例化B类时，它会发现B类依赖于A类。因此，它会先实例化A类，并将其引用存储在缓存中。然后，它会继续实例化B类，并将A类的引用注入到B类中。

> “实例化bean之后、初始化bean之前”。这个操作具体指的是什么？以A为例子，是他创建了A的对象，但是没有初始化b这个属性，是这个意思吗？

是的，您的理解是正确的。在Spring框架中，实例化bean指的是创建bean的实例，但不包括初始化bean。初始化bean指的是在实例化bean之后、使用bean之前对bean进行配置和设置。


> 三级缓存是那三级？三级缓存如何在解决循环依赖的时候发挥作用?

在Spring框架中，三级缓存指的是singletonObjects、earlySingletonObjects和singletonFactories这三个缓存。它们都是在DefaultSingletonBeanRegistry类中定义的。

singletonObjects：这是一个用于存储完全初始化的单例bean的缓存。当一个单例bean被完全初始化后，它会被存储在这个缓存中。
earlySingletonObjects：这是一个用于存储早期的单例bean的缓存。当一个单例bean被实例化但尚未完全初始化时，它会被存储在这个缓存中。
singletonFactories：这是一个用于存储单例bean工厂的缓存。当一个单例bean被实例化但尚未完全初始化时，它对应的工厂会被存储在这个缓存中。
当Spring容器检测到循环依赖时，它会使用三级缓存来解决这个问题。具体来说，当Spring容器实例化一个单例bean但尚未完全初始化它时，它会将这个bean的工厂存储在singletonFactories缓存中。然后，当另一个bean需要注入这个bean时，Spring容器会从singletonFactories缓存中获取这个bean的工厂，并使用它来创建一个早期的bean实例。接着，Spring容器会将这个早期的bean实例存储在earlySingletonObjects缓存中，并将其注入到另一个bean中。


> “但是，Spring解决循环依赖是有一定限制的，首先就是要求互相依赖的Bean必须要是单例的Bean，另外就是依赖注入的方式不能都是构造函数注入的方式。” 为什么无法解决都是构造函数注入的方式的循环依赖？

当两个Bean都是通过构造函数注入对方时，Spring无法解决循环依赖问题。这是因为在构造函数注入时，Bean的实例化必须在构造函数执行完成后才能完成。如果两个Bean都是通过构造函数注入对方，那么它们都需要等待对方的实例化完成才能完成自己的实例化，从而导致循环依赖的问题1。

相比之下，当使用属性或setter方法注入时，Bean的实例化可以在注入之前完成，因此Spring可以通过三级缓存机制来解决循环依赖问题。具体来说，Spring会先创建一个不完整的Bean实例并将其放入缓存中，然后在注入时从缓存中获取该Bean实例并完成注入1。


> 请你基于上面的两个bean Java类。将spring使用三级缓存解决两个Bean实例之间循环依赖的整个过程按步骤一步一步分析说明下

```java

@Component
public class A {
    @Autowired
    private B b;

    public void print() {
        System.out.println("A");
    }
}

@Component
public class B {
    @Autowired
    private A a;

    public void print() {
        System.out.println("B");
        a.print();
    }
}
```

当Spring容器启动时，它会扫描并实例化所有的Bean。在这个过程中，Spring会使用三级缓存来解决循环依赖的问题。下面是Spring使用三级缓存解决上述两个Bean之间循环依赖的整个过程：

1. 假设Spring容器首先实例化Bean A。在实例化Bean A之前，Spring会将其添加到singletonsCurrentlyInCreation集合中，表示该Bean正在创建中。

2.在实例化Bean A时，Spring会发现它依赖于Bean B。因此，Spring会尝试实例化Bean B。

3. 在实例化Bean B之前，Spring会将其添加到singletonsCurrentlyInCreation集合中。

4. 在实例化Bean B时，Spring会发现它依赖于Bean A。由于Bean A已经在singletonsCurrentlyInCreation集合中，因此Spring知道存在循环依赖。

5. 此时，Spring会尝试通过三级缓存来解决循环依赖的问题。首先，它会检查一级缓存singletonObjects中是否存在Bean A的实例。由于Bean A还没有被完全实例化，因此一级缓存中不会存在Bean A的实例。

6. 接下来，Spring会检查二级缓存earlySingletonObjects中是否存在Bean A的实例。由于这是第一次检查二级缓存，因此二级缓存中也不会存在Bean A的实例。

7. 然后，Spring会检查三级缓存singletonFactories中是否存在一个可以创建Bean A实例的工厂对象。由于这是第一次检查三级缓存，因此三级缓存中也不会存在可以创建Bean A实例的工厂对象。

8. 由于三级缓存中不存在可以创建Bean A实例的工厂对象，因此Spring会创建一个不完整的Bean A实例（即只调用了构造函数但还没有完成依赖注入），并将一个可以返回该不完整实例的工厂对象添加到三级缓存中。

9. 接下来，Spring会继续实例化Bean B。在完成依赖注入时，它会再次检查三级缓存，并发现可以创建Bean A实例的工厂对象。因此，它会使用该工厂对象获取到不完整的Bean A实例，并完成对Bean B的依赖注入。

10.在完成对Bean B的依赖注入后，Spring会将完整的Bean B实例添加到一级缓存中，并从singletonsCurrentlyInCreation集合中移除。

11. 接下来，Spring会继续完成对Bean A的依赖注入。在完成依赖注入时，它会从一级缓存中获取到完整的Bean B实例，并完成对Bean A的依赖注入。

12. 在完成对Bean A的依赖注入后，Spring会将完整的Bean A实例添加到一级缓存中，并从singletonsCurrentlyInCreation集合中移除。

通过上述步骤，Spring成功地使用三级缓存解决了两个Bean之间循环依赖的问题。


> spring bean的生命周期是怎样的？

Spring Bean的生命周期包括以下几个阶段：

实例化：Spring容器根据Bean定义信息创建Bean实例。

属性赋值：Spring容器根据Bean定义信息为Bean实例的属性赋值。

初始化：Spring容器调用Bean的初始化方法（如果有）来完成Bean的初始化。初始化方法可以通过@PostConstruct注解或者在Bean定义中指定。

Bean后置处理器：Spring容器会调用已注册的Bean后置处理器对Bean实例进行后置处理。这一步可以用来对Bean实例进行额外的配置或者代理。

使用：当应用程序需要使用Bean时，Spring容器会将Bean实例提供给应用程序使用。

销毁：当应用程序关闭或者Spring容器关闭时，Spring容器会调用Bean的销毁方法（如果有）来完成Bean的销毁。销毁方法可以通过@PreDestroy注解或者在Bean定义中指定。

以上就是Spring Bean的生命周期。在整个生命周期中，Spring容器负责管理和维护Bean实例，确保它们能够正常工作。

# 二、spring事务的传播机制

Spring事务传播机制定义了当一个事务方法被另一个事务方法调用时，事务如何在这两个方法之间传播。Spring提供了7种事务传播行为：

PROPAGATION_REQUIRED：如果当前没有事务，就新建一个事务；如果当前存在事务，就加入该事务。这是默认的传播行为。

PROPAGATION_SUPPORTS：如果当前存在事务，就加入该事务；如果当前没有事务，就以非事务方式执行。

PROPAGATION_MANDATORY：如果当前存在事务，就加入该事务；如果当前没有事务，就抛出异常。

PROPAGATION_REQUIRES_NEW：新建一个事务，并暂停当前的事务（如果存在）。

PROPAGATION_NOT_SUPPORTED：以非事务方式执行，并暂停当前的事务（如果存在）。

PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，就抛出异常。

PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务中执行；如果当前没有事务，就新建一个事务。

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

如果A方法在事务中执行，那么B方法会加入A方法的事务中执行。 如果A方法不在事务中执行，那么B方法会新建一个事务并在其中执行。 PROPAGATION_SUPPORTS：

如果A方法在事务中执行，那么B方法会加入A方法的事务中执行。 如果A方法不在事务中执行，那么B方法也不在事务中执行。 PROPAGATION_MANDATORY：

如果A方法在事务中执行，那么B方法会加入A方法的事务中执行。 如果A方法不在事务中执行，那么B方法会抛出异常。 PROPAGATION_REQUIRES_NEW：

无论A方法是否在事务中执行，B方法都会新建一个事务并在其中执行。如果A方法在事务中执行，那么A方法的事务会被暂停，直到B方法的事务完成。 PROPAGATION_NOT_SUPPORTED：

无论A方法是否在事务中执行，B方法都不在事务中执行。如果A方法在事务中执行，那么A方法的事务会被暂停，直到B方法完成。 PROPAGATION_NEVER：

如果A方法不在事务中执行，那么B方法也不在事务中执行。 如果A方法在事务中执行，那么B方法会抛出异常。 PROPAGATION_NESTED：

如果A方法在事务中执行，那么B方法会在一个嵌套事务中执行。嵌套事务是一个子事务，它可以独立于父事务进行提交或回滚。 如果A方法不在事务中执行，那么B方法会新建一个事务并在其中执行。 以上就是根据不同的传播行为，A调用B时可能出现的各种情况。

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
当从容器中获取FactoryBean类型的Bean时，会得到FactoryBean的实例本身，而不是通过FactoryBean创建的Bean实例。如果想获取FactoryBean创建的Bean实例，需要在Bean名称前加上"&"前缀。
例如，考虑一个名为"MyServiceFactoryBean"的FactoryBean实现，它创建类型为"MyService"的Bean实例。当从容器中获取"MyService"时，实际上会得到"MyServiceFactoryBean"
的实例。要获取由"MyServiceFactoryBean"创建的"MyService"实例，需要使用"&myService"来获取。

总之，BeanFactory是Spring的核心接口，负责管理Bean的创建和生命周期。而FactoryBean是BeanFactory的一种特殊实现，用于创建和管理特定类型的Bean实例，提供了更高级别的功能和自定义逻辑的能力。