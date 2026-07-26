# Spring6 核心技术手册

  **阅读指南**：由浅入深，每个知识点均配有精简示例。建议按顺序阅读，环环相扣。

---

## 目录

[1. Spring6概述](#1-spring6概述)
- [1.1 Spring核心体系结构](#11-spring核心体系结构)
- [1.2 Spring6新特性](#12-spring6新特性)

[2. IoC控制反转](#2-ioc控制反转)
- [2.1 IoC思想](#21-ioc思想)
- [2.2 自定义对象容器](#22-自定义对象容器)
- [2.3 Spring实现IoC](#23-spring实现ioc)
- [2.4 Spring容器类型](#24-spring容器类型)
- [2.5 对象的创建方式](#25-对象的创建方式)
- [2.6 对象的创建策略](#26-对象的创建策略)
- [2.7 对象的销毁时机](#27-对象的销毁时机)
- [2.8 对象的生命周期](#28-对象的生命周期)
- [2.9 获取对象的方式](#29-获取对象的方式)
- [2.10 BeanPostProcessor](#210-beanpostprocessor)
- [2.11 FactoryBean与BeanFactory](#211-factorybean与beanfactory)

[3. DI依赖注入](#3-di依赖注入)
- [3.1 什么是依赖注入](#31-什么是依赖注入)
- [3.2 Setter注入](#32-setter注入)
- [3.3 构造方法注入](#33-构造方法注入)
- [3.4 自动注入](#34-自动注入)
- [3.5 注入Bean类型](#35-注入bean类型)
- [3.6 注入简单数据类型](#36-注入简单数据类型)
- [3.7 注入集合类型](#37-注入集合类型)

[4. 注解实现IoC](#4-注解实现ioc)
- [4.1 @Component](#41-component)
- [4.2 @Repository、@Service、@Controller](#42-repositoryservicecontroller)
- [4.3 @Scope](#43-scope)
- [4.4 @Autowired](#44-autowired)
- [4.5 @Qualifier](#45-qualifier)
- [4.6 @Value](#46-value)
- [4.7 @Bean](#47-bean)
- [4.8 @Import](#48-import)
- [4.9 @Conditional](#49-conditional)
- [4.10 @Primary、@Lazy、@DependsOn](#410-primarylazy-dependson)

[5. SpEL表达式](#5-spel表达式)
- [5.1 SpEL基础语法](#51-spel基础语法)
- [5.2 SpEL在注解中的使用](#52-spel在注解中的使用)
- [5.3 SpEL在XML中的使用](#53-spel在xml中的使用)

[6. Spring整合MyBatis](#6-spring整合mybatis)
- [6.1 搭建环境](#61-搭建环境)
- [6.2 编写配置文件](#62-编写配置文件)
- [6.3 编写Java代码](#63-编写java代码)
- [6.4 单元测试](#64-单元测试)
- [6.5 自动创建代理对象](#65-自动创建代理对象)

[7. AOP面向切面](#7-aop面向切面)
- [7.1 AOP入门](#71-aop入门)
- [7.2 通知类型](#72-通知类型)
- [7.3 切点表达式](#73-切点表达式)
- [7.4 多切面配置](#74-多切面配置)
- [7.5 注解配置AOP](#75-注解配置aop)
- [7.6 原生Spring实现AOP](#76-原生spring实现aop)
- [7.7 Schema-Based实现AOP](#77-schema-based实现aop)

[8. Spring事务管理](#8-spring事务管理)
- [8.1 事务管理方案](#81-事务管理方案)
- [8.2 Spring事务管理器](#82-spring事务管理器)
- [8.3 事务控制的API](#83-事务控制的api)
- [8.4 事务的相关配置](#84-事务的相关配置)
- [8.5 事务的传播行为与隔离级别](#85-事务的传播行为与隔离级别)
- [8.6 注解配置事务](#86-注解配置事务)

[9. Spring事件机制](#9-spring事件机制)
- [9.1 ApplicationEvent事件](#91-applicationevent事件)
- [9.2 ApplicationListener监听器](#92-applicationlistener监听器)
- [9.3 @EventListener注解](#93-eventlistener注解)

[10. Environment与Profile](#10-environment与profile)
- [10.1 Environment环境抽象](#101-environment环境抽象)
- [10.2 Profile多环境配置](#102-profile多环境配置)

[11. Spring Task定时任务](#11-spring-task定时任务)
- [11.1 入门案例](#111-入门案例)
- [11.2 Cron表达式](#112-cron表达式)
- [11.3 Cron实战案例](#113-cron实战案例)
- [11.4 注解配置定时任务](#114-注解配置定时任务)
- [11.5 多线程任务](#115-多线程任务)

---

## 1. Spring6概述

### 1.1 Spring核心体系结构

Spring框架是一个轻量级的Java开发框架，核心模块包括：

- **core**：核心工具类
- **beans**：Bean定义和管理
- **context**：应用上下文
- **tx**：事务管理
- **spel**：表达式语言
- **aop**：面向切面编程

**核心思想**：IoC（控制反转）和AOP（面向切面编程）

```java
// Spring容器启动示例
ApplicationContext context = 
    new ClassPathXmlApplicationContext("applicationContext.xml");
```

### 1.2 Spring6新特性

Spring6要求JDK 17+，带来以下新特性：

- 原生镜像支持（GraalVM）
- 响应式编程增强
- 简化的注解配置
- 更强的类型安全

```java
// Spring6简化的启动方式
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

---

## 2. IoC控制反转

### 2.1 IoC思想

IoC（Inversion of Control）控制反转，将对象的创建和管理权交给容器。

**传统方式**：程序员手动创建对象
```java
// 传统方式：手动创建依赖对象
UserService service = new UserService();
service.setUserDao(new UserDao());
```

**IoC方式**：容器负责创建和注入对象
```java
// IoC方式：从容器获取对象，无需手动创建
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
UserService service = context.getBean(UserService.class);
```

### 2.2 自定义对象容器

手动实现一个简单的IoC容器：

```java
// 自定义容器类
public class SimpleContainer {
    private Map<String, Object> beans = new HashMap<>();
    
    // 注册Bean
    public void register(String name, Object bean) {
        beans.put(name, bean);
    }
    
    // 获取Bean
    public Object get(String name) {
        return beans.get(name);
    }
}

// 使用自定义容器
SimpleContainer container = new SimpleContainer();
container.register("userService", new UserService());
UserService service = (UserService) container.get("userService");
```

### 2.3 Spring实现IoC

Spring通过BeanFactory和ApplicationContext实现IoC：

```java
// 方式1：ClassPathXmlApplicationContext（从类路径加载）
ApplicationContext context1 = 
    new ClassPathXmlApplicationContext("applicationContext.xml");

// 方式2：FileSystemXmlApplicationContext（从文件系统加载）
ApplicationContext context2 = 
    new FileSystemXmlApplicationContext("src/main/resources/applicationContext.xml");
```

### 2.4 Spring容器类型

| 容器类型 | 说明 |
|---------|------|
| BeanFactory | 基础容器，延迟加载 |
| ApplicationContext | 增强容器，立即加载 |
| WebApplicationContext | Web环境专用容器 |

```java
// BeanFactory延迟加载示例
BeanFactory factory = new XmlBeanFactory(new ClassPathResource("applicationContext.xml"));
// 此时Bean尚未创建，调用getBean时才创建
UserService service = factory.getBean(UserService.class);
```

### 2.5 对象的创建方式

Spring支持多种Bean创建方式：

**方式1：默认构造方法**（最常用）
```xml
<bean id="userService" class="com.example.UserService"/>
```

**方式2：静态工厂方法**
```java
public class UserServiceFactory {
    public static UserService createService() {
        return new UserService();
    }
}
```
```xml
<bean id="userService" class="com.example.UserServiceFactory" 
      factory-method="createService"/>
```

**方式3：实例工厂方法**
```java
public class UserServiceFactory {
    public UserService createService() {
        return new UserService();
    }
}
```
```xml
<bean id="factory" class="com.example.UserServiceFactory"/>
<bean id="userService" factory-bean="factory" factory-method="createService"/>
```

### 2.6 对象的创建策略

**单例模式**（默认）：容器中只有一个实例
```xml
<bean id="userService" class="com.example.UserService" scope="singleton"/>
```

**原型模式**：每次获取时创建新实例
```xml
<bean id="userService" class="com.example.UserService" scope="prototype"/>
```

### 2.7 对象的销毁时机

- **singleton Bean**：容器关闭时销毁
- **prototype Bean**：Spring不负责销毁，由垃圾回收机制处理

```java
// 注册关闭钩子，确保容器正常销毁
AbstractApplicationContext context = 
    new ClassPathXmlApplicationContext("applicationContext.xml");
context.registerShutdownHook();
```

### 2.8 对象的生命周期

完整的Bean生命周期：

1. 实例化（Instantiation）
2. 设置属性（Populate Properties）
3. 前置处理（BeanPostProcessor.postProcessBeforeInitialization）
4. 初始化（InitializingBean.afterPropertiesSet 或 @PostConstruct）
5. 后置处理（BeanPostProcessor.postProcessAfterInitialization）
6. 使用中（In Use）
7. 销毁（DisposableBean.destroy 或 @PreDestroy）

```java
public class UserService implements InitializingBean, DisposableBean {
    
    @Override
    public void afterPropertiesSet() throws Exception {
        // 初始化逻辑
        System.out.println("Bean初始化完成");
    }
    
    @Override
    public void destroy() throws Exception {
        // 销毁逻辑
        System.out.println("Bean销毁");
    }
}
```

### 2.9 获取对象的方式

```java
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

// 方式1：根据Bean名称获取（需要强制类型转换）
UserService service1 = (UserService) context.getBean("userService");

// 方式2：根据类型获取（推荐）
UserService service2 = context.getBean(UserService.class);

// 方式3：根据名称和类型获取（解决同类型多个Bean的问题）
UserService service3 = context.getBean("userService", UserService.class);

// 方式4：获取所有同类型Bean
Map<String, UserService> services = context.getBeansOfType(UserService.class);
```

### 2.10 BeanPostProcessor

BeanPostProcessor是Spring的后置处理器，用于在Bean初始化前后进行增强处理。

```java
// 自定义BeanPostProcessor
public class MyBeanPostProcessor implements BeanPostProcessor {
    
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) 
            throws BeansException {
        System.out.println("初始化前处理: " + beanName);
        return bean;
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) 
            throws BeansException {
        System.out.println("初始化后处理: " + beanName);
        return bean;
    }
}
```

```xml
<!-- 注册后置处理器 -->
<bean class="com.example.MyBeanPostProcessor"/>
```

### 2.11 FactoryBean与BeanFactory

**BeanFactory**：Spring容器的根接口，管理Bean的创建和获取。

**FactoryBean**：创建特定类型Bean的工厂接口。

```java
// 实现FactoryBean接口
public class UserServiceFactoryBean implements FactoryBean<UserService> {
    
    @Override
    public UserService getObject() throws Exception {
        return new UserService();
    }
    
    @Override
    public Class<?> getObjectType() {
        return UserService.class;
    }
    
    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

```xml
<!-- 使用FactoryBean -->
<bean id="userService" class="com.example.UserServiceFactoryBean"/>
```

---

## 3. DI依赖注入

### 3.1 什么是依赖注入

DI（Dependency Injection）依赖注入，是IoC的具体实现方式。容器负责将依赖对象注入到目标对象中。

**核心优势**：解耦，便于测试和维护。

### 3.2 Setter注入

通过setter方法注入依赖：

```java
public class UserService {
    private UserDao userDao;
    
    // Setter方法
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
}
```

```xml
<!-- Setter注入配置 -->
<bean id="userDao" class="com.example.UserDao"/>
<bean id="userService" class="com.example.UserService">
    <property name="userDao" ref="userDao"/>
</bean>
```

### 3.3 构造方法注入

通过构造方法注入依赖（Spring4.3+推荐）：

```java
public class UserService {
    private final UserDao userDao;
    
    // 构造方法注入（推荐使用final）
    public UserService(UserDao userDao) {
        this.userDao = userDao;
    }
}
```

```xml
<!-- 构造方法注入配置 -->
<bean id="userDao" class="com.example.UserDao"/>
<bean id="userService" class="com.example.UserService">
    <constructor-arg ref="userDao"/>
</bean>
```

### 3.4 自动注入

Spring支持自动注入，无需显式配置依赖关系：

```xml
<!-- byName：根据属性名匹配Bean名称 -->
<bean id="userService" class="com.example.UserService" autowire="byName"/>

<!-- byType：根据属性类型匹配Bean -->
<bean id="userService" class="com.example.UserService" autowire="byType"/>

<!-- constructor：构造方法自动注入 -->
<bean id="userService" class="com.example.UserService" autowire="constructor"/>
```

### 3.5 注入Bean类型

注入另一个Bean作为依赖：

```java
public class OrderService {
    private UserService userService;
    
    public void setUserService(UserService userService) {
        this.userService = userService;
    }
}
```

```xml
<bean id="userService" class="com.example.UserService"/>
<bean id="orderService" class="com.example.OrderService">
    <property name="userService" ref="userService"/>
</bean>
```

### 3.6 注入简单数据类型

注入基本类型、String等简单数据：

```java
public class UserService {
    private String serviceName;
    private int maxUsers;
    
    public void setServiceName(String serviceName) {
        this.serviceName = serviceName;
    }
    public void setMaxUsers(int maxUsers) {
        this.maxUsers = maxUsers;
    }
}
```

```xml
<bean id="userService" class="com.example.UserService">
    <property name="serviceName" value="UserManagementService"/>
    <property name="maxUsers" value="1000"/>
</bean>
```

### 3.7 注入集合类型

注入List、Set、Map、Properties等集合：

```java
public class UserService {
    private List<String> roles;
    private Set<String> permissions;
    private Map<String, String> configs;
    private Properties props;
    
    // Setter方法省略...
}
```

```xml
<bean id="userService" class="com.example.UserService">
    <property name="roles">
        <list>
            <value>admin</value>
            <value>user</value>
            <value>guest</value>
        </list>
    </property>
    <property name="permissions">
        <set>
            <value>read</value>
            <value>write</value>
        </set>
    </property>
    <property name="configs">
        <map>
            <entry key="timeout" value="30"/>
            <entry key="maxSize" value="100"/>
        </map>
    </property>
    <property name="props">
        <props>
            <prop key="driver">com.mysql.cj.jdbc.Driver</prop>
            <prop key="url">jdbc:mysql://localhost:3306/test</prop>
        </props>
    </property>
</bean>
```

---

## 4. 注解实现IoC

### 4.1 @Component

@Component是通用的组件注解，标记类为Spring管理的Bean。

```java
// 标注为Spring组件
@Component("userService")
public class UserService {
    public void addUser() {
        System.out.println("添加用户");
    }
}
```

```xml
<!-- 扫描组件所在包 -->
<context:component-scan base-package="com.example"/>
```

### 4.2 @Repository、@Service、@Controller

这三个注解是@Component的衍生注解，用于不同层：

| 注解 | 用途 |
|-----|------|
| @Repository | 数据访问层（DAO） |
| @Service | 业务逻辑层 |
| @Controller | 控制层（MVC） |

```java
@Repository
public class UserDao {
    // 数据访问逻辑
}

@Service
public class UserService {
    // 业务逻辑
}

@Controller
public class UserController {
    // 控制层逻辑
}
```

### 4.3 @Scope

@Scope注解指定Bean的作用域：

```java
@Component
@Scope("singleton")      // 默认：单例
// @Scope("prototype")   // 原型：每次获取新实例
// @Scope("request")     // 请求级别（Web环境）
// @Scope("session")     // 会话级别（Web环境）
public class UserService {
}
```

### 4.4 @Autowired

@Autowired实现自动注入，默认按类型匹配：

```java
@Service
public class UserService {
    
    // 字段注入
    @Autowired
    private UserDao userDao;
    
    // Setter注入
    @Autowired
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
    
    // 构造方法注入（推荐）
    @Autowired
    public UserService(UserDao userDao) {
        this.userDao = userDao;
    }
}
```

### 4.5 @Qualifier

当同一类型有多个Bean时，使用@Qualifier指定Bean名称：

```java
@Service
public class UserService {
    
    @Autowired
    @Qualifier("userDaoImpl")  // 指定Bean名称
    private UserDao userDao;
}
```

### 4.6 @Value

@Value用于注入简单数据类型和SpEL表达式：

```java
@Service
public class UserService {
    
    // 注入普通值
    @Value("UserManagementService")
    private String serviceName;
    
    // 注入配置文件中的值
    @Value("${app.maxUsers}")
    private int maxUsers;
    
    // SpEL表达式
    @Value("#{T(java.lang.Math).PI}")
    private double pi;
}
```

### 4.7 @Bean

@Bean用于在配置类中声明Bean：

```java
@Configuration
public class AppConfig {
    
    // 声明Bean
    @Bean("userDao")
    public UserDao createUserDao() {
        return new UserDao();
    }
    
    @Bean
    public UserService userService(UserDao userDao) {
        UserService service = new UserService();
        service.setUserDao(userDao);
        return service;
    }
}
```

### 4.8 @Import

@Import用于导入其他配置类或Bean：

```java
@Configuration
@Import({DataSourceConfig.class, MyBatisConfig.class})
public class AppConfig {
    // 当前配置类的Bean定义
}
```

### 4.9 @Conditional

@Conditional根据条件决定是否创建Bean：

```java
// 自定义条件
public class MyCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, 
                          AnnotatedTypeMetadata metadata) {
        // 根据环境变量判断
        String env = context.getEnvironment().getProperty("env");
        return "prod".equals(env);
    }
}

@Configuration
public class AppConfig {
    @Bean
    @Conditional(MyCondition.class)
    public UserService userService() {
        return new UserService();
    }
}
```

### 4.10 @Primary、@Lazy、@DependsOn

```java
@Service
@Primary  // 当同类型有多个Bean时，优先使用此Bean
public class PrimaryUserService implements UserService {
}

@Service
@Lazy  // 延迟初始化，首次使用时才创建
public class LazyUserService {
}

@Service
@DependsOn("userDao")  // 依赖userDao，确保userDao先初始化
public class UserService {
}
```

---

## 5. SpEL表达式

### 5.1 SpEL基础语法

SpEL（Spring Expression Language）是Spring的表达式语言，支持运行时查询和操作对象。

```java
// SpEL表达式解析
ExpressionParser parser = new SpelExpressionParser();

// 字面量
Expression exp1 = parser.parseExpression("'Hello World'");
String result1 = (String) exp1.getValue();  // Hello World

// 数学运算
Expression exp2 = parser.parseExpression("10 + 20");
int result2 = (int) exp2.getValue();  // 30

// 逻辑运算
Expression exp3 = parser.parseExpression("true and false");
boolean result3 = (boolean) exp3.getValue();  // false

// 访问对象属性
User user = new User("张三", 25);
EvaluationContext context = new StandardEvaluationContext(user);
Expression exp4 = parser.parseExpression("name");
String result4 = (String) exp4.getValue(context);  // 张三
```

### 5.2 SpEL在注解中的使用

```java
@Service
public class UserService {
    
    // 注入系统属性
    @Value("#{systemProperties['user.name']}")
    private String userName;
    
    // 注入Bean属性
    @Value("#{userDao.count}")
    private int userCount;
    
    // 方法调用
    @Value("#{userDao.getUser(1).name}")
    private String firstName;
}
```

### 5.3 SpEL在XML中的使用

```xml
<bean id="userService" class="com.example.UserService">
    <!-- 注入系统属性 -->
    <property name="userName" value="#{systemProperties['user.name']}"/>
    
    <!-- 注入Bean属性 -->
    <property name="userCount" value="#{userDao.count}"/>
    
    <!-- 方法调用 -->
    <property name="firstName" value="#{userDao.getUser(1).name}"/>
</bean>
```

---

## 6. Spring整合MyBatis

### 6.1 搭建环境

**Maven依赖**：

```xml
<!-- Spring核心依赖 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>6.1.0</version>
</dependency>

<!-- Spring JDBC -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>6.1.0</version>
</dependency>

<!-- MyBatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.15</version>
</dependency>

<!-- MyBatis-Spring整合 -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>3.0.3</version>
</dependency>

<!-- MySQL驱动 -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.3.0</version>
</dependency>
```

### 6.2 编写配置文件

**applicationContext.xml**：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <!-- 配置数据源 -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/test"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
    </bean>
    
    <!-- 配置SqlSessionFactory -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!-- 配置MyBatis配置文件 -->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <!-- 配置Mapper扫描路径 -->
        <property name="mapperLocations" value="classpath:mapper/*.xml"/>
    </bean>
    
    <!-- 配置Mapper扫描器 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.example.mapper"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean>
    
    <!-- 配置Service -->
    <bean id="userService" class="com.example.service.UserService">
        <property name="userMapper" ref="userMapper"/>
    </bean>
</beans>
```

**mybatis-config.xml**：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!-- 开启驼峰命名转换 -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
</configuration>
```

### 6.3 编写Java代码

**User实体类**：

```java
public class User {
    private Long id;
    private String userName;
    private Integer age;
    
    // Getter/Setter省略...
}
```

**UserMapper接口**：

```java
public interface UserMapper {
    // 根据ID查询用户
    User selectById(Long id);
    
    // 新增用户
    int insert(User user);
    
    // 更新用户
    int update(User user);
    
    // 删除用户
    int deleteById(Long id);
}
```

**UserMapper.xml**：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.UserMapper">
    
    <select id="selectById" resultType="com.example.entity.User">
        SELECT id, user_name, age FROM user WHERE id = #{id}
    </select>
    
    <insert id="insert" parameterType="com.example.entity.User">
        INSERT INTO user(user_name, age) VALUES(#{userName}, #{age})
    </insert>
    
    <update id="update" parameterType="com.example.entity.User">
        UPDATE user SET user_name = #{userName}, age = #{age} WHERE id = #{id}
    </update>
    
    <delete id="deleteById" parameterType="java.lang.Long">
        DELETE FROM user WHERE id = #{id}
    </delete>
</mapper>
```

**UserService**：

```java
public class UserService {
    private UserMapper userMapper;
    
    public void setUserMapper(UserMapper userMapper) {
        this.userMapper = userMapper;
    }
    
    public User getUserById(Long id) {
        return userMapper.selectById(id);
    }
    
    public void addUser(User user) {
        userMapper.insert(user);
    }
}
```

### 6.4 单元测试

```java
public class UserServiceTest {
    
    @Test
    public void testGetUserById() {
        ApplicationContext context = 
            new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService service = context.getBean(UserService.class);
        
        User user = service.getUserById(1L);
        System.out.println("用户姓名: " + user.getUserName());
    }
}
```

### 6.5 自动创建代理对象

MyBatis-Spring会自动为Mapper接口创建代理对象，无需手动实现：

```java
// MyBatis自动创建代理对象，直接注入使用
@Service
public class UserService {
    
    @Autowired
    private UserMapper userMapper;  // 代理对象自动注入
    
    public User getUser(Long id) {
        return userMapper.selectById(id);
    }
}
```

---

## 7. AOP面向切面

### 7.1 AOP入门

AOP（Aspect-Oriented Programming）面向切面编程，将横切关注点（如日志、事务）从业务逻辑中分离。

**核心概念**：
- **切面（Aspect）**：横切关注点的模块化
- **通知（Advice）**：切面的具体行为
- **切点（Pointcut）**：通知应用的位置
- **连接点（JoinPoint）**：程序执行过程中的点

### 7.2 通知类型

| 通知类型 | 说明 |
|---------|------|
| @Before | 方法执行前 |
| @After | 方法执行后（无论是否异常） |
| @AfterReturning | 方法正常返回后 |
| @AfterThrowing | 方法抛出异常后 |
| @Around | 环绕通知（最强大） |

```java
@Aspect
@Component
public class LogAspect {
    
    @Before("execution(* com.example.service.*.*(..))")
    public void beforeMethod(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("方法执行前: " + methodName);
    }
    
    @After("execution(* com.example.service.*.*(..))")
    public void afterMethod(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("方法执行后: " + methodName);
    }
    
    @AfterReturning(pointcut = "execution(* com.example.service.*.*(..))",
                   returning = "result")
    public void afterReturning(JoinPoint joinPoint, Object result) {
        System.out.println("返回值: " + result);
    }
    
    @AfterThrowing(pointcut = "execution(* com.example.service.*.*(..))",
                  throwing = "ex")
    public void afterThrowing(JoinPoint joinPoint, Exception ex) {
        System.out.println("异常信息: " + ex.getMessage());
    }
    
    @Around("execution(* com.example.service.*.*(..))")
    public Object aroundMethod(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("环绕前");
        Object result = pjp.proceed();  // 执行目标方法
        System.out.println("环绕后");
        return result;
    }
}
```

### 7.3 切点表达式

切点表达式用于指定通知应用的位置：

```java
@Aspect
@Component
public class LogAspect {
    
    // 匹配com.example.service包下所有类的所有方法
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceMethods() {}
    
    // 匹配特定方法
    @Pointcut("execution(* com.example.service.UserService.addUser(..))")
    public void addUserMethod() {}
    
    // 匹配带特定注解的方法
    @Pointcut("@annotation(com.example.annotation.Log)")
    public void annotatedMethods() {}
    
    // 匹配实现特定接口的类的方法
    @Pointcut("within(com.example.service.*)")
    public void withinService() {}
}
```

### 7.4 多切面配置

当多个切面同时作用于一个方法时，执行顺序由@Order注解控制：

```java
@Aspect
@Component
@Order(1)  // 数字越小优先级越高
public class LogAspect {
    @Before("execution(* com.example.service.*.*(..))")
    public void before() {
        System.out.println("日志切面");
    }
}

@Aspect
@Component
@Order(2)
public class TransactionAspect {
    @Before("execution(* com.example.service.*.*(..))")
    public void before() {
        System.out.println("事务切面");
    }
}
```

### 7.5 注解配置AOP

```java
@Configuration
@EnableAspectJAutoProxy  // 开启AOP支持
@ComponentScan("com.example")
public class AppConfig {
}
```

### 7.6 原生Spring实现AOP

原生Spring实现AOP使用ProxyFactoryBean或ProxyFactory进行编程式AOP配置：

**方式1：使用ProxyFactoryBean（XML配置）**

```java
// 实现MethodBeforeAdvice接口
public class LogBeforeAdvice implements MethodBeforeAdvice {
    @Override
    public void before(Method method, Object[] args, Object target) 
            throws Throwable {
        System.out.println("方法执行前: " + method.getName());
    }
}

// 实现AfterReturningAdvice接口
public class LogAfterAdvice implements AfterReturningAdvice {
    @Override
    public void afterReturning(Object returnValue, Method method, 
                               Object[] args, Object target) throws Throwable {
        System.out.println("方法执行后: " + method.getName());
    }
}
```

```xml
<!-- 目标Bean -->
<bean id="userService" class="com.example.service.UserService"/>

<!-- 通知Bean -->
<bean id="logBeforeAdvice" class="com.example.aspect.LogBeforeAdvice"/>
<bean id="logAfterAdvice" class="com.example.aspect.LogAfterAdvice"/>

<!-- ProxyFactoryBean配置代理 -->
<bean id="userServiceProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target" ref="userService"/>
    <property name="interceptorNames">
        <list>
            <value>logBeforeAdvice</value>
            <value>logAfterAdvice</value>
        </list>
    </property>
</bean>
```

**方式2：使用ProxyFactory（编程式）**

```java
@Configuration
public class AppConfig {
    
    @Bean
    public UserService userService() {
        return new UserService();
    }
    
    @Bean
    public UserService userServiceProxy(UserService userService) {
        ProxyFactory factory = new ProxyFactory(userService);
        
        // 添加前置通知
        factory.addAdvice((MethodBeforeAdvice) (method, args, target) -> 
            System.out.println("方法执行前: " + method.getName()));
        
        // 添加后置通知
        factory.addAdvice((AfterReturningAdvice) (returnValue, method, args, target) -> 
            System.out.println("方法执行后: " + method.getName()));
        
        return (UserService) factory.getProxy();
    }
}
```

### 7.7 Schema-Based实现AOP

通过XML配置AOP：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd">
    
    <!-- 目标Bean -->
    <bean id="userService" class="com.example.service.UserService"/>
    
    <!-- 切面Bean -->
    <bean id="logAspect" class="com.example.aspect.LogAspect"/>
    
    <!-- AOP配置 -->
    <aop:config>
        <aop:aspect ref="logAspect">
            <aop:pointcut id="serviceMethods" 
                         expression="execution(* com.example.service.*.*(..))"/>
            <aop:before pointcut-ref="serviceMethods" method="beforeMethod"/>
            <aop:after pointcut-ref="serviceMethods" method="afterMethod"/>
        </aop:aspect>
    </aop:config>
</beans>
```

---

## 8. Spring事务管理

### 8.1 事务管理方案

Spring支持两种事务管理方式：

| 方式 | 说明 |
|-----|------|
| 编程式事务 | 通过代码手动控制事务 |
| 声明式事务 | 通过注解或XML配置自动管理事务 |

### 8.2 Spring事务管理器

Spring提供多种事务管理器实现：

```java
@Configuration
public class TransactionConfig {
    
    // JDBC事务管理器
    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
    
    // JPA事务管理器
    // @Bean
    // public PlatformTransactionManager jpaTransactionManager(EntityManagerFactory emf) {
    //     return new JpaTransactionManager(emf);
    // }
}
```

### 8.3 事务控制的API

```java
@Service
public class UserService {
    
    @Autowired
    private PlatformTransactionManager transactionManager;
    
    public void transfer(Long fromId, Long toId, Double amount) {
        // 定义事务属性
        TransactionDefinition definition = 
            new DefaultTransactionDefinition(TransactionDefinition.PROPAGATION_REQUIRED);
        
        // 开启事务
        TransactionStatus status = transactionManager.getTransaction(definition);
        
        try {
            // 业务逻辑
            userDao.decreaseBalance(fromId, amount);
            userDao.increaseBalance(toId, amount);
            
            // 提交事务
            transactionManager.commit(status);
        } catch (Exception e) {
            // 回滚事务
            transactionManager.rollback(status);
            throw e;
        }
    }
}
```

### 8.4 事务的相关配置

```java
@Transactional(
    propagation = Propagation.REQUIRED,  // 传播行为
    isolation = Isolation.DEFAULT,        // 隔离级别
    readOnly = false,                     // 是否只读
    timeout = -1,                         // 超时时间（秒）
    rollbackFor = Exception.class,        // 指定回滚的异常类型
    noRollbackFor = RuntimeException.class // 指定不回滚的异常类型
)
public void transfer(Long fromId, Long toId, Double amount) {
    // 业务逻辑
}
```

### 8.5 事务的传播行为与隔离级别

**传播行为**：

| 传播行为 | 说明 |
|---------|------|
| REQUIRED | 如果当前有事务，则加入；否则新建 |
| REQUIRES_NEW | 总是新建事务 |
| SUPPORTS | 如果当前有事务，则加入；否则以非事务方式执行 |
| NOT_SUPPORTED | 以非事务方式执行，如果当前有事务则挂起 |
| NEVER | 以非事务方式执行，如果当前有事务则抛出异常 |

**隔离级别**：

| 隔离级别 | 说明 |
|---------|------|
| DEFAULT | 使用数据库默认隔离级别 |
| READ_UNCOMMITTED | 允许读取未提交的数据 |
| READ_COMMITTED | 允许读取已提交的数据 |
| REPEATABLE_READ | 保证同一事务中多次读取结果一致 |
| SERIALIZABLE | 最高隔离级别，串行执行 |

### 8.6 注解配置事务

```java
@Configuration
@EnableTransactionManagement  // 开启事务管理支持
public class AppConfig {
    
    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}

@Service
public class UserService {
    
    @Autowired
    private UserDao userDao;
    
    // 声明事务
    @Transactional(propagation = Propagation.REQUIRED,
                  isolation = Isolation.READ_COMMITTED)
    public void transfer(Long fromId, Long toId, Double amount) {
        userDao.decreaseBalance(fromId, amount);
        userDao.increaseBalance(toId, amount);
    }
}
```

---

## 9. Spring事件机制

### 9.1 ApplicationEvent事件

Spring事件机制基于观察者模式，实现松耦合的组件通信。

```java
// 自定义事件
public class UserRegisterEvent extends ApplicationEvent {
    private User user;
    
    public UserRegisterEvent(Object source, User user) {
        super(source);
        this.user = user;
    }
    
    public User getUser() {
        return user;
    }
}
```

### 9.2 ApplicationListener监听器

```java
// 自定义监听器
@Component
public class UserRegisterListener implements ApplicationListener<UserRegisterEvent> {
    
    @Override
    public void onApplicationEvent(UserRegisterEvent event) {
        User user = event.getUser();
        System.out.println("用户注册成功: " + user.getUserName());
        // 发送欢迎邮件等逻辑
    }
}
```

### 9.3 @EventListener注解

使用注解简化监听器配置：

```java
@Component
public class UserRegisterListener {
    
    // 监听UserRegisterEvent事件
    @EventListener
    public void handleUserRegister(UserRegisterEvent event) {
        User user = event.getUser();
        System.out.println("用户注册成功: " + user.getUserName());
    }
    
    // 监听多个事件
    @EventListener({UserRegisterEvent.class, UserDeleteEvent.class})
    public void handleUserEvent(ApplicationEvent event) {
        System.out.println("用户事件触发");
    }
    
    // 条件监听
    @EventListener(condition = "#event.user.age >= 18")
    public void handleAdultRegister(UserRegisterEvent event) {
        System.out.println("成年用户注册");
    }
}
```

**发布事件**：

```java
@Service
public class UserService {
    
    @Autowired
    private ApplicationEventPublisher eventPublisher;
    
    public void register(User user) {
        // 注册逻辑
        userDao.insert(user);
        
        // 发布事件
        eventPublisher.publishEvent(new UserRegisterEvent(this, user));
    }
}
```

---

## 10. Environment与Profile

### 10.1 Environment环境抽象

Environment提供统一的环境配置访问接口：

```java
@Configuration
public class AppConfig {
    
    @Autowired
    private Environment env;
    
    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource ds = new DriverManagerDataSource();
        ds.setDriverClassName(env.getProperty("jdbc.driver"));
        ds.setUrl(env.getProperty("jdbc.url"));
        ds.setUsername(env.getProperty("jdbc.username"));
        ds.setPassword(env.getProperty("jdbc.password"));
        return ds;
    }
}
```

### 10.2 Profile多环境配置

Profile支持多环境切换：

```java
@Configuration
public class DataSourceConfig {
    
    // 开发环境
    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        return new DriverManagerDataSource("jdbc:h2:mem:devDB");
    }
    
    // 测试环境
    @Bean
    @Profile("test")
    public DataSource testDataSource() {
        return new DriverManagerDataSource("jdbc:h2:mem:testDB");
    }
    
    // 生产环境
    @Bean
    @Profile("prod")
    public DataSource prodDataSource() {
        DriverManagerDataSource ds = new DriverManagerDataSource();
        ds.setUrl("jdbc:mysql://localhost:3306/prod");
        return ds;
    }
}
```

**激活Profile**：

```java
// 方式1：通过代码激活
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
context.getEnvironment().setActiveProfiles("dev");
context.register(AppConfig.class);
context.refresh();

// 方式2：通过JVM参数
// -Dspring.profiles.active=prod
```

---

## 11. Spring Task定时任务

### 11.1 入门案例

```java
@Configuration
@EnableScheduling  // 开启定时任务支持
public class TaskConfig {
    
    @Scheduled(fixedRate = 5000)  // 每5秒执行一次
    public void task1() {
        System.out.println("定时任务1执行: " + new Date());
    }
    
    @Scheduled(fixedDelay = 3000)  // 上次执行完成后3秒执行
    public void task2() {
        System.out.println("定时任务2执行: " + new Date());
    }
}
```

### 11.2 Cron表达式

Cron表达式格式：`秒 分 时 日 月 周`

| 字段 | 范围 |
|-----|------|
| 秒 | 0-59 |
| 分 | 0-59 |
| 时 | 0-23 |
| 日 | 1-31 |
| 月 | 1-12 或 JAN-DEC |
| 周 | 1-7 或 SUN-SAT |

**常用符号**：
- `*`：匹配任意值
- `?`：不指定值（用于日和周）
- `-`：范围
- `,`：枚举
- `/`：步长

**示例**：
- `0 0 2 * * ?`：每天凌晨2点执行
- `0 0/30 9-17 * * ?`：工作日9:00-17:00每30分钟执行
- `0 0 12 * * MON-FRI`：工作日中午12点执行

### 11.3 Cron实战案例

```java
@Service
public class ScheduledTasks {
    
    // 每天凌晨2点执行
    @Scheduled(cron = "0 0 2 * * ?")
    public void dailyTask() {
        System.out.println("每日定时任务执行");
    }
    
    // 每小时执行一次
    @Scheduled(cron = "0 0 * * * ?")
    public void hourlyTask() {
        System.out.println("每小时定时任务执行");
    }
    
    // 工作日9:00-18:00每30分钟执行
    @Scheduled(cron = "0 0/30 9-18 * * MON-FRI")
    public void workdayTask() {
        System.out.println("工作日定时任务执行");
    }
    
    // 每月1号凌晨执行
    @Scheduled(cron = "0 0 0 1 * ?")
    public void monthlyTask() {
        System.out.println("每月定时任务执行");
    }
}
```

### 11.4 注解配置定时任务

```java
@Configuration
@EnableScheduling
@ComponentScan("com.example")
public class AppConfig {
    
    // 自定义任务调度器（可选）
    @Bean
    public TaskScheduler taskScheduler() {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(5);  // 线程池大小
        scheduler.setThreadNamePrefix("task-");
        scheduler.setAwaitTerminationSeconds(60);
        return scheduler;
    }
}
```

### 11.5 多线程任务

默认情况下，Spring Task使用单线程执行所有定时任务。启用多线程支持：

```java
@Configuration
@EnableScheduling
public class TaskConfig {
    
    @Bean
    public TaskScheduler taskScheduler() {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(10);
        scheduler.setThreadNamePrefix("scheduled-task-");
        scheduler.setWaitForTasksToCompleteOnShutdown(true);
        scheduler.setAwaitTerminationSeconds(60);
        return scheduler;
    }
    
    // 多个任务并行执行
    @Scheduled(cron = "0/5 * * * * ?")
    public void taskA() {
        System.out.println("Task A - " + Thread.currentThread().getName());
    }
    
    @Scheduled(cron = "0/5 * * * * ?")
    public void taskB() {
        System.out.println("Task B - " + Thread.currentThread().getName());
    }
    
    @Scheduled(cron = "0/5 * * * * ?")
    public void taskC() {
        System.out.println("Task C - " + Thread.currentThread().getName());
    }
}
```

---

## 附录

### A. Spring6核心注解速查表

| 注解 | 用途 |
|-----|------|
| @Component | 通用组件 |
| @Repository | 数据访问层 |
| @Service | 业务逻辑层 |
| @Controller | 控制层 |
| @Configuration | 配置类 |
| @Bean | 声明Bean |
| @Autowired | 自动注入 |
| @Qualifier | 指定Bean名称 |
| @Value | 注入值 |
| @Scope | 指定作用域 |
| @Primary | 优先Bean |
| @Lazy | 延迟加载 |
| @Conditional | 条件创建 |
| @Transactional | 事务管理 |
| @Aspect | 切面 |
| @Pointcut | 切点 |
| @Before/@After/@Around | 通知类型 |
| @EventListener | 事件监听 |
| @Scheduled | 定时任务 |
| @EnableScheduling | 开启定时任务 |
| @EnableTransactionManagement | 开启事务管理 |
| @EnableAspectJAutoProxy | 开启AOP |

### B. Spring容器类型对比

| 容器类型 | 加载方式 | 适用场景 |
|---------|---------|---------|
| BeanFactory | 延迟加载 | 资源敏感环境 |
| ApplicationContext | 立即加载 | 大多数场景 |
| WebApplicationContext | 立即加载 | Web应用 |

### C. Bean作用域对比

| 作用域 | 说明 |
|-------|------|
| singleton | 默认，容器中唯一实例 |
| prototype | 每次获取新实例 |
| request | 每个请求一个实例（Web） |
| session | 每个会话一个实例（Web） |
| globalSession | 全局会话（Portlet） |