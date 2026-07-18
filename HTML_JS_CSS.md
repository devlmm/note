# HTML、HTML5、CSS、JavaScript 核心技术手册 

---

## 目录

### 1. HTML 基础
- [1.1 HTML 文档结构](#11-html-文档结构)
- [1.2 常用标签](#12-常用标签)
- [1.3 表单元素](#13-表单元素)
- [1.4 语义化标签](#14-语义化标签)

### 2. HTML5 新特性
- [2.1 HTML5 语义化标签](#21-html5-语义化标签)
- [2.2 多媒体标签](#22-多媒体标签)
- [2.3 表单新特性](#23-表单新特性)
- [2.4 本地存储](#24-本地存储)
- [2.5 Canvas 画布](#25-canvas-画布)

### 3. CSS 核心
- [3.1 CSS 选择器](#31-css-选择器)
- [3.2 盒模型](#32-盒模型)
- [3.3 布局方式](#33-布局方式)
- [3.4 定位机制](#34-定位机制)
- [3.5 响应式设计](#35-响应式设计)
- [3.6 动画与过渡](#36-动画与过渡)

### 4. JavaScript 核心
- [4.1 变量与数据类型](#41-变量与数据类型)
- [4.2 运算符与表达式](#42-运算符与表达式)
- [4.3 流程控制](#43-流程控制)
- [4.4 函数](#44-函数)
- [4.5 对象与数组](#45-对象与数组)
- [4.6 DOM 操作](#46-dom-操作)
- [4.7 事件处理](#47-事件处理)
- [4.8 ES6+ 新特性](#48-es6-新特性)
- [4.9 闭包与作用域](#49-闭包与作用域)
- [4.10 原型链与继承](#410-原型链与继承)
- [4.11 事件循环](#411-事件循环)

---

## 1. HTML 基础

### 1.1 HTML 文档结构

HTML（HyperText Markup Language）是构建网页的基础标记语言。

```html
<!DOCTYPE html>          <!-- 声明文档类型为 HTML5 -->
<html lang="zh-CN">      <!-- 根元素，指定语言 -->
<head>                   <!-- 头部，包含元信息 -->
    <meta charset="UTF-8">
    <title>页面标题</title>
</head>
<body>                   <!-- 主体，显示页面内容 -->
    <h1>Hello World</h1>
</body>
</html>
```

**关键点：**
- `<!DOCTYPE>` 声明必须放在文档最顶部
- `<html>` 是文档根元素
- `<head>` 包含页面元数据（不可见）
- `<body>` 包含可见内容

### 1.2 常用标签

```html
<!-- 文本标签 -->
<h1>一级标题</h1>        <!-- 最大标题，h1-h6递减 -->
<p>段落文本</p>           <!-- 段落 -->
<span>行内文本</span>     <!-- 行内容器 -->
<br>                     <!-- 换行 -->

<!-- 链接与图片 -->
<a href="https://example.com">链接</a>
<img src="image.jpg" alt="描述文字">

<!-- 列表 -->
<ul>                     <!-- 无序列表 -->
    <li>项目1</li>
</ul>
<ol>                     <!-- 有序列表 -->
    <li>项目A</li>
</ol>
```

### 1.3 表单元素

表单用于收集用户输入：

```html
<form action="/submit" method="post">
    <input type="text" name="username" placeholder="用户名">
    <input type="password" name="pwd">
    <input type="radio" name="gender" value="male">男
    <input type="checkbox" name="hobby" value="code">编程
    <select name="city">
        <option value="bj">北京</option>
    </select>
    <textarea name="msg"></textarea>
    <button type="submit">提交</button>
</form>
```

**常用 input 类型：**
- `text` - 文本框
- `password` - 密码框
- `email` - 邮箱验证
- `number` - 数字输入
- `date` - 日期选择

### 1.4 语义化标签

语义化标签让页面结构更清晰，利于SEO和可访问性：

```html
<div>通用容器</div>      <!-- 无语义块级容器 -->
<span>行内容器</span>    <!-- 无语义行内容器 -->
<header>页面头部</header>
<nav>导航栏</nav>
<main>主要内容</main>
<footer>页面底部</footer>
```

**语义化优势：**
- 代码可读性更好
- 搜索引擎更易理解内容
- 屏幕阅读器支持更佳

---

## 2. HTML5 新特性

### 2.1 HTML5 语义化标签

HTML5 引入了更多语义化标签，使页面结构更加清晰：

```html
<article>                  <!-- 独立文章内容 -->
    <header><h2>文章标题</h2></header>
    <section>内容段落</section>
</article>
<aside>侧边栏</aside>       <!-- 辅助内容 -->
<figure>                   <!-- 图文组合 -->
    <img src="pic.jpg">
    <figcaption>图片说明</figcaption>
</figure>
<mark>高亮文本</mark>       <!-- 高亮标记 -->
<time datetime="2024-01-01">2024年</time>
```

**语义化标签分类：**
- 页面结构：`<header>`, `<nav>`, `<main>`, `<footer>`
- 内容分组：`<section>`, `<article>`, `<aside>`
- 文本语义：`<mark>`, `<time>`, `<figure>`

### 2.2 多媒体标签

HTML5 原生支持音频和视频，无需插件：

```html
<!-- 音频 -->
<audio src="music.mp3" controls>
    您的浏览器不支持音频
</audio>

<!-- 视频 -->
<video src="video.mp4" controls width="640">
    您的浏览器不支持视频
</video>

<!-- 多源支持 -->
<video controls>
    <source src="video.webm" type="video/webm">
    <source src="video.mp4" type="video/mp4">
</video>
```

**常用属性：**
- `controls` - 显示播放控件
- `autoplay` - 自动播放
- `loop` - 循环播放
- `poster` - 视频封面图

### 2.3 表单新特性

HTML5 增强了表单功能：

```html
<form>
    <input type="email" required>          <!-- 邮箱验证 -->
    <input type="url" placeholder="网址">   <!-- URL输入 -->
    <input type="tel" pattern="[0-9]{11}">  <!-- 手机号验证 -->
    <input type="search">                   <!-- 搜索框 -->
    <input type="range" min="0" max="100"> <!-- 滑块 -->
    <input type="color">                   <!-- 颜色选择器 -->
    <input type="date">                    <!-- 日期选择器 -->
    <input type="file" accept="image/*">   <!-- 文件上传 -->
</form>
```

**新增属性：**
- `required` - 必填字段
- `placeholder` - 占位提示
- `pattern` - 正则验证
- `autocomplete` - 自动完成

### 2.4 本地存储

HTML5 提供两种本地存储方式：

```javascript
// localStorage - 持久存储，无过期时间
localStorage.setItem('username', 'admin');
let user = localStorage.getItem('username');
localStorage.removeItem('username');

// sessionStorage - 会话存储，关闭浏览器清除
sessionStorage.setItem('token', 'abc123');
sessionStorage.clear();

// 存储复杂数据需序列化
let data = { name: 'Tom', age: 25 };
localStorage.setItem('user', JSON.stringify(data));
let userData = JSON.parse(localStorage.getItem('user'));
```

**存储限制：**
- 每个域名约 5MB
- 仅存储字符串类型
- 不同域名之间隔离

### 2.5 Canvas 画布

Canvas 用于绘制图形、动画和游戏：

```html
<canvas id="myCanvas" width="300" height="200"></canvas>

<script>
let canvas = document.getElementById('myCanvas');
let ctx = canvas.getContext('2d');

// 绘制矩形
ctx.fillStyle = 'red';
ctx.fillRect(10, 10, 50, 50);

// 绘制圆形
ctx.beginPath();
ctx.arc(100, 100, 30, 0, Math.PI * 2);
ctx.fillStyle = 'blue';
ctx.fill();

// 绘制文字
ctx.font = '20px Arial';
ctx.fillText('Hello Canvas', 50, 180);
</script>
```

**常用方法：**
- `fillRect()` - 填充矩形
- `strokeRect()` - 描边矩形
- `arc()` - 绘制圆弧
- `lineTo()` - 绘制直线
- `fillText()` - 绘制文字

---

## 3. CSS 核心

### 3.1 CSS 选择器

CSS（Cascading Style Sheets）用于描述页面样式。选择器决定样式作用于哪些元素：

```css
/* 元素选择器 */
p { color: red; }

/* 类选择器 */
.text-primary { color: blue; }

/* ID选择器 */
#header { background: #eee; }

/* 后代选择器 */
nav a { text-decoration: none; }

/* 子元素选择器 */
ul > li { list-style: none; }

/* 属性选择器 */
input[type="text"] { border: 1px solid #ccc; }

/* 伪类选择器 */
a:hover { color: green; }
input:focus { outline: none; }

/* 伪元素选择器 */
p::first-line { font-weight: bold; }
```

**选择器优先级：** ID > 类 > 元素

### 3.2 盒模型

每个元素都是一个矩形盒子，包含内容、内边距、边框和外边距：

```css
.box {
    width: 200px;       /* 内容宽度 */
    padding: 10px;      /* 内边距 */
    border: 2px solid #000;  /* 边框 */
    margin: 15px;       /* 外边距 */
    box-sizing: border-box;  /* 宽度包含padding和border */
}
```

**盒模型组成：**
- `content` - 内容区域
- `padding` - 内边距（内容与边框之间）
- `border` - 边框
- `margin` - 外边距（边框外部空间）

**两种盒模型：**
- `content-box` - 默认，width仅指内容宽度
- `border-box` - width包含padding和border

### 3.3 布局方式

#### 3.3.1 浮动布局

```css
.left { float: left; width: 50%; }
.right { float: right; width: 50%; }
.clearfix::after {
    content: '';
    display: block;
    clear: both;
}
```

#### 3.3.2 Flexbox 布局

```css
.container {
    display: flex;              /* 启用flex布局 */
    justify-content: center;    /* 水平居中 */
    align-items: center;        /* 垂直居中 */
    gap: 10px;                  /* 元素间距 */
}
.item { flex: 1; }              /* 均分空间 */
```

**常用属性：**
- `flex-direction` - 排列方向（row/column）
- `flex-wrap` - 是否换行
- `justify-content` - 主轴对齐
- `align-items` - 交叉轴对齐

#### 3.3.3 Grid 布局

```css
.grid-container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);  /* 3列等宽 */
    gap: 10px;
}
.item { grid-column: span 2; }  /* 跨2列 */
```

### 3.4 定位机制

```css
.static { position: static; }    /* 默认，正常流 */
.relative { 
    position: relative; 
    top: 10px;                   /* 相对自身偏移 */
}
.absolute { 
    position: absolute; 
    top: 0; right: 0;           /* 相对于最近定位祖先 */
}
.fixed { 
    position: fixed; 
    top: 0; left: 0;            /* 相对于视口 */
}
.sticky { 
    position: sticky; 
    top: 0;                     /* 滚动时固定 */
}
```

**定位层级：**
- `z-index` 控制重叠元素的层级
- 只有定位元素（非static）才有z-index

### 3.5 响应式设计

通过媒体查询适配不同设备：

```css
/* 默认样式 */
.container { width: 1200px; }

/* 平板 */
@media (max-width: 768px) {
    .container { width: 100%; }
}

/* 手机 */
@media (max-width: 480px) {
    .container { padding: 10px; }
}
```

**响应式策略：**
- 移动优先：先写移动端样式，再向上扩展
- 弹性布局：使用百分比、rem单位
- 图片适配：`max-width: 100%`
- 断点设置：根据主流设备宽度划分

### 3.6 动画与过渡

CSS3 提供强大的动画能力，无需JavaScript：

#### 3.6.1 过渡（Transition）

```css
.box {
    width: 100px;
    transition: width 0.3s ease;  /* 属性 时长 缓动 */
}
.box:hover {
    width: 200px;
}
```

**过渡属性：**
- `transition-property` - 指定过渡属性
- `transition-duration` - 过渡时长
- `transition-timing-function` - 缓动函数
- `transition-delay` - 延迟开始

#### 3.6.2 动画（Animation）

```css
@keyframes fadeIn {
    0% { opacity: 0; transform: translateY(-20px); }
    100% { opacity: 1; transform: translateY(0); }
}

.animate {
    animation: fadeIn 0.5s ease-out;
}

/* 无限循环动画 */
@keyframes spin {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
}
.spinner {
    animation: spin 1s linear infinite;
}
```

**动画属性：**
- `animation-name` - 动画名称
- `animation-duration` - 动画时长
- `animation-timing-function` - 缓动函数
- `animation-iteration-count` - 播放次数（infinite无限）
- `animation-direction` - 播放方向（normal/reverse/alternate）

#### 3.6.3 变换（Transform）

```css
.box {
    transform: translate(50px, 20px);    /* 平移 */
    transform: rotate(45deg);            /* 旋转 */
    transform: scale(1.5);               /* 缩放 */
    transform: skew(10deg, 5deg);        /* 倾斜 */
    transform-origin: center center;     /* 变换原点 */
}
```

---

## 4. JavaScript 核心

### 4.1 变量与数据类型

JavaScript 是弱类型语言，变量类型可动态变化：

```javascript
// 变量声明
let name = 'Tom';      // 块级作用域
const PI = 3.14;       // 常量，不可修改
var age = 25;          // 函数作用域（ES5）

// 基本数据类型
let str = 'hello';     // 字符串
let num = 100;         // 数字（整数/小数）
let bool = true;       // 布尔值
let undef = undefined; // 未定义
let nul = null;        // 空值
let sym = Symbol('id'); // 唯一标识

// 复杂数据类型
let obj = { name: 'Tom' };  // 对象
let arr = [1, 2, 3];        // 数组
let fn = function() {};     // 函数
```

**类型判断：**
```javascript
typeof 'hello'  // "string"
typeof 123      // "number"
typeof true     // "boolean"
typeof undefined // "undefined"
typeof null     // "object"（历史遗留bug）
typeof {}       // "object"
Array.isArray([]) // true
```

### 4.2 运算符与表达式

```javascript
// 算术运算符
let sum = 1 + 2;    // 加法
let diff = 5 - 3;   // 减法
let prod = 4 * 5;   // 乘法
let quot = 10 / 2;  // 除法
let mod = 7 % 3;    // 取余

// 赋值运算符
let a = 10;
a += 5;  // 等价于 a = a + 5

// 比较运算符
10 > 5   // true
10 >= 10 // true
10 == '10' // true（类型转换）
10 === '10' // false（严格相等）

// 逻辑运算符
true && false // false（与）
true || false // true（或）
!true        // false（非）

// 三元运算符
let result = score >= 60 ? '及格' : '不及格';
```

### 4.3 流程控制

#### 4.3.1 条件语句

```javascript
// if-else
if (age >= 18) {
    console.log('成年');
} else {
    console.log('未成年');
}

// switch
switch (day) {
    case 1: console.log('周一'); break;
    case 2: console.log('周二'); break;
    default: console.log('其他');
}
```

#### 4.3.2 循环语句

```javascript
// for 循环
for (let i = 0; i < 5; i++) {
    console.log(i);
}

// for...of（遍历数组）
for (let item of [1, 2, 3]) {
    console.log(item);
}

// for...in（遍历对象属性）
for (let key in {a:1, b:2}) {
    console.log(key);
}

// while 循环
let i = 0;
while (i < 5) {
    console.log(i);
    i++;
}

// do...while（至少执行一次）
do {
    console.log(i);
    i++;
} while (i < 5);
```

### 4.4 函数

```javascript
// 函数声明
function add(a, b) {
    return a + b;
}

// 函数表达式
let multiply = function(a, b) {
    return a * b;
};

// 箭头函数（ES6）
let divide = (a, b) => a / b;

// 立即执行函数（IIFE）
(function() {
    console.log('立即执行');
})();

// 带参数默认值
function greet(name = 'Guest') {
    console.log(`Hello, ${name}`);
}

// 剩余参数
function sumAll(...nums) {
    return nums.reduce((a, b) => a + b);
}
```

**函数作用域：**
- 内部可访问外部变量
- 外部不可访问内部变量
- `let` 和 `const` 是块级作用域
- `var` 是函数作用域

### 4.5 对象与数组

#### 4.5.1 对象

```javascript
// 对象字面量
let person = {
    name: 'Tom',
    age: 25,
    greet: function() {
        console.log(`Hello, ${this.name}`);
    }
};

// 访问属性
person.name;       // 点语法
person['age'];     // 方括号语法

// 添加/修改属性
person.gender = 'male';
person.age = 26;

// 删除属性
delete person.gender;
```

#### 4.5.2 数组

```javascript
// 创建数组
let arr = [1, 2, 3, 4, 5];

// 常用方法
arr.push(6);       // 末尾添加
arr.pop();         // 末尾删除
arr.shift();       // 头部删除
arr.unshift(0);    // 头部添加
arr.slice(1, 3);   // 截取[1,3)
arr.splice(2, 1);  // 从索引2删除1个
arr.join('-');     // 转为字符串

// 迭代方法（ES5+）
arr.forEach(item => console.log(item));
arr.map(item => item * 2);      // 映射
arr.filter(item => item > 2);   // 过滤
arr.reduce((sum, item) => sum + item, 0); // 累加
```

### 4.6 DOM 操作

DOM（Document Object Model）是页面的编程接口：

```javascript
// 获取元素
document.getElementById('id');       // 按ID获取
document.getElementsByClassName('class'); // 按类名
document.getElementsByTagName('tag');    // 按标签名
document.querySelector('.class');       // CSS选择器
document.querySelectorAll('p');         // 选择所有匹配

// 修改元素内容
let el = document.getElementById('myDiv');
el.textContent = '新内容';           // 纯文本
el.innerHTML = '<strong>加粗</strong>'; // HTML

// 修改样式
el.style.color = 'red';
el.style.fontSize = '20px';

// 添加/移除类
el.classList.add('active');
el.classList.remove('active');
el.classList.toggle('active');

// 创建元素
let newEl = document.createElement('p');
newEl.textContent = '新段落';
document.body.appendChild(newEl);

// 删除元素
el.remove();
```

### 4.7 事件处理

事件处理使页面具有交互性：

```javascript
// 方式1：HTML属性（不推荐）
// <button onclick="handleClick()">点击</button>

// 方式2：DOM属性
let btn = document.getElementById('btn');
btn.onclick = function() {
    console.log('点击了');
};

// 方式3：addEventListener（推荐）
btn.addEventListener('click', function(e) {
    console.log(e.target);       // 事件目标
    console.log(e.type);         // 事件类型
    e.preventDefault();          // 阻止默认行为
});

// 事件委托
document.addEventListener('click', function(e) {
    if (e.target.matches('.btn')) {
        console.log('按钮被点击');
    }
});

// 常用事件
// click, mouseover, mouseout, keydown, keyup, submit, change
```

### 4.8 ES6+ 新特性

#### 4.8.1 解构赋值

```javascript
// 数组解构
let [a, b, c] = [1, 2, 3];

// 对象解构
let { name, age } = { name: 'Tom', age: 25 };

// 默认值
let { gender = 'male' } = { name: 'Tom' };
```

#### 4.8.2 模板字符串

```javascript
let name = 'Tom';
let str = `Hello, ${name}!`;     // 插值
let multi = `这是
多行字符串`;
```

#### 4.8.3 类

```javascript
class Person {
    constructor(name) {
        this.name = name;
    }
    greet() {
        console.log(`Hello, ${this.name}`);
    }
}

let p = new Person('Tom');
p.greet();
```

#### 4.8.4 模块

```javascript
// 导出
// module.js
export const PI = 3.14;
export function add(a, b) { return a + b; }

// 导入
// main.js
import { PI, add } from './module.js';
```

#### 4.8.5 Promise

```javascript
function fetchData() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve('成功');
            // reject('失败');
        }, 1000);
    });
}

fetchData()
    .then(data => console.log(data))
    .catch(err => console.error(err));
```

#### 4.8.6 async/await

```javascript
async function getData() {
    try {
        let data = await fetchData();
        console.log(data);
    } catch (err) {
        console.error(err);
    }
}

getData();
```

#### 4.8.7 箭头函数

```javascript
// 基本语法
let fn = (a, b) => a + b;

// 无参数
let fn2 = () => console.log('hello');

// 单行对象
let fn3 = () => ({ name: 'Tom' });

// 注意：箭头函数没有this绑定
```

#### 4.8.8 展开运算符

```javascript
// 数组展开
let arr1 = [1, 2];
let arr2 = [...arr1, 3, 4];  // [1,2,3,4]

// 对象展开
let obj1 = { a: 1 };
let obj2 = { ...obj1, b: 2 }; // {a:1, b:2}

// 函数参数展开
function sum(...nums) {
    return nums.reduce((a, b) => a + b);
}
sum(1, 2, 3);  // 6
```

### 4.9 闭包与作用域

闭包是JavaScript最重要的概念之一，指函数能够访问其词法作用域之外的变量：

```javascript
function outer() {
    let count = 0;           // 外部变量
    return function inner() {
        count++;             // 访问外部变量
        return count;
    };
}

let counter = outer();
counter();  // 1
counter();  // 2
counter();  // 3
```

**闭包应用场景：**

```javascript
// 数据封装（私有变量）
function createPerson(name) {
    let age = 0;
    return {
        getName: () => name,
        getAge: () => age,
        setAge: (newAge) => { age = newAge; }
    };
}

let p = createPerson('Tom');
p.setAge(25);
p.getAge();  // 25

// 模块化
let module = (function() {
    let privateVar = 'secret';
    return {
        publicMethod: () => privateVar
    };
})();
```

**作用域链：**
- 内层函数可访问外层函数的变量
- 查找变量时，从内向外查找
- 每个函数创建时形成自己的作用域

### 4.10 原型链与继承

JavaScript 使用原型链实现继承：

```javascript
// 原型对象
let animal = {
    eat: function() { console.log('eating'); }
};

// 创建对象并设置原型
let dog = Object.create(animal);
dog.bark = function() { console.log('woof'); };

dog.bark();  // woof（自身方法）
dog.eat();   // eating（继承方法）

// 原型链查找：dog → animal → Object → null
```

**构造函数与原型：**

```javascript
function Person(name) {
    this.name = name;
}

Person.prototype.greet = function() {
    console.log(`Hello, ${this.name}`);
};

let p1 = new Person('Tom');
let p2 = new Person('Jerry');

p1.greet();  // Hello, Tom
p2.greet();  // Hello, Jerry

// 实例共享原型方法
p1.greet === p2.greet;  // true
```

**ES6 Class 继承：**

```javascript
class Animal {
    constructor(name) {
        this.name = name;
    }
    eat() { console.log('eating'); }
}

class Dog extends Animal {
    constructor(name, breed) {
        super(name);       // 调用父类构造
        this.breed = breed;
    }
    bark() { console.log('woof'); }
}

let dog = new Dog('Buddy', 'Golden');
dog.eat();   // eating（继承）
dog.bark();  // woof（自身）
```

**原型链规则：**
- `obj.__proto__` 指向构造函数的原型
- `obj.constructor.prototype` 等价于 `obj.__proto__`
- `Object.prototype.__proto__` 为 `null`（链的终点）

### 4.11 事件循环

JavaScript 是单线程语言，通过事件循环处理异步操作：

```javascript
// 执行顺序说明
console.log('1');  // 同步：立即执行

setTimeout(() => {
    console.log('2');  // 宏任务：下一轮执行
}, 0);

Promise.resolve().then(() => {
    console.log('3');  // 微任务：本轮末尾执行
});

console.log('4');  // 同步：立即执行

// 输出顺序：1 → 4 → 3 → 2
```

**任务队列分类：**

```javascript
// 宏任务（Macro Task）
setTimeout(() => {}, 0);
setInterval(() => {}, 0);
requestAnimationFrame(() => {});
I/O 操作
UI 渲染

// 微任务（Micro Task）
Promise.then()
Promise.catch()
Promise.finally()
MutationObserver
queueMicrotask(() => {})
```

**事件循环流程：**

```
1. 执行同步代码（调用栈）
2. 清空微任务队列（按顺序执行所有微任务）
3. 执行一个宏任务
4. 重复步骤2-3
```

**实战示例：**

```javascript
console.log('start');

setTimeout(() => {
    console.log('timeout1');
    Promise.resolve().then(() => {
        console.log('promise in timeout1');
    });
}, 0);

Promise.resolve().then(() => {
    console.log('promise1');
    setTimeout(() => {
        console.log('timeout2');
    }, 0);
});

console.log('end');

// 输出：start → end → promise1 → timeout1 → promise in timeout1 → timeout2
```