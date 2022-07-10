# spring如何装配Bean

> 有三种方式：

1. 在XML中进行显示配置
2. 在Java配置类中进行显示配置
3. 通过全注解，隐式的注入和自动装配

三种方式优先选择自动配置，其次选择Java配置类配置，最后选择XML配置



#### 全自动化装配Bean

需要用到的注解

> @Configuration

> @ComponentScan

> @Autowired

> @Component

详细步骤：

1. 首先仍然需要一个配置类，贴上@Configuration，里面不需要写啥东西，但是还需要贴上@ComponentScan注解，扫描的范围自定义或者使用默认范围；
2. 给需要放入spring容器的类（组件）贴上@Component注解，这些类就会作为spring的组件放入spring容器中；
3. 在使用到相应组件的时候，相应变量贴上@Autowired注解，就可自动装配注入；

#### 通过java配置类装配Bean

需要用到的注解

> @Configuration

> @Bean

> @Autowired

> @Component

详细步骤：

1. 需要创建java配置类，贴上@Configuration注解，

2. 在配置类中用方法创建具体实例，有参数的直接传参数，spring会自动装配

   ```
   @Bean
   public CdPlayer cdPlayer(CompactDise cd) {
       return new CdPlayer(cd);
   }
   ```

3.在方法上贴上@Bean注解，由此组件就会被放入到容器中

其他使用形式同上

#### 使用XML配置文件装配Bean

装配无需使用注解

详细步骤：

1. 创建一个XML文件，并引入spring的相关约束

   ```
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:c="http://www.springframework.org/schema/c"
          xmlns:P="http://www.springframework.org/schema/p"
          xmlns:util="http://www.springframework.org/schema/util"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/util
          http://www.springframework.org/schema/util/spring-util.xsd">
   
   </beans>
   ```

2.声明一个Bean

```
<bean class="soundsystem.model.impl.CdPlayer" id="cdPlayer">
</bean>
```

3.为声明的Bean初始化属性

传入属性分为**构造器属性**和**基本属性**

构造器属性的传入分为**字面量值**得传入和**引用值**得传入

基本属性也分为**字面量**传入和**引用值**传入

同时传入方式可以用标签进行传入，也能用名称空间进行传入，需要注意得是，标签传入支持集合得装配，名称空间不支持

