# Vue3入门到高级知识使用例子

> 基于Vue3最新文档的完整学习指南，从基础语法到高级应用的详细示例
> 
> 文档创建时间：2025年1月
> 
> 每个示例都包含详细的注释说明，帮助理解Vue3的核心概念和最佳实践

## 📚 目录

### 🌟 基础篇
- [1. Vue3基础概念与安装](#1-vue3基础概念与安装)
- [2. 响应式数据基础](#2-响应式数据基础)
- [3. 模板语法与插值](#3-模板语法与插值)
- [4. 条件渲染与列表渲染](#4-条件渲染与列表渲染)
- [5. 事件处理](#5-事件处理)
- [6. 表单输入绑定](#6-表单输入绑定)

### 🚀 进阶篇
- [7. Composition API详解](#7-composition-api详解)
- [8. 计算属性与侦听器](#8-计算属性与侦听器)
- [9. 生命周期钩子](#9-生命周期钩子)
- [10. 组件基础](#10-组件基础)
- [11. Props与Emit](#11-props与emit)
- [12. 插槽Slots](#12-插槽slots)

### 🔥 高级篇
- [13. 组合函数Composables](#13-组合函数composables)
- [14. 依赖注入](#14-依赖注入)
- [15. 模板引用与DOM操作](#15-模板引用与dom操作)
- [16. 异步组件与Suspense](#16-异步组件与suspense)
- [17. Teleport传送门](#17-teleport传送门)
- [18. 自定义指令](#18-自定义指令)

### 💡 专家篇
- [19. 渲染函数与JSX](#19-渲染函数与jsx)
- [20. 插件开发](#20-插件开发)
- [21. TypeScript集成](#21-typescript集成)
- [22. 性能优化技巧](#22-性能优化技巧)
- [23. 测试策略](#23-测试策略)
- [24. 最佳实践与设计模式](#24-最佳实践与设计模式)

---

## 1. Vue3基础概念与安装

### 1.1 什么是Vue3
Vue3是一个用于构建用户界面的渐进式JavaScript框架，具有以下特点：
- 更好的性能
- 更好的TypeScript支持
- Composition API
- 更小的包体积

### 1.2 安装与创建项目

```bash
# 使用Vite创建Vue3项目（推荐）
npm create vue@latest my-vue-app

# 或使用Vue CLI
npm install -g @vue/cli
vue create my-vue-app
```

### 1.3 第一个Vue3应用

```vue
<!-- App.vue -->
<template>
  <!-- 模板部分：定义组件的HTML结构 -->
  <div id="app">
    <h1>{{ title }}</h1>
    <p>当前计数：{{ count }}</p>
    <button @click="increment">点击增加</button>
  </div>
</template>

<script setup>
// 使用Composition API的script setup语法
import { ref } from 'vue'

// 定义响应式数据
const title = ref('我的第一个Vue3应用') // ref用于包装基本类型，使其响应式
const count = ref(0) // 初始值为0的响应式数字

// 定义方法
const increment = () => {
  count.value++ // 注意：访问ref的值需要使用.value
}
</script>

<style scoped>
/* 样式部分：scoped表示样式只作用于当前组件 */
#app {
  text-align: center;
  margin-top: 50px;
}

button {
  padding: 10px 20px;
  font-size: 16px;
  background-color: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #369870;
}
</style>
```

### 1.4 Options API vs Composition API对比

```vue
<!-- Options API写法 -->
<template>
  <div>
    <p>计数：{{ count }}</p>
    <button @click="increment">增加</button>
  </div>
</template>

<script>
export default {
  // 数据选项
  data() {
    return {
      count: 0
    }
  },
  // 方法选项
  methods: {
    increment() {
      this.count++
    }
  },
  // 生命周期钩子
  mounted() {
    console.log('组件已挂载')
  }
}
</script>
```

```vue
<!-- Composition API写法 -->
<template>
  <div>
    <p>计数：{{ count }}</p>
    <button @click="increment">增加</button>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

// 所有逻辑都在setup中定义
const count = ref(0) // 响应式数据

const increment = () => { // 方法定义
  count.value++
}

onMounted(() => { // 生命周期钩子
  console.log('组件已挂载')
})
</script>
```

---

## 2. 响应式数据基础

### 2.1 ref() - 基本类型响应式

```vue
<template>
  <div>
    <h2>ref()基本用法</h2>
    
    <!-- 模板中自动解包，不需要.value -->
    <p>姓名：{{ name }}</p>
    <p>年龄：{{ age }}</p>
    <p>是否学生：{{ isStudent }}</p>
    
    <button @click="updateInfo">更新信息</button>
    <button @click="showValues">在控制台显示值</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// ref用于包装基本类型数据，使其具有响应式
const name = ref('张三') // 字符串
const age = ref(25) // 数字
const isStudent = ref(true) // 布尔值

const updateInfo = () => {
  // 在JavaScript中访问和修改ref的值需要使用.value
  name.value = '李四'
  age.value = 30
  isStudent.value = false
}

const showValues = () => {
  // 输出ref的值到控制台
  console.log('姓名:', name.value)
  console.log('年龄:', age.value)
  console.log('是否学生:', isStudent.value)
}
</script>
```

### 2.2 reactive() - 对象响应式

```vue
<template>
  <div>
    <h2>reactive()对象响应式</h2>
    
    <div>
      <h3>用户信息</h3>
      <p>姓名：{{ user.name }}</p>
      <p>邮箱：{{ user.email }}</p>
      <p>地址：{{ user.address.city }}, {{ user.address.country }}</p>
    </div>
    
    <div>
      <h3>爱好列表</h3>
      <ul>
        <li v-for="(hobby, index) in user.hobbies" :key="index">
          {{ hobby }}
        </li>
      </ul>
    </div>
    
    <button @click="updateUser">更新用户信息</button>
    <button @click="addHobby">添加爱好</button>
  </div>
</template>

<script setup>
import { reactive } from 'vue'

// reactive用于创建深度响应式的对象
const user = reactive({
  name: '王五',
  email: 'wangwu@example.com',
  address: {
    city: '北京',
    country: '中国'
  },
  hobbies: ['阅读', '游泳', '编程']
})

const updateUser = () => {
  // 直接修改reactive对象的属性
  user.name = '赵六'
  user.email = 'zhaoliu@example.com'
  user.address.city = '上海' // 深度响应式，嵌套对象也是响应式的
}

const addHobby = () => {
  // 数组操作也是响应式的
  user.hobbies.push('旅行')
}
</script>
```

### 2.3 ref vs reactive 对比示例

```vue
<template>
  <div>
    <h2>ref vs reactive 对比</h2>
    
    <div style="display: flex; gap: 20px;">
      <!-- ref包装对象 -->
      <div>
        <h3>ref包装的对象</h3>
        <p>标题：{{ refObj.title }}</p>
        <p>计数：{{ refObj.count }}</p>
        <button @click="updateRefObj">更新ref对象</button>
      </div>
      
      <!-- reactive对象 -->
      <div>
        <h3>reactive对象</h3>
        <p>标题：{{ reactiveObj.title }}</p>
        <p>计数：{{ reactiveObj.count }}</p>
        <button @click="updateReactiveObj">更新reactive对象</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

// ref也可以包装对象，内部会自动调用reactive
const refObj = ref({
  title: 'ref对象',
  count: 0
})

// reactive直接创建响应式对象
const reactiveObj = reactive({
  title: 'reactive对象',
  count: 0
})

const updateRefObj = () => {
  // ref包装的对象需要通过.value访问
  refObj.value.title = 'ref对象已更新'
  refObj.value.count++
}

const updateReactiveObj = () => {
  // reactive对象直接访问属性
  reactiveObj.title = 'reactive对象已更新'
  reactiveObj.count++
}

// 使用场景建议：
// - 基本类型（string, number, boolean）使用ref
// - 对象和数组使用reactive
// - 需要整体替换对象时使用ref
</script>
```

### 2.4 响应式工具函数

```vue
<template>
  <div>
    <h2>响应式工具函数</h2>
    
    <div>
      <p>只读数据：{{ readonlyData.name }}</p>
      <p>浅层响应式：{{ shallowObj.level1.level2.value }}</p>
      <button @click="tryModifyReadonly">尝试修改只读数据</button>
      <button @click="modifyShallow">修改浅层响应式</button>
    </div>
    
    <div>
      <h3>转换工具演示</h3>
      <p>原始对象计数：{{ state.count }}</p>
      <p>转换为refs后：{{ stateRefs.count }}</p>
      <button @click="modifyState">修改状态</button>
    </div>
  </div>
</template>

<script setup>
import { 
  reactive, 
  readonly, 
  shallowReactive, 
  toRefs, 
  toRef,
  unref,
  isRef,
  isReactive
} from 'vue'

// readonly：创建只读的响应式数据
const originalData = reactive({ name: '原始数据', value: 100 })
const readonlyData = readonly(originalData) // 只读代理

// shallowReactive：只有第一层是响应式的
const shallowObj = shallowReactive({
  level1: {
    level2: {
      value: '深层数据'
    }
  }
})

// toRefs：将reactive对象转换为ref对象
const state = reactive({
  count: 0,
  name: '状态'
})
const stateRefs = toRefs(state) // 转换后可以解构而不失去响应式

const tryModifyReadonly = () => {
  try {
    readonlyData.name = '尝试修改' // 这会在开发环境中产生警告
  } catch (error) {
    console.warn('无法修改只读数据')
  }
}

const modifyShallow = () => {
  // 第一层修改是响应式的
  shallowObj.level1 = { level2: { value: '新的深层数据' } }
  
  // 深层修改不是响应式的（除非整体替换）
  // shallowObj.level1.level2.value = '直接修改深层' // 这不会触发更新
}

const modifyState = () => {
  state.count++ // 修改原对象
  // stateRefs.count.value++ // 或修改转换后的ref，效果相同
}

// 工具函数演示
console.log('isRef(stateRefs.count):', isRef(stateRefs.count)) // true
console.log('isReactive(state):', isReactive(state)) // true
console.log('unref(stateRefs.count):', unref(stateRefs.count)) // 获取ref的值
</script>
```

---

## 3. 模板语法与插值

### 3.1 文本插值与指令基础

```vue
<template>
  <div>
    <h2>模板语法基础</h2>
    
    <!-- 文本插值：双大括号语法 -->
    <p>普通文本插值：{{ message }}</p>
    <p>JavaScript表达式：{{ number + 1 }}</p>
    <p>函数调用：{{ formatDate(new Date()) }}</p>
    <p>三元运算符：{{ isVisible ? '显示' : '隐藏' }}</p>
    
    <!-- v-text指令：纯文本内容 -->
    <p v-text="message"></p>
    
    <!-- v-html指令：HTML内容（注意XSS安全） -->
    <div v-html="htmlContent"></div>
    
    <!-- v-pre指令：跳过编译 -->
    <p v-pre>{{ 这里不会被编译 }}</p>
    
    <!-- v-once指令：只渲染一次 -->
    <p v-once>只渲染一次：{{ message }}</p>
    
    <button @click="updateMessage">更新消息</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// 响应式数据
const message = ref('Hello Vue3!')
const number = ref(10)
const isVisible = ref(true)
const htmlContent = ref('<strong style="color: red;">这是HTML内容</strong>')

// 方法定义
const formatDate = (date) => {
  return date.toLocaleDateString('zh-CN')
}

const updateMessage = () => {
  message.value = '消息已更新 - ' + new Date().toLocaleTimeString()
}
</script>
```

### 3.2 属性绑定与动态属性

```vue
<template>
  <div>
    <h2>属性绑定示例</h2>
    
    <!-- v-bind属性绑定（可简写为:） -->
    <img v-bind:src="imageSrc" v-bind:alt="imageAlt" />
    <img :src="imageSrc" :alt="imageAlt" /> <!-- 简写形式 -->
    
    <!-- 动态属性名 -->
    <div :[attributeName]="attributeValue">动态属性</div>
    
    <!-- 布尔属性绑定 -->
    <button :disabled="isDisabled">{{ isDisabled ? '禁用按钮' : '启用按钮' }}</button>
    
    <!-- 绑定对象到多个属性 -->
    <div v-bind="objectAttrs">绑定对象属性</div>
    
    <!-- class绑定 -->
    <div :class="{ active: isActive, error: hasError }">条件class</div>
    <div :class="[baseClass, { active: isActive }]">数组class</div>
    <div :class="computedClass">计算class</div>
    
    <!-- style绑定 -->
    <div :style="{ color: textColor, fontSize: fontSize + 'px' }">内联样式对象</div>
    <div :style="[baseStyle, overrideStyle]">样式数组</div>
    
    <div>
      <button @click="toggleStates">切换状态</button>
      <button @click="changeAttribute">改变动态属性</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

// 响应式数据
const imageSrc = ref('https://vuejs.org/images/logo.png')
const imageAlt = ref('Vue.js Logo')
const attributeName = ref('id')
const attributeValue = ref('dynamic-id')
const isDisabled = ref(false)
const isActive = ref(true)
const hasError = ref(false)
const baseClass = ref('base-class')
const textColor = ref('blue')
const fontSize = ref(16)

// 对象属性绑定
const objectAttrs = ref({
  id: 'object-bind',
  class: 'object-class',
  'data-value': 'object-data'
})

// 样式对象
const baseStyle = ref({
  border: '1px solid #ccc',
  padding: '10px'
})

const overrideStyle = ref({
  backgroundColor: '#f0f0f0'
})

// 计算属性
const computedClass = computed(() => {
  return {
    'computed-active': isActive.value,
    'computed-error': hasError.value
  }
})

// 方法
const toggleStates = () => {
  isDisabled.value = !isDisabled.value
  isActive.value = !isActive.value
  hasError.value = !hasError.value
  textColor.value = textColor.value === 'blue' ? 'red' : 'blue'
  fontSize.value = fontSize.value === 16 ? 20 : 16
}

const changeAttribute = () => {
  attributeName.value = attributeName.value === 'id' ? 'data-test' : 'id'
  attributeValue.value = 'new-' + Date.now()
}
</script>

<style scoped>
.base-class {
  margin: 10px 0;
}

.active {
  background-color: #42b883;
  color: white;
}

.error {
  border: 2px solid red;
}

.computed-active {
  transform: scale(1.1);
}

.computed-error {
  box-shadow: 0 0 10px red;
}
</style>
```

---

## 4. 条件渲染与列表渲染

### 4.1 条件渲染 v-if / v-show

```vue
<template>
  <div>
    <h2>条件渲染示例</h2>
    
    <div>
      <button @click="toggleShow">切换v-show</button>
      <button @click="toggleIf">切换v-if</button>
      <button @click="changeType">改变类型</button>
    </div>
    
    <!-- v-show：通过CSS display控制显示隐藏 -->
    <div v-show="showElement" class="box">
      v-show控制的元素（DOM中存在，只是隐藏）
    </div>
    
    <!-- v-if：条件性地渲染元素 -->
    <div v-if="ifElement" class="box">
      v-if控制的元素（DOM中动态添加/移除）
    </div>
    
    <!-- v-if / v-else-if / v-else -->
    <div>
      <h3>用户类型显示</h3>
      <p v-if="userType === 'admin'" class="admin">
        管理员用户：拥有所有权限
      </p>
      <p v-else-if="userType === 'member'" class="member">
        会员用户：拥有基础权限
      </p>
      <p v-else-if="userType === 'guest'" class="guest">
        游客用户：只能浏览
      </p>
      <p v-else class="unknown">
        未知用户类型
      </p>
    </div>
    
    <!-- template包装器与v-if -->
    <template v-if="showGroup">
      <h3>分组内容</h3>
      <p>这是一组相关的内容</p>
      <p>它们会同时显示或隐藏</p>
    </template>
    
    <div>
      <label>
        <input type="checkbox" v-model="showGroup" />
        显示分组内容
      </label>
    </div>
    
    <!-- 动态组件与v-if结合 -->
    <div>
      <h3>动态组件示例</h3>
      <component :is="currentComponent" v-if="currentComponent" />
      <button @click="switchComponent">切换组件</button>
    </div>
  </div>
</template>

<script setup>
import { ref, markRaw } from 'vue'

// 响应式数据
const showElement = ref(true)
const ifElement = ref(true)
const userType = ref('admin')
const showGroup = ref(true)
const currentComponent = ref(null)

// 示例组件
const ComponentA = markRaw({
  template: '<div class="component-a">组件A</div>'
})

const ComponentB = markRaw({
  template: '<div class="component-b">组件B</div>'
})

const components = [null, ComponentA, ComponentB]
let componentIndex = 0

// 方法
const toggleShow = () => {
  showElement.value = !showElement.value
}

const toggleIf = () => {
  ifElement.value = !ifElement.value
}

const changeType = () => {
  const types = ['admin', 'member', 'guest', 'unknown']
  const currentIndex = types.indexOf(userType.value)
  userType.value = types[(currentIndex + 1) % types.length]
}

const switchComponent = () => {
  componentIndex = (componentIndex + 1) % components.length
  currentComponent.value = components[componentIndex]
}

// 性能提示：
// v-show适用于频繁切换的元素（开销在初始渲染）
// v-if适用于条件很少改变的元素（开销在切换）
</script>

<style scoped>
.box {
  padding: 10px;
  margin: 10px 0;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.admin { color: red; font-weight: bold; }
.member { color: blue; }
.guest { color: green; }
.unknown { color: gray; }

.component-a {
  background-color: #ffcccc;
  padding: 10px;
}

.component-b {
  background-color: #ccffcc;
  padding: 10px;
}
</style>
```

### 4.2 列表渲染 v-for

```vue
<template>
  <div>
    <h2>列表渲染示例</h2>
    
    <div>
      <button @click="addItem">添加项目</button>
      <button @click="removeItem">删除项目</button>
      <button @click="shuffleItems">打乱顺序</button>
      <button @click="updateItem">更新第一项</button>
    </div>
    
    <!-- 基本数组渲染 -->
    <div>
      <h3>基本数组渲染</h3>
      <ul>
        <li v-for="(item, index) in items" :key="item.id">
          {{ index + 1 }}. {{ item.name }} - {{ item.value }}
          <button @click="removeSpecificItem(item.id)">删除</button>
        </li>
      </ul>
    </div>
    
    <!-- 对象渲染 -->
    <div>
      <h3>对象属性渲染</h3>
      <ul>
        <li v-for="(value, key, index) in userInfo" :key="key">
          {{ index + 1 }}. {{ key }}: {{ value }}
        </li>
      </ul>
    </div>
    
    <!-- 数字范围渲染 -->
    <div>
      <h3>数字范围渲染</h3>
      <span v-for="n in 10" :key="n" class="number">
        {{ n }}
      </span>
    </div>
    
    <!-- 嵌套v-for -->
    <div>
      <h3>嵌套列表渲染</h3>
      <div v-for="category in categories" :key="category.id" class="category">
        <h4>{{ category.name }}</h4>
        <ul>
          <li v-for="product in category.products" :key="product.id">
            {{ product.name }} - ¥{{ product.price }}
          </li>
        </ul>
      </div>
    </div>
    
    <!-- 模板元素与v-for -->
    <div>
      <h3>模板元素分组</h3>
      <template v-for="item in groupedItems" :key="item.id">
        <h4>{{ item.title }}</h4>
        <p>{{ item.description }}</p>
        <hr>
      </template>
    </div>
    
    <!-- v-for与v-if结合（不推荐在同一元素上） -->
    <div>
      <h3>过滤列表渲染</h3>
      <template v-for="item in items" :key="item.id">
        <li v-if="item.visible">
          {{ item.name }} (仅显示可见项)
        </li>
      </template>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

// 响应式数据
const items = ref([
  { id: 1, name: '项目1', value: 100, visible: true },
  { id: 2, name: '项目2', value: 200, visible: false },
  { id: 3, name: '项目3', value: 300, visible: true }
])

const userInfo = reactive({
  name: '张三',
  age: 25,
  email: 'zhangsan@example.com',
  city: '北京'
})

const categories = ref([
  {
    id: 1,
    name: '电子产品',
    products: [
      { id: 101, name: '手机', price: 3999 },
      { id: 102, name: '笔记本', price: 5999 }
    ]
  },
  {
    id: 2,
    name: '服装',
    products: [
      { id: 201, name: 'T恤', price: 99 },
      { id: 202, name: '牛仔裤', price: 299 }
    ]
  }
])

const groupedItems = ref([
  {
    id: 1,
    title: '标题1',
    description: '这是第一组内容的描述'
  },
  {
    id: 2,
    title: '标题2',
    description: '这是第二组内容的描述'
  }
])

let nextId = 4

// 方法
const addItem = () => {
  items.value.push({
    id: nextId++,
    name: `新项目${nextId - 1}`,
    value: Math.floor(Math.random() * 1000),
    visible: Math.random() > 0.5
  })
}

const removeItem = () => {
  if (items.value.length > 0) {
    items.value.pop()
  }
}

const removeSpecificItem = (id) => {
  const index = items.value.findIndex(item => item.id === id)
  if (index > -1) {
    items.value.splice(index, 1)
  }
}

const shuffleItems = () => {
  // Fisher-Yates洗牌算法
  for (let i = items.value.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[items.value[i], items.value[j]] = [items.value[j], items.value[i]]
  }
}

const updateItem = () => {
  if (items.value.length > 0) {
    items.value[0].name = '已更新的项目1'
    items.value[0].value = Math.floor(Math.random() * 1000)
  }
}

// 重要提示：
// 1. 始终为v-for提供唯一的key，最好使用稳定的id
// 2. 避免使用index作为key，除非列表是静态的
// 3. 不要在同一元素上同时使用v-for和v-if
</script>

<style scoped>
.number {
  display: inline-block;
  margin: 2px;
  padding: 5px 8px;
  background-color: #42b883;
  color: white;
  border-radius: 4px;
}

.category {
  margin: 10px 0;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.category h4 {
  margin: 0 0 10px 0;
  color: #42b883;
}
</style>
```

---

## 5. 事件处理

### 5.1 基础事件处理

```vue
<template>
  <div>
    <h2>事件处理基础</h2>
    
    <!-- 基本事件绑定 -->
    <div>
      <h3>基本点击事件</h3>
      <button @click="simpleClick">简单点击</button>
      <button @click="clickWithParam('参数')">带参数点击</button>
      <button @click="clickCounter++">内联表达式</button>
      <p>点击计数：{{ clickCounter }}</p>
    </div>
    
    <!-- 事件对象访问 -->
    <div>
      <h3>事件对象访问</h3>
      <button @click="handleEvent">获取事件对象</button>
      <button @click="handleEventWithParam($event, '额外参数')">事件对象+参数</button>
    </div>
    
    <!-- 多种事件类型 -->
    <div>
      <h3>多种事件类型</h3>
      <input 
        type="text" 
        @input="handleInput"
        @focus="handleFocus"
        @blur="handleBlur"
        @keyup="handleKeyup"
        placeholder="试试输入、聚焦、失焦"
      />
      <p>输入值：{{ inputValue }}</p>
      <p>事件日志：{{ eventLog }}</p>
    </div>
    
    <!-- 鼠标事件 -->
    <div>
      <h3>鼠标事件</h3>
      <div 
        class="mouse-area"
        @mouseenter="handleMouseEnter"
        @mouseleave="handleMouseLeave"
        @mousemove="handleMouseMove"
        @click="handleMouseClick"
        @dblclick="handleDoubleClick"
        @contextmenu.prevent="handleRightClick"
      >
        鼠标操作区域<br>
        坐标：({{ mouseX }}, {{ mouseY }})<br>
        状态：{{ mouseStatus }}
      </div>
    </div>
    
    <!-- 键盘事件 -->
    <div>
      <h3>键盘事件</h3>
      <input 
        type="text"
        @keydown="handleKeydown"
        @keyup="handleKeyup"
        @keypress="handleKeypress"
        placeholder="试试按键盘"
      />
      <p>按键信息：{{ keyInfo }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// 响应式数据
const clickCounter = ref(0)
const inputValue = ref('')
const eventLog = ref('无事件')
const mouseX = ref(0)
const mouseY = ref(0)
const mouseStatus = ref('外部')
const keyInfo = ref('无按键')

// 事件处理方法
const simpleClick = () => {
  console.log('简单点击事件触发')
  eventLog.value = '简单点击 - ' + new Date().toLocaleTimeString()
}

const clickWithParam = (param) => {
  console.log('带参数点击:', param)
  eventLog.value = `带参数点击: ${param}`
}

const handleEvent = (event) => {
  console.log('事件对象:', event)
  eventLog.value = `事件类型: ${event.type}, 目标: ${event.target.tagName}`
}

const handleEventWithParam = (event, param) => {
  console.log('事件对象:', event, '参数:', param)
  eventLog.value = `${event.type} + 参数: ${param}`
}

const handleInput = (event) => {
  inputValue.value = event.target.value
  eventLog.value = `输入事件: ${event.target.value}`
}

const handleFocus = (event) => {
  eventLog.value = '输入框获得焦点'
}

const handleBlur = (event) => {
  eventLog.value = '输入框失去焦点'
}

const handleKeyup = (event) => {
  eventLog.value = `按键释放: ${event.key}`
}

// 鼠标事件处理
const handleMouseEnter = () => {
  mouseStatus.value = '内部'
}

const handleMouseLeave = () => {
  mouseStatus.value = '外部'
}

const handleMouseMove = (event) => {
  mouseX.value = event.offsetX
  mouseY.value = event.offsetY
}

const handleMouseClick = (event) => {
  eventLog.value = `鼠标单击: (${event.offsetX}, ${event.offsetY})`
}

const handleDoubleClick = (event) => {
  eventLog.value = `鼠标双击: (${event.offsetX}, ${event.offsetY})`
}

const handleRightClick = (event) => {
  eventLog.value = '鼠标右键点击（阻止了默认菜单）'
}

// 键盘事件处理
const handleKeydown = (event) => {
  keyInfo.value = `按下: ${event.key} (code: ${event.code})`
}

const handleKeypress = (event) => {
  keyInfo.value = `按键: ${event.key} (charCode: ${event.charCode})`
}
</script>

<style scoped>
.mouse-area {
  width: 300px;
  height: 150px;
  border: 2px solid #42b883;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  cursor: pointer;
  user-select: none;
  margin: 10px 0;
}

.mouse-area:hover {
  background-color: #f0f9ff;
}

input {
  padding: 8px;
  margin: 5px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

input:focus {
  outline: none;
  border-color: #42b883;
}
</style>
```

### 5.2 事件修饰符详细示例

```vue
<template>
  <div>
    <h2>事件修饰符全面示例</h2>
    
    <!-- .prevent修饰符：阻止默认行为 -->
    <div>
      <h3>.prevent - 阻止默认行为</h3>
      <form @submit.prevent="handleSubmit">
        <input type="text" placeholder="输入内容" />
        <button type="submit">提交 (阻止页面刷新)</button>
      </form>
      <a href="https://vue.js.org" @click.prevent="handleLinkClick">
        点击链接 (阻止跳转)
      </a>
    </div>
    
    <!-- .stop修饰符：阻止事件冒泡 -->
    <div>
      <h3>.stop - 阻止事件冒泡</h3>
      <div class="outer-box" @click="handleOuterClick">
        外层容器 (会触发)
        <div class="inner-box" @click.stop="handleInnerClick">
          内层容器 (阻止冒泡)
        </div>
      </div>
      <p>冒泡测试结果：{{ bubbleResult }}</p>
    </div>
    
    <!-- 按键修饰符 -->
    <div>
      <h3>按键修饰符</h3>
      <input 
        type="text" 
        @keyup.enter="handleEnter"
        @keyup.tab="handleTab"
        @keyup.delete="handleDelete"
        @keyup.esc="handleEsc"
        placeholder="试试Enter、Tab、Delete、Esc键"
      />
      <p>按键操作记录：{{ keyResult }}</p>
    </div>
    
    <!-- 系统修饰键 -->
    <div>
      <h3>系统修饰键组合</h3>
      <button @click.ctrl="handleCtrlClick">Ctrl + Click</button>
      <button @click.alt="handleAltClick">Alt + Click</button>
      <button @click.shift="handleShiftClick">Shift + Click</button>
      <input 
        type="text" 
        @keyup.ctrl.enter="handleCtrlEnter"
        placeholder="试试Ctrl+Enter"
      />
      <p>系统键操作：{{ systemKeyResult }}</p>
    </div>
    
    <!-- .once修饰符：只触发一次 -->
    <div>
      <h3>.once - 只触发一次</h3>
      <button @click.once="handleOnceClick">只能点击一次的按钮</button>
      <p>点击次数：{{ onceClickCount }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// 响应式数据
const bubbleResult = ref('等待测试')
const keyResult = ref('等待按键操作')
const systemKeyResult = ref('等待系统键操作')
const onceClickCount = ref(0)

// .prevent 阻止默认行为
const handleSubmit = () => {
  console.log('表单提交，但页面不会刷新')
  alert('表单已提交(阻止了默认刷新行为)')
}

const handleLinkClick = () => {
  console.log('链接被点击，但不会跳转')
  alert('链接被点击(阻止了默认跳转行为)')
}

// .stop 阻止事件冒泡
const handleOuterClick = () => {
  console.log('外层容器被点击')
  bubbleResult.value = '外层容器被点击 - ' + new Date().toLocaleTimeString()
}

const handleInnerClick = () => {
  console.log('内层容器被点击，阻止冒泡')
  bubbleResult.value = '内层容器被点击(已阻止冒泡) - ' + new Date().toLocaleTimeString()
}

// 按键修饰符处理
const handleEnter = () => {
  keyResult.value = 'Enter键被按下 - ' + new Date().toLocaleTimeString()
}

const handleTab = () => {
  keyResult.value = 'Tab键被按下 - ' + new Date().toLocaleTimeString()
}

const handleDelete = () => {
  keyResult.value = 'Delete键被按下 - ' + new Date().toLocaleTimeString()
}

const handleEsc = () => {
  keyResult.value = 'Esc键被按下 - ' + new Date().toLocaleTimeString()
}

// 系统修饰键处理
const handleCtrlClick = () => {
  systemKeyResult.value = 'Ctrl + 点击 - ' + new Date().toLocaleTimeString()
}

const handleAltClick = () => {
  systemKeyResult.value = 'Alt + 点击 - ' + new Date().toLocaleTimeString()
}

const handleShiftClick = () => {
  systemKeyResult.value = 'Shift + 点击 - ' + new Date().toLocaleTimeString()
}

const handleCtrlEnter = () => {
  systemKeyResult.value = 'Ctrl + Enter 组合键 - ' + new Date().toLocaleTimeString()
}

// .once 只触发一次
const handleOnceClick = () => {
  onceClickCount.value++
  console.log('只能触发一次的事件')
  alert('这个事件只能触发一次!')
}
</script>

<style scoped>
.outer-box {
  padding: 30px;
  background-color: #e3f2fd;
  border: 2px solid #2196f3;
  border-radius: 8px;
  cursor: pointer;
  margin: 10px 0;
}

.inner-box {
  padding: 20px;
  background-color: #ffecb3;
  border: 2px solid #ffa000;
  border-radius: 4px;
  margin: 10px;
}

button {
  margin: 5px;
  padding: 8px 16px;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
  background-color: #f5f5f5;
}

button:hover {
  background-color: #e0e0e0;
}

input {
  display: block;
  margin: 10px 0;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  width: 100%;
  max-width: 400px;
}
</style>
```

**事件修饰符使用要点：**

1. **性能优化**：
   - `.passive` 主要用于滚动事件，提升滚动性能
   - `.once` 减少不必要的重复执行

2. **修饰符顺序**：
   - 修饰符的顺序很重要：`@click.prevent.self` 会阻止所有点击的默认行为
   - 而 `@click.self.prevent` 只会阻止对元素自身点击的默认行为

3. **按键码**：
   - 可以使用按键码：`@keyup.13` 等同于 `@keyup.enter`
   - 推荐使用语义化的按键名称

---

## 6. 表单输入绑定

### 6.1 基础表单绑定

```vue
<template>
  <div>
    <h2>Vue3表单输入绑定完整示例</h2>
    
    <!-- 文本输入 -->
    <div class="form-section">
      <h3>文本输入绑定</h3>
      
      <!-- 单行文本输入 -->
      <div>
        <label>单行文本：</label>
        <input 
          type="text" 
          v-model="textInput" 
          placeholder="请输入文本"
        />
        <p>输入值：{{ textInput }}</p>
      </div>
      
      <!-- 多行文本输入 -->
      <div>
        <label>多行文本：</label>
        <textarea 
          v-model="textareaInput" 
          placeholder="请输入多行文本"
          rows="4"
        ></textarea>
        <p>多行文本内容：</p>
        <pre>{{ textareaInput }}</pre>
      </div>
    </div>
    
    <!-- 复选框绑定 -->
    <div class="form-section">
      <h3>复选框绑定</h3>
      
      <!-- 单个复选框 -->
      <div>
        <input 
          type="checkbox" 
          id="single-checkbox" 
          v-model="singleChecked"
        />
        <label for="single-checkbox">单个复选框</label>
        <p>选中状态：{{ singleChecked }}</p>
      </div>
      
      <!-- 多个复选框绑定到数组 -->
      <div>
        <p>选择你喜欢的技术：</p>
        <input type="checkbox" id="vue" value="Vue" v-model="checkedTechs" />
        <label for="vue">Vue</label>
        
        <input type="checkbox" id="react" value="React" v-model="checkedTechs" />
        <label for="react">React</label>
        
        <input type="checkbox" id="angular" value="Angular" v-model="checkedTechs" />
        <label for="angular">Angular</label>
        
        <p>已选择：{{ checkedTechs }}</p>
      </div>
    </div>
    
    <!-- 单选按钮绑定 -->
    <div class="form-section">
      <h3>单选按钮绑定</h3>
      
      <div>
        <p>选择你的经验水平：</p>
        <input type="radio" id="beginner" value="初学者" v-model="experienceLevel" />
        <label for="beginner">初学者</label>
        
        <input type="radio" id="intermediate" value="中级" v-model="experienceLevel" />
        <label for="intermediate">中级</label>
        
        <input type="radio" id="advanced" value="高级" v-model="experienceLevel" />
        <label for="advanced">高级</label>
        
        <p>选择的水平：{{ experienceLevel }}</p>
      </div>
    </div>
    
    <!-- 选择框绑定 -->
    <div class="form-section">
      <h3>选择框绑定</h3>
      
      <!-- 单选下拉框 -->
      <div>
        <label>选择城市：</label>
        <select v-model="selectedCity">
          <option disabled value="">请选择城市</option>
          <option value="北京">北京</option>
          <option value="上海">上海</option>
          <option value="广州">广州</option>
          <option value="深圳">深圳</option>
        </select>
        <p>选择的城市：{{ selectedCity }}</p>
      </div>
      
      <!-- 多选下拉框 -->
      <div>
        <label>选择编程语言（多选）：</label>
        <select v-model="selectedLanguages" multiple size="4">
          <option value="JavaScript">JavaScript</option>
          <option value="TypeScript">TypeScript</option>
          <option value="Python">Python</option>
          <option value="Java">Java</option>
        </select>
        <p>选择的语言：{{ selectedLanguages }}</p>
      </div>
      
      <!-- 动态选项 -->
      <div>
        <label>动态选项：</label>
        <select v-model="dynamicSelected">
          <option 
            v-for="option in dynamicOptions" 
            :key="option.value" 
            :value="option.value"
          >
            {{ option.text }}
          </option>
        </select>
        <p>动态选择：{{ dynamicSelected }}</p>
        <button @click="addDynamicOption">添加选项</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// 文本输入响应式数据
const textInput = ref('') // 单行文本输入绑定
const textareaInput = ref('') // 多行文本输入绑定

// 复选框响应式数据
const singleChecked = ref(false) // 单个复选框状态
const checkedTechs = ref([]) // 多个复选框绑定到数组

// 单选按钮响应式数据
const experienceLevel = ref('') // 单选按钮组绑定

// 选择框响应式数据
const selectedCity = ref('') // 单选下拉框
const selectedLanguages = ref([]) // 多选下拉框绑定到数组
const dynamicSelected = ref('A') // 动态选项选择

// 动态选项数据
const dynamicOptions = ref([
  { text: '选项一', value: 'A' },
  { text: '选项二', value: 'B' },
  { text: '选项三', value: 'C' }
])

// 添加动态选项的方法
const addDynamicOption = () => {
  const newValue = String.fromCharCode(65 + dynamicOptions.value.length) // A, B, C, D...
  dynamicOptions.value.push({
    text: `选项${dynamicOptions.value.length + 1}`,
    value: newValue
  })
}
</script>

<style scoped>
.form-section {
  margin: 30px 0;
  padding: 20px;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  background-color: #fafafa;
}

.form-section h3 {
  margin-top: 0;
  color: #42b883;
  border-bottom: 2px solid #42b883;
  padding-bottom: 10px;
}

label {
  display: inline-block;
  margin-right: 10px;
  margin-bottom: 5px;
  font-weight: 500;
}

input[type="text"],
textarea,
select {
  width: 100%;
  max-width: 400px;
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  margin-bottom: 10px;
}

input[type="text"]:focus,
textarea:focus,
select:focus {
  outline: none;
  border-color: #42b883;
  box-shadow: 0 0 0 2px rgba(66, 184, 131, 0.2);
}

input[type="checkbox"],
input[type="radio"] {
  margin-right: 8px;
  margin-left: 15px;
}

input[type="checkbox"]:first-child,
input[type="radio"]:first-child {
  margin-left: 0;
}

select[multiple] {
  height: auto;
}

pre {
  background-color: #f5f5f5;
  padding: 10px;
  border-radius: 4px;
  border: 1px solid #ddd;
  white-space: pre-wrap;
}

button {
  padding: 8px 16px;
  background-color: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin: 5px 0;
}

button:hover {
  background-color: #369870;
}
</style>
```

### 6.2 表单修饰符

```vue
<template>
  <div>
    <h2>v-model修饰符示例</h2>
    
    <!-- .lazy修饰符：在change事件后更新 -->
    <div class="form-section">
      <h3>.lazy - 延迟更新</h3>
      <input 
        type="text" 
        v-model.lazy="lazyText" 
        placeholder="失去焦点后才更新"
      />
      <p>延迟更新值：{{ lazyText }}</p>
      
      <!-- 对比普通的v-model -->
      <input 
        type="text" 
        v-model="normalText" 
        placeholder="实时更新"
      />
      <p>普通更新值：{{ normalText }}</p>
    </div>
    
    <!-- .number修饰符：自动类型转换 -->
    <div class="form-section">
      <h3>.number - 数字类型转换</h3>
      <input 
        type="number" 
        v-model.number="numberValue" 
        placeholder="输入数字"
      />
      <p>数字值：{{ numberValue }} (类型: {{ typeof numberValue }})</p>
      
      <!-- 对比普通的v-model -->
      <input 
        type="number" 
        v-model="stringNumber" 
        placeholder="普通数字输入"
      />
      <p>字符串数字：{{ stringNumber }} (类型: {{ typeof stringNumber }})</p>
    </div>
    
    <!-- .trim修饰符：去除首尾空格 -->
    <div class="form-section">
      <h3>.trim - 去除首尾空格</h3>
      <input 
        type="text" 
        v-model.trim="trimText" 
        placeholder="输入带空格的文本"
      />
      <p>去除空格后：'{{ trimText }}'</p>
      
      <!-- 对比普通的v-model -->
      <input 
        type="text" 
        v-model="normalTrimText" 
        placeholder="普通文本输入"
      />
      <p>保留空格：'{{ normalTrimText }}'</p>
    </div>
    
    <!-- 组合使用修饰符 -->
    <div class="form-section">
      <h3>组合使用修饰符</h3>
      <input 
        type="number" 
        v-model.number.lazy.trim="combinedValue" 
        placeholder="组合使用: number + lazy + trim"
      />
      <p>组合处理结果：{{ combinedValue }} (类型: {{ typeof combinedValue }})</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// 修饰符示例响应式数据
const lazyText = ref('') // .lazy修饰符示例
const normalText = ref('') // 普通更新对比

const numberValue = ref(0) // .number修饰符示例
const stringNumber = ref('') // 普通数字输入对比

const trimText = ref('') // .trim修饰符示例
const normalTrimText = ref('') // 普通文本输入对比

const combinedValue = ref(0) // 组合修饰符示例
</script>

<style scoped>
/* 样式与上面保持一致 */
.form-section {
  margin: 30px 0;
  padding: 20px;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  background-color: #fafafa;
}

.form-section h3 {
  margin-top: 0;
  color: #42b883;
  border-bottom: 2px solid #42b883;
  padding-bottom: 10px;
}

input {
  width: 100%;
  max-width: 400px;
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  margin-bottom: 10px;
}

input:focus {
  outline: none;
  border-color: #42b883;
  box-shadow: 0 0 0 2px rgba(66, 184, 131, 0.2);
}

p {
  margin: 10px 0;
  padding: 10px;
  background-color: #f0f8ff;
  border-left: 4px solid #42b883;
  border-radius: 4px;
}
</style>
```

**v-model修饰符总结：**

1. **性能优化**：
   - `.lazy` 可以减少频繁的数据更新，提升性能
   - 适用于不需要实时数据同步的场景

2. **数据类型**：
   - `.number` 确保数字输入的正确类型，避免字符串计算错误
   - 对于数学计算类的表单非常重要

3. **数据清理**：
   - `.trim` 自动处理用户输入的空格，提升用户体验
   - 适用于用户名、邮箱等输入框

---

## 7. Composition API详解

### 7.1 setup函数基础

```vue
<template>
  <div>
    <h2>Composition API setup函数详解</h2>
    
    <!-- 使用setup返回的数据和方法 -->
    <div class="demo-section">
      <h3>基础使用</h3>
      <p>计数器：{{ count }}</p>
      <p>消息：{{ message }}</p>
      <button @click="increment">增加</button>
      <button @click="updateMessage">更新消息</button>
    </div>
    
    <!-- setup中的计算属性 -->
    <div class="demo-section">
      <h3>计算属性</h3>
      <p>双倍值：{{ doubleCount }}</p>
      <p>特殊消息：{{ specialMessage }}</p>
    </div>
    
    <!-- setup中的侦听器 -->
    <div class="demo-section">
      <h3>侦听器演示</h3>
      <p>侦听日志：{{ watchLog }}</p>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'

export default {
  name: 'CompositionAPIDemo',
  
  // setup函数是Composition API的入口点
  setup(props, context) {
    console.log('setup函数被调用')
    console.log('props:', props)
    console.log('context:', context)
    
    // 1. 响应式数据定义
    const count = ref(0) // 使用ref创建响应式数据
    const message = ref('初始消息') // 字符串类型的响应式数据
    const watchLog = ref('等待侦听事件') // 用于显示侦听日志
    
    // 2. 计算属性定义
    const doubleCount = computed(() => {
      console.log('计算属性重新计算')
      return count.value * 2 // 计算属性依赖count的值
    })
    
    const specialMessage = computed(() => {
      return `特殊处理: ${message.value.toUpperCase()}` // 字符串处理
    })
    
    // 3. 方法定义
    const increment = () => {
      count.value++ // 修改ref的值需要使用.value
      console.log('计数器增加到:', count.value)
    }
    
    const updateMessage = () => {
      const messages = ['你好世界', 'Hello Vue3', '组合式API很棒']
      const randomIndex = Math.floor(Math.random() * messages.length)
      message.value = messages[randomIndex]
      console.log('消息已更新为:', message.value)
    }
    
    // 4. 侦听器定义
    watch(count, (newValue, oldValue) => {
      watchLog.value = `count从 ${oldValue} 变为 ${newValue}`
      console.log('count值发生变化:', { newValue, oldValue })
    })
    
    watch(message, (newValue, oldValue) => {
      watchLog.value = `message从 "${oldValue}" 变为 "${newValue}"`
      console.log('message发生变化:', { newValue, oldValue })
    })
    
    // 5. 生命周期钩子
    onMounted(() => {
      console.log('组件已挂载 - onMounted')
      watchLog.value = '组件已成功挂载'
    })
    
    onUnmounted(() => {
      console.log('组件即将卸载 - onUnmounted')
    })
    
    // 6. 返回模板中需要使用的数据和方法
    return {
      // 响应式数据
      count,
      message,
      watchLog,
      
      // 计算属性
      doubleCount,
      specialMessage,
      
      // 方法
      increment,
      updateMessage
    }
  },
  
  // 也可以与Options API混用
  mounted() {
    console.log('Options API mounted钩子也会执行')
  }
}
</script>

<style scoped>
.demo-section {
  margin: 20px 0;
  padding: 15px;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  background-color: #f9f9f9;
}

.demo-section h3 {
  margin-top: 0;
  color: #42b883;
}

button {
  margin: 5px;
  padding: 8px 16px;
  background-color: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #369870;
}

p {
  margin: 10px 0;
}
</style>
```

### 7.2 <script setup> 语法糖

```vue
<template>
  <div>
    <h2><script setup> 语法糖示例</h2>
    
    <!-- 所有在script setup中定义的变量都自动暴露给模板 -->
    <div class="setup-section">
      <h3>自动暴露的响应式数据</h3>
      <p>用户名：{{ username }}</p>
      <p>年龄：{{ age }}</p>
      <p>是否是管理员：{{ isAdmin ? '是' : '否' }}</p>
      
      <button @click="updateUserInfo">更新用户信息</button>
      <button @click="toggleAdmin">切换管理员状态</button>
    </div>
    
    <!-- 生命周期钩子 -->
    <div class="setup-section">
      <h3>生命周期事件</h3>
      <p>组件状态：{{ lifecycleStatus }}</p>
    </div>
  </div>
</template>

<script setup>
// <script setup> 语法糖的优势：
// 1. 更简洁的语法，不需要setup函数和return
// 2. 更好的TypeScript支持
// 3. 编译时性能更好
// 4. 组件和组合式函数自动导入

import { ref, reactive, computed, watch, onMounted, onUpdated, onUnmounted } from 'vue'

// 1. 响应式数据定义 - 自动暴露给模板
const username = ref('张三') // 基本类型使用ref
const age = ref(25)
const isAdmin = ref(false)

// 使用reactive创建响应式对象
const userInfo = reactive({
  id: 1,
  name: '张三',
  email: 'zhangsan@example.com',
  avatar: 'https://via.placeholder.com/100',
  lastLogin: new Date().toLocaleString()
})

// 2. 计算属性 - 自动暴露
const userDisplayName = computed(() => {
  return isAdmin.value ? `[Admin] ${username.value}` : username.value
})

// 3. 方法定义 - 自动暴露
const updateUserInfo = () => {
  const names = ['张三', '李四', '王五', '赵六']
  const randomName = names[Math.floor(Math.random() * names.length)]
  
  username.value = randomName
  age.value = Math.floor(Math.random() * 50) + 18
  
  // 更新reactive对象
  userInfo.name = randomName
  userInfo.lastLogin = new Date().toLocaleString()
}

const toggleAdmin = () => {
  isAdmin.value = !isAdmin.value
}

// 4. 生命周期钩子
const lifecycleStatus = ref('组件初始化中...')

onMounted(() => {
  console.log('组件已挂载')
  lifecycleStatus.value = '组件已挂载并正常运行'
})

onUpdated(() => {
  console.log('组件已更新')
})

onUnmounted(() => {
  console.log('组件即将卸载')
  lifecycleStatus.value = '组件即将卸载'
})

// 5. 侦听器
watch(username, (newName, oldName) => {
  console.log(`用户名从 ${oldName} 变更为 ${newName}`)
  // 同步更新userInfo
  userInfo.name = newName
})

// 侦听多个数据源
watch([age, isAdmin], ([newAge, newIsAdmin], [oldAge, oldIsAdmin]) => {
  console.log('用户信息发生变化:', {
    age: { old: oldAge, new: newAge },
    isAdmin: { old: oldIsAdmin, new: newIsAdmin }
  })
})

// 侦听reactive对象
watch(userInfo, (newUserInfo) => {
  console.log('用户信息对象发生变化:', newUserInfo)
}, { deep: true }) // 深度侦听
</script>

<style scoped>
.setup-section {
  margin: 25px 0;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  background-color: #f8f9fa;
}

.setup-section h3 {
  margin-top: 0;
  color: #42b883;
  border-bottom: 1px solid #42b883;
  padding-bottom: 8px;
}

button {
  margin: 5px;
  padding: 10px 15px;
  background-color: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

button:hover {
  background-color: #369870;
}

button:active {
  transform: translateY(1px);
}

p {
  margin: 8px 0;
  font-size: 14px;
}
</style>
```

---

## 8. 计算属性与侦听器

### 8.1 computed计算属性详解

```vue
<template>
  <div>
    <h2>Vue3计算属性深入使用</h2>
    
    <!-- 基础计算属性 -->
    <div class="computed-section">
      <h3>基础计算属性</h3>
      <div>
        <label>名字：</label>
        <input v-model="firstName" placeholder="请输入名字" />
      </div>
      <div>
        <label>姓氏：</label>
        <input v-model="lastName" placeholder="请输入姓氏" />
      </div>
      <p><strong>完整姓名：</strong>{{ fullName }}</p>
      <p><strong>姓名长度：</strong>{{ nameLength }}</p>
    </div>
    
    <!-- 可写计算属性 -->
    <div class="computed-section">
      <h3>可写计算属性</h3>
      <div>
        <label>完整姓名：</label>
        <input v-model="writableFullName" placeholder="输入完整姓名会自动分离" />
      </div>
      <p><strong>分离后的名字：</strong>{{ firstName }}</p>
      <p><strong>分离后的姓氏：</strong>{{ lastName }}</p>
    </div>
    
    <!-- 计算属性缓存演示 -->
    <div class="computed-section">
      <h3>计算属性缓存机制</h3>
      <p>当前时间（方法调用）：{{ getCurrentTime() }}</p>
      <p>当前时间（计算属性）：{{ computedCurrentTime }}</p>
      <p>依赖变化的时间：{{ dependentTime }}</p>
      <button @click="triggerChange">触发依赖变化</button>
      <p><small>注意：方法每次都会重新执行，计算属性只在依赖变化时重新计算</small></p>
    </div>
    
    <!-- 复杂计算属性示例 -->
    <div class="computed-section">
      <h3>复杂计算属性示例</h3>
      <div>
        <h4>购物车计算</h4>
        <div v-for="item in cartItems" :key="item.id" class="cart-item">
          <span>{{ item.name }}</span>
          <span>单价：¥{{ item.price }}</span>
          <input 
            type="number" 
            v-model.number="item.quantity" 
            min="0" 
            style="width: 60px; margin: 0 10px;"
          />
          <span>小计：¥{{ (item.price * item.quantity).toFixed(2) }}</span>
        </div>
        <div class="cart-summary">
          <p><strong>商品总数：</strong>{{ totalQuantity }}</p>
          <p><strong>总金额：</strong>¥{{ totalPrice }}</p>
          <p><strong>优惠金额：</strong>¥{{ discountAmount }}</p>
          <p><strong>最终价格：</strong>¥{{ finalPrice }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, reactive } from 'vue'

// 基础数据
const firstName = ref('张')
const lastName = ref('三')
const dependencyTrigger = ref(0) // 用于触发依赖变化

// 1. 基础只读计算属性
const fullName = computed(() => {
  console.log('fullName计算属性被重新计算')
  return `${lastName.value} ${firstName.value}`
})

const nameLength = computed(() => {
  return fullName.value.length
})

// 2. 可写计算属性
const writableFullName = computed({
  // getter：读取时的逻辑
  get() {
    return `${lastName.value} ${firstName.value}`
  },
  // setter：写入时的逻辑
  set(newValue) {
    console.log('设置新的完整姓名:', newValue)
    const names = newValue.split(' ')
    if (names.length >= 2) {
      lastName.value = names[0] || ''
      firstName.value = names.slice(1).join(' ') || ''
    } else {
      firstName.value = newValue
    }
  }
})

// 3. 缓存机制演示
// 方法：每次调用都重新执行
const getCurrentTime = () => {
  console.log('getCurrentTime方法被调用')
  return new Date().toLocaleTimeString()
}

// 计算属性：只在依赖变化时重新计算
const computedCurrentTime = computed(() => {
  console.log('computedCurrentTime计算属性被重新计算')
  return new Date().toLocaleTimeString()
})

// 依赖于响应式数据的计算属性
const dependentTime = computed(() => {
  console.log('dependentTime计算属性被重新计算')
  // 依赖于dependencyTrigger的变化
  return `${dependencyTrigger.value} - ${new Date().toLocaleTimeString()}`
})

const triggerChange = () => {
  dependencyTrigger.value++
}

// 4. 复杂计算属性：购物车示例
const cartItems = reactive([
  { id: 1, name: 'iPhone 14', price: 6999, quantity: 1 },
  { id: 2, name: 'MacBook Pro', price: 15999, quantity: 1 },
  { id: 3, name: 'AirPods Pro', price: 1999, quantity: 2 }
])

// 计算总数量
const totalQuantity = computed(() => {
  return cartItems.reduce((total, item) => total + item.quantity, 0)
})

// 计算总价格
const totalPrice = computed(() => {
  const total = cartItems.reduce((sum, item) => {
    return sum + (item.price * item.quantity)
  }, 0)
  return total.toFixed(2)
})

// 计算优惠金额（示例：满10000减500）
const discountAmount = computed(() => {
  const total = parseFloat(totalPrice.value)
  if (total >= 10000) {
    return '500.00'
  } else if (total >= 5000) {
    return '200.00'
  }
  return '0.00'
})

// 计算最终价格
const finalPrice = computed(() => {
  const total = parseFloat(totalPrice.value)
  const discount = parseFloat(discountAmount.value)
  return (total - discount).toFixed(2)
})
</script>

<style scoped>
.computed-section {
  margin: 25px 0;
  padding: 20px;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  background-color: #fafafa;
}

.computed-section h3 {
  margin-top: 0;
  color: #42b883;
  border-bottom: 2px solid #42b883;
  padding-bottom: 10px;
}

.computed-section h4 {
  color: #666;
  margin: 15px 0 10px 0;
}

label {
  display: inline-block;
  width: 80px;
  font-weight: bold;
  margin-right: 10px;
}

input {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-bottom: 10px;
}

input:focus {
  outline: none;
  border-color: #42b883;
  box-shadow: 0 0 0 2px rgba(66, 184, 131, 0.2);
}

.cart-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-bottom: 10px;
  background-color: white;
}

.cart-summary {
  margin-top: 20px;
  padding: 15px;
  background-color: #e8f5e8;
  border-radius: 4px;
  border-left: 4px solid #42b883;
}

button {
  padding: 10px 15px;
  background-color: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin: 10px 0;
}

button:hover {
  background-color: #369870;
}

small {
  color: #666;
  font-style: italic;
}
</style>
```

### 8.2 watch和watchEffect详解

```vue
<template>
  <div>
    <h2>Vue3侦听器深入使用</h2>
    
    <!-- 基础watch使用 -->
    <div class="watch-section">
      <h3>基础watch侦听器</h3>
      <div>
        <label>用户名：</label>
        <input v-model="username" placeholder="输入用户名" />
      </div>
      <div>
        <label>邮箱：</label>
        <input v-model="email" placeholder="输入邮箱" />
      </div>
      <div class="log-display">
        <h4>侦听日志：</h4>
        <div v-for="(log, index) in watchLogs" :key="index" class="log-item">
          {{ log }}
        </div>
      </div>
    </div>
    
    <!-- watch多数据源 -->
    <div class="watch-section">
      <h3>侦听多个数据源</h3>
      <div>
        <label>年龄：</label>
        <input type="number" v-model.number="age" min="0" max="120" />
      </div>
      <div>
        <label>城市：</label>
        <select v-model="city">
          <option value="">请选择城市</option>
          <option value="北京">北京</option>
          <option value="上海">上海</option>
          <option value="广州">广州</option>
          <option value="深圳">深圳</option>
        </select>
      </div>
    </div>
    
    <!-- 深度侦听 -->
    <div class="watch-section">
      <h3>深度侦听对象</h3>
      <div>
        <h4>用户配置：</h4>
        <label>主题：</label>
        <select v-model="userSettings.theme">
          <option value="light">浅色主题</option>
          <option value="dark">深色主题</option>
        </select>
      </div>
      <div>
        <label>语言：</label>
        <select v-model="userSettings.language">
          <option value="zh">中文</option>
          <option value="en">English</option>
        </select>
      </div>
      <div>
        <label>字体大小：</label>
        <input type="range" v-model.number="userSettings.fontSize" min="12" max="24" />
        <span>{{ userSettings.fontSize }}px</span>
      </div>
    </div>
    
    <!-- watchEffect使用 -->
    <div class="watch-section">
      <h3>watchEffect自动依赖收集</h3>
      <div>
        <label>搜索关键词：</label>
        <input v-model="searchKeyword" placeholder="输入搜索关键词" />
      </div>
      <div>
        <label>搜索类型：</label>
        <select v-model="searchType">
          <option value="all">全部</option>
          <option value="user">用户</option>
          <option value="post">文章</option>
        </select>
      </div>
      <div class="search-result">
        <h4>搜索结果：</h4>
        <p>{{ searchResult }}</p>
      </div>
    </div>
    
    <!-- 异步watch -->
    <div class="watch-section">
      <h3>异步侦听器</h3>
      <div>
        <label>API URL：</label>
        <input v-model="apiUrl" placeholder="输入API地址" />
      </div>
      <div class="api-result">
        <h4>API响应：</h4>
        <p v-if="apiLoading">加载中...</p>
        <pre v-else-if="apiError" class="error">{{ apiError }}</pre>
        <pre v-else class="success">{{ apiResponse }}</pre>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, watch, watchEffect, nextTick } from 'vue'

// 基础数据
const username = ref('')
const email = ref('')
const age = ref(18)
const city = ref('')
const watchLogs = ref([])

// 用户设置对象
const userSettings = reactive({
  theme: 'light',
  language: 'zh',
  fontSize: 16
})

// 搜索相关
const searchKeyword = ref('')
const searchType = ref('all')
const searchResult = ref('')

// API相关
const apiUrl = ref('')
const apiResponse = ref('')
const apiLoading = ref(false)
const apiError = ref('')

// 1. 基础watch - 侦听单个数据源
watch(username, (newUsername, oldUsername) => {
  const log = `用户名从 "${oldUsername}" 变更为 "${newUsername}" - ${new Date().toLocaleTimeString()}`
  watchLogs.value.unshift(log)
  console.log(log)
  
  // 限制日志条数
  if (watchLogs.value.length > 10) {
    watchLogs.value = watchLogs.value.slice(0, 10)
  }
})

// 侦听email变化
watch(email, (newEmail, oldEmail) => {
  const log = `邮箱从 "${oldEmail}" 变更为 "${newEmail}" - ${new Date().toLocaleTimeString()}`
  watchLogs.value.unshift(log)
  
  // 邮箱格式验证示例
  if (newEmail && !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(newEmail)) {
    const errorLog = `邮箱格式不正确: ${newEmail}`
    watchLogs.value.unshift(errorLog)
  }
}, {
  // 立即执行一次
  immediate: false
})

// 2. 侦听多个数据源
watch([age, city], ([newAge, newCity], [oldAge, oldCity]) => {
  const log = `用户信息变更 - 年龄: ${oldAge} → ${newAge}, 城市: "${oldCity}" → "${newCity}"`
  watchLogs.value.unshift(log)
  console.log('多数据源侦听:', { age: newAge, city: newCity })
})

// 3. 深度侦听对象
watch(userSettings, (newSettings, oldSettings) => {
  console.log('用户设置发生变化:', newSettings)
  const log = `设置变更 - 主题: ${newSettings.theme}, 语言: ${newSettings.language}, 字体: ${newSettings.fontSize}px`
  watchLogs.value.unshift(log)
  
  // 模拟保存设置到localStorage
  localStorage.setItem('userSettings', JSON.stringify(newSettings))
}, {
  deep: true, // 深度侦听
  immediate: true // 立即执行一次
})

// 4. watchEffect - 自动收集依赖
watchEffect(() => {
  // 这个函数中使用的所有响应式数据都会被自动侦听
  if (searchKeyword.value && searchType.value) {
    searchResult.value = `搜索 "${searchKeyword.value}" 在 "${searchType.value}" 类型中的结果...`
    
    // 模拟搜索API调用
    console.log(`执行搜索: 关键词="${searchKeyword.value}", 类型="${searchType.value}"`)
  } else {
    searchResult.value = '请输入搜索关键词和选择类型'
  }
})

// 5. 异步侦听器
watch(apiUrl, async (newUrl) => {
  if (!newUrl) {
    apiResponse.value = ''
    apiError.value = ''
    return
  }
  
  apiLoading.value = true
  apiError.value = ''
  
  try {
    console.log('开始请求API:', newUrl)
    
    // 模拟API请求（实际项目中使用fetch或axios）
    const response = await new Promise((resolve, reject) => {
      setTimeout(() => {
        if (newUrl.includes('error')) {
          reject(new Error('API请求失败'))
        } else {
          resolve({
            data: { message: 'API请求成功', url: newUrl, timestamp: Date.now() },
            status: 200
          })
        }
      }, 1000)
    })
    
    apiResponse.value = JSON.stringify(response, null, 2)
  } catch (error) {
    apiError.value = error.message
    console.error('API请求错误:', error)
  } finally {
    apiLoading.value = false
  }
}, {
  // 防抖处理，避免频繁请求
  flush: 'post' // 在组件更新后执行
})

// 6. 侦听器的停止和清理
const stopWatcher = watchEffect(() => {
  // 某些需要手动停止的侦听逻辑
  console.log('这个侦听器可以被手动停止')
})

// 可以调用返回的函数来停止侦听
// stopWatcher()

// 清理函数示例
watchEffect((onCleanup) => {
  const timer = setTimeout(() => {
    console.log('定时器执行')
  }, 1000)
  
  // 清理函数：在侦听器重新执行或组件卸载时调用
  onCleanup(() => {
    clearTimeout(timer)
    console.log('清理定时器')
  })
})
</script>

<style scoped>
.watch-section {
  margin: 25px 0;
  padding: 20px;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  background-color: #fafafa;
}

.watch-section h3 {
  margin-top: 0;
  color: #42b883;
  border-bottom: 2px solid #42b883;
  padding-bottom: 10px;
}

.watch-section h4 {
  color: #666;
  margin: 15px 0 10px 0;
}

label {
  display: inline-block;
  width: 100px;
  font-weight: bold;
  margin-right: 10px;
}

input, select {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-bottom: 10px;
  margin-right: 10px;
}

input:focus, select:focus {
  outline: none;
  border-color: #42b883;
  box-shadow: 0 0 0 2px rgba(66, 184, 131, 0.2);
}

.log-display {
  margin-top: 15px;
  max-height: 200px;
  overflow-y: auto;
  background-color: #f5f5f5;
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 10px;
}

.log-item {
  font-size: 12px;
  padding: 2px 0;
  border-bottom: 1px solid #eee;
  color: #666;
}

.search-result, .api-result {
  margin-top: 15px;
  padding: 10px;
  background-color: #f0f8ff;
  border-radius: 4px;
  border-left: 4px solid #42b883;
}

pre {
  background-color: #f5f5f5;
  padding: 10px;
  border-radius: 4px;
  overflow-x: auto;
  font-size: 12px;
  line-height: 1.4;
}

.error {
  background-color: #ffe6e6;
  color: #d63384;
  border-left: 4px solid #d63384;
}

.success {
  background-color: #e6f7e6;
  color: #198754;
  border-left: 4px solid #198754;
}
</style>