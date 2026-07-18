
# SpringMVC核心技术详解

[1. SpringMVC核心架构与执行流程](#1-springmvc核心架构与执行流程)
  - [1.1 SpringMVC简介](#11-springmvc简介)
  - [1.2 MVC设计模式](#12-mvc设计模式)
  - [1.3 SpringMVC执行流程详解](#13-springmvc执行流程详解)
  - [1.4 DispatcherServlet核心流程](#14-dispatcherservlet核心流程)
  - [1.5 HandlerMapping与HandlerAdapter机制](#15-handlermapping与handleradapter机制)

[2. 请求参数绑定](#2-请求参数绑定)
  - [2.1 简单数据类型绑定](#21-简单数据类型绑定)
  - [2.2 对象类型绑定](#22-对象类型绑定)
  - [2.3 关联对象绑定](#23-关联对象绑定)
  - [2.4 集合类型绑定](#24-集合类型绑定)
  - [2.5 参数类型转换器](#25-参数类型转换器)
  - [2.6 Servlet原生对象](#26-servlet原生对象)
  - [2.7 编码过滤器](#27-编码过滤器)
  - [2.8 JSR-303数据校验](#28-jsr-303数据校验)

[3. 响应处理与视图](#3-响应处理与视图)
  - [3.1 视图解析器](#31-视图解析器)
  - [3.2 返回值类型](#32-返回值类型)
  - [3.3 域对象数据传递](#33-域对象数据传递)
  - [3.4 请求转发与重定向](#34-请求转发与重定向)
  - [3.5 HttpMessageConverter原理](#35-httpmessageconverter原理)

[4. 核心注解详解](#4-核心注解详解)
  - [4.1 @Controller与@RequestMapping](#41-controller与-requestmapping)
  - [4.2 @RequestParam、@RequestHeader、@CookieValue](#42-requestparam-requestheader-cookievalue)
  - [4.3 @SessionAttributes与@ModelAttribute](#43-sessionattributes与-modelattribute)
  - [4.4 RESTful风格与@PathVariable](#44-restful风格与-pathvariable)
  - [4.5 HiddenHttpMethodFilter](#45-hiddenhttpmethodfilter)
  - [4.6 @ResponseBody与@RestController](#46-responsebody与-restcontroller)
  - [4.7 @RequestBody](#47-requestbody)
  - [4.8 静态资源映射](#48-静态资源映射)
  - [4.9 Postman使用说明](#49-postman使用说明)

[5. 文件上传与下载](#5-文件上传与下载)
  - [5.1 SpringMVC文件上传](#51-springmvc文件上传)
  - [5.2 多文件上传](#52-多文件上传)
  - [5.3 异步上传](#53-异步上传)
  - [5.4 跨服务器上传](#54-跨服务器上传)
  - [5.5 文件下载](#55-文件下载)

[6. 异常处理](#6-异常处理)
  - [6.1 单个控制器异常处理](#61-单个控制器异常处理)
  - [6.2 全局异常处理](#62-全局异常处理)
  - [6.3 HandlerExceptionResolver原理](#63-handlerexceptionresolver原理)

[7. 拦截器](#7-拦截器)
  - [7.1 拦截器简介](#71-拦截器简介)
  - [7.2 拦截器的使用](#72-拦截器的使用)
  - [7.3 拦截器链与执行顺序](#73-拦截器链与执行顺序)
  - [7.4 过滤敏感词实战](#74-过滤敏感词实战)
  - [7.5 拦截器与Filter区别](#75-拦截器与filter区别)

[8. 跨域请求](#8-跨域请求)
  - [8.1 同源策略](#81-同源策略)
  - [8.2 SpringMVC跨域处理](#82-springmvc跨域处理)

---

## 1. SpringMVC核心架构与执行流程

### 1.1 SpringMVC简介

SpringMVC是Spring框架的一个模块，用于构建Web应用程序，实现了MVC（Model-View-Controller）设计模式。它通过DispatcherServlet作为核心控制器，协调各个组件完成请求处理。

**核心特点：**
- 松耦合的组件设计
- 灵活的请求映射机制
- 强大的数据绑定能力
- 可扩展的视图解析机制

### 1.2 MVC设计模式

MVC将应用程序分为三个核心部分：

| 组件 | 职责 | 说明 |
|------|------|------|
| **Model** | 业务数据与状态 | 包含实体类、DTO、业务逻辑 |
| **View** | 数据展示 | JSP、Thymeleaf、JSON等 |
| **Controller** | 请求处理与调度 | 接收请求、调用业务、返回响应 |

**MVC工作流程：**
```
浏览器请求 → Controller(处理请求) → Service(业务逻辑) 
    → Repository(数据访问) → 返回Model → View(渲染展示)
```

### 1.3 SpringMVC执行流程详解

SpringMVC的完整执行流程涉及多个核心组件的协作：

```
1. 浏览器发送请求 → DispatcherServlet
2. DispatcherServlet → HandlerMapping(查找处理器)
3. HandlerMapping → 返回HandlerExecutionChain
4. DispatcherServlet → HandlerAdapter(适配执行)
5. HandlerAdapter → 调用Controller方法
6. Controller → 返回ModelAndView
7. DispatcherServlet → ViewResolver(解析视图)
8. ViewResolver → 返回View对象
9. DispatcherServlet → 渲染View
10. 返回HTTP响应 → 浏览器
```

**核心组件说明：**
- **DispatcherServlet**：前端控制器，统一调度
- **HandlerMapping**：根据URL查找处理器
- **HandlerAdapter**：适配器，执行处理器方法
- **ViewResolver**：视图解析器，解析视图路径
- **ModelAndView**：封装模型数据与视图信息

### 1.4 DispatcherServlet核心流程

DispatcherServlet是SpringMVC的入口，其核心方法为`doDispatch()`：

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    
    // 1. 获取HandlerExecutionChain
    mappedHandler = getHandler(processedRequest);
    if (mappedHandler == null) {
        noHandlerFound(processedRequest, response);
        return;
    }
    
    // 2. 获取HandlerAdapter
    HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
    
    // 3. 执行前置拦截器
    if (!mappedHandler.applyPreHandle(processedRequest, response)) {
        return;
    }
    
    // 4. 执行处理器方法，返回ModelAndView
    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
    
    // 5. 执行后置拦截器
    mappedHandler.applyPostHandle(processedRequest, response, mv);
    
    // 6. 渲染视图
    processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
}
```

**流程解析：**
1. 通过`getHandler()`查找匹配的处理器
2. 通过`getHandlerAdapter()`获取适配器
3. 执行拦截器链的前置方法
4. 调用控制器方法获取ModelAndView
5. 执行拦截器链的后置方法
6. 处理视图渲染或异常

### 1.5 HandlerMapping与HandlerAdapter机制

**HandlerMapping（处理器映射）：**

负责将请求URL映射到对应的Controller方法，常见实现：

```java
// BeanNameUrlHandlerMapping - 根据Bean名称映射
// SimpleUrlHandlerMapping - 配置URL映射
// RequestMappingHandlerMapping - 根据@RequestMapping注解映射（最常用）

@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        // 直接映射URL到视图
        registry.addViewController("/hello").setViewName("hello");
    }
}
```

**HandlerAdapter（处理器适配器）：**

负责调用处理器方法，适配不同类型的处理器：

```java
// SimpleControllerHandlerAdapter - 适配Controller接口
// HttpRequestHandlerAdapter - 适配HttpRequestHandler接口
// RequestMappingHandlerAdapter - 适配@RequestMapping注解方法（最常用）

// HandlerAdapter核心方法
ModelAndView handle(HttpServletRequest request, 
                    HttpServletResponse response, 
                    Object handler) throws Exception;
```

**适配器模式优势：**
- 支持多种处理器类型，无需修改DispatcherServlet
- 解耦处理器定义与调用方式
- 便于扩展新的处理器类型

---

## 2. 请求参数绑定

### 2.1 简单数据类型绑定

直接绑定基本类型参数，支持自动类型转换：

```java
@Controller
@RequestMapping("/user")
public class UserController {
    
    // 请求URL: /user/login?username=admin&age=25
    @RequestMapping("/login")
    public String login(String username, Integer age) {
        // 自动绑定参数
        System.out.println("用户名: " + username);
        System.out.println("年龄: " + age);
        return "success";
    }
}
```

**参数命名规则：**
- 方法参数名需与请求参数名一致
- 支持基本类型及其包装类
- 支持String类型

### 2.2 对象类型绑定

自动将请求参数封装到Java对象中：

```java
// 用户实体类
public class User {
    private String username;
    private Integer age;
    private String email;
    // getter/setter省略
}

@Controller
@RequestMapping("/user")
public class UserController {
    
    // 请求URL: /user/register?username=admin&age=25&email=admin@test.com
    @RequestMapping("/register")
    public String register(User user) {
        // 自动封装到User对象
        System.out.println("用户: " + user.getUsername());
        return "success";
    }
}
```

**绑定规则：**
- 请求参数名需与对象属性名一致
- 支持级联属性绑定
- 自动进行类型转换

### 2.3 关联对象绑定

支持嵌套对象的参数绑定：

```java
// 地址类
public class Address {
    private String province;
    private String city;
    // getter/setter省略
}

// 用户类（包含地址）
public class User {
    private String username;
    private Address address;  // 关联对象
    // getter/setter省略
}

@Controller
@RequestMapping("/user")
public class UserController {
    
    // 请求URL: /user/add?username=admin&address.province=北京&address.city=朝阳区
    @RequestMapping("/add")
    public String addUser(User user) {
        System.out.println(user.getUsername());
        System.out.println(user.getAddress().getProvince());
        return "success";
    }
}
```

**绑定格式：**
- 使用`.`分隔嵌套属性：`address.province`
- 支持多层嵌套

### 2.4 集合类型绑定

支持List、Map等集合类型的参数绑定：

```java
@Controller
@RequestMapping("/user")
public class UserController {
    
    // 绑定List集合
    // 请求: /user/list?ids=1&ids=2&ids=3
    @RequestMapping("/list")
    public String getUserList(@RequestParam("ids") List<Integer> ids) {
        System.out.println("ID列表: " + ids);
        return "success";
    }
    
    // 绑定Map集合
    @RequestMapping("/map")
    public String getMap(@RequestParam Map<String, String> params) {
        System.out.println("参数Map: " + params);
        return "success";
    }
    
    // 对象集合（表单批量提交）
    @RequestMapping("/batch")
    public String batchAdd(@RequestBody List<User> users) {
        System.out.println("用户数量: " + users.size());
        return "success";
    }
}
```

**集合绑定要点：**
- List使用`@RequestParam`注解
- Map直接绑定所有请求参数
- 对象集合通常使用`@RequestBody`接收JSON

### 2.5 参数类型转换器

SpringMVC支持自定义类型转换器：

```java
// 自定义日期转换器
public class DateConverter implements Converter<String, Date> {
    private SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
    
    @Override
    public Date convert(String source) {
        try {
            return sdf.parse(source);
        } catch (ParseException e) {
            throw new IllegalArgumentException("日期格式错误");
        }
    }
}

// 注册转换器
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new DateConverter());
    }
}

// 使用示例
@Controller
@RequestMapping("/user")
public class UserController {
    @RequestMapping("/add")
    public String add(Date birthday) {
        System.out.println("生日: " + birthday);
        return "success";
    }
}
```

**内置转换器：**
- String → Integer
- String → Long
- String → Boolean
- String → Date（需配置格式）

### 2.6 Servlet原生对象

可以直接获取Servlet原生对象：

```java
@Controller
@RequestMapping("/servlet")
public class ServletController {
    
    @RequestMapping("/test")
    public String test(HttpServletRequest request, 
                       HttpServletResponse response,
                       HttpSession session,
                       ServletContext context) {
        
        // 获取请求参数
        String name = request.getParameter("name");
        
        // 设置Session属性
        session.setAttribute("user", "admin");
        
        // 获取应用上下文
        String appName = context.getInitParameter("appName");
        
        return "success";
    }
}
```

**支持的原生对象：**
- HttpServletRequest
- HttpServletResponse
- HttpSession
- ServletContext
- MultipartRequest（文件上传）

### 2.7 编码过滤器

处理中文乱码问题，SpringMVC提供`CharacterEncodingFilter`：

```java
@Configuration
public class WebConfig {
    
    @Bean
    public FilterRegistrationBean<CharacterEncodingFilter> encodingFilter() {
        FilterRegistrationBean<CharacterEncodingFilter> registration = 
                new FilterRegistrationBean<>();
        CharacterEncodingFilter filter = new CharacterEncodingFilter();
        
        filter.setEncoding("UTF-8");
        filter.setForceEncoding(true);  // 强制编码
        
        registration.setFilter(filter);
        registration.addUrlPatterns("/*");  // 拦截所有请求
        registration.setOrder(1);  // 优先级最高
        
        return registration;
    }
}
```

**乱码问题分析：**
| 请求方式 | 乱码原因 | 解决方案 |
|---------|---------|---------|
| POST | 请求体未指定编码 | CharacterEncodingFilter |
| GET | Tomcat默认ISO-8859-1 | server.xml配置URIEncoding="UTF-8" |

**Tomcat配置（server.xml）：**
```xml
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443"
           URIEncoding="UTF-8"/>
```

### 2.8 JSR-303数据校验

使用注解进行数据校验：

```java
// 引入依赖
// spring-boot-starter-validation

// 使用校验注解
public class User {
    @NotBlank(message = "用户名不能为空")
    private String username;
    
    @Min(value = 18, message = "年龄不能小于18")
    private Integer age;
    
    @Email(message = "邮箱格式不正确")
    private String email;
    // getter/setter省略
}

@Controller
@RequestMapping("/user")
public class UserController {
    
    @RequestMapping("/validate")
    public String validate(@Valid User user, BindingResult result) {
        // 检查校验结果
        if (result.hasErrors()) {
            // 获取错误信息
            List<FieldError> errors = result.getFieldErrors();
            for (FieldError error : errors) {
                System.out.println(error.getField() + ": " + error.getDefaultMessage());
            }
            return "error";
        }
        return "success";
    }
}
```

**常用校验注解：**
| 注解 | 说明 |
|------|------|
| @NotNull | 不能为null |
| @NotBlank | 字符串不能为空 |
| @Size | 长度范围 |
| @Min/@Max | 数值范围 |
| @Email | 邮箱格式 |
| @Pattern | 正则表达式 |

---

## 3. 响应处理与视图

### 3.1 视图解析器

SpringMVC支持多种视图解析器：

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");  // 视图前缀
        resolver.setSuffix(".jsp");              // 视图后缀
        resolver.setViewClass(JstlView.class);   // JSTL视图
        return resolver;
    }
}

@Controller
@RequestMapping("/view")
public class ViewController {
    
    // 返回视图名，自动拼接前缀后缀
    // 最终路径: /WEB-INF/views/hello.jsp
    @RequestMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

**常见视图解析器：**
- InternalResourceViewResolver（JSP）
- ThymeleafViewResolver（Thymeleaf）
- FreeMarkerViewResolver（FreeMarker）
- ResourceBundleViewResolver（国际化视图）

### 3.2 返回值类型

SpringMVC支持多种返回值类型：

```java
@Controller
@RequestMapping("/return")
public class ReturnController {
    
    // 返回String（视图名）
    @RequestMapping("/string")
    public String returnString(Model model) {
        model.addAttribute("message", "Hello");
        return "success";
    }
    
    // 返回ModelAndView
    @RequestMapping("/mv")
    public ModelAndView returnModelAndView() {
        ModelAndView mv = new ModelAndView("success");
        mv.addObject("message", "Hello MVC");
        return mv;
    }
    
    // 返回void（直接操作response）
    @RequestMapping("/void")
    public void returnVoid(HttpServletResponse response) throws Exception {
        response.getWriter().write("直接响应");
    }
    
    // 返回自定义对象（需@ResponseBody）
    @RequestMapping("/object")
    @ResponseBody
    public User returnObject() {
        User user = new User();
        user.setUsername("admin");
        return user;  // 自动转为JSON
    }
}
```

**返回值类型总结：**
| 返回类型 | 说明 |
|---------|------|
| String | 视图名 |
| ModelAndView | 模型数据+视图 |
| void | 直接操作response |
| Object | 需@ResponseBody，转为JSON |
| View | 自定义视图对象 |

### 3.3 域对象数据传递

向不同域对象设置数据：

```java
@Controller
@RequestMapping("/domain")
public class DomainController {
    
    // 向Request域设置数据
    @RequestMapping("/request")
    public String requestScope(Model model, HttpServletRequest request) {
        model.addAttribute("msg", "Model设置");
        request.setAttribute("msg2", "Request设置");
        return "page";
    }
    
    // 向Session域设置数据
    @RequestMapping("/session")
    public String sessionScope(HttpSession session) {
        session.setAttribute("user", "admin");
        return "page";
    }
    
    // 向Application域设置数据
    @RequestMapping("/application")
    public String applicationScope(HttpSession session) {
        ServletContext context = session.getServletContext();
        context.setAttribute("appName", "MyApp");
        return "page";
    }
    
    // 使用@SessionAttributes注解
    @SessionAttributes("user")
    public class SessionController {
        @RequestMapping("/sessionAttr")
        public String sessionAttr(Model model) {
            model.addAttribute("user", "admin");  // 自动放入Session
            return "page";
        }
    }
}
```

**域对象生命周期：**
| 域对象 | 生命周期 | 作用范围 |
|--------|---------|---------|
| Request | 单次请求 | 当前请求 |
| Session | 一次会话 | 当前用户 |
| Application | 应用启动到停止 | 整个应用 |

### 3.4 请求转发与重定向

控制请求的转发与重定向：

```java
@Controller
@RequestMapping("/redirect")
public class RedirectController {
    
    // 请求转发（默认）
    @RequestMapping("/forward")
    public String forward() {
        return "forward:/user/list";
    }
    
    // 重定向
    @RequestMapping("/redirect")
    public String redirect() {
        return "redirect:/user/list";
    }
    
    // 带参数重定向
    @RequestMapping("/redirectParam")
    public String redirectParam() {
        return "redirect:/user/detail?id=1";
    }
}
```

**转发与重定向区别：**

| 特性 | 转发（forward） | 重定向（redirect） |
|------|---------------|------------------|
| URL变化 | 不变 | 改变 |
| 请求次数 | 1次 | 2次 |
| 数据传递 | 可共享Request数据 | 不可共享 |
| 访问范围 | 仅内部资源 | 可外部资源 |

### 3.5 HttpMessageConverter原理

负责HTTP请求/响应的消息转换：

```java
// 内置MessageConverter
// StringHttpMessageConverter - 字符串转换
// MappingJackson2HttpMessageConverter - JSON转换（需jackson依赖）
// ByteArrayHttpMessageConverter - 字节数组转换
// FormHttpMessageConverter - 表单转换

@Controller
@RequestMapping("/converter")
public class ConverterController {
    
    // 请求体JSON → 对象
    @PostMapping("/json")
    @ResponseBody
    public User processJson(@RequestBody User user) {
        System.out.println("接收JSON: " + user.getUsername());
        return user;  // 对象 → JSON响应
    }
    
    // 自定义MessageConverter
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
        @Override
        public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
            // 配置自定义转换器
            converters.add(new MyMessageConverter());
        }
    }
}
```

**@ResponseBody工作原理：**
1. 返回对象时，查找匹配的HttpMessageConverter
2. 根据Accept请求头选择合适的转换器
3. 将对象转换为响应体（如JSON）
4. 设置响应Content-Type

---

## 4. 核心注解详解

### 4.1 @Controller与@RequestMapping

**@Controller**：标识控制器类

```java
@Controller
@RequestMapping("/user")  // 类级别映射
public class UserController {
    
    @RequestMapping("/list")  // 方法级别映射
    public String list() {
        return "user/list";
    }
    
    // 指定请求方法
    @RequestMapping(value = "/add", method = RequestMethod.POST)
    public String add() {
        return "success";
    }
}
```

**@RequestMapping属性：**
| 属性 | 说明 | 示例 |
|------|------|------|
| value | 请求路径 | "/user/list" |
| method | 请求方法 | POST/GET/PUT/DELETE |
| params | 请求参数条件 | "id!=1" |
| headers | 请求头条件 | "Accept=application/json" |
| consumes | 消费内容类型 | "application/json" |
| produces | 生产内容类型 | "application/json" |

### 4.2 @RequestParam、@RequestHeader、@CookieValue

**@RequestParam**：获取请求参数

```java
@Controller
@RequestMapping("/param")
public class ParamController {
    
    // 默认绑定
    @RequestMapping("/test1")
    public String test1(@RequestParam String username) {
        return "success";
    }
    
    // 指定参数名
    @RequestMapping("/test2")
    public String test2(@RequestParam("name") String username) {
        return "success";
    }
    
    // 可选参数
    @RequestMapping("/test3")
    public String test3(@RequestParam(value = "age", required = false) Integer age) {
        return "success";
    }
    
    // 默认值
    @RequestMapping("/test4")
    public String test4(@RequestParam(defaultValue = "10") Integer pageSize) {
        return "success";
    }
}
```

**@RequestHeader**：获取请求头信息

```java
@RequestMapping("/header")
public String getHeader(@RequestHeader("User-Agent") String userAgent) {
    System.out.println("浏览器: " + userAgent);
    return "success";
}
```

**@CookieValue**：获取Cookie值

```java
@RequestMapping("/cookie")
public String getCookie(@CookieValue("JSESSIONID") String sessionId) {
    System.out.println("SessionID: " + sessionId);
    return "success";
}
```

### 4.3 @SessionAttributes与@ModelAttribute

**@SessionAttributes**：将Model属性同步到Session

```java
@Controller
@RequestMapping("/session")
@SessionAttributes("user")  // 指定需要存入Session的属性
public class SessionController {
    
    @RequestMapping("/set")
    public String setSession(Model model) {
        User user = new User();
        user.setUsername("admin");
        model.addAttribute("user", user);  // 自动存入Session
        return "success";
    }
    
    @RequestMapping("/get")
    public String getSession(@ModelAttribute("user") User user) {
        System.out.println("Session中的用户: " + user.getUsername());
        return "success";
    }
}
```

**@ModelAttribute**：预处理方法或获取Model属性

```java
@Controller
@RequestMapping("/model")
public class ModelController {
    
    // 前置方法，每次请求前执行
    @ModelAttribute
    public void initModel(Model model) {
        model.addAttribute("common", "公共数据");
    }
    
    // 获取Model属性
    @RequestMapping("/test")
    public String test(@ModelAttribute("common") String common) {
        System.out.println("公共数据: " + common);
        return "success";
    }
}
```

### 4.4 RESTful风格与@PathVariable

**RESTful风格**：使用URL表达资源和操作

```java
@Controller
@RequestMapping("/api/users")
public class UserRestController {
    
    // GET /api/users - 查询所有用户
    @RequestMapping(method = RequestMethod.GET)
    @ResponseBody
    public List<User> list() {
        return userService.findAll();
    }
    
    // GET /api/users/1 - 查询单个用户
    @RequestMapping(value = "/{id}", method = RequestMethod.GET)
    @ResponseBody
    public User getById(@PathVariable Integer id) {
        return userService.findById(id);
    }
    
    // POST /api/users - 新增用户
    @RequestMapping(method = RequestMethod.POST)
    @ResponseBody
    public User create(@RequestBody User user) {
        return userService.save(user);
    }
    
    // PUT /api/users/1 - 更新用户
    @RequestMapping(value = "/{id}", method = RequestMethod.PUT)
    @ResponseBody
    public User update(@PathVariable Integer id, @RequestBody User user) {
        user.setId(id);
        return userService.save(user);
    }
    
    // DELETE /api/users/1 - 删除用户
    @RequestMapping(value = "/{id}", method = RequestMethod.DELETE)
    @ResponseBody
    public void delete(@PathVariable Integer id) {
        userService.delete(id);
    }
}
```

**@PathVariable**：从URL路径中提取参数

| HTTP方法 | URL | 操作 |
|---------|-----|------|
| GET | /api/users | 查询列表 |
| GET | /api/users/{id} | 查询单个 |
| POST | /api/users | 新增 |
| PUT | /api/users/{id} | 更新 |
| DELETE | /api/users/{id} | 删除 |

### 4.5 HiddenHttpMethodFilter

将POST请求转换为PUT/DELETE等HTTP方法：

```java
@Configuration
public class WebConfig {
    
    @Bean
    public FilterRegistrationBean<HiddenHttpMethodFilter> httpMethodFilter() {
        FilterRegistrationBean<HiddenHttpMethodFilter> registration = 
                new FilterRegistrationBean<>();
        registration.setFilter(new HiddenHttpMethodFilter());
        registration.addUrlPatterns("/*");
        return registration;
    }
}
```

**前端表单使用：**
```html
<form action="/api/users/1" method="post">
    <input type="hidden" name="_method" value="PUT">
    <button type="submit">更新用户</button>
</form>
```

**工作原理：**
1. 表单发送POST请求，携带`_method=PUT`参数
2. HiddenHttpMethodFilter拦截请求
3. 将请求方法改为PUT
4. DispatcherServlet按PUT方法查找处理器

### 4.6 @ResponseBody与@RestController

**@ResponseBody**：将返回值直接写入响应体

```java
@Controller
@RequestMapping("/api")
public class ApiController {
    
    @RequestMapping("/user")
    @ResponseBody
    public User getUser() {
        User user = new User();
        user.setUsername("admin");
        return user;  // 自动转为JSON
    }
}
```

**@RestController**：@Controller + @ResponseBody的组合

```java
@RestController  // 等价于 @Controller + @ResponseBody
@RequestMapping("/api")
public class ApiController {
    
    @GetMapping("/user")
    public User getUser() {
        User user = new User();
        user.setUsername("admin");
        return user;  // 自动转为JSON
    }
    
    @GetMapping("/users")
    public List<User> getUsers() {
        return userService.findAll();
    }
}
```

**区别对比：**
| 注解 | 作用 | 返回值处理 |
|------|------|-----------|
| @Controller | 标识控制器 | 返回视图名 |
| @ResponseBody | 响应体转换 | 返回对象转为JSON |
| @RestController | 组合注解 | 所有方法返回JSON |

### 4.7 @RequestBody

**@RequestBody**：接收请求体JSON数据

```java
@RestController
@RequestMapping("/api")
public class ApiController {
    
    // 接收JSON请求体
    @PostMapping("/user")
    public User createUser(@RequestBody User user) {
        System.out.println("用户名: " + user.getUsername());
        return userService.save(user);
    }
    
    // 接收复杂JSON
    @PostMapping("/order")
    public Order createOrder(@RequestBody OrderDTO orderDTO) {
        return orderService.create(orderDTO);
    }
}
```

**注意事项：**
- 需要引入Jackson依赖
- 请求Content-Type需为application/json
- 支持复杂对象和集合类型

### 4.8 静态资源映射

配置静态资源访问路径：

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        // 映射/static/**到classpath:/static/
        registry.addResourceHandler("/static/**")
                .addResourceLocations("classpath:/static/");
        
        // 映射/images/**到classpath:/images/
        registry.addResourceHandler("/images/**")
                .addResourceLocations("classpath:/images/");
        
        // 映射外部目录
        registry.addResourceHandler("/uploads/**")
                .addResourceLocations("file:D:/uploads/");
    }
}
```

**静态资源目录结构：**
```
src/main/resources/
├── static/          # CSS、JS、图标等
├── images/          # 图片资源
└── templates/       # 视图模板
```

**访问方式：**
- CSS：`/static/style.css` → `classpath:/static/style.css`
- 图片：`/images/logo.png` → `classpath:/images/logo.png`
- 上传文件：`/uploads/test.jpg` → `D:/uploads/test.jpg`

### 4.9 Postman使用说明

Postman是常用的API测试工具：

**基本使用步骤：**
1. 选择HTTP方法（GET/POST/PUT/DELETE）
2. 输入请求URL
3. 设置请求头（如Content-Type）
4. 添加请求参数或请求体
5. 点击Send发送请求
6. 查看响应结果

**POST请求示例：**
```
URL: http://localhost:8080/api/users
Method: POST
Headers: Content-Type: application/json
Body (raw JSON):
{
    "username": "admin",
    "age": 25,
    "email": "admin@test.com"
}
```

**GET请求示例：**
```
URL: http://localhost:8080/api/users?id=1
Method: GET
```

**常用功能：**
- Collections：管理一组API接口
- Environment：管理不同环境（开发/测试/生产）
- Tests：编写自动化测试脚本
- Mock Server：模拟接口响应

---

## 5. 文件上传与下载

### 5.1 SpringMVC文件上传

配置文件上传解析器：

```java
@Configuration
public class WebConfig {
    
    @Bean
    public MultipartResolver multipartResolver() {
        CommonsMultipartResolver resolver = new CommonsMultipartResolver();
        resolver.setMaxUploadSize(10 * 1024 * 1024);  // 10MB
        resolver.setMaxUploadSizePerFile(5 * 1024 * 1024);  // 单文件5MB
        resolver.setDefaultEncoding("UTF-8");
        return resolver;
    }
}

@Controller
@RequestMapping("/upload")
public class UploadController {
    
    @PostMapping("/single")
    public String uploadFile(@RequestParam("file") MultipartFile file) throws IOException {
        if (!file.isEmpty()) {
            // 获取文件名
            String filename = file.getOriginalFilename();
            // 保存文件
            file.transferTo(new File("D:/uploads/" + filename));
        }
        return "success";
    }
}
```

**前端表单：**

```html
<form action="/upload/single" method="post" enctype="multipart/form-data">
    <input type="file" name="file">
    <button type="submit">上传</button>
</form>
```

### 5.2 多文件上传

支持同时上传多个文件：

```java
@Controller
@RequestMapping("/upload")
public class UploadController {
    
    // 方式1：数组
    @PostMapping("/multi")
    public String uploadMulti(@RequestParam("files") MultipartFile[] files) throws IOException {
        for (MultipartFile file : files) {
            if (!file.isEmpty()) {
                file.transferTo(new File("D:/uploads/" + file.getOriginalFilename()));
            }
        }
        return "success";
    }
    
    // 方式2：List
    @PostMapping("/multiList")
    public String uploadMultiList(@RequestParam("files") List<MultipartFile> files) throws IOException {
        for (MultipartFile file : files) {
            file.transferTo(new File("D:/uploads/" + file.getOriginalFilename()));
        }
        return "success";
    }
}
```

**前端表单：**

```html
<form action="/upload/multi" method="post" enctype="multipart/form-data">
    <input type="file" name="files" multiple>
    <button type="submit">批量上传</button>
</form>
```

### 5.3 异步上传

使用Ajax实现异步文件上传：

```html
<!-- 前端页面 -->
<input type="file" id="fileInput">
<button onclick="uploadFile()">上传</button>

<script>
function uploadFile() {
    const fileInput = document.getElementById('fileInput');
    const file = fileInput.files[0];
    
    const formData = new FormData();
    formData.append('file', file);
    
    fetch('/upload/async', {
        method: 'POST',
        body: formData
    })
    .then(response => response.json())
    .then(data => {
        console.log('上传成功:', data);
    })
    .catch(error => {
        console.error('上传失败:', error);
    });
}
</script>
```

**后端控制器：**
```java
@Controller
@RequestMapping("/upload")
public class UploadController {
    
    @PostMapping("/async")
    @ResponseBody
    public Map<String, Object> asyncUpload(@RequestParam("file") MultipartFile file) 
            throws IOException {
        Map<String, Object> result = new HashMap<>();
        
        if (!file.isEmpty()) {
            file.transferTo(new File("D:/uploads/" + file.getOriginalFilename()));
            result.put("success", true);
            result.put("message", "上传成功");
        } else {
            result.put("success", false);
            result.put("message", "请选择文件");
        }
        
        return result;
    }
}
```

### 5.4 跨服务器上传

上传文件到其他服务器：

```java
@Controller
@RequestMapping("/upload")
public class UploadController {
    
    // 目标服务器地址
    private static final String SERVER_URL = "http://localhost:8081/upload/receive";
    
    @PostMapping("/cross")
    @ResponseBody
    public Map<String, Object> crossServerUpload(@RequestParam("file") MultipartFile file) 
            throws IOException {
        Map<String, Object> result = new HashMap<>();
        
        RestTemplate restTemplate = new RestTemplate();
        
        // 将文件转为字节数组
        byte[] bytes = file.getBytes();
        ByteArrayResource resource = new ByteArrayResource(bytes) {
            @Override
            public String getFilename() {
                return file.getOriginalFilename();
            }
        };
        
        // 构建请求体
        MultiValueMap<String, Object> body = new LinkedMultiValueMap<>();
        body.add("file", resource);
        
        // 设置请求头
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.MULTIPART_FORM_DATA);
        
        HttpEntity<MultiValueMap<String, Object>> request = 
                new HttpEntity<>(body, headers);
        
        // 发送请求到目标服务器
        ResponseEntity<String> response = restTemplate.postForEntity(
                SERVER_URL, request, String.class);
        
        result.put("success", response.getStatusCode().is2xxSuccessful());
        result.put("message", response.getBody());
        
        return result;
    }
}

// 目标服务器接收端
@Controller
@RequestMapping("/upload")
public class ReceiveController {
    
    @PostMapping("/receive")
    @ResponseBody
    public String receiveFile(@RequestParam("file") MultipartFile file) 
            throws IOException {
        file.transferTo(new File("D:/remote_uploads/" + file.getOriginalFilename()));
        return "文件接收成功";
    }
}
```

### 5.5 文件下载

实现文件下载功能：

```java
@Controller
@RequestMapping("/download")
public class DownloadController {
    
    @GetMapping("/file")
    public ResponseEntity<Resource> downloadFile(@RequestParam String filename) throws IOException {
        // 获取文件
        File file = new File("D:/uploads/" + filename);
        Resource resource = new FileSystemResource(file);
        
        // 设置响应头
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
        headers.setContentDisposition(ContentDisposition.attachment()
                .filename(filename, StandardCharsets.UTF_8).build());
        
        return new ResponseEntity<>(resource, headers, HttpStatus.OK);
    }
    
    // 传统方式下载
    @GetMapping("/file2")
    public void downloadFile2(@RequestParam String filename, 
                              HttpServletResponse response) throws IOException {
        File file = new File("D:/uploads/" + filename);
        
        response.setContentType("application/octet-stream");
        response.setHeader("Content-Disposition", "attachment; filename=" + 
                URLEncoder.encode(filename, "UTF-8"));
        
        // 复制文件流到响应
        Files.copy(file.toPath(), response.getOutputStream());
    }
}
```

**下载链接：**

```html
<a href="/download/file?filename=test.pdf">下载文件</a>
```

---

## 6. 异常处理

### 6.1 单个控制器异常处理

使用@ExceptionHandler处理当前控制器的异常：

```java
@Controller
@RequestMapping("/exception")
public class ExceptionController {
    
    @RequestMapping("/test")
    public String test() {
        throw new RuntimeException("运行时异常");
    }
    
    // 处理当前控制器的所有异常
    @ExceptionHandler(Exception.class)
    public String handleException(Exception e, Model model) {
        model.addAttribute("error", e.getMessage());
        return "error";
    }
    
    // 处理特定类型异常
    @ExceptionHandler(NullPointerException.class)
    public String handleNullPointerException(NullPointerException e) {
        return "nullError";
    }
}
```

### 6.2 全局异常处理

使用@ControllerAdvice实现全局异常处理：

```java
@ControllerAdvice  // 全局异常处理器
public class GlobalExceptionHandler {
    
    // 处理所有异常
    @ExceptionHandler(Exception.class)
    public ModelAndView handleException(Exception e) {
        ModelAndView mv = new ModelAndView("error");
        mv.addObject("error", e.getMessage());
        return mv;
    }
    
    // 处理运行时异常
    @ExceptionHandler(RuntimeException.class)
    @ResponseBody
    public Map<String, Object> handleRuntimeException(RuntimeException e) {
        Map<String, Object> result = new HashMap<>();
        result.put("code", 500);
        result.put("message", e.getMessage());
        return result;
    }
    
    // 处理自定义异常
    @ExceptionHandler(BusinessException.class)
    @ResponseBody
    public Map<String, Object> handleBusinessException(BusinessException e) {
        Map<String, Object> result = new HashMap<>();
        result.put("code", e.getCode());
        result.put("message", e.getMessage());
        return result;
    }
}

// 自定义异常
public class BusinessException extends RuntimeException {
    private int code;
    
    public BusinessException(int code, String message) {
        super(message);
        this.code = code;
    }
    
    public int getCode() { return code; }
}
```

### 6.3 HandlerExceptionResolver原理

SpringMVC的异常解析机制：

```java
// HandlerExceptionResolver接口
public interface HandlerExceptionResolver {
    ModelAndView resolveException(HttpServletRequest request,
                                   HttpServletResponse response,
                                   Object handler,
                                   Exception ex);
}

// 内置实现
// SimpleMappingExceptionResolver - 配置异常与视图映射
// DefaultHandlerExceptionResolver - 处理SpringMVC内置异常
// ExceptionHandlerExceptionResolver - 处理@ExceptionHandler注解

@Configuration
public class WebConfig {
    
    @Bean
    public HandlerExceptionResolver customExceptionResolver() {
        SimpleMappingExceptionResolver resolver = new SimpleMappingExceptionResolver();
        
        // 配置异常映射
        Properties mappings = new Properties();
        mappings.setProperty("java.lang.RuntimeException", "runtimeError");
        mappings.setProperty("com.example.BusinessException", "businessError");
        resolver.setExceptionMappings(mappings);
        
        // 默认错误视图
        resolver.setDefaultErrorView("error");
        return resolver;
    }
}
```

**异常处理优先级：**
1. @ExceptionHandler（控制器级别）
2. @ControllerAdvice（全局级别）
3. HandlerExceptionResolver（配置级别）

---

## 7. 拦截器

### 7.1 拦截器简介

SpringMVC拦截器用于在请求处理前后执行特定逻辑：

**拦截器与Filter的区别：**

| 特性 | 拦截器（Interceptor） | 过滤器（Filter） |
|------|----------------------|----------------|
| 所属框架 | SpringMVC | Servlet |
| 触发时机 | DispatcherServlet之后 | DispatcherServlet之前 |
| 功能 | 方法级别拦截 | 请求级别拦截 |
| 访问对象 | Spring容器Bean | 仅Servlet API |
| 配置方式 | Spring配置 | web.xml或注解 |

### 7.2 拦截器的使用

实现HandlerInterceptor接口：

```java
public class LoginInterceptor implements HandlerInterceptor {
    
    // 请求处理前执行（返回true继续，false中断）
    @Override
    public boolean preHandle(HttpServletRequest request, 
                             HttpServletResponse response, 
                             Object handler) throws Exception {
        // 检查登录状态
        HttpSession session = request.getSession();
        if (session.getAttribute("user") == null) {
            // 未登录，重定向到登录页
            response.sendRedirect("/login");
            return false;
        }
        return true;
    }
    
    // 请求处理后，视图渲染前执行
    @Override
    public void postHandle(HttpServletRequest request, 
                           HttpServletResponse response, 
                           Object handler, 
                           ModelAndView modelAndView) throws Exception {
        // 添加公共数据到视图
        if (modelAndView != null) {
            modelAndView.addObject("currentTime", new Date());
        }
    }
    
    // 视图渲染后执行（用于资源清理）
    @Override
    public void afterCompletion(HttpServletRequest request, 
                                HttpServletResponse response, 
                                Object handler, 
                                Exception ex) throws Exception {
        // 记录日志
        System.out.println("请求处理完成");
    }
}
```

**注册拦截器：**

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/**")           // 拦截所有路径
                .excludePathPatterns("/login");   // 排除登录页
    }
}
```

### 7.3 拦截器链与执行顺序

支持多个拦截器组成拦截器链：

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 按注册顺序执行
        registry.addInterceptor(new FirstInterceptor())
                .addPathPatterns("/**");
        
        registry.addInterceptor(new SecondInterceptor())
                .addPathPatterns("/**");
    }
}
```

**执行顺序：**

```
请求 → preHandle1 → preHandle2 → Controller方法
    → postHandle2 → postHandle1 → 视图渲染
    → afterCompletion2 → afterCompletion1 → 响应
```

**执行流程说明：**
- preHandle按顺序执行，任意返回false则中断
- postHandle按逆序执行
- afterCompletion按逆序执行

### 7.4 过滤敏感词实战

使用拦截器过滤请求中的敏感词：

```java
public class SensitiveWordInterceptor implements HandlerInterceptor {
    
    // 敏感词列表
    private static final Set<String> SENSITIVE_WORDS = Set.of(
            "敏感词1", "敏感词2", "敏感词3"
    );
    
    @Override
    public boolean preHandle(HttpServletRequest request, 
                             HttpServletResponse response, 
                             Object handler) throws Exception {
        
        // 获取请求参数
        Map<String, String[]> parameterMap = request.getParameterMap();
        
        for (String key : parameterMap.keySet()) {
            String[] values = parameterMap.get(key);
            for (String value : values) {
                if (containsSensitiveWord(value)) {
                    // 返回错误信息
                    response.setContentType("application/json;charset=UTF-8");
                    response.getWriter().write(
                            "{\"code\": 400, \"message\": \"内容包含敏感词\"}");
                    return false;
                }
            }
        }
        
        return true;
    }
    
    private boolean containsSensitiveWord(String content) {
        for (String word : SENSITIVE_WORDS) {
            if (content.contains(word)) {
                return true;
            }
        }
        return false;
    }
}
```

**注册拦截器：**
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new SensitiveWordInterceptor())
                .addPathPatterns("/**")
                .excludePathPatterns("/login", "/register");
    }
}
```

**高级敏感词过滤（使用DFA算法）：**
```java
public class SensitiveWordFilter {
    
    private Map<Character, Object> sensitiveWordMap;
    
    public void init(Set<String> sensitiveWords) {
        sensitiveWordMap = new HashMap<>();
        for (String word : sensitiveWords) {
            Map<Character, Object> currentMap = sensitiveWordMap;
            for (int i = 0; i < word.length(); i++) {
                char c = word.charAt(i);
                if (!currentMap.containsKey(c)) {
                    currentMap.put(c, new HashMap<>());
                }
                currentMap = (Map<Character, Object>) currentMap.get(c);
                if (i == word.length() - 1) {
                    currentMap.put('isEnd', true);
                }
            }
        }
    }
    
    public boolean containsSensitiveWord(String content) {
        for (int i = 0; i < content.length(); i++) {
            if (checkSensitiveWord(content, i)) {
                return true;
            }
        }
        return false;
    }
    
    private boolean checkSensitiveWord(String content, int startIndex) {
        Map<Character, Object> currentMap = sensitiveWordMap;
        for (int i = startIndex; i < content.length(); i++) {
            char c = content.charAt(i);
            if (!currentMap.containsKey(c)) {
                break;
            }
            currentMap = (Map<Character, Object>) currentMap.get(c);
            if (currentMap.containsKey('isEnd')) {
                return true;
            }
        }
        return false;
    }
}
```

### 7.5 拦截器与Filter区别

**详细对比：**

| 维度 | 拦截器 | 过滤器 |
|------|--------|--------|
| **技术基础** | Spring AOP | Servlet规范 |
| **作用范围** | 仅SpringMVC请求 | 所有Web请求 |
| **执行顺序** | 在DispatcherServlet之后 | 在DispatcherServlet之前 |
| **可访问对象** | Controller、Service、Model | 仅HttpServletRequest/Response |
| **配置方式** | Java配置/WebMvcConfigurer | web.xml/@WebFilter |
| **中断方式** | 返回false | chain.doFilter()不调用 |
| **异常处理** | 可通过@ExceptionHandler处理 | 需要自己处理 |

**使用场景建议：**
- **Filter**：编码处理、全局日志、跨域处理
- **Interceptor**：登录校验、权限控制、请求耗时统计

---

## 8. 跨域请求

### 8.1 同源策略

同源策略是浏览器的安全机制，限制不同源的页面之间的交互。

**同源条件：**
- 协议相同（http/https）
- 域名相同
- 端口相同

**跨域场景：**
| 场景 | 是否跨域 | 说明 |
|------|---------|------|
| http://a.com → http://a.com | 否 | 同源 |
| http://a.com → https://a.com | 是 | 协议不同 |
| http://a.com → http://b.com | 是 | 域名不同 |
| http://a.com:8080 → http://a.com:8081 | 是 | 端口不同 |

### 8.2 SpringMVC跨域处理

**方式1：@CrossOrigin注解**

```java
@RestController
@RequestMapping("/api")
@CrossOrigin(origins = "http://localhost:8080")  // 允许指定源跨域
public class ApiController {
    
    @GetMapping("/user")
    @CrossOrigin(origins = {"http://localhost:8080", "http://localhost:9090"})
    public User getUser() {
        return userService.getUser();
    }
}
```

**方式2：全局配置**

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")                // 匹配路径
                .allowedOrigins("http://localhost:8080")  // 允许的源
                .allowedMethods("GET", "POST", "PUT", "DELETE")  // 允许的方法
                .allowedHeaders("*")                  // 允许的请求头
                .allowCredentials(true)              // 是否允许携带凭证
                .maxAge(3600);                       // 预检请求缓存时间
    }
}
```

**方式3：CorsFilter过滤器**

```java
@Configuration
public class CorsConfig {
    
    @Bean
    public CorsFilter corsFilter() {
        CorsConfiguration config = new CorsConfiguration();
        config.addAllowedOrigin("http://localhost:8080");
        config.addAllowedMethod("*");
        config.addAllowedHeader("*");
        config.setAllowCredentials(true);
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        
        return new CorsFilter(source);
    }
}
```

**跨域请求流程：**

```
1. 浏览器发送预检请求（OPTIONS）
2. 服务器返回跨域允许信息
3. 浏览器确认允许后，发送真实请求
4. 服务器处理并返回响应
```

**常见问题：**
- 需同时配置`allowedOrigins`和`allowedMethods`
- `allowCredentials=true`时，`allowedOrigins`不能为`*`
- 预检请求会缓存，`maxAge`控制缓存时间

---

## 附录：常用注解速查

| 注解 | 作用 | 使用位置 |
|------|------|---------|
| @Controller | 标识控制器类 | 类 |
| @RestController | REST控制器（@Controller+@ResponseBody） | 类 |
| @RequestMapping | 请求映射 | 类/方法 |
| @GetMapping | GET请求映射 | 方法 |
| @PostMapping | POST请求映射 | 方法 |
| @PutMapping | PUT请求映射 | 方法 |
| @DeleteMapping | DELETE请求映射 | 方法 |
| @RequestParam | 获取请求参数 | 方法参数 |
| @PathVariable | 获取路径参数 | 方法参数 |
| @RequestBody | 获取请求体JSON | 方法参数 |
| @ResponseBody | 将返回值写入响应体 | 方法/类 |
| @RequestHeader | 获取请求头 | 方法参数 |
| @CookieValue | 获取Cookie值 | 方法参数 |
| @SessionAttributes | 同步Model到Session | 类 |
| @ModelAttribute | 预处理/获取Model属性 | 方法/参数 |
| @Valid | 触发数据校验 | 方法参数 |
| @ExceptionHandler | 处理异常 | 方法 |
| @ControllerAdvice | 全局异常处理 | 类 |
| @CrossOrigin | 跨域配置 | 类/方法 |
