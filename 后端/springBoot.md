# 使用maven快速搭建springboot项目

##### 第一步：新建一个maven项目

##### 第二步：引入jar依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-parent</artifactId>
        <version>2.1.1.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.1.1.RELEASE</version>
    </dependency>
</dependencies>
```

##### 第三步：编写主程序类

```java
@SpringBootApplication
public class HelloMainApplication {
    public static void main(String[] args) {
        SpringApplication.run(HelloMainApplication.class, args);
    }
}
```

##### 第四步：编写controller方法并启动进行访问即可

# springboot的快速部署：（jar方式）

##### 第一步：引入maven插件

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

##### 第二步：点击打包

![image-20200112140116829](C:\Users\xiaopu\AppData\Roaming\Typora\typora-user-images\image-20200112140116829.png)

# springboot中yaml文件的基本语法

### 三种语法形式

#### 1.普通键值对

格式为*<u>【配置项】：值</u>*

```
spring:
  mvc:
    servlet:
      load-on-startup: 1
```

#### 2.对象和map类型

格式为*【对象类型】：属性键值对*

```
student:
  name: 李明
  age: 23
  grade: 六年级一班
```

行内写法

```
student: {name: 李明,age: 23,grade: 五年级二班}
```

#### 3.数组、集合类型（array，list）

格式为*【数组/集合名】：数组元素*

```
hobby:
    - 唱
    - 跳
    - rap
    - 篮球
```

行内写法

```
hobby: [唱, 跳, rap, 篮球]
```

### 总结

1.yaml文件以数据为中心，根据换行缩进来表示层级关系

2.要十分注意的是，键值对之间要有一个空格，否则无法生效

3.对与同一个设置下的属性配置，缩进之后列对齐的表示为同一层级的设置（如上面学生类型的属性为同一级）

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

@ConfigurationProperties(prefix = "dog")：作用是让springboot将拥有本注解的实体类和配置文件中的相关属性相互绑定，从而实现数据注入，prefix属性的作用就是指明将配置文件中的哪些数据进行绑定，默认从全局配置文件中读取数据

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

|                      | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持     |
| SpEL                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |

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

# 在springboot中使用springsecurity

在springboot中使用springsecurity可以说十分的简单，springboot通过很多的自动配置让springsecurity只需引入jar后就能产生效果，通过简单的配置就能完成很多安全的功能。

## 过滤web请求

springsecurity通过一系列的Servlet Filter(过滤器)来提供各种安全性功能。因此我们想可能需要在web.xml上或者Java配置类上配置大量的filter，但实际上因为spring的存在，只需配置一个Filter就行了。

DelegatingFilterProxy是一个特殊的Filter，本质上是一个代理，也就是说它本身是不做任何功能上的处理的。而是将需要做的工作交给一个javax.servlet.Filter的实现类去做，而这个实现类，就像我们平时在Java配置类中注册的Bean一样，他会放入spring容器中供我们使用。

在非springboot应用中，我们需要手动配置这个实现类，名为springSecurityFilterChain（就是上面说的Filter的实现）,DelegatingFilterProxy会将过滤请求的事情都交给它去做。但是在springboot中，这一步已经自动帮我们配置好了。

所以当你有一个index页面的时候你就可以启动应用并访问了，springsecurity会生成一个用户名为user，默认密码会打印到控制台上，你的所有请求都会被过滤并拦截，最后，你需要使用user和密码来登录并获得认证之后才能正常访问请求。

## 基本配置

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

}
```

通过继承WebSecurityConfigurerAdapter类即可实现并覆盖相应的方法即可完成对springsecurity的配置，实际上只要是实现了WebSecurityConfigurer接口的类都能对springsecurity进行配置，而WebSecurityConfigurerAdapter就是这个接口的实现。

对于web应用，我们通常经过重载这三个方法来配置安全属性   

| 方法                                    | 描述                                             |
| --------------------------------------- | ------------------------------------------------ |
| configure(AuthenticationManagerBuilder) | 覆盖此方法：配置用户细节相关的配置               |
| configure(WebSecurity）                 | 覆盖此方法：配置springsecurity的Filter链         |
| configure(HttpSecurity)                 | 覆盖此方发：配置拦截器拦截哪些请求和放开哪些请求 |

### 基于内存的用户认证配置

通过重载configure(AuthenticationManagerBuilder)方法，设置其中的相应属性就能完成对用户细节的配置

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("user").password("123").roles("user")
                .and()
                .withUser("admin").password("123").roles("user","admin");
    }
}
```

基于内存的用户配置详细方法

| 方法               | 描述                       |
| ------------------ | -------------------------- |
| accountExpired     | 定义账号是否已经过期       |
| accountLocked      | 定义账号是否已经锁定       |
| and                | 链接配置                   |
| authorities        | 授予某个用户一项或多项权限 |
| credentialsExpired | 定义凭证是否已经过期       |
| disabled           | 定义账号是否已经被禁用     |
| password           | 定义用户密码               |
| roles              | 赋予用户一项或多项权限     |

### 基于数据库的用户认证配置

使用基于数据库的用户认证，只需要我们注入一个数据源即可，spring boot的数据源可以通过yml配置文件配置，spring会识别到它并放入到容器中。也可以通过JavaConfig来显示配置。

这样一个默认的基于配置数据库的数据认证就可以了，此时是默认的配置，也就是说你需要在数据库中有和他对应的表，字段要一样，这样它才能自动从数据库中查询到用户信息并认证成功。

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private DataSource dataSource;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.jdbcAuthentication()
                .dataSource(dataSource)
                .passwordEncoder(new BCryptPasswordEncoder());
    }

}
```

#### 默认配置的查询语句

当你只配置了数据源的时候，他会使用如下的sql语句来查询 

- 查询用户

  ```java
  public static final String DEF_USERS_BY_USERNAME_QUERY = "select username,password,enabled from users where username = ?";
  ```

- 查询权限

  ```java
  public static final String DEF_AUTHORITIES_BY_USERNAME_QUERY = "select username,authority from authorities where username = ?";
  ```

- 查询用户群组权限

  ```java
  public static final String DEF_GROUP_AUTHORITIES_BY_USERNAME_QUERY = "select g.id, g.group_name, ga.authority from groups g, group_members gm, group_authorities ga where gm.username = ? and g.id = ga.group_id and g.id = gm.group_id";
  ```

如果你数据库中的表都满足上面所说的字段和表要求，则只配置一个数据源就能完成用户的认证和权限操作了。

#### 重写默认用户查询

如果想在用户认证操作上拥有更多的自主控制权，那么可以尝试按照自己的意愿重写这些查询语句。

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private DataSource dataSource;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception { 
        auth.jdbcAuthentication()
                .dataSource(dataSource)
                .usersByUsernameQuery("select usernae,password,from users where username =?")
                .authoritiesByUsernameQuery("select username,authority from authorities where username = ?")
                .groupAuthoritiesByUsername("select g.id, g.group_name, ga.authority from groups g, group_members gm, group_authorities ga where gm.username = ? and g.id = ga.group_id and g.id = gm.group_id");
        

    }
}
```

通过以上三个方法就可覆盖默认的查询语句，这里的查询语句完全自定义，但是还需要遵守一些规则。

1. 所有的查询都将用户名（username）作为唯一参数
2. 认证查询会选取用户名、密码、启用状态信息
3. 权限查询会查询该用户的权限信息
4. 群组权限查询会查询群组id、群组名、以及权限

这些都是必要的字段，请不要丢失。

### 密码加密

在数据库中使用明文存储用户密码是一件十分危险的事情。所以注册用户的时候通常会将密码字段转码成密文进行存储，

在springsecurity中密码的认证是这样的，用户注册的是明文密码，传入后端后进行加密，所以在数据库中存储的是密文，而在认证的过程中，后端会获取全段传入的明文密码，以当时注册时候的加密方式来编码此明文密码，转码形成密文后和数据库中的密文密码进行匹配。

为了实现这个过程需要使用passwordEncoder()方法来指定一个密码转换器

```java
auth.jdbcAuthentication()
        .dataSource(dataSource)
        .passwordEncoder(new BCryptPasswordEncoder());
```

此时，认证的时候就会以特定的方式先对密码进行转码。

passwordEncoder()方法可以接收springsecurity中PasswordEncoder接口的任意实现，springsecurity加密模块一共包含了三个这样的实现。

- BCryptPasswordEncoder	
- NoOpPasswordEncoder
- StandardPasswordEncoder

所以在注册的时候可以使用这三个实现的任意一个

## 拦截请求

对于一个安全的web应用来说，很多请求在没有经过认证的时候是不应该被访问的，比如用户信息，因为这个时候根本就没有用户信息。所以进入这个页面毫无意义。

因此需要对一部分请求进行拦截，让其无法访问并提示他应该经过登录才行。

对于请求的拦截，需要重写configure(HttpSecurity)方法

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    public void configure(HttpSecurity http) throws Exception{
        http.formLogin()
                .and()
                .authorizeRequests()
                .antMatchers("/page/**").authenticated()
                .anyRequest().permitAll();
    }

}
```

antMatcher()方法是匹配需要保护的路径必须使用的方法。antMatchers()方法中的路径支持通配符和正则表达式

除了路径选择方法antMatchers()，还能通过authenticated()方法来要求这个路径必须是经过认证的用户才能访问。而permitAll()方法则可以放开权限让所有用户都能访问。

除了以上方法，还有其他方法来对路径拦截进行规范。列表如下：

| 方法                          | 描述                                               |
| ----------------------------- | -------------------------------------------------- |
| access(String)                | 给定的SpEL表达式计算结果为true，就允许访问         |
| anonymous()                   | 允许匿名用户访问                                   |
| authenticated()               | 允许认证过的用户访问                               |
| denyAll()                     | 无条件拒绝所有访问                                 |
| fullyAuthenticated()          | 如果用户是完整的认证（不通过rememberMe）就允许访问 |
| hasAnyAuthority(String . . .) | 如果用户具备给定全险中的某一个，就允许访问         |
| hasAnyRole(String . . .)      | 如果用户具备给定角色中的某一个，就允许访问         |
| hasAuthority(String)          | 如果用户具备给定权限，就允许访问                   |
| hasIpAddress(String)          | 如果用户具备给定的ip地址，就允许访问               |
| hasRoles(String)              | 如果用户具备给定的角色，允许访问                   |
| not()                         | 对其他方法的结果求反                               |
| permitAll()                   | 无条件允许所有访问                                 |
| rememberMe()                  | 如果是通过rememberMe功能进行认证的，允许访问。     |

配置原则：越具体的路径配置在最前面，越不具体的路径，配置在最后面，防止后面的覆盖前面的配置而失效。

## ？强制使用安全通道

使用http传输数据的时候，往往是很不安全的，而https的方式则使用加密传输，相比之下更为安全，我们可以通过指定的方法来指定，哪些请求使用https传输，而哪些使用http传输。

```java
@Override
public void configure(HttpSecurity http) throws Exception{
    http.formLogin()
            .and()
            .authorizeRequests()
            .antMatchers("/page/**").authenticated()
            .anyRequest().permitAll()
            .and()
            .requiresChannel()
            .antMatchers("/").requiresSecure();
}
```

使用此种方式会重定向并使用https，但是此处貌似需要重新设置端口号，现在为止还没有成功，故这里还有问题

## 认证用户

### 默认登录页面

如果我们使用了springsecurity的默认配置的，并且没有重写configure(HttpSecurity）方法，那么springsecurity会为我们提供一个默认的登录页面，尽管不好看，但是比较有效。

但是一旦我们重写了configure(HttpSecurity）方法，这个默认页面就消失了。不过我们仍然可以找回来

```java
@Override
public void configure(HttpSecurity http) throws Exception{
    http.formLogin();
}
```

使用formLogin()方法即可。

### 自定义用户界面

使用formLogin.loginPage()方法即可指定自己的登录界面地址

使用loginProcessingUrl()方法即可指定表单验证的地址，验证逻辑有springsecurity提供。

```java
@Override
public void configure(HttpSecurity http) throws Exception{
    http.formLogin().loginPage("/page/toLogin").loginProcessingUrl("/login");
}
```

使用usernameParameter()方法可以设置接收用户名的key值，使用passwordParameter()方法指定接收password的key值，这里设置的值要和前端表单中的name属性一致，但是也不要混淆，这里是接收前端的值，验证数据库中的值使用的字段还是username或者你自己自定义的名字，两者并不冲突。

```java
@Override
public void configure(HttpSecurity http) throws Exception{
    http.formLogin().loginPage("/page/toLogin").loginProcessingUrl("/login")
            .usernameParameter("user")
            .passwordParameter("pwd");
}
```

### 开启rememberMe功能

开启rememberMe功能十分简单，对于默认的登录页面来说，你只需进行如下配置就行了。

```java
@Override
public void configure(HttpSecurity http) throws Exception{
    http.rememberMe();
}
```

但是对于自己的登录页，需要添加如下的html元素

```html
记住我：<input type="checkbox" name="remember-me">
```

### 退出

在默认配置下，登出功能已经注册并且可以使用，只需要访问 “/logout” 地址就能实现退出

但是如果想自定义退出后的页面，则需要如下配置

```java
@Override
public void configure(HttpSecurity http) throws Exception{
    
    http.logout().logoutSuccessUrl("/");
}
```

### 与thymeleaf的整合

整合之前需要导入整合包，版本自己控制

```xml
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity4</artifactId>
    <version>3.0.2.RELEASE</version>
</dependency>
```

在html中引入命名空间

```thml
<html lang="en" xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity4">
```

现在就可以使用一些springsecurity和thymeleaf整合的标签或者属性来有选择的显示页面元素了。

显示用户名可以使用这个属性

```html
<span sec:authentication="name"></span>
```