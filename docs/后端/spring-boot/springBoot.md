## SpringBoot启动过程的各个阶段

### 启动初始化阶段

`org.springframework.boot.SpringApplication#SpringApplication(org.springframework.core.io.ResourceLoader, java.lang.Class<?>...)`

- **启动点**：Spring Boot应用通常从包含`main`方法的主类开始，该类通常使用`@SpringBootApplication`注解。
- **创建`SpringApplication`实例**：
    - 加载META-INF/spring.factories文件中的实现类并缓存（其中包含自动配置相关实现类）
    - 设置初始化器和监听器（Initializers/Listeners）`ApplicationListeners`和`ApplicationContextInitializers`
      。这些组件会响应特定的应用事件或对上下文进行额外配置。
    - 推断主应用类
    - 确定应用类型（SERVLET、REACTIVE、NONE）

### 环境准备阶段

```java 
Startup startup = Startup.create();
if (this.registerShutdownHook) 
    SpringApplication.shutdownHook.enableShutdownHookAddition();
DefaultBootstrapContext bootstrapContext = createBootstrapContext();
ConfigurableApplicationContext context = null;
configureHeadlessProperty();
SpringApplicationRunListeners listeners = getRunListeners(args);
listeners.starting(bootstrapContext, this.mainApplicationClass);
try {
    ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
    ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
    Banner printedBanner = printBanner(environment);
}
```

- 创建和配置环境（Environment）
    - 加载application.properties/application.yml配置文件
        - 先加载bootstrap配置文件（如果使用Spring Cloud）
        - 然后按优先级加载配置文件：
            - 命令行参数
            - Java系统属性
            - OS环境变量
            - application-{profile}.properties/yml（外部）
            - application.properties/yml（外部）
            - application-{profile}.properties/yml（内部）
            - application.properties/yml（内部）
- 加载系统属性和环境变量
- 处理命令行参数
- 打印启动Banner

### 创建并预处理应用上下文阶段

```java 
context = createApplicationContext();
context.setApplicationStartup(this.applicationStartup);
prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
```

- **创建`ApplicationContext`**
  ：根据应用类型（如Servlet、Reactive）创建具体的上下文实现（如`AnnotationConfigServletWebServerApplicationContext`）。
    - 创建IOC容器BeanFactory(beanDefinitionMap)
- **上下文预处理**
    - 将环境（Environment）设置到应用上下文
    - 添加BeanFactoryPostProcessor
    - 创建BeanDefinitionLoader
    - 添加启动类到IOC容器中（注册主配置类的BeanDefinition）

### 刷新上下文阶段（视应用复杂度而定）

这是最复杂和耗时的阶段，主要完成以下工作：

![springboot刷新上下文.png](springboot刷新上下文.png)

#### 预刷新

- 处理配置文件和上下文环境中的占位符

#### 加载`BeanDefinition`

##### 加载配置

- 处理@Configuration类
- 处理@Import注解，导入配置
- 处理@ImportResource注解，导入XML配置

##### 处理注解

- 处理`@ComponentScan`注解，根据`@ComponentScan`扫描类路径，找到带有`@Component`、`@Service`、`@Repository`、`@Controller`
  等注解的类，注册为bean定义`BeanDefinition`。加载的方式包括注解扫描、XML配置等。
- **处理`BeanFactoryPostProcessor`**：调用所有的`BeanFactoryPostProcessor`，如`ConfigurationClassPostProcessor`
  ，这些处理器可以修改bean定义的元数据。

##### 自动配置

- **处理`@EnableAutoConfiguration`**
  ：加载自动配置类并注册相关bean定义。加载META-INF/spring.factories中的自动配置类，根据条件注解（@Conditional）判断是否需要自动配置

#### 初始化并创建bean

- **初始化单例bean**：Spring容器开始创建和初始化所有单例bean，包括依赖注入、属性设置、初始化方法调用等。
- **处理`@PostConstruct`和`@PreDestroy`注解**：执行生命周期回调方法。

### 启动嵌入式Web服务器

- **启动Tomcat、Jetty或Netty**：如果是一个Web应用，Spring
  Boot会启动嵌入式的Web服务器（如Tomcat）。这个过程包括创建`ServletContext`、注册`DispatcherServlet`等。
- **注册Servlet和Filter**：注册所有的Servlet、Filter和监听器（Listener）。

### 运行应用

- **发布`ApplicationReadyEvent`事件**：此时应用已经准备就绪，可以接受请求或执行其他任务。
- **启动`CommandLineRunner`和`ApplicationRunner`**：如果有实现了`CommandLineRunner`或`ApplicationRunner`
  接口的bean，会在应用启动后立即执行它们的`run`方法。

### 应用运行中

- **处理请求**：对于Web应用，处理来自客户端的HTTP请求。
- **应用运行**：持续运行，等待处理请求或执行其他任务，直到应用被手动停止。

### 关闭阶段

- **发布`ContextClosedEvent`事件**：在应用上下文关闭时，发布事件。
- **销毁单例bean**：调用单例bean的`destroy`方法，清理资源。

### 应用终止

- **释放资源**：关闭所有应用资源，如数据库连接池、线程池等。
- **关闭应用**：关闭应用上下文，停止所有相关服务和组件。

=======================================

PostProcessor（后置处理器）在Spring
Boot中扮演着非常重要的角色，它们允许我们在Bean生命周期的不同阶段对Bean进行修改或增强。这些处理器主要分为两类：BeanPostProcessor和BeanFactoryPostProcessor。

作用：

1. 允许自定义或修改Bean的属性
2. 在Bean初始化前后执行特定的逻辑
3. 替换或修改Bean定义
4. 条件化地修改应用上下文的内部bean工厂

举个例子：
假设我们有一个ConfigurationPropertiesBindingPostProcessor。这个处理器的作用是将配置文件（如application.properties）中的属性值绑定到带有@ConfigurationProperties注解的Bean上。

形象的比喻：
想象Spring Boot是一个大型的玩具工厂，而PostProcessor就像是工厂流水线上的质检员和改装工人。

1. BeanFactoryPostProcessor就像是在玩具设计图纸阶段的质检员。他们可以查看并修改玩具的"设计图"（Bean定义），以确保玩具的设计符合要求。

2. BeanPostProcessor则像是在玩具即将出厂前的改装工人。他们可以在玩具制作完成后，但在送到"商店"（应用程序）之前，对玩具进行最后的调整和改装。

3. 以ConfigurationPropertiesBindingPostProcessor为例，它就像是一个特殊的改装工人，负责给玩具"贴标签"。它会根据"工厂说明书"
   （配置文件）给每个玩具贴上正确的标签（属性值），确保玩具有正确的"参数"。

这些"质检员"和"改装工人"确保了从工厂出来的每个"玩具"（Bean）都是符合规格的，并且具有正确的"配置"和"功能"
。它们的存在使得整个"玩具制作过程"（Spring Boot启动过程）更加灵活和可定制，能够生产出更符合"顾客"（开发者）需求的"玩具"（应用程序）。

PostProcessor模式在Spring框架中的实现主要基于两种设计模式：

1. 观察者模式（Observer Pattern）

2. 模板方法模式（Template Method Pattern）

让我详细解释一下这两种模式如何在PostProcessor中体现：

1. 观察者模式：

PostProcessor本质上是一种观察者模式的实现。在这个模式中：

- Spring容器acts作为主题（Subject）
- 各种PostProcessor acts作为观察者（Observer）

当Spring容器在处理Bean的不同生命周期阶段时，它会通知（调用）相应的PostProcessor。这些PostProcessor"观察"
着Bean的创建和初始化过程，并在适当的时候进行干预。

2. 模板方法模式：

PostProcessor接口定义了一些钩子方法（hook
methods），如postProcessBeforeInitialization和postProcessAfterInitialization。这些方法在Bean的生命周期中的特定点被调用。

Spring提供了这些方法的默认实现（通常是空实现），而具体的PostProcessor可以选择覆盖这些方法来提供自定义行为。这符合模板方法模式的特征，即在抽象类中定义算法的骨架，而将一些步骤的实现延迟到子类中。

举例说明：

```java
public interface BeanPostProcessor {
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}

// 自定义PostProcessor
public class CustomBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        // 自定义前置处理逻辑
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        // 自定义后置处理逻辑
        return bean;
    }
}
```

在这个例子中，BeanPostProcessor接口定义了处理Bean的模板方法，而CustomBeanPostProcessor实现了这些方法来提供具体的处理逻辑。

这种设计允许Spring框架在不修改核心代码的情况下，通过添加新的PostProcessor来扩展其功能，体现了开闭原则。同时，它也提供了一种灵活的机制来干预和自定义Bean的创建和初始化过程。

## SpringBoot中如何让BeanB在BeanA之前被加载

#### 使用依赖关系

```java 
@Component
public class A {
    @Autowired
    private B b;
}
```

在上面的代码中，从依赖关系上看，A依赖于B，因此要完整的加载A，需要先加载B

#### 使用依赖注解@DepensdOn

```java 

@DepensdOn("B")
@Component
public class A {
    @Autowired
    private B b;
} 

```

使用@DepensdOn注解，可以指定BeanB在BeanA之前被加载。

#### 使用BeanFactoryPostProcessor

SpringBoot中启动过程中，会先执行BeanDefinition的注册，其次再执行非懒加载单例Bean的初始化。但是在这中间会执行BeanFactoryPostProcessor钩子。因此只要在这个PostProcessor中初始化目标Bean，他一定会在所有其他Bean初始化之前完成初始化

```java 

@Component
public class PrimaryBeanPostProcessor implements BeanFactoryPostProcessor {
    
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        PrimaryBean bean = beanFactory.getBean(PrimaryBean.class);//调用getBean方法即可，如果没有被初始化的Bean，他会根据BeanDefinition进行初始化，然后缓存
        System.out.println(bean);
    }
}

```

## SpringBoot执行定时任务

#### 启动类上添加注解@EnableScheduling

```java
@SpringBootApplication
@EnableScheduling
public class Application {
      public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

#### 定时任务类交给Spring管理

```java

@Component
public class DelDataScheduleTask {
    @Scheduled(cron = "0 0 13 * * ?")
    public void executeTask() {
        System.out.println("每天下午1点执行任务");
        // 在这里添加你的任务逻辑
    }
}

```

#### 请详细解释cron表达式的每一个元素都有什么作用

Cron表达式是一种用于指定计划任务执行时间的字符串，由多个字段组成，每个字段代表不同的时间单位。以下是每个字段的详细解释：

### Cron表达式的基本结构

一个标准的Cron表达式通常由六个字段组成，按顺序分别表示：

```
秒 分 时 日 月 星期 [年]
```

其中，年字段是可选的。每个字段之间用空格分隔。

### 字段详解

1. **秒（Seconds）**：取值范围为0-59。
    - 示例：`0` 表示每分钟的第0秒执行。

2. **分（Minutes）**：取值范围为0-59。
    - 示例：`0` 表示每小时的第0分钟执行。

3. **时（Hours）**：取值范围为0-23。
    - 示例：`13` 表示下午1点执行。

4. **日（Day of Month）**：取值范围为1-31。
    - 示例：`15` 表示每月15日执行。

5. **月（Month）**：取值范围为1-12，或使用缩写（JAN-DEC）。
    - 示例：`7` 或 `JUL` 表示七月执行。

6. **星期（Day of Week）**：取值范围为0-7（0和7都表示星期日），或使用缩写（SUN-SAT）。
    - 示例：`1` 或 `MON` 表示星期一执行。

7. **年（Year）**：取值范围为1970-2099（可选）。
    - 示例：`2024` 表示在2024年执行。

### 特殊字符

- **`*`（星号）**：表示匹配该字段的所有值。
    - 示例：`* * * * *` 表示每分钟执行一次。

- **`?`（问号）**：只能用于“日”和“星期”字段，表示不指定具体的值。
    - 示例：`0 0 12 * * ?` 表示每天中午12点执行，不关心具体是星期几。

- **`-`（连字符）**：表示一个范围。
    - 示例：`10-12` 表示从10到12点之间每小时执行一次。

- **`,`（逗号）**：表示列出多个值。
    - 示例：`MON,WED,FRI` 表示星期一、三、五执行。

- **`/`（斜杠）**：表示指定增量。
    - 示例：`0/15` 表示从0分钟开始，每15分钟执行一次。

- **`L`（最后）**：只能用于“日”和“星期”字段，表示最后一天或最后一个星期几。
    - 示例：`L` 表示每月的最后一天。

- **`W`（工作日）**：只能用于“日”字段，表示最近的工作日。
    - 示例：`15W` 表示每月15日最近的工作日执行。

- **`#`（第几个星期几）**：只能用于“星期”字段，表示某月的第几个星期几。
    - 示例：`2#1` 表示每月的第一个星期二。

### 示例

- `0 0 12 * * ?`：每天中午12点执行。
- `0 15 10 ? * MON-FRI`：每周一到周五上午10:15执行。
- `0 0 0 1 1 ?`：每年1月1日午夜执行。

通过理解这些字段和特殊字符，你可以灵活地编写Cron表达式来满足各种定时任务的需求

## SpringBoot定时任务与数据库事务结合导致的依赖注入失败问题

正常情况下，当我们的Controller中使用了类似Aop功能，那么实际在使用Controller实例时，是通过生成的代理对象来调用的。大致的过程是，Tomcat容器启动一个线程，通过反射调用代理对象的方法，代理对象再调用原始对象的方法。

考虑这样一个场景：

```java
@Log4j2
@Component
public class DelDataScheduleTask {

    @Resource
    private FileService fileService;
    

    /**
     * 定时任务
     */
    @Scheduled(cron = "0 0 0 ? * SUN")
    @Transactional
    public void executeTask() {
        // 删除过期的图片资源
        deleteImageFile();
    }
    
    private void deleteImageFile() {
        List<FilePO> expiredData = fileService.getExpiredData();
        if (expiredData == null || expiredData.isEmpty())
            return;
        List<String> urls = expiredData.stream().map(FilePO::getUrl).map(url -> GlobalVariables.PROJECT_RESOURCE_PATH + GlobalVariables.SOURCE_IMAGE_PATH + url).toList();
        deletePhysicalFile(urls);
    }

}

```

如上的代码中，定时任务本身是一个通过@Transactional标注的存在事务的方法，因此调用时，应该时通过代理对象来调用这个异步任务的。同时该Bean中存在一个fileService的依赖。这个依赖本身也是一个Bean，且它内部的方法也会使用到事务。此时当定时任务启动时，实际使用的fileService对象也是一个代理对象。
当你真正使用代理对象调用到fileService内部的方法时，可能会出现fileService内部字段没有被依赖注入

```java

@Service
public class FileServiceImpl implements FileService {

    @Resource
    private FileMapper fileMapper;
}
```

比如你会发现fileService内部的这个fileMapper字段是null，因此在执行fileService内部方法的时候会出现空指针异常。经过一些测试，发现如果fileService中没有使用事务注解@Transactional，那么这个问题就不会出现了。据此猜测，我认为应该是在实例化异步任务类的时候，依赖注入没有完整的完成。导致创建出来的代理对象是不完整的。

通过调试一个普通的Controller请求会发现，正常的请求路径是先走早Controller的代理对象，代理对象再调用原始对象的方法。而原始对象的依赖注入是没有问题的。
上面的空指针异常发生在直接使用了代理对象的方法，并没有通过代理对象调用到原始对象的方法中去。而为了让这个异步任务实例的依赖注入完整。我使用了`@DependsOn({"fileServiceImpl"})`
注解。要求在实例化异步任务类的时候，必须吧它依赖的其他service实例完整的创建出来。也就是先实例化A再实例化B。保持这个顺序。

```java 
@DependsOn({"fileServiceImpl"})
@Log4j2
@Component
public class DelDataScheduleTask {

    @Resource
    private FileService fileService;
}
```

之后该为题得到了解决。 这大概验证了上面我的猜测，但是最直接的依赖注入问题现象没有办法直接观察到。只知道不使用此方法，异步任务中的依赖注入确实存在问题