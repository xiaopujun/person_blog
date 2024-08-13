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

##### 看似只需要一层缓存就行了，为什么不叫一级缓存？二级缓存？为什么是三级缓存？

孩子，你本身创建出来的这些单例Bean是不是就需要缓存？
不然难道每次都重新创建嘛？那就不叫单例Bean了。因此就算没有循环依赖，创建出来的Bean也需要缓存一份。而确实，二级缓存就可以解决循环依赖的问题。
但是还有一些特殊型情况。如下代码，加入A依赖的不是纯正的B对象，而是B对象的代理对象，又当如何？分析一下之后你会发现，就算是要注入代理对象，二级缓存也未尝不可吧？难道二级缓存就不能实现代理对象的循环依赖问题了嘛？
我的理解是，可以！但是这其中有一个设计问题，或者说代码的健壮性问题。对于这样的一个通用性框架，你要不要把是否创建代理对象的逻辑直接写到属性注入的逻辑里去？是不是要写死？我认为，Spring肯定不会这么干。原因很简单，
不是所有的对象都需要代理对象。你要创建代理对象，你最好自己判断。而如何自己判断？如果说要判断是否创建代理对象的这个过程你能向参数一样传递进来，是否扩展性就大大提高了。因为传入了一个方法，那么不管可以判断是否创建的是代理对象。
其他任何情况也都可以判断。而在二级缓存中Map的value是Object，这过于粗暴。要执行一些接口方法还需要强制转型，并且二级缓存他的目标也很清晰，是存储半成品的地方。所以不要和这些其他的逻辑搞在一起。因此三级缓存的出现最好不过。
三级缓存的Map中的value是Bean工厂类型。最重要的时候他是一个接口。接口就满足我们上面说的扩展性要求。因此，我想这才是设计三级缓存更合适的考量。

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

- **PROPAGATION_REQUIRED**： 如果A方法在事务中执行，那么B方法会加入A方法的事务中执行。 如果A方法不在事务中执行，那么B方法会新建一个事务并在其中执行。
- **PROPAGATION_SUPPORTS**： 如果A方法在事务中执行，那么B方法会加入A方法的事务中执行。 如果A方法不在事务中执行，那么B方法也不在事务中执行。
- **PROPAGATION_MANDATORY**： 如果A方法在事务中执行，那么B方法会加入A方法的事务中执行。 如果A方法不在事务中执行，那么B方法会抛出异常。
- **PROPAGATION_REQUIRES_NEW**： 无论A方法是否在事务中执行，B方法都会新建一个事务并在其中执行。如果A方法在事务中执行，那么A方法的事务会被暂停，直到B方法的事务完成。
- **PROPAGATION_NOT_SUPPORTED**： 无论A方法是否在事务中执行，B方法都不在事务中执行。如果A方法在事务中执行，那么A方法的事务会被暂停，直到B方法完成。
- **PROPAGATION_NEVER**： 如果A方法不在事务中执行，那么B方法也不在事务中执行。 如果A方法在事务中执行，那么B方法会抛出异常。
- **PROPAGATION_NESTED**： 如果A方法在事务中执行，那么B方法会在一个嵌套事务中执行。嵌套事务是一个子事务，它可以独立于父事务进行提交或回滚。
  如果A方法不在事务中执行，那么B方法会新建一个事务并在其中执行。

以上就是根据不同的传播行为，A调用B时可能出现的各种情况。

对于PROPAGATION_NESTED，特别注意他在嵌套事务的父事务和子事务的传播行为
在Spring事务的传播机制中，`PROPAGATION_NESTED`传播行为下，设A为父事务，B、C为子事务，父事务和子事务的行为如下：

#### 1. 正常提交情况

- **A提交，B和C也提交：**
    - 如果事务B成功执行并提交，那么它的更改将被保存。
    - 同样，如果事务C成功执行并提交，它的更改也将被保存。
    - 最后，如果父事务A提交，则所有的更改（包括B和C所做的更改）都会被持久化到数据库中。

#### 2. 子事务异常情况

- **A提交，但B或C中有一个异常：**
    - 如果事务B执行过程中发生异常并且没有被显式地提交或回滚，那么它会自动回滚，但不会影响父事务A的状态。
    - 相同地，如果事务C执行过程中发生异常并且没有被显式地提交或回滚，它也会自动回滚。
    - 父事务A可以继续执行，并且如果成功提交，除了发生异常的子事务外，其他子事务的更改仍然会被保存。

#### 3. 父事务异常情况

- **A回滚，B和C也回滚：**
    - 如果父事务A执行过程中发生异常或者显式地选择回滚，那么所有嵌套在其内的子事务（包括B和C）都将被回滚。
    - 即使B和C都已成功提交，一旦父事务A回滚，所有子事务的更改都将丢失。

#### 4. 子事务回滚对父事务的影响

- **B或C回滚，A是否回滚取决于业务逻辑：**
    - 如果子事务B或C回滚，这并不会自动导致父事务A回滚。
    - 是否让父事务A回滚取决于业务逻辑以及你如何处理这些异常。通常情况下，如果子事务中的错误被认为重要到足以影响整个事务的完整性，那么你可能需要在父事务中捕获异常并决定是否回滚整个事务。

#### 技术实现层面

- 在使用`PROPAGATION_NESTED`时，数据库必须支持保存点（savepoint），这样才能够实现子事务的嵌套特性。大多数关系型数据库系统如MySQL、PostgreSQL等都支持保存点功能。

总结来说，在`PROPAGATION_NESTED`模式下，每个子事务都有独立的回滚边界，但它们的最终状态取决于父事务的状态。如果父事务选择回滚，那么所有子事务的更改都会被撤销。

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

代理对象是对象级别的代理。而JDK动态代理基于接口，Cglib动代理是基于继承的。因此导致final类型的无法进行复写，因此无法支持代理。static更不用说，不是对象级别的方法。无法代理

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

## Spring中的Bean是否线程安全？

这个问题首先看上下文环境。首先是明确一点，是不是多线程环境下就会有问题？显然不是的。只要当多个线程访问一个共享变量的时候才会出现线程安全问题。线程安全问题最根本上是多个线程操作同一份数据导致的数据不一致性问题。
因此这个问题可以拆分为什么情况下多个线程会操作同一份Bean实例对象。

根据上面的推论，结合Bean本身的作用域规则。显然只有在单例Bean或者Application等全局共享同一份Bean实例并且处于多线程环境下才会有线程安全问题。比如多个线程同时操作单例Bean中的一个属性值。反之如果使用的原型作用域，
每次操作也都会创建一个新的实例对象。这根本不会导致多线程之间的资源竞争关系。因此形成不了线程安全的前提条件。

## Spring AOP是什么？

> 使用JVM参数 -Dcglib.debugLocation=D:/dev_project/light-chaser-server/proxy 可以将代理类生成到指定目录下

SpringAOP的代理对象中有用其对应的切面配置，因此可以通过代理对象执行切面方法

面向切面编程（AOP，Aspect Oriented Programming）是一种编程思想，它是对面向对象编程的一种补充⁴。以下是一些关于AOP的重要概念以及他们的作用：

1. **_连接点(JoinPoint)_**：应用中可以被增强的点，一般是方法。
2. **_切入点(PointCut)_** ：实际被增强的连接点。比如应用有100个方法可以被增强，但我实际真实的只增强了其中的3个方法。这3三个方法就是切入点
3. **_通知/增强(Advice)_** ：要增加或插入的具体逻辑代码，通知分为5种：前置、后置、返回、异常、环绕通知
4. **_切面(Aspect)_** ：通知和切入点的结合就是切面
5. **_织入(Weaving)_**：将切面应用到目标方法上的过程，织入有3个时机，编译期、类加载期、运行期。

spring aop功能是spring框架的核心功能之一，他能在不改变源代码的情况下在某一个步骤之前或者之后直接插入一个新的功能，极大的实现了程序的解耦性。

#### spring对AOP的支持

spring框架提供了4种类型的AOP支持

1. Spring AOP

这是Spring框架自带的AOP实现。

特点：

- 基于代理的AOP实现
- 只支持方法级别的连接点
- 可以使用JDK动态代理或CGLIB代理

使用方式：

- 使用@Aspect注解
- 通过XML配置

示例：

```java
@Aspect
@Component
public class LoggingAspect {
    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        // 前置通知逻辑
    }
}
```

2. AspectJ

这是一个完整的AOP框架，Spring可以集成AspectJ。

特点：

- 支持所有级别的连接点（方法调用、字段访问等）
- 性能较好，因为在编译时或类加载时织入
- 功能更强大，但学习曲线较陡

使用方式：

- 可以使用AspectJ注解
- 需要特殊的编译器或类加载时织入

示例：

```java
@Aspect
public class SecurityAspect {
    @Before("execution(* com.example.service.*.*(..)) && @annotation(Secured)")
    public void checkSecurity(JoinPoint joinPoint) {
        // 安全检查逻辑
    }
}
```

3. Spring AOP + AspectJ注解

这是一种混合方式，使用AspectJ的注解，但底层仍然使用Spring AOP的代理机制。

特点：

- 结合了Spring AOP的简单性和AspectJ注解的表达能力
- 仍然只支持方法级别的连接点

使用方式：

- 使用AspectJ的注解
- 无需特殊的编译器或加载时织入

示例：

```java
@Aspect
@Component
public class TransactionAspect {
    @Around("@annotation(Transactional)")
    public Object manageTransaction(ProceedingJoinPoint pjp) throws Throwable {
        // 事务管理逻辑
    }
}
```

4. 基于Schema的AOP（XML配置）

这是一种通过XML配置实现AOP的方式。

特点：

- 不需要在Java代码中使用注解
- 适合需要在配置文件中集中管理切面的场景

使用方式：

- 在XML文件中定义切面、切点和通知

示例：

```xml

<aop:config>
    <aop:aspect id="myAspect" ref="aBean">
        <aop:pointcut id="businessService"
                      expression="execution(* com.xyz.myapp.service.*.*(..))"/>
        <aop:before pointcut-ref="businessService"
                    method="doBeforeTask"/>
    </aop:aspect>
</aop:config>
```

选择建议：

1. 对于大多数Spring应用，使用Spring AOP就足够了，特别是只需要方法级别的AOP时。

2. 如果需要更强大的AOP功能（如字段级别的拦截），可以考虑使用完整的AspectJ。

3. Spring AOP + AspectJ注解是一个很好的折中方案，提供了强大的表达能力，同时保持了Spring AOP的简单性。

4. 基于Schema的AOP适合那些偏好XML配置或需要集中管理AOP配置的项目。

每种方式都有其优缺点和适用场景。选择哪种方式取决于项目的具体需求、团队的技术栈和个人偏好。

#### spring支持的切点指示器

Spring AOP提供了多种切点指示器（Pointcut Designators），用于定义在何处应用切面。以下是主要的切点指示器及其作用：

1. execution()
   作用：匹配方法执行的连接点

   这是最常用的切点指示器，用于匹配方法执行。

   示例：
   ```java
   @Pointcut("execution(* com.example.service.*.*(..))")
   ```
   这匹配com.example.service包中任何类的任何方法。

2. within()
   作用：匹配特定类型内的方法执行

   用于匹配指定类型内的所有方法执行。

   示例：
   ```java
   @Pointcut("within(com.example.service.*)")
   ```
   这匹配com.example.service包中所有类的所有方法。

3. this()
   作用：匹配代理对象为指定类型的连接点

   匹配当前AOP代理对象类型的执行方法。

   示例：
   ```java
   @Pointcut("this(com.example.service.UserService)")
   ```

4. target()
   作用：匹配目标对象为指定类型的连接点

   匹配当前目标对象类型的执行方法。

   示例：
   ```java
   @Pointcut("target(com.example.service.UserService)")
   ```

5. args()
   作用：匹配参数为指定类型的连接点

   用于匹配当前执行的方法传入的参数为指定类型的执行方法。

   示例：
   ```java
   @Pointcut("args(java.lang.String)")
   ```
   这匹配所有只接受一个String参数的方法。

6. @target()
   作用：匹配当前目标对象类型的执行方法，其中目标对象持有指定的注解

   示例：
   ```java
   @Pointcut("@target(org.springframework.stereotype.Service)")
   ```

7. @args()
   作用：匹配当前执行的方法传入的参数持有指定注解的执行

   示例：
   ```java
   @Pointcut("@args(com.example.annotation.Validated)")
   ```

8. @within()
   作用：匹配所有持有指定注解类型内的方法

   示例：
   ```java
   @Pointcut("@within(org.springframework.stereotype.Service)")
   ```

9. @annotation
   作用：匹配当前执行方法持有指定注解的方法

   示例：
   ```java
   @Pointcut("@annotation(org.springframework.transaction.annotation.Transactional)")
   ```

10. bean()
    作用：匹配特定名称的Bean对象的方法

    这是Spring AOP特有的，可以用来匹配特定名称的Bean。

    示例：
    ```java
    @Pointcut("bean(userService)")
    ```

这些切点指示器可以通过逻辑运算符（&&, ||, !）组合使用，以创建更复杂的切点表达式。例如：

```java
@Pointcut("execution(* com.example.service.*.*(..)) && @annotation(org.springframework.transaction.annotation.Transactional)")
```

这个切点匹配com.example.service包中的所有方法，但只有当方法同时带有@Transactional注解时才会被匹配。

理解这些切点指示器及其用法对于有效使用Spring AOP非常重要。它们提供了精确定义切面应用位置的能力，使得AOP的使用更加灵活和强大。

以上的匹配器中，只有execution指示器是用来执行匹配工作的，其他指示器都是用来限制匹配的。

#### 切点表达式详解

```java
execution(* com.ctbu.spring.aop.test.impl.AopImpl.pay(..))
```

execution表示在目标方法执行时触发此切面

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

#### 使用注解声明切面

| 注解              | 通知          |
|-----------------|-------------|
| @After          | 后置通知        |
| @Before         | 前置通知        |
| @AfterReturning | 目标方法返回后调用   |
| @AfterThrowing  | 目标方法抛出异常后调用 |
| @Around         | 环绕通知        |

#### @Aspect

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

#### @Pointcut

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

#### @EnableAspectJAutoProxy

如果就此止步的话，标注有@Aspect注解的类，虽然我们知道它是一个注解类，但实际上他和普通的Java类没什么区别。所以首先应该把他交给spring容器，使用@Component的自动扫描或者显示配置。其次，使用@EnableAspectJAutoProxy注解开启自动代理功能，这样一个切面的配置才算是真正完成

需要注意的一点是，在使用AOP的时候，注入的对象类型最好是接口（父类）类型。否则容易代理错误。

#### 环绕通知

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

#### 通知中的参数

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

## Spring装配Bean有那些方式？

在Spring框架中，主要有以下三种方式来装配Bean：

1. **XML配置文件**：这是最传统的方式，通过在Spring的XML配置文件中使用`<bean>`标签来定义Bean，并使用`<property>`
   或`<constructor-arg>`标签来注入依赖。

2. **注解**：Spring提供了一系列注解来支持基于注解的依赖注入，例如`@Autowired`、`@Resource`、`@Inject`
   等。这些注解可以用在字段、构造器、或者setter方法上。

3. **Java配置**：Spring 4开始，支持使用Java类作为配置类，通过`@Configuration`、`@Bean`、`@ComponentScan`等注解来定义Bean和装配依赖。

以上就是Spring中装配Bean的主要方式，每种方式都有其适用的场景，可以根据实际需求来选择

## BeanFactory和FactoryBean的关系？

BeanFactory和FactoryBean是Spring框架中的两个重要概念，它们之间存在一些关系，但功能和作用是不同的。

1. **BeanFactory**：
    - BeanFactory是Spring框架中最基本的工厂接口，用于管理Spring容器中的bean。
    - 它负责实例化、配置、组装bean以及管理bean的生命周期和作用域。
    - BeanFactory是延迟初始化的，只有在第一次请求bean时才会创建bean实例。

2. **FactoryBean**：
    - FactoryBean是一个特殊的bean，它允许你自定义bean的创建过程。
    - 实现FactoryBean接口的类可以控制bean的实例化过程，例如，可以返回对象的一个复杂构造的结果，或者返回一个对象的代理。
    - FactoryBean通常用于创建复杂对象，或者需要延迟初始化的对象。

3. **关系**：
   FactoryBean可以被视为BeanFactory的扩展。当你在Spring配置中定义了一个FactoryBean时，Spring容器会调用FactoryBean的`getObject()`
   方法来获取实际的bean实例，而不是直接调用构造函数来创建实例。
    - 你可以将FactoryBean视为一个工厂的工厂，即它是一个创建工厂的工厂，而BeanFactory是管理这些工厂（包括FactoryBean）的容器。

4. **使用场景**：
    - BeanFactory通常用于Spring应用的上下文中，作为获取bean的标准方式。
    - FactoryBean通常用于需要特殊创建逻辑的bean，比如数据库连接池、复杂对象的构造等。

总结来说，BeanFactory是Spring容器的基础，用于管理所有bean的生命周期，而FactoryBean是一个特殊的bean，用于自定义bean的创建逻辑。在某些情况下，FactoryBean可以作为BeanFactory的一部分来使用，以实现更高级的bean创建和管理策略。

## SpringMVC路由的全过程

路由地址到Controller的方法底层数据结构使用HashMap，key为RequestMappingInfo，value为AbstractHandlerMethodMapping$MappingRegistration内部类对象。

1. tomcat容器使用线程池启动一个任务org.apache.tomcat.util.threads.TaskThread.WrappingRunnable#run

2. 执行tomcat容器中过滤器 org.apache.catalina.core.ApplicationFilterChain#doFilter
    1. 此处包含自定义注册的过滤器

3. 进入servlet容器中
    1. jakarta.servlet.Servlet#service
    2. org.springframework.web.servlet.FrameworkServlet#service
    3. jakarta.servlet.http.HttpServlet#doPost
    4. 需要注意的是`DispatcherServlet extend FrameworkServlet extend HttpServlet`

4. 进入请求分发
    1. org.springframework.web.servlet.DispatcherServlet#doService
    2. org.springframework.web.servlet.DispatcherServlet#doDispatch

5. 获取Controller处理器 org.springframework.web.servlet.DispatcherServlet#getHandler，其中使用SpringIOC容器

6. 获取到目标Controller的目标方法，并传递参数
   org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter#handleInternal

7. 使用反射调用Controller方法
