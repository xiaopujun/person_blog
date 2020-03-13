# spring注解

###### @Configuration

声明被标注的类是一个spring的配置类；



###### @Import

使用spring中的java配置类进行配置的时候，有多个配置类可以使用@Import注解将其他配置类引入到某一个配置类中。



###### @ImportResource

如果希望同时使用XML配置和Java类配置的时候，可以在Java类中使用此注解，将XML中的配置进行引入

```
@ImportResource("classpath:application.xml")
```



###### @ContextConfiguration

会指明spring到哪里加载配置，如果是配置类的会这样配置

```
@ContextConfiguration(classes = CdPlayerConfig.class)
```

这样spring就会自动到CdPlayerConfig.class去加载配置



###### @Bean

声明被标注的类或方法的返回值会成为spring的一个组件；



###### @Component

声明被标注的类会成为spring的一个组件



###### @Named（基本不用）

声明被标注的类会成为spring的组件，和@Component差不多，有一点差别，都支持自定义命名；



###### @Autowired（自动装配/注入）

会自动将对应类的bean（实例）注入到被标注的变量上



###### @Inject（基本不用）

和@Autowired几乎一样的功能，来自java规范，在spring中也得到了支持



###### @ComponentScan

开启组件扫描功能，会自动扫描带有@Component注解的类，并创建对应类的bean（实例），一般用在配置类上，本注解会扫描本类所在的包以及本包下所有子包的类，如果不满足需求，自己指定需要扫描的包

```
@ComponentScan(value = {"com.ctbu"})
```



###### @EnableAspectJAutoProxy

开启切面自动代理，用于开启spring aop功能，用在配置类上；



###### @Aspect

声明一个切面类，表明这个类是一个切面，用在类上；



###### @Before

spring aop的注解，表明被标注的方法，将在指定切点方法的前面执行，用在方法上。具体用法如下

```
@Before(value = "execution(* person.BraveKnight.embarkOnQuest(..))")
```



###### @After

spring aop的注解，表明被标注的方法，将在指定切点方法的后面执行，用在方法上。具体用法如下

```
@After(value = "execution(* person.BraveKnight.embarkOnQuest(..))")
```



###### @Around

spring aop的注解，被表示的方法为环绕通知，将被通知方法完全封装。

```
@Around("performance()")
public void watchPerformance(ProceedingJoinPoint pj){
    try {
        System.out.println("观众就坐");
        System.out.println("观众手机静音");
        pj.proceed();
        System.out.println("观众鼓掌");
    } catch (Throwable throwable) {
        System.out.println("观众要求退票");
        throwable.printStackTrace();
    }
}
```



###### @Pointcut

在面向切面编程中指定一个具体的切点，方便引用

```
@Pointcut("execution(* aop.service.impl.Dance.perform(..))")
```



###### @DeclareParents

此注解为aop注解，作用是将两个类抽象为一个类，使得其中一个目标类的方法得到增强。这两个类之间可以相互转型。



###### @RunWith(SpringJUnit4ClassRunner.class)

spring的测试注解，使用junit4进行测试的时候，spring会自动创建容器（应用上下文）



###### @ContextConfiguration

spring测试注解，使用测试的时候告诉spring从哪里加载配置（从配置类，还是xml文件），他的值要自己指定，也就是配置文件或者配置类的地址



###### @Profile

spring中用来限定某些bean在特定条件下才会被创建，

```
@Profile("dev")
```

如上，表示被标注的这个bean只有在dev的状态下才会被创建

本注解可以标注在类上，也可以标注在方法上



###### @ActiveProfiles

用在集成测试的使用，用于激活特定条件下的bean



###### @Conditional

本注解用于条件化创建bean实例，只有当满足本注解中某些值得条件下，猜会创建标注了本注解得这些Bean。传值方式如下；

```
@Conditional(BlankDisc.class)
```

传入的这个类应该继承自Condition接口



###### @Qualifier

限定词注解，可以用来限定注入那种类型的bean，或者其他限定用途，比如在注入实例的时候可以限定注入某一种特定类型的bean



###### @Primary

标注装配的首选Bean，当自动装配实例的时候，有多个类型可以选择，此时spring无法做出合理选择，需要用优先级加以区分，此注解标识的bean说明是优先选择的。可以和全自动装配组合使用，也可以和Java配置类的显示配置组合使用。



###### @Scope

用来指定被标注的bena的范围，是单例的，还是原型的。有四个级别，单例、原型、会话、请求

```
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
```



###### @PropertySource

用来标注从指定配置文件中加载数据

```
@PropertySource("classpath:app.properties")
```



###### @EnableWebMvc

springmvc中的注解，用于启动springmvc



###### @Controller

springmvc中的注解，用于表示控制器