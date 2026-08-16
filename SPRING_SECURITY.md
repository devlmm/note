# Spring Security 核心技术文档

> 由浅入深，环环相扣 | 每个知识点配有精简代码示例与注释
>
> **版本基线**：Spring Security 5.7.x / Spring Boot 2.7.x（与 5.x 全系列通用，6.x 仅配置风格略有差异，文中会注明）。

---

## 目录

- [1. Spring Security 介绍](#1-spring-security-介绍)
  - [1.1 是什么](#11-是什么)
  - [1.2 核心概念](#12-核心概念)
  - [1.3 整体过滤链](#13-整体过滤链)
- [2. 认证（Authentication）](#2-认证authentication)
  - [2.1 项目搭建](#21-项目搭建)
  - [2.2 内存认证](#22-内存认证)
  - [2.3 UserDetailsService](#23-userdetailsservice)
  - [2.4 数据库认证](#24-数据库认证)
  - [2.5 PasswordEncoder](#25-passwordencoder)
  - [2.6 自定义登录页面](#26-自定义登录页面)
  - [2.7 CSRF 防护](#27-csrf-防护)
  - [2.8 认证成功后的处理方式](#28-认证成功后的处理方式)
  - [2.9 认证失败后的处理方式](#29-认证失败后的处理方式)
  - [2.10 退出登录](#210-退出登录)
  - [2.11 退出成功处理器](#211-退出成功处理器)
  - [2.12 RememberMe](#212-rememberme)
  - [2.13 会话管理](#213-会话管理)
  - [2.14 会话失效处理](#214-会话失效处理)
  - [2.15 会话并发控制](#215-会话并发控制)
  - [2.16 主动踢人下线](#216-主动踢人下线)
- [3. 授权（Authorization）](#3-授权authorization)
  - [3.1 RBAC 模型](#31-rbac-模型)
  - [3.2 权限表设计](#32-权限表设计)
  - [3.3 编写查询权限方法](#33-编写查询权限方法)
  - [3.4 配置类设置访问控制](#34-配置类设置访问控制)
  - [3.5 自定义访问控制逻辑](#35-自定义访问控制逻辑)
  - [3.6 注解设置访问控制](#36-注解设置访问控制)
  - [3.7 在前端进行访问控制](#37-在前端进行访问控制)
  - [3.8 403 处理方案](#38-403-处理方案)
- [4. 前后端分离架构](#4-前后端分离架构)
  - [4.1 架构设计：传统MVC vs 前后端分离](#41-架构设计传统mvc-vs-前后端分离)
  - [4.2 两种主流认证方案对比](#42-两种主流认证方案对比)
  - [4.3 核心组件与职责划分](#43-核心组件与职责划分)
  - [4.4 登录认证交互流程](#44-登录认证交互流程)
  - [4.5 业务请求授权交互流程](#45-业务请求授权交互流程)
  - [4.6 搭建步骤①：Maven 依赖引入](#46-搭建步骤maven-依赖引入)
  - [4.7 搭建步骤②：Security 核心配置](#47-搭建步骤security-核心配置)
  - [4.8 搭建步骤③：JWT 工具类](#48-搭建步骤jwt-工具类)
  - [4.9 搭建步骤④：JWT 认证过滤器](#49-搭建步骤jwt-认证过滤器)
  - [4.10 搭建步骤⑤：登录接口实现](#410-搭建步骤登录接口实现)
  - [4.11 搭建步骤⑥：统一异常与认证入口](#411-搭建步骤统一异常与认证入口)
  - [4.12 搭建步骤⑦：跨域 CORS 配置](#412-搭建步骤跨域-cors-配置)
  - [4.13 前端联调要点](#413-前端联调要点)
  - [4.14 常见陷阱与最佳实践](#414-常见陷阱与最佳实践)

---

## 1. Spring Security 介绍

### 1.1 是什么

Spring Security 是 Spring 生态中专注于**认证（Authentication，"你是谁"）**与**授权（Authorization，"你能做什么"）**的安全框架，同时提供 CSRF、会话固定、安全头、密码加密等开箱即用的防护能力。

**核心价值**：

- 与 Spring Boot 无缝集成，依赖一行即可启用整套安全过滤链。
- 高度可定制：任何过滤器、认证提供者、决策器都可替换。
- 业界标准：默认遵循 OWASP 最佳实践（密码哈希、CSRF Token、安全响应头）。

### 1.2 核心概念

| 概念 | 说明 |
|------|------|
| `Authentication` | 认证对象，封装"主体（Principal）、凭证（Credentials）、权限（Authorities）" |
| `AuthenticationManager` | 认证入口，委托给 `AuthenticationProvider` 完成具体认证 |
| `UserDetailsService` | 用户加载接口，返回 `UserDetails`（用户名、密码、权限） |
| `PasswordEncoder` | 密码编码器，负责加密与比对 |
| `SecurityContext` | 持有当前 `Authentication`，默认存于 `SecurityContextHolder`（ThreadLocal） |
| `FilterChainProxy` | 安全过滤链代理，串联所有 `Filter` |
| `GrantedAuthority` | 权限标识，如 `ROLE_ADMIN`、`user:read` |

### 1.3 整体过滤链

请求进入后依次经过一组安全过滤器，最终到达 Controller：

```
HTTP Request
   │
   ▼
FilterChainProxy（SecurityFilterChain）
   ├─ SecurityContextPersistenceFilter   // 恢复 SecurityContext
   ├─ UsernamePasswordAuthenticationFilter // 表单登录认证
   ├─ DefaultLoginPageGeneratingFilter  // 默认登录页
   ├─ BasicAuthenticationFilter          // HTTP Basic 认证
   ├─ RememberMeAuthenticationFilter     // 记住我
   ├─ AuthorizationFilter                // 授权决策
   └─ ExceptionTranslationFilter         // 异常翻译
   │
   ▼
Controller
```

> **关键认知**：Spring Security 的本质就是一条 Servlet Filter 链。理解了过滤链，就理解了 Spring Security 的运行机理。

---

## 2. 认证（Authentication）

### 2.1 项目搭建

**Maven 依赖**（Spring Boot 场景下引入 starter 即可）：

```xml
<!-- 引入安全 starter：自动装配过滤链与默认配置 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

**最小可用配置类**（继承 `WebSecurityConfigurerAdapter`，5.7 起已标记 `@Deprecated`，6.x 完全移除，推荐用 `SecurityFilterChain` Bean）：

```java
@Configuration // 5.x 风格：继承适配器（已不推荐）
@EnableWebSecurity // 开启 Web 安全
public class SecurityConfig extends WebSecurityConfigurerAdapter {
}
```

**推荐写法（5.7+ / 6.x）**：

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean // 声明过滤链 Bean，替代继承适配器
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(a -> a.anyRequest().authenticated()) // 所有请求需认证
            .formLogin(f -> {}); // 启用表单登录
        return http.build();
    }
}
```

> **建包建议**：`config/` 放配置，`service/` 放 `UserDetailsService`，`controller/` 放接口，`entity/` 放实体，分工清晰。

### 2.2 内存认证

内存认证用于**快速验证框架是否跑通**，用户信息硬编码在内存中，**严禁用于生产**。

```java
@Bean
public UserDetailsService users() {
    // 内存中存放两个用户，密码用 {noop} 前缀表示明文（仅演示用）
    UserDetails admin = User.withUsername("admin")
            .password("{noop}123456")   // {noop}=NoOpPasswordEncoder，不加密
            .roles("ADMIN")             // 等价于 ROLE_ADMIN
            .build();
    UserDetails user = User.withUsername("user")
            .password("{noop}123456").roles("USER").build();
    return new InMemoryUserDetailsManager(admin, user); // 内存用户管理器
}
```

**验证**：启动后访问任意接口被重定向到 `/login`，输入 `admin/123456` 即可登录。

> **避坑**：若不指定 `{noop}` 前缀又未配置 `PasswordEncoder`，会抛 `There is no PasswordEncoder mapped for the id "null"`。这是初学者最常踩的第一个坑。

### 2.3 UserDetailsService

`UserDetailsService` 是**自定义用户数据源的统一入口**——无论用户在内存、数据库、LDAP 还是 JWT 中，最终都通过它封装为 `UserDetails`。

```java
@Service
public class MyUserDetailsService implements UserDetailsService {
    @Autowired
    private UserMapper userMapper; // 假设已有 MyBatis Mapper

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 1. 按用户名查 DB
        UserEntity user = userMapper.findByUsername(username);
        if (user == null) {
            throw new UsernameNotFoundException("用户不存在"); // 抛异常即认证失败
        }
        // 2. 收集权限（ GrantedAuthority 实现）
        List<SimpleGrantedAuthority> auths = user.getRoles().stream()
                .map(r -> new SimpleGrantedAuthority("ROLE_" + r.getName()))
                .collect(Collectors.toList());
        // 3. 返回 UserDetails（Spring 自带 User.builder）
        return User.withUsername(user.getUsername())
                .password(user.getPassword()) // 必须是已加密的密文
                .authorities(auths)
                .accountExpired(!user.isActive()) // 账号状态字段
                .build();
    }
}
```

> **设计要点**：`UserDetailsService` 只负责"查"，**不负责"比对密码"**。密码比对由 `DaoAuthenticationProvider` 委托 `PasswordEncoder` 完成。职责分离是 Spring Security 的精髓。

### 2.4 数据库认证

数据库认证 = `UserDetailsService` 查 DB + `PasswordEncoder` 校验密码。结合 [2.3](#23-userdetailsservice) 的实现，只需让框架识别即可：

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(a -> a
                .antMatchers("/login", "/css/**").permitAll() // 放行登录与静态资源
                .anyRequest().authenticated())
            .formLogin(f -> f.loginPage("/login").permitAll()) // 自定义登录页
            .csrf(c -> {}); // 暂时启用 CSRF（见 2.7）
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(); // DB 中存储 BCrypt 密文
    }
}
```

**典型表结构**（简化版）：

```sql
CREATE TABLE sys_user (
    id          BIGINT PRIMARY KEY AUTO_INCREMENT,
    username    VARCHAR(50)  UNIQUE NOT NULL,
    password    VARCHAR(100) NOT NULL,  -- BCrypt 密文
    enabled     TINYINT(1)   DEFAULT 1
);
```

> **经验**：密码字段长度建议 ≥ 72，BCrypt 密文固定 60 位，留余量以兼容未来切换算法。

### 2.5 PasswordEncoder

`PasswordEncoder` 统一了"加密"与"校验"两件事。**明文存库是 2005 年以前的做法，现在属于严重安全漏洞**。

| 实现类 | 特点 | 适用场景 |
|--------|------|----------|
| `NoOpPasswordEncoder` | 不加密，明文比对 | **仅测试**，已废弃 |
| `BCryptPasswordEncoder` | 自带盐，可调强度，抗彩虹表 | **生产首选** |
| `Pbkdf2PasswordEncoder` | 可迭代次数 | 兼容旧系统 |
| `SCryptPasswordEncoder` | 内存硬化，抗 GPU 破解 | 高安全要求 |
| `DelegatingPasswordEncoder` | 多算法并存，密文带 `{id}` 前缀 | **迁移过渡** |

```java
@Bean
public PasswordEncoder passwordEncoder() {
    // DelegatingPasswordEncoder：密文形如 {bcrypt}$2a$10$... 支持平滑升级算法
    return PasswordEncoderFactories.createDelegatingPasswordEncoder();
}

// 注册时加密
String raw = "123456";
String encoded = passwordEncoder.encode(raw); // 每次结果不同（自带盐）

// 登录时校验
boolean ok = passwordEncoder.matches(raw, encoded); // true 表示匹配
```

> **算法升级技巧**：用 `DelegatingPasswordEncoder` + `upgradeEncoding(true)`，登录成功时若密文用的是旧算法会自动重新加密为新算法，**零停机完成密码哈希升级**。

### 2.6 自定义登录页面

默认登录页简陋，实际项目都需定制。核心是覆盖 `formLogin` 配置：

```java
http.formLogin(f -> f
    .loginPage("/login.html")           // 自定义登录页地址
    .loginProcessingUrl("/doLogin")     // 表单提交地址（框架拦截）
    .usernameParameter("uname")         // 表单用户名参数名
    .passwordParameter("pwd")           // 表单密码参数名
    .defaultSuccessUrl("/index")       // 登录成功默认跳转
    .failureUrl("/login.html?error")    // 登录失败跳转
    .permitAll());
```

**登录页表单要点**（关键参数名必须与配置一致）：

```html
<form action="/doLogin" method="post">
    <input name="uname" placeholder="用户名"> <!-- 对应 usernameParameter -->
    <input name="pwd" type="password">        <!-- 对应 passwordParameter -->
    <!-- CSRF Token：表单必须带，否则 POST 被拦截（见 2.7） -->
    <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}">
    <button type="submit">登录</button>
</form>
```

> **认知**：`loginProcessingUrl` 由框架接管，**无需自己写 Controller**，框架自动调用 `UsernamePasswordAuthenticationFilter` 完成认证。

### 2.7 CSRF 防护

CSRF（Cross-Site Request Forgery）跨站请求伪造：攻击者诱导已登录用户在恶意网站发起请求。Spring Security 默认对 POST/PUT/DELETE 强制校验 CSRF Token。

```java
// 生产环境：保留默认开启
http.csrf(c -> c.csrfTokenRepository(
        CookieCsrfTokenRepository.withHttpOnlyFalse())); // Token 存 Cookie，前后端分离友好

// 仅后端无前端、内部接口时可关闭（不推荐）
http.csrf(c -> c.disable());
```

**前后端分离场景**（Token 通过响应头下发，前端回写 Header）：

```javascript
// 前端从 Cookie 读取 XSRF-TOKEN，请求头带回 X-XSRF-TOKEN
const token = Cookies.get('XSRF-TOKEN');
axios.post('/api/save', data, { headers: { 'X-XSRF-TOKEN': token } });
```

> **关闭 CSRF 的代价**：相当于把"防伪签名"拆除，恶意网站可伪造请求。**仅纯无状态 API（如 JWT）才可考虑关闭**。

### 2.8 认证成功后的处理方式

`defaultSuccessUrl` 只能做简单跳转。需要返回 JSON、记录日志、生成 Token 时，用 `AuthenticationSuccessHandler`。

```java
http.formLogin(f -> f.successHandler((req, resp, auth) -> {
    // auth.getPrincipal() 即 UserDetails
    Map<String, Object> result = Map.of(
        "code", 200,
        "msg", "登录成功",
        "user", auth.getName());
    resp.setContentType("application/json;charset=UTF-8");
    resp.getWriter().write(new ObjectMapper().writeValueAsString(result)); // 返回 JSON
}));
```

**常用实现**：

- `SavedRequestAwareAuthenticationSuccessHandler`：跳转到登录前被拦截的原始 URL（默认行为）。
- `ForwardAuthenticationSuccessHandler`：服务端 forward。
- 自定义：前后端分离返回 JSON。

> **进阶**：JWT 场景在此处签发 Token，而非在 Controller 中签发——职责更内聚。

### 2.9 认证失败后的处理方式

失败处理对应 `AuthenticationFailureHandler`。默认跳转 `failureUrl`，但前后端分离需要返回 JSON。

```java
http.formLogin(f -> f.failureHandler((req, resp, ex) -> {
    String msg = "登录失败";
    if (ex instanceof BadCredentialsException) msg = "用户名或密码错误";
    else if (ex instanceof LockedException) msg = "账号被锁定";
    else if (ex instanceof DisabledException) msg = "账号被禁用";
    resp.setContentType("application/json;charset=UTF-8");
    resp.getWriter().write("{\"code\":401,\"msg\":\"" + msg + "\"}"); // 返回 JSON
}));
```

> **价值**：异常类型即失败原因，**不要在前端写"用户名或密码错误"的统一提示**——区分"账号锁定"能极大提升用户体验与安全可观测性。

### 2.10 退出登录

退出由 `LogoutFilter` 处理，默认拦截 `/logout`（GET 请求展示确认页，POST 真正退出）。

```java
http.logout(l -> l
    .logoutUrl("/logout")            // 触发退出的 URL（POST）
    .logoutRequestMatcher(new AntPathRequestMatcher("/logout", "POST"))
    .logoutSuccessUrl("/login.html") // 退出成功跳转
    .deleteCookies("JSESSIONID")     // 清除指定 Cookie
    .invalidateHttpSession(true)     // 使 Session 失效
    .clearAuthentication(true));     // 清除认证信息
```

> **常见坑**：前端用 `<a href="/logout">` 退出会触发 GET，CSRF 开启时被拦截。**必须用表单或 AJAX 发 POST**。

### 2.11 退出成功处理器

需要返回 JSON 或记录登出日志时，用 `LogoutSuccessHandler`。

```java
http.logout(l -> l.logoutSuccessHandler((req, resp, auth) -> {
    log.info("用户 {} 已退出", auth != null ? auth.getName() : "未知");
    resp.setContentType("application/json;charset=UTF-8");
    resp.getWriter().write("{\"code\":200,\"msg\":\"退出成功\"}");
}));
```

> **联动设计**：登出时清除 Remember-Me Token、推送下线消息、记录审计日志，都应在此 Handler 中编排。

### 2.12 RememberMe

记住我：用户关闭浏览器后仍能保持登录。分两种策略——**简单哈希**与**持久化 Token**。

```java
@Bean
public PersistentTokenRepository tokenRepository(DataSource ds) {
    JdbcTokenRepositoryImpl repo = new JdbcTokenRepositoryImpl();
    repo.setDataSource(ds);
    repo.setCreateTableOnStartup(true); // 自动建表（生产用 SQL 手工建）
    return repo;
}

http.rememberMe(r -> r
    .key("mySecretKey")                       // 加密 Token 的密钥
    .tokenRepository(tokenRepository)         // 持久化到 DB
    .tokenValiditySeconds(7 * 24 * 3600)       // 有效期 7 天
    .userDetailsService(userDetailsService)); // 重新加载用户
```

**持久化表**：

```sql
CREATE TABLE persistent_logins (
    username  VARCHAR(64)  NOT NULL,
    series    VARCHAR(64)  PRIMARY KEY,
    token     VARCHAR(64)  NOT NULL,
    last_used TIMESTAMP    NOT NULL
);
```

> **安全提示**：持久化 Token 检测到 series 不匹配会**判定 Token 被盗用，强制全设备退出**——这是简单哈希方案没有的防护。

### 2.13 会话管理

Session 是有状态认证的核心载体。`SessionManagementFilter` 负责会话创建与超时控制。

```java
http.sessionManagement(s -> s
    .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED) // 按需创建
    .maximumSessions(1)                                       // 同一用户最大会话数
    .maxSessionsPreventsLogin(false)                          // false=踢旧 false;true=拒新
    .expiredUrl("/login.html?expired"));                      // 被挤下线跳转
```

**`SessionCreationPolicy` 取值**：

| 值 | 行为 |
|----|------|
| `ALWAYS` | 每次请求都创建 Session |
| `IF_REQUIRED` | 需要时才创建（默认） |
| `NEVER` | 不主动创建，存在则用 |
| `STATELESS` | 完全不创建（**REST/JWT 必选**） |

> **JWT 取舍**：选 `STATELESS`，框架会跳过大部分会话相关过滤器，性能与简洁性双赢。

### 2.14 会话失效处理

Session 超时（默认 30 分钟）后，需引导用户重新登录。

**`application.yml` 配置超时**：

```yaml
server:
  servlet:
    session:
      timeout: 30m   # 最小 1 分钟，低于 1 分钟按 1 分钟处理
```

**失效处理**（两种方式二选一）：

```java
// 方式一：失效后跳转
http.sessionManagement(s -> s.invalidSessionUrl("/login.html?invalid"));

// 方式二：失效后返回 JSON（前后端分离）
http.sessionManagement(s -> s.invalidSessionStrategy((req, resp) -> {
    resp.setContentType("application/json;charset=UTF-8");
    resp.getWriter().write("{\"code\":401,\"msg\":\"会话已过期，请重新登录\"}");
}));
```

> **隐藏坑**：会话失效 ≠ 会话并发挤下线。失效是"超时自然过期"，挤下线是"新登录顶掉旧会话"，二者处理器不同，别混淆。

### 2.15 会话并发控制

限制同一账号的并发登录数。底层依赖 `SessionRegistryImpl` 监听会话事件。

```java
@Bean
public HttpSessionEventPublisher httpSessionEventPublisher() {
    return new HttpSessionEventPublisher(); // 必须！否则并发控制不生效
}

http.sessionManagement(s -> s
    .maximumSessions(1)                    // 同一用户最多 1 个会话
    .maxSessionsPreventsLogin(false)      // false=后者踢掉前者；true=后者登录被拒
    .expiredSessionStrategy(e -> {        // 被挤下线时的处理
        e.getResponse().setContentType("application/json;charset=UTF-8");
        e.getResponse().getWriter().write("{\"msg\":\"您的账号在别处登录\"}");
    }));
```

> **核心要点**：`HttpSessionEventPublisher` 是并发控制能工作的**前提**——它让 Spring 监听到 Session 的创建与销毁，从而更新 `SessionRegistry`。漏配它是最常见的"配置了不生效"问题。

### 2.16 主动踢人下线

管理员后台需要主动让某用户下线。通过 `SessionRegistry` 获取并使其会话失效：

```java
@Service
public class SessionService {
    @Autowired
    private SessionRegistry sessionRegistry;

    public void kickOut(String username) {
        // 遍历该用户所有未过期 Principal
        sessionRegistry.getAllSessions(
                new User(username, "", Collections.emptyList()), false)
            .forEach(info -> info.expireNow()); // 标记为过期，下次请求即被踢
    }
}
```

**注入 `SessionRegistry`**（与并发控制共用，框架自动注册）：

```java
http.sessionManagement(s -> s
    .maximumSessions(-1)                       // 不限制并发，仅用 Registry 做管理
    .sessionRegistry(sessionRegistry()));
```

> **联动场景**：修改用户角色后踢其下线，强制重新登录加载新权限——这是 RBAC 系统的常见需求。

---

## 3. 授权（Authorization）

### 3.1 RBAC 模型

RBAC（Role-Based Access Control，基于角色的访问控制）是企业授权的事实标准。

```
用户(User) ——N:M—— 角色(Role) ——N:M—— 权限(Permission/Resource)
   │                  │                      │
   谁是谁             干什么的                能操作什么
```

**三层优势**：

1. 用户与权限解耦：调岗只改角色，不动用户。
2. 权限可复用：一个权限可挂多个角色。
3. 角色可继承：`ROLE_ADMIN` 继承 `ROLE_USER` 的权限。

> **权限标识约定**：角色以 `ROLE_` 前缀（`hasRole()` 自动补前缀），权限用 `资源:操作`（如 `user:read`，用 `hasAuthority()` 判断）。区分二者能避免"管理员"与"读权限"被同一 API 混淆。

### 3.2 权限表设计

经典五表 RBAC 设计（用户、角色、权限 + 两张中间表）：

```sql
-- 用户表
CREATE TABLE sys_user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(100) NOT NULL,
    enabled TINYINT(1) DEFAULT 1
);

-- 角色表
CREATE TABLE sys_role (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) UNIQUE NOT NULL,   -- 如 ADMIN
    code VARCHAR(50) UNIQUE NOT NULL    -- 如 ROLE_ADMIN
);

-- 权限表
CREATE TABLE sys_menu (                  -- 权限以"菜单/资源"形式管理
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,           -- 如 用户管理
    permission VARCHAR(100) NOT NULL,    -- 如 user:read
    type TINYINT(1),                     -- 1菜单 2按钮 3接口
    url VARCHAR(200)                     -- 前端路由或接口路径
);

-- 用户-角色
CREATE TABLE sys_user_role (
    user_id BIGINT, role_id BIGINT,
    PRIMARY KEY (user_id, role_id)
);

-- 角色-权限
CREATE TABLE sys_role_menu (
    role_id BIGINT, menu_id BIGINT,
    PRIMARY KEY (role_id, menu_id)
);
```

> **设计权衡**：权限表加 `type` 字段，把"菜单可见性"与"接口可达性"统一管理——前端菜单与后端接口用同一套权限数据，避免不一致。

### 3.3 编写查询权限方法

根据登录用户名，联表查出其所有角色与权限，供 `UserDetailsService` 装配 `GrantedAuthority`。

```java
// Mapper：一次性查出用户的所有角色 code 与权限 code
<select id="findAuthsByUsername" resultType="string">
    SELECT DISTINCT rm.permission FROM sys_user u
    JOIN sys_user_role ur ON u.id = ur.user_id
    JOIN sys_role r ON ur.role_id = r.id
    JOIN sys_role_menu rrm ON r.id = rrm.role_id
    JOIN sys_menu rm ON rrm.menu_id = rm.id
    WHERE u.username = #{username}
    UNION
    SELECT DISTINCT r.code FROM sys_user u
    JOIN sys_user_role ur ON u.id = ur.user_id
    JOIN sys_role r ON ur.role_id = r.id
    WHERE u.username = #{username}
</select>
```

```java
// Service：装配为 GrantedAuthority
public Collection<? extends GrantedAuthority> loadAuthorities(String username) {
    List<String> auths = userMapper.findAuthsByUsername(username);
    return auths.stream()
            .map(SimpleGrantedAuthority::new) // 角色 ROLE_* 与权限 user:* 统一处理
            .collect(Collectors.toList());
}
```

> **性能优化**：登录时一次查全量权限放入 `SecurityContext`，后续请求**不再查库**。权限变更需配合 [2.16](#216-主动踢人下线) 踢人下线才能即时生效。

### 3.4 配置类设置访问控制

`HttpSecurity` 的 `authorizeHttpRequests` 是**基于 URL 的粗粒度授权**，按顺序匹配。

```java
http.authorizeHttpRequests(a -> a
    .antMatchers("/login", "/css/**", "/js/**").permitAll()  // 公开资源
    .antMatchers("/admin/**").hasRole("ADMIN")                // 需 ROLE_ADMIN
    .antMatchers("/user/**").hasAnyRole("ADMIN", "USER")      // 任一角色
    .antMatchers("/api/**").hasAuthority("user:read")         // 需具体权限
    .antMatchers(HttpMethod.POST, "/order/**").authenticated()// POST 需登录
    .anyRequest().authenticated());                            // 兜底：其余需登录
```

**常用方法**：

| 方法 | 含义 |
|------|------|
| `permitAll()` | 任何人可访问 |
| `denyAll()` | 任何人都不可访问 |
| `authenticated()` | 已登录即可 |
| `hasRole("X")` | 需 `ROLE_X`（自动补前缀） |
| `hasAuthority("x")` | 需具体权限 |
| `hasIpAddress("x")` | IP 限制 |

> **匹配顺序陷阱**：规则按声明顺序短路匹配。**最具体的规则放最前**，`anyRequest()` 必须放最后。

### 3.5 自定义访问控制逻辑

当 `hasRole` / `hasAuthority` 无法表达复杂规则（如"只能访问自己部门的数据"），用自定义 `AccessDecisionVoter` 或 SpEL。

**方式一：SpEL 表达式**（推荐，简洁）：

```java
http.authorizeHttpRequests(a -> a
    .antMatchers("/order/**").access(
        new WebExpressionAuthorizationManager(
            "@orderSecurity.check(authentication, request)"))); // 调用 Bean
```

```java
@Component("orderSecurity")
public class OrderSecurity {
    public boolean check(Authentication auth, HttpServletRequest req) {
        Long userId = ((UserDetails) auth.getPrincipal()).getUsername().hashCode() & 0L; // 演示
        Long orderOwner = Long.parseLong(req.getParameter("ownerId")); // 业务参数
        return userId.equals(orderOwner); // 只能访问自己的订单
    }
}
```

**方式二：自定义 `AuthorizationManager`**（5.8+，替代 Voter）：

```java
public class OwnerAuthorizationManager implements AuthorizationManager<RequestAuthorizationContext> {
    @Override
    public AuthorizationDecision check(Supplier<Authentication> auth, RequestAuthorizationContext ctx) {
        boolean ok = /* 业务判断 */;
        return new AuthorizationDecision(ok); // true=放行
    }
}
```

> **设计要点**：把"复杂授权"封装为安全 Bean，既能被 URL 配置调用，也能被方法注解调用——**授权逻辑只写一处，两处复用**。

### 3.6 注解设置访问控制

注解是**基于方法的细粒度授权**，比 URL 配置更贴近业务。

**开启注解**：

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity // 5.8+（替代旧的 @EnableGlobalMethodSecurity）
public class SecurityConfig {}
```

**三种注解对比**：

```java
@PreAuthorize("hasRole('ADMIN')")                      // 方法执行前校验
@GetMapping("/user/delete")
public void delete(@RequestParam Long id) { /* ... */ }

@PostAuthorize("returnObject.owner == authentication.name") // 执行后校验，false 抛 AccessDenied
@GetMapping("/order/{id}")
public Order get(@PathVariable Long id) { return orderService.get(id); }

@PreAuthorize("hasAuthority('user:write')")
@PostMapping("/user")
public void save(@RequestBody User u) { /* ... */ }
```

**SpEL 常用内置对象**：

| 对象 | 用途 |
|------|------|
| `authentication` | 当前认证对象 |
| `principal` | 当前用户 |
| `hasRole/hasAuthority` | 角色权限判断 |
| `#参数名` | 引用方法参数，如 `#id` |

> **注解 vs URL 配置**：URL 配置保护"入口"，注解保护"业务方法"。**二者叠加才有纵深防御**——避免绕过 Controller 直接调用 Service 的越权。

### 3.7 在前端进行访问控制

前端控制只是**体验优化**，真正授权在后端。后端把用户权限下发前端，前端据此渲染菜单与按钮。

**后端接口**：返回当前用户的菜单树与权限码列表。

```java
@GetMapping("/userInfo")
public Map<String, Object> currentUser() {
    Authentication auth = SecurityContextHolder.getContext().getAuthentication();
    Set<String> perms = auth.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.toSet());           // 权限码集合
    return Map.of("username", auth.getName(), "perms", perms);
}
```

**前端（Vue）指令式控制**：

```javascript
// 注册自定义指令 v-permission
app.directive('permission', {
    mounted(el, binding) {
        const perms = store.getters.perms;          // 全局权限集合
        if (!perms.includes(binding.value)) {
            el.parentNode && el.parentNode.removeChild(el); // 无权限则移除按钮
        }
    }
});
```

```html
<button v-permission="'user:delete'">删除</button> <!-- 无权限则不显示 -->
```

> **安全铁律**：前端控制只防"君子不防小人"——用户可绕过前端直接调接口。**后端的 URL 配置 + 方法注解才是不可绕过的授权**。前端控制仅为提升体验。

### 3.8 403 处理方案

授权失败抛 `AccessDeniedException`，默认显示空白 403 页面。需定制以引导用户。

**方式一：统一异常处理入口**（前后端分离推荐）：

```java
@Component
public class MyAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest req, HttpServletResponse resp,
                       AccessDeniedException ex) {
        resp.setStatus(HttpStatus.FORBIDDEN.value());  // 403
        resp.setContentType("application/json;charset=UTF-8");
        resp.getWriter().write("{\"code\":403,\"msg\":\"权限不足，请联系管理员\"}");
    }
}
```

```java
http.exceptionHandling(e -> e.accessDeniedHandler(myAccessDeniedHandler));
```

**方式二：全局 `@RestControllerAdvice`**（捕获方法注解抛出的异常）：

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(AccessDeniedException.class)
    public Map<String, Object> handle403(AccessDeniedException e) {
        return Map.of("code", 403, "msg", "权限不足：" + e.getMessage());
    }
}
```

**403 vs 401 的语义区分**：

| 状态码 | 含义 | 触发场景 |
|--------|------|----------|
| 401 Unauthorized | 未认证 | 未登录、Token 失效 |
| 403 Forbidden | 已认证但无权限 | 登录了但角色/权限不足 |

> **终极经验**：区分 401/403 能让前端精准引导用户——401 跳登录页，403 弹"权限不足"提示。把两者混为一谈是接口设计的大忌，会让用户体验与可观测性同时降级。

---

## 4. 前后端分离架构

前后端分离（SPA + RESTful API）已是现代 Web 的主流架构。它与传统 MVC 共享同一套认证授权内核，但**交互形态、状态管理、异常处理**完全不同，需要对 Spring Security 的默认配置做系统性改造。

### 4.1 架构设计：传统MVC vs 前后端分离

| 维度 | 传统 MVC（JSP/Thymeleaf） | 前后端分离（Vue/React + REST） |
|------|---------------------------|--------------------------------|
| **页面渲染** | 后端模板引擎渲染 HTML | 前端独立工程渲染，后端只返回 JSON |
| **状态管理** | 有状态，Session 存服务端 | 无状态，Token 存前端（或 Session + Redis） |
| **登录方式** | 表单提交 → 302 重定向 | AJAX 发 JSON → 返回 200 + Token |
| **异常处理** | 跳转错误页面 | 统一返回 `{code, msg}`，前端路由跳转 |
| **跨域** | 同域，无需 CORS | 必配 CORS 或走反向代理 |
| **CSRF** | 默认开启，Token 写表单 | 纯 JWT 可关闭；Cookie 方案仍需开启 |

**架构定位**：Spring Security 从"页面保护者"转变为"API 网关守卫"——只认 Token、只出 JSON、从不重定向。

### 4.2 两种主流认证方案对比

前后端分离下，"身份如何携带"是核心决策。两大流派并存：

| 维度 | 方案A：Session + Redis（中心化） | 方案B：JWT Token（去中心化） |
|------|---------------------------------|------------------------------|
| **身份载体** | `JSESSIONID` Cookie | `Authorization: Bearer <token>` Header |
| **状态存储** | Session 内容存 Redis，服务端有状态 | 信息自包含在 Token 中，服务端无状态 |
| **过期控制** | 服务端主动失效（删 Redis Key） | 依赖 Token 自身 `exp`，难主动吊销（需黑名单） |
| **分布式部署** | 天然支持（共享 Redis） | 天然支持（无需共享存储） |
| **续期机制** | 访问自动续期（Session 刷新） | 需主动实现"双 Token"或"临近过期刷新" |
| **数据载荷** | 可存大量用户信息（查 Redis） | 载荷 ≤ 几 KB，敏感信息不放 |
| **适用场景** | 内部系统、踢人下线要求高、权限变更频繁 | 高并发、微服务跨域、移动端 APP |

> **选型建议**：中小企业项目 **JWT 方案上手更快**（无需 Redis）。企业级后台对"踢人下线""实时权限变更"要求高的，首选 **Session + Redis**。本文以最通用的 **JWT 方案**展开，Session 方案核心差异仅在过滤器中的身份取回方式。

### 4.3 核心组件与职责划分

```
┌───────────────────────────────────────────────────────────────┐
│                        前端（Vue/React）                       │
│  ┌────────────┐   ┌─────────────┐   ┌─────────────────────┐   │
│  │ Login 页面 │→│ Axios 拦截器 │→│ localStorage/Cookie  │   │
│  │ (收集账号) │   │ (带Token)   │   │ (存Token)            │   │
│  └────────────┘   └─────────────┘   └─────────────────────┘   │
└───────────────────────────────┬───────────────────────────────┘
                                │ HTTPS + JSON
                                ▼
┌───────────────────────────────────────────────────────────────┐
│                    Spring Security 过滤链                      │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │ 1. CorsFilter              （跨域预检放行）               │  │
│  │ 2. JwtAuthenticationFilter （从Header解析Token→装上下文） │  │
│  │ 3. UsernamePassword...     （可禁用，走自定义登录接口）  │  │
│  │ 4. AuthorizationFilter     （URL/方法授权判断）          │  │
│  │ 5. ExceptionTranslation    （401/403 → JSON）           │  │
│  └─────────────────────────────────────────────────────────┘  │
└───────────────────────────────┬───────────────────────────────┘
                                ▼
┌───────────────────────────────────────────────────────────────┐
│                       业务层 / Controller                       │
│  ┌──────────────┐   ┌──────────────────┐   ┌───────────────┐  │
│  │ /api/login   │→│ AuthenticationMgr │→│ JwtUtil 签发   │  │
│  │ 登录接口     │   │ （认证账号密码） │   │ Token 返回     │  │
│  └──────────────┘   └──────────────────┘   └───────────────┘  │
└───────────────────────────────────────────────────────────────┘
```

**关键职责切分**：
- **前端只做"呈现"**：输入账号、存 Token、带 Token、根据状态码跳转，**永不做授权判断**。
- **JwtAuthenticationFilter 是桥梁**：把 Token 翻译为 Spring Security 能识别的 `Authentication`，后续授权逻辑与第 3 章完全复用。
- **登录接口自己写**：不再依赖 `formLogin` 的默认过滤器，直接在 Controller 调 `AuthenticationManager` 并签发 JWT。

### 4.4 登录认证交互流程

```
前端 Vue/React                        Spring Boot 后端
     │                                      │
     │ 1. POST /api/login                   │
     │    {username, password}              │
     │─────────────────────────────────────▶│
     │                                      │── 放行：/api/login 在 permitAll
     │                                      │
     │                                      │── AuthController 接收
     │                                      │   ↓
     │                                      │── AuthenticationManager.authenticate()
     │                                      │   → DaoAuthenticationProvider
     │                                      │     → UserDetailsService（查DB）
     │                                      │     → PasswordEncoder.matches()
     │                                      │   ↓
     │                                      │── 成功：JwtUtil.generateToken(user)
     │                                      │   ↓
     │ 2. 200 OK                            │
     │    {code:200, token:"xxx",           │
     │     userInfo:{...}}                  │
     │◀─────────────────────────────────────│
     │                                      │
     │── localStorage.setItem("token", xxx) │
     │── 路由跳转 /dashboard                │
     │                                      │
```

**设计要点**：
1. `/api/login` 必须加入 `permitAll`，否则未登录状态下连登录接口都调不了。
2. 登录接口返回的 Token **只给前端，后端不存**（JWT 去中心化）；如果需要"主动吊销"，后端要配套一个 Redis 黑名单。
3. 失败时**不要 302**，统一返回 `{code:401, msg:"用户名或密码错误"}`，前端根据 code 自行弹窗/跳转。

### 4.5 业务请求授权交互流程

```
前端（携带 Token）                    Spring Boot 后端
     │                                      │
     │ 1. GET /api/order/1                  │
     │    Header: Authorization:            │
     │            Bearer <access_token>     │
     │─────────────────────────────────────▶│
     │                                      │
     │                                      │── JwtAuthenticationFilter
     │                                      │   ① 取 Header → 提取 Token
     │                                      │   ② JwtUtil.parseToken() → 校验签名+过期
     │                                      │   ③ 拿 username 调 UserDetailsService
     │                                      │   ④ 组装 UsernamePasswordAuthenticationToken
     │                                      │   ⑤ SecurityContextHolder.getContext()
     │                                      │      .setAuthentication(auth)
     │                                      │
     │                                      │── AuthorizationFilter
     │                                      │   ① 匹配 URL 规则（hasRole/hasAuthority）
     │                                      │   ② 命中 @PreAuthorize 注解（AOP）
     │                                      │
     │                                      │── Controller 执行业务
     │                                      │
     │ 2. 200 OK {code:200, data:...}       │
     │◀─────────────────────────────────────│
     │                                      │
     │ 异常分支（过滤器/授权任一失败）：        │
     │ 401 {code:401, msg:"Token无效"}      │
     │ 403 {code:403, msg:"权限不足"}        │
     │◀─────────────────────────────────────│
```

**核心洞察**：第 2、3 章学的 `UserDetailsService`、`PasswordEncoder`、`@PreAuthorize`、RBAC 全部复用。前后端分离只改了"身份怎么带过来"（Header Token vs Cookie Session），不改"身份怎么判断、权限怎么校验"。

### 4.6 搭建步骤①：Maven 依赖引入

```xml
<!-- 1. 安全框架（核心） -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<!-- 2. Web：提供 REST Controller -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!-- 3. JWT 签发与解析（jjwt 0.11.5，稳定版） -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
```

> **版本提示**：jjwt 0.12.x 包名迁移到 `io.jsonwebtoken.lang`，代码略有差异。初学者用 0.11.x 资料更丰富，稳定生产用 0.11.5 完全够用。

### 4.7 搭建步骤②：Security 核心配置

这是前后端分离的**中枢配置**，改对了后面一通百通。

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity          // 启用 @PreAuthorize 方法注解
public class SecurityConfig {

    @Autowired private JwtAuthenticationFilter jwtFilter;
    @Autowired private UserDetailsService userDetailsService;
    @Autowired private AccessDeniedHandler accessDeniedHandler;
    @Autowired private AuthenticationEntryPoint authEntryPoint;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // ① 关闭 CSRF：纯 JWT + Bearer Token 模式不受 CSRF 攻击
            .csrf(c -> c.disable())
            // ② 启用 CORS（见 4.12 跨域配置 Bean）
            .cors(c -> {})
            // ③ 会话：STATELESS 完全不创建 Session，请求不带 JSESSIONID
            .sessionManagement(s -> s
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            // ④ URL 放行规则
            .authorizeHttpRequests(a -> a
                .requestMatchers("/api/login",
                                 "/api/captcha",
                                 "/error",
                                 "/doc.html",        // Knife4j/Swagger
                                 "/v3/api-docs/**").permitAll()
                .anyRequest().authenticated())
            // ⑤ 异常处理：未认证/无权限 → 统一返回 JSON
            .exceptionHandling(e -> e
                .authenticationEntryPoint(authEntryPoint)   // 401 入口
                .accessDeniedHandler(accessDeniedHandler))  // 403 处理器
            // ⑥ 禁用默认 formLogin 和 httpBasic（走自定义接口）
            .formLogin(f -> f.disable())
            .httpBasic(h -> h.disable())
            // ⑦ 在 UsernamePasswordAuthenticationFilter 之前插入 JWT 过滤器
            .addFilterBefore(jwtFilter,
                UsernamePasswordAuthenticationFilter.class);
        return http.build();
    }

    /**
     * 认证管理器：登录接口需注入调 authenticate()
     * Spring Boot 2.7+ 不再自动暴露，需手动声明
     */
    @Bean
    public AuthenticationManager authenticationManager(
            HttpSecurity http, PasswordEncoder encoder) throws Exception {
        return http.getSharedObject(AuthenticationManagerBuilder.class)
                .userDetailsService(userDetailsService)
                .passwordEncoder(encoder)
                .and().build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

> **核心三条铁律**（记不住就写在代码注释里）：① `STATELESS`；② 异常 Handler 全 JSON；③ 过滤器顺序 `JwtFilter` 在 `UsernamePassword` 之前。

### 4.8 搭建步骤③：JWT 工具类

JWT 三件事：**生成、解析、校验过期**。统一封装，避免散落各处。

```java
@Component
@ConfigurationProperties(prefix = "jwt")     // 从 yml 读密钥/过期时间
@Data                                        // Lombok，给 secret、expiration 生成 getter/setter
public class JwtUtil {
    private String secret = "my-32-byte-secret-key-at-least-256bit!!!"; // 生产随机生成
    private Long expiration = 7200000L;       // 默认 2 小时（毫秒）

    /** 生成 Token：把 username + 过期时间 + HS256 签名封装 */
    public String generateToken(String username) {
        Date now = new Date();
        Date exp = new Date(now.getTime() + expiration);
        return Jwts.builder()
                .setSubject(username)              // 主题=用户名（唯一标识）
                .setIssuedAt(now)                  // 签发时间
                .setExpiration(exp)                // 过期时间
                .signWith(Keys.hmacShaKeyFor(     // HS256 签名
                            secret.getBytes(StandardCharsets.UTF_8)),
                        SignatureAlgorithm.HS256)
                .compact();
    }

    /** 解析 Token：拿到 username（校验失败抛 SignatureException/ExpiredJwtException） */
    public String parseUsername(String token) {
        return Jwts.parserBuilder()
                .setSigningKey(Keys.hmacShaKeyFor(
                        secret.getBytes(StandardCharsets.UTF_8)))
                .build()
                .parseClaimsJws(token)     // 校验签名 + 过期，失败直接抛
                .getBody()
                .getSubject();
    }

    /** 只校验不抛：给过滤器用，过期直接返回 false */
    public boolean validateToken(String token) {
        try { parseUsername(token); return true; }
        catch (JwtException | IllegalArgumentException e) { return false; }
    }
}
```

对应 `application.yml`：

```yaml
jwt:
  secret: "生产环境请用32字节以上随机字符串,建议从环境变量读取"
  expiration: 7200000   # 2小时
```

> **安全提示**：`secret` 不要写死在代码里，生产用 `${JWT_SECRET}` 从环境变量/配置中心/Nacos 读取。密钥泄露 = 任意用户可伪造 Token。

### 4.9 搭建步骤④：JWT 认证过滤器

这是前后端分离的**核心桥梁**——每个业务请求必经，把 Token 翻译成 `SecurityContext`。

```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    // OncePerRequestFilter：保证一次请求只过一次（避免 Forward/Include 重复执行）

    @Autowired private JwtUtil jwtUtil;
    @Autowired private UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest req,
                                    HttpServletResponse resp,
                                    FilterChain chain)
                            throws ServletException, IOException {
        // ① 从 Header 取 Token
        String header = req.getHeader("Authorization");
        if (header != null && header.startsWith("Bearer ")) {
            String token = header.substring(7);              // 去掉 "Bearer " 前缀
            if (jwtUtil.validateToken(token)) {
                String username = jwtUtil.parseUsername(token);
                // ② 查用户与权限（实际生产可缓存到 Redis，避免每次查库）
                UserDetails user = userDetailsService.loadUserByUsername(username);
                // ③ 组装 Authentication：principal=userDetails, credentials=null, authorities
                UsernamePasswordAuthenticationToken auth =
                    new UsernamePasswordAuthenticationToken(
                            user, null, user.getAuthorities());
                auth.setDetails(new WebAuthenticationDetailsSource()
                        .buildDetails(req));                // 带 IP/Session 等详情
                // ④ 写入上下文：后续 AuthorizationFilter 从这里取
                SecurityContextHolder.getContext().setAuthentication(auth);
            }
        }
        // ⑤ 无论是否登录成功，都要继续走后面的过滤器链
        chain.doFilter(req, resp);
    }
}
```

**设计细节**：
- Token 为空/无效 → 不设上下文，后面 `AuthorizationFilter` 配合 `anyRequest().authenticated()` 会抛 401，交给 `AuthenticationEntryPoint` 出 JSON。
- **不要在过滤器里写 JSON 响应**（除了登出这种特殊场景），异常由统一 Handler 出，职责清晰。
- 生产环境把 `loadUserByUsername` 结果缓存 10~30 分钟，减少 DB 压力（配合 4.14 的踢人方案）。

### 4.10 搭建步骤⑤：登录接口实现

脱离默认 `formLogin`，**自己写 Controller 调认证管理器**，签发 JWT 返回。

```java
@RestController
@RequestMapping("/api")
public class AuthController {

    @Autowired private AuthenticationManager authManager;
    @Autowired private JwtUtil jwtUtil;

    /**
     * 登录入参 DTO（建议单独建 domain/vo 包）
     * @Data public class LoginDTO { private String username; private String password; }
     */
    @PostMapping("/login")
    public Map<String, Object> login(@RequestBody LoginDTO dto) {
        // ① 组装未认证 Token：principal=用户名，credentials=明文密码
        Authentication token =
            new UsernamePasswordAuthenticationToken(
                    dto.getUsername(), dto.getPassword());
        // ② 交给管理器：内部调 UserDetailsService + PasswordEncoder
        Authentication auth = authManager.authenticate(token); // 失败抛 BadCredentialsException
        // ③ 认证成功，签发 JWT
        String jwt = jwtUtil.generateToken(auth.getName());
        // ④ 组装返回：Token + 用户简要信息（前端展示用）
        UserDetails ud = (UserDetails) auth.getPrincipal();
        Set<String> perms = ud.getAuthorities().stream()
                .map(GrantedAuthority::getAuthority).collect(Collectors.toSet());
        return Map.of(
            "code", 200,
            "msg", "登录成功",
            "token", jwt,
            "userInfo", Map.of(
                "username", ud.getUsername(),
                "permissions", perms
            )
        );
    }
}
```

> **职责边界**：`AuthenticationManager.authenticate()` 内部会**自动**调你在 2.3/2.4 节写的 `UserDetailsService` 和 `PasswordEncoder`，无需自己查库比密码。这就是 Spring Security "认证内核复用"的价值。

### 4.11 搭建步骤⑥：统一异常与认证入口

前后端分离**绝对不能让框架默认的 302 跳转**出现，所有异常必须统一出 JSON。

```java
/**
 * 401 未认证入口：Token 为空/过期/非法，或访问受保护资源未登录
 */
@Component
public class JsonAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest req, HttpServletResponse resp,
                         AuthenticationException ex) throws IOException {
        resp.setStatus(HttpStatus.UNAUTHORIZED.value());   // HTTP 401
        resp.setContentType("application/json;charset=UTF-8");
        String msg = (ex instanceof BadCredentialsException)
                ? "用户名或密码错误" : "未登录或Token已过期";
        resp.getWriter().write(
            new ObjectMapper().writeValueAsString(
                Map.of("code", 401, "msg", msg)));
    }
}

/**
 * 403 已认证但无权限：见 3.8 节，前后端分离下实现一致
 */
@Component
public class JsonAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest req, HttpServletResponse resp,
                       AccessDeniedException ex) throws IOException {
        resp.setStatus(HttpStatus.FORBIDDEN.value());      // HTTP 403
        resp.setContentType("application/json;charset=UTF-8");
        resp.getWriter().write(
            new ObjectMapper().writeValueAsString(
                Map.of("code", 403, "msg", "权限不足")));
    }
}
```

> **与 3.8 节 `@RestControllerAdvice` 的关系**：EntryPoint/Handler 抓的是**过滤器层**抛出的异常（还没到 Controller）；`@RestControllerAdvice` 抓的是 **Controller/Service 层**抛出的异常。两者缺一不可，必须同时配。

### 4.12 搭建步骤⑦：跨域 CORS 配置

前端跑在 `localhost:5173`（Vite），后端跑在 `localhost:8080`，**同源策略**会阻止 AJAX，必须开 CORS。

```java
@Configuration
public class CorsConfig {
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration cfg = new CorsConfiguration();
        // ① 允许的前端地址：生产精确指定域名，不要写 *
        cfg.setAllowedOriginPatterns(List.of(
                "http://localhost:5173",      // Vue3 Vite
                "http://localhost:3000",      // React
                "https://admin.example.com"   // 生产域名
        ));
        cfg.setAllowedMethods(List.of("GET","POST","PUT","DELETE","OPTIONS"));
        cfg.setAllowedHeaders(List.of("*"));        // 放行所有 Header（含 Authorization）
        cfg.setAllowCredentials(true);              // 允许带 Cookie（Session 方案必选）
        cfg.setMaxAge(3600L);                       // 预检 OPTIONS 缓存 1 小时
        UrlBasedCorsConfigurationSource src = new UrlBasedCorsConfigurationSource();
        src.registerCorsConfiguration("/**", cfg);  // 所有路径生效
        return src;
    }
}
```

> **与 Security 的配合**：`SecurityConfig` 里写了 `.cors(c -> {})`，框架会自动找上面这个 `CorsConfigurationSource` Bean，无需再写 WebMvcConfigurer 的 `addCorsMappings`——两边都写会冲突。

### 4.13 前端联调要点

以前端 Vue3 + Axios 为例，给出最小可用模板。

**① Axios 封装（request.ts）**：

```typescript
import axios from 'axios'
import router from '@/router'

const req = axios.create({
  baseURL: '/api',                         // 走 Vite 代理 → 后端 8080
  timeout: 10000
})

// 请求拦截器：自动带 Token
req.interceptors.request.use(config => {
  const token = localStorage.getItem('token')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

// 响应拦截器：统一处理 401/403
req.interceptors.response.use(
  res => res.data,                          // 直接解出后端的 {code,msg,data}
  err => {
    const status = err.response?.status
    if (status === 401) {
      localStorage.removeItem('token')
      router.push('/login')                 // 401 跳登录页
    } else if (status === 403) {
      alert(err.response.data.msg || '权限不足')  // 403 弹提示
    }
    return Promise.reject(err)
  }
)
export default req
```

**② Vite 代理（vite.config.ts）**：开发期避跨域，前后端同域感受。

```typescript
export default defineConfig({
  server: {
    port: 5173,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',    // 转发到后端
        changeOrigin: true,
        rewrite: (p) => p.replace(/^\/api/, '') // 如需去掉 /api 前缀
      }
    }
  }
})
```

**③ 登录页调用**：

```typescript
import req from '@/utils/request'

// 登录
async function handleLogin() {
  const res: any = await req.post('/api/login', {
    username: form.username,
    password: form.password
  })
  if (res.code === 200) {
    localStorage.setItem('token', res.token)    // 存 Token
    localStorage.setItem('perms', JSON.stringify(res.userInfo.permissions))
    router.push('/dashboard')
  } else {
    alert(res.msg)
  }
}
```

> **Token 存储选型**：`localStorage` 简单但易受 XSS 攻击窃取；**更安全**是 `Cookie + HttpOnly + Secure`（服务端登录成功下发 `Set-Cookie`，前端读不到，浏览器自动带）。后者需要配合后端配置 `CookieCsrfTokenRepository` 防 CSRF，复杂度高但抗 XSS。银行/金融类系统必须用 Cookie HttpOnly。

### 4.14 常见陷阱与最佳实践

**陷阱 1：双 SecurityFilterChain 并存冲突**
```
错误做法：项目已经有框架内置安全配置（如 iPlat4J、Ruoyi），自己再加一个 SecurityFilterChain Bean。
后果：启动报错 "Found WebSecurityConfigurerAdapter as well as SecurityFilterChain" 或过滤器重复。
正确做法：在框架已有的配置扩展点上改（白名单/属性开关），不叠加第二套安全入口。
```

**陷阱 2：登录接口没放 permitAll**
```
表现：调 /api/login 返回 401，以为是"账号密码错"，实际是"未登录不让调登录接口"。
排查：SecurityConfig.authorizeHttpRequests 里确认 /api/login 在 permitAll 列表且位置在 anyRequest 之前。
```

**陷阱 3：过滤器没继续 chain.doFilter**
```
表现：请求发出去浏览器一直 pending。
原因：JwtAuthenticationFilter 分支逻辑忘记写 chain.doFilter(req, resp)。
原则：所有自定义 Filter 末尾必须无条件继续下一个过滤器（除非明确要拦截返回）。
```

**最佳实践 A：JWT 主动吊销 + 踢人下线**
JWT 默认不能主动失效。对"改密码""管理员踢人"场景，加**Redis 黑名单**：
```java
// 登出/踢人时：把 token 剩余有效期写入 Redis 黑名单
redisTemplate.opsForValue().set("blacklist:" + tokenId, "1",
        remainingSeconds, TimeUnit.SECONDS);
// JwtAuthenticationFilter 校验前先查：
if (Boolean.TRUE.equals(redisTemplate.hasKey("blacklist:" + tokenId))) {
    // 返回 401
}
```

**最佳实践 B：双 Token 续期方案**
单 Token 2 小时过期用户体验差。改进：
- `access_token`：有效期 2 小时，放 Header
- `refresh_token`：有效期 7 天，放 HttpOnly Cookie
- 前端拦截器发现 `access_token` 即将过期（剩余 < 5 分钟），用 `refresh_token` 静默换发新 access_token
- `refresh_token` 被盗风险可控：仅用于换发接口，且服务端可一次性失效

**最佳实践 C：生产环境的三道防线**
1. **HTTPS 全链路**：Token 在明文中传输与裸奔无异，必须 HTTPS。
2. **网关限流**：对 `/api/login` 做 IP 限流（如 1 分钟 5 次），防暴力破解。
3. **审计日志**：登录成功/失败、登出、403 越权尝试，写入审计表供事后追溯。

---

## 附录：认证授权全链路速查图

```
请求 → FilterChain → SecurityContextPersistence（恢复上下文）
                   → UsernamePasswordAuthenticationFilter（表单认证）
                        │ 调 AuthenticationManager
                        │   → DaoAuthenticationProvider
                        │       → UserDetailsService.loadUserByUsername（查 DB）
                        │       → PasswordEncoder.matches（校验密码）
                        │ 成功 → SecurityContext 存 Authentication
                        │        → SuccessHandler（跳转/JSON/JWT 签发）
                        │ 失败 → FailureHandler（返回错误信息）
                   → RememberMeAuthenticationFilter（记住我）
                   → AuthorizationFilter（URL 授权）
                        │ 失败 → AccessDeniedHandler（403）
                   → 方法注解 @PreAuthorize（细粒度授权）
                   → Controller → 返回响应
```

> **一句话总结**：Spring Security = 一条 Filter 链 + 认证（UserDetailsService + PasswordEncoder）+ 授权（URL 配置 + 方法注解）+ 会话（Session + RememberMe）。把这四块吃透，再复杂的业务安全场景都能拆解落地。

---

*文档结束。如需补充 OAuth2、JWT、OAuth2 Resource Server 等进阶主题，可在第 2/3 章基础上扩展为第 4 章。*
