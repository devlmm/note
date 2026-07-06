# Vue 3 知识点完整目录（由浅入深，环环相扣）

  **阅读指南**：由浅入深，每个知识点均配有精简示例。建议按顺序阅读，环环相扣。

---

# 📑 目录

- [**第一章：环境与入门**](#第一章环境与入门)
  - [1.1 创建项目](#11-创建项目)
  - [1.2 项目结构](#12-项目结构)
  - [1.3 Hello World](#13-hello-world)
  - [1.4 两个核心概念](#14-两个核心概念)
- [**第二章：核心概念与模板语法**](#第二章核心概念与模板语法)
  - [2.1 模板语法基础](#21-模板语法基础)
  - [2.2 指令系统](#22-指令系统)
- [**第三章：组件基础**](#第三章组件基础)
  - [3.1 组件注册与使用](#31-组件注册与使用)
  - [3.2 单文件组件（SFC）](#32-单文件组件sfc)
  - [3.3 Props（父→子数据传递）](#33-props父子数据传递)
  - [3.4 组件树示例](#34-组件树示例)
- [**第四章：事件与表单绑定**](#第四章事件与表单绑定)
  - [4.1 事件处理（v-on / @）](#41-事件处理v-on--)
  - [4.2 表单绑定（v-model）](#42-表单绑定v-model)
- [**第五章：生命周期与组件通信**](#第五章生命周期与组件通信)
  - [5.1 生命周期钩子](#51-生命周期钩子)
  - [5.2 组件通信](#52-组件通信)
- [**第六章：组合式 API**](#第六章组合式-api)
  - [6.1 setup() 入口](#61-setup-入口)
  - [6.2 响应式系统](#62-响应式系统)
  - [6.3 组合函数（Composables）](#63-组合函数composables)
  - [6.4 script setup 语法糖](#64-script-setup-语法糖)
  - [6.5 生命周期映射](#65-生命周期映射)
- [**第七章：状态管理与路由**](#第七章状态管理与路由)
  - [7.1 Pinia（官方推荐状态管理）](#71-pinia官方推荐状态管理)
  - [7.2 Vue Router](#72-vue-router)
- [**第八章：常用工具与工程化**](#第八章常用工具与工程化)
  - [8.1 Axios 网络请求](#81-axios-网络请求)
  - [8.2 Element Plus（UI 组件库）](#82-element-plus-ui-组件库)
  - [8.3 VueUse（组合式工具集）](#83-vueuse组合式工具集)
  - [8.4 常用工具库](#84-常用工具库)
- [**第九章：性能优化与调试**](#第九章性能优化与调试)
  - [9.1 响应式优化](#91-响应式优化)
  - [9.2 组件优化](#92-组件优化)
  - [9.3 组件销毁清理](#93-组件销毁清理)
  - [9.4 调试技巧](#94-调试技巧)
- [**附录：关键知识点归纳**](#附录关键知识点归纳)
  - [A.1 响应式原理](#a1-响应式原理)
  - [A.2 computed vs watch](#a2-computed-vs-watch)
  - [A.3 TypeScript 支持](#a3-typescript-支持)
  - [A.4 自定义指令](#a4-自定义指令)
  - [A.5 项目结构](#a5-项目结构)
  - [A.6 命名规范](#a6-命名规范)
  - [A.7 Vue 3 新特性速查](#a7-vue-3-新特性速查)
  - [A.8 Vue 3 破坏性变更](#a8-vue-3-破坏性变更)

---

# 第一章：环境与入门

## 1.1 创建项目

```bash
# Vue CLI（Vue 2/3）
npm create vue@latest my-app  # 推荐，交互式选择

# Vite（推荐，更快）
npm create vite@latest my-app -- --template vue
```

```bash
cd my-app
npm install
npm run dev  # 启动开发服务器
```

## 1.2 项目结构

```
src/
├── assets/          # 静态资源
├── components/      # 组件（PascalCase）
├── App.vue          # 根组件
└── main.js          # 入口文件
```

## 1.3 Hello World

```javascript
// main.js
import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')  // 挂载到 #app
```

```html
<!-- index.html -->
<div id="app"></div>
```

```html
<!-- App.vue -->
<template>
  <h1>Hello, {{ name }}!</h1>
</template>

<script setup>
const name = 'Vue 3'
</script>
```

## 1.4 两个核心概念

| 概念 | 说明 |
|------|------|
| **声明式** | 描述"是什么"，而非"怎么做" |
| **响应式** | 数据变化自动更新视图 |

---

# 第二章：核心概念与模板语法

## 2.1 模板语法基础

### 2.1.1 文本插值

```html
<span>{{ message }}</span>           <!-- 普通文本 -->
<span>{{ count + 1 }}</span>         <!-- 表达式 -->
<span>{{ ok ? '是' : '否' }}</span>   <!-- 三元表达式 -->
```

```javascript
export default {
  data() {
    return { message: '你好', count: 0, ok: true }
  }
}
```

### 2.1.2 原始 HTML（v-html）

```html
<p v-html="rawHtml"></p>  <!-- ⚠️ XSS风险，禁止用于用户输入 -->
```

### 2.1.3 属性绑定（v-bind）

```html
<img :src="imageUrl" :alt="desc" />  <!-- 简写形式 -->
<div v-bind:id="dynamicId"></div>     <!-- 完整形式 -->
```

### 2.1.4 动态参数

```html
<a :[attrName]="value">动态属性</a>
<button @[eventName]="handler">动态事件</button>
```

## 2.2 指令系统

### 2.2.1 常用指令速查

| 指令 | 用途 | 缩写 |
|------|------|------|
| `v-bind` | 绑定属性 | `:` |
| `v-on` | 绑定事件 | `@` |
| `v-model` | 双向绑定 | — |
| `v-for` | 列表渲染 | — |
| `v-if` | 条件渲染 | — |
| `v-show` | CSS显示隐藏 | — |

### 2.2.2 条件渲染（v-if）

```html
<p v-if="show">显示</p>
<p v-else-if="other">其他</p>
<p v-else>隐藏</p>
```

```html
<!-- 批量条件：template不渲染 -->
<template v-if="type === 'A'">
  <h1>标题</h1>
  <p>内容</p>
</template>
```

### 2.2.3 v-if vs v-show

| 特性 | v-if | v-show |
|------|------|--------|
| 渲染 | 不渲染（条件为false） | 始终渲染，CSS切换 |
| 切换开销 | 高（销毁/重建） | 低（仅display） |
| 初始开销 | 低（条件false不渲染） | 高（始终渲染） |
| 场景 | 条件稳定 | 频繁切换 |

📐 **决策**：频繁切换用 `v-show`，条件稳定用 `v-if`

### 2.2.4 列表渲染（v-for）

```html
<!-- 基础 -->
<li v-for="item in items" :key="item.id">
  {{ item.name }}
</li>

<!-- 带索引 -->
<li v-for="(item, index) in items" :key="index">

<!-- 对象遍历 -->
<div v-for="(value, key, index) in obj" :key="key">

<!-- 数字范围 -->
<span v-for="n in 10" :key="n">{{ n }}</span>
```

```javascript
export default {
  data() {
    return {
      items: [
        { id: 1, name: '苹果' },
        { id: 2, name: '香蕉' }
      ]
    }
  }
}
```

### 2.2.5 Key 的重要性

```html
<!-- ✅ 最佳：唯一稳定ID -->
<li v-for="item in items" :key="item.id">

<!-- ⚠️ 次选：仅列表静态时 -->
<li v-for="(item, index) in items" :key="index">

<!-- ❌ 禁止：随机值 -->
<li v-for="item in items" :key="Math.random()">
```

📐 **原则**：key 必须唯一且稳定，默认用 `item.id`

---

# 第三章：组件基础

## 3.1 组件注册与使用

```javascript
// 引入
import MyComponent from './components/MyComponent.vue'

export default {
  components: { MyComponent }  // 局部注册
}
```

```html
<!-- 使用：PascalCase 或 kebab-case -->
<MyComponent />
<my-component />
```

## 3.2 单文件组件（SFC）

```vue
<template>
  <div class="container">
    <h3>{{ title }}</h3>
  </div>
</template>

<script>
export default {
  name: 'MyComponent',
  props: ['title']  // 接收父组件数据
}
</script>

<style scoped>
.container { color: red; }
</style>
```

## 3.3 Props（父→子数据传递）

### 3.3.1 基本用法

```html
<!-- 父组件 -->
<ChildComponent title="标题" :count="10" />
```

```javascript
// 子组件声明
export default {
  props: ['title', 'count']  // 简单声明
}
```

### 3.3.2 带验证的声明

```javascript
export default {
  props: {
    title: String,                    // 类型
    count: { type: Number, default: 0 },  // 默认值
    user: { type: Object, required: true } // 必填
  }
}
```

### 3.3.3 Props 单向数据流

```javascript
// ❌ 禁止：直接修改 props
this.title = 'new'

// ✅ 正确：通知父组件
this.$emit('update:title', 'new')

// ✅ 正确：本地副本
data() { return { localTitle: this.title } }
```

📐 **原则**：props 只能从父组件流向子组件，不可反方向修改

## 3.4 组件树示例

```
App（根组件）
├── Header
│   └── Nav
├── Sidebar
│   ├── MenuItem
│   └── SubMenu
└── Footer
```

---

# 第四章：事件与表单绑定

## 4.1 事件处理（v-on / @）

### 4.1.1 基础用法

```html
<!-- 内联 -->
<button @click="count++">+1</button>

<!-- 方法 -->
<button @click="greet">打招呼</button>

<!-- 传参 + 事件对象 -->
<button @click="say('hi', $event)">Say</button>
```

```javascript
export default {
  methods: {
    greet(event) {
      console.log('Hello', event.target)
    },
    say(msg, event) {
      console.log(msg)
    }
  }
}
```

### 4.1.2 事件修饰符

```html
@click.stop="handler"      <!-- 阻止冒泡 -->
@click.prevent="handler"   <!-- 阻止默认行为 -->
@click.once="handler"      <!-- 只触发一次 -->
@click.self="handler"      <!-- 仅自身触发 -->
```

### 4.1.3 按键修饰符

```html
<input @keyup.enter="submit" />
<input @keyup.ctrl.enter="submit" />
<button @click.meta="save">Meta</button>
```

| 修饰符 | 键名 |
|--------|------|
| `.enter` | 回车 |
| `.tab` | Tab |
| `.esc` | 退出 |
| `.space` | 空格 |
| `.up/down/left/right` | 方向键 |

## 4.2 表单绑定（v-model）

### 4.2.1 基础用法

```html
<!-- 文本输入 -->
<input v-model="text" />

<!-- 多行文本 -->
<textarea v-model="text"></textarea>
```

```javascript
data() { return { text: '' } }
```

### 4.2.2 不同表单元素

```html
<!-- 复选框（单个） -->
<input type="checkbox" v-model="checked" />

<!-- 复选框（多个） -->
<input type="checkbox" value="A" v-model="checked" />
<input type="checkbox" value="B" v-model="checked" />

<!-- 单选 -->
<input type="radio" value="A" v-model="picked" />

<!-- 下拉框 -->
<select v-model="selected">
  <option value="">请选择</option>
  <option value="A">选项A</option>
</select>
```

### 4.2.3 v-model 本质

```html
<!-- 等价于 -->
<input v-model="text" />
<!-- ↓ -->
<input :value="text" @input="text = $event.target.value" />
```

### 4.2.4 修饰符

```html
<input v-model.lazy="text" />    <!-- change后同步 -->
<input v-model.number="num" />   <!-- 自动转数字 -->
<input v-model.trim="name" />   <!-- 去除首尾空格 -->
```

---

# 第五章：生命周期与组件通信

## 5.1 生命周期钩子

### 5.1.1 四个阶段

```
创建 → 挂载 → 更新 → 卸载
```

| 阶段 | Options API | Composition API |
|------|-------------|-----------------|
| 创建 | `beforeCreate`, `created` | `setup()` |
| 挂载 | `beforeMount`, `mounted` | `onBeforeMount`, `onMounted` |
| 更新 | `beforeUpdate`, `updated` | `onBeforeUpdate`, `onUpdated` |
| 卸载 | `beforeUnmount`, `unmounted` | `onBeforeUnmount`, `onUnmounted` |

### 5.1.2 最佳实践

```javascript
export default {
  created() {
    // ✅ 适合：数据初始化、API调用
  },
  mounted() {
    // ✅ 适合：DOM操作、第三方库初始化
  },
  beforeUnmount() {
    // ✅ 适合：清理定时器、移除事件监听
  }
}
```

📐 **记忆口诀**：创建→挂载→更新→卸载（8个核心钩子）

## 5.2 组件通信

### 5.2.1 父子通信（Props + Emit）

```html
<!-- 父组件 -->
<Child :title="title" @update="handleUpdate" />
```

```javascript
// 子组件
export default {
  props: ['title'],
  emits: ['update'],  // 声明自定义事件
  methods: {
    send() {
      this.$emit('update', 'new value')  // 触发事件
    }
  }
}
```

### 5.2.2 $attrs（隔代传递）

```javascript
// 祖代组件
<Parent :title="title" :count="count" class="custom" />

// 子组件获取所有 attrs（排除 props 声明的）
console.log(this.$attrs)  // { class: 'custom' }
```

### 5.2.3 Provide / Inject（跨层级）

```javascript
// 祖先组件
import { provide, ref } from 'vue'
export default {
  setup() {
    const theme = ref('dark')
    provide('theme', theme)  // 提供
  }
}
```

```javascript
// 后代组件
import { inject } from 'vue'
export default {
  setup() {
    const theme = inject('theme', 'light')  // 注入（带默认值）
  }
}
```

### 5.2.4 组件通信对比

| 方式 | 适用场景 |
|------|----------|
| props/$emit | 父子 |
| $attrs | 隔代 |
| provide/inject | 祖孙（插件/配置） |
| Pinia | 全局状态 |
| eventBus | 任意组件 → 用 `mitt` 替代 |

---

# 第六章：组合式 API

## 6.1 setup() 入口

```javascript
export default {
  setup(props, context) {
    // props: 响应式 props
    // context: { attrs, slots, emit, expose, parent, root }
    const count = ref(0)
    return { count }  // 必须返回才能在模板中使用
  }
}
```

## 6.2 响应式系统

### 6.2.1 ref vs reactive

| API | 适用类型 | 访问方式 |
|-----|----------|----------|
| `ref()` | 任意类型 | `.value` |
| `reactive()` | 对象/数组 | 直接访问 |

```javascript
import { ref, reactive, toRefs } from 'vue'

const count = ref(0)           // 基础类型 → ref
const state = reactive({       // 对象 → reactive
  name: '张三',
  age: 18
})

count.value++                  // 修改 ref
state.name = '李四'            // 修改 reactive

// 解构时保持响应式
const { name, age } = toRefs(state)
```

📐 **选择**：基础类型用 `ref`，对象用 `reactive`

### 6.2.2 computed（计算属性）

```javascript
import { computed } from 'vue'

const firstName = ref('张')
const lastName = ref('三')

const fullName = computed(() => {
  return firstName.value + lastName.value
})
```

### 6.2.3 watch vs watchEffect

```javascript
import { watch, watchEffect } from 'vue'

// watchEffect：立即执行，自动收集依赖
watchEffect(() => {
  console.log(msg.value)  // msg 变化时触发
})

// watch：惰性执行，精确指定依赖
watch(msg, (newVal, oldVal) => {
  console.log('变化了')
}, { immediate: true })  // immediate: 立即执行
```

## 6.3 组合函数（Composables）

### 6.3.1 逻辑复用

```javascript
// 鼠标位置 composable
function useMouse() {
  const x = ref(0)
  const y = ref(0)
  
  const update = (e) => { x.value = e.pageX; y.value = e.pageY }
  
  onMounted(() => window.addEventListener('mousemove', update))
  onUnmounted(() => window.removeEventListener('mousemove', update))
  
  return { x, y }
}
```

```javascript
// 组件中使用
const { x, y } = useMouse()
```

📐 **优势**：无命名冲突、来源清晰、支持 Tree-shaking

### 6.3.2 计数器示例

```javascript
function useCounter(initial = 0) {
  const count = ref(initial)
  const increment = () => count.value++
  const decrement = () => count.value--
  return { count, increment, decrement }
}
```

## 6.4 script setup 语法糖

```vue
<script setup>
// 顶层变量自动暴露给模板
const msg = 'Hello'
const count = ref(0)

const handleClick = () => count.value++
</script>

<template>
  <p>{{ msg }} - {{ count }}</p>
  <button @click="handleClick">+1</button>
</template>
```

## 6.5 生命周期映射

| Options API | Composition API |
|-------------|-----------------|
| `beforeCreate` | `setup()` |
| `created` | `setup()` |
| `beforeMount` | `onBeforeMount` |
| `mounted` | `onMounted` |
| `beforeUpdate` | `onBeforeUpdate` |
| `updated` | `onUpdated` |
| `beforeUnmount` | `onBeforeUnmount` |
| `unmounted` | `onUnmounted` |

---

# 第七章：状态管理与路由

## 7.1 Pinia（官方推荐状态管理）

### 7.1.1 创建 Store

```javascript
// stores/counter.js
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  state: () => ({ count: 0, name: 'iwen' }),
  getters: {
    double: (state) => state.count * 2
  },
  actions: {
    increment() { this.count++ }
  }
})
```

### 7.1.2 组件中使用

```javascript
import { useCounterStore } from '@/stores/counter'
import { storeToRefs } from 'pinia'

export default {
  setup() {
    const store = useCounterStore()
    
    store.count          // 读取
    store.increment()     // 调用 action
    storeToRefs(store).count  // 保持响应式解构
    
    return { store }
  }
}
```

### 7.1.3 组合式风格

```javascript
export const useCounterStore = defineStore('counter', () => {
  const count = ref(0)
  const double = computed(() => count.value * 2)
  
  function increment() { count.value++ }
  
  return { count, double, increment }
})
```

### 7.1.4 Pinia vs Vuex

| 特性 | Pinia | Vuex 4 |
|------|-------|--------|
| TypeScript | ✅ 原生 | ⚠️ 需适配 |
| 模块 | 自动 | 手动 |
| mutations | ❌ 无 | ✅ 有 |
| 体积 | ~1KB | ~10KB |
| 官方推荐 | ✅ 是 | — |

📐 **建议**：新项目用 Pinia，现有 Vuex 项目可继续使用

## 7.2 Vue Router

### 7.2.1 基础配置

```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'

const routes = [
  { path: '/', name: 'home', component: () => import('../views/Home.vue') },
  { path: '/about', name: 'about', component: () => import('../views/About.vue') }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

```javascript
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

createApp(App).use(router).mount('#app')
```

### 7.2.2 路由使用

```html
<!-- 声明式导航 -->
<router-link to="/">首页</router-link>
<router-link :to="{ path: '/about', query: { id: 1 } }">关于</router-link>

<!-- 编程式导航 -->
this.$router.push('/')
this.$router.push({ name: 'about', params: { id: 1 } })

<!-- 路由出口 -->
<router-view />
```

### 7.2.3 路由传参

```javascript
// 路径参数
{ path: '/user/:id', component: User }

// 接收
this.$route.params.id
this.$route.query.id  // query 参数
```

### 7.2.4 路由守卫

```javascript
// 全局前置守卫
router.beforeEach((to, from) => {
  // to: 目标路由, from: 来源
  return true   // true → 通过
  return false  // 拒绝
})

// 组件内守卫
onMounted(() => {
  const unsubscribe = router.beforeEach((to, from) => {
    return true
  })
  onUnmounted(() => unsubscribe())
})
```

---

# 第八章：常用工具与工程化

## 8.1 Axios 网络请求

### 8.1.1 基础封装

```javascript
// utils/request.js
import axios from 'axios'
import qs from 'querystring'

const instance = axios.create({
  baseURL: '/api',
  timeout: 10000
})

// 请求拦截器
instance.interceptors.request.use(config => {
  if (config.method === 'post') {
    config.data = qs.stringify(config.data)
  }
  const token = localStorage.getItem('token')
  if (token) config.headers.Authorization = `Bearer ${token}`
  return config
})

// 响应拦截器
instance.interceptors.response.use(
  response => response.data,
  error => {
    if (error.response?.status === 401) {
      // 跳转登录
    }
    return Promise.reject(error)
  }
)

export default instance
```

### 8.1.2 API 封装

```javascript
// api/user.js
import request from '@/utils/request'

export const getUserInfo = (id) => request.get(`/user/${id}`)
export const login = (data) => request.post('/login', data)
```

### 8.1.3 跨域代理（Vite）

```javascript
// vite.config.js
export default {
  server: {
    proxy: {
      '/api': {
        target: 'http://api.example.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  }
}
```

## 8.2 Element Plus（UI 组件库）

```bash
npm install element-plus
```

```javascript
// main.js
import { createApp } from 'vue'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import App from './App.vue'

createApp(App).use(ElementPlus).mount('#app')
```

```html
<!-- 按需引入（推荐） -->
<template>
  <el-button type="primary">按钮</el-button>
  <el-input v-model="text" placeholder="输入" />
</template>
```

## 8.3 VueUse（组合式工具集）

```bash
npm install @vueuse/core
```

```javascript
import { useLocalStorage, useDebounceFn } from '@vueuse/core'

// 本地存储
const name = useLocalStorage('name', 'default')

// 防抖
const debouncedFn = useDebounceFn(() => {
  // 搜索建议等
}, 300)
```

| 常用 API | 用途 |
|----------|------|
| `useLocalStorage` | 持久化存储 |
| `useDebounceFn` | 防抖函数 |
| `useThrottleFn` | 节流函数 |
| `useMouse` | 鼠标位置 |
| `useEventListener` | 事件监听 |
| `useFetch` | 数据请求 |

## 8.4 常用工具库

| 库 | 用途 |
|----|------|
| `dayjs` | 时间处理（轻量） |
| `lodash-es` | 函数工具（按需引入） |
| `axios` | HTTP 请求 |
| `mitt` | 事件总线 |

---

# 第九章：性能优化与调试

## 9.1 响应式优化

### 9.1.1 shallowRef vs ref

```javascript
const state = ref({ count: 0 })        // 深度响应式
const shallow = shallowRef({ count: 0 })  // 浅层响应式

state.value.count++      // ✅ 触发更新
shallow.value.count++    // ❌ 不触发，需整体替换
shallow.value = { count: 1 }  // ✅ 触发更新
```

📐 **场景**：大对象不需要深层响应式时用 `shallowRef`

### 9.1.2 v-memo

```html
<!-- 只在 list 或 total 变化时更新 -->
<div v-memo="[list, total]">
  <ExpensiveComponent />
</div>
```

## 9.2 组件优化

### 9.2.1 路由懒加载

```javascript
// ✅ 推荐
{ path: '/about', component: () => import('../views/About.vue') }

// ❌ 不推荐
import About from '../views/About.vue'
```

### 9.2.2 keep-alive 缓存

```html
<keep-alive include="Home,About" :max="10">
  <component :is="currentView" />
</keep-alive>
<!-- 切换时不销毁，activated/deactivated 生效 -->
```

### 9.2.3 defineAsyncComponent

```javascript
import { defineAsyncComponent } from 'vue'

const AsyncComponent = defineAsyncComponent(() =>
  import('./components/Heavy.vue')
)
```

## 9.3 组件销毁清理

```javascript
// ⚠️ 必须清理，否则内存泄漏
onUnmounted(() => {
  clearInterval(timerId)                    // 定时器
  window.removeEventListener('resize', fn)  // 事件监听
  ws.close()                                // WebSocket
  cancelAnimationFrame(id)                  // 动画帧
})
```

## 9.4 调试技巧

### 9.4.1 DevTools

- **Components**：查看组件树、props、state
- **Timeline**：性能分析
- **Pinia**：状态管理调试

### 9.4.2 常见报错

| 错误 | 原因 | 解决 |
|------|------|------|
| `xxx is not reactive` | 直接赋值 | 用 `ref/reactive` |
| `avoid mutating prop` | 子组件改 props | emit 或本地副本 |
| `NavigationDuplicated` | 导航重复 | `catch(err => {})` |

---

# 附录：关键知识点归纳

## A.1 响应式原理

```
Proxy → get/set → 依赖收集 → 触发更新
```

📐 **注意**：`this.xxx = val` 无法触发响应式，必须用 `ref/reactive`

## A.2 computed vs watch

| 特性 | computed | watch |
|------|----------|-------|
| 缓存 | ✅ | ❌ |
| 返回值 | ✅ 必须 | ❌ 可选 |
| 适用 | 派生状态 | 副作用 |
| 异步 | ❌ | ✅ |

## A.3 TypeScript 支持

```typescript
// defineProps
defineProps<{ name: string; age?: number }>()

// withDefaults
withDefaults(defineProps<{ age: number }>(), { age: 18 })

// defineEmits
const emit = defineEmits<{
  (e: 'update', value: string): void
}>()

// defineExpose
defineExpose({ fetchData: () => {} })
```

## A.4 自定义指令

```javascript
// 全局注册
app.directive('focus', {
  mounted(el) { el.focus() }
})

// 局部注册
export default {
  directives: {
    focus: { mounted(el) { el.focus() } }
  }
}
```

## A.5 项目结构

```
src/
├── api/            # 接口请求
├── assets/         # 静态资源
├── components/     # 公共组件
├── composables/    # 组合函数
├── router/         # 路由配置
├── stores/         # 状态管理（Pinia）
├── utils/          # 工具函数
├── App.vue
└── main.js
```

## A.6 命名规范

| 类型 | 规范 | 示例 |
|------|------|------|
| 组件 | PascalCase | `UserCard.vue` |
| 组合函数 | use + 语义 | `useCounter.js` |
| 工具函数 | 语义/下划线 | `formatDate.js` |
| API 方法 | RESTful | `getUser` |
| 常量 | UPPER_SNAKE | `MAX_COUNT` |

📐 **口诀**：组件大驼峰，函数 use 开头，工具小写下划线

## A.7 Vue 3 新特性速查

| 特性 | 说明 |
|------|------|
| `Fragment` | 模板无需根节点 |
| `Teleport` | 传送到指定 DOM |
| `Suspense` | 异步加载状态 |
| `Composition API` | 逻辑组织更灵活 |
| `shallowRef` | 浅层响应式 |
| `script setup` | 语法糖 |

## A.8 Vue 3 破坏性变更

| Vue 2 | Vue 3 |
|-------|-------|
| `new Vue()` | `createApp()` |
| `Vue.component()` | `app.component()` |
| `Vue.use()` | `app.use()` |
| `v-model + .sync` | 统一 `v-model` |
| `$on/$emit` | 移除，用 `mitt` |
| `filters` | 移除，用方法 |

---

> 📌 **文档版本**：v1.0
> 
> 🔄 **最后更新**：2024
