# Spring WebClient 核心技术手册

**阅读指南**：由浅入深，每个知识点均配有精简示例与注释。建议按顺序阅读，环环相扣。WebClient 是 Spring 5+ 响应式栈的非阻塞 HTTP 客户端，本手册聚焦核心原理与生产实战。

---

## 目录

- [1. 基础入门](#1-基础入门)
  - [1.1 为什么需要 WebClient](#11-为什么需要-webclient)
  - [1.2 核心概念：Mono 和 Flux](#12-核心概念mono-和-flux)
  - [1.3 环境搭建与依赖配置](#13-环境搭建与依赖配置)
  - [1.4 创建 WebClient 实例的三种方式](#14-创建-webclient-实例的三种方式)
- [2. Spring Boot 集成方式](#2-spring-boot-集成方式)
  - [2.1 添加 spring-boot-starter-webflux 依赖](#21-添加-spring-boot-starter-webflux-依赖)
  - [2.2 注入自动配置的 WebClient.Builder（核心方式）](#22-注入自动配置的-webclientbuilder核心方式)
  - [2.3 通过 WebClientCustomizer 实现全局配置](#23-通过-webclientcustomizer-实现全局配置)
  - [2.4 自定义 ClientHttpConnector 覆盖底层实现](#24-自定义-clienthttpconnector-覆盖底层实现)
  - [2.5 与 Spring Cloud LoadBalancer 集成实现服务发现](#25-与-spring-cloud-loadbalancer-集成实现服务发现)
- [3. 生产级配置](#3-生产级配置)
  - [3.1 全局单例 Bean 配置](#31-全局单例-bean-配置)
  - [3.2 Reactor Netty 连接池配置](#32-reactor-netty-连接池配置)
  - [3.3 超时控制（连接超时、读取超时、响应超时）](#33-超时控制连接超时读取超时响应超时)
  - [3.4 内存限制（maxInMemorySize）](#34-内存限制maxinmemorysize)
  - [3.5 底层 HTTP 客户端库选型](#35-底层-http-客户端库选型)
- [4. 核心请求与响应](#4-核心请求与响应)
  - [4.1 GET / POST / PUT / DELETE / PATCH 请求](#41-get--post--put--delete--patch-请求)
  - [4.2 retrieve() vs exchangeToMono() 获取响应](#42-retrieve-vs-exchangetomono-获取响应)
  - [4.3 响应解码：bodyToMono / bodyToFlux](#43-响应解码bodytomono--bodytoflux)
  - [4.4 同步阻塞调用 vs 异步响应式调用](#44-同步阻塞调用-vs-异步响应式调用)
- [5. 高级特性与实战](#5-高级特性与实战)
  - [5.1 ExchangeFilterFunction 请求/响应拦截](#51-exchangefilterfunction-请求响应拦截)
  - [5.2 错误处理：onStatus、onErrorResume](#52-错误处理onstatusonerrorresume)
  - [5.3 重试机制：retryWhen 与退避策略](#53-重试机制retrywhen-与退避策略)
  - [5.4 请求头与 Cookie 管理](#54-请求头与-cookie-管理)
  - [5.5 流式数据处理（SSE、大文件下载）](#55-流式数据处理sse大文件下载)
- [6. 生产环境最佳实践](#6-生产环境最佳实践)
  - [6.1 微服务间高频调用场景](#61-微服务间高频调用场景)
  - [6.2 性能优化：连接池调优、Mono.zip 并发请求](#62-性能优化连接池调优monozip-并发请求)
  - [6.3 监控与告警（请求量、响应时间、错误率）](#63-监控与告警请求量响应时间错误率)
  - [6.4 安全与认证（OAuth2、SSL/TLS）](#64-安全与认证oauth2ssltls)
  - [6.5 Java 21+ 虚拟线程集成](#65-java-21-虚拟线程集成)
- [7. 测试与调试](#7-测试与调试)
  - [7.1 单元测试：MockWebServer 模拟服务端](#71-单元测试mockwebserver-模拟服务端)
  - [7.2 Reactor Netty 调试日志开启](#72-reactor-netty-调试日志开启)
  - [7.3 请求/响应拦截器打印日志](#73-请求响应拦截器打印日志)

---

## 1. 基础入门

### 1.1 为什么需要 WebClient

WebClient 是 Spring WebFlux 提供的**非阻塞、响应式** HTTP 客户端，自 Spring 5.0 起作为 RestTemplate 在响应式栈中的替代方案。

**同步阻塞 vs 异步非阻塞的本质区别：**

| 维度 | RestTemplate / RestClient（同步） | WebClient（异步非阻塞） |
|------|------|------|
| I/O 模型 | 阻塞 I/O，一个请求占一个线程 | 非阻塞 I/O，少量线程处理大量并发 |
| 线程消耗 | 并发数 ≈ 线程数 | 少量 EventLoop 线程支撑高并发 |
| 编程模型 | 命令式、易理解 | 声明式、函数式、需理解 Reactor |
| 流式支持 | 弱（需自行处理） | 原生支持 SSE、流式、背压 |
| 适用栈 | Servlet（spring-boot-starter-web） | 响应式（spring-boot-starter-webflux） |

**核心优势：**
- **非阻塞 I/O**：基于 Reactor Netty，少量线程支撑高并发下游调用
- **背压支持**：响应式流（Reactive Streams）规范，生产者不会压垮消费者
- **流式处理**：原生支持 SSE、分块传输、大文件流式收发，避免 OOM
- **函数式 API**：链式声明，组合性强

**选型建议：**
- 传统 Servlet 栈、简单同步调用 → 优先用 **RestClient**（Spring 6.1+）
- 响应式栈（WebFlux）、高并发下游聚合、SSE/流式场景 → 用 **WebClient**
- 老项目已用 RestTemplate → 不必硬迁，新模块按栈选型

> ⚠️ 误区：WebClient 不是"更快的 RestTemplate"。在阻塞调用（`block()`）场景下，它的优势无法发挥，反而增加心智成本。其价值在于**非阻塞链式编排**。

### 1.2 核心概念：Mono 和 Flux

WebClient 的返回值是 Reactor 的 `Mono` / `Flux`，理解它们是使用 WebClient 的前提。它们都实现 `org.reactivestreams.Publisher` 接口。

| 类型 | 元素数量 | 典型语义 |
|------|---------|---------|
| `Mono<T>` | 0 或 1 | 一次 HTTP 调用的响应、单个对象 |
| `Flux<T>` | 0..N | 流式响应、集合、SSE 事件序列 |

**关键特性：声明式 + 延迟执行**

```java
// 仅声明，未订阅则不会真正发起 HTTP 请求
Mono<User> mono = webClient.get().uri("/users/1").retrieve().bodyToMono(User.class);
mono.subscribe(user -> System.out.println(user)); // 订阅后才触发请求
```

**高频操作符速查：**

```java
mono.map(u -> u.getName())              // 同步转换：User → String
    .flatMap(u -> fetchProfile(u))      // 异步转换：返回新 Mono
    .doOnNext(u -> log.info(u))         // 副作用：记录日志
    .onErrorResume(e -> Mono.empty());  // 错误降级
```

**并发聚合用 `Mono.zip`：**

```java
// 并行发起两个请求，等两者都完成后合并
Mono.zip(getUserMono(id), getOrdersMono(id))
    .map(t -> new UserDTO(t.getT1(), t.getT2()));
```

**背压（Backpressure）：** `Flux` 支持下游按需拉取（`request(n)`），生产端不会无限推送压垮消费端，这是流式处理不 OOM 的基础。

> 心法：**操作符是描述、订阅是执行**。在响应式链中混用 `block()` 会破坏非阻塞语义，应尽量避免。

### 1.3 环境搭建与依赖配置

**Maven 依赖（Spring Boot 3.x + Java 17+）：**

```xml
<!-- 响应式 Web 栈，包含 WebClient 与 Reactor Netty -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

**版本要点：**
- Spring Boot 3.x → Spring Framework 6.x → WebClient 内置 Reactor Netty
- JDK 最低 17（推荐 21 以获得虚拟线程支持）
- `spring-boot-starter-web` 与 `spring-boot-starter-webflux` 同时存在时，默认走 Servlet 栈，但 WebClient 仍可独立使用

**最小可运行示例：**

```java
WebClient client = WebClient.create("https://api.github.com");
String body = client.get().uri("/users/octocat")
        .retrieve().bodyToMono(String.class).block(); // 仅 demo 用 block
```

### 1.4 创建 WebClient 实例的三种方式

**方式一：`WebClient.create()` 静态工厂（最简单，适合 demo/脚本）**

```java
WebClient client1 = WebClient.create();               // 无默认 baseUrl
WebClient client2 = WebClient.create("https://api.xxx.com"); // 指定 baseUrl
```

**方式二：`WebClient.builder()` 链式构建（灵活配置）**

```java
WebClient client = WebClient.builder()
        .baseUrl("https://api.xxx.com")
        .defaultHeader("Accept", "application/json")
        .defaultCookie("lang", "zh")
        .codecs(c -> c.defaultCodecs().maxInMemorySize(2 * 1024 * 1024)) // 2MB
        .build();
```

**方式三：注入 `WebClient.Builder` Bean（Spring Boot 推荐，生产级）**

```java
@Service
public class UserService {
    private final WebClient webClient;
    // Spring Boot 自动配置 WebClient.Builder，自动应用所有 WebClientCustomizer
    public UserService(WebClient.Builder builder) {
        this.webClient = builder.baseUrl("https://api.xxx.com").build();
    }
}
```

**三者对比：**

| 方式 | 适用场景 | 是否复用全局配置 |
|------|---------|----------------|
| `create()` | 快速验证、无配置需求 | 否 |
| `builder()` | 需要自定义的独立实例 | 否 |
| 注入 `Builder` Bean | **生产推荐**，集成自动配置与 Customizer | 是 |

> 实践：生产环境统一用方式三，配合 `WebClientCustomizer`（见 2.3）实现全局配置收敛。

---

## 2. Spring Boot 集成方式

### 2.1 添加 spring-boot-starter-webflux 依赖

`spring-boot-starter-webflux` 同时提供：响应式服务端（WebFlux）与响应式客户端（WebClient）。即使项目是传统 Servlet 栈，也可仅引入它来使用 WebClient。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

**自动配置触发条件：** 类路径存在 `WebClient` 类时，`WebClientAutoConfiguration` 自动注册 `WebClient.Builder` Bean。该 Builder 已预置：
- 默认 `ExchangeStrategies`（Jackson 编解码）
- 所有 `WebClientCustomizer` Bean 的定制
- 所有 `ExchangeFilterFunction` Bean 作为过滤器

### 2.2 注入自动配置的 WebClient.Builder（核心方式）

这是 Spring Boot 官方推荐的生产用法。注入的 Builder 已被自动配置"加工"过，可复用全局定制。

```java
@Configuration
public class WebClientConfig {
    @Bean
    public WebClient webClient(WebClient.Builder builder) {
        // builder 已被自动配置加工，再按需微调
        return builder.baseUrl("https://api.xxx.com").build();
    }
}
```

```java
@Service
public class OrderService {
    private final WebClient webClient;
    public OrderService(WebClient webClient) { this.webClient = webClient; } // 直接注入成品
}
```

> 关键点：`WebClient.Builder` 是**原型作用域**（prototype），每次注入都是新实例，避免多个 Bean 互相污染配置；而 `WebClient` 本身线程安全，应作为单例共享。

### 2.3 通过 WebClientCustomizer 实现全局配置

`WebClientCustomizer` 是对 `WebClient.Builder` 的全局钩子：所有 Customizer Bean 会被自动应用到每一个注入的 Builder 上，实现"配置一处，处处生效"。

```java
@Configuration
public class GlobalWebClientConfig {
    @Bean
    public WebClientCustomizer globalCustomizer() {
        return builder -> builder
                .defaultHeader("X-App", "order-service")
                .defaultStatusHandler(
                    status -> status.is4xxClientError(),
                    resp -> Mono.error(new BizException("客户端错误")))
                .filter((req, next) -> {
                    long start = System.nanoTime();
                    return next.exchange(req).doAfterTerminate(() ->
                        log.info("耗时={}ms", (System.nanoTime() - start) / 1_000_000));
                });
    }
}
```

**适用场景：** 统一 TraceId 注入、统一鉴权头、统一错误码映射、统一耗时日志。多个 Customizer 按 `@Order` 顺序执行。

### 2.4 自定义 ClientHttpConnector 覆盖底层实现

`ClientHttpConnector` 是 WebClient 与底层 HTTP 库之间的抽象。默认用 Reactor Netty，可通过自定义 Connector 切换或深度定制底层。

```java
@Bean
public ClientHttpConnector customConnector() {
    // 切换为 JDK HttpClient（无需 Netty 依赖的场景）
    JdkClientHttpConnector connector = new JdkClientHttpConnector();
    connector.setReadTimeout(Duration.ofSeconds(5));
    return connector;
}
```

```java
// Reactor Netty 深度定制（生产主流）
HttpClient httpClient = HttpClient.create()
        .responseTimeout(Duration.ofSeconds(10))
        .compress(true);
WebClient client = WebClient.builder()
        .clientConnector(new ReactorClientHttpConnector(httpClient))
        .build();
```

| Connector 实现 | 底层库 | 特点 |
|---------------|--------|------|
| `ReactorClientHttpConnector` | Reactor Netty | **默认**，性能最佳，功能最全 |
| `JdkClientHttpConnector` | JDK 11+ HttpClient | 零额外依赖，HTTP/2 支持 |
| `JettyClientHttpConnector` | Jetty | 适合已用 Jetty 的环境 |

### 2.5 与 Spring Cloud LoadBalancer 集成实现服务发现

微服务场景下，用服务名替代 IP 端口，由 LoadBalancer 解析实例并轮询/随机。

**依赖：**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

**使用：在 `WebClient.Builder` 上加 `@LoadBalanced`**

```java
@Bean
@LoadBalanced // 关键注解：使该 Builder 支持 service-id 解析
public WebClient.Builder lbBuilder() {
    return WebClient.builder();
}
```

```java
// host 写服务名，LoadBalancer 自动解析为真实实例
Mono<Order> order = webClient.get()
        .uri("http://order-service/orders/{id}", 123)
        .retrieve().bodyToMono(Order.class);
```

**自定义负载策略（如同机房优先）：** 实现 `ReactorLoadBalancer` 或用 `@LoadBalancerClients(defaultConfiguration = ...)` 覆盖默认 `RoundRobinLoadBalancer`。

---

## 3. 生产级配置

### 3.1 全局单例 Bean 配置

WebClient 设计为**线程安全**的单例，应全局共享，**严禁每次请求 new 一个**（会重复创建连接池与 EventLoop，造成资源泄漏与性能下降）。

```java
@Configuration
public class WebClientConfig {
    @Bean
    public WebClient webClient(WebClient.Builder builder) {
        return builder.baseUrl("https://api.xxx.com").build(); // 单例
    }
}
```

```java
// 反例：每次调用都新建 —— 连接池无法复用，性能急剧下降
public Mono<User> getUser(String id) {
    return WebClient.create().get()...; // ❌ 错误做法
}
```

> 验证：JMH 基准测试下，单例 WebClient 的吞吐量是"每次 new"的 5~10 倍。

### 3.2 Reactor Netty 连接池配置

连接池是高并发的性能核心。默认 `ConnectionProvider` 连接数有限，生产环境必须按 QPS 调优。

```java
// 自定义连接池
ConnectionProvider provider = ConnectionProvider.builder("custom")
        .maxConnections(200)                              // 最大连接数
        .maxIdleTime(Duration.ofSeconds(30))              // 空闲连接存活时间
        .maxLifeTime(Duration.ofMinutes(5))               // 连接最大生命周期
        .pendingAcquireTimeout(Duration.ofSeconds(10))    // 获取连接超时
        .evictInBackground(Duration.ofSeconds(60))        // 后台清理
        .build();
HttpClient httpClient = HttpClient.create(provider);
WebClient client = WebClient.builder()
        .clientConnector(new ReactorClientHttpConnector(httpClient))
        .build();
```

**关键参数调优表：**

| 参数 | 默认值 | 调优建议 |
|------|--------|---------|
| `maxConnections` | 500（`elastic`）/ 受限 | 按 `峰值QPS × 平均RT(s)` 估算 |
| `pendingAcquireTimeout` | 45s | 建议调小至 5~10s，快速失败 |
| `maxIdleTime` | 无限 | 设 30~60s，及时回收空闲连接 |
| `evictInBackground` | 无 | 必设，定期清理失效连接 |

> 经验：连接池耗尽表现为 `PoolAcquisitionTimeoutException`，根因常是 `maxConnections` 过小或下游慢导致连接占而不还。

### 3.3 超时控制（连接超时、读取超时、响应超时）

WebClient 的超时是**分层**的，需在 Reactor Netty 层配置，没有统一的"全局超时"。

```java
HttpClient httpClient = HttpClient.create()
        .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 3000) // 连接建立超时
        .responseTimeout(Duration.ofSeconds(10))            // 整个响应超时（推荐）
        .doOnConnected(conn -> conn
            .addHandlerLast(new ReadTimeoutHandler(10))     // 读空闲超时
            .addHandlerLast(new WriteTimeoutHandler(5)));   // 写空闲超时
WebClient client = WebClient.builder()
        .clientConnector(new ReactorClientHttpConnector(httpClient))
        .build();
```

**超时层次与触发顺序：**

```
连接超时(3s) → 写超时(5s) → 响应超时(10s) → 读超时(10s)
```

| 超时类型 | 触发场景 | 配置方式 |
|---------|---------|---------|
| 连接超时 | TCP 三次握手未完成 | `CONNECT_TIMEOUT_MILLIS` |
| 响应超时 | 等待首字节或整体响应过久 | `responseTimeout`（最常用） |
| 读/写超时 | 连接已建立但数据传输停滞 | `ReadTimeoutHandler`/`WriteTimeoutHandler` |

> 实践：生产环境至少配 `responseTimeout` + `pendingAcquireTimeout`，形成"获取连接→等响应"双层兜底。

### 3.4 内存限制（maxInMemorySize）

WebClient 默认将响应体**完整加载到内存**，默认上限 **256KB**。响应体超限会抛 `DataBufferLimitException`，这是处理大 JSON 的常见坑。

```java
// 方式一：Builder 阶段配置（推荐）
WebClient client = WebClient.builder()
        .codecs(c -> c.defaultCodecs().maxInMemorySize(4 * 1024 * 1024)) // 4MB
        .build();
```

```java
// 方式二：通过 ExchangeStrategies 配置
ExchangeStrategies strategies = ExchangeStrategies.builder()
        .codecs(c -> c.defaultCodecs().maxInMemorySize(4 * 1024 * 1024))
        .build();
WebClient client = WebClient.builder().exchangeStrategies(strategies).build();
```

> ⚠️ 调大 `maxInMemorySize` 会增加 OOM 风险。真正的大响应（如导出文件）应改用**流式处理**（见 5.5），而非无脑调大限制。

### 3.5 底层 HTTP 客户端库选型

| 维度 | Reactor Netty（默认） | JDK HttpClient | Jetty |
|------|------|------|------|
| 性能 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| HTTP/2 | 支持 | 原生支持 | 支持 |
| 连接池 | 功能最全 | 基础 | 完善 |
| 额外依赖 | 需要 | 无（JDK 自带） | 需要 |
| 适用场景 | **生产首选** | 轻量/无依赖场景 | 已用 Jetty 栈 |

**选型结论：** 99% 的 Spring Boot 项目用默认的 Reactor Netty 即可。仅在"无法引入 Netty 依赖"或"特殊协议需求"时才考虑替代。

---

## 4. 核心请求与响应

### 4.1 GET / POST / PUT / DELETE / PATCH 请求

WebClient 用链式 API 表达 HTTP 请求，方法名即 HTTP 动词。

**GET 请求（路径变量 + 查询参数）：**

```java
// 路径变量与查询参数
Mono<User> user = webClient.get()
        .uri("/users/{id}?detail={detail}", 123, true) // 路径变量 + 查询参数
        .retrieve().bodyToMono(User.class);
```

**POST 请求（JSON Body）：**

```java
Mono<Order> result = webClient.post()
        .uri("/orders")
        .contentType(MediaType.APPLICATION_JSON)
        .bodyValue(new CreateOrderReq("SKU001", 2)) // 自动 Jackson 序列化
        .retrieve().bodyToMono(Order.class);
```

**PUT / PATCH / DELETE：**

```java
webClient.put().uri("/users/1").bodyValue(dto).retrieve().toBodilessEntity();      // 整体更新
webClient.patch().uri("/users/1").bodyValue(patchDto).retrieve().toBodilessEntity(); // 部分更新
webClient.delete().uri("/users/1").retrieve().toBodilessEntity();                  // 删除
```

> `toBodilessEntity()` 用于无响应体的场景，返回 `ResponseEntity<Void>`，可取状态码与头。

### 4.2 retrieve() vs exchangeToMono() 获取响应

两种获取响应的 API，对应"高层封装"与"底层控制"两个层次。

**`retrieve()` —— 高层 API（默认推荐）**

```java
Mono<User> user = webClient.get().uri("/users/1")
        .retrieve()                                    // 自动处理状态码与解码
        .onStatus(s -> s.is4xxClientError(), resp -> Mono.error(new BizException()))
        .bodyToMono(User.class);
```

**`exchangeToMono()` —— 底层 API（精细控制场景）**

```java
Mono<User> user = webClient.get().uri("/users/1")
        .exchangeToMono(resp -> {
            if (resp.statusCode().is2xxSuccessful()) {
                return resp.bodyToMono(User.class);    // 成功才解码
            } else if (resp.statusCode().value() == 404) {
                return Mono.empty();                   // 404 返回空，不报错
            }
            return resp.bodyToMono(String.class)       // 失败读错误体
                    .flatMap(msg -> Mono.error(new BizException(msg)));
        });
```

**核心区别：**

| 维度 | `retrieve()` | `exchangeToMono()` |
|------|--------------|---------------------|
| 状态码处理 | 提供 `onStatus` 钩子 | 完全自行处理 |
| 资源释放 | 框架自动释放响应体 | **必须**自行消费/释放，否则泄漏 |
| 适用场景 | 90% 常规场景 | 需按状态码分支处理响应体 |

> ⚠️ `exchange()`（旧 API）已废弃，因其易导致响应体未释放。务必改用 `exchangeToMono/Flux`，并在所有分支消费响应体。**铁律：拿到 `ClientResponse` 后，每个分支都必须消费 body 或显式 `releaseBody()`。**

### 4.3 响应解码：bodyToMono / bodyToFlux

**`bodyToMono` —— 单对象（0或1元素）**

```java
Mono<User> user = webClient.get().uri("/users/1")
        .retrieve().bodyToMono(User.class); // JSON → 单对象
```

**`bodyToFlux` —— 集合/流（N元素）**

```java
Flux<User> users = webClient.get().uri("/users")
        .retrieve().bodyToFlux(User.class); // JSON 数组 → Flux 逐元素
```

**泛型嵌套（`ParameterizedTypeReference`）**

```java
// List<User>、Page<User> 等泛型必须用类型引用，否则类型擦除导致解码失败
Mono<List<User>> list = webClient.get().uri("/users")
        .retrieve().bodyToMono(new ParameterizedTypeReference<List<User>>() {});
```

```java
// 分页响应（含泛型字段）
Mono<Page<User>> page = webClient.get().uri("/users/page")
        .retrieve().bodyToMono(new ParameterizedTypeReference<Page<User>>() {});
```

> 关键：`bodyToMono(List.class)` 会得到 `List<LinkedHashMap>`，丢失泛型；必须用 `ParameterizedTypeReference<List<User>>(){}` 保留类型信息。

### 4.4 同步阻塞调用 vs 异步响应式调用

WebClient 本质是异步的，但可与同步代码集成。

**异步非阻塞（响应式链内，推荐）**

```java
// 在 WebFlux Controller / 响应式 Service 中，全程不阻塞
@GetMapping("/users/{id}/profile")
public Mono<UserProfile> profile(@PathVariable Long id) {
    return getUserMono(id).flatMap(u -> getProfileMono(u.getId()));
}
```

**同步阻塞（与传统 Servlet 代码集成）**

```java
// 在传统 Service 中调用，用 block() 转同步
public User getUserSync(Long id) {
    return webClient.get().uri("/users/{id}", id)
            .retrieve().bodyToMono(User.class)
            .block(Duration.ofSeconds(5)); // 设置超时，避免无限等待
}
```

**阻塞调用的禁忌：**

```java
// ❌ 在 WebFlux 响应式线程（parallel-/netty-）中 block，会抛 IllegalBlockingInReacti‌veException
public Mono<X> wrong() {
    return Mono.fromCallable(() -> webClient.get()...block()); // 阻塞响应式线程
}
```

| 场景 | 推荐方式 | 备注 |
|------|---------|------|
| WebFlux 全栈 | 全程非阻塞，不用 `block()` | 发挥 WebClient 价值 |
| Servlet 栈调用下游 | `block(timeout)` 或用 RestClient | 务必设超时 |
| 响应式线程内需调阻塞 | `subscribeOn(Schedulers.boundedElastic())` | 隔离阻塞线程池 |

> 心法：**`block()` 是 WebClient 与同步世界对接的"逃生舱"，而非日常手段**。能用非阻塞链就用非阻塞链。

---

## 5. 高级特性与实战

### 5.1 ExchangeFilterFunction 请求/响应拦截

`ExchangeFilterFunction` 是 WebClient 的过滤器，类似 Servlet Filter，可拦截请求与响应。签名：`(ClientRequest, ExchangeFunction) -> Mono<ClientResponse>`。

**请求拦截（统一加 Token）：**

```java
ExchangeFilterFunction authFilter = (req, next) -> {
    String token = TokenHolder.get(); // 从上下文取 token
    ClientRequest newReq = ClientRequest.from(req)
            .header("Authorization", "Bearer " + token)
            .build();
    return next.exchange(newReq); // 继续链路
};
WebClient client = WebClient.builder().filter(authFilter).build();
```

**响应拦截（统一耗时日志）：**

```java
ExchangeFilterFunction logFilter = (req, next) -> {
    long start = System.nanoTime();
    return next.exchange(req).doAfterTerminate(() ->
            log.info("{} {} 耗时={}ms", req.method(), req.uri(),
                    (System.nanoTime() - start) / 1_000_000));
};
```

**过滤器顺序：** 多个 filter 按 `andThen` 顺序形成洋葱模型，请求由外向内、响应由内向外。可用 `@Order` 或 `Ordered` 控制顺序。

### 5.2 错误处理：onStatus、onErrorResume

**`onStatus` —— 按 HTTP 状态码映射异常**

```java
Mono<User> user = webClient.get().uri("/users/{id}", id)
        .retrieve()
        .onStatus(s -> s.is4xxClientError(), resp ->
            resp.bodyToMono(String.class)              // 读取错误体
                .flatMap(msg -> Mono.error(new BizException(msg))))
        .onStatus(s -> s.value() == 404, resp ->
            Mono.error(new NotFoundException("用户不存在")))
        .bodyToMono(User.class);
```

**`onErrorResume` —— 任意异常降级**

```java
Mono<User> fallback = webClient.get().uri("/users/{id}", id)
        .retrieve().bodyToMono(User.class)
        .onErrorResume(TimeoutException.class, e -> {
            log.warn("超时，走降级", e);
            return Mono.just(User.defaultUser()); // 超时降级为默认值
        })
        .onErrorResume(e -> cache.get(id));       // 其他异常走缓存
```

**错误处理策略对比：**

| API | 触发条件 | 典型用途 |
|-----|---------|---------|
| `onStatus` | 特定 HTTP 状态码 | 状态码 → 业务异常映射 |
| `onErrorResume` | 任意异常（含网络异常） | 降级、fallback |
| `onErrorReturn` | 任意异常 | 返回固定默认值 |
| `onErrorMap` | 任意异常 | 异常类型转换 |

### 5.3 重试机制：retryWhen 与退避策略

下游调用失败时，合理重试可提升成功率，但必须配合**退避**与**异常过滤**避免雪崩。

**简单重试 `retry(n)`（谨慎使用）：**

```java
// 失败立即重试 3 次，无退避 —— 不推荐，易放大下游压力
webClient.get().uri("/data").retrieve().bodyToMono(String.class).retry(3);
```

**指数退避 `retryWhen`（生产推荐）：**

```java
import reactor.util.retry.Retry;

Mono<String> result = webClient.get().uri("/data")
        .retrieve().bodyToMono(String.class)
        .retryWhen(Retry.backoff(3, Duration.ofMillis(500)) // 3次，初始500ms
                .maxBackoff(Duration.ofSeconds(2))          // 最大间隔2s
                .jitter(0.5)                                 // 抖动避免惊群
                .filter(this::isRetryable));                 // 仅重试可恢复异常
```

**可重试异常判定：**

```java
private boolean isRetryable(Throwable e) {
    // 网络超时、连接重置可重试；4xx 业务错误不可重试
    return e instanceof TimeoutException
            || e instanceof IOException
            || (e instanceof WebClientResponseException
                && ((WebClientResponseException) e).getStatusCode().is5xxServerError());
}
```

> ⚠️ **幂等性红线**：重试必须确保接口幂等。POST 创建类请求若重试可能导致重复创建，需配合幂等键（Idempotency-Key）或仅对幂等接口重试。

### 5.4 请求头与 Cookie 管理

**单次请求设置头：**

```java
webClient.get().uri("/data")
        .header("Authorization", "Bearer xxx")
        .header("X-Trace-Id", MDC.get("traceId"))
        .headers(h -> h.setBearerAuth(token)) // 快捷设置 Bearer
        .accept(MediaType.APPLICATION_JSON)
        .retrieve().bodyToMono(String.class);
```

**默认头（全局）：**

```java
WebClient.builder()
        .defaultHeader("User-Agent", "order-service/1.0")
        .defaultCookie("session", sessionId)
        .build();
```

**Cookie 管理：**

```java
// 单次请求带 Cookie
webClient.get().uri("/data").cookie("token", "abc").retrieve()...;
// 多请求共享会话：用默认 cookie，或用 ExchangeFilterFunction 从响应 Set-Cookie 提取并回填
```

> 注意：WebClient 默认**不维护 cookie 存储**（不像浏览器）。跨请求保持会话需自行用 filter 提取 `Set-Cookie` 并在后续请求回填。

### 5.5 流式数据处理（SSE、大文件下载）

流式处理是 WebClient 的杀手锏：响应体不全部加载到内存，而是按块/按事件处理，天然防 OOM。

**Server-Sent Events（SSE）接收：**

```java
// 服务端 text/event-stream，逐事件接收
Flux<ServerSentEvent<String>> sse = webClient.get()
        .uri("/stream/events")
        .accept(MediaType.TEXT_EVENT_STREAM)
        .retrieve().bodyToFlux(new ParameterizedTypeReference<ServerSentEvent<String>>() {});

sse.subscribe(evt -> log.info("事件={} 数据={}", evt.event(), evt.data())); // 逐条消费
```

**大文件流式下载（避免 OOM）：**

```java
// 响应体转为 DataBuffer 流，逐块写入文件，不占内存
Mono<Void> download = webClient.get().uri("/bigfile.zip")
        .retrieve().bodyToFlux(DataBuffer.class)
        .buffer() // 或直接逐块写
        .flatMap(buffer -> writeToFile(buffer, path))
        .then(); // 写完返回
```

```java
// 更优雅：用 DataBufferUtils 写入
Mono<Void> save = DataBufferUtils.write(
        webClient.get().uri("/bigfile.zip").retrieve().bodyToFlux(DataBuffer.class),
        outputStream);
```

| 场景 | 媒体类型 | 解码方式 |
|------|---------|---------|
| SSE 事件流 | `text/event-stream` | `bodyToFlux(ServerSentEvent)` |
| NDJSON（每行一对象） | `application/x-ndjson` | `bodyToFlux(MyType.class)` |
| 大文件 | `application/octet-stream` | `bodyToFlux(DataBuffer.class)` 流式写盘 |

> 心法：**只要响应体可能超过 `maxInMemorySize`，就必须用 `Flux` 流式处理**，而非 `bodyToMono` 整体加载。

---

## 6. 生产环境最佳实践

### 6.1 微服务间高频调用场景

微服务高频调用下游时，WebClient 的非阻塞特性可大幅降低线程开销。

**典型架构：单例 WebClient + 响应式链 + 连接池**

```java
@Service
public class AggregationService {
    private final WebClient webClient; // 单例注入

    // 聚合多个下游服务，全部非阻塞
    public Mono<AggDTO> aggregate(Long userId) {
        return Mono.zip(
                webClient.get().uri("/users/{id}", userId).retrieve().bodyToMono(User.class),
                webClient.get().uri("/orders?uid={id}", userId).retrieve().bodyToFlux(Order.class).collectList(),
                webClient.get().uri("/points/{id}", userId).retrieve().bodyToMono(Points.class)
        ).map(t -> new AggDTO(t.getT1(), t.getT2(), t.getT3()));
    }
}
```

**收益：** 三个下游调用并行（`zip`），总耗时 ≈ 最慢的一个，而非三者之和；全程不占阻塞线程，单机可支撑数千并发聚合。

### 6.2 性能优化：连接池调优、Mono.zip 并发请求

**连接池容量估算公式：**

```
maxConnections ≈ 峰值QPS × 平均响应时间(s) + 冗余(20%)
# 例：QPS=1000，RT=0.05s → 1000×0.05×1.2 = 60 连接
```

**并发请求聚合 `Mono.zip`：**

```java
// 三个独立请求并行，等全部完成
Mono.zip(req1, req2, req3).map(t -> merge(t.getT1(), t.getT2(), t.getT3()));
```

**串行依赖用 `flatMap`：**

```java
// 后者依赖前者结果，串行
req1.flatMap(r1 -> req2(r1.getId())).flatMap(r2 -> req3(r2.getCode()));
```

**有上限的并发用 `flatMap(..., concurrency)`：**

```java
// 批量请求，但限制并发为 10，避免打爆下游
Flux.range(1, 1000)
        .flatMap(id -> webClient.get().uri("/items/{id}", id).retrieve().bodyToMono(Item.class), 10);
```

> 反模式：用 `Flux.fromIterable(ids).flatMap(...)` 默认并发 256，可能瞬间打爆下游。务必显式设置合理的 `concurrency`。

### 6.3 监控与告警（请求量、响应时间、错误率）

WebClient 内置 Micrometer Observation 支持，开箱即用采集请求指标。

**开启 Observation：**

```java
// 引入 actuator + observation
WebClient client = WebClient.builder()
        .observationRegistry(observationRegistry) // 注入全局 ObservationRegistry
        .build();
```

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

**关键指标（自动采集）：**

| 指标 | 含义 | 告警阈值参考 |
|------|------|-------------|
| `http.client.requests.count` | 请求总量 | 监控 QPS |
| `http.client.requests.duration` | 响应耗时 | TP99 > 1s |
| `http.client.requests.errors` | 错误数 | 错误率 > 1% |

**分布式追踪：** 配合 `micrometer-tracing`（Brave/OTel）自动注入 TraceId/SpanId，跨服务串联调用链。

### 6.4 安全与认证（OAuth2、SSL/TLS）

**OAuth2 客户端凭据（client_credentials）集成：**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

```java
// 注册 OAuth2 配置，自动获取并刷新 token，自动附加到请求头
@Bean
ReactiveOAuth2AuthorizedClientManager manager(ReactiveClientRegistrationRepository repo,
                                              ReactiveOAuth2AuthorizedClientService service) {
    return new AuthorizedClientServiceReactiveOAuth2AuthorizedClientManager(repo, service);
}
```

```java
// 用 ServletOAuth2AuthorizedClientExchangeFilterFunction 自动加 token
ServletOAuth2AuthorizedClientExchangeFilterFunction oauth =
        new ServletOAuth2AuthorizedClientExchangeFilterFunction(manager);
WebClient client = WebClient.builder().apply(oauth.oauth2Configuration()).build();

// 调用时指定 clientRegistrationId
Mono<String> result = client.get().uri("/secure")
        .attributes(clientRegistrationId("my-client"))
        .retrieve().bodyToMono(String.class);
```

**SSL/TLS 配置（信任自签证书，仅测试）：**

```java
// 生产禁用！仅用于测试自签名证书
HttpClient httpClient = HttpClient.create()
        .secure(spec -> spec.sslContext(SslContextBuilder.forClient()
                .trustManager(InsecureTrustManagerFactory.INSTANCE).build()));
```

**双向认证（mTLS）：**

```java
// 加载 keystore（含客户端证书）与 truststore（含服务端证书）
SslContext sslContext = SslContextBuilder.forClient()
        .keyManager(clientCert, clientKey)               // 客户端证书
        .trustManager(trustStoreFile)                    // 信任的服务端证书
        .build();
HttpClient httpClient = HttpClient.create().secure(spec -> spec.sslContext(sslContext));
```

> 安全红线：**生产环境绝不使用 `InsecureTrustManagerFactory`**（等于关闭证书校验，易遭中间人攻击）。

### 6.5 Java 21+ 虚拟线程集成

Java 21 虚拟线程为"用同步代码写高并发"提供新选项，与 WebClient 形成互补。

**响应式 vs 虚拟线程选型：**

| 维度 | 响应式（WebClient + Reactor） | 虚拟线程（RestClient + 虚拟线程） |
|------|------------------------------|----------------------------------|
| 编程模型 | 声明式、链式 | 同步命令式（更易读） |
| 学习曲线 | 陡峭 | 平缓 |
| 流式/SSE | 原生支持 | 支持但不如响应式自然 |
| 调试 | 较难（栈帧断续） | 友好（完整栈） |
| 生态成熟度 | 成熟 | 较新（JDK 21 GA） |

**WebClient 与虚拟线程协作（混合场景）：**

```java
// 在虚拟线程中安全调用 WebClient（用 boundedElastic 调度器隔离）
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    Future<User> future = executor.submit(() ->
        webClient.get().uri("/users/1").retrieve().bodyToMono(User.class)
            .block()); // 虚拟线程中 block 不会占用平台线程
}
```

> 心法：**新项目（JDK 21+）若不需要流式/SSE，可优先用 RestClient + 虚拟线程**，代码更直观；需要流式背压或与 WebFlux 全栈集成时，仍用 WebClient。

---

## 7. 测试与调试

### 7.1 单元测试：MockWebServer 模拟服务端

`MockWebServer`（OkHttp 提供）可模拟下游服务，控制响应内容、状态码、延迟，是 WebClient 单测的事实标准。

**依赖：**

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>mockwebserver</artifactId>
    <scope>test</scope>
</dependency>
```

**测试示例：**

```java
@Test
void shouldReturnUser() throws Exception {
    MockWebServer server = new MockWebServer();
    server.enqueue(new MockResponse()
            .setBody("{\"id\":1,\"name\":\"Tom\"}")
            .addHeader("Content-Type", "application/json"));
    server.start();

    WebClient client = WebClient.create(server.url("/").toString());
    UserService service = new UserService(client);

    // 用 StepVerifier 验证响应式流
    StepVerifier.create(service.getUser(1L))
            .expectNextMatches(u -> u.getName().equals("Tom"))
            .verifyComplete();

    // 校验请求被正确发出
    RecordedRequest req = server.takeRequest();
    assertEquals("/users/1", req.getPath());
    server.shutdown();
}
```

**模拟异常场景：**

```java
// 模拟超时：设置 body 延迟超过客户端 responseTimeout
server.enqueue(new MockResponse().setBodyDelay(5, TimeUnit.SECONDS).setBody("slow"));
// 模拟 500 错误
server.enqueue(new MockResponse().setResponseCode(500).setBody("error"));
```

### 7.2 Reactor Netty 调试日志开启

Reactor Netty 的 `wiretap` 可打印完整报文（含头与体），是排查协议级问题的利器。

**针对单个 HttpClient 开启 wiretap：**

```java
HttpClient httpClient = HttpClient.create()
        .wiretap("reactor.netty.http.client", LogLevel.INFO, AdvancedByteBufFormat.TEXTUAL);
WebClient client = WebClient.builder()
        .clientConnector(new ReactorClientHttpConnector(httpClient))
        .build();
```

**全局日志级别（application.yml）：**

```yaml
logging:
  level:
    reactor.netty.http.client: DEBUG  # 打印连接、请求、响应概要
    # io.netty.handler.logging: DEBUG # 更底层的字节级日志
```

> ⚠️ wiretap 会打印完整请求/响应体（含敏感字段），**生产环境严禁开启**，仅用于本地或测试环境定位问题。

**Reactor 调试增强（定位操作符链问题）：**

```java
// 程序启动时开启，异常栈会带操作符链信息（有性能开销，仅调试用）
Hooks.onOperatorDebug();
// 或单点：在可疑位置加 .checkpoint("my-checkpoint")
```

### 7.3 请求/响应拦截器打印日志

生产环境推荐用**自定义 ExchangeFilterFunction** 打印结构化日志，可控脱敏。

```java
ExchangeFilterFunction logFilter = (req, next) -> {
    String traceId = MDC.get("traceId");
    log.info("[{}] -> {} {} headers={}", traceId, req.method(), req.uri(), maskHeaders(req.headers()));
    return next.exchange(req).doOnNext(resp ->
        log.info("[{}] <- {} headers={}", traceId, resp.statusCode(), resp.headers()));
};
```

**脱敏处理：**

```java
// 脱敏 Authorization、Cookie 等敏感头
private Map<String, String> maskHeaders(HttpHeaders headers) {
    return headers.toSingleValueMap().entrySet().stream()
            .collect(Collectors.toMap(
                Map.Entry::getKey,
                e -> SENSITIVE.contains(e.getKey()) ? "***" : e.getValue()));
}
```

**生产日志规范：**

| 环境 | 打印内容 | 注意事项 |
|------|---------|---------|
| 开发 | 完整 URL、头、体 | wiretap 全开 |
| 测试 | URL、状态码、耗时 | 便于复现问题 |
| 生产 | URL、状态码、耗时、TraceId | **不打印体**，脱敏敏感头 |

> 心法：日志是排查"调用黑盒"的唯一线索，但生产日志务必遵循**最小必要 + 脱敏**原则，避免泄露与日志爆炸。

---

## 附录：知识脉络速查

```
WebClient
├── 基础：非阻塞 I/O + Reactor（Mono/Flux）+ 声明式 API
├── 创建：create() / builder() / 注入 Builder Bean（推荐）
├── 配置：连接池(Reactor Netty) + 超时(分层) + 内存限制 + Connector 选型
├── 请求：get/post/... → retrieve() / exchangeToMono()
├── 解码：bodyToMono / bodyToFlux / ParameterizedTypeReference
├── 增强：Filter 拦截 + onStatus 错误映射 + retryWhen 退避重试 + 流式 SSE
├── 生产：单例 + zip 并发 + Observation 监控 + OAuth2/SSL 安全
└── 调试：MockWebServer 单测 + wiretap 报文日志 + 结构化脱敏日志
```

**三条铁律：**
1. **WebClient 必须单例**，严禁每次请求 new
2. **`exchangeToMono` 必须消费响应体**，否则资源泄漏
3. **重试必须幂等 + 退避**，否则放大下游压力引发雪崩
