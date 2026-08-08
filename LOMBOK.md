# Lombok 核心技术手册

  **阅读指南**：由浅入深，每个知识点均配有精简示例。建议按顺序阅读，环环相扣。

---

## 📑 目录

- [**第一部分：Lombok 简介与环境配置**](#第一部分lombok-简介与环境配置)
  - [1.1 Lombok 概述与原理](#11-lombok-概述与原理)
  - [1.2 IDEA 插件安装](#12-idea-插件安装)
  - [1.3 Maven 依赖配置](#13-maven-依赖配置)
- [**第二部分：基础注解（属性访问器）**](#第二部分基础注解属性访问器)
  - [2.1 @Getter 与 @Setter](#21-getter-与-setter)
  - [2.2 @ToString](#22-tostring)
  - [2.3 @EqualsAndHashCode](#23-equalsandhashcode)
  - [2.4 @NonNull](#24-nonnull)
- [**第三部分：构造方法相关注解**](#第三部分构造方法相关注解)
  - [3.1 @NoArgsConstructor](#31-noargsconstructor)
  - [3.2 @RequiredArgsConstructor](#32-requiredargsconstructor)
  - [3.3 @AllArgsConstructor](#33-allargsconstructor)
  - [3.4 三者对比与选择](#34-三者对比与选择)
- [**第四部分：组合与构建注解**](#第四部分组合与构建注解)
  - [4.1 @Data](#41-data)
  - [4.2 @Builder](#42-builder)
  - [4.3 @Value](#43-value)
  - [4.4 @With](#44-with)
- [**第五部分：日志注解**](#第五部分日志注解)
  - [5.1 @Log 家族概览](#51-log-家族概览)
  - [5.2 @Slf4j 实战](#52-slf4j-实战)
- [**第六部分：资源与异常注解**](#第六部分资源与异常注解)
  - [6.1 @Cleanup](#61-cleanup)
  - [6.2 @SneakyThrows](#62-sneakythrows)
- [**第七部分：进阶特性**](#第七部分进阶特性)
  - [7.1 val 与 var](#71-val-与-var)
  - [7.2 @Accessors 链式调用](#72-accessors-链式调用)
  - [7.3 @Singular 集合构建](#73-singular-集合构建)
- [**第八部分：常见陷阱与最佳实践**](#第八部分常见陷阱与最佳实践)
  - [8.1 与 Jackson 序列化兼容](#81-与-jackson-序列化兼容)
  - [8.2 继承体系注意事项](#82-继承体系注意事项)
  - [8.3 最佳实践建议](#83-最佳实践建议)

---

## 第一部分：Lombok 简介与环境配置

### 1.1 Lombok 概述与原理

```
┌─────────────────────────────────────────────────────────────┐
│                    Lombok 核心价值                           │
├─────────────────────────────────────────────────────────────┤
│  ① 消除样板代码：getter/setter/toString/equals 等           │
│  ② 编译期生成：无运行时反射，性能零损耗                      │
│  ③ 提升可读性：注解即契约，类定义回归简洁                    │
│  ④ 工具链友好：IDE/Maven/Gradle 全面支持                    │
└─────────────────────────────────────────────────────────────┘
```

| 核心概念 | 说明 |
|----------|------|
| **编译期注入** | 基于 JSR 269 在 javac 编译阶段修改 AST（抽象语法树） |
| **非运行时** | 字节码与手写一致，不依赖反射，无性能损耗 |
| **scope=provided** | 仅编译期可见，不打包进最终产物 |
| **delombok** | 可反编译查看生成代码，便于调试与排查 |

### 1.2 IDEA 插件安装

**安装步骤：**

1. `File → Settings → Plugins` 搜索 `Lombok` 并安装
2. 重启 IDEA
3. 开启 `Settings → Build → Compiler → Annotation Processors → Enable annotation processing`

> ⚠️ **注意：** IDEA 2020.3 及以上版本已内置 Lombok 插件，无需手动安装，但仍需开启注解处理器。

### 1.3 Maven 依赖配置

```xml
<!-- Lombok 核心依赖：仅编译期有效，不传递 -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.30</version>
    <scope>provided</scope> <!-- 关键：不打包进产物 -->
</dependency>
```

**验证生成代码：**

```bash
javap -p target/classes/com/example/User.class  # 查看Lombok生成的方法
```

---

## 第二部分：基础注解（属性访问器）

### 2.1 @Getter 与 @Setter

为字段自动生成 getter/setter，可作用于**类或字段**。

```java
@Getter @Setter                       // 类级别：所有非静态字段生效
public class User {
    private String name;
    @Setter(AccessLevel.PROTECTED)    // 单独控制某字段的访问级别
    private Integer age;
    @Getter(AccessLevel.NONE)         // 排除该字段，不生成getter
    private String password;
}
```

**常用参数：**

| 参数 | 作用 | 取值 |
|------|------|------|
| `value` | 访问级别 | PUBLIC / PROTECTED / PACKAGE / PRIVATE / NONE |
| `lazy` | 懒加载（仅 @Getter） | true：需配合 final 字段，计算成本高时使用 |
| `onMethod_` / `onParam_` | 给生成的方法/参数加注解 | 如 `onMethod_=@JsonAlias("u")` |

### 2.2 @ToString

自动生成 `toString()`，便于日志输出与调试。

```java
@ToString                          // 默认包含所有非静态字段
@ToString(exclude = {"password"})  // 排除敏感字段
@ToString(callSuper = true)        // 输出包含父类字段
public class User { ... }
```

| 参数 | 说明 | 示例 |
|------|------|------|
| `of` | 仅包含指定字段 | `of = {"id","name"}` |
| `exclude` | 排除指定字段 | `exclude = "password"` |
| `callSuper` | 是否调用父类 toString | `true`（继承时必加） |
| `includeFieldNames` | 是否显示字段名 | `false` 时只输出值 |

### 2.3 @EqualsAndHashCode

自动生成 `equals()` 和 `hashCode()`，遵循 Object 契约。

```java
@EqualsAndHashCode                       // 默认所有非静态字段参与
@EqualsAndHashCode(of = "id")            // 仅按 id 判断相等（JPA实体常用）
@EqualsAndHashCode(callSuper = true)     // 父类字段也参与判断
public class User { ... }
```

> ⚠️ **JPA/Hibernate 实体警告：** 默认 `callSuper=false`，Lombok 会提示警告。实体类建议使用 `of="id"` 或手写 equals/hashCode，避免懒加载触发 N+1 查询。

### 2.4 @NonNull

自动生成空值校验，方法入口抛出 `NullPointerException`。

```java
public class OrderService {
    // 方法开头自动插入空校验
    public void process(@NonNull String orderId) {
        System.out.println(orderId);
    }
}
```

**作用位置：**

| 位置 | 行为 |
|------|------|
| **方法参数** | 方法入口处插入 `if (x == null) throw new NullPointerException("x");` |
| **字段** | 在 setter / 构造方法中插入空校验 |

> 💡 与 Spring 的 `@NotNull`（JSR303）区别：Lombok 在**方法体内**校验，JSR303 在**调用前**由校验器校验，二者可配合使用。

---

## 第三部分：构造方法相关注解

### 3.1 @NoArgsConstructor

生成**无参构造方法**，JavaBean 规范要求。

```java
@NoArgsConstructor(force = true)   // force: final字段初始化为默认值
public class User {
    private final String id = "0"; // force 使 final 字段可被无参构造
}
```

| 参数 | 说明 |
|------|------|
| `force` | 为 final 字段赋默认值（0/false/null），使其支持无参构造 |
| `access` | 控制构造方法访问级别 |

### 3.2 @RequiredArgsConstructor

生成包含**所有 final 字段和 @NonNull 字段**的构造方法，是 Spring 依赖注入的**推荐方式**。

```java
@RequiredArgsConstructor
@Service
public class UserService {
    private final UserRepository repo;   // 通过构造器注入，Spring官方推荐
    private final RedisTemplate<String,Object> redis;
    @NonNull private String cachePrefix; // @NonNull字段也会被加入
}
```

> 💡 **Spring 推荐：** 构造器注入优于 `@Autowired` 字段注入。`final` 字段保证不可变，配合此注解可彻底告别 `@Autowired`。

### 3.3 @AllArgsConstructor

生成**包含所有字段**的构造方法。

```java
@AllArgsConstructor
public class User {
    private String name;
    private Integer age;
}
// 生成：User(String name, Integer age)
```

### 3.4 三者对比与选择

| 注解 | 包含字段 | 典型场景 | 推荐度 |
|------|---------|---------|--------|
| `@NoArgsConstructor` | 无 | JavaBean 规范 / JPA 实体 / JSON 反序列化 | ⭐⭐⭐⭐⭐ |
| `@RequiredArgsConstructor` | final + @NonNull | Spring 依赖注入 / 不可变依赖 | ⭐⭐⭐⭐⭐ |
| `@AllArgsConstructor` | 所有字段 | DTO / Builder 模式基础 | ⭐⭐⭐ |

> 💡 **黄金组合：** `@Data + @NoArgsConstructor + @AllArgsConstructor` 是 DTO 的经典写法。

---

## 第四部分：组合与构建注解

### 4.1 @Data

**组合注解**，相当于一次性应用以下注解：

```
@Getter + @Setter + @ToString + @EqualsAndHashCode + @RequiredArgsConstructor
```

```java
@Data
public class UserDTO {
    private Long id;
    private String name;
    private Integer age;
}
```

| 适用场景 | 注意事项 |
|---------|---------|
| POJO / DTO / VO | ✅ 推荐 |
| JPA 实体 | ⚠️ 慎用，equals/hashCode 可能触发懒加载 |
| 不可变对象 | ❌ 改用 @Value |

### 4.2 @Builder

实现**建造者模式**，链式构造复杂对象，参数可任意顺序、可省略。

```java
@Builder
@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
    @Builder.Default private Integer status = 1; // 默认值
}

// 使用：链式调用，可省略任意字段
User u = User.builder().id(1L).name("Tom").age(20).build();
```

| 参数 | 说明 |
|------|------|
| `builderMethodName` | Builder 工厂方法名，默认 `builder` |
| `buildMethodName` | 构建方法名，默认 `build` |
| `toBuilder` | 是否生成 `toBuilder()`，从已有对象克隆并修改 |
| `@Builder.Default` | 字段默认值（Builder 未设置时生效） |

### 4.3 @Value

**不可变对象**版本，相当于 `final @Data`，所有字段隐式 `final` 且无 setter。

```java
@Value
public class Config {
    String env;      // 隐式 final
    Integer timeout; // 隐式 final
}
// 生成：全参构造 + getter + equals + hashCode + toString，无 setter
```

> 💡 适用场景：配置类、值对象、多线程共享的不可变数据。JDK14+ 的 `record` 是其官方替代方案。

### 4.4 @With

生成**克隆并修改单字段**的方法，返回新对象，适合不可变对象。

```java
@Value
public class Config {
    String env;
    @With Integer timeout; // 生成 withTimeout(Integer) 方法
}

Config c1 = new Config("prod", 100);
Config c2 = c1.withTimeout(200); // 返回新对象，原对象不变
```

> 💡 与 setter 区别：setter **修改原对象**（可变），`@With` **返回新对象**（不可变），是函数式编程的推荐方式。

---

## 第五部分：日志注解

### 5.1 @Log 家族概览

自动注入日志对象，免去手动声明 `Logger` 字段。

| 注解 | 生成的字段 | 适用日志框架 |
|------|-----------|------------|
| `@Log` | `java.util.logging.Logger` | JDK Log |
| `@CommonsLog` | `org.apache.commons.logging.Log` | Apache Commons Logging |
| `@Log4j` | `org.apache.log4j.Logger` | Log4j 1.x |
| `@Log4j2` | `org.apache.logging.log4j.Logger` | Log4j 2 |
| **`@Slf4j`** | `org.slf4j.Logger` | **SLF4J（推荐）** |
| `@XSlf4j` | `org.slf4j.ext.XLogger` | XSlf4j 扩展 |

### 5.2 @Slf4j 实战

```java
@Slf4j                          // 自动注入：private static final Logger log = ...
@RestController
public class UserController {

    @GetMapping("/users/{id}")
    public User get(@PathVariable Long id) {
        log.info("查询用户 id={}", id);   // 使用占位符，避免字符串拼接
        try {
            return userService.getById(id);
        } catch (Exception e) {
            log.error("查询失败 id={}", id, e); // 异常对象作为最后一个参数
            throw e;
        }
    }
}
```

> 💡 **生产规范：** 必须使用 `{}` 占位符，避免无谓的字符串拼接开销；异常堆栈作为最后一个参数传入即可打印完整栈。

---

## 第六部分：资源与异常注解

### 6.1 @Cleanup

自动调用 `close()` 释放资源，等同于 try-with-resources。

```java
public String read(String path) throws IOException {
    @Cleanup InputStream in = new FileInputStream(path); // 作用域结束自动 close()
    @Cleanup BufferedReader br = new BufferedReader(new InputStreamReader(in));
    return br.readLine();
}
```

| 参数 | 说明 |
|------|------|
| `value` | 指定清理方法名，默认 `close`。如 `@Cleanup("dispose")` |

> ⚠️ **慎用：** 无法处理异常传播，且 JDK7+ 的 try-with-resources 是更标准的方案。仅在遗留代码中使用。

### 6.2 @SneakyThrows

**偷抛受检异常**，绕过 `throws` 声明，适合不想处理受检异常的场景。

```java
@SneakyThrows                        // 无需声明 throws InterruptedException
public void delay() {
    Thread.sleep(1000);              // 受检异常被"偷抛"出去
}
```

**原理：** 通过字节码技巧（`Lombok.sneakyThrow()`）抛出，绕过编译器的受检异常检查。

> ⚠️ **慎用：** 破坏 Java 异常契约，调用方无法感知受检异常。仅建议用于：
> - 不应发生的受检异常（如 `CloneNotSupportedException`）
> - Lambda/Stream 中无法声明 throws 的场景

---

## 第七部分：进阶特性

### 7.1 val 与 var

局部变量类型自动推断，简化声明。

```java
val list = new ArrayList<String>();   // 等价 final List<String> list
var user = new User();                 // 可变，类型推断为 User
list.add("hello");                     // ✅ 可调用方法
// list = new ArrayList<>();          // ❌ val 不可重新赋值
```

| 关键字 | 可变性 | 等价写法 |
|--------|--------|---------|
| `val` | 不可变（final） | `final List<String> list` |
| `var` | 可变 | `List<String> list` |

> 💡 JDK10+ 已内置 `var`，Lombok 的 `var` 主要用于低版本兼容；`val` 仍是 Lombok 独有。

### 7.2 @Accessors 链式调用

让 setter 返回 `this`，实现链式调用。

```java
@Accessors(chain = true)              // setter 返回 this
@Data
public class User {
    private String name;
    private Integer age;
}

// 链式设置
User u = new User().setName("Tom").setAge(20);
```

| 参数 | 说明 | 效果 |
|------|------|------|
| `chain` | setter 返回 this | `setName()` 返回 `User` |
| `fluent` | 方法名去掉 set/get 前缀 | `u.name("Tom").age(20)` |
| `prefix` | 去除字段前缀生成方法名 | 字段 `mName` → 方法 `setName()` |

> 💡 `fluent=true` 常与 `@Builder` 配合，生成风格统一的流式 API。

### 7.3 @Singular 集合构建

配合 `@Builder` 生成集合的**单元素追加方法**与**批量设置方法**。

```java
@Builder
public class Team {
    @Singular private List<String> members;
}

// 单个追加 + 批量设置并存
Team t = Team.builder()
    .member("Tom")                   // 单元素追加（singular方法名）
    .member("Jerry")
    .members(List.of("A", "B"))      // 批量设置（复数方法名）
    .build();
```

> 💡 **设计意图：** Builder 模式中集合字段用 `@Singular`，可灵活地一个个追加，而不必一次性构造完整集合。

---

## 第八部分：常见陷阱与最佳实践

### 8.1 与 Jackson 序列化兼容

**问题：** `@Data` + `@AllArgsConstructor` 缺少无参构造，导致 Jackson 反序列化失败。

```java
@Data
@NoArgsConstructor            // 反序列化必须！Jackson 需要无参构造
@AllArgsConstructor
public class UserDTO {
    private Long id;
    private String name;
}
```

**Builder 场景：** 使用 `@Jacksonized` 替代 `@Builder`，自动配置 Jackson 使用 Builder 反序列化。

```java
@Data @Jacksonized @Builder
public class UserDTO { ... }  // Jackson 优先通过 Builder 反序列化
```

### 8.2 继承体系注意事项

**问题：** 子类默认 `callSuper=false`，会忽略父类字段，导致 equals/hashCode/toString 不完整。

```java
@EqualsAndHashCode(callSuper = true)  // 父类字段参与判断
@ToString(callSuper = true)            // 父类字段输出
@Data
public class Admin extends User {
    private String role;
}
```

| 注解 | 默认值 | 继承时建议 |
|------|--------|----------|
| `@EqualsAndHashCode` | `callSuper=false` | 显式设为 `true` |
| `@ToString` | `callSuper=false` | 显式设为 `true` |
| `@Getter/@Setter` | 不涉及父类 | 父类字段需父类自行注解 |

### 8.3 最佳实践建议

```
┌─────────────────────────────────────────────────────────────┐
│              Lombok 使用决策树（按场景选择）                  │
├─────────────────────────────────────────────────────────────┤
│  ① DTO / VO / POJO        → @Data + @NoArgsConstructor     │
│  ② Spring 服务/组件        → @RequiredArgsConstructor       │
│  ③ 复杂对象构建            → @Builder + @Builder.Default    │
│  ④ 不可变配置/值对象       → @Value 或 record               │
│  ⑤ 日志                    → @Slf4j                         │
│  ⑥ JPA 实体                → @Getter/@Setter + 手写 equals  │
└─────────────────────────────────────────────────────────────┘
```

**核心规范：**

1. **DTO 必加 `@NoArgsConstructor`**，否则 Jackson/JSON 反序列化失败
2. **依赖注入用 `@RequiredArgsConstructor`**，字段声明 `final`，告别 `@Autowired`
3. **生产环境关闭 `@ToString` 敏感字段**（password / token / idCard）
4. **JPA 实体避免 `@Data`**，equals/hashCode 触发懒加载，改用 `@Getter @Setter`
5. **慎用 `@SneakyThrows`**，破坏异常契约，仅用于不应发生的受检异常
6. **慎用 `@Cleanup`**，优先使用 JDK7+ 的 try-with-resources
7. **继承时显式设置 `callSuper=true`**，避免遗漏父类字段

---

> **小结：** Lombok 通过编译期代码生成，让 Java 回归简洁。核心掌握 `@Data` / `@Builder` / `@RequiredArgsConstructor` / `@Slf4j` 四件套，配合场景化选择不可变（`@Value`）与链式（`@Accessors`），即可覆盖 90% 日常需求。陷阱集中在序列化、继承、JPA 三个方向，遵循本章规范即可规避。
