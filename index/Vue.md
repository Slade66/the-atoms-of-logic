
### Vue 有什么用？

Vue 是用来构建用户界面的 JavaScript 框架。

没有 Vue 之前，你得手动操作页面元素（DOM）：找元素、改内容。

有了 Vue 之后，你只管改数据的逻辑和把数据插在页面哪个位置，数据更新后的页面更新就交给 Vue 来做。

Vue = “前端的模板 + 状态管理系统”。
你写数据和页面怎么对应、用户操作时数据怎么变。
Vue 负责：跟踪数据变化，更新页面。

### 声明式

你直接描述数据插在页面的这个地方，而不是命令式的操作 DOM 给元素赋值。

### 响应式（Reactivity）

数据变了，页面上依赖这个数据的地方自动跟着变。

Vue 会自动跟踪 JavaScript 状态的变化，并在状态发生变化时，把受影响的部分重新计算并更新 DOM。

### 单文件组件（Single-File Components）

单文件组件是一个 .vue 文件，把同属于一个组件的 HTML 模板、CSS 样式和 JavaScript 逻辑封装在了一个文件中。

页面可以拆成很多可复用的小组件。

结构清晰：一个组件的模板、逻辑、样式都在一起。
易于复用：写好的组件可以在很多地方重复使用。
易于维护：相关的 HTML、CSS、JavaScript 都在一个文件里，找问题时不用找来找去，在多个文件来回跳。

### 插值表达式

{{ count }}：表示把 count 的值渲染到页面中。

### setup()

组件的初始化逻辑写在这里。

### `<script setup>`



### createApp

创建一个 Vue 应用

### ref

创建一个响应式数据。

ref(0) 表示创建一个初始值为 0 的响应式数据

ref 会返回一个包裹对象，并在 .value 属性下暴露内部值。

```javascript
import { ref } from 'vue'

const message = ref('Hello World!')

console.log(message.value) // "Hello World!"
message.value = 'Changed'
```

在模板中访问的 ref 时不需要使用 .value：它会被自动解包，让使用更简单。

### @click="count++"

点击事件：“点击时执行这段 JS”。

### scoped

“组件级样式隔离”：这个样式只作用于当前组件，不会影响别的组件。

### Vue 组件的两种 API 风格

这两种 API 风格都完全能实现同样功能，它们只是建立在同一个底层系统之上的两种不同接口，只是“写代码的组织方式不同”

#### Options API

使用 Options API 时，我们通过一个“选项对象”来定义组件逻辑，比如 data、methods 和 mounted。
由这些选项定义出来的属性，会在函数内部通过 this 暴露出来，而 this 指向当前组件实例。

实际上，Options API 本身就是在 Composition API 的基础上实现的！

Options API 以“组件实例”为中心（也就是例子里的 this），这种方式通常更符合来自面向对象语言背景的开发者的思维模型。

按“功能类别”分区组织代码。
比如：
数据写在 data
方法写在 methods
生命周期写在 mounted
数据、方法、生命周期分栏摆放。

容易理解：“数据区”“方法区”“生命周期区”分得很清楚。
很规整：代码结构很固定，新手容易找位置。

同一个功能的代码，可能分散在不同选项里。

```html
<script>
export default {
  // data() 返回的属性会成为响应式状态
  // 并且会暴露到 `this` 上。
  data() {
    return {
      count: 0
    }
  },

  // methods 中定义的方法用于修改状态并触发更新。
  // 它们也可以在模板中绑定为事件处理函数。
  methods: {
    increment() {
      this.count++
    }
  },

  // 生命周期钩子会在组件生命周期的不同阶段被调用。
  // 这个函数会在组件挂载完成时执行。
  mounted() {
    console.log(`The initial count is ${this.count}.`)
  }
}
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

#### Composition API

一个组件的功能一多，如果都按 Options API 分到 data / methods / computed / watch / mounted 里，相关逻辑会被拆得比较散。而 Composition API 可以把每块逻辑按功能收拢起来，甚至抽成独立函数复用。

使用 Composition API 时，我们通过导入的 API 函数来定义组件逻辑。

“在一个函数作用域里，自由组织你的逻辑”。你可以把这些相关逻辑（功能相关的代码）直接放一起，所以它更适合复杂业务。

更灵活：你可以把同一个功能相关的状态、方法、生命周期写在一起。

“按功能组织逻辑”

```html
<script setup>
import { ref, onMounted } from 'vue'

// 响应式状态
const count = ref(0)

// 修改状态并触发更新的函数
function increment() {
  count.value++
}

// 生命周期钩子
onMounted(() => {
  console.log(`The initial count is ${count.value}.`)
})
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```
方法直接写成普通函数。
没有 this，而是直接操作变量本身。

### 如何在本地快速创建一个 Vue 单页应用

官方推荐用 create-vue 创建项目，底层构建工具是 Vite。

create-vue 是一个脚手架式创建项目骨架的工具，它帮你自动生成：
目录结构
配置文件
入口文件
你不用自己从 0 开始手写。

`pnpm create vue@latest`

这条命令会安装并执行 create-vue，它是 Vue 官方的项目脚手架工具。随后你会看到一系列交互式提问，用来选择一些可选功能，按你的选项生成一套项目模板：

项目名称、是否使用 TypeScript、JSX、Vue Router、Pinia、ESLint、Vitest、E2E、Prettier

项目创建完成后，进入项目文件夹安装依赖并启动开发服务器：
cd test
pnpm install
pnpm dev

当你准备把应用发布到生产环境时，运行：
pnpm run build

这条命令会在项目的 ./dist 目录中生成一个可用于生产环境部署的构建结果。
执行后会生成 dist 目录，里面就是部署用的静态资源文件。
里面通常是：

压缩后的 JS
压缩后的 CSS
打包后的 HTML
构建后的资源文件

把这个目录部署到服务器或静态托管平台，就能上线。

### npm create vue@latest 到底干了什么？

这句命令的作用是：下载并执行 Vue 官方脚手架 create-vue。

下载并执行 create-vue 脚手架。

本质是执行 npm 上的脚手架 CLI，通过交互收集配置，然后基于模板生成项目结构。

临时下载 create-vue 这个 npm 包，执行它里面的 CLI 程序。

**为什么没有写 create-vue 却执行了它？**

npm 内部做了一个命名约定：当你写 npm create xxx 时 👉 实际执行的是：npm exec create-xxx，本质就是执行 create-xxx 这个 npm 包。

npm 自动补全 → 自动下载 → 自动执行

执行流程：
npm create vue@latest
        ↓
解析成 create-vue@latest
        ↓
临时下载这个 npm 包
        ↓
执行 package.json 里的 bin 命令
        ↓
启动脚手架 CLI

### reactive 

声明响应式状态。

reactive() 只适用于对象 (包括数组和内置类型，如 Map 和 Set)。而另一个 API ref() 则可以接受任何值类型。

```javascript
import { reactive } from 'vue'

const counter = reactive({
  count: 0
})

console.log(counter.count) // 0
counter.count++
```

### 模板

双花括号中的内容可以是 JavaScript 表达式。

### v-bind

把 Vue 里的数据动态绑定到 HTML 标签属性上，让属性值不用写死。

给 HTML 标签绑定一个动态的属性值，让 HTML 标签属性的值动态地根据 Vue 里的数据变化而变化。

把 JS 变量的值，塞到标签属性里。

值是可以访问组件状态的 JavaScript 表达式。

**简写语法：**
v-bind:属性名="变量" 可以简写成: 属性名="变量"
<img :src="imageUrl"> 等价于 <img v-bind:src="imageUrl">

### v-on

设置 DOM 事件处理函数。

v-on:事件名="函数名" 可以简写成 @事件名="函数名"
<button @click="increment">Count is: {{ count }}</button> 等价于 <button v-on:click="increment">Count is: {{ count }}</button>

在函数中，我们可以通过修改 ref 来更新组件状态，从而修改 DOM。

v-on 处理函数会接收原生 DOM 事件作为其参数。

### v-model

把表单的值，绑定到变量上，让输入框和数据“自动双向绑定”，改 UI = 改数据

v-model 绑定什么，取决于 表单类型
| 元素类型       | 绑定的是什么       |
| ---------- | ------------ |
| input text | 文本内容         |
| checkbox   | true / false |
| radio      | 选中的值         |
| select     | 选中的 option   |


v-model 会将被绑定的值与 <input> 的值自动同步，这样我们就不必再使用 v-bind 和 v-on 事件处理函数了（修改元素的值触发函数，函数把修改后的值设置到变量，变量再把值同步到页面的元素），它实际上是上述操作的语法糖。

v-model 不仅支持文本输入框，也支持诸如多选框、单选框、下拉框之类的输入类型。

### v-if

有条件地渲染元素。

当变量为 true 才渲染。为 false 从页面 DOM 移除。

v-else 和 v-else-if 表示其他的条件分支。

### v-for

循环数组，渲染元素。

```javascript
<ul>
  <li v-for="todo in todos" :key="todo.id">
    {{ todo.text }}
  </li>
</ul>
```

带索引：
```javascript
<li v-for="(item, index) in teaList" :key="index">
  {{ index }} - {{ item }}
</li>
```

一定要写 :key

局部变量表示当前正在迭代的数组元素。它只能在 v-for 所绑定的元素上或是其内部访问，就像函数的作用域一样。

### js 数组删除元素

arr.filter(item => item != deleteItem)

### computed()

根据数据算新数据

创建一个计算属性 ref，这个 ref 的值不是固定的，而是会动态地根据其他响应式数据源来计算其 .value。

计算属性会自动跟踪其计算中所使用的到的其他响应式状态，并将它们收集为自己的依赖。计算结果会被缓存，并只有在其依赖发生改变时才会被自动更新。

### onMounted()

在挂载之后执行代码。

生命周期钩子——它允许我们注册一个在组件的特定生命周期调用的回调函数。还有一些其他的钩子如 onUpdated 和 onUnmounted。

### 模板引用（Template Ref）


给 DOM 起名字，然后直接拿到页面上的 DOM，用 JS 去操作它。

1. 给元素起名：ref="xxx"
2. JS 里声明一个同名的 ref 变量名，值是 ref(null)

模板引用只在组件挂载后才能用。因为当 `<script setup>` 执行时，DOM 元素还不存在。

### watch

监听一个变量的数据变化，一旦变了就执行你写的函数。

```javascript
import { ref, watch } from 'vue'

const count = ref(0)

// 侦听 count 的变化
watch(count, (newValue, oldValue) => {
  console.log(`count 从 ${oldValue} 变成了 ${newValue}`)
})
```

也可以只接一个值：
```html
<template>
  <input v-model="keyword">
</template>

<script setup>
import { ref, watch } from 'vue'

const keyword = ref('')

watch(keyword, (newVal) => {
  console.log('发请求搜索:', newVal)
})
</script>
```

默认监听不到对象内部变化,必须加：{ deep: true }

立即执行（常用）：{ immediate: true} 页面一加载就执行一次

### Props

父组件给子组件传数据

子组件可以通过 props 从父组件接受动态数据。

子组件：
首先，需要声明它所接受的 props：
```html
<template>
  <h1>{{ title }}</h1>
</template>

<script setup>
defineProps({
  title: String
})
</script>
```

父组件：
```html
<template>
  <Child :title="teaName" />
</template>

<script setup>
import Child from './Child.vue'

const teaName = '龙井茶'
</script>
```

defineProps() 是一个编译时宏，并不需要导入。

父组件可以像声明 HTML attributes 一样传递 props。若要传递动态值，也可以使用 v-bind 语法:
<ChildComp :msg="greeting" /> 

多个 props:
<Child :name="tea.name" :price="tea.price" />

单向数据流：Props 是 只读的

### Emits

子组件把数据“通知”给父组件（触发事件）

子组件：
```html
<script setup>
// 声明触发的事件
const emit = defineEmits(['response'])

// 带参数触发
emit('response', 'hello from child')
</script>
```

emit() 的第一个参数是事件的名称。其他所有参数都将传递给事件监听器。

父组件可以使用 v-on 监听子组件触发的事件——这里的处理函数接收子组件触发事件时的额外参数

```html
<ChildComp @response="(msg) => childMsg = msg" />
```

### slots（插槽）

子组件挖坑，父组件填内容

让父组件可以往子组件“指定位置塞内容”

让组件更灵活，不写死内容，由使用组件的人决定显示什么

子组件用 <slot></slot>，“这里留个坑，等父组件来填”。父组件在子组件标签包括的地方塞的东西会传到这里来。

1️⃣ 子组件（定义插槽）
<!-- MyCard.vue -->
<template>
  <div class="card">
    <h3>标题</h3>
    <slot></slot> <!-- 插槽位置 -->
  </div>
</template>
2️⃣ 父组件（往插槽塞内容）
<template>
  <MyCard>
    <p>这里是内容</p>
  </MyCard>
</template>
3️⃣ 渲染结果（页面看到）
<div class="card">
  <h3>标题</h3>
  <p>这里是内容</p>
</div>

#### 具名插槽


[[Vue Router]]

### 开 TUN 无法访问前端 Vite

vite --host


