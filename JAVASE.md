# JavaSE(十八章节)

**阅读指南**：由浅入深，每个知识点均配有精简示例。建议按顺序阅读，环环相扣。

## 目录

* [**第一部分：编程基础**](JAVASE.md#第一部分编程基础)
  * [1.1 基本数据类型（4类8种）](JAVASE.md#11-基本数据类型4类8种)
  * [1.2 类型转换](JAVASE.md#12-类型转换)
  * [1.3 进制转换](JAVASE.md#13-进制转换)
  * [1.4 编码规范](JAVASE.md#14-编码规范)
  * [1.5 运算符](JAVASE.md#15-运算符)
  * [1.6 控制语法](JAVASE.md#16-控制语法)
* [**第二部分：面向对象入门**](JAVASE.md#第二部分面向对象入门)
  * [2.1 类与对象的基本概念](JAVASE.md#21-类与对象的基本概念)
  * [2.2 代码块](JAVASE.md#22-代码块)
  * [2.3 关键字](JAVASE.md#23-关键字)
  * [2.4 封装](JAVASE.md#24-封装)
  * [2.5 继承与多态基础](JAVASE.md#25-继承与多态基础)
* [**第三部分：面向对象进阶**](JAVASE.md#第三部分面向对象进阶)
  * [3.1 抽象类与接口](JAVASE.md#31-抽象类与接口)
  * [3.2 枚举类（enum）](JAVASE.md#32-枚举类enum)
  * [3.3 内部类](JAVASE.md#33-内部类)
  * [3.4 包装类与自动拆装箱](JAVASE.md#34-包装类与自动拆装箱)
  * [3.5 向上转型与向下转型](JAVASE.md#35-向上转型与向下转型)
* [**第四部分：异常处理**](JAVASE.md#第四部分异常处理)
  * [4.1 Throwable 体系](JAVASE.md#41-throwable-体系)
  * [4.2 异常处理语法](JAVASE.md#42-异常处理语法)
  * [4.3 异常链与自定义异常](JAVASE.md#43-异常链与自定义异常)
* [**第五部分：数组与常用工具类**](JAVASE.md#第五部分数组与常用工具类)
  * [5.1 数组](JAVASE.md#51-数组)
  * [5.2 Arrays 工具类](JAVASE.md#52-arrays-工具类)
  * [5.3 Comparable 与 Comparator](JAVASE.md#53-comparable-与-comparator)
  * [5.4 字符串家族](JAVASE.md#54-字符串家族)
  * [5.5 其他常用工具类](JAVASE.md#55-其他常用工具类)
* [**第六部分：泛型**](JAVASE.md#第六部分泛型)
  * [6.1 泛型基础](JAVASE.md#61-泛型基础)
  * [6.2 泛型通配符](JAVASE.md#62-泛型通配符)
* [**第七部分：容器框架（集合）**](JAVASE.md#第七部分容器框架集合)
  * [7.1 体系概览](JAVASE.md#71-体系概览)
  * [7.2 List](JAVASE.md#72-list)
  * [7.3 Set](JAVASE.md#73-set)
  * [7.4 Map](JAVASE.md#74-map)
  * [7.5 Queue / Deque](JAVASE.md#75-queue--deque)
  * [7.6 迭代器与 Collections](JAVASE.md#76-迭代器与-collections)
* [**第八部分：IO 流（基础）**](JAVASE.md#第八部分io-流基础)
  * [8.1 流分类](JAVASE.md#81-流分类)
  * [8.2 文件流 + 缓冲流](JAVASE.md#82-文件流--缓冲流)
  * [8.3 数据流与对象流](JAVASE.md#83-数据流与对象流)
  * [8.4 Apache Commons IO](JAVASE.md#84-apache-commons-io)
* [**第九部分：NIO（New I/O）—— IO 进阶**](JAVASE.md#第九部分nio-new-io--io-进阶)
  * [9.1 核心组件](JAVASE.md#91-核心组件)
  * [9.2 Buffer 操作](JAVASE.md#92-buffer-操作)
  * [9.3 Channel 与 Selector](JAVASE.md#93-channel-与-selector)
* [**第十部分：多线程（基础）**](JAVASE.md#第十部分多线程基础)
  * [10.1 线程创建方式](JAVASE.md#101-线程创建方式)
  * [10.2 线程生命周期](JAVASE.md#102-线程生命周期)
  * [10.3 线程常用方法](JAVASE.md#103-线程常用方法)
  * [10.4 synchronized 与 volatile](JAVASE.md#104-synchronized-与-volatile)
  * [10.5 wait / notify / notifyAll](JAVASE.md#105-wait--notify--notifyall)
* [**第十一部分：JUC 并发包 —— 多线程进阶**](JAVASE.md#第十一部分juc-并发包--多线程进阶)
  * [11.1 线程池](JAVASE.md#111-线程池)
  * [11.2 锁框架](JAVASE.md#112-锁框架)
  * [11.3 原子类](JAVASE.md#113-原子类)
  * [11.4 并发容器](JAVASE.md#114-并发容器)
  * [11.5 同步工具](JAVASE.md#115-同步工具)
  * [11.6 ThreadLocal](JAVASE.md#116-threadlocal)
* [**第十二部分：网络编程**](JAVASE.md#第十二部分网络编程)
  * [12.1 TCP 编程](JAVASE.md#121-tcp-编程)
  * [12.2 UDP 编程](JAVASE.md#122-udp-编程)
* [**第十三部分：反射、类加载机制与动态代理**](JAVASE.md#第十三部分反射类加载机制与动态代理)
  * [13.1 反射（Reflection）](JAVASE.md#131-反射reflection)
  * [13.2 类加载机制（ClassLoader）](JAVASE.md#132-类加载机制classloader)
  * [13.3 动态代理（Dynamic Proxy）](JAVASE.md#133-动态代理dynamic-proxy)
* [**第十四部分：注解（Annotation）**](JAVASE.md#第十四部分注解annotation)
  * [14.1 内置注解与元注解](JAVASE.md#141-内置注解与元注解)
  * [14.2 自定义注解 + 反射解析](JAVASE.md#142-自定义注解--反射解析)
* [**第十五部分：Lambda 与函数式编程**](JAVASE.md#第十五部分lambda-与函数式编程)
  * [15.1 Lambda 表达式与函数式接口](JAVASE.md#151-lambda-表达式与函数式接口)
  * [15.2 Stream API](JAVASE.md#152-stream-api)
* [**第十六部分：常用工具类补充**](JAVASE.md#第十六部分常用工具类补充)
  * [16.1 Optional 类（避免空指针）](JAVASE.md#161-optional-类避免空指针)
  * [16.2 时间 API（java.time，Java 8+）](JAVASE.md#162-时间-apijavatime-java-8)
* [**第十七部分：数据结构与简单算法**](JAVASE.md#第十七部分数据结构与简单算法)
  * [17.1 数据结构基础](JAVASE.md#171-数据结构基础)
  * [17.2 常用算法](JAVASE.md#172-常用算法)
* [**第十八部分：JVM 内存结构与垃圾回收**](JAVASE.md#第十八部分jvm-内存结构与垃圾回收)
  * [18.1 运行时数据区](JAVASE.md#181-运行时数据区)
  * [18.2 对象创建与内存布局](JAVASE.md#182-对象创建与内存布局)
  * [18.3 垃圾回收（GC）](JAVASE.md#183-垃圾回收gc)
  * [18.4 类加载过程](JAVASE.md#184-类加载过程)
  * [18.5 Java 内存模型（JMM）](JAVASE.md#185-java-内存模型jmm)

## 第一部分：编程基础

### 1.1 基本数据类型（4类8种）

| 类型 | 关键字       | 字节 | 默认值      | 范围                     |
| -- | --------- | -- | -------- | ---------------------- |
| 字节 | `byte`    | 1  | 0        | -128 \~ 127            |
| 短整 | `short`   | 2  | 0        | -2¹⁵ \~ 2¹⁵-1          |
| 整型 | `int`     | 4  | 0        | -2³¹ \~ 2³¹-1          |
| 长整 | `long`    | 8  | 0L       | -2⁶³ \~ 2⁶³-1          |
| 单浮 | `float`   | 4  | 0.0f     | ±3.4E-38 \~ ±3.4E+38   |
| 双浮 | `double`  | 8  | 0.0d     | ±1.7E-308 \~ ±1.7E+308 |
| 字符 | `char`    | 2  | '\u0000' | 0 \~ 65535（Unicode）    |
| 布尔 | `boolean` | —  | false    | true / false           |

```java
int a = 10;          // 整型默认 int
long b = 100L;       // long 需加 L 后缀
float f = 3.14f;     // float 需加 f 后缀，否则默认 double
char c = 'A';        // 字符用单引号
boolean flag = true;
```

### 1.2 类型转换

* **自动类型提升**：小范围 → 大范围，编译自动完成（`byte→short→int→long→float→double`，`char→int`）
* **强制类型转换**：大范围 → 小范围，需手动 `(type)`，可能丢失精度或溢出

```java
// 自动提升：int → long
long l = 100;                     // int 自动提升为 long
// 强制转换：double → int（截断小数）
int i = (int) 3.99;               // i = 3，精度丢失
// 运算时自动提升
byte x = 10; byte y = 20;
int sum = x + y;                  // byte 运算前自动提升为 int
```

### 1.3 进制转换

* 二进制 `0b`、八进制 `0`、十六进制 `0x` 前缀
* `Integer.toBinaryString()` / `Integer.toHexString()` 等工具方法

```java
int bin = 0b1010;                 // 二进制 → 10
int oct = 017;                    // 八进制 → 15
int hex = 0x1F;                   // 十六进制 → 31
String s = Integer.toBinaryString(10); // "1010"
```

### 1.4 编码规范

* **注释**：`//` 单行、`/* */` 多行、`/** */` 文档注释（可生成 JavaDoc）
* **标识符**：字母/下划线/`$` 开头，不能数字开头，不能是关键字，**驼峰命名**
* `import` 导包，`package` 声明包路径（必须在第一行）

```java
package com.example.util;         // 必须首行
import java.util.List;            // 导包
/**
 * 工具类（文档注释）
 */
public class StringUtil {
    // 单行注释
    public static boolean isEmpty(String str) { /* 多行 */ return str == null || str.isEmpty(); }
}
```

### 1.5 运算符

| 分类   | 运算符                            | 注意                                     |
| ---- | ------------------------------ | -------------------------------------- |
| 算术   | `+` `-` `*` `/` `%`            | `/` 整数除得整数，`%` 取余                      |
| 自增自减 | `++` `--`                      | 前缀先变后用，后缀先用后变                          |
| 关系   | `==` `!=` `>` `<` `>=` `<=`    | `==` 比较基本类型值，引用类型比较地址                  |
| 逻辑   | `&&` \`                        |                                        |
| 位运算  | `&` \`                         | ^ \~ << >> >>>\`                       |
| 三元   | `条件 ? 值1 : 值2`                 | 简洁的 if-else 替代                         |
| 赋值   | `=` `+=` `-=` `*=` `/=` `%=` 等 | `a += b` 等价于 `a = (type)(a + b)` 含隐式强转 |

```java
// 逻辑短路：&& 左边 false 不执行右边
boolean r = false && (1/0 > 0);    // 不会抛异常，因为短路
// 位运算
int flags = 0b1010;               // 10
int mask  = 0b0110;               // 6
int and = flags & mask;           // 0b0010 = 2（按位与）
int or  = flags | mask;           // 0b1110 = 14（按位或）
int xor = flags ^ mask;           // 0b1100 = 12（异或）
int leftShift = 1 << 3;           // 8（左移3位 = 乘以2³）
// 三元运算符
int max = a > b ? a : b;          // 取最大值
```

### 1.6 控制语法

* **分支**：`if/else if/else`、`switch`（JDK14+ 支持箭头表达式 `case X ->`）
* **循环**：`for`、`foreach`（增强 for，遍历数组/集合）、`while`、`do-while`
* **跳转**：`break`（跳出循环）、`continue`（跳过本次迭代）

```java
// switch 表达式（JDK 14+）
String season = switch (month) {
    case 12, 1, 2  -> "冬";
    case 3, 4, 5   -> "春";
    case 6, 7, 8   -> "夏";
    case 9, 10, 11 -> "秋";
    default -> "无";
};
// foreach
for (int num : new int[]{1,2,3}) { System.out.println(num); }
// break 带标签跳出外层循环
outer: for (int i = 0; i < 3; i++)
    for (int j = 0; j < 3; j++)
        if (i == 1) break outer;  // 直接跳出外层循环
```

***

## 第二部分：面向对象入门

### 2.1 类与对象的基本概念

* **类**是模板，**对象**是实例；`new` 关键字在堆中分配内存
* **构造方法**：与类同名、无返回值、可重载；默认无参构造在定义其他构造后消失
* **方法重载**：方法名相同、参数列表不同（个数/类型/顺序），与返回值无关
* **成员变量**：类中定义，有默认值；**局部变量**：方法内定义，**必须初始化**才能使用

```java
public class Person {
    private String name;           // 成员变量（堆，默认 null）
    private int age;               // 成员变量（堆，默认 0）

    public Person() {}             // 无参构造
    public Person(String name, int age) { this.name = name; this.age = age; } // 重载

    public void say() {            // 普通方法
        int x = 10;                // 局部变量（栈，必须初始化）
        System.out.println(name + ": " + x);
    }
    public void say(String msg) {} // 方法重载：参数不同
}
```

### 2.2 代码块

* **普通代码块**（`{}`）：每次 `new` 对象时执行，在构造方法**之前**执行
* **静态代码块**（`static {}`）：类加载时执行**一次**，用于初始化静态资源

```java
public class Demo {
    static { System.out.println("1.静态块，类加载时执行一次"); }
    { System.out.println("2.实例块，new 时执行，先于构造方法"); }
    public Demo() { System.out.println("3.构造方法"); }
}
// new Demo() 输出: 1 → 2 → 3
```

### 2.3 关键字

| 关键字      | 作用                               |
| -------- | -------------------------------- |
| `this`   | 指向当前对象引用；`this()` 调用本类其他构造（必须首行） |
| `static` | 属于类而非实例；静态方法只能访问静态成员，不能使用`this`  |
| `final`  | 修饰类→不可继承；修饰方法→不可重写；修饰变量→不可修改（常量） |

```java
public class Student {
    private String name;
    static String school = "北大";  // 静态变量，所有实例共享

    public Student() { this("无名"); }  // this() 调用有参构造
    public Student(String name) { this.name = name; }

    public static void printSchool() {  // 静态方法
        System.out.println(school);     // ✅ 可访问静态变量
        // System.out.println(name);    // ❌ 不能访问实例变量
    }
}
// final 用法
final class MathUtil {}                // 不可继承
class Base { final void go() {} }      // 不可重写
final double PI = 3.14159;             // 常量，不可修改
```

### 2.4 封装

* **访问修饰符**：控制可见性，实现信息隐藏

| 修饰符         | 本类 | 同包 | 子类 | 全局 |
| ----------- | -- | -- | -- | -- |
| `private`   | ✅  | ❌  | ❌  | ❌  |
| default     | ✅  | ✅  | ❌  | ❌  |
| `protected` | ✅  | ✅  | ✅  | ❌  |
| `public`    | ✅  | ✅  | ✅  | ✅  |

* **JavaBean**：`private` 属性 + `getter/setter` + **无参构造**（反射/框架依赖）

```java
public class User {
    private String name;     // 私有属性
    public User() {}         // 无参构造（JavaBean 必须）
    public String getName() { return name; }   // getter
    public void setName(String name) { this.name = name; } // setter
}
```

### 2.5 继承与多态基础

* **继承**：`extends`，单继承；子类拥有父类所有非 `private` 成员
* **方法重写**：子类重写父类方法，`@Override` 注解检查，访问权限不能更低
* **`super`**：`super.xxx` 访问父类成员，`super()` 调用父类构造（必须首行）
* **多态**：父类引用指向子类对象，方法调用看右边（动态绑定），属性看左边

```java
class Animal {
    String name = "动物";
    void eat() { System.out.println("吃"); }
}
class Dog extends Animal {
    String name = "狗";
    @Override
    void eat() { System.out.println("吃骨头"); }  // 重写
}
// 多态
Animal a = new Dog();
a.eat();           // "吃骨头" — 方法看右边（Dog），动态绑定
System.out.println(a.name); // "动物" — 属性看左边（Animal），无多态
```

* **`instanceof`**：判断对象是否属于某类型，用于安全向下转型

```java
if (a instanceof Dog) { Dog d = (Dog) a; d.eat(); }  // 安全转型
// JDK 16+ 模式匹配
if (a instanceof Dog d) { d.eat(); }                  // 直接绑定变量
```

* **`Object` 核心方法**：所有类的祖先，重要方法：
  * `equals()`：默认比较地址（`==`），需重写按内容比较
  * `hashCode()`：返回哈希值，**equals 相等则 hashCode 必须相等**（反之不强制）
  * `toString()`：默认返回 `类名@哈希码`，建议重写

```java
public class Student {
    private int id;
    private String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Student)) return false;
        Student s = (Student) o;
        return id == s.id && Objects.equals(name, s.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);  // equals 用到的字段必须参与 hashCode
    }

    @Override
    public String toString() {
        return "Student{id=" + id + ", name='" + name + "'}";
    }
}
// equals 与 hashCode 契约：
// ① equals 相等 ⇒ hashCode 必相等（否则 HashMap 会出 bug）
// ② hashCode 相等 ⇏ equals 相等（哈希冲突正常）
```

***

## 第三部分：面向对象进阶

### 3.1 抽象类与接口

| 特性    | 抽象类                 | 接口（Java 8+）                |
| ----- | ------------------- | -------------------------- |
| 关键字   | `abstract class`    | `interface`                |
| 继承/实现 | 单继承                 | 多实现                        |
| 构造方法  | 有（供子类调用）            | 无                          |
| 方法    | 抽象方法 + 普通方法         | 抽象方法 +`default` + `static` |
| 变量    | 普通成员变量              | 只能`public static final` 常量 |
| 选择    | "is-a" 关系，共享状态/构造逻辑 | "can-do" 能力，解耦契约           |

```java
abstract class Animal {
    protected String name;
    public Animal(String name) { this.name = name; }
    public abstract void cry();
    public void sleep() { System.out.println(name + "睡觉"); }
}
interface Flyable {
    void fly();
    default void land() { System.out.println("降落"); }
    static void info() { System.out.println("飞行接口"); }
}
class Bird extends Animal implements Flyable {
    public Bird() { super("鸟"); }
    public void cry() { System.out.println("叽叽"); }
    public void fly() { System.out.println("扑翅飞"); }
}
```

### 枚举类（`enum`）

* `enum` 本质是 `final class` 继承 `Enum`，实例在类加载时创建（线程安全）
* 构造方法**默认 private**，不能外部实例化

```java
enum Season { SPRING, SUMMER, AUTUMN, WINTER; }
Season s = Season.SUMMER;
System.out.println(s.ordinal());   // 1
System.out.println(s.name());      // "SUMMER"

enum Color {
    RED("红"), GREEN("绿"), BLUE("蓝");
    private final String cn;
    Color(String cn) { this.cn = cn; }
    public String getCn() { return cn; }
}

enum Singleton { INSTANCE; public void doWork() {} }

interface Greet { void sayHi(); }
enum Week implements Greet {
    MON { public void sayHi() { System.out.println("周一好"); } },
    FRI { public void sayHi() { System.out.println("周五啦"); } };
}
```

### 3.3 内部类

> **内存模型**：外部类与内部类生成独立的 `.class` 文件（`Outer$Inner.class`），堆中也是两个独立对象。非静态内部类持有 `this$0` 引用指向外部类实例。

| 类型    | 修饰符      | 特点                       |
| ----- | -------- | ------------------------ |
| 成员内部类 | 无        | 持有外部类引用，需外部类实例来创建        |
| 静态内部类 | `static` | 不持有外部类引用，可直接创建（推荐优先使用）   |
| 局部内部类 | 无        | 定义在方法内，作用域仅限于方法          |
| 匿名内部类 | 无        | 一次性使用，Lambda 可替代（仅函数式接口） |

```java
class Outer {
    private int x = 10;
    class Inner { int getX() { return x; } }
    static class StaticInner { void go() {} }
    void method() {
        class Local { void print() { System.out.println(x); } }
        new Local().print();
        Runnable r = new Runnable() {
            public void run() { System.out.println("匿名内部类"); }
        };
        Runnable r2 = () -> System.out.println("Lambda");
    }
}
Outer.Inner inner = new Outer().new Inner();
Outer.StaticInner si = new Outer.StaticInner();
```

### 3.4 包装类与自动拆装箱

* 每种基本类型对应一个包装类；**自动装箱**：`Integer.valueOf()`；**自动拆箱**：`intValue()`
* **缓存池**：`Integer` 默认缓存 `-128 ~ 127`，`Character` 缓存 `0 ~ 127`

```java
Integer a = 127;   Integer b = 127;
System.out.println(a == b);   // true，127 在缓存池内，同一对象
Integer c = 128;   Integer d = 128;
System.out.println(c == d);   // false，超出缓存池，不同对象
System.out.println(c.equals(d)); // true — 值比较用 equals
Integer x = null;
int y = x;                     // NPE！自动拆箱 null 会抛 NullPointerException
```

### 3.5 向上转型与向下转型

* **向上转型**（子→父）：自动，安全，丢失子类特有方法
* **向下转型**（父→子）：强制 `(type)`，需 `instanceof` 检查，否则可能 `ClassCastException`

```java
Animal a = new Dog();          // 向上转型，自动
if (a instanceof Dog) {
    Dog d = (Dog) a;           // 向下转型，需强转
    d.bark();
}
```

***

## 第四部分：异常处理

### 4.1 `Throwable` 体系

```
Throwable
├── Error（JVM 级错误，不应捕获）
│   ├── OutOfMemoryError
│   └── StackOverflowError
└── Exception
    ├── RuntimeException（unchecked，编译不检查）
    │   ├── NullPointerException
    │   ├── IndexOutOfBoundsException
    │   └── IllegalArgumentException
    └── 其他 Exception（checked，必须处理）
        ├── IOException
        └── SQLException
```

* **Checked**：编译期强制处理（`try/catch` 或 `throws`）
* **Unchecked**：编译不检查，通常是程序逻辑 bug，不应捕获而是修复代码

### 4.2 异常处理语法

```java
// finally 总会执行（即使 try 中有 return）
public int test() {
    try { return 1; }
    finally { System.out.println("finally 执行"); }
}

// try-with-resources（JDK 7+）
BufferedReader br = new BufferedReader(new FileReader("a.txt"));
try (br) { return br.readLine(); }

// multi-catch（JDK 7+）
try { int[] arr = new int[2]; arr[3] = 5; }
catch (ArrayIndexOutOfBoundsException | NumberFormatException e) {
    System.out.println("统一处理: " + e.getMessage());
}
```

### 4.3 异常链与自定义异常

```java
class BusinessException extends RuntimeException {
    public BusinessException(String msg) { super(msg); }
    public BusinessException(String msg, Throwable cause) { super(msg, cause); }
}
try {
    new FileInputStream("不存在.txt");
} catch (FileNotFoundException e) {
    throw new BusinessException("读取文件失败", e);
}
```

> **最佳实践**：① 用具体异常而非泛化 `Exception`；② 异常信息包含关键参数；③ 不要空 `catch` 吞异常；④ checked 异常暴露过多实现细节时可包装为 unchecked。

***

## 第五部分：数组与常用工具类

### 5.1 数组

* **声明**：`int[] arr`（推荐）；数组是对象，`length` 是属性，存储在堆中

```java
int[] a = new int[5];              // 默认值 0，a 在栈，元素在堆
int[] b = {1, 2, 3};               // 静态初始化
int[][] matrix = {{1,2},{3,4,5}};  // 锯齿数组，每行长度可不同
int[] copy = Arrays.copyOf(b, b.length * 2);  // [1,2,3,0,0,0]
System.arraycopy(b, 0, copy, 0, b.length);    // 本地方法，高性能
```

### 5.2 `Arrays` 工具类

```java
int[] arr = {3, 1, 2};
Arrays.sort(arr);                  // 升序排序
int idx = Arrays.binarySearch(arr, 2);  // 二分查找（先排序！）
Arrays.fill(arr, 0);               // 全部填充为 0
int[] copy = Arrays.copyOfRange(arr, 0, 2); // [3,1]
String s = Arrays.toString(arr);   // "[3, 1, 2]"
List<Integer> list = Arrays.asList(1, 2, 3); // 定长，不可 add/remove
```

### 5.3 `Comparable` 与 `Comparator`

```java
// Comparable — "自己能比"
class User implements Comparable<User> {
    int age;
    public int compareTo(User o) { return Integer.compare(this.age, o.age); }
}
// Comparator — "裁判来比"
Comparator<User> byName = (u1, u2) -> u1.name.compareTo(u2.name);
users.sort(byName);
users.sort(Comparator.comparing(User::getAge).thenComparing(User::getName));
```

### 5.4 字符串家族

| 类               | 可变性 | 线程安全 | 适用场景                  |
| --------------- | --- | ---- | --------------------- |
| `String`        | 不可变 | —    | 少量字符串操作               |
| `StringBuilder` | 可变  | 否    | 单线程频繁拼接（首选）           |
| `StringBuffer`  | 可变  | 是    | 多线程拼接（`synchronized`） |

```java
String a = "abc";                  // 字面量，放入常量池
String b = "abc";                  // 常量池已有，复用
System.out.println(a == b);        // true，同一引用
String c = new String("abc");      // 强制在堆中新建对象
System.out.println(a == c);        // false
System.out.println(a.equals(c));   // true

String d = new String("abc").intern();
System.out.println(a == d);        // true

StringBuilder sb = new StringBuilder();
sb.append("Hello").append(" World");
String result = sb.toString();
```

### 5.5 其他常用工具类

```java
Math.max(1, 2);
Math.random();                     // [0.0, 1.0)
Objects.equals(a, b);              // null 安全的 equals
Objects.requireNonNull(obj, "不能为null");
Objects.hash(a, b, c);             // 方便重写 hashCode
ThreadLocalRandom.current().nextInt(1, 100); // 并发场景性能更好
```

***

## 第六部分：泛型

### 6.1 泛型基础

* **泛型类/接口**：`class Box<T>`；**泛型方法**：`<T> T method(T t)`
* **类型擦除**：编译后泛型信息被擦除，`T` 替换为边界类型（默认 `Object`）

```java
class Box<T> {
    private T item;
    public T get() { return item; }
    public void set(T item) { this.item = item; }
}
Box<String> box = new Box<>();     // JDK 7+ 钻石语法
box.set("hello");

public static <T> T getFirst(List<T> list) { return list.get(0); }
// T[] arr = new T[10];           // ❌ 编译错误！
```

### 6.2 泛型通配符

| 通配符           | 含义                         | 读/写 | 场景        |
| ------------- | -------------------------- | --- | --------- |
| `? extends T` | T 或 T 的子类（上界）              | 只读  | 生产者（读取数据） |
| `? super T`   | T 或 T 的父类（下界）              | 只写  | 消费者（写入数据） |
| `?`           | 任意类型（等价`? extends Object`） | —   | 不关心类型     |

```java
// PECS 原则：Producer Extends, Consumer Super
void print(List<? extends Number> list) {
    for (Number n : list) { System.out.println(n); }  // 可读
    // list.add(1);           // ❌ 不能写
}
void addNumbers(List<? super Integer> list) {
    list.add(1);               // ✅ 可写
    // Integer n = list.get(0); // ❌ 读出来是 Object
}
```

> **类型擦除的影响**：① 不能用 `instanceof` 检查泛型类型；② 不能 `new T()`；③ 不能创建泛型数组；④ 泛型类的静态成员不能引用类型参数。

***

## 第七部分：容器框架（集合）

### 7.1 体系概览

```
Collection                      Map
├── List（有序可重复）           ├── HashMap（数组+链表+红黑树）
│   ├── ArrayList               ├── LinkedHashMap（保持插入顺序）
│   └── LinkedList              ├── TreeMap（红黑树，按键排序）
├── Set（无序不可重复）          ├── Hashtable（遗留，线程安全）
│   ├── HashSet                 └── ConcurrentHashMap（并发安全）
│   ├── LinkedHashSet
│   └── TreeSet
└── Queue/Deque
    ├── LinkedList（双向链表）
    └── PriorityQueue（堆）
```

### 7.2 List

| 实现           | 底层   | 查询   | 增删   | 线程安全 |
| ------------ | ---- | ---- | ---- | ---- |
| `ArrayList`  | 数组   | O(1) | O(n) | 否    |
| `LinkedList` | 双向链表 | O(n) | O(1) | 否    |

```java
List<String> list = new ArrayList<>();
list.add("A"); list.add(0, "B");
list.get(0);                           // 随机访问 O(1)
list.remove(0);
list.contains("A");

LinkedList<String> linked = new LinkedList<>();
linked.addFirst("头"); linked.addLast("尾");
linked.pollFirst();
```

### 7.3 Set

* `HashSet`：基于 `HashMap`，无序，O(1)，**必须正确重写 `equals()`/`hashCode()`**
* `LinkedHashSet`：继承 `HashSet`，双向链表保持插入顺序
* `TreeSet`：红黑树，元素有序（自然序或 `Comparator`），O(log n)

```java
Set<String> hashSet = new HashSet<>();
hashSet.add("A"); hashSet.add("B"); hashSet.add("A");
System.out.println(hashSet);           // [A, B]（无序，重复 A 未加入）

Set<String> linkedSet = new LinkedHashSet<>();
linkedSet.add("B"); linkedSet.add("A");
System.out.println(linkedSet);         // [B, A]（保持插入顺序）

Set<Integer> treeSet = new TreeSet<>();
treeSet.add(3); treeSet.add(1); treeSet.add(2);
System.out.println(treeSet);           // [1, 2, 3]
```

### 7.4 Map

| 实现              | 底层           | 顺序     | null 键 | 线程安全  |
| --------------- | ------------ | ------ | ------ | ----- |
| `HashMap`       | 数组+链表+红黑树    | 无序     | ✅ 1个   | 否     |
| `LinkedHashMap` | HashMap+双向链表 | 插入/访问序 | ✅      | 否     |
| `TreeMap`       | 红黑树          | 键排序    | ❌      | 否     |
| `Hashtable`     | 数组+链表        | 无序     | ❌      | 是（淘汰） |

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1); map.put("B", 2);
map.get("A");                          // 1
map.getOrDefault("C", 0);              // 不存在返回默认值 0
map.putIfAbsent("A", 100);              // key 已存在不覆盖
map.forEach((k, v) -> System.out.println(k + ":" + v));
for (Map.Entry<String, Integer> e : map.entrySet()) {
    System.out.println(e.getKey() + "=" + e.getValue());
}
// HashMap：初始容量16，负载因子0.75，链表长度≥8且容量≥64时转红黑树
// ConcurrentHashMap：JDK7分段锁，JDK8 CAS+synchronized锁链表头节点
```

### 7.5 Queue / Deque

```java
Queue<Integer> pq = new PriorityQueue<>();
pq.offer(3); pq.offer(1); pq.offer(2);
System.out.println(pq.poll());         // 1（最小先出）

Deque<Integer> stack = new ArrayDeque<>();
stack.push(1); stack.push(2);
int top = stack.pop();                 // 2（后进先出）

Deque<Integer> queue = new ArrayDeque<>();
queue.offer(1); queue.offer(2);
int head = queue.poll();               // 1（先进先出）
```

### 7.6 迭代器与 Collections

```java
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String s = it.next();
    if ("A".equals(s)) it.remove();    // 安全删除
}
// foreach 是迭代器语法糖；遍历中修改集合→ConcurrentModificationException

Collections.sort(list);
Collections.shuffle(list);
Collections.unmodifiableList(list);    // 不可变集合
Collections.synchronizedList(list);    // 同步包装
```

***

## 第八部分：IO 流（基础）

### 8.1 流分类

| 分类   | 基类/接口          | 典型实现                                                 |
| ---- | -------------- | ---------------------------------------------------- |
| 字节输入 | `InputStream`  | `FileInputStream`, `BufferedInputStream`             |
| 字节输出 | `OutputStream` | `FileOutputStream`, `BufferedOutputStream`           |
| 字符输入 | `Reader`       | `FileReader`, `BufferedReader`, `InputStreamReader`  |
| 字符输出 | `Writer`       | `FileWriter`, `BufferedWriter`, `OutputStreamWriter` |

* **字节流与字符流的桥梁**：`InputStreamReader`（字节→字符）、`OutputStreamWriter`（字符→字节）

### 8.2 文件流 + 缓冲流

```java
// 字节流读文件
try (FileInputStream fis = new FileInputStream("in.txt");
     BufferedInputStream bis = new BufferedInputStream(fis)) {
    byte[] buf = new byte[1024];
    int len;
    while ((len = bis.read(buf)) != -1) {
        System.out.print(new String(buf, 0, len));
    }
}

// 字符流按行读（最常用）
try (BufferedReader br = new BufferedReader(new FileReader("in.txt"))) {
    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }
}

// 字符流写入
try (BufferedWriter bw = new BufferedWriter(new FileWriter("out.txt"))) {
    bw.write("Hello World");
    bw.newLine();                  // 跨平台换行
}
```

### 8.3 数据流与对象流

```java
// DataStream — 读写基本类型
try (DataOutputStream dos = new DataOutputStream(new FileOutputStream("data.bin"))) {
    dos.writeInt(42); dos.writeDouble(3.14); dos.writeUTF("你好");
}
try (DataInputStream dis = new DataInputStream(new FileInputStream("data.bin"))) {
    int i = dis.readInt();         // 必须按写入顺序读取
    double d = dis.readDouble();
    String s = dis.readUTF();
}

// ObjectStream — 对象序列化
class User implements Serializable {
    private static final long serialVersionUID = 1L;
    String name;
    transient String password;                 // transient 不参与序列化
}
try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("user.obj"))) {
    oos.writeObject(new User());
}
try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("user.obj"))) {
    User u = (User) ois.readObject();          // 反序列化不调用构造方法
}
```

### 8.4 Apache Commons IO

```java
File file = new File("test.txt");
String content = FileUtils.readFileToString(file, "UTF-8");
List<String> lines = FileUtils.readLines(file, "UTF-8");
FileUtils.writeStringToFile(file, "Hello", "UTF-8");
IOUtils.copy(inputStream, outputStream);
```

***

## 第九部分：NIO（New I/O）—— IO 进阶

### 9.1 核心组件

| 组件         | 作用                                        |
| ---------- | ----------------------------------------- |
| `Buffer`   | 内存缓冲区，所有数据读写都经过 Buffer（`ByteBuffer` 最常用）  |
| `Channel`  | 双向通道，可读可写（`FileChannel`, `SocketChannel`） |
| `Selector` | 多路复用器，单线程管理多个 Channel                     |

**NIO vs 传统 IO**：IO 面向流（单向）、阻塞；NIO 面向缓冲区（双向）、非阻塞模式。

### 9.2 Buffer 操作

```java
ByteBuffer buf = ByteBuffer.allocate(1024);  // 堆内分配
// ByteBuffer buf = ByteBuffer.allocateDirect(1024); // 堆外分配（零拷贝，创建成本高）

buf.put("hello".getBytes());     // 写入 → position 后移
buf.flip();                      // 切换为读模式：limit=position, position=0
byte[] dst = new byte[buf.limit()];
buf.get(dst);                    // 读取
buf.clear();                     // 切换为写模式：position=0, limit=capacity
```

### 9.3 Channel 与 Selector

```java
// FileChannel — 零拷贝
try (FileChannel src = FileChannel.open(Paths.get("src.txt"), StandardOpenOption.READ);
     FileChannel dst = FileChannel.open(Paths.get("dst.txt"), StandardOpenOption.WRITE, StandardOpenOption.CREATE)) {
    src.transferTo(0, src.size(), dst);
}

// Selector — 多路复用（单线程管理多连接）
try (ServerSocketChannel server = ServerSocketChannel.open();
     Selector selector = Selector.open()) {
    server.bind(new InetSocketAddress(8080));
    server.configureBlocking(false);       // 非阻塞模式
    server.register(selector, SelectionKey.OP_ACCEPT);

    while (true) {
        selector.select();                 // 阻塞直到有事件就绪
        for (SelectionKey key : selector.selectedKeys()) {
            if (key.isAcceptable()) { /* 处理新连接 */ }
            if (key.isReadable())   { /* 处理读事件 */ }
        }
        selector.selectedKeys().clear();
    }
}
```

> **AIO（NIO 2.0）**：真正的异步非阻塞 I/O，基于回调（`CompletionHandler`），读写操作由操作系统完成后通知应用。

***

## 第十部分：多线程（基础）

### 10.1 线程创建方式

| 方式                        | 特点               | 返回值 |
| ------------------------- | ---------------- | --- |
| 继承`Thread`                | 单继承限制，不推荐        | 无   |
| 实现`Runnable`              | 无返回值，推荐（解耦任务与线程） | 无   |
| `Callable` + `FutureTask` | 有返回值，可抛异常        | 有   |

```java
// 继承 Thread
class MyThread extends Thread {
    public void run() { System.out.println("Thread 运行"); }
}
new MyThread().start();

// 实现 Runnable（推荐）
class MyTask implements Runnable {
    public void run() { System.out.println("Runnable 运行"); }
}
new Thread(new MyTask()).start();
new Thread(() -> System.out.println("Lambda 线程")).start();

// Callable + FutureTask（有返回值）
Callable<Integer> task = () -> { Thread.sleep(100); return 42; };
FutureTask<Integer> ft = new FutureTask<>(task);
new Thread(ft).start();
Integer result = ft.get();               // 阻塞等待结果
```

### 10.2 线程生命周期

```
NEW → RUNNABLE → BLOCKED/WAITING/TIMED_WAITING → TERMINATED
     ↑______________↓ (就绪 ⇄ 运行，由 CPU 调度切换)
```

| 状态              | 说明                  | 进入方式                              |
| --------------- | ------------------- | --------------------------------- |
| `NEW`           | 创建但未`start()`       | `new Thread()`                    |
| `RUNNABLE`      | 就绪或运行中（含 CPU 时间片切换） | `start()` 后                       |
| `BLOCKED`       | 等待`synchronized` 锁  | 竞争锁失败                             |
| `WAITING`       | 无限等待，需其他线程唤醒        | `wait()`, `join()`, `park()`      |
| `TIMED_WAITING` | 限时等待，超时自动唤醒         | `sleep()`, `wait(ms)`, `join(ms)` |
| `TERMINATED`    | 执行完毕                | `run()` 结束                        |

### 10.3 线程常用方法

```java
Thread t = new Thread(() -> {
    while (!Thread.currentThread().isInterrupted()) { /* 检查中断标志 */ }
});
t.start();
t.join();                  // 当前线程等待 t 执行完毕
Thread.sleep(1000);        // 当前线程休眠 1s，不释放锁
Thread.yield();            // 暗示让出 CPU（不保证）

t.interrupt();             // 设置中断标志位（不会强制停止）

// 守护线程
Thread daemon = new Thread(() -> { while(true){} });
daemon.setDaemon(true);    // 必须在 start() 前设置
daemon.start();
```

### 10.4 `synchronized` 与 `volatile`

```java
// synchronized — 保证原子性、可见性、有序性（偏向锁→轻量锁→重量锁）
class Counter {
    private int count = 0;
    public synchronized void increment() { count++; }  // 同步方法，锁 this
    public void decrement() {
        synchronized (this) { count--; }               // 同步代码块
    }
    public static synchronized void staticMethod() {}  // 锁 class 对象
}

// volatile — 保证可见性 + 禁止指令重排，不保证原子性
class Flag {
    volatile boolean running = true;   // 修改后立即刷回主存，其他线程立即可见
    // 适用：状态标志、DCL单例；不适用：count++（复合操作）
}
```

### 10.5 `wait` / `notify` / `notifyAll`

> 必须持有对象锁（在 `synchronized` 块内）才能调用；`wait()` 释放锁，`sleep()` 不释放锁。

```java
class Message {
    private String msg;
    private boolean empty = true;

    public synchronized String take() {
        while (empty) {                // 用 while 而非 if，防止虚假唤醒
            try { wait(); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
        }
        empty = true;
        notifyAll();
        return msg;
    }

    public synchronized void put(String msg) {
        while (!empty) { try { wait(); } catch (InterruptedException e) {} }
        this.msg = msg;
        empty = false;
        notifyAll();
    }
}
```

***

## 第十一部分：JUC 并发包 —— 多线程进阶

### 11.1 线程池

**`ThreadPoolExecutor` 核心 7 参数**：核心线程数、最大线程数、空闲存活时间、时间单位、工作队列、线程工厂、拒绝策略。执行流程：核心线程 → 工作队列 → 最大线程 → 拒绝策略。

```java
ThreadPoolExecutor pool = new ThreadPoolExecutor(
    2,                                // 核心线程数
    4,                                // 最大线程数
    60, TimeUnit.SECONDS,             // 空闲存活时间
    new LinkedBlockingQueue<>(100),   // 有界队列（必须！防止 OOM）
    new ThreadPoolExecutor.CallerRunsPolicy() // 拒绝策略
);
pool.execute(() -> System.out.println("执行任务"));

CompletableFuture.supplyAsync(() -> "Hello", pool)
    .thenApply(String::toUpperCase)
    .thenAccept(System.out::println);

pool.shutdown();
```

| 拒绝策略                  | 行为                            |
| --------------------- | ----------------------------- |
| `AbortPolicy`（默认）     | 抛`RejectedExecutionException` |
| `CallerRunsPolicy`    | 由提交任务的线程自己执行                  |
| `DiscardPolicy`       | 直接丢弃新任务，不抛异常                  |
| `DiscardOldestPolicy` | 丢弃队首任务，重试提交                   |

### 11.2 锁框架

| 锁                        | 特性                                         |
| ------------------------ | ------------------------------------------ |
| `ReentrantLock`          | 可重入、可中断、可限时、可公平（默认非公平）、**必须手动 `unlock()`** |
| `ReentrantReadWriteLock` | 读读共享，读写互斥，写写互斥，适合读多写少                      |
| `StampedLock`            | 乐观读（不阻塞读），性能更高，不可重入                        |

```java
ReentrantLock lock = new ReentrantLock();
lock.lock();
try { /* 临界区代码 */ } finally { lock.unlock(); } // 必须在 finally 释放！

lock.lockInterruptibly();          // 等待锁时可被 interrupt() 中断
if (lock.tryLock(1, TimeUnit.SECONDS)) { /* ... */ }

ReadWriteLock rwLock = new ReentrantReadWriteLock();
rwLock.readLock().lock();          // 多个线程可同时持有读锁
rwLock.writeLock().lock();         // 写锁独占
```

### 11.3 原子类

```java
AtomicInteger ai = new AtomicInteger(0);
ai.incrementAndGet();              // 原子自增 → 1
ai.compareAndSet(1, 100);          // 如果当前值==1，则设为100
int old = ai.getAndUpdate(x -> x * 2); // 原子更新

LongAdder adder = new LongAdder(); // 高并发累加首选
adder.increment();
long sum = adder.sum();
```

### 11.4 并发容器

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
map.put("key", 1);
map.computeIfAbsent("key2", k -> k.length()); // 原子计算

CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>();
cowList.add("A");                  // 写时复制整个数组，昂贵

BlockingQueue<String> queue = new ArrayBlockingQueue<>(10);
queue.put("item");                 // 队列满则阻塞
String item = queue.take();        // 队列空则阻塞
```

### 11.5 同步工具

```java
// CountDownLatch — 倒计时门闩
CountDownLatch latch = new CountDownLatch(3);
for (int i = 0; i < 3; i++) new Thread(() -> { /* work */ latch.countDown(); }).start();
latch.await();

// CyclicBarrier — 循环栅栏
CyclicBarrier barrier = new CyclicBarrier(3, () -> System.out.println("全员到齐！"));
for (int i = 0; i < 3; i++)
    new Thread(() -> { try { barrier.await(); } catch (Exception e) {} }).start();

// Semaphore — 信号量，限流
Semaphore semaphore = new Semaphore(3);
semaphore.acquire();
semaphore.release();

// CompletableFuture — 异步编排
CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> f2 = CompletableFuture.supplyAsync(() -> "World");
f1.thenCombine(f2, (a, b) -> a + " " + b).thenAccept(System.out::println); // "Hello World"
```

### 11.6 ThreadLocal

* 每个线程拥有独立副本，底层 `ThreadLocalMap`（Thread的成员变量），Key为`ThreadLocal`弱引用
* **必须 `remove()`**，否则线程池场景会导致内存泄漏

```java
ThreadLocal<String> tl = new ThreadLocal<>();
tl.set("线程A的数据");
String data = tl.get();
tl.remove();                       // 用完后必须 remove，防止内存泄漏
```

***

## 第十二部分：网络编程

### 12.1 TCP 编程

* TCP：面向连接、可靠、有序、字节流
* `ServerSocket` 绑定端口等待连接，`Socket` 建立连接传输数据

```java
// 服务端
try (ServerSocket server = new ServerSocket(8080)) {
    while (true) {
        Socket client = server.accept();          // 阻塞等待连接
        new Thread(() -> {
            try (BufferedReader in = new BufferedReader(new InputStreamReader(client.getInputStream()));
                 PrintWriter out = new PrintWriter(client.getOutputStream(), true)) {
                String msg = in.readLine();       // 读取客户端消息
                out.println("Echo: " + msg);      // 回复
            } catch (IOException e) { e.printStackTrace(); }
        }).start();
    }
}

// 客户端
try (Socket socket = new Socket("localhost", 8080);
     PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
     BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()))) {
    out.println("Hello Server");                  // 发送
    String response = in.readLine();              // 接收
    System.out.println(response);                 // "Echo: Hello Server"
}
```

### 12.2 UDP 编程

* UDP：无连接、不可靠、报文（数据报）、速度快

```java
// 接收端
DatagramSocket receiver = new DatagramSocket(9090);
byte[] buf = new byte[1024];
DatagramPacket packet = new DatagramPacket(buf, buf.length);
receiver.receive(packet);          // 阻塞等待
String msg = new String(packet.getData(), 0, packet.getLength());

// 发送端
DatagramSocket sender = new DatagramSocket();
byte[] data = "Hello UDP".getBytes();
DatagramPacket p = new DatagramPacket(data, data.length, InetAddress.getByName("localhost"), 9090);
sender.send(p);
```

***

## 第十三部分：反射、类加载机制与动态代理

### 13.1 反射（Reflection）

* **获取 `Class` 对象三种方式**：`类名.class`、`对象.getClass()`、`Class.forName("全限定名")`
* 反射破坏封装，性能低于直接调用，但框架（Spring、MyBatis）依赖它

```java
// 获取 Class 对象
Class<?> clazz = Class.forName("com.example.User");  // 方式一：全限定名
Class<?> clazz2 = User.class;                         // 方式二：类名.class
Class<?> clazz3 = new User().getClass();              // 方式三：对象.getClass()

// 反射创建实例
Constructor<?> cons = clazz.getDeclaredConstructor(String.class, int.class);
Object obj = cons.newInstance("张三", 25);           // 等价于 new User("张三", 25)
// 调用私有构造
Constructor<?> privateCons = clazz.getDeclaredConstructor();
privateCons.setAccessible(true);                      // 绕过访问检查
Object obj2 = privateCons.newInstance();

// 反射调用方法
Method method = clazz.getDeclaredMethod("setName", String.class);
method.invoke(obj, "李四");                           // obj.setName("李四")
// 静态方法 invoke 传 null
Method staticMethod = clazz.getDeclaredMethod("staticMethod");
staticMethod.invoke(null);

// 反射读写属性
Field field = clazz.getDeclaredField("name");
field.setAccessible(true);                             // 私有字段必须
field.set(obj, "王五");                                // 写
String name = (String) field.get(obj);                 // 读
```

### 13.2 类加载机制（ClassLoader）

* **类加载过程**：加载（读取字节码）→ 验证（检查格式）→ 准备（分配内存、默认值）→ 解析（符号引用→直接引用）→ 初始化（执行 `<clinit>`，静态块+静态变量赋值）
* **双亲委派模型**：AppClassLoader → ExtClassLoader → BootstrapClassLoader，从下往上委派，从上往下查找

```java
// 获取类加载器
ClassLoader loader = String.class.getClassLoader();       // null（Bootstrap，C++ 实现）
ClassLoader appLoader = MyClass.class.getClassLoader();   // AppClassLoader

// Class.forName vs ClassLoader.loadClass
Class.forName("com.example.User");          // 会触发初始化（执行静态块）
ClassLoader.getSystemClassLoader().loadClass("com.example.User"); // 不触发初始化
```

### 13.3 动态代理（Dynamic Proxy）

| 方式       | 原理                            | 要求         | 性能 |
| -------- | ----------------------------- | ---------- | -- |
| JDK 动态代理 | `Proxy` + `InvocationHandler` | 必须有接口      | 较高 |
| CGLIB    | 生成子类（ASM 字节码）                 | 类不能`final` | 较高 |

```java
// JDK 动态代理
interface Service { void doSomething(); }
class ServiceImpl implements Service {
    public void doSomething() { System.out.println("执行业务"); }
}
// 代理处理器
class LogHandler implements InvocationHandler {
    private final Object target;
    public LogHandler(Object target) { this.target = target; }
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("【前置日志】调用 " + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("【后置日志】完成");
        return result;
    }
}
// 创建代理
Service proxy = (Service) Proxy.newProxyInstance(
    Service.class.getClassLoader(),
    new Class<?>[]{Service.class},
    new LogHandler(new ServiceImpl())
);
proxy.doSomething(); // 输出: 【前置日志】→ 执行业务 → 【后置日志】

// CGLIB 代理（需引入 cglib 依赖）
// Enhancer enhancer = new Enhancer();
// enhancer.setSuperclass(ServiceImpl.class);
// enhancer.setCallback((MethodInterceptor) (obj, method, args, methodProxy) -> {
//     System.out.println("CGLIB 前置");
//     return methodProxy.invokeSuper(obj, args);
// });
// ServiceImpl proxy = (ServiceImpl) enhancer.create();
```

> **应用场景**：Spring AOP（默认 JDK 代理，无接口则 CGLIB）、MyBatis Mapper 代理、RPC 远程调用。

***

## 第十四部分：注解（Annotation）

### 14.1 内置注解与元注解

| 注解                     | 作用                              |
| ---------------------- | ------------------------------- |
| `@Override`            | 检查方法是否正确重写父类方法                  |
| `@Deprecated`          | 标记已过时（`since`, `forRemoval` 属性） |
| `@SuppressWarnings`    | 抑制编译器警告                         |
| `@FunctionalInterface` | 检查是否为函数式接口（只能一个抽象方法）            |
| `@SafeVarargs`         | 抑制泛型可变参数警告                      |

**元注解**（修饰注解的注解）：

| 元注解           | 作用                                            |
| ------------- | --------------------------------------------- |
| `@Retention`  | 注解保留阶段：`SOURCE`/`CLASS`/`RUNTIME`（常用 RUNTIME） |
| `@Target`     | 注解可用位置：`TYPE`/`METHOD`/`FIELD` 等              |
| `@Documented` | 是否包含在 JavaDoc                                 |
| `@Inherited`  | 子类是否继承父类注解                                    |

### 14.2 自定义注解 + 反射解析

```java
// 定义注解
@Retention(RetentionPolicy.RUNTIME)  // 运行时可通过反射读取
@Target({ElementType.TYPE, ElementType.METHOD})
@interface Info {
    String value() default "";       // 属性（非抽象方法写法）
    int version() default 1;
}

// 使用注解
@Info(value = "用户服务", version = 2)
class UserService {
    @Info("查询方法")
    public void query() {}
}

// 反射解析注解
Class<?> clazz = UserService.class;
if (clazz.isAnnotationPresent(Info.class)) {
    Info info = clazz.getAnnotation(Info.class);
    System.out.println(info.value());  // "用户服务"
}
```

***

## 第十五部分：Lambda 与函数式编程

### Lambda 表达式与函数式接口

* **Lambda 语法**：`(参数) -> { 代码 }`，本质是函数式接口的匿名实现
* **函数式接口**：只有一个抽象方法的接口（`@FunctionalInterface`）

| 内置函数式接口             | 方法                | 输入   | 输出      | 用途      |
| ------------------- | ----------------- | ---- | ------- | ------- |
| `Predicate<T>`      | `boolean test(T)` | T    | boolean | 判断/过滤   |
| `Consumer<T>`       | `void accept(T)`  | T    | void    | 消费数据    |
| `Function<T,R>`     | `R apply(T)`      | T    | R       | 转换/映射   |
| `Supplier<T>`       | `T get()`         | void | T       | 提供数据    |
| `UnaryOperator<T>`  | `T apply(T)`      | T    | T       | 同类型一元运算 |
| `BiFunction<T,U,R>` | `R apply(T,U)`    | T,U  | R       | 双参数转换   |

```java
// Lambda 示例
Predicate<String> isEmpty = s -> s == null || s.isEmpty();
Consumer<String> print = s -> System.out.println(s);
Function<String, Integer> len = String::length;   // 方法引用
Supplier<Double> random = Math::random;

List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.stream()
    .filter(s -> s.length() > 3)   // Predicate
    .map(String::toUpperCase)      // Function
    .forEach(System.out::println); // Consumer  → 输出: ALICE CHARLIE
```

* **方法引用**：`类名::静态方法`、`对象::实例方法`、`类名::实例方法`（第一个参数作为调用者）、`类名::new`

### 15.2 Stream API

```java
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5, 6);

// 中间操作（惰性求值，不触发计算）
List<Integer> result = nums.stream()
    .filter(n -> n % 2 == 0)       // 过滤偶数 → [2,4,6]
    .map(n -> n * n)               // 平方 → [4,16,36]
    .sorted(Comparator.reverseOrder()) // 倒序 → [36,16,4]
    .distinct()                    // 去重
    .skip(1)                       // 跳过前1 → [16,4]
    .limit(1)                      // 取前1 → [16]
    .collect(Collectors.toList()); // 终端操作 → [16]

// 常用终端操作
long count = nums.stream().count();                    // 计数
boolean any = nums.stream().anyMatch(n -> n > 5);      // 是否有 >5
Optional<Integer> first = nums.stream().findFirst();   // 第一个
nums.stream().reduce(0, Integer::sum);                 // 累加求和
```

***

## 第十六部分：常用工具类补充

### 16.1 `Optional` 类（避免空指针）

* 容器对象，明确表示"可能为 null"，强制使用者处理空值情况

```java
// 创建
Optional<String> opt1 = Optional.of("hello");           // 值不能为 null
Optional<String> opt2 = Optional.ofNullable(maybeNull); // 值可为 null
Optional<String> opt3 = Optional.empty();               // 空 Optional

// 常用方法
opt1.isPresent();                 // 是否有值
opt1.ifPresent(v -> System.out.println(v));  // 有值则消费
String val = opt2.orElse("默认");  // 为 null 返回默认值
String val2 = opt2.orElseGet(() -> heavyCompute()); // 懒加载默认值
String val3 = opt2.orElseThrow(() -> new RuntimeException("值为空"));

// 链式转换
Optional<Integer> len = opt1.map(String::length);       // 有值则转换
Optional<String> flat = opt1.flatMap(v -> Optional.of(v.toUpperCase()));
Optional<String> filtered = opt1.filter(v -> v.length() > 3);
```

### 16.2 时间 API（`java.time`，Java 8+）

| 类               | 说明           | 示例                                         |
| --------------- | ------------ | ------------------------------------------ |
| `LocalDate`     | 日期（不含时间）     | `2026-06-08`                               |
| `LocalTime`     | 时间（不含日期）     | `14:30:00`                                 |
| `LocalDateTime` | 日期+时间（不含时区）  | `2026-06-08T14:30:00`                      |
| `ZonedDateTime` | 带时区的日期时间     | `2026-06-08T14:30:00+08:00[Asia/Shanghai]` |
| `Instant`       | 时间戳（UTC 时间点） | 机器可读时间                                     |
| `Duration`      | 时间间隔（时分秒）    | `PT2H30M`                                  |
| `Period`        | 日期间隔（年月日）    | `P1Y2M3D`                                  |

```java
// 创建
LocalDate date = LocalDate.now();          // 当前日期
LocalDate date2 = LocalDate.of(2026, 6, 8);
LocalTime time = LocalTime.of(14, 30);
LocalDateTime dt = LocalDateTime.of(date, time);

// 运算（返回新对象，不修改原对象）
LocalDate nextWeek = date.plusWeeks(1);    // +1周
LocalDate lastMonth = date.minusMonths(1); // -1月
LocalDate firstDay = date.withDayOfMonth(1); // 本月第1天

// 格式化
DateTimeFormatter fmt = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String str = dt.format(fmt);
LocalDateTime parsed = LocalDateTime.parse(str, fmt);

// 与旧 API 互转
Date oldDate = Date.from(Instant.now());
Instant instant = oldDate.toInstant();
```

***

## 第十七部分：数据结构与简单算法

### 17.1 数据结构基础

| 结构 | Java 实现                       | 特点                   |
| -- | ----------------------------- | -------------------- |
| 堆  | `PriorityQueue`               | 小顶堆（默认），O(log n)     |
| 栈  | `ArrayDeque`（推荐）/ `Stack`（遗留） | LIFO，`push/pop/peek` |
| 链表 | `LinkedList`                  | 双向链表，增删 O(1)         |
| 树  | `TreeMap` / `TreeSet`         | 红黑树，自平衡二叉搜索树         |

```java
// 大顶堆
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.offer(1); maxHeap.offer(3); maxHeap.offer(2);
System.out.println(maxHeap.poll());    // 3（最大先出）

// 栈（用 ArrayDeque，Stack 已淘汰）
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1); stack.push(2);
int top = stack.peek();                // 2（查看栈顶，不弹出）
int pop = stack.pop();                 // 2（弹出）
```

### 17.2 常用算法

```java
// 递归 — 斐波那契（仅示例，生产用迭代或记忆化）
int fib(int n) { return n <= 1 ? n : fib(n - 1) + fib(n - 2); }

// 快速排序（双指针分治）
void quickSort(int[] arr, int low, int high) {
    if (low >= high) return;
    int pivot = arr[low], i = low, j = high;
    while (i < j) {
        while (i < j && arr[j] >= pivot) j--;
        arr[i] = arr[j];
        while (i < j && arr[i] <= pivot) i++;
        arr[j] = arr[i];
    }
    arr[i] = pivot;
    quickSort(arr, low, i - 1);
    quickSort(arr, i + 1, high);
}

// 二分查找（要求有序）
int binarySearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;  // 防溢出
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
```

***

## 第十八部分：JVM 内存结构与垃圾回收

### 18.1 运行时数据区

| 区域                    | 线程共享 | 存储内容                    |
| --------------------- | ---- | ----------------------- |
| 堆（Heap）               | 是    | 所有对象实例、数组；分为新生代、老年代     |
| 元空间（Metaspace, JDK8+） | 是    | 类元信息、方法代码结构；直接内存，默认无上限  |
| 虚拟机栈（VM Stack）        | 否    | 栈帧：局部变量表、操作数栈、方法指针、返回地址 |
| 程序计数器（PC Register）    | 否    | 当前线程执行的字节码行号            |
| 本地方法栈（Native Stack）   | 否    | 执行 native 方法            |
| 字符串常量池                | 是    | 字符串字面量（JDK7+ 移到堆中）      |

### 18.2 对象创建与内存布局

1. **类加载**：元空间加载类字节码，生成方法代码结构
2. **栈帧准备**：在虚拟机栈中为方法创建栈帧，声明引用变量
3. **堆分配**：在堆的新生代分配对象内存；字符串常量池懒加载
4. **构造方法执行**：压入构造方法栈帧 → 执行初始化 → 弹出 → 对象地址赋值给栈中变量
5. **方法调用**：通过栈帧中的方法指针找到元空间中方法代码执行

```java
// 对象内存模型示意
Person p1 = new Person();    // p1 在栈（局部变量表），new Person() 在堆
p1.name = "张三";            // "张三" 在字符串常量池（堆中）
// 对象头(Mark Word + Klass Pointer) + 实例数据(name,age) + 对齐填充
```

### 18.3 垃圾回收（GC）

**堆分区**：新生代（Eden + Survivor S0 + Survivor S1，默认 8:1:1）、老年代。

**GC 类型**：

* **Minor GC**：只回收新生代，Eden 满触发，频繁、速度快
* **Major GC / Full GC**：回收老年代+元空间，STW 时间长，尽量避免

**回收算法**：

| 算法    | 原理            | 优点  | 缺点     | 应用           |
| ----- | ------------- | --- | ------ | ------------ |
| 标记-清除 | 标记存活 → 清除未标记  | 简单  | 内存碎片   | CMS 老年代      |
| 标记-复制 | 标记存活 → 复制到新区域 | 无碎片 | 浪费一半空间 | 新生代（S0⇄S1）   |
| 标记-整理 | 标记存活 → 向一端移动  | 无碎片 | 耗时较长   | Serial Old 等 |

**GC 判定**：

* **引用计数法**：被引用时+1，释放时-1，为0回收（JVM不用，无法解决循环引用）
* **可达性分析**（JVM 使用）：从 GC Roots 出发，不可达的对象判定为可回收
  * GC Roots：栈中引用、静态变量、常量池引用、JNI 引用等

**常用 GC 收集器**：

| 收集器               | 区域  | 算法    | 特点                   |
| ----------------- | --- | ----- | -------------------- |
| Serial            | 新生代 | 标记-复制 | 单线程，Client 模式默认      |
| Parallel Scavenge | 新生代 | 标记-复制 | 多线程，吞吐量优先            |
| ParNew            | 新生代 | 标记-复制 | 多线程，配合 CMS           |
| CMS               | 老年代 | 标记-清除 | 低延迟，并发回收（JDK14 移除）   |
| G1（JDK9 默认）       | 整堆  | 分区+复制 | 低延迟+可控停顿，分区回收        |
| ZGC               | 整堆  | 染色指针  | 超低延迟（<1ms），JDK15+ 生产 |

**对象晋升**：大对象直接进入老年代；长期存活（默认15次 Minor GC）从新生代 → 老年代。

### 18.4 类加载过程

```
加载 → 验证 → 准备 → 解析 → 初始化
(Loading)  (Verification)  (Preparation)  (Resolution)  (Initialization)

- 加载：通过全限定名获取字节码 → 生成 Class 对象
- 验证：检查 class 文件格式、元数据、字节码、符号引用
- 准备：为 static 变量分配内存并设默认值（非代码中赋值）
- 解析：符号引用 → 直接引用（内存地址）
- 初始化：执行 <clinit>()，即 static 块 + static 变量赋值
```

```java
// 主动引用（会触发初始化）
// ① new 对象          ② 读写静态字段（非 final）
// ③ 调用静态方法      ④ 反射 forName()
// ⑤ 初始化子类时先初始化父类  ⑥ main 方法所在类

// 被动引用（不会触发初始化）
class Parent { static { System.out.println("Parent init"); } static int v = 1; }
class Child extends Parent { static { System.out.println("Child init"); } }
System.out.println(Child.v);  // 输出 Parent init → 1（子类不初始化！）
```

### 18.5 Java 内存模型（JMM）

> JMM 定义多线程环境下共享变量的**可见性**、**有序性**、**原子性**规则。

* **主内存**：所有线程共享，存放共享变量
* **工作内存**：每个线程私有，存放共享变量副本
* **8 种原子操作**：lock/unlock/read/load/use/assign/store/write
* **happens-before 原则**：程序次序、锁、volatile、传递性、start/join 等

```java
// volatile 保证可见性 + 禁止指令重排
volatile boolean flag = false;
// 线程A: flag = true;   → 立即写回主内存
// 线程B: if (flag) {}   → 每次都从主内存读，保证可见

// DCL 单例 — volatile 禁止指令重排（否则可能拿到半初始化对象）
class Singleton {
    private static volatile Singleton instance;     // volatile 必须！
    public static Singleton getInstance() {
        if (instance == null) {                     // 第一次检查
            synchronized (Singleton.class) {
                if (instance == null) {             // 第二次检查
                    instance = new Singleton();     // 非原子：分配内存→初始化→赋值
                }
            }
        }
        return instance;
    }
}
```

***
