# SpringDoc OpenAPI 实战手册

> 由浅入深，环环相扣 | 每个知识点配有精简代码示例与注释

## 目录

- [一、概述：什么是 SpringDoc](#一概述什么是-springdoc)
- [二、快速开始](#二快速开始)
  - [2.1 添加依赖](#21-添加依赖)
  - [2.2 默认访问地址](#22-默认访问地址)
- [三、核心配置](#三核心配置)
  - [3.1 基础配置（application.yml）](#31-基础配置applicationyml)
  - [3.2 自定义 API 全局信息](#32-自定义-api-全局信息)
- [四、核心注解详解（由浅入深）](#四核心注解详解由浅入深)
  - [4.1 @Tag —— 模块分组（Controller 层）](#41-tag--模块分组controller-层)
  - [4.2 @Operation —— 接口方法描述](#42-operation--接口方法描述)
  - [4.3 @ApiResponse 与 @ApiResponses —— 响应定义](#43-apiresponse-与-apiresponses--响应定义)
  - [4.4 @Parameter —— 参数描述](#44-parameter--参数描述)
  - [4.5 @Schema —— 实体类/字段描述](#45-schema--实体类字段描述)
  - [4.6 @Hidden —— 隐藏接口或字段](#46-hidden--隐藏接口或字段)
- [五、安全认证集成](#五安全认证集成)
  - [5.1 JWT Bearer Token 配置](#51-jwt-bearer-token-配置)
  - [5.2 Spring Security 白名单配置](#52-spring-security-白名单配置)
- [六、高级配置](#六高级配置)
  - [6.1 分组文档（多组 API）](#61-分组文档多组-api)
  - [6.2 全局响应与异常处理](#62-全局响应与异常处理)
  - [6.3 响应式 WebFlux 支持](#63-响应式-webflux-支持)
  - [6.4 自定义 OperationCustomizer](#64-自定义-operationcustomizer)
- [七、生产环境最佳实践](#七生产环境最佳实践)
  - [7.1 环境隔离与开关控制](#71-环境隔离与开关控制)
  - [7.2 包路径与接口过滤](#72-包路径与接口过滤)
  - [7.3 性能优化建议](#73-性能优化建议)
- [八、常见问题排查](#八常见问题排查)
  - [8.1 未生成 API 文档](#81-未生成-api-文档)
  - [8.2 文档显示错误](#82-文档显示错误)
  - [8.3 接口调用失败](#83-接口调用失败)

---

## 一、概述：什么是 SpringDoc

SpringDoc 是 **Spring Boot 3.x** 官方推荐的 OpenAPI 3.0 文档生成工具，取代了老旧的 SpringFox（Swagger2）。

**核心优势：**
- ✅ 原生支持 Spring Boot 3.x / Jakarta EE 命名空间
- ✅ 基于 OpenAPI 3.0 规范，功能更强大
- ✅ 支持 Spring WebMvc、WebFlux、Security、Actuator 等生态
- ✅ 零额外注解即可自动扫描接口

**与 SpringFox 对比：**

| 特性 | SpringDoc (推荐) | SpringFox (旧) |
|------|------------------|----------------|
| Spring Boot 3.x | ✅ 原生支持 | ❌ 不支持 |
| OpenAPI 版本 | 3.0 | 2.0 |
| 命名空间 | jakarta.* | javax.* |
| 维护状态 | 活跃 | 停止维护 |

> ⚠️ **注意**：Spring Boot 3.x 必须使用 SpringDoc 2.x 版本。

---

## 二、快速开始

### 2.1 添加依赖

**Maven（pom.xml）：**

```xml
<!-- SpringDoc 核心依赖（WebMvc） -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.6.0</version>
</dependency>
```

**Gradle（build.gradle）：**

```groovy
// SpringDoc 核心依赖（WebMvc）
implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.6.0'
```

> 💡 WebFlux 项目请替换为 `springdoc-openapi-starter-webflux-ui`。

### 2.2 默认访问地址

启动项目后，可通过以下地址访问：

| 资源 | URL |
|------|-----|
| Swagger UI 可视化界面 | `http://localhost:8080/swagger-ui.html` |
| OpenAPI JSON 描述 | `http://localhost:8080/v3/api-docs` |
| OpenAPI YAML 描述 | `http://localhost:8080/v3/api-docs.yaml` |

最简 Controller 示例，启动即可看到文档：

```java
@RestController
@RequestMapping("/api/demo")
public class DemoController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello SpringDoc!";
    }
}
```

---

## 三、核心配置

### 3.1 基础配置（application.yml）

```yaml
springdoc:
  # 扫描包路径（限定范围可提升启动速度）
  packages-to-scan: com.example.controller
  # 匹配指定路径的接口才生成文档
  paths-to-match: /api/**
  swagger-ui:
    # 自定义 Swagger UI 路径
    path: /swagger-ui.html
    # 标签按字母排序
    tags-sorter: alpha
    # 接口按路径排序
    operations-sorter: alpha
    # 禁用 Try-It-Out 按钮（生产环境可关闭）
    try-it-out-enabled: true
  api-docs:
    # 自定义 JSON 描述路径
    path: /v3/api-docs
    # 启用/禁用文档生成（生产建议关闭）
    enabled: true
```

### 3.2 自定义 API 全局信息

通过配置类自定义标题、版本、联系人、License 等元数据：

```java
package com.example.config;

import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Contact;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.info.License;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SpringDocConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("用户管理系统 API")            // API 标题
                .version("1.0.0")                    // 版本号
                .description("用户 CRUD 与权限管理")  // 详细描述
                .contact(new Contact()               // 联系人
                    .name("张三")
                    .email("zhangsan@example.com")
                    .url("https://example.com"))
                .license(new License()               // 许可证
                    .name("Apache 2.0")
                    .url("https://www.apache.org")));
    }
}
```

---

## 四、核心注解详解（由浅入深）

### 4.1 @Tag —— 模块分组（Controller 层）

用于对 Controller 进行分组和描述，在 Swagger UI 中显示为独立标签页。

```java
package com.example.controller;

import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.web.bind.annotation.*;

// name: 标签名称；description: 标签说明
@Tag(name = "用户管理", description = "用户 CRUD 接口")
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping
    public String list() { return "用户列表"; }
}
```

> 💡 多个 Controller 可使用相同 `name` 合并到同一个标签下。

### 4.2 @Operation —— 接口方法描述

标注在方法上，说明接口的用途、业务含义。

```java
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.web.bind.annotation.*;

@Tag(name = "用户管理")
@RestController
@RequestMapping("/api/users")
public class UserController {

    @Operation(
        summary = "根据ID查询用户",       // 简短标题（UI 上显示在方法旁）
        description = "传入用户唯一ID，返回对应用户详情。ID 必须为正整数。"
    )
    @GetMapping("/{id}")
    public String getById(@PathVariable Long id) {
        return "用户ID: " + id;
    }
}
```

### 4.3 @ApiResponse 与 @ApiResponses —— 响应定义

描述接口可能返回的 HTTP 状态码及含义。

```java
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/users")
public class UserController {

    @Operation(summary = "删除用户")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "删除成功"),
        @ApiResponse(responseCode = "400", description = "参数不合法"),
        @ApiResponse(responseCode = "404", description = "用户不存在")
    })
    @DeleteMapping("/{id}")
    public String delete(@PathVariable Long id) {
        return "已删除: " + id;
    }
}
```

### 4.4 @Parameter —— 参数描述

用于描述单个请求参数的含义、是否必填、示例值等。

```java
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/users")
public class UserController {

    @Operation(summary = "分页查询用户列表")
    @GetMapping
    public String list(
        @Parameter(description = "页码，从1开始", required = true, example = "1")
        @RequestParam(defaultValue = "1") Integer pageNum,

        @Parameter(description = "每页条数", example = "10")
        @RequestParam(defaultValue = "10") Integer pageSize,

        @Parameter(description = "用户名模糊搜索", example = "张")
        @RequestParam(required = false) String keyword
    ) {
        return "第" + pageNum + "页，共" + pageSize + "条";
    }
}
```

### 4.5 @Schema —— 实体类/字段描述

用于 DTO/Entity 上，描述类和字段的含义、校验规则、示例值。

```java
package com.example.dto;

import io.swagger.v3.oas.annotations.media.Schema;
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import lombok.Data;

@Data
@Schema(description = "用户创建请求对象")
public class UserCreateDTO {

    @NotBlank(message = "用户名不能为空")
    @Schema(description = "用户名", example = "zhangsan", requiredMode = Schema.RequiredMode.REQUIRED)
    private String username;

    @Email(message = "邮箱格式错误")
    @Schema(description = "邮箱地址", example = "zhangsan@example.com")
    private String email;

    @Schema(description = "年龄", example = "25", minimum = "0", maximum = "150")
    private Integer age;
}
```

Controller 中直接使用 DTO：

```java
@PostMapping
public String create(@RequestBody UserCreateDTO dto) {
    return "创建用户: " + dto.getUsername();
}
```

### 4.6 @Hidden —— 隐藏接口或字段

用于隐藏不想暴露在文档中的接口、Controller 或 DTO 字段。

```java
import io.swagger.v3.oas.annotations.Hidden;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.web.bind.annotation.*;

@Tag(name = "内部接口")
@RestController
@RequestMapping("/internal")
public class InternalController {

    @Hidden           // 文档中不显示此接口
    @GetMapping("/debug")
    public String debug() { return "调试专用"; }

    @GetMapping("/health")
    public String health() { return "OK"; }
}
```

隐藏 DTO 中的敏感字段：

```java
@Data
public class UserRespDTO {
    private Long id;
    private String username;

    @Hidden  // 密码等敏感字段不展示在文档中
    private String password;
}
```

---

## 五、安全认证集成

### 5.1 JWT Bearer Token 配置

在配置类中定义安全方案，Swagger UI 就会出现 🔒 授权按钮。

```java
package com.example.config;

import io.swagger.v3.oas.models.Components;
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.security.SecurityRequirement;
import io.swagger.v3.oas.models.security.SecurityScheme;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SpringDocConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        // 方案名称，全局引用
        final String securitySchemeName = "bearerAuth";

        return new OpenAPI()
            // 全局要求所有接口携带 Token（也可在单个 Controller/方法上用 @SecurityRequirement）
            .addSecurityItem(new SecurityRequirement().addList(securitySchemeName))
            .components(new Components()
                .addSecuritySchemes(securitySchemeName,
                    new SecurityScheme()
                        .name(securitySchemeName)
                        .type(SecurityScheme.Type.HTTP)
                        .scheme("bearer")
                        .bearerFormat("JWT")))
            .info(new Info().title("带 JWT 认证的 API").version("1.0"));
    }
}
```

若只想部分接口启用认证，去掉全局的 `addSecurityItem`，改为在方法上单独标注：

```java
import io.swagger.v3.oas.annotations.security.SecurityRequirement;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api")
public class OrderController {

    @SecurityRequirement(name = "bearerAuth")  // 仅该接口需要 Token
    @GetMapping("/orders")
    public String myOrders() { return "订单列表"; }

    @PostMapping("/login")  // 登录接口不需要 Token
    public String login() { return "返回 Token"; }
}
```

### 5.2 Spring Security 白名单配置

使用 Spring Security 时，必须放行 Swagger 相关路径，否则页面 401。

```java
package com.example.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                // ===== SpringDoc 白名单 START =====
                .requestMatchers(
                    "/v3/api-docs/**",          // OpenAPI JSON
                    "/v3/api-docs.yaml",        // OpenAPI YAML
                    "/swagger-ui/**",           // Swagger UI 静态资源
                    "/swagger-ui.html",         // Swagger UI 入口
                    "/swagger-resources/**",    // 兼容旧版 Swagger
                    "/webjars/**"               // WebJar 资源
                ).permitAll()
                // ===== SpringDoc 白名单 END =====
                .anyRequest().authenticated()
            );
        return http.build();
    }
}
```

---

## 六、高级配置

### 6.1 分组文档（多组 API）

当项目接口较多时，可按业务模块拆分为多组文档，Swagger UI 左上角可切换分组。

```java
package com.example.config;

import org.springdoc.core.models.GroupedOpenApi;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SpringDocGroupConfig {

    // 用户模块组：匹配 /api/user/** 路径
    @Bean
    public GroupedOpenApi userApi() {
        return GroupedOpenApi.builder()
            .group("user-module")            // 分组唯一标识
            .displayName("用户模块")          // UI 显示名
            .pathsToMatch("/api/user/**")    // 匹配路径
            .packagesToScan("com.example.user.controller")
            .build();
    }

    // 订单模块组：匹配 /api/order/** 路径
    @Bean
    public GroupedOpenApi orderApi() {
        return GroupedOpenApi.builder()
            .group("order-module")
            .displayName("订单模块")
            .pathsToMatch("/api/order/**")
            .build();
    }

    // 管理后台组：匹配 /api/admin/**
    @Bean
    public GroupedOpenApi adminApi() {
        return GroupedOpenApi.builder()
            .group("admin-module")
            .displayName("管理后台")
            .pathsToMatch("/api/admin/**")
            .build();
    }
}
```

### 6.2 全局响应与异常处理

通过 `@ControllerAdvice` 统一异常处理，SpringDoc 会自动识别并生成响应结构。

```java
package com.example.exception;

import io.swagger.v3.oas.annotations.media.Schema;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Schema(description = "统一响应结果")
public class R<T> {

    @Schema(description = "状态码 200=成功", example = "200")
    private Integer code;

    @Schema(description = "提示信息", example = "操作成功")
    private String msg;

    @Schema(description = "业务数据")
    private T data;

    public static <T> R<T> ok(T data) {
        return new R<>(200, "操作成功", data);
    }

    public static <T> R<T> fail(Integer code, String msg) {
        return new R<>(code, msg, null);
    }
}
```

全局异常处理：

```java
package com.example.exception;

import io.swagger.v3.oas.annotations.Hidden;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@Hidden  // 避免在文档中暴露此类
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public R<Void> handleValid(MethodArgumentNotValidException e) {
        String msg = e.getBindingResult().getFieldError() != null
            ? e.getBindingResult().getFieldError().getDefaultMessage()
            : "参数校验失败";
        return R.fail(400, msg);
    }

    @ExceptionHandler(Exception.class)
    public R<Void> handleAll(Exception e) {
        return R.fail(500, "服务异常: " + e.getMessage());
    }
}
```

### 6.3 响应式 WebFlux 支持

WebFlux 项目只需要替换依赖，注解使用方式完全一致。

```xml
<!-- 替换 webmvc-ui 为 webflux-ui -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webflux-ui</artifactId>
    <version>2.6.0</version>
</dependency>
```

WebFlux Router 函数式接口生成文档示例：

```java
package com.example.config;

import org.springdoc.core.models.GroupedOpenApi;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.function.RouterFunction;
import org.springframework.web.servlet.function.ServerResponse;

import static org.springframework.web.servlet.function.RouterFunctions.route;
import static org.springframework.web.servlet.function.ServerResponse.ok;

@Configuration
public class WebFluxRouterConfig {

    @Bean
    public RouterFunction<ServerResponse> helloRouter() {
        return route()
            .GET("/func/hello", req -> ok().body("Hello Router!"))
            .build();
    }

    // 分组配置同样可用
    @Bean
    public GroupedOpenApi funcApi() {
        return GroupedOpenApi.builder()
            .group("func")
            .pathsToMatch("/func/**")
            .build();
    }
}
```

### 6.4 自定义 OperationCustomizer

批量给所有接口添加通用参数、通用响应，避免重复写注解。

```java
package com.example.config;

import io.swagger.v3.oas.models.Operation;
import io.swagger.v3.oas.models.media.StringSchema;
import io.swagger.v3.oas.models.parameters.Parameter;
import org.springdoc.core.customizers.OperationCustomizer;
import org.springframework.stereotype.Component;
import org.springframework.web.method.HandlerMethod;

/**
 * 给所有接口自动追加 X-Trace-Id 请求头参数
 */
@Component
public class GlobalOperationCustomizer implements OperationCustomizer {

    @Override
    public Operation customize(Operation operation, HandlerMethod handlerMethod) {
        Parameter traceId = new Parameter()
            .in("header")                        // header 参数
            .name("X-Trace-Id")                  // 参数名
            .description("链路追踪ID（可选）")
            .required(false)
            .schema(new StringSchema().example("abc-123"));

        operation.addParametersItem(traceId);
        return operation;
    }
}
```

---

## 七、生产环境最佳实践

### 7.1 环境隔离与开关控制

**核心原则：生产环境默认关闭 Swagger UI。**

```yaml
spring:
  profiles:
    active: prod  # 激活环境

---
# 开发 / 测试环境：开启文档
spring:
  config:
    activate:
      on-profile: dev
springdoc:
  api-docs:
    enabled: true
  swagger-ui:
    enabled: true
    try-it-out-enabled: true

---
# 生产环境：彻底关闭文档入口
spring:
  config:
    activate:
      on-profile: prod
springdoc:
  api-docs:
    enabled: false
  swagger-ui:
    enabled: false
```

也可以通过启动参数动态控制：

```bash
# 临时开启生产环境文档（紧急调试用，用完立即重启关闭）
java -jar app.jar --spring.profiles.active=prod --springdoc.api-docs.enabled=true --springdoc.swagger-ui.enabled=true
```

### 7.2 包路径与接口过滤

精确控制哪些接口进入文档，避免把内部接口、Actuator 等暴露出去。

```java
package com.example.config;

import org.springdoc.core.models.GroupedOpenApi;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ApiFilterConfig {

    @Bean
    public GroupedOpenApi publicApi() {
        return GroupedOpenApi.builder()
            .group("public")
            // 只扫描业务 Controller 包
            .packagesToScan("com.example.business.controller")
            // 排除内部接口路径
            .pathsToExclude("/internal/**", "/actuator/**", "/error")
            .build();
    }
}
```

### 7.3 性能优化建议

| 优化点 | 做法 |
|--------|------|
| 限定扫描范围 | `packages-to-scan` 精确到业务包，避免扫描整个 classpath |
| 排除 Actuator | 若不需要监控文档，显式排除 `/actuator/**` |
| 生产关闭 | 生产环境把 `springdoc.api-docs.enabled` 设为 `false` |
| 禁用模型解析 | 可选：`springdoc.default-produces-media-type=application/json` 减少解析 |

典型生产配置：

```yaml
springdoc:
  packages-to-scan: com.example.business.controller
  paths-to-match: /api/**
  paths-to-exclude: /actuator/**, /internal/**
  cache:
    disabled: false  # 启用缓存，避免每次请求重新解析
  writer:
    with-default-pretty-printer: false  # 关闭 JSON 格式化，减小响应体积
```

---

## 八、常见问题排查

### 8.1 未生成 API 文档

**现象：** 访问 `/v3/api-docs` 返回 404 或空对象。

**排查步骤：**

1. **检查依赖是否正确**
   ```
   ✅ springdoc-openapi-starter-webmvc-ui（Spring MVC）
   ✅ springdoc-openapi-starter-webflux-ui（WebFlux）
   ❌ 两者不能同时引入
   ```

2. **检查 yml 配置是否误关闭**
   ```yaml
   springdoc:
     api-docs:
       enabled: true   # 确保不是 false
   ```

3. **检查扫描包是否覆盖到 Controller**
   ```yaml
   springdoc:
     packages-to-scan: com.example.controller  # 路径必须正确
   ```
   > 💡 若配置了 `packages-to-scan`，必须包含所有 Controller 包路径。

4. **检查 Spring Boot 版本兼容**
   ```
   Spring Boot 3.x  →  SpringDoc 2.x
   Spring Boot 2.x  →  SpringDoc 1.x
   ```

### 8.2 文档显示错误

**现象：** Swagger UI 能打开，但参数/响应显示乱码、缺失、或加载失败。

**常见原因：**

| 现象 | 原因 | 修复 |
|------|------|------|
| 响应中文乱码 | 未配置 UTF-8 编码 | `spring.messages.encoding=UTF-8` |
| DTO 字段缺失 | 字段无 getter/setter | 加 Lombok `@Data` 或手写 getter |
| 日期类型显示数字 | Jackson 默认时间戳 | 加 `spring.jackson.date-format=yyyy-MM-dd HH:mm:ss` |
| Swagger UI 加载白屏 | 静态资源被拦截 | 检查 Security/拦截器是否放行 `/swagger-ui/**`、`/webjars/**` |
| 枚举显示整数 | 未加 `@Schema` | 在枚举字段加 `@Schema(implementation = MyEnum.class)` |

**日期格式修复示例：**

```java
// 方式1：全局配置（application.yml）
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
    serialization:
      write-dates-as-timestamps: false

// 方式2：单独字段标注
@Schema(type = "string", format = "date-time", example = "2024-01-01 12:00:00")
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
private LocalDateTime createTime;
```

### 8.3 接口调用失败

**现象：** Swagger UI 中点击 "Try it out" 后 Execute 报错 / 401 / 403。

**排查清单：**

1. **接口 401 未认证**
   - 确认是否配置了 JWT 安全方案（见 5.1 节）
   - 点击 Swagger UI 右上角 🔒 **Authorize** 按钮，填入 Bearer Token：
     ```
     Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6...
     ```

2. **接口 403 被 Spring Security 拦截**
   - 确认 Spring Security 白名单已配置（见 5.2 节）
   - 若请求的业务接口需要权限，需要先在 Authorize 中填入 Token

3. **CORS 跨域问题**
   ```java
   // 全局跨域配置
   @Configuration
   public class CorsConfig {
       @Bean
       public WebMvcConfigurer corsConfigurer() {
           return new WebMvcConfigurer() {
               @Override
               public void addCorsMappings(CorsRegistry registry) {
                   registry.addMapping("/**")
                       .allowedOriginPatterns("*")
                       .allowedMethods("*")
                       .allowedHeaders("*")
                       .allowCredentials(true);
               }
           };
       }
   }
   ```

4. **请求路径缺失 Context-Path**
   - 如果项目配置了 `server.servlet.context-path=/api`，Swagger 发出的请求会自动加上，无需手动调整。
   - 若仍错误，检查 Nginx / 网关反向代理是否正确转发了 `X-Forwarded-*` 头：
     ```yaml
     server:
       forward-headers-strategy: native  # Spring Boot 正确识别代理头
     ```
