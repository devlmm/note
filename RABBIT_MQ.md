# RabbitMQ 核心技术知识手册

> 本文档由浅入深、环环相扣地梳理 RabbitMQ 的核心知识体系，每个知识点均配有精简代码示例与注释，便于快速上手与查漏补缺。

---

## 目录

- [1. RabbitMQ 概念](#1-rabbitmq-概念)
  - [1.1 MQ 概念](#11-mq-概念)
  - [1.2 MQ 的优势](#12-mq-的优势)
  - [1.3 MQ 的劣势](#13-mq-的劣势)
  - [1.4 MQ 的应用场景](#14-mq-的应用场景)
  - [1.5 AMQP 协议](#15-amqp-协议)
  - [1.6 RabbitMQ 工作原理](#16-rabbitmq-工作原理)
- [2. RabbitMQ 安装](#2-rabbitmq-安装)
  - [2.1 安装 Erlang](#21-安装-erlang)
  - [2.2 安装 RabbitMQ](#22-安装-rabbitmq)
  - [2.3 启动 RabbitMQ](#23-启动-rabbitmq)
  - [2.4 账户管理](#24-账户管理)
  - [2.5 管控台](#25-管控台)
  - [2.6 Docker 安装](#26-docker-安装)
- [3. RabbitMQ 工作模式](#3-rabbitmq-工作模式)
  - [3.1 简单模式](#31-简单模式)
  - [3.2 工作队列模式](#32-工作队列模式)
  - [3.3 发布订阅模式](#33-发布订阅模式)
  - [3.4 路由模式](#34-路由模式)
  - [3.5 通配符模式](#35-通配符模式)
- [4. SpringBoot 整合 RabbitMQ](#4-springboot-整合-rabbitmq)
  - [4.1 项目搭建](#41-项目搭建)
  - [4.2 创建队列和交换机](#42-创建队列和交换机)
  - [4.3 编写生产者](#43-编写生产者)
  - [4.4 编写消费者](#44-编写消费者)
- [5. 消息的可靠性投递](#5-消息的可靠性投递)
  - [5.1 概念](#51-概念)
  - [5.2 确认模式](#52-确认模式)
  - [5.3 退回模式](#53-退回模式)
  - [5.4 Ack 自动签收](#54-ack-自动签收)
  - [5.5 Ack 手动签收](#55-ack-手动签收)
- [6. RabbitMQ 高级特性](#6-rabbitmq-高级特性)
  - [6.1 消费端限流](#61-消费端限流)
  - [6.2 公平分发](#62-公平分发)
  - [6.3 限流实现不公平分发](#63-限流实现不公平分发)
  - [6.4 设置队列所有消息存活时间](#64-设置队列所有消息存活时间)
  - [6.5 设置单条消息存活时间](#65-设置单条消息存活时间)
  - [6.6 优先级队列](#66-优先级队列)
- [7. RabbitMQ 死信队列](#7-rabbitmq-死信队列)
  - [7.1 概念](#71-概念)
  - [7.2 创建死信队列](#72-创建死信队列)
  - [7.3 测试死信队列](#73-测试死信队列)
- [8. RabbitMQ 延迟队列](#8-rabbitmq-延迟队列)
  - [8.1 概念](#81-概念)
  - [8.2 死信队列实现](#82-死信队列实现)
  - [8.3 安装延迟队列插件](#83-安装延迟队列插件)
  - [8.4 使用插件实现延迟队列](#84-使用插件实现延迟队列)
- [9. RabbitMQ 集群](#9-rabbitmq-集群)
  - [9.1 搭建集群](#91-搭建集群)
  - [9.2 镜像队列](#92-镜像队列)
  - [9.3 负载均衡](#93-负载均衡)

---

## 1. RabbitMQ 概念

### 1.1 MQ 概念

**MQ（Message Queue，消息队列）** 是一种"先进先出"的中间件，用于在分布式系统中保存和传递消息。

- **生产者** 将消息发送到队列，**消费者** 从队列中取出消息处理。
- 起到 **"缓冲、解耦、异步"** 三大核心作用。

```
Producer -> [ Message Queue ] -> Consumer
```

> 本质：把"同步调用"转换为"异步消息"，让生产者与消费者互不阻塞。

### 1.2 MQ 的优势

| 优势 | 说明 |
| --- | --- |
| **解耦** | 生产者无需知道消费者是谁，系统间通过消息交互，降低耦合度 |
| **异步** | 耗时操作丢入队列立即返回，主流程不必等待，提升响应速度 |
| **削峰** | 高并发请求先入队列排队，消费端按自身能力消费，保护下游系统 |
| **顺序性** | 单队列单消费者可保证消息顺序处理 |
| **可靠性** | 持久化 + ACK 机制，保证消息不丢失 |

**典型削峰示例：** 秒杀场景下，10 万请求先入队列，下游订单系统按 1000/s 消费，避免数据库被压垮。

### 1.3 MQ 的劣势

| 劣势 | 说明 |
| --- | --- |
| **系统复杂度增加** | 引入 MQ 需要保证高可用、维护队列，运维成本上升 |
| **可用性降低** | MQ 宕机会导致整条链路瘫痪，需做集群保证可用性 |
| **一致性问题** | 消息消费失败可能导致数据不一致，需引入事务/对账机制 |
| **延迟增加** | 相比同步直接调用，多了一次入队/出队，存在网络与处理延迟 |

> 取舍原则：低延迟、强一致场景慎用 MQ；高吞吐、可异步、可最终一致场景适合 MQ。

### 1.4 MQ 的应用场景

| 场景 | 说明 | 典型例子 |
| --- | --- | --- |
| 异步处理 | 注册后发送短信/邮件，主流程不等 | 用户注册 |
| 应用解耦 | 订单系统与库存系统通过消息交互 | 下单扣库存 |
| 流量削峰 | 秒杀/抢购请求入队排队 | 电商秒杀 |
| 日志收集 | 各服务日志统一投递到 MQ 再落地 | ELK 日志系统 |
| 任务分发 | 多个 worker 处理同一批任务 | 图片转码、视频处理 |
| 远程过程调用 | RPC 调用，借助 reply 队列返回结果 | 微服务调用 |

### 1.5 AMQP 协议

**AMQP（Advanced Message Queuing Protocol，高级消息队列协议）** 是一个 **应用层** 的二进制协议，RabbitMQ 默认实现的是 AMQP 0-9-1。

| 概念 | 说明 |
| --- | --- |
| **Producer** | 消息生产者，向 Exchange 发送消息 |
| **Exchange** | 交换机，接收消息并按规则路由到 Queue |
| **Queue** | 队列，存储消息的缓冲区 |
| **Binding** | 绑定关系，Exchange 与 Queue 之间的路由规则 |
| **Consumer** | 消息消费者，从 Queue 中取出消息处理 |
| **Virtual Host** | 虚拟主机，隔离不同业务（权限隔离单位） |
| **Connection** | TCP 长连接 |
| **Channel** | 轻量级连接，复用 Connection，避免频繁建连开销 |

### 1.6 RabbitMQ 工作原理

```
Producer -> Exchange --(Binding)--> Queue -> Consumer
                          |
                    Virtual Host(隔离)
```

| 步骤 | 说明 |
| --- | --- |
| 1 | Producer 建立到 Broker 的 Connection，再创建 Channel |
| 2 | Producer 将消息发往指定的 Exchange |
| 3 | Exchange 根据 Type 和 Binding Key 决定路由到哪些 Queue |
| 4 | Queue 持久化消息（可选）等待消费 |
| 5 | Consumer 通过 Channel 从 Queue 拉取/接收消息 |
| 6 | Consumer 处理完毕后发送 ACK，Broker 删除消息 |

**Exchange 四种类型：**

| 类型 | 路由规则 |
| --- | --- |
| **direct** | 精确匹配 routing key |
| **fanout** | 广播到所有绑定队列（忽略 routing key） |
| **topic** | 按 routing key 通配符匹配（`*` 一个词、`#` 零或多个词） |
| **headers** | 按消息 header 匹配（不常用） |

---

## 2. RabbitMQ 安装

### 2.1 安装 Erlang

RabbitMQ 由 Erlang 编写，需先安装 Erlang 运行环境。

```bash
# Windows: 官网下载 OTP 安装包，双击安装即可
# Linux (CentOS):
yum install -y erlang
# Linux (Ubuntu):
apt-get install -y erlang
```

> ⚠️ **版本对应**：RabbitMQ 版本对 Erlang 版本有严格依赖，安装前需查阅官方版本对应表，避免启动失败。

### 2.2 安装 RabbitMQ

```bash
# CentOS: 下载 rpm 包安装
rpm --import https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey
yum install -y rabbitmq-server

# Ubuntu:
apt-get install -y rabbitmq-server
```

### 2.3 启动 RabbitMQ

```bash
systemctl start rabbitmq-server    # 启动服务
systemctl status rabbitmq-server   # 查看状态
systemctl enable rabbitmq-server   # 开机自启
systemctl stop rabbitmq-server     # 停止服务
```

### 2.4 账户管理

RabbitMQ 默认仅提供 `guest/guest` 账号，且 **只能本机访问**，生产环境必须新建账号。

```bash
rabbitmqctl add_user admin 123456      # 新增用户
rabbitmqctl set_user_tags admin administrator  # 设置管理员角色
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"  # 授予所有 vhost 权限
rabbitmqctl list_users                  # 查看用户列表
rabbitmqctl delete_user guest           # 删除 guest（生产建议）
```

**用户角色说明：**

| 角色 | 权限 |
| --- | --- |
| `management` | 可访问管控台 |
| `policymaker` | management + 策略管理 |
| `monitoring` | management + 监控 |
| `administrator` | 全部权限 |

### 2.5 管控台

RabbitMQ 自带 Web 管控台插件，需手动开启：

```bash
rabbitmq-plugins enable rabbitmq_management  # 开启管控台插件
# 访问：http://服务器IP:15672  (默认账号 guest/guest，仅本机)
```

**管控台常用 Tab：**

| 页面 | 作用 |
| --- | --- |
| Overview | 集群/节点概览、消息速率 |
| Connections | 连接管理 |
| Channels | 信道管理 |
| Exchanges | 交换机管理 |
| Queues | 队列管理（最常用，可发消息测试） |
| Admin | 用户与权限管理 |

### 2.6 Docker 安装

生产/测试推荐使用 Docker，一键拉起并开启管控台：

```bash
# 单容器：含管控台与延迟队列插件示例端口映射
docker run -d --name rabbitmq \
  -p 5672:5672 -p 15672:15672 \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=123456 \
  rabbitmq:3.13-management
```

> 端口说明：`5672` AMQP 通信端口；`15672` Web 管控台端口；`25672` 集群通信端口。

---

## 3. RabbitMQ 工作模式

RabbitMQ 官方提供 5 种工作模式，本质是 **Exchange 类型与路由规则** 的不同组合。下面以原生 Java AMQP 客户端演示（`amqp-client` 依赖）。

### 3.1 简单模式

最简单的"一生产一消费"模型，无 Exchange（使用默认交换机）。

```
Producer -> [ Queue ] -> Consumer
```

**编写生产者：**

```java
// 创建连接工厂并设置地址
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("127.0.0.1");
factory.setPort(5672);
factory.setUsername("admin");
factory.setPassword("123456");
try (Connection conn = factory.newConnection();
     Channel channel = conn.createChannel()) {
    // 声明队列（不存在则创建），参数：队列名、是否持久化、是否独占、是否自动删除、参数
    channel.queueDeclare("simple_queue", false, false, false, null);
    // 发送消息到默认交换机（"" 表示默认 direct 交换机）
    channel.basicPublish("", "simple_queue", null, "hello".getBytes());
}
```

**编写消费者：**

```java
Channel channel = conn.createChannel();
channel.queueDeclare("simple_queue", false, false, false, null);
// 消费消息，第二个参数 false 表示手动 ACK
channel.basicConsume("simple_queue", false, (consumerTag, msg) -> {
    System.out.println("收到：" + new String(msg.getBody()));
    // 手动签收 deliveryTag
    channel.basicAck(msg.getEnvelope().getDeliveryTag(), false);
}, consumerTag -> {});
```

### 3.2 工作队列模式

一个生产者、一个队列、**多个消费者**，消息被轮询分发（默认）。

```
        ┌-> Consumer1
Producer -> [ Queue ] ┼-> Consumer2
        └-> Consumer3
```

**编写生产者：**

```java
channel.queueDeclare("work_queue", false, false, false, null);
// 循环发送多条消息，模拟任务堆积
for (int i = 0; i < 50; i++) {
    channel.basicPublish("", "work_queue", null, ("task-" + i).getBytes());
    Thread.sleep(10);
}
```

**编写消费者（多实例消费同一队列）：**

```java
channel.queueDeclare("work_queue", false, false, false, null);
// prefetchCount=1，处理完一条才拿下一条（配合公平分发）
channel.basicQos(1);
channel.basicConsume("work_queue", false, (tag, msg) -> {
    Thread.sleep(1000); // 模拟处理耗时
    channel.basicAck(msg.getEnvelope().getDeliveryTag(), false);
}, tag -> {});
```

> 默认轮询：消息均匀分给所有消费者。慢消费者不会拖累快消费者，但可能造成消息积压在慢端（见 6.2 公平分发）。

### 3.3 发布订阅模式

使用 **fanout 交换机**，消息广播到所有绑定的队列，每个消费者都能收到全量消息。

```
                ┌-> Queue1 -> Consumer1
Producer -> fanout
                └-> Queue2 -> Consumer2
```

**编写生产者：**

```java
// 声明 fanout 交换机
channel.exchangeDeclare("fanout_exchange", "fanout");
// 广播消息，routing key 无意义（fanout 忽略）
channel.basicPublish("fanout_exchange", "", null, "broadcast".getBytes());
```

**编写消费者：**

```java
channel.exchangeDeclare("fanout_exchange", "fanout");
// 声明临时队列（随机名，断开自动删除）
String queueName = channel.queueDeclare().getQueue();
// 将临时队列绑定到 fanout 交换机
channel.queueBind(queueName, "fanout_exchange", "");
channel.basicConsume(queueName, true, (tag, msg) -> {
    System.out.println("收到：" + new String(msg.getBody()));
}, tag -> {});
```

### 3.4 路由模式

使用 **direct 交换机**，按 routing key **精确匹配**，只投递给绑定该 key 的队列。

```
                    --[info]--> Queue1 -> Consumer1
Producer -> direct
                    --[error]--> Queue2 -> Consumer2
```

**编写生产者：**

```java
channel.exchangeDeclare("direct_exchange", "direct");
// 按 routing key 区分消息类型
channel.basicPublish("direct_exchange", "info", null, "普通日志".getBytes());
channel.basicPublish("direct_exchange", "error", null, "错误日志".getBytes());
```

**编写消费者：**

```java
channel.exchangeDeclare("direct_exchange", "direct");
String queueName = channel.queueDeclare().getQueue();
// 只订阅 error 级别，绑定多个 key 需多次调用
channel.queueBind(queueName, "direct_exchange", "error");
channel.basicConsume(queueName, true, (tag, msg) -> {
    System.out.println("错误：" + new String(msg.getBody()));
}, tag -> {});
```

### 3.5 通配符模式

使用 **topic 交换机**，routing key 支持通配符匹配，更灵活。

| 通配符 | 含义 |
| --- | --- |
| `*` | 匹配一个单词（如 `order.*` 匹配 `order.create`） |
| `#` | 匹配零个或多个单词（如 `order.#` 匹配 `order`、`order.a.b`） |

> routing key 用 `.` 分隔，如 `order.create.success`。

**编写生产者：**

```java
channel.exchangeDeclare("topic_exchange", "topic");
channel.basicPublish("topic_exchange", "order.create.success", null, "下单成功".getBytes());
channel.basicPublish("topic_exchange", "order.cancel", null, "取消订单".getBytes());
```

**编写消费者：**

```java
channel.exchangeDeclare("topic_exchange", "topic");
String queueName = channel.queueDeclare().getQueue();
// 订阅 order 下所有事件
channel.queueBind(queueName, "topic_exchange", "order.#");
channel.basicConsume(queueName, true, (tag, msg) -> {
    System.out.println("路由键：" + msg.getEnvelope().getRoutingKey()
            + "，内容：" + new String(msg.getBody()));
}, tag -> {});
```

**五种模式对比：**

| 模式 | 交换机 | routing key | 是否广播 |
| --- | --- | --- | --- |
| 简单模式 | 默认 | 队列名 | 否 |
| 工作队列 | 默认 | 队列名 | 否（轮询） |
| 发布订阅 | fanout | 忽略 | 是 |
| 路由模式 | direct | 精确匹配 | 否 |
| 通配符模式 | topic | 通配匹配 | 视绑定而定 |

---

## 4. SpringBoot 整合 RabbitMQ

原生 API 繁琐，生产环境多采用 SpringBoot 整合 `spring-boot-starter-amqp`，声明式、注解化。

### 4.1 项目搭建

**Maven 依赖：**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

**application.yml：**

```yaml
spring:
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: admin
    password: 123456
    virtual-host: /
    publisher-confirm-type: correlated   # 开启确认模式
    publisher-returns: true              # 开启退回模式
    listener:
      simple:
        acknowledge-mode: manual        # 手动 ACK
        prefetch: 1                      # 限流（每个消费者未确认消息数）
```

### 4.2 创建队列和交换机

使用 `@Configuration` + `Bean` 声明交换机、队列、绑定关系：

```java
@Configuration
public class RabbitConfig {
    public static final String EXCHANGE = "boot_exchange";
    public static final String QUEUE = "boot_queue";
    public static final String KEY = "boot.key";

    @Bean
    public TopicExchange exchange() {
        return new TopicExchange(EXCHANGE); // topic 交换机
    }

    @Bean
    public Queue queue() {
        return new Queue(QUEUE, true); // true 表示持久化
    }

    @Bean
    public Binding binding(Queue queue, TopicExchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with(KEY);
    }
}
```

> 其他交换机：`DirectExchange`、`FanoutExchange`、`HeadersExchange`。

### 4.3 编写生产者

注入 `RabbitTemplate` 发送消息：

```java
@Service
public class Producer {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void send(String msg) {
        // 发送消息：交换机、路由键、消息对象
        rabbitTemplate.convertAndSend(RabbitConfig.EXCHANGE,
                RabbitConfig.KEY, msg);
        System.out.println("已发送：" + msg);
    }
}
```

> 若需发送对象，需配置 `Jackson2JsonMessageConverter` 序列化。

### 4.4 编写消费者

使用 `@RabbitListener` 注解监听队列：

```java
@Component
public class Consumer {
    @RabbitListener(queues = RabbitConfig.QUEUE)
    public void receive(String msg, Channel channel, Message message) throws IOException {
        long tag = message.getMessageProperties().getDeliveryTag();
        try {
            System.out.println("消费：" + msg);
            channel.basicAck(tag, false);          // 处理成功，手动签收
        } catch (Exception e) {
            channel.basicNack(tag, false, false);  // 处理失败，丢弃消息（第三个参数 false 不重回队列）
        }
    }
}
```

> `basicNack(tag, multiple, requeue)`：requeue=false 丢弃/进死信，true 重回队列（慎用，会无限循环）。

---

## 5. 消息的可靠性投递

消息从生产者到消费者经过多个环节，任一环节失败都会丢消息。可靠性投递解决 **"消息不丢失"** 问题。

### 5.1 概念

消息投递链路及可靠性保障：

```
Producer --①--> Exchange --②--> Queue --③--> Consumer
   确认模式      退回模式     持久化      手动 ACK
```

| 环节 | 保障机制 |
| --- | --- |
| ① Producer -> Broker | **确认模式（Confirm）**，Broker 收到后回 ACK |
| ② Exchange -> Queue | **退回模式（Return）**，路由失败时退回给 Producer |
| ③ Broker 存储 | **持久化**（Exchange、Queue、Message 都要 durable） |
| ④ Consumer -> Broker | **手动 ACK**，处理完成才确认 |

### 5.2 确认模式

确认 Producer 发送到 Exchange 是否成功。需在 yml 配置 `publisher-confirm-type: correlated`。

```java
@Configuration
public class ConfirmConfig {
    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory cf) {
        RabbitTemplate template = new RabbitTemplate(cf);
        // 设置确认回调，Broker 收到后异步触发
        template.setConfirmCallback((corr, ack, cause) -> {
            if (ack) {
                System.out.println("消息到达 Exchange 成功");
            } else {
                System.err.println("失败原因：" + cause); // 记录并重发
            }
        });
        return template;
    }
}
```

> 仅能确认"是否到达 Exchange"，无法知道是否路由到 Queue。

### 5.3 退回模式

当消息到达 Exchange 但 **无法路由到任何 Queue** 时触发。需配置 `publisher-returns: true`。

```java
template.setReturnsCallback(returned -> {
    System.err.println("路由失败：" + returned.getReplyText()
            + "，消息：" + new String(returned.getMessage().getBody()));
});
// 开启强制路由（找不到队列时触发退回，而非丢弃）
template.setMandatory(true);
```

> 典型场景：消费者都下线、routing key 写错。

### 5.4 Ack 自动签收

Broker 推送消息后 **立即删除**，不等消费者处理完。配置 `acknowledge-mode: auto`。

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        acknowledge-mode: auto   # 消费者抛异常时由 Spring 重试
```

> 优点：简单；缺点：处理失败时消息可能丢失（Spring 默认会重试 3 次后丢弃），生产环境慎用。

### 5.5 Ack 手动签收

消费者处理完成后 **主动通知 Broker** 删除消息，保证"处理成功才丢消息"。配置 `acknowledge-mode: manual`。

```java
@RabbitListener(queues = "boot_queue")
public void receive(String msg, Channel channel, Message message) throws IOException {
    long tag = message.getMessageProperties().getDeliveryTag();
    try {
        doBusiness(msg);                 // 执行业务逻辑
        channel.basicAck(tag, false);   // 成功：签收（multiple=false 仅签当前消息）
    } catch (Exception e) {
        // 第三个参数 requeue：true 重回队列（注意死循环），false 丢弃/进死信队列
        channel.basicNack(tag, false, false);
    }
}
```

| 方法 | 含义 |
| --- | --- |
| `basicAck` | 确认消费成功 |
| `basicNack` | 拒绝（可批量），可选择重回队列或丢弃 |
| `basicReject` | 拒绝（单条），与 nack 类似 |

---

## 6. RabbitMQ 高级特性

### 6.1 消费端限流

防止消费者被瞬时大流量压垮，限制 **未确认消息数**（prefetch）。

```java
// 原生 API：每次只推送 1 条未确认消息
channel.basicQos(1);
```

```yaml
# SpringBoot 配置：每个消费者最多有 1 条未确认消息
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 1
```

> 工作原理：Broker 仅在消费者 ACK 一条后才推送下一条，配合手动 ACK 才生效。

### 6.2 公平分发

**默认轮询**：即使消费者 A 处理慢、B 处理快，仍各分一半，导致 A 积压。

**公平分发**：根据消费者处理能力动态分配——谁 ACK 快谁拿得多。实现方式：`prefetch=1` + 手动 ACK。

```java
// 慢消费者一次只拿 1 条
channel.basicQos(1);
channel.basicConsume("work_queue", false, (tag, msg) -> {
    Thread.sleep(5000); // 慢处理
    channel.basicAck(msg.getEnvelope().getDeliveryTag(), false);
}, tag -> {});
```

> 慢消费者 ACK 前不会拿到新消息，快消费者自然拿得多。

### 6.3 限流实现不公平分发

`prefetch=1` 既限流又公平，本质是 **"谁空闲谁拿"**。

```java
// 消费者 A：处理快，1 秒 1 条
channel.basicQos(1);
// 消费者 B：处理慢，10 秒 1 条
channel.basicQos(1);
// 结果：A 拿到绝大多数消息，B 仅在空闲时拿 1 条
```

> 调优建议：prefetch 太小吞吐低，太大不公平。一般取 10-50，根据业务 RT 调整。

### 6.4 设置队列所有消息存活时间

**TTL（Time To Live）**：消息存活时间，超时自动过期（进入死信队列）。

```java
Map<String, Object> args = new HashMap<>();
// 队列所有消息存活 60 秒（单位毫秒）
args.put("x-message-ttl", 60000);
channel.queueDeclare("ttl_queue", false, false, false, args);
```

```java
// SpringBoot 方式
@Bean
public Queue ttlQueue() {
    return QueueBuilder.durable("ttl_queue")
            .withArgument("x-message-ttl", 60000)
            .build();
}
```

> 队列级 TTL：消息进入队列即开始计时，**先入先过期**（FIFO 中超时的消息会被整体清理）。

### 6.5 设置单条消息存活时间

```java
AMQP.BasicProperties props = new AMQP.BasicProperties()
        .builder()
        .expiration("10000") // 单条消息 10 秒后过期
        .build();
channel.basicPublish("", "queue_name", props, "msg".getBytes());
```

```java
// SpringBoot 方式
MessageProperties props = MessagePropertiesBuilder.newInstance()
        .setExpiration("10000").build();
Message msg = MessageBuilder.withBody("data".getBytes()).andProperties(props).build();
rabbitTemplate.send("", "queue_name", msg);
```

> 单条 TTL：从发送时计时；与队列 TTL 同时存在时，取较小值。

### 6.6 优先级队列

支持消息按优先级消费，**高优先级先被处理**。

```java
Map<String, Object> args = new HashMap<>();
// 声明队列支持 0-10 优先级
args.put("x-max-priority", 10);
channel.queueDeclare("priority_queue", false, false, false, args);

// 发送消息时指定优先级
AMQP.BasicProperties props = new AMQP.BasicProperties()
        .builder().priority(8).build();
channel.basicPublish("", "priority_queue", props, "urgent".getBytes());
```

> ⚠️ 仅在队列有积压时才生效——若消费速度跟得上，消息刚到就被消费，优先级无意义。

---

## 7. RabbitMQ 死信队列

### 7.1 概念

**DLX（Dead Letter Exchange，死信交换机）**：消息变成"死信"后，会被重新投递到 DLX，再路由到死信队列。

**消息成为死信的三种情况：**

| 情形 | 说明 |
| --- | --- |
| 消息被 `basicNack`/`basicReject` 且 `requeue=false` | 消费者拒绝且不重回队列 |
| 消息 TTL 过期 | 队列/消息 TTL 到期 |
| 队列达到最大长度 | 队列满，新消息被挤掉 |

```
正常队列 --(死信)--> DLX --> 死信队列 --> 死信消费者
```

> 应用：订单超时取消、消息重试后归档、延迟队列的基础。

### 7.2 创建死信队列

通过 `x-dead-letter-exchange` 参数为正常队列绑定 DLX：

```java
// 1. 声明死信交换机和死信队列
channel.exchangeDeclare("dlx_exchange", "direct");
channel.queueDeclare("dlx_queue", false, false, false, null);
channel.queueBind("dlx_queue", "dlx_exchange", "dlx.key");

// 2. 声明正常队列，绑定 DLX
Map<String, Object> args = new HashMap<>();
args.put("x-dead-letter-exchange", "dlx_exchange"); // 死信交换机
args.put("x-dead-letter-routing-key", "dlx.key");     // 死信路由键
args.put("x-message-ttl", 10000);                      // 10 秒 TTL 触发死信
channel.queueDeclare("business_queue", false, false, false, args);
```

```java
// SpringBoot 方式：链式声明
@Bean
public Queue businessQueue() {
    return QueueBuilder.durable("business_queue")
            .withArgument("x-dead-letter-exchange", "dlx_exchange")
            .withArgument("x-dead-letter-routing-key", "dlx.key")
            .withArgument("x-message-ttl", 10000)
            .build();
}
```

### 7.3 测试死信队列

**方式一：TTL 过期触发死信**

```java
// 生产者发消息到正常队列，但无消费者
channel.basicPublish("", "business_queue", null, "order_001".getBytes());
// 10 秒后，消息因 TTL 过期自动进入 dlx_queue
```

**方式二：消费者拒绝触发死信**

```java
channel.basicConsume("business_queue", false, (tag, msg) -> {
    // 处理失败，拒绝且不重回队列 -> 进入死信队列
    channel.basicReject(msg.getEnvelope().getDeliveryTag(), false);
}, tag -> {});
```

> 死信队列可被普通消费者消费，常用于"重试 N 次后归档"。

---

## 8. RabbitMQ 延迟队列

### 8.1 概念

延迟队列：消息发送后 **延迟指定时间** 才被消费。

**典型场景：** 下单后 30 分钟未支付自动取消、用户注册 7 天后发提醒。

**RabbitMQ 本身无延迟队列概念**，需通过两种方式实现：

| 方式 | 原理 | 优劣 |
| --- | --- | --- |
| 死信队列 + TTL | 消息 TTL 过期进 DLX | 不支持任意时间延迟（见下） |
| 延迟插件 rabbitmq_delayed_message_exchange | Exchange 内置延迟 | 支持任意延迟时间（推荐） |

### 8.2 死信队列实现

```java
// 正常队列：设置 TTL，绑定 DLX，无消费者
Map<String, Object> args = new HashMap<>();
args.put("x-dead-letter-exchange", "dlx_exchange");
args.put("x-message-ttl", 30000); // 30 秒后进入死信队列
channel.queueDeclare("delay_queue", false, false, false, args);

// 发送消息，30 秒后被死信消费者处理
channel.basicPublish("", "delay_queue", null, "delayed_msg".getBytes());
```

**致命缺陷：** RabbitMQ 只校验 **队头消息** 的 TTL。若队头消息延迟 1 小时、后续消息延迟 1 分钟，后续消息会被阻塞，无法按各自 TTL 过期——**无法实现"每条消息独立延迟时间"**。

### 8.3 安装延迟队列插件

```bash
# 1. 下载插件（版本须与 RabbitMQ 版本匹配）
#    https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases
# 2. 放到插件目录
cp rabbitmq_delayed_message_exchange-3.13.0.ez /usr/lib/rabbitmq/lib/rabbitmq_server-3.13.0/plugins/

# 3. 启用插件并重启
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
systemctl restart rabbitmq-server
```

> 启用后在管控台 Exchange 的 type 中会出现 `x-delayed-message` 选项。

### 8.4 使用插件实现延迟队列

**声明延迟交换机（custom 类型 x-delayed-message）：**

```java
Map<String, Object> args = new HashMap<>();
// 指定延迟交换机的底层路由类型（direct/topic/fanout）
args.put("x-delayed-type", "direct");
// 声明延迟交换机，类型为 x-delayed-message
channel.exchangeDeclare("delayed_exchange", "x-delayed-message", true, false, args);
channel.queueDeclare("delayed_queue", true, false, false, null);
channel.queueBind("delayed_queue", "delayed_exchange", "delay.key");
```

**发送延迟消息（在 header 中指定延迟时间）：**

```java
Map<String, Object> headers = new HashMap<>();
headers.put("x-delay", 30000); // 延迟 30 秒（毫秒）
AMQP.BasicProperties props = new AMQP.BasicProperties()
        .builder().headers(headers).build();
channel.basicPublish("delayed_exchange", "delay.key", props, "order_timeout".getBytes());
```

```java
// SpringBoot 方式
rabbitTemplate.convertAndSend("delayed_exchange", "delay.key", msg, message -> {
    message.getMessageProperties().setHeader("x-delay", 30000);
    return message;
});
```

> 优点：每条消息独立延迟时间，互不影响；支持任意延迟，精确到毫秒。

---

## 9. RabbitMQ 集群

### 9.1 搭建集群

RabbitMQ 集群分为两种节点：

| 节点类型 | 说明 |
| --- | --- |
| **磁盘节点（disc）** | 持久化元数据（队列/交换机/绑定），默认类型 |
| **内存节点（ram）** | 元数据存内存，性能高但重启丢失 |

> ⚠️ 集群只同步 **元数据**（队列定义），**不同步队列内容**！队列内容默认只在某一节点（见 9.2 镜像队列）。

**搭建步骤（以 3 节点为例）：**

```bash
# 1. 三台机器 hosts 互相解析
# /etc/hosts:
#   192.168.1.11 node1
#   192.168.1.12 node2
#   192.168.1.13 node3

# 2. 同步 erlang cookie（集群节点 cookie 必须一致）
scp /var/lib/rabbitmq/.erlang.cookie node2:/var/lib/rabbitmq/
scp /var/lib/rabbitmq/.erlang.cookie node3:/var/lib/rabbitmq/

# 3. 在 node2、node3 上加入 node1
rabbitmqctl stop_app
rabbitmqctl join_cluster rabbit@node1
rabbitmqctl start_app

# 4. 查看集群状态
rabbitmqctl cluster_status
```

> 生产建议：至少一个 disc 节点保证元数据持久化；为避免单点，至少两个 disc 节点。

### 9.2 镜像队列

普通集群中队列内容仅存于单节点，该节点宕机则队列消息丢失。**镜像队列** 在多个节点间复制队列内容，实现高可用。

```bash
# 策略：所有队列在集群内镜像 2 份
rabbitmqctl set_policy ha-all "^" '{"ha-mode":"exactly","ha-params":2,"ha-sync-mode":"automatic"}'

# 策略：所有队列在所有节点镜像
rabbitmqctl set_policy ha-all "^" '{"ha-mode":"all","ha-sync-mode":"automatic"}'
```

| 参数 | 说明 |
| --- | --- |
| `ha-mode` | `all` 全部节点镜像；`exactly` 指定数量；`nodes` 指定节点 |
| `ha-params` | exactly 模式下的镜像数 |
| `ha-sync-mode` | `automatic` 新节点自动同步 |

> 主节点宕机后，从镜像中选举新主，**对客户端透明**，无需改连接配置。

### 9.3 负载均衡

集群只解决高可用，**不自动负载均衡**——客户端连哪个节点就访问哪个节点。需外置负载均衡器。

**方案一：HAProxy（推荐）**

```haproxy
listen rabbitmq
    bind 0.0.0.0:5672
    mode tcp
    balance roundrobin            # 轮询负载
    server node1 192.168.1.11:5672 check inter 5000 rise 2 fall 3
    server node2 192.168.1.12:5672 check inter 5000 rise 2 fall 3
    server node3 192.168.1.13:5672 check inter 5000 rise 2 fall 3
```

**方案二：Keepalived + HAProxy（VIP 高可用）**

```
Client -> VIP(Keepalived) -> HAProxy -> RabbitMQ集群
```

**方案三：客户端直连 + 故障转移**

```java
ConnectionFactory factory = new ConnectionFactory();
// 多地址列表，自动故障转移
factory.setHost("192.168.1.11");
factory.newConnection(addresses); // 传入 Address[]
```

**负载均衡注意事项：**

1. **生产者与消费者连接不同节点**，避免单节点压力过大。
2. 镜像队列主节点不固定，HAProxy 轮询可能导致"远程消费"（消息在 A，消费者连 B）——开启 `ha-sync-mode=automatic` 减少影响。
3. 管控台 `15672` 也需负载均衡，避免单点访问。

---

## 附录：核心知识点速查

| 知识点 | 关键配置/方法 |
| --- | --- |
| 持久化 | Queue durable=true、Message deliveryMode=2 |
| 手动 ACK | acknowledge-mode=manual + channel.basicAck |
| 限流 | prefetch=1 + 手动 ACK |
| 可靠投递 | publisher-confirm-type + publisher-returns |
| 死信队列 | x-dead-letter-exchange |
| 延迟队列 | x-delayed-message 插件 + x-delay header |
| 队列 TTL | x-message-ttl |
| 单条 TTL | expiration / setExpiration |
| 优先级 | x-max-priority + priority |
| 镜像队列 | ha-mode=all |

> **生产环境黄金配置**：持久化 + 手动 ACK + 确认/退回模式 + 镜像队列集群 + 合理 prefetch，缺一不可。
