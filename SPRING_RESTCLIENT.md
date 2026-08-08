# Spring RestClient 核心技术手册

**阅读指南**：由浅入深，每个知识点均配有精简示例与注释。建议按顺序阅读，环环相扣。RestClient 是 Spring 6.1 / Boot 3.2+ 推出的**现代同步 

---

## 目录

- [1. 认知篇](#1-认知篇)
  - [1.1 为什么需要专门的 HTTP 客户端](#11-为什么需要专门的-http-客户端)
  - [1.2 Spring 生态中 HTTP 客户端的演进史](#12-spring-生态中-http-客户端的演进史)
  - [1.3 RestClient vs RestTemplate vs WebClient 选型决策矩阵](#13-restclient-vs-resttemplate-vs-webclient-选型决策矩阵)
  - [1.4 快速入门：一分钟跑通第一个 GET 请求](#14-快速入门一分钟跑通第一个-get-请求)
- [2. 基础实战篇](#2-基础实战篇)
  - [2.1 创建与销毁：RestClient 实例化策略](#21-创建与销毁restclient-实例化策略)
  - [2.2 URI 构建艺术](#22-uri-构建艺术)
  - [2.3 请求体的序列化](#23-请求体的序列化)
  - [2.4 响应体的反序列化](#24-响应体的反序列化)
  - [2.5 同步阻塞调用：retrieve() 与 exchange()](#25-同步阻塞调用retrieve-与-exchange)
  - [2.6 异步编排：CompletableFuture 与虚拟线程](#26-异步编排completablefuture-与虚拟线程)
- [3. 进阶配置篇](#3-进阶配置篇)
  - [3.1 底层网络层配置（ClientHttpRequestFactory）](#31-底层网络层配置clienthttprequestfactory)
  - [3.2 超时矩阵的精细化管理](#32-超时矩阵的精细化管理)
  - [3.3 重试机制的几种实现方式](#33-重试机制的几种实现方式)
- [4. 拦截器与契约篇](#4-拦截器与契约篇)
  - [4.1 请求拦截器（ClientRequestInterceptor）实战](#41-请求拦截器clientrequestinterceptor实战)
  - [4.2 响应错误处理器全局统一](#42-响应错误处理器全局统一)
  - [4.3 适配老系统：拦截器实现报文加签验签](#43-适配老系统拦截器实现报文加签验签)
- [5. 生产级可观测性](#5-生产级可观测性)
  - [5.1 日志分级输出（Wire Logging）](#51-日志分级输出wire-logging)
  - [5.2 接入 Micrometer 监控指标](#52-接入-micrometer-监控指标)
  - [5.3 分布式追踪集成](#53-分布式追踪集成)
- [6. 大单体集成场景](#6-大单体集成场景)
  - [6.1 场景一：XML 接口的兼容](#61-场景一xml-接口的兼容)
  - [6.2 场景二：多版本管理（多 BaseURL）](#62-场景二多版本管理多-baseurl)
  - [6.3 场景三：大文件上传下载的流式处理](#63-场景三大文件上传下载的流式处理)
  - [6.4 场景四：Form 表单与 MultipartFile 混合上传](#64-场景四form-表单与-multipartfile-混合上传)
- [7. 微服务集成场景](#7-微服务集成场景)
  - [7.1 服务注册中心的透明化调用](#71-服务注册中心的透明化调用)
  - [7.2 声明式 HTTP 调用：从 RestClient 到 HttpExchange](#72-声明式-http-调用从-restclient-到-httpexchange)
  - [7.3 微服务上下文透传](#73-微服务上下文透传)
  - [7.4 熔断降级的深度整合](#74-熔断降级的深度整合)
- [8. 安全篇](#8-安全篇)
  - [8.1 SSL/TLS 基础：信任库与密钥库配置](#81-ssltls-基础信任库与密钥库配置)
  - [8.2 自签名证书的绕过方案](#82-自签名证书的绕过方案)
  - [8.3 双向认证 mTLS 在金融级接口中的落地](#83-双向认证-mtls-在金融级接口中的落地)
  - [8.4 敏感凭证的动态刷新](#84-敏感凭证的动态刷新)
- [9. 测试篇](#9-测试篇)
  - [9.1 使用 MockWebServer 模拟服务端行为](#91-使用-mockwebserver-模拟服务端行为)
  - [9.2 使用 WireMock 桩服务进行契约测试](#92-使用-wiremock-桩服务进行契约测试)
  - [9.3 @RestClientTest 切片测试与 Mock 注入](#93-restclienttest-切片测试与-mock-注入)
  - [9.4 性能基准测试（JMH）](#94-性能基准测试jmh)
- [10. 故障排查与最佳实践](#10-故障排查与最佳实践)
  - [10.1 经典故障：连接池耗尽的定位与修复](#101-经典故障连接池耗尽的定位与修复)
  - [10.2 经典故障：内存泄漏（Direct ByteBuffer 未释放）](#102-经典故障内存泄漏direct-bytebuffer-未释放)
  - [10.3 经典故障：HTTP/2 协议升级踩坑](#103-经典故障http2-协议升级踩坑)
  - [10.4 编码规范：静态工厂方法 vs 实例注入](#104-编码规范静态工厂方法-vs-实例注入)
  - [10.5 配置模板：生产环境推荐的 8 个核心配置项](#105-配置模板生产环境推荐的-8-个核心配置项)
  - [10.6 平滑迁移：从 RestTemplate 到 RestClient](#106-平滑迁移从-resttemplate-到-restclient)

---

## 1. 认知篇

> 重新审视 HTTP 客户端：先建立"为什么"，再谈"怎么用"。

### 1.1 为什么需要专门的 HTTP 客户端

在微服务架构中，服务间调用、第三方 API 集成是高频动作。直接使用 `java.net.HttpURLConnection` 或手写 socket 既繁琐又易错，专业的 HTTP 客户端需解决以下共性问题：

| 痛点 | 说明 | 专门客户端的解法 |
|------|------|----------------|
| 连接管理 | 每次 new Socket 都有 TCP 握手开销 | 连接池复用长连接 |
| 序列化 | JSON/XML ↔ Java 对象的来回转换 | 内置 HttpMessageConverter |
| 错误处理 | 4xx/5xx、超时、连接重置需统一处理 | onStatus / 异常体系 |
| 可观测性 | 调用黑盒，难定位慢调用 | 拦截器 + Micrometer + Tracing |
| 横切关注点 | 鉴权、TraceId、签名、日志 | 拦截器统一注入 |

> 心法：HTTP 客户端不是"能发请求就行"，而是**把网络调用的不确定性封装成可控、可观测、可治理的基础设施**。

### 1.2 Spring 生态中 HTTP 客户端的演进史

#### 1.2.1 远古时期的 Apache HttpClient 与 OkHttp

在 Spring 抽象层成熟之前，Java 工程师直接面对底层库：

- **Apache HttpClient**：连接池、重试、拦截器一应俱全，但 API 偏底层（`CloseableHttpClient`、`HttpGet`、`EntityUtils`），样板代码多。
- **OkHttp**：API 简洁、性能优异、原生支持 HTTP/2 与连接复用，Android 生态首选。

```java
// Apache HttpClient 5 裸用：样板代码冗长
try (CloseableHttpClient client = HttpClients.createDefault()) {
    HttpGet get = new HttpGet("https://api.github.com/users/octocat");
    try (CloseableHttpResponse resp = client.execute(get)) {
        String body = EntityUtils.toString(resp.getEntity()); // 手动消费实体
    }
}
```

问题：每个项目都自己封装一层，重复造轮子，缺乏统一的序列化与错误处理约定。

#### 1.2.2 模板模式的巅峰与终结：RestTemplate 的缺陷分析

`RestTemplate`（Spring 3.0, 2009）用**模板方法模式**统一了上述样板代码，统治了十年。但其设计缺陷随时间暴露：

1. **API 臃肿**：每个 HTTP 方法都有十几个重载（`getForObject`、`getForEntity`、`exchange`...），参数顺序难记，新人易踩坑。
2. **基于继承的可扩展性差**：自定义行为靠覆写 `doExecute`，侵入性强。
3. **字段可变、非线程安全设计遗留**：`RestTemplate` 本身线程安全，但其 `ClientHttpRequestInterceptor` 链以 List 持有，配置后不可变约束弱。
4. **同步阻塞独占线程**：高并发下游调用时线程数膨胀。
5. **错误处理割裂**：`DefaultErrorHandler` 抛 `RestClientException`，状态码与异常类型映射不直观。

> 结论：Spring 官方在 6.1 起推荐 RestClient，并在 7.0 路线正式 deprecate RestTemplate。新项目应直接用 RestClient。

#### 1.2.3 响应式浪潮：WebClient 的兴起

`WebClient`（Spring 5.0, 2017）是响应式栈的非阻塞客户端（详见 [SPRING_WEBCLIENT.md](file:///d:/workspace/note/SPRING_WEBCLIENT.md)）。它解决高并发下游聚合问题，但代价是**强制引入 Reactor 心智成本**，在传统 Servlet 栈中常被误用为"更快的 RestTemplate"（`block()` 后优势尽失）。

#### 1.2.4 集大成者：RestClient 的设计哲学

RestClient（Spring 6.1 / Boot 3.2, 2023）的设计目标可概括为：**保留同步编程的简单性，吸收 WebClient 的流式 API 优雅性**。

- **流式 API**：`client.get().uri(...).retrieve().body(Class)` 一气呵成，无重载地狱。
- **接口隔离**：`RestClient`（不可变入口）、`RequestHeadersSpec`、`ResponseSpec` 分阶段类型安全。
- **统一的错误处理**：`onStatus(Predicate, Function)` 按状态码精准映射。
- **底层可插拔**：通过 `ClientHttpRequestFactory` 切换 JDK / Apache / OkHttp / Jetty / Reactor Netty。
- **天然集成 Observation**：开箱即用接入 Micrometer 与 Tracing。

### 1.3 RestClient vs RestTemplate vs WebClient 选型决策矩阵

| 维度 | RestClient | RestTemplate | WebClient |
|------|-----------|--------------|-----------|
| 引入版本 | Spring 6.1 / Boot 3.2 | Spring 3.0（已 deprecated） | Spring 5.0 |
| 编程模型 | 同步、流式 | 同步、模板方法 | 异步、响应式 |
| I/O 模型 | 阻塞 | 阻塞 | 非阻塞 |
| API 风格 | Fluent，类型安全 | 重载多，易混乱 | Fluent + Mono/Flux |
| 线程消耗 | 一请求一线程 | 一请求一线程 | 少量 EventLoop |
| 错误处理 | `onStatus` 精准 | `ResponseErrorHandler` 粗粒度 | `onStatus` + Reactor |
| 流式/SSE | 弱（需自行处理流） | 弱 | 原生强支持 |
| 适用栈 | Servlet（推荐默认） | 老项目存量 | WebFlux / 高并发聚合 |
| 迁移成本 | 新项目零成本 | — | 需学 Reactor |

**一句话选型：**
- 新项目 / Servlet 栈 → **RestClient**（默认首选）
- 老项目存量 RestTemplate → 不必硬迁，新增调用逐步用 RestClient
- 响应式栈 / SSE / 高并发下游聚合 → **WebClient**

### 1.4 快速入门：一分钟跑通第一个 GET 请求

**依赖（Spring Boot 3.2+，`spring-web` 已随 web starter 引入）：**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

**最小可运行示例：**

```java
RestClient client = RestClient.create("https://api.github.com"); // 指定 baseUrl
User user = client.get()                       // GET 方法
        .uri("/users/{login}", "octocat")      // 路径变量
        .retrieve()                            // 进入响应阶段
        .body(User.class);                     // 反序列化为 User
System.out.println(user.name());
```

```java
public record User(String login, String name, String url) {} // Java record 作 DTO
```

> 三步心法：**选方法 → 拼 URI → 取 body**。RestClient 的流式 API 让意图一目了然。

---

## 2. 基础实战篇

> 从简单调用到参数细节，掌握日常 80% 的使用场景。

### 2.1 创建与销毁：RestClient 实例化策略

#### 方式一：静态工厂 `RestClient.create()`（适合 demo / 脚本）

```java
RestClient c1 = RestClient.create();                      // 无 baseUrl
RestClient c2 = RestClient.create("https://api.xxx.com"); // 带 baseUrl
```

#### 方式二：`RestClient.builder()` 链式构建（灵活配置）

```java
RestClient client = RestClient.builder()
        .baseUrl("https://api.xxx.com")
        .defaultHeader("Accept", "application/json")
        .defaultCookie("lang", "zh")
        .requestInterceptor(new TraceIdInterceptor())     // 见 4.1
        .build();
```

#### 方式三：注入 `RestClient.Builder` Bean（Spring Boot 生产推荐）

Spring Boot 自动配置 `RestClient.Builder`，并应用所有 `RestClientCustomizer` 与 `ClientHttpRequestFactoryBuilder`（按 classpath 自动探测底层库）。

```java
@Configuration
public class RestClientConfig {
    @Bean
    public RestClient restClient(RestClient.Builder builder) {
        return builder.baseUrl("https://api.xxx.com").build(); // 复用全局定制
    }
}
```

```java
@Service
public class OrderService {
    private final RestClient restClient;
    public OrderService(RestClient restClient) { this.restClient = restClient; } // 注入成品单例
}
```

**三者对比：**

| 方式 | 适用 | 复用全局配置 | 线程安全 |
|------|------|-------------|---------|
| `create()` | 快速验证 | 否 | 是 |
| `builder()` | 独立实例定制 | 否 | 是 |
| 注入 `Builder` Bean | **生产推荐** | 是 | 是 |

> 关键：`RestClient` 实例**线程安全且不可变**，应作为单例共享；`RestClient.Builder` 是原型作用域，每次注入新实例，避免多 Bean 互相污染。

**销毁：** RestClient 本身无需显式销毁；但底层 `ClientHttpRequestFactory`（如 Apache HttpClient 5 的 `CloseableHttpClient`）持有连接池，**应在 `@PreDestroy` 中关闭**，否则连接池线程与 socket 泄漏。

```java
@Bean(destroyMethod = "close") // HttpClient 实现了 Closeable，容器负责关闭
public CloseableHttpClient httpClient() { return HttpClients.createDefault(); }
```

### 2.2 URI 构建艺术

#### 2.2.1 路径变量（Path Variable）的占位符替换

```java
// {id} 由可变参数按序填充
client.get().uri("/orders/{id}", 42L).retrieve().body(Order.class);

// 命名变量更清晰，推荐多变量场景
client.get().uri("/users/{user}/repos/{repo}", Map.of("user", "octo", "repo", "kit"))
      .retrieve().body(Repo.class);
```

#### 2.2.2 请求参数（Request Params）的拼接与编码

```java
// 方式一：uriFunction 手动拼（最直观）
client.get().uri(uri -> uri.path("/search").queryParam("q", "spring").queryParam("page", 1).build())
      .retrieve().body(String.class);

// 方式二：UriBuilder 编码更安全（自动 URL encode 中文/特殊字符）
URI uri = UriComponentsBuilder.fromPath("/search")
        .queryParam("q", "春 季").build().encode().toUri();
client.get().uri(uri).retrieve().body(String.class); // q=%E6%98%A5%20%E5%AD%A3
```

> 踩坑：直接字符串拼接 `"/search?q=" + keyword` 不会编码，遇到空格/中文/`&` 会破坏 URL。**始终用 `UriComponentsBuilder` 或 `queryParam`**。

### 2.3 请求体的序列化

#### 2.3.1 默认的 Jackson 序列化机制

RestClient 默认注册 Jackson（若 classpath 有 `jackson-databind`），`body(Object)` 自动序列化为 JSON。

```java
client.post().uri("/orders")
        .contentType(MediaType.APPLICATION_JSON)
        .body(new OrderCreate("SKU-1", 2))           // 对象 → JSON
        .retrieve().toBodilessEntity();              // 忽略响应体
```

#### 2.3.2 自定义序列化器（处理 Date、BigDecimal 等特殊类型）

全局行为通过自定义 `ObjectMapper` 注入；局部行为用 `Jackson2ObjectMapperBuilder`。

```java
@Bean
public ObjectMapper objectMapper() {
    return JsonMapper.builder()
            .addModule(new JavaTimeModule())                 // Java 8 时间
            .defaultDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"))
            .build();
}
```

```java
// BigDecimal 序列化为字符串，避免浮点精度丢失（金融场景）
public class MoneySerializer extends JsonSerializer<BigDecimal> {
    @Override public void serialize(BigDecimal v, JsonGenerator g, SerializerProvider p) throws IOException {
        g.writeString(v.setScale(2, RoundingMode.HALF_UP).toPlainString());
    }
}
// 在字段上标注：@JsonSerialize(using = MoneySerializer.class)
```

### 2.4 响应体的反序列化

#### 2.4.1 处理泛型嵌套（List<T>、Page<T> 的 ParameterizedTypeReference）

Java 泛型擦除使 `body(List.class)` 退化为 `List<LinkedHashMap>`，必须用 `ParameterizedTypeReference` 保留类型。

```java
// ✅ 正确：保留 List<User> 的元素类型
List<User> users = client.get().uri("/users").retrieve()
        .body(new ParameterizedTypeReference<List<User>>() {});

// ❌ 错误：元素被反序列化为 LinkedHashMap，遍历时 ClassCastException
List<User> wrong = client.get().uri("/users").retrieve().body(List.class);
```

```java
// 嵌套泛型 Page<User> 同理
Page<User> page = client.get().uri("/users/page").retrieve()
        .body(new ParameterizedTypeReference<Page<User>>() {});
```

#### 2.4.2 空值、未知字段的处理策略

- **空响应体**：`.toBodilessEntity()` 返回 `ResponseEntity<Void>`；`.body(Class)` 对 204 返回 `null`。
- **未知字段**：Jackson 默认忽略（`FAIL_ON_UNKNOWN_PROPERTIES=false`）。严格模式可开启以尽早发现契约漂移。

```java
@Bean
public ObjectMapper objectMapper() {
    return JsonMapper.builder()
            .enable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES) // 严格：未知字段报错
            .build();
}
```

> 实践：**生产推荐开启严格模式**，把"下游偷偷加字段"的契约漂移暴露在测试期，而非线上静默吞掉。

### 2.5 同步阻塞调用：retrieve() 与 exchange()

#### `retrieve()`：高层封装，覆盖 90% 场景

```java
User user = client.get().uri("/users/{id}", 1)
        .retrieve()
        .onStatus(HttpStatusCode::is4xxClientError, (req, resp) -> {
            throw new BizException("客户端错误: " + resp.getStatusCode()); // 自定义异常
        })
        .body(User.class);
```

`retrieve()` 默认对 4xx/5xx 抛 `HttpClientErrorException` / `HttpServerErrorException`，可用 `onStatus` 覆盖。

#### `exchange()`：底层控制，可访问完整请求与响应

`exchange(ExchangeStrategy)` 把 `ClientRequest` 与 `ClientResponse` 都交给你，适合需要读取响应头、按状态分支解码的场景。

```java
User user = client.get().uri("/users/{id}", 1)
        .exchange((req, resp) -> {
            if (resp.getStatusCode().is4xxClientError()) {
                ApiError err = resp.bodyTo(ApiError.class); // 错误体解码
                throw new BizException(err.code(), err.msg());
            }
            return resp.bodyTo(User.class); // 成功体解码
        });
```

| 维度 | `retrieve()` | `exchange()` |
|------|-------------|-------------|
| 抽象层级 | 高 | 低 |
| 错误处理 | `onStatus` 声明式 | 手写分支 |
| 响应头访问 | `toEntity(Class)` | 直接 `resp.getHeaders()` |
| 适用 | 常规调用 | 需精细控制响应解码 |
| 风险 | 低 | 高（需自行消费响应体，否则连接泄漏） |

> ⚠️ `exchange()` 中**必须消费或关闭响应体**（`bodyTo` / `toEntity` 已自动处理），否则连接不归还连接池，最终耗尽。

### 2.6 异步编排：CompletableFuture 与虚拟线程

> 重要澄清：**RestClient 本身是同步阻塞的**，没有原生 `async()` 方法（这与 WebClient 的 `Mono` 不同）。要获得"异步"效果，需把阻塞调用外包到独立线程。Spring 官方在 7.0 路线也未给 RestClient 加原生异步，而是用 **虚拟线程（Loom）** 让"一请求一线程"变得廉价。

#### 方式一：`CompletableFuture.supplyAsync` + 自定义线程池

```java
Executor pool = Executors.newFixedThreadPool(50); // 受控线程池
CompletableFuture<User> future = CompletableFuture.supplyAsync(
        () -> client.get().uri("/users/{id}", 1).retrieve().body(User.class), pool);

// 多个下游并发聚合
CompletableFuture<User> u = supplyAsync(() -> getUser(1), pool);
CompletableFuture<List<Order>> o = supplyAsync(() -> getOrders(1), pool);
UserDTO dto = u.thenCombine(o, UserDTO::new).join(); // 等两者完成合并
```

#### 方式二：虚拟线程（Java 21+，推荐）

```java
// 每个阻塞调用跑在虚拟线程上，百万并发也不爆 OS 线程
Thread.startVirtualThread(() -> { /* RestClient 阻塞调用 */ });

// Spring Boot 一键启用：spring.threads.virtual.enabled=true
// @Async 方法、Tomcat 请求线程自动用虚拟线程，RestClient 阻塞调用几乎零成本
```

| 方式 | 线程成本 | 编程模型 | 适用 |
|------|---------|---------|------|
| `supplyAsync` + 池 | 受限于池大小 | Future 链式 | Java 17 / 需精细控并发 |
| 虚拟线程 | 极低 | 同步代码直接写 | **Java 21+ 首选** |

> 心法：**RestClient + 虚拟线程 = 同步写法、异步性能**。不要为了"异步"硬上 WebClient，虚拟线程让阻塞调用的成本趋近于零。

---

## 3. 进阶配置篇

> 连接池、超时、重试是生产稳定的三大基石。

### 3.1 底层网络层配置（ClientHttpRequestFactory）

`ClientHttpRequestFactory` 是 RestClient 与底层 HTTP 库的抽象边界。Spring Boot 按 classpath 自动探测：有 Apache HttpComponents 5 用之，否则用 JDK HttpClient。

#### 3.1.1 配置 JdkClientHttpRequestFactory（零额外依赖）

```java
JdkClientHttpRequestFactory factory = new JdkClientHttpRequestFactory();
factory.setReadTimeout(Duration.ofSeconds(5)); // 读取超时
RestClient client = RestClient.builder().requestFactory(factory).build();
```

#### 3.1.2 配置 Apache HttpClient 5 连接池（生产主流）

```java
PoolingHttpClientConnectionManager pool = new PoolingHttpClientConnectionManager();
pool.setMaxTotal(200);                 // 全局最大连接数
pool.setDefaultMaxPerRoute(50);        // 单路由（同 host）最大并发

CloseableHttpClient httpClient = HttpClients.custom()
        .setConnectionManager(pool)
        .evictIdleConnections(TimeValue.ofSeconds(30)) // 自动回收空闲连接
        .build();

HttpComponentsClientHttpRequestFactory factory =
        new HttpComponentsClientHttpRequestFactory(httpClient);
factory.setConnectTimeout(3000); // 连接超时 ms
```

> 关键参数：`maxTotal` 控总连接、`defaultMaxPerRoute` 控单点并发。**单路由过大、总连接过小**是连接池耗尽最常见配置错误。

#### 3.1.3 配置 OkHttp（连接复用、缓存）

```java
OkHttpClient ok = new OkHttpClient.Builder()
        .connectionPool(new ConnectionPool(50, 5, TimeUnit.MINUTES)) // 50 连接，5 分钟保活
        .connectTimeout(3, TimeUnit.SECONDS)
        .build();
RestClient client = RestClient.builder()
        .requestFactory(new OkHttpClientHttpRequestFactory(ok)).build();
```

### 3.2 超时矩阵的精细化管理

#### 3.2.1 连接超时（ConnectTimeout）

TCP 三次握手的最长等待。设过短会误杀跨城/弱网，设过长则故障时线程被占满。

```java
factory.setConnectTimeout(3000); // 3s，生产经验值
```

#### 3.2.2 读取超时（ReadTimeout）

建立连接后，**等待响应数据的最大间隔**。注意不是总耗时，而是两次数据之间的间隔。

```java
factory.setReadTimeout(5000); // 5s，按业务最慢接口设定
```

#### 3.2.3 写入超时（WriteTimeout）与全局兜底超时

- **WriteTimeout**：发送请求体的最大时间（大文件上传关键）。JDK/Apache 实现差异，OkHttp 显式支持。
- **全局兜底**：用断路器或 `CompletableFuture.orTimeout` 防止单请求拖垮线程池。

```java
// 全局兜底：不论底层超时，总耗时上限 8s
CompletableFuture<User> f = supplyAsync(() -> get(), pool)
        .orTimeout(8, TimeUnit.SECONDS)
        .exceptionally(ex -> fallbackUser()); // 超时降级
```

> 经验矩阵：连接 3s / 读取 5s / 兜底 8s。慢接口（报表、导出）单独配更长读取超时，**用多个 RestClient Bean 隔离**，避免相互拖累。

### 3.3 重试机制的几种实现方式

#### 3.3.1 基于 Spring Retry 的声明式重试

```java
@Retryable(retryFor = {ResourceAccessException.class}, // 网络抖动才重试
           maxAttempts = 3, backoff = @Backoff(delay = 500, multiplier = 2))
public User getUser(Long id) {
    return client.get().uri("/users/{id}", id).retrieve().body(User.class);
}
```

#### 3.3.2 基于 Resilience4j 的带退避（Backoff）策略重试

```java
RetryConfig config = RetryConfig.custom()
        .maxAttempts(3)
        .waitDuration(Duration.ofMillis(500))
        .retryOnException(e -> e instanceof ResourceAccessException)
        .build();
Retry retry = Retry.of("getUser", config);
User user = retry.executeSupplier(() -> client.get().uri("/users/{id}", 1)
        .retrieve().body(User.class));
```

#### 3.3.3 幂等性校验：重试时如何避免"重复下单"

> 重试的隐形前提是**幂等**。非幂等接口（POST 下单）重试会导致重复扣款。

- **GET / PUT / DELETE**：天然幂等，可放心重试。
- **POST**：需服务端支持幂等键（`Idempotency-Key` 头），或客户端去重。

```java
// 注入幂等键：相同键的重复请求，服务端返回首次结果
String key = UUID.randomUUID().toString();
client.post().uri("/orders")
        .header("Idempotency-Key", key)
        .body(req)
        .retrieve().body(Order.class);
// 重试时复用同一 key，服务端识别后直接返回首次创建的订单
```

> 心法：**先问"幂等吗"，再决定"重试吗"**。没有幂等保障的重试，是线上资损的定时炸弹。

---

## 4. 拦截器与契约篇

> 横切关注点（鉴权、日志、签名）的统一治理是工程化的分水岭。

### 4.1 请求拦截器（ClientRequestInterceptor）实战

`ClientRequestInterceptor` 在请求发出前介入，可改写请求头、体、URI。

#### 4.1.1 统一添加身份认证（Basic Auth / Bearer Token）

```java
// Bearer Token 注入
RestClient client = RestClient.builder()
        .requestInterceptor((req, body, exec) -> {
            req.getHeaders().setBearerAuth(tokenProvider.get()); // 动态 token
            return exec.execute(req, body);
        }).build();

// Basic Auth
client.get().uri("/api").basicAuthentication("user", "pass").retrieve().body(String.class);
```

#### 4.1.2 全链路 TraceId 注入（MDC 与日志关联）

```java
public class TraceIdInterceptor implements ClientRequestInterceptor {
    @Override
    public ClientHttpResponse intercept(HttpRequest req, byte[] body, ClientHttpRequestExecution exec)
            throws IOException {
        String traceId = MDC.get("traceId"); // 从当前线程 MDC 取
        if (traceId != null) req.getHeaders().set("X-Trace-Id", traceId); // 透传下游
        return exec.execute(req, body);
    }
}
```

> 下游服务把 `X-Trace-Id` 放回日志 MDC，即可串联整条调用链，定位问题不求人。

#### 4.1.3 请求/响应体的统一日志脱敏与审计

注意：请求体 `byte[] body` 在拦截器中已序列化，可直接记录；响应体是流，需用缓冲包装才能重复读取。

```java
// 请求侧：记录并脱敏
String safe = mask(body); // 脱敏手机号、身份证等
log.info("→ {} body={}", req.getURI(), safe);
```

### 4.2 响应错误处理器全局统一

#### 4.2.1 区分 HTTP 状态码（4xx vs 5xx）的降级策略

```java
User user = client.get().uri("/users/{id}", id).retrieve()
        .onStatus(HttpStatusCode::is4xxClientError, (req, resp) -> {
            // 4xx 客户端错误：不重试，直接业务异常
            throw new BizException(ErrorCode.CLIENT_ERROR, resp.getStatusCode());
        })
        .onStatus(HttpStatusCode::is5xxServerError, (req, resp) -> {
            // 5xx 服务端错误：可重试或降级
            throw new RetryableException("下游 5xx");
        })
        .body(User.class);
```

#### 4.2.2 解析服务端返回的错误码（ErrorCode）并转化为业务异常

```java
.onStatus(s -> s.isError(), (req, resp) -> {
    ApiError err = new ObjectMapper().readValue(resp.getBody(), ApiError.class);
    // 把下游错误码映射为本地业务异常，屏蔽 HTTP 细节
    throw new BizException(ErrorCode.fromRemote(err.code()), err.msg());
})
```

> 工程化要点：**对外暴露业务异常，对内记录 HTTP 细节**。不要让 `HttpServerErrorException` 蔓延到业务层。

### 4.3 适配老系统：拦截器实现报文加签验签

金融/对接老系统的接口常要求请求加签、响应验签（RSA / HMAC）。

```java
public class SignInterceptor implements ClientRequestInterceptor {
    @Override
    public ClientHttpResponse intercept(HttpRequest req, byte[] body, ClientHttpRequestExecution exec)
            throws IOException {
        String sign = HmacUtils.hmacSha256Hex(secretKey, new String(body, StandardCharsets.UTF_8));
        req.getHeaders().set("X-Sign", sign); // 请求加签
        ClientHttpResponse resp = exec.execute(req, body);
        // 响应验签：需 BufferingClientHttpResponseWrapper 包装才能重复读 body
        return new BufferingResponseWrapper(resp);
    }
}
```

> ⚠️ 默认响应体只能读一次。验签后又要把 body 交给 RestClient 反序列化，必须用 `BufferingClientHttpResponseWrapper` 包装响应，使其支持重复读取。

---

## 5. 生产级可观测性

> 让调用黑盒变白盒：日志、指标、追踪三件套。

### 5.1 日志分级输出（Wire Logging）

#### 5.1.1 开发环境打印完整 Headers & Body

通过底层库的 wire log 开启（Apache HttpClient 5 示例）：

```xml
<!-- logback.xml -->
<logger name="org.apache.hc.client5.http.wire" level="DEBUG"/> <!-- 打印完整报文 -->
```

或用拦截器在应用层打印（更可控）：

```java
.requestInterceptor((req, body, exec) -> {
    log.debug("→ {} headers={} body={}", req.getURI(), req.getHeaders(), new String(body));
    return exec.execute(req, body);
})
```

#### 5.1.2 生产环境只打印 URL 和耗时（避免敏感信息泄露）

生产禁用 wire log（密码、token 会明文打印），仅记录 URL 与耗时：

```java
.requestInterceptor((req, body, exec) -> {
    long start = System.nanoTime();
    try {
        return exec.execute(req, body);
    } finally {
        log.info("{} 耗时={}ms", req.getURI(), (System.nanoTime() - start) / 1_000_000);
    }
})
```

> 红线：**生产环境绝不打印 Authorization、Cookie、身份证、银行卡**。脱敏必须在拦截器层统一完成。

### 5.2 接入 Micrometer 监控指标

#### 5.2.1 自定义 RestClient 观察者（Observation）

RestClient 内置 Observation 支持，注入 `ObservationRegistry` 即自动埋点：

```java
RestClient client = RestClient.builder()
        .observationRegistry(observationRegistry) // 自动记录 http.client.requests 指标
        .build();
```

#### 5.2.2 统计 QPS、错误率、TP99 延迟

自动埋点后，Micrometer 暴露 `http.client.requests` 指标，含 `outcome`、`status`、`method`、`uri` 标签，Prometheus 可直接查询：

```
# QPS
rate(http_client_requests_seconds_count{uri="/users"}[1m])
# 错误率
rate(http_client_requests_seconds_count{outcome="SERVER_ERROR"}[1m]) 
  / rate(http_client_requests_seconds_count[1m])
# TP99
histogram_quantile(0.99, rate(http_client_requests_seconds_bucket[5m]))
```

> 关键：**URI 模板化**（用 `/users/{id}` 而非 `/users/123`），否则高基数标签打爆内存。

### 5.3 分布式追踪集成

#### 5.3.1 借助 Brave/OpenTelemetry 自动注入 Trace Context

RestClient 的 Observation 集成会自动注入 W3C `traceparent` 头，下游只要也接入同套 Tracing 即自动串联。配合 Spring Cloud Sleuth / Micrometer Tracing：

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-brave</artifactId>
</dependency>
```

```java
// RestClient.builder().observationRegistry(registry) 后，traceparent 头自动注入
// Zipkin/Jaeger UI 中可看到完整跨服务调用链
```

#### 5.3.2 跨服务传递 Baggage（业务自定义透传参数）

Baggage 用于透传业务字段（如租户 ID、灰度标识），随 Trace 上下文一并传播：

```java
BaggageField tenant = BaggageField.create("tenant-id");
tenant.update("acme"); // 当前线程设置
// RestClient 调用自动透传 baggage 到下游，无需手动加头
```

---

## 6. 大单体集成场景

> 老旧系统调用的优雅方案。

### 6.1 场景一：XML 接口的兼容（JAXB 与 Jackson 并存）

老系统 SOAP-like 接口仍用 XML。RestClient 通过 `contentType(MediaType.APPLICATION_XML)` + JAXB 序列化器支持。

```java
// 配置支持 XML 的 message converter
RestClient client = RestClient.builder()
        .messageConverters(c -> c.add(new Jaxb2RootElementHttpMessageConverter()))
        .build();

OrderResp resp = client.post().uri("/legacy/order")
        .contentType(MediaType.APPLICATION_XML)
        .accept(MediaType.APPLICATION_XML)
        .body(orderReq)                 // JAXb 注解的 POJO → XML
        .retrieve().body(OrderResp.class); // XML → POJO
```

> 老 POJO 用 `@XmlRootElement` 注解；新接口 JSON 用 Jackson `@JsonProperty`，两者可共存于不同 DTO。

### 6.2 场景二：多版本管理（多 BaseURL）

不同环境/版本的服务需多 BaseURL，用多 RestClient Bean 隔离，各自配置互不污染。

```java
@Configuration
public class MultiClientConfig {
    @Bean("userV1") RestClient userV1(RestClient.Builder b) { return b.baseUrl("https://user.api/v1").build(); }
    @Bean("userV2") RestClient userV2(RestClient.Builder b) { return b.baseUrl("https://user.api/v2").build(); }
}

// Spring 7.0 还可用 ApiVersionInserter 在单客户端内统一管理版本
RestClient client = RestClient.builder()
        .defaultVersion("1.2")
        .apiVersionInserter(ApiVersionInserter.fromHeader("API-Version").build())
        .build();
```

### 6.3 场景三：大文件上传下载的流式处理（避免 OOM）

整文件读入内存会 OOM。下载用 `InputStream` 流式消费，上传用 `InputStreamResource` 或 `Resource` 零拷贝。

```java
// 下载：流式写入文件，不进内存
try (InputStream in = client.get().uri("/big.zip").retrieve()
        .body(InputStream.class);
     OutputStream out = new FileOutputStream("big.zip")) {
    in.transferTo(out); // 边读边写
}

// 上传：用 Resource 避免全量读入
client.post().uri("/upload")
        .body(new FileSystemResource(Paths.get("big.dat"))) // 流式上传
        .retrieve().toBodilessEntity();
```

> 关键：**大文件场景所有 body 处理都走流**，绝不用 `byte[]` / `String` 全量承接。

### 6.4 场景四：Form 表单与 MultipartFile 混合上传

```java
// 普通 form
client.post().uri("/login")
        .contentType(MediaType.APPLICATION_FORM_URLENCODED)
        .body(BodyInserters.fromFormData("user", "tom").with("pwd", "123"))
        .retrieve().body(String.class);

// multipart：文件 + 普通字段混合
MultiValueMap<String, Object> parts = new LinkedMultiValueMap<>();
parts.add("file", new FileSystemResource("a.jpg"));     // 文件部分
parts.add("desc", "avatar");                            // 普通字段
client.post().uri("/upload")
        .contentType(MediaType.MULTIPART_FORM_DATA)
        .body(parts)
        .retrieve().body(String.class);
```

---

## 7. 微服务集成场景

> 服务发现与负载均衡下的透明调用。

### 7.1 服务注册中心的透明化调用

#### 7.1.1 集成 Spring Cloud LoadBalancer（替换 Ribbon）

通过 `@LoadBalanced` 标注的 `RestClient.Builder`，URI 中用服务名替代 host，自动解析实例并负载均衡。

```java
@Bean
@LoadBalanced
RestClient.Builder lbBuilder() { return RestClient.builder(); }

// 使用：host 写服务名 "user-service"，自动解析为实际 IP:port
RestClient client = lbBuilder().baseUrl("http://user-service").build();
User u = client.get().uri("/users/{id}", 1).retrieve().body(User.class);
```

#### 7.1.2 自定义服务实例选择策略（同机房优先、灰度标签路由）

实现 `ReactorLoadBalancer` 或用 `LoadBalancerRequestTransformer` 注入元数据过滤：

```java
public class SameZoneFilter implements ServiceInstanceListSupplier {
    // 仅返回与当前实例同 zone 的候选，减少跨机房延迟
    @Override public List<ServiceInstance> get(String serviceId) {
        return delegate.get(serviceId).stream()
                .filter(i -> i.getMetadata().get("zone").equals(currentZone))
                .toList();
    }
}
```

### 7.2 声明式 HTTP 调用：从 RestClient 到 HttpExchange

`@HttpExchange` 接口 + `HttpServiceProxyFactory` 提供 Feign 式声明式客户端，底层用 RestClient。

```java
public interface UserApi {
    @GetExchange("/users/{id}")
    User getById(@PathVariable Long id);

    @PostExchange("/users")
    User create(@RequestBody UserCreate req);
}

// 注册代理
@Bean
UserApi userApi(RestClient client) {
    return HttpServiceProxyFactory.builderFor(ClientAdapter.create(client)).build()
            .createClient(UserApi.class);
}
```

> Spring 7.0 引入 `@ImportHttpServices(group=...)` 批量注册同组接口，共享同一 RestClient，大幅简化多客户端配置。

### 7.3 微服务上下文透传

#### 7.3.1 自动透传全局 TraceId

见 [4.1.2 全链路 TraceId 注入](#412-全链路-traceid-注入mdc-与日志关联)，拦截器从 MDC 取 TraceId 注入请求头，配合 LoadBalancer 透传到下游。

#### 7.3.2 自动透传用户身份 Token 与 TenantId

```java
.requestInterceptor((req, body, exec) -> {
    req.getHeaders().setBearerAuth(SecurityContext.getToken()); // 用户 token
    req.getHeaders().set("X-Tenant-Id", TenantContext.get());   // 租户
    return exec.execute(req, body);
})
```

> 注意：透传依赖线程上下文（ThreadLocal / MDC）。**跨线程（`@Async` / 虚拟线程）时上下文会丢失**，需用 `TaskDecorator` 复制上下文，或用 Micrometer Context Propagation 自动桥接。

### 7.4 熔断降级的深度整合

#### 7.4.1 配置线程池隔离与信号量隔离

Resilience4j 的 `CircuitBreaker` + `Bulkhead`：线程池隔离防级联雪崩，信号量隔离省线程开销。

```java
CircuitBreaker cb = CircuitBreaker.of("user-svc", CircuitBreakerConfig.custom()
        .failureRateThreshold(50)              // 失败率 50% 触发熔断
        .waitDurationInOpenState(Duration.ofSeconds(10))
        .slidingWindowSize(20)
        .build());
Bulkhead bulkhead = Bulkhead.of("user-svc", BulkheadConfig.custom()
        .maxConcurrentCalls(20).build());      // 信号量隔离：最多 20 并发

User u = Decorators.ofSupplier(() -> client.get().uri("/users/{id}", 1).retrieve().body(User.class))
        .withBulkhead(bulkhead).withCircuitBreaker(cb).decorate().get();
```

#### 7.4.2 远程调用超时后的业务 Fallback 兜底逻辑

```java
User u = cb.executeSupplier(() -> client.get().uri("/users/{id}", 1).retrieve().body(User.class));
// 熔断开启或调用异常时，用 Try/recover 兜底
User safe = Try.ofSupplier(() -> cb.executeSupplier(call))
        .recoverAny(throwable -> defaultUser()).get(); // 降级返回默认用户
```

> 心法：**熔断器保护的是"自己"，降级保护的是"用户体验"**。两者搭配，下游挂了也要给用户一个体面的兜底。

---

## 8. 安全篇

> 构建可信的 HTTPS 链路。

### 8.1 SSL/TLS 基础：信任库与密钥库配置

- **TrustStore**（信任库）：存放可信 CA 证书，用于验证服务端身份（单向认证）。
- **KeyStore**（密钥库）：存放本端私钥与证书，用于 mTLS 时向服务端证明身份。

```java
SSLContext ssl = SSLContextBuilder.create()
        .loadTrustMaterial(new File("trust.p12"), "changeit".toCharArray()) // 信任库
        .loadKeyMaterial(new File("client.p12"), "changeit".toCharArray())  // 密钥库（mTLS）
        .build();
HttpClient http = HttpClients.custom().setSSLContext(ssl).build();
RestClient client = RestClient.builder()
        .requestFactory(new HttpComponentsClientHttpRequestFactory(http)).build();
```

### 8.2 自签名证书的绕过方案（仅限测试环境）

> ⚠️ 严禁用于生产。这会让客户端接受任何证书，完全失去中间人防护。

```java
// 信任所有证书（测试专用）
SSLContext allTrust = SSLContextBuilder.create()
        .loadTrustMaterial(null, (chain, authType) -> true).build();
HttpClient http = HttpClients.custom()
        .setSSLContext(allTrust)
        .setSSLHostnameVerifier(NoopHostnameVerifier.INSTANCE) // 跳过主机名校验
        .build();
```

### 8.3 双向认证 mTLS 在金融级接口中的落地

mTLS 要求双向验证：服务端验客户端证书，客户端验服务端证书。配置见 8.1 同时加载 trust 与 key material。生产要点：

1. 证书由企业 PKI 签发，私钥存 KeyStore，密码用环境变量/K8s Secret 注入。
2. 证书到期前 30 天自动告警，避免线上批量失败。
3. 不同业务用不同证书，按最小权限隔离。

### 8.4 敏感凭证的动态刷新

Token / 证书会过期，需动态刷新而非启动时写死。

```java
// Bearer Token 动态刷新（每次请求取最新 token）
.requestInterceptor((req, body, exec) -> {
    req.getHeaders().setBearerAuth(tokenProvider.get()); // tokenProvider 内部处理刷新
    return exec.execute(req, body);
})

// 证书动态轮换：配合 Spring Cloud Vault / K8s Secrets 定时拉取新证书，重建 SSLContext
```

---

## 9. 测试篇

> 让外部依赖不再成为单元测试的阻碍。

### 9.1 使用 MockWebServer（OkHttp）模拟服务端行为

`MockWebServer` 启动本地端口，按入队顺序返回预设响应，适合精细控制响应序列。

```java
MockWebServer server = new MockWebServer();
server.enqueue(new MockResponse().setBody("{\"login\":\"octo\"}").addHeader("Content-Type", "application/json"));
server.start();
RestClient client = RestClient.create(server.url("/").toString());

User u = client.get().uri("/users/1").retrieve().body(User.class);
assertThat(u.login()).isEqualTo("octo");
server.shutdown();
```

### 9.2 使用 WireMock 桩服务进行契约测试

WireMock 用规则匹配请求并返回桩响应，适合定义跨团队契约。

```java
WireMockServer wire = new WireMockServer();
wire.start();
wire.stubFor(get(urlEqualTo("/users/1"))
        .willReturn(aResponse().withStatus(200).withBody("{\"login\":\"octo\"}")));
// RestClient 指向 wire.baseUrl() 进行调用与断言
```

### 9.3 @RestClientTest 切片测试与 Mock 注入

Spring Boot 的 `@RestClientTest` 只加载 RestClient 相关组件，用 `MockRestServiceServer` 拦截真实网络调用。

```java
@RestClientTest(UserClient.class)
class UserClientTest {
    @Autowired UserClient userClient;
    @Autowired MockRestServiceServer server; // 拦截注入的 RestClient

    @Test void getById() {
        server.expect(requestTo("/users/1"))
              .andRespond(withSuccess("{\"login\":\"octo\"}", MediaType.APPLICATION_JSON));
        assertThat(userClient.getById(1L).login()).isEqualTo("octo");
    }
}
```

> Spring 7.0 新增 `RestTestClient`，提供类似 `WebTestClient` 的流式断言 API，可对接真实服务器或 MockMvc。

### 9.4 性能基准测试（JMH）验证连接池配置是否合理

JMH 衡量不同连接池配置下的吞吐与延迟，避免拍脑袋调参。

```java
@BenchmarkMode(Mode.Throughput) @State(Scope.Benchmark)
public class RestClientBenchmark {
    RestClient client;
    @Setup public void setup() { client = RestClient.create("http://localhost:8080"); }

    @Benchmark public User getUser() {
        return client.get().uri("/users/1").retrieve().body(User.class); // 测吞吐
    }
}
```

> 经验：基准测试要在**与服务同机/同网络**环境跑，关注 P99 而非均值。均值掩盖长尾，P99 才是用户体验真相。

---

## 10. 故障排查与最佳实践

> 30 年老兵的避坑指南。

### 10.1 经典故障：连接池耗尽的定位与修复

**现象**：高峰期大量 `ConnectionPoolTimeoutException` / `TimeoutException`，接口卡死，线程堆栈全卡在 `PoolingHttpClientConnectionManager.leaseConnection`。

**定位三步**：
1. jstack 看线程：大量 `BLOCKED` 在连接池租借。
2. 看 Micrometer `http.client.requests` 是否 P99 飙升、`outcome` 集中在 5xx。
3. 查下游是否有慢接口长期占用连接（读取超时设置过长）。

**根因与修复**：
- 读取超时设 60s，慢接口占满连接池 → 调小读取超时，慢接口独立池。
- `defaultMaxPerRoute=2` 太小 → 调大单路由并发。
- 忘记消费响应体，连接不归还 → `exchange()` 中确保 `bodyTo`。
- 未启用空闲连接回收 → 加 `evictIdleConnections`。

### 10.2 经典故障：内存泄漏（Direct ByteBuffer 未释放）

**现象**：堆外内存持续上涨直至 OOM（`Direct buffer memory`），常见于大文件/流式响应未正确关闭。

**根因**：`InputStream` / `ResponseEntity` 持有底层 Direct ByteBuffer，未 close 导致不释放。

**修复**：

```java
// ✅ try-with-resources 确保 InputStream 关闭
try (InputStream in = client.get().uri("/big").retrieve().body(InputStream.class)) {
    process(in);
}
// ❌ 忘记 close → 堆外内存泄漏
InputStream in = client.get().uri("/big").retrieve().body(InputStream.class);
process(in); // 未关闭
```

### 10.3 经典故障：HTTP/2 协议升级踩坑

**现象**：升级 HTTP/2 后偶发 `Connection reset by peer`、首请求慢、连接复用异常。

**根因**：
- 服务端/中间件（部分 Nginx、F5）HTTP/2 实现不完整，连接复用语义与 HTTP/1.1 不同。
- H2 连接多路复用，单连接上多请求共享，一个请求的协议错误影响整连接。

**对策**：
- 灰度验证：先在低流量服务验证，观察 `http.client.requests` 与错误率。
- 回退开关：保留 HTTP/1.1 配置，出问题快速切回。
- JDK HttpClient 默认 H2 协商，可强制 1.1：`HttpClient.newBuilder().version(HTTP_1_1)`。

### 10.4 编码规范：静态工厂方法 vs 实例注入

| 方式 | 示例 | 适用 |
|------|------|------|
| 静态工厂 | `RestClient.create(url)` | demo / 工具脚本 / 一次性 |
| 实例注入 | `@Bean RestClient` + 构造注入 | **生产唯一推荐** |

```java
// ❌ 反模式：每次调用 new 一个 RestClient（连接池不复用、性能差）
public User get(Long id) {
    return RestClient.create("https://api").get().uri("/users/{id}", id).retrieve().body(User.class);
}

// ✅ 正解：注入单例，复用连接池与配置
private final RestClient client;
public User get(Long id) { return client.get().uri("/users/{id}", id).retrieve().body(User.class); }
```

### 10.5 配置模板：生产环境推荐的 8 个核心配置项

```java
@Bean
public RestClient restClient(RestClient.Builder builder, CloseableHttpClient httpClient) {
    return builder
        .requestFactory(new HttpComponentsClientHttpRequestFactory(httpClient)) // ① Apache 连接池
        .baseUrl("https://api.xxx.com")                  // ② 统一 baseUrl
        .defaultHeader("Accept", "application/json")     // ③ 默认 Accept
        .requestInterceptor(new TraceIdInterceptor())    // ④ TraceId 透传
        .requestInterceptor(new AuthInterceptor())       // ⑤ 鉴权注入
        .observationRegistry(observationRegistry)        // ⑥ Micrometer 埋点
        .defaultStatusHandler(HttpStatusCode::isError,   // ⑦ 全局错误处理
            (req, resp) -> { throw new BizException(...); })
        .build();
}
// ⑧ 超时矩阵：连接 3s / 读取 5s / 兜底 8s（在 factory 层设置，见 3.2）
```

### 10.6 平滑迁移：从 RestTemplate 到 RestClient

**迁移映射表：**

| RestTemplate | RestClient |
|--------------|-----------|
| `getForObject(url, Class)` | `get().uri(url).retrieve().body(Class)` |
| `getForEntity(url, Class)` | `get().uri(url).retrieve().toEntity(Class)` |
| `postForObject(url, req, Class)` | `post().uri(url).body(req).retrieve().body(Class)` |
| `exchange(url, GET, entity, Class)` | `get().uri(url).retrieve().body(Class)` |
| `setErrorHandler` | `onStatus` / `defaultStatusHandler` |
| `ClientHttpRequestInterceptor` | `ClientRequestInterceptor`（同接口） |

**渐进迁移策略**（降低风险）：

1. **共存期**：新代码用 RestClient，老代码不动，两套并行。
2. **适配层**：公共错误处理、拦截器抽到共享模块，两套客户端复用。
3. **逐步替换**：按模块灰度替换，每替换一个模块跑一轮回归。
4. **下线 RestTemplate**：全量替换后删除老 Bean，统一到 RestClient。

```java
// 机械映射示例：RestTemplate
restTemplate.getForObject("/users/{id}", User.class, 1);
// → RestClient
client.get().uri("/users/{id}", 1).retrieve().body(User.class);
```

> 心法：迁移不是一次性重写，而是**可回滚的渐进替换**。每一步都可独立上线、独立回滚，胜过一次大爆炸式重构。

---

**结语**：RestClient 用同步的简单性承载了现代 HTTP 客户端的核心诉求——流式 API、可插拔底层、统一错误处理、开箱即用的可观测性。掌握它，等于为服务间通信装上了可控、可观测、可治理的基础设施。配合虚拟线程（Java 21+），同步代码也能享受异步性能——这正是后端工程的优雅所在。
