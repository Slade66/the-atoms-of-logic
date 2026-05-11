
# Vue Router

作用：控制“页面跳转”（组件切换，不刷新页面）

访问 /about 渲染 About.vue 页面

#### 引入

代码示例：
```js
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

createApp(App)
  .use(router)   // 👈 关键
  .mount('#app')
```

```js
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'
import Home from '../views/Home.vue'
import About from '../views/About.vue'

const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```


#### `<router-view>`

作用：显示“当前路由对应的页面组件”

router-view = “页面内容显示的位置”

一次跳转发生了什么：
1. 用户访问 /about
2. vue-router 匹配 routes
3. 找到 About.vue
4. 把它渲染到 <router-view>

<template>
  <router-view />
</template>

它就是一个“占位符”：
| URL    | router-view 显示 |
| ------ | -------------- |
| /      | Home.vue       |
| /about | About.vue      |


**工作原理：**

路由配置
const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About }
]

访问 /about

<router-view>
  👉 渲染 About.vue
</router-view>

#### createWebHistory

作用：让 URL 变成“正常路径”（没有 #），更像真实网站

❌ 不用它（hash 模式）
http://localhost:5173/#/about

✅ 用 createWebHistory（history 模式）
http://localhost:5173/about

注意：history 模式需要后端支持（否则刷新 404），所有路径都返回 index.html

#### router-link

作用：写页面跳转（类似 a 标签，但不会刷新页面）

```html
<router-link to="/">首页</router-link>
<router-link to="/about">关于</router-link>
```
点击 “关于”，页面切换到 /about。

#### 在 JS 里控制跳转

```js
import { useRouter } from 'vue-router'

const router = useRouter()

function go() {
  router.push('/about')
}
```

调用 go()，跳转到 /about。

#### URL 路径参数

```js
// router
{ path: '/user/:id', component: User }
```

```html
<!-- User.vue -->
<script setup>
import { useRoute } from 'vue-router'

const route = useRoute()
console.log(route.params.id)
</script>
```

访问 /user/100，打印 100。

`router.push({ name: 'user', params: { id: 1 } })`

route.params.id 也可以拿到 1

#### query 传参

?id=1

router.push({ path: '/user', query: { id: 1 } })

跳转 /user?id=1，route.query.id 拿到 1

#### 全局前置导航守卫（beforeEach）

作用：在进入页面之前做“拦截判断”，决定能不能访问这个页面（比如是否登录）


router.beforeEach((to, from, next) => {
  const token = localStorage.getItem('token')

  if (to.path === '/order' && !token) {
    next('/login') // 未登录 → 去登录
  } else {
    next() // 放行
  }
})

to：你要去哪个页面（目标页面）
from：你从哪个页面来
next：控制是否放行（必须调用）

### useRouter

useRouter() 返回的是 整个路由实例。你可以把它想象成导航控制器。

找 Router 去干活（跳转）

router.push('/home')	

### useRoute 

它包含了当前路由 URL 的所有信息的对象

useRoute 找 Route 要资料（参数）。

常用属性：path（路径）、params（动态参数）、query（查询参数）、name（路由名称）等。

它用来读取 URL 里的 查询参数。例如，用户如果从购物车被拦截到登录页，跳转后的 URL 可能是 /login?redirect=/cart。useRoute 就能拿到这个 /cart，让程序知道登录完该带用户回哪去。
