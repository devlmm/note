# Java 项目架构体系（由浅入深，环环相扣）

> **阅读指南**：由浅入深，每个知识点均配有精简示例。建议按顺序阅读，环环相扣。

## 📑 目录

- [**第一部分：多单体模块架构**](#第一部分多单体模块架构)
  - [1.1 架构概述与适用场景](#11-架构概述与适用场景)
  - [1.2 标准模块划分](#12-标准模块划分)
  - [1.3 三层架构与两层实现](#13-三层架构与两层实现)
  - [1.4 典型工程示例](#14-典型工程示例)
  - [1.5 前后端分离架构](#15-前后端分离架构)
  - [1.6 核心设计原则与注意事项](#16-核心设计原则与注意事项)
  - [1.7 POM 配置参考](#17-pom-配置参考)
- [**第二部分：微服务架构**](#第二部分微服务架构)
  - [（预留：新大团队项目采用）](#预留新大团队项目采用)
- [**第三部分：微核插件架构**](#第三部分微核插件架构)
  - [（预留：纯基础研发项目采用）](#预留纯基础研发项目采用)

---

## 第一部分：多单体模块架构

### 1.1 架构概述与适用场景

```
┌─────────────────────────────────────────────────────────────┐
│                    多单体模块架构核心思想                     │
├─────────────────────────────────────────────────────────────┤
│  • 新项目小团队不建议上微服务，应采用多单体模块架构             │
│  • 按业务领域拆分模块，每个模块独立开发、独立部署               │
│  • 通过 Maven 聚合+继承实现版本统一管理                       │
│  • 模块间通过 HTTP Restful 或 MQ 事件驱动通信                 │
│  • 兼顾了代码复用、团队协作和部署灵活性                       │
└─────────────────────────────────────────────────────────────┘
```

| 维度 | 说明 |
|------|------|
| **适用场景** | 新项目、小团队（3-10人）、业务复杂度中等 |
| **核心优势** | 模块解耦、代码复用、部署灵活、学习成本低 |
| **拆分原则** | 按业务领域拆分，高内聚低耦合 |
| **通信方式** | HTTP Restful（B端）/ MQ 事件驱动（C端） |
| **部署方式** | 每个聚合模块独立部署，可按需水平扩展 |

```
演进路径：
单模块单体 → 多模块单体 → 多单体模块 → 微服务
   ①           ②            ③         ④
```

### 1.2 标准模块划分

```
父顶层 pom（约束版本，packaging = pom）
├── Common 公共模块           ← 标准父类、工具类、常量
├── Plugin 基础设施模块        ← Redis、MQ、OSS、Excel 等配置
├── 业务模块 1                 ← 领域业务逻辑（无启动类）
├── 业务模块 2                 ← 领域业务逻辑（无启动类）
├── 业务模块 N                 ← 领域业务逻辑（无启动类）
├── 独立启动聚合模块（B端）     ← Controller + Service，聚合业务API
└── 独立启动聚合模块（C端）     ← Controller + Service，聚合业务API
```

| 模块类型 | 职责 | 启动类 | 依赖方向 |
|----------|------|--------|----------|
| **父顶层 pom** | 版本统一管理、依赖管理 | ❌ | 被所有模块继承 |
| **Common 公共模块** | Response、Request、BaseController、Result、ErrorCode | ❌ | 被所有模块依赖 |
| **Plugin 基础设施模块** | Redis、MQ、OSS、Excel 等中间件配置 | ❌ | 被业务模块依赖 |
| **业务模块** | Service、Mapper、领域模型 | ❌ | 依赖 Common、Plugin |
| **聚合模块（B端）** | Controller、Service 聚合、启动类 | ✅ | 依赖业务模块 |
| **聚合模块（C端）** | Controller、Service 聚合、启动类 | ✅ | 依赖业务模块 |

```
依赖方向（不可反向）：
聚合模块 → 业务模块 → Plugin → Common
          业务模块之间互不依赖
          聚合模块之间互不依赖
```

### 1.3 三层架构与两层实现

```
标准三层架构：
┌─────────────┐
│ Controller  │  ← 控制层：接收请求、参数校验、响应封装
├─────────────┤
│  Service    │  ← 业务层：核心业务逻辑、事务控制
├─────────────┤
│   Mapper    │  ← 数据访问层：SQL 执行、数据库交互
└─────────────┘

实际 package 分层（两层实现）：
┌─────────────────────────────┐
│ controller / service        │  ← 聚合模块中
├─────────────────────────────┤
│ service / mapper            │  ← 业务模块中
└─────────────────────────────┘
```

```
业务模块结构示例（xxx-service 模块）：
src/main/java/com/example/xxx/
├── service/
│   ├── XxxService.java          ← 接口
│   └── impl/
│       └── XxxServiceImpl.java   ← 实现类
├── mapper/
│   ├── XxxMapper.java           ← 接口
│   └── xml/
│       └── XxxMapper.xml        ← XML映射
└── entity/
    └── Xxx.java                 ← 实体类

聚合模块结构示例（xxx-admin 模块）：
src/main/java/com/example/xxx/
├── controller/
│   └── XxxController.java       ← 控制层
└── service/
    ├── XxxAggService.java       ← 聚合业务接口
    └── impl/
        └── XxxAggServiceImpl.java ← 聚合业务实现
```

```java
// 业务模块：Service 接口定义
public interface UserService {
    User getById(Long id);
}

// 业务模块：Service 实现（含 Mapper 注入）
@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private UserMapper userMapper;  // ✅ 业务模块内：Service → Mapper
    
    @Override
    public User getById(Long id) {
        return userMapper.selectById(id);
    }
}

// 聚合模块：Controller
@RestController
@RequestMapping("/admin/user")
public class UserController {
    @Autowired
    private UserService userService;  // ✅ 聚合模块：Controller → 业务Service
    
    @GetMapping("/{id}")
    public Result<User> getById(@PathVariable Long id) {
        return Result.success(userService.getById(id));
    }
}

// ❌ 禁止：Mapper 之间相互注入（避免循环依赖）
// ❌ 禁止：业务模块 Service 之间相互注入（在聚合层联调）
```

### 1.4 典型工程示例

#### 1.4.1 B端鉴权中心后端工程

```
auth-parent/（父顶层 pom）
├── auth-common/          ← 公共模块（Response、Result、ErrorCode等）
├── auth-plugin/          ← 基础设施模块（Redis、MQ、OSS等）
├── auth-service/         ← 鉴权业务模块
│   ├── service/          ← 业务逻辑
│   └── mapper/           ← 数据访问
├── auth-admin/           ← B端聚合启动模块
│   ├── controller/       ← B端接口
│   └── service/          ← B端聚合业务
│   └── 启动类            ← https://api.role.xxx.com/admin
└── auth-app/             ← C端聚合启动模块
    ├── controller/       ← C端接口
    └── service/          ← C端聚合业务
    └── 启动类            ← https://api.role.xxx.com/app
```

#### 1.4.2 CRM 后端工程

```
crm-parent/（父顶层 pom）
├── crm-common/           ← 公共模块
├── crm-plugin/           ← 基础设施模块
├── crm-customer/         ← 客户管理业务模块
├── crm-sales/            ← 销售管理业务模块
├── crm-report/           ← 报表管理业务模块
├── crm-admin/            ← B端聚合启动模块
│   └── 启动类            ← https://api.crm.xxx.com/admin
└── crm-app/              ← B端App聚合启动模块
    └── 启动类            ← https://api.crm.xxx.com/app
```

> **注意**：CRM 的 Admin 和 App 聚合模块通过 HTTP Restful 方式调用 CRM 各业务模块的 API

#### 1.4.3 电商互联网后端工程

```
shop-parent/（父顶层 pom）
├── shop-common/          ← 公共模块
├── shop-plugin/          ← 基础设施模块
├── shop-product/         ← 商品管理业务模块
├── shop-order/           ← 订单管理业务模块
├── shop-content/         ← 内容管理业务模块
├── shop-admin/           ← B端聚合启动模块
│   └── 启动类            ← https://api.shop.xxx.com/admin
└── shop-app/             ← C端聚合启动模块
    └── 启动类            ← https://api.shop.xxx.com/app
```

### 1.5 前后端分离架构

```
┌─────────────────────────────────────────────────────────────┐
│                      前后端分离架构                           │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  前端层：                                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │ Admin 管理后台 │  │  PC 网站终端  │  │  H5/App 终端  │    │
│  │ admin.xxx.com │  │ shop.xxx.com │  │ app.xxx.com  │    │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘    │
│         │                  │                  │             │
│         └──────────────────┼──────────────────┘             │
│                            │  HTTP Restful                  │
│  ┌─────────────────────────┼─────────────────────────┐    │
│  │  API 网关 / Nginx 负载均衡  │                          │    │
│  └─────────────────────────┼─────────────────────────┘    │
│                            │                                │
│  后端层：                   │                                │
│  ┌──────────────┐  ┌───────┴──────┐  ┌──────────────┐    │
│  │ 鉴权中心服务   │  │  CRM 服务    │  │  电商服务     │    │
│  │ api.role.xxx  │  │ api.crm.xxx  │  │ api.shop.xxx │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

| 前端工程 | 访问地址 | 说明 |
|----------|----------|------|
| **Admin 管理后台** | https://admin.xxx.com/ | 统一的企业后台管理系统，B端统一鉴权 |
| **PC 网站终端** | https://shop.xxx.com/ | 直接访问对应系统的聚合模块 API |
| **H5/App 终端** | https://app.xxx.com/ | 直接访问对应系统的 C端聚合模块 API |

```
Admin 管理后台特点：
• 统一的企业后台管理系统页面（B端统一鉴权）
• 不同业务系统都在一个菜单内管理
• 不同业务系统访问不同的系统 Http Restfull 接口

网站/App 终端特点：
• 直接访问对应系统的独立启动聚合模块暴露的 Http Restfull 接口
• 无需统一后台菜单，直接面向终端用户
```

### 1.6 核心设计原则与注意事项

| 编号 | 原则 | 说明 |
|------|------|------|
| **a** | **鉴权分离** | B端鉴权、C端鉴权，对应不同的权限体系表，但在同一个数据库 |
| **b** | **Mapper 独立** | Mapper 与 Mapper 不能相互依赖注入调用，数据联调应在 Service 层，避免循环依赖 |
| **c** | **Service 独立** | 多个业务模块之间的 Service 不能相互依赖注入，数据联调应在聚合模块的 Service 层 |
| **d** | **B端联调方式** | B端企业管理工程的系统之间调用，可通过 HTTP Restful 或 MQ 事件驱动联调 |
| **e** | **C端联调方式** | C端互联网工程的系统之间调用与削峰，都可用 MQ 事件驱动联调 |
| **f** | **分表分库** | 任何工程项目可自行决定是否分表分库 |
| **g** | **负载均衡** | 任何工程项目可自行决定是否部署多个服务且前置负载均衡 |

```
循环依赖规避策略：

❌ 错误示例（业务模块间循环依赖）：
  UserService → OrderService → UserService （死循环！）

✅ 正确示例（在聚合层联调）：
  UserAggService（聚合层）
    ├── 注入 UserService（业务模块1）
    ├── 注入 OrderService（业务模块2）
    └── 在聚合方法内组装调用结果
```

```java
// ✅ 正确：在聚合模块的 Service 中联调多个业务模块
@Service
public class UserAggServiceImpl implements UserAggService {
    @Autowired
    private UserService userService;       // 业务模块1
    @Autowired
    private OrderService orderService;     // 业务模块2
    
    @Override
    public UserOrderVO getUserWithOrders(Long userId) {
        User user = userService.getById(userId);           // 调业务模块1
        List<Order> orders = orderService.getByUserId(userId); // 调业务模块2
        // 在聚合层组装结果
        UserOrderVO vo = new UserOrderVO();
        vo.setUser(user);
        vo.setOrders(orders);
        return vo;
    }
}
```

### 1.7 POM 配置参考

#### 1.7.1 父顶层 pom

```xml
<!-- parent/pom.xml -->
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>demo-parent</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>  <!-- 聚合工程必须是pom -->
    
    <!-- 模块列表 -->
    <modules>
        <module>demo-common</module>
        <module>demo-plugin</module>
        <module>demo-service</module>
        <module>demo-admin</module>
        <module>demo-app</module>
    </modules>
    
    <!-- 版本属性统一管理 -->
    <properties>
        <java.version>11</java.version>
        <spring-boot.version>2.7.18</spring-boot.version>
        <mybatis-plus.version>3.5.3.1</mybatis-plus.version>
        <hutool.version>5.8.20</hutool.version>
    </properties>
    
    <!-- 依赖管理（子模块继承时无需指定version） -->
    <dependencyManagement>
        <dependencies>
            <!-- Spring Boot 统一版本 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!-- 内部模块版本 -->
            <dependency>
                <groupId>com.example</groupId>
                <artifactId>demo-common</artifactId>
                <version>${project.version}</version>
            </dependency>
            <dependency>
                <groupId>com.example</groupId>
                <artifactId>demo-plugin</artifactId>
                <version>${project.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

#### 1.7.2 公共模块 pom

```xml
<!-- demo-common/pom.xml -->
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>demo-parent</artifactId>
        <version>1.0.0</version>
    </parent>
    <artifactId>demo-common</artifactId>
    
    <dependencies>
        <!-- 通用工具包 -->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
        </dependency>
        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
</project>
```

#### 1.7.3 基础设施模块 pom

```xml
<!-- demo-plugin/pom.xml -->
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>demo-parent</artifactId>
        <version>1.0.0</version>
    </parent>
    <artifactId>demo-plugin</artifactId>
    
    <dependencies>
        <!-- 依赖公共模块 -->
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>demo-common</artifactId>
        </dependency>
        <!-- Redis -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <!-- MQ -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
    </dependencies>
</project>
```

#### 1.7.4 业务模块 pom

```xml
<!-- demo-service/pom.xml -->
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>demo-parent</artifactId>
        <version>1.0.0</version>
    </parent>
    <artifactId>demo-service</artifactId>
    
    <dependencies>
        <!-- 依赖基础设施模块 -->
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>demo-plugin</artifactId>
        </dependency>
        <!-- MyBatis Plus -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </dependency>
    </dependencies>
</project>
```

#### 1.7.5 聚合启动模块 pom

```xml
<!-- demo-admin/pom.xml -->
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>demo-parent</artifactId>
        <version>1.0.0</version>
    </parent>
    <artifactId>demo-admin</artifactId>
    
    <dependencies>
        <!-- 依赖业务模块 -->
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>demo-service</artifactId>
        </dependency>
        <!-- Web 支持 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <!-- Spring Boot 打包插件（可执行jar） -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## 第二部分：微服务架构

### （预留：新大团队项目采用）

> 待补充内容

---

## 第三部分：微核插件架构

### （预留：纯基础研发项目采用）

> 待补充内容

---

**总结**：多单体模块架构是小团队新项目的最佳实践，通过 Maven 聚合+继承实现版本统一，按业务领域拆分模块，在聚合层进行业务联调，有效避免循环依赖。当团队规模和业务复杂度增长时，可平滑演进到微服务架构。