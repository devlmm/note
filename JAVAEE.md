# JavaEE 知识点完整目录（由浅入深，环环相扣）

**阅读指南**：由浅入深，每个知识点均配有精简示例。建议按顺序阅读，环环相扣。

## 📑 目录

- [第一部分：JDBC（Java 数据库连接）](#第一部分jdbcjava-数据库连接)
  - [1.1 JDBC 典型编程步骤](#11-jdbc-典型编程步骤)
  - [1.2 DriverManager 与 DataSource 对比](#12-drivermanager-与-datasource-对比)
  - [1.3 PreparedStatement 防止 SQL 注入](#13-preparedstatement-防止-sql-注入)
  - [1.4 事务管理](#14-事务管理)
  - [1.5 批量操作](#15-批量操作)
  - [1.6 ResultSet 处理与 POJO 映射](#16-resultset-处理与-pojo-映射)
  - [1.7 SQLException 处理建议](#17-sqlexception-处理建议)
- [第二部分：Servlet（Http 服务请求响应处理规范）](#第二部分servlethttp-服务请求响应处理规范)
  - [2.1 JavaEE 简介](#21-javaee-简介)
  - [2.2 Servlet 简介](#22-servlet-简介)
  - [2.3 第一个 Servlet](#23-第一个-servlet)
  - [2.4 Servlet 继承结构](#24-servlet-继承结构)
  - [2.5 Servlet 生命周期](#25-servlet-生命周期)
  - [2.6 Tomcat 处理请求流程](#26-tomcat-处理请求流程)
  - [2.7 Servlet 作用](#27-servlet-作用)
  - [2.8 IDEA 创建 Web 工程](#28-idea-创建-web-工程)
  - [2.9 请求处理（HttpServletRequest）](#29-请求处理httpservletrequest)
    - [2.9.1 获取请求信息](#291-获取请求信息)
    - [2.9.2 获取表单数据](#292-获取表单数据)
    - [2.9.3 设置请求编码](#293-设置请求编码)
    - [2.9.4 资源访问路径](#294-资源访问路径)
    - [2.9.5 HttpServletRequest 生命周期](#295-httpservletrequest-生命周期)
  - [2.10 响应处理（HttpServletResponse）](#210-响应处理httpservletresponse)
    - [2.10.1 设置响应类型](#2101-设置响应类型)
    - [2.10.2 字符响应](#2102-字符响应)
    - [2.10.3 字节响应](#2103-字节响应)
    - [2.10.4 设置响应编码](#2104-设置响应编码)
    - [2.10.5 重定向响应](#2105-重定向响应)
    - [2.10.6 请求转发 vs 重定向](#2106-请求转发-vs-重定向)
    - [2.10.7 文件下载](#2107-文件下载)
  - [2.11 会话管理（Cookie & Session）](#211-会话管理cookie--session)
    - [2.11.1 Cookie 对象](#2111-cookie-对象)
    - [2.11.2 HttpSession 对象](#2112-httpsession-对象)
    - [2.11.3 Cookie vs Session 对比](#2113-cookie-vs-session-对比)
    - [2.11.4 会话维持流程](#2114-会话维持流程)
  - [2.12 Servlet 高级特性](#212-servlet-高级特性)
    - [2.12.1 ServletContext 对象](#2121-servletcontext-对象)
    - [2.12.2 ServletConfig 对象](#2122-servletconfig-对象)
    - [2.12.3 ServletContext vs ServletConfig](#2123-servletcontext-vs-servletconfig)
    - [2.12.4 自启动 Servlet](#2124-自启动-servlet)
    - [2.12.5 Servlet 线程安全问题](#2125-servlet-线程安全问题)
    - [2.12.6 URL Pattern 配置](#2126-url-pattern-配置)
    - [2.12.7 文件上传](#2127-文件上传)
  - [2.13 Filter 过滤器](#213-filter-过滤器)
    - [2.13.1 Filter 简介](#2131-filter-简介)
    - [2.13.2 Filter 实现](#2132-filter-实现)
    - [2.13.3 Filter 生命周期](#2133-filter-生命周期)
    - [2.13.4 FilterConfig 对象](#2134-filterconfig-对象)
    - [2.13.5 FilterChain（过滤器链）](#2135-filterchain过滤器链)
    - [2.13.6 Filter url-pattern 配置](#2136-filter-url-pattern-配置)
  - [2.14 Listener 监听器](#214-listener-监听器)
    - [2.14.1 Listener 简介](#2141-listener-简介)
    - [2.14.2 监听器分类](#2142-监听器分类)
    - [2.14.3 ServletContext 监听器](#2143-servletcontext-监听器)
    - [2.14.4 HttpSession 监听器](#2144-httpsession-监听器)
    - [2.14.5 HttpServletRequest 监听器](#2145-httpservletrequest-监听器)
    - [2.14.6 监听器速查表](#2146-监听器速查表)
  - [附录：关键知识点归纳](#附录关键知识点归纳)

---

## 第一部分：JDBC（Java 数据库连接）

### 1.1 JDBC 典型编程步骤

```java
// 1. 加载驱动（JDBC 4.0+ 可省略，SPI 自动加载）
Class.forName("com.mysql.cj.jdbc.Driver");

// 2. 获取 Connection
String url = "jdbc:mysql://localhost:3306/db?useSSL=false&serverTimezone=UTC";
Connection conn = DriverManager.getConnection(url, "user", "password");

// 3. 创建 Statement/PreparedStatement
PreparedStatement ps = conn.prepareStatement("SELECT * FROM users WHERE id = ?");

// 4. 设置参数并执行
ps.setInt(1, 100);  // 参数索引从 1 开始
ResultSet rs = ps.executeQuery();

// 5. 处理 ResultSet
while (rs.next()) {
    String name = rs.getString("name");
    System.out.println(name);
}

// 6. 关闭资源（倒序关闭，建议 try-with-resources）
rs.close(); ps.close(); conn.close();
```

### 1.2 DriverManager 与 DataSource 对比

| 特性           | DriverManager                  | DataSource                     |
| -------------- | ------------------------------ | ------------------------------ |
| 连接管理       | 每次创建新连接                 | 连接池，复用连接               |
| 性能           | 低（频繁创建/销毁）            | 高（连接复用）                 |
| 生产环境       | 不推荐                         | **推荐**（HikariCP、Druid）    |
| 事务支持       | 手动管理                       | 支持分布式事务（JTA）          |

```java
// DriverManager — 简单场景
Connection conn = DriverManager.getConnection(url, user, pwd);

// DataSource — 生产环境（HikariCP）
HikariConfig config = new HikariConfig();
config.setJdbcUrl(url); config.setUsername(user); config.setPassword(pwd);
DataSource ds = new HikariDataSource(config);
Connection conn = ds.getConnection();  // 从连接池获取
```

### 1.3 PreparedStatement 防止 SQL 注入

```java
// ❌ Statement — SQL 注入风险
String name = "' OR '1'='1";
Statement stmt = conn.createStatement();
stmt.executeQuery("SELECT * FROM users WHERE name = '" + name + "'");  // 危险！

// ✅ PreparedStatement — 参数化查询，自动转义
PreparedStatement ps = conn.prepareStatement("SELECT * FROM users WHERE name = ?");
ps.setString(1, name);  // 安全，参数不会被解释为 SQL 语法
ResultSet rs = ps.executeQuery();
```

### 1.4 事务管理

```java
// 关闭自动提交，开启事务
conn.setAutoCommit(false);

try {
    // 执行多个 SQL
    conn.createStatement().executeUpdate("UPDATE account SET balance = balance - 100 WHERE id = 1");
    conn.createStatement().executeUpdate("UPDATE account SET balance = balance + 100 WHERE id = 2");

    conn.commit();  // 提交事务
} catch (SQLException e) {
    conn.rollback();  // 回滚事务
    throw e;
} finally {
    conn.setAutoCommit(true);  // 恢复自动提交
}

// 设置隔离级别（防止脏读/不可重复读/幻读）
conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);  // 读已提交
// 级别：READ_UNCOMMITTED（脏读）、READ_COMMITTED（防脏读）、REPEATABLE_READ（防不可重复读）、SERIALIZABLE（防幻读）
```

### 1.5 批量操作

```java
PreparedStatement ps = conn.prepareStatement("INSERT INTO users(name, age) VALUES(?, ?)");

// 禁用自动提交，提升性能
conn.setAutoCommit(false);

for (int i = 0; i < 1000; i++) {
    ps.setString(1, "User" + i);
    ps.setInt(2, 20 + i % 30);
    ps.addBatch();  // 添加到批次

    // 每 500 条执行一次，避免内存溢出
    if (i % 500 == 0) {
        ps.executeBatch();  // 执行批次
        conn.commit();      // 提交
    }
}

ps.executeBatch();  // 执行剩余
conn.commit();
conn.setAutoCommit(true);
```

### 1.6 ResultSet 处理与 POJO 映射

```java
// POJO 类
class User {
    private int id;
    private String name;
    // getter/setter...
}

// 手动映射
PreparedStatement ps = conn.prepareStatement("SELECT id, name FROM users WHERE id = ?");
ps.setInt(1, 100);
ResultSet rs = ps.executeQuery();

User user = null;
if (rs.next()) {
    user = new User();
    user.setId(rs.getInt("id"));
    user.setName(rs.getString("name"));
}

// ResultSetMetaData 动态映射（通用查询）
ResultSetMetaData meta = rs.getMetaData();
int colCount = meta.getColumnCount();
while (rs.next()) {
    for (int i = 1; i <= colCount; i++) {
        String colName = meta.getColumnName(i);
        Object value = rs.getObject(i);
        System.out.println(colName + ": " + value);
    }
}
```

### 1.7 SQLException 处理建议

```java
try {
    conn.createStatement().execute("INSERT INTO users(name) VALUES('Tom')");
} catch (SQLException e) {
    // 记录关键信息用于排查
    System.err.println("SQLState: " + e.getSQLState());    // SQL 状态码（如 23000 唯一约束冲突）
    System.err.println("Error Code: " + e.getErrorCode()); // 数据库特定错误码
    System.err.println("Message: " + e.getMessage());

    // 链式异常（多个异常）
    while (e != null) {
        System.err.println("→ " + e.getMessage());
        e = e.getNextException();
    }

    // 根据错误码处理
    if (e.getErrorCode() == 1062) {  // MySQL 唯一约束冲突
        System.out.println("记录已存在");
    }
}
```

---

## 第二部分：Servlet（Http 服务请求响应处理规范）

### 2.1 JavaEE 简介

JavaEE（Java Enterprise Edition）是企业级 Java 开发平台，核心组件包括：

| 组件 | 说明 |
|------|------|
| Servlet | Web 请求处理 |
| JSP | 动态页面 |
| JDBC | 数据库连接 |
| EJB | 企业级组件 |
| JNDI | 命名服务 |

### 2.2 Servlet 简介

**Servlet**：运行在服务器端的 Java 程序，用于处理 HTTP 请求并生成响应。

```
浏览器 → HTTP请求 → Tomcat → Servlet → 处理 → HTTP响应 → 浏览器
```

**核心特点**：
- 运行在容器中（Tomcat/Jetty）
- 单实例多线程（效率高）
- 生命周期由容器管理

### 2.3 第一个 Servlet

#### 2.3.1 编写 Servlet

```java
@WebServlet("/hello")  // 注解方式，URL映射
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) 
            throws ServletException, IOException {
        resp.setContentType("text/html;charset=UTF-8");  // 设置响应类型
        resp.getWriter().write("Hello Servlet!");         // 输出响应内容
    }
}
```

#### 2.3.2 传统配置方式（web.xml）

```xml
<!-- web.xml -->
<servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>com.example.HelloServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/hello</url-pattern>  <!-- URL映射 -->
</servlet-mapping>
```

📐 **推荐**：使用 `@WebServlet` 注解，简洁高效

### 2.4 Servlet 继承结构

```
Servlet（接口）
    ↑
GenericServlet（抽象类）- 通用的Servlet实现
    ↑
HttpServlet（抽象类）- HTTP协议专用
    ↑
自定义Servlet（继承HttpServlet）
```

| 类/接口 | 说明 |
|---------|------|
| `Servlet` | 接口，定义生命周期方法 |
| `GenericServlet` | 抽象类，实现通用逻辑 |
| `HttpServlet` | 抽象类，区分 GET/POST 等请求 |
| `自定义Servlet` | 继承 HttpServlet，重写 doGet/doPost |

### 2.5 Servlet 生命周期

```
加载 → 实例化 → 初始化 → 服务 → 销毁
```

| 阶段 | 方法 | 调用时机 |
|------|------|----------|
| 加载 | — | 容器启动或首次请求 |
| 初始化 | `init()` | 实例化后调用一次 |
| 服务 | `service()` → `doGet/doPost` | 每次请求 |
| 销毁 | `destroy()` | 容器关闭 |

```java
@Override
public void init() throws ServletException {
    // 初始化代码（只执行一次）
}

@Override
public void destroy() {
    // 清理资源（容器关闭时）
}
```

📐 **核心**：init 只执行一次，service 每次请求执行

### 2.6 Tomcat 处理请求流程

```
1. 浏览器发送HTTP请求
2. Tomcat接收请求，解析请求信息
3. Tomcat根据URL找到对应Servlet
4. Tomcat调用Servlet的service()方法
5. Servlet处理请求，生成响应
6. Tomcat将响应返回给浏览器
```

### 2.7 Servlet 作用

- **接收请求**：获取请求参数、请求头
- **处理业务**：调用业务逻辑
- **生成响应**：返回 HTML/JSON/文件
- **会话管理**：Cookie/Session

### 2.8 IDEA 创建 Web 工程

```
1. New Project → Java → Web Application
2. 添加 Tomcat 依赖
3. 创建 Servlet 类（继承 HttpServlet）
4. 配置 @WebServlet 注解或 web.xml
5. Run → 配置 Tomcat Server
```

### 2.9 请求处理（HttpServletRequest）

#### 2.9.1 获取请求信息

##### 2.9.1.1 基本信息

```java
String method = req.getMethod();          // GET/POST
String uri = req.getRequestURI();         // /project/hello
String url = req.getRequestURL().toString(); // 完整URL
String ip = req.getRemoteAddr();          // 客户端IP
String context = req.getContextPath();    // 项目路径 /project
```

##### 2.9.1.2 获取请求头

```java
String agent = req.getHeader("User-Agent");   // 浏览器信息
String referer = req.getHeader("Referer");    // 来源页面
Enumeration<String> headers = req.getHeaderNames(); // 所有头名称
```

#### 2.9.2 获取表单数据

##### 2.9.2.1 基本方式

```java
String name = req.getParameter("username");   // 单值参数
String pwd = req.getParameter("password");
```

##### 2.9.2.2 多值参数（复选框）

```java
String[] hobbies = req.getParameterValues("hobby");  // 多选值
```

##### 2.9.2.3 获取所有参数名

```java
Enumeration<String> names = req.getParameterNames();  // 所有Key
```

##### 2.9.2.4 Map 方式获取

```java
Map<String, String[]> map = req.getParameterMap();   // Key-Value数组
for (String key : map.keySet()) {
    String[] values = map.get(key);
}
```

#### 2.9.3 设置请求编码

```java
req.setCharacterEncoding("UTF-8");  // POST请求编码设置
```

⚠️ **注意**：必须在 `getParameter()` 之前调用，仅对 POST 有效

GET 请求编码需在 Tomcat 配置或手动转码：

```java
String name = new String(req.getParameter("name").getBytes("ISO-8859-1"), "UTF-8");
```

#### 2.9.4 资源访问路径

| 路径类型 | 说明 | 示例 |
|----------|------|------|
| 绝对路径 | 从根开始 | `/project/page.jsp` |
| 相对路径 | 从当前路径 | `../page.jsp` |
| 项目路径 | 加 contextPath | `${pageContext.request.contextPath}/page.jsp` |

```java
String path = req.getContextPath();  // 获取项目路径
String realPath = req.getServletContext().getRealPath("/file.txt"); // 物理路径
```

#### 2.9.5 HttpServletRequest 生命周期

- **创建**：每次请求到达时创建
- **销毁**：响应结束后销毁
- **作用域**：仅在一次请求内有效

### 2.10 响应处理（HttpServletResponse）

#### 2.10.1 设置响应类型

```java
resp.setContentType("text/html;charset=UTF-8");   // HTML响应
resp.setContentType("application/json;charset=UTF-8"); // JSON响应
resp.setContentType("image/jpeg");                // 图片响应
```

#### 2.10.2 字符响应

```java
PrintWriter out = resp.getWriter();  // 获取字符输出流
out.write("Hello World");            // 输出内容
out.flush();                         // 刷新缓冲区
```

#### 2.10.3 字节响应

```java
ServletOutputStream out = resp.getOutputStream();  // 字节输出流
byte[] data = Files.readAllBytes(Paths.get("image.jpg"));
out.write(data);  // 输出二进制数据
```

⚠️ **注意**：`getWriter()` 和 `getOutputStream()` 不能同时使用

#### 2.10.4 设置响应编码

```java
resp.setCharacterEncoding("UTF-8");       // 设置编码
resp.setContentType("text/html;charset=UTF-8");  // 推荐，同时设置类型和编码
```

#### 2.10.5 重定向响应

```java
resp.sendRedirect("/project/login.jsp");  // 重定向到新地址
```

**重定向特点**：
- 客户端行为（浏览器跳转）
- URL 会改变
- 两次请求（request 数据丢失）
- 状态码 302

##### 2.10.5.1 重定向案例

```java
if (loginSuccess) {
    resp.sendRedirect(req.getContextPath() + "/home.jsp");  // 登录成功跳转
} else {
    resp.sendRedirect(req.getContextPath() + "/login.jsp?error=1");  // 失败带参数
}
```

#### 2.10.6 请求转发 vs 重定向

| 特性 | 请求转发 | 重定向 |
|------|----------|--------|
| 行为 | 服务端 | 客户端 |
| URL | 不变 | 变化 |
| 请求次数 | 1次 | 2次 |
| request 数据 | 保留 | 丢失 |
| 方法 | `req.getRequestDispatcher().forward()` | `resp.sendRedirect()` |

```java
// 请求转发
req.getRequestDispatcher("/page.jsp").forward(req, resp);

// 重定向
resp.sendRedirect("/page.jsp");
```

#### 2.10.7 文件下载

```java
resp.setContentType("application/octet-stream");  // 二进制流类型
resp.setHeader("Content-Disposition", "attachment;filename=" + filename);

ServletOutputStream out = resp.getOutputStream();
out.write(Files.readAllBytes(Paths.get(filePath)));  // 输出文件内容
```

##### 2.10.7.1 解决中文文件名乱码

```java
String filename = URLEncoder.encode("中文文件.txt", "UTF-8");  // URL编码
resp.setHeader("Content-Disposition", "attachment;filename=" + filename);
```

### 2.11 会话管理（Cookie & Session）

#### 2.11.1 Cookie 对象

##### 2.11.1.1 Cookie 特点

- 存储在**客户端**（浏览器）
- 数据量小（≤4KB）
- 可设置过期时间
- 每次请求自动携带

##### 2.11.1.2 创建 Cookie

```java
Cookie cookie = new Cookie("username", "admin");  // 创建
cookie.setMaxAge(3600);           // 存活时间（秒），-1=浏览器关闭删除
cookie.setPath("/project");       // 有效路径
resp.addCookie(cookie);           // 发送给客户端
```

##### 2.11.1.3 获取 Cookie

```java
Cookie[] cookies = req.getCookies();  // 获取所有Cookie
for (Cookie c : cookies) {
    if ("username".equals(c.getName())) {
        String value = c.getValue();  // 获取值
    }
}
```

##### 2.11.1.4 解决 Cookie 中文问题

```java
// 存储（URL编码）
Cookie cookie = new Cookie("name", URLEncoder.encode("张三", "UTF-8"));

// 获取（URL解码）
String value = URLDecoder.decode(cookie.getValue(), "UTF-8");
```

##### 2.11.1.5 Cookie 跨域问题

```java
cookie.setDomain(".example.com");  // 设置域名，允许子域共享
```

##### 2.11.1.6 状态 Cookie vs 持久化 Cookie

| 类型 | MaxAge | 特点 |
|------|--------|------|
| 状态 Cookie | -1 或未设置 | 浏览器关闭后删除 |
| 持久化 Cookie | > 0 | 存储到硬盘，过期后删除 |

#### 2.11.2 HttpSession 对象

##### 2.11.2.1 Session 特点

- 存储在**服务端**
- 数据量大（无限制）
- 默认30分钟过期
- 通过 Cookie 中的 JSESSIONID 关联

##### 2.11.2.2 创建 Session

```java
HttpSession session = req.getSession();      // 获取/创建Session
HttpSession session = req.getSession(false); // 仅获取，不创建
```

##### 2.11.2.3 Session 使用

```java
HttpSession session = req.getSession();
session.setAttribute("user", userObj);        // 存储数据
User user = (User) session.getAttribute("user"); // 获取数据
session.removeAttribute("user");              // 删除数据
```

##### 2.11.2.4 Session 销毁方式

```java
session.invalidate();  // 手动销毁

// web.xml 配置超时（分钟）
<session-config>
    <session-timeout>30</session-timeout>
</session-config>
```

##### 2.11.2.5 Session 生命周期

| 阶段 | 说明 |
|------|------|
| 创建 | 首次调用 `getSession()` |
| 有效 | 默认30分钟无操作过期 |
| 销毁 | `invalidate()` 或超时或服务器关闭 |

#### 2.11.3 Cookie vs Session 对比

| 特性 | Cookie | Session |
|------|--------|---------|
| 存储位置 | 客户端 | 服务端 |
| 数据量 | ≤4KB | 无限制 |
| 安全性 | 较低（可篡改） | 较高 |
| 性能 | 每次请求携带 | 服务端内存占用 |
| 适用场景 | 登录状态、个性化 | 敏感数据、购物车 |

#### 2.11.4 会话维持流程

```
1. 首次请求 → 创建Session → JSESSIONID写入Cookie
2. 后续请求 → Cookie携带JSESSIONID → 服务端找到对应Session
3. Session过期 → JSESSIONID失效 → 重新创建Session
```

### 2.12 Servlet 高级特性

#### 2.12.1 ServletContext 对象

##### 2.12.1.1 简介

ServletContext 是**全局上下文对象**，一个 Web 应用只有一个，所有 Servlet 共享。

| 特性 | 说明 |
|------|------|
| 作用域 | 全局（整个应用） |
| 生命周期 | 应用启动到关闭 |
| 用途 | 全局数据共享、资源访问 |

##### 2.12.1.2 获取方式

```java
ServletContext ctx = req.getServletContext();       // 通过request
ServletContext ctx = this.getServletContext();      // 通过Servlet本身
```

##### 2.12.1.3 相对路径转绝对路径

```java
String realPath = ctx.getRealPath("/WEB-INF/config.properties"); // 物理路径
```

##### 2.12.1.4 获取容器附加信息

```java
String serverInfo = ctx.getServerInfo();       // Tomcat版本
String version = ctx.getServletContextName();  // 应用名称
```

##### 2.12.1.5 获取 web.xml 配置

```xml
<!-- web.xml 全局参数 -->
<context-param>
    <param-name>dbUrl</param-name>
    <param-value>jdbc:mysql://localhost:3306/mydb</param-value>
</context-param>
```

```java
String dbUrl = ctx.getInitParameter("dbUrl");  // 获取全局参数
```

##### 2.12.1.6 全局数据共享

```java
ctx.setAttribute("counter", 100);              // 存储
int count = (Integer) ctx.getAttribute("counter"); // 获取
ctx.removeAttribute("counter");                // 删除
```

#### 2.12.2 ServletConfig 对象

##### 2.12.2.1 简介

ServletConfig 是**单个 Servlet 的配置对象**，每个 Servlet 独享。

```java
String name = config.getServletName();         // Servlet名称
String value = config.getInitParameter("key"); // 获取初始化参数
```

##### 2.12.2.2 web.xml 配置

```xml
<servlet>
    <servlet-name>myservlet</servlet-name>
    <servlet-class>com.example.MyServlet</servlet-class>
    <init-param>
        <param-name>config</param-name>
        <param-value>/config.properties</param-value>
    </init-param>
</servlet>
```

##### 2.12.2.3 注解方式配置

```java
@WebServlet(
    urlPatterns = "/myservlet",
    initParams = @WebInitParam(name = "config", value = "/config.properties")
)
```

#### 2.12.3 ServletContext vs ServletConfig

| 特性 | ServletContext | ServletConfig |
|------|----------------|---------------|
| 作用域 | 全局 | 单个Servlet |
| 数量 | 1个 | 每Servlet 1个 |
| 配置来源 | `<context-param>` | `<init-param>` |

#### 2.12.4 自启动 Servlet

##### 2.12.4.1 配置

```xml
<servlet>
    <servlet-name>initServlet</servlet-name>
    <servlet-class>com.example.InitServlet</servlet-class>
    <load-on-startup>1</load-on-startup>  <!-- 启动时加载，数字越小越先 -->
</servlet>
```

##### 2.12.4.2 用途

```java
@Override
public void init() throws ServletException {
    // 应用启动时执行：读取配置、初始化资源、连接池等
    Properties props = new Properties();
    props.load(getServletContext().getResourceAsStream("/WEB-INF/config.properties"));
    getServletContext().setAttribute("config", props);
}
```

#### 2.12.5 Servlet 线程安全问题

**问题**：Servlet 是单实例多线程，共享实例变量可能导致并发问题。

```java
// ❌ 不安全：实例变量
public class MyServlet extends HttpServlet {
    private int counter;  // 多线程共享
    
    protected void doGet(...) {
        counter++;  // 非原子操作，存在竞态条件
    }
}
```

```java
// ✅ 安全：使用局部变量或同步
protected void doGet(...) {
    int localCounter = 0;  // 局部变量，线程私有
    
    // 或使用同步块
    synchronized(this) {
        counter++;
    }
}
```

📐 **原则**：避免使用实例变量存储请求相关数据

#### 2.12.6 URL Pattern 配置

##### 2.12.6.1 匹配方式

| 匹配方式 | 示例 | 说明 |
|----------|------|------|
| 精确匹配 | `/hello` | 完全匹配 |
| 目录匹配 | `/admin/*` | 匹配目录下所有 |
| 扩展名匹配 | `*.do` | 匹配扩展名 |
| 默认匹配 | `/` | 匹配所有（优先级最低） |

##### 2.12.6.2 多 URL 映射

```java
@WebServlet(urlPatterns = {"/hello", "/hi", "*.do"})  // 多个映射
```

```xml
<servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/hello</url-pattern>
    <url-pattern>/hi</url-pattern>
</servlet-mapping>
```

#### 2.12.7 文件上传

##### 2.12.7.1 表单配置

```html
<form action="upload" method="post" enctype="multipart/form-data">
    <input type="file" name="file">
    <input type="submit" value="上传">
</form>
```

##### 2.12.7.2 Servlet 处理

```java
@MultipartConfig  // 必须添加注解
@WebServlet("/upload")
public class UploadServlet extends HttpServlet {
    protected void doPost(...) throws ... {
        Part part = req.getPart("file");                // 获取上传文件
        String filename = part.getSubmittedFileName();  // 文件名
        part.write(getServletContext().getRealPath("/upload/" + filename)); // 保存
    }
}
```

### 2.13 Filter 过滤器

#### 2.13.1 Filter 简介

Filter 是介于请求和 Servlet 之间的拦截器，可预处理请求和后处理响应。

```
请求 → Filter1 → Filter2 → Servlet → Filter2 → Filter1 → 响应
```

**用途**：
- 统一编码设置
- 登录权限验证
- 日志记录
- XSS/SQL注入过滤

#### 2.13.2 Filter 实现

```java
@WebFilter("/*")  // 拦截所有请求
public class EncodingFilter implements Filter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) 
            throws IOException, ServletException {
        req.setCharacterEncoding("UTF-8");   // 请求编码
        resp.setContentType("text/html;charset=UTF-8");  // 响应编码
        chain.doFilter(req, resp);           // 放行（必须调用）
    }
}
```

#### 2.13.3 Filter 生命周期

| 方法 | 调用时机 |
|------|----------|
| `init()` | Filter 初始化（应用启动） |
| `doFilter()` | 每次拦截请求 |
| `destroy()` | Filter 销毁（应用关闭） |

#### 2.13.4 FilterConfig 对象

```java
@Override
public void init(FilterConfig config) throws ServletException {
    String name = config.getFilterName();         // Filter名称
    String value = config.getInitParameter("key"); // 初始化参数
}
```

```xml
<filter>
    <filter-name>myFilter</filter-name>
    <filter-class>com.example.MyFilter</filter-class>
    <init-param>
        <param-name>key</param-name>
        <param-value>value</param-value>
    </init-param>
</filter>
```

#### 2.13.5 FilterChain（过滤器链）

```java
@WebFilter({"/admin/*", "/user/*"})
public class AuthFilter implements Filter {
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) {
        HttpServletRequest hreq = (HttpServletRequest) req;
        if (hreq.getSession().getAttribute("user") == null) {
            hreq.getRequestDispatcher("/login.jsp").forward(req, resp);  // 未登录跳转
            return;  // 不放行
        }
        chain.doFilter(req, resp);  // 已登录放行
    }
}
```

**执行顺序**：按配置顺序执行，放行后反向执行

#### 2.13.6 Filter url-pattern 配置

| 配置方式 | 示例 | 说明 |
|----------|------|------|
| 精确匹配 | `/login` | 拦截特定路径 |
| 目录匹配 | `/admin/*` | 拦截目录 |
| 扩展名匹配 | `*.jsp` | 拦截扩展名 |
| 全匹配 | `/*` | 拦截所有 |

### 2.14 Listener 监听器

#### 2.14.1 Listener 简介

Listener 用于监听 ServletContext、HttpSession、HttpServletRequest 的生命周期和属性变化。

#### 2.14.2 监听器分类

| 类型 | 监听对象 | 用途 |
|------|----------|------|
| 生命周期监听器 | 创建/销毁 | 初始化/清理资源 |
| 属性监听器 | 属性变化 | 数据变化通知 |

#### 2.14.3 ServletContext 监听器

##### 2.14.3.1 生命周期监听器

```java
@WebListener
public class AppListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        // 应用启动时执行：初始化数据库连接池、读取配置等
        System.out.println("应用启动");
    }
    
    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        // 应用关闭时执行：释放资源
        System.out.println("应用关闭");
    }
}
```

##### 2.14.3.2 属性监听器

```java
@WebListener
public class AttrListener implements ServletContextAttributeListener {
    @Override
    public void attributeAdded(ServletContextAttributeEvent scae) {
        System.out.println("添加属性：" + scae.getName() + "=" + scae.getValue());
    }
    
    @Override
    public void attributeRemoved(ServletContextAttributeEvent scae) {
        System.out.println("删除属性：" + scae.getName());
    }
    
    @Override
    public void attributeReplaced(ServletContextAttributeEvent scae) {
        System.out.println("替换属性：" + scae.getName());
    }
}
```

#### 2.14.4 HttpSession 监听器

##### 2.14.4.1 生命周期监听器

```java
@WebListener
public class SessionListener implements HttpSessionListener {
    @Override
    public void sessionCreated(HttpSessionEvent se) {
        // Session创建时
        System.out.println("Session创建：" + se.getSession().getId());
    }
    
    @Override
    public void sessionDestroyed(HttpSessionEvent se) {
        // Session销毁时
        System.out.println("Session销毁");
    }
}
```

##### 2.14.4.2 属性监听器

```java
@WebListener
public class SessionAttrListener implements HttpSessionAttributeListener {
    // attributeAdded、attributeRemoved、attributeReplaced
}
```

#### 2.14.5 HttpServletRequest 监听器

##### 2.14.5.1 生命周期监听器

```java
@WebListener
public class RequestListener implements ServletRequestListener {
    @Override
    public void requestInitialized(ServletRequestEvent sre) {
        // 请求开始时
        HttpServletRequest req = (HttpServletRequest) sre.getServletRequest();
        System.out.println("请求：" + req.getRequestURI());
    }
    
    @Override
    public void requestDestroyed(ServletRequestEvent sre) {
        // 请求结束时
    }
}
```

#### 2.14.6 监听器速查表

| 监听器接口 | 监听对象 | 方法 |
|-------------|----------|------|
| `ServletContextListener` | 生命周期 | `contextInitialized/Destroyed` |
| `ServletContextAttributeListener` | 属性 | `added/removed/replaced` |
| `HttpSessionListener` | 生命周期 | `sessionCreated/Destroyed` |
| `HttpSessionAttributeListener` | 属性 | `added/removed/replaced` |
| `ServletRequestListener` | 生命周期 | `requestInitialized/Destroyed` |
| `ServletRequestAttributeListener` | 属性 | `added/removed/replaced` |

---

## 附录：关键知识点归纳

### A.1 Servlet 生命周期

```
加载 → 实例化 → init() → [service() → doGet/doPost]循环 → destroy()
```

| 方法 | 执行次数 |
|------|----------|
| `init()` | 1次 |
| `service()` | 每次请求 |
| `destroy()` | 1次 |

### A.2 请求转发 vs 重定向

| 特性 | 请求转发 | 重定向 |
|------|----------|--------|
| 方法 | `forward()` | `sendRedirect()` |
| URL | 不变 | 变化 |
| 请求次数 | 1次 | 2次 |
| request | 共享 | 不共享 |

### A.3 Cookie vs Session

| 特性 | Cookie | Session |
|------|--------|---------|
| 存储 | 客户端 | 服务端 |
| 大小 | ≤4KB | 无限制 |
| 安全 | 低 | 高 |

### A.4 ServletContext vs ServletConfig

| 特性 | ServletContext | ServletConfig |
|------|----------------|---------------|
| 作用域 | 全局 | 单Servlet |
| 配置 | `<context-param>` | `<init-param>` |

### A.5 Filter 执行流程

```
请求 → Filter1.doFilter → Filter2.doFilter → Servlet → Filter2 → Filter1 → 响应
           ↓                    ↓
      chain.doFilter()    chain.doFilter()
```

### A.6 Listener 分类速查

| 监听对象 | 生命周期监听器 | 属性监听器 |
|----------|----------------|-----------|
| ServletContext | `ServletContextListener` | `ServletContextAttributeListener` |
| HttpSession | `HttpSessionListener` | `HttpSessionAttributeListener` |
| HttpServletRequest | `ServletRequestListener` | `ServletRequestAttributeListener` |

### A.7 常用注解速查

| 注解 | 用途 |
|------|------|
| `@WebServlet` | 定义Servlet |
| `@WebFilter` | 定义Filter |
| `@WebListener` | 定义Listener |
| `@WebInitParam` | 初始化参数 |
| `@MultipartConfig` | 文件上传 |

### A.8 编码设置要点

```java
// POST请求编码
req.setCharacterEncoding("UTF-8");  // 必须在getParameter前

// 响应编码
resp.setContentType("text/html;charset=UTF-8");  // 推荐

// Filter统一编码
@WebFilter("/*")
public class EncodingFilter implements Filter {
    public void doFilter(...) {
        req.setCharacterEncoding("UTF-8");
        resp.setContentType("text/html;charset=UTF-8");
        chain.doFilter(req, resp);
    }
}
```

### A.9 线程安全原则

- ❌ 避免使用实例变量存储请求数据
- ✅ 使用局部变量（线程私有）
- ✅ 使用 HttpSession 或 ServletContext 存储共享数据
- ✅ 必要时使用 synchronized 同步

### A.10 项目结构

```
webapp/
├── WEB-INF/
│   ├── web.xml          # 配置文件
│   ├── lib/             # 依赖库
│   └── classes/         # 编译输出
├── static/              # 静态资源
├── pages/               # JSP页面
└── index.jsp            # 首页
```

---