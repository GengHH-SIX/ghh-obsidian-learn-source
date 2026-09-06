基于 Vite + Vue3 + TypeScript 创建 SSR 项目（`create ssr-vue-ts`），项目结构会同时包含客户端和服务端渲染所需的文件。下面详细介绍主要文件的作用和整个运行流程。
 
---
  官网文献参考:
  - vue ssr [[https://cn.vuejs.org/guide/scaling-up/ssr.html]]
  - vite ssr [[https://cn.vitejs.dev/guide/ssr]]

### 一、项目核心文件及作用

  1. `index.html` — HTML 模板

```html
<!DOCTYPE html>

<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Vite SSR App</title>
</head>

<body>
  <div id="app"><!--ssr-outlet--></div>
  <script type="module" src="/src/entry-client.ts"></script>
</body>

</html>

```

  作用：

- 作为服务端渲染的 HTML 模板

- `<!--ssr-outlet-->` 是占位符，服务端渲染的 HTML 字符串会替换此处

- 引入客户端入口文件 `entry-client.ts`，用于浏览器端激活应用

  ---

  

2. `src/main.ts` — 应用工厂函数

```typescript

import { createSSRApp } from 'vue'
import App from './App.vue'
import { createRouter } from './router'

  
export function createApp() {

  const app = createSSRApp(App)
  const router = createRouter()
  app.use(router)
  return { app, router }
}

```

  作用：

- 使用 `createSSRApp` 替代 `createApp`，创建支持 SSR 的应用实例

- 导出工厂函数而非直接挂载，确保每个请求都能创建独立的应用实例，避免请求间状态污染

- 集中注册路由、状态管理等插件
  
---
  

3. `src/entry-client.ts` — ==客户端入口==
 
```typescript

import { createApp } from './main'

const { app, router } = createApp()

  
router.isReady().then(() => {
  app.mount('app', true)  // 第二个参数 true 表示 hydrate（水合）

})

```  

作用：

- 浏览器端执行的入口文件

- 等待路由就绪后，将应用挂载到 `app` 节点

- `mount('app', true)` 中的 `true` 表示激活水合过程，将服务端生成的静态 HTML 转换为可交互的 Vue 应用  

---

  4. `src/entry-server.ts` — ==服务端入口==
  

```typescript

import { renderToString } from 'vue/server-renderer'
import { createApp } from './main'

export async function render(url: string) {

  const { app, router } = createApp()
  await router.push(url)
  await router.isReady()


  const ctx = {}
  const html = await renderToString(app, ctx)
  
  return { html }
}
```

  
作用：

- 服务端执行的入口文件，导出 `render` 函数

- 接收请求 URL，设置路由匹配

- 调用 `renderToString` 将 Vue 组件渲染为 HTML 字符串

- 返回渲染结果供 Node 服务器使用

  ---

  
5. `server.js` — Node 服务器

```javascript
import fs from 'node:fs/promises'

import express from 'express'

import { createServer as createViteServer } from 'vite'


const port = process.env.PORT || 5173
const base = process.env.BASE || '/'


async function createServer() {

  const app = express()

  // 以中间件模式创建 Vite 开发服务器

  const vite = await createViteServer({
    server: { middlewareMode: true },
    appType: 'custom'
  })

  app.use(vite.middlewares)
  

  // 处理所有页面请求
  app.use('*', async (req, res) => {
    try {
      const url = req.originalUrl
  
      // 1. 读取 HTML 模板
      let template = await fs.readFile('index.html', 'utf-8')

      // 2. 应用 Vite HTML 转换（注入热更新脚本等）
      template = await vite.transformIndexHtml(url, template)

      // 3. 加载服务端入口模块
      const { render } = await vite.ssrLoadModule('/src/entry-server.ts')

      // 4. 渲染应用 HTML
      const { html: appHtml } = await render(url)

      // 5. 替换占位符
      const html = template.replace('<!--ssr-outlet-->', appHtml)

      // 6. 返回完整 HTML
      res.status(200).set({ 'Content-Type': 'text/html' }).end(html)

    } catch (e) {
      vite.ssrFixStacktrace(e)
      next(e)
    }
  })

  
  app.listen(port, () => {
    console.log(`Server running at http://localhost:${port}`)
  })
}

  
createServer()

```

  
作用：

- 创建 Express 服务器，集成 Vite 开发中间件

- 拦截所有页面请求，执行 SSR 渲染流程

- 在开发模式下提供热更新支持

- 生产模式下可替换为预构建的静态资源服务

  
---

  
6. `src/App.vue` — 根组件
  
```vue
<template>

  <div id="app">
    <router-view />
  </div>

</template>
```

  
作用：

- 应用根组件，包含 `<router-view>` 用于渲染路由匹配的页面组件

- 服务端和客户端共享同一组件代码，实现同构

---

  
7. `src/router/index.ts` — 路由配置
  
```typescript

import { createRouter, createMemoryHistory, createWebHistory } from 'vue-router'
import Home from '../views/Home.vue'
import About from '../views/About.vue'

  
export function createRouter() {

  return createRouter({
    // 服务端使用 memory history，客户端使用 web history
    history: import.meta.env.SSR ? createMemoryHistory() : createWebHistory(),
    routes: [
      { path: '/', component: Home },
      { path: '/about', component: About }
    ]
  })
}

```

  

作用：

- 根据运行环境选择不同的路由模式：服务端用 `createMemoryHistory`，客户端用 `createWebHistory`

- 确保服务端和客户端路由状态一致，避免水合不匹配

  
---

### 二、项目运行完整流程
  
```

用户请求 → Node 服务器 → Vite 中间件 → 服务端渲染 → 返回 HTML → 浏览器激活

```
  
#### 第一步：启动服务

运行 `npm run dev`，Vite 启动开发服务器，同时 Express 监听指定端口（默认 5173）。

#### 第二步：接收请求

用户在浏览器访问 `http://localhost:5173`，请求到达 Express 服务器。

#### 第三步：读取并转换模板

服务器读取 `index.html`，通过 `vite.transformIndexHtml` 注入开发环境所需的脚本（如热更新客户端）。

#### 第四步：加载服务端入口

通过 `vite.ssrLoadModule('/src/entry-server.ts')` 动态加载服务端入口模块，Vite 会处理 TypeScript 编译和模块转换。

#### 第五步：路由匹配与渲染

`entry-server.ts` 中的 `render(url)` 函数：

1. 创建新的应用实例和路由实例

2. 调用 `router.push(url)` 匹配当前请求的路由

3. 等待路由异步组件加载完成

4. 调用 `renderToString(app)` 将组件树渲染为 HTML 字符串

  
#### 第六步：组装完整 HTML

  将渲染后的 HTML 字符串替换模板中的 `<!--ssr-outlet-->` 占位符，生成完整的 HTML 页面。

#### 第七步：返回响应

服务器将完整 HTML 返回给浏览器，浏览器立即显示首屏内容，无需等待 JavaScript 下载。
 

#### 第八步：客户端水合

浏览器解析 HTML 后加载 `entry-client.ts`：

1. 创建新的应用实例

2. 等待路由就绪

3. 调用 `app.mount('app', true)` 执行水合

4. Vue 将服务端生成的静态 DOM 与客户端虚拟 DOM 进行比对，绑定事件监听器，使页面变为可交互状态
  

---
  

### 三、关键设计要点

  
| 设计要点            | 说明                                                      |
| --------------- | ------------------------------------------------------- |
| ‌工厂函数模式‌        | createApp() 为每个请求创建独立实例，避免状态污染                          |
| ‌同构代码‌          | 组件代码在服务端和客户端共享，需避免使用 window、document 等浏览器专属 API         |
| ‌路由模式分离‌        | 服务端用 createMemoryHistory，客户端用 createWebHistory，确保路由状态一致 |
| ‌水合激活‌          | mount('#app', true) 的第二个参数启用水合，复用服务端 DOM 而非重新创建         |
| ‌Vite SSR 模块加载‌ | vite.ssrLoadModule 在开发模式下即时编译 TypeScript，无需预构建          |
  

整个流程的核心在于：服务端负责首屏 HTML 的快速呈现，客户端接管后续交互，两者通过水合机制无缝衔接，兼顾了 SEO 和用户体验。<br>
### 附件资料
- 参考资料<br>[1] [Vue SSR 实现原理详解_vue ssr原理-CSDN博客 - CSDN博客](https://blog.csdn.net/My_wife_QBL/article/details/140207262)<br>[2] [服务端渲染(SSR)的概念、原理及其在Vue项目中的高级配置和实战应用,包括Node.js环境搭建、服务器端数据预取、SEO优化策略等。 - CSDN博客](https://blog.csdn.net/dxh_888/article/details/145586444)<br>[3] [Vue服务端渲染技术深度解析与优化策略 - 百度智能云](https://cloud.baidu.com/article/3369719)<br>[4] [Vue&SSR详解 VueServerRenderer Nutx - CSDN博客](https://blog.csdn.net/m0_55049655/article/details/143382614)<br>[5] [vue创建项目以及各个文件的作用 - CSDN博客](https://blog.csdn.net/zm2674728427/article/details/143672626)<br>[6] [vue项目工程主文件之间的联系(项目组成逻辑) - CSDN博客](https://blog.csdn.net/qq_52335693/article/details/145477671)<br>[7] [Vue3中的SSR函数详解:实现服务器端渲染 - php中文网](https://www.php.cn/faq/561748.html)<br>[8] [基于Vue3 + TypeScript 开发SSR系统(一):初始创建SSR - CSDN博客](https://blog.csdn.net/qq_18813875/article/details/126872162)<br>[9] [基于Vue3 + TypeScript 开发SSR系统:初始创建SSR - 百度开发者中心](https://developer.baidu.com/article/detail.html?id=2836290)<br>[10] [简易实现ssr-CSDN博客 - CSDN博客](https://blog.csdn.net/2302_80473627/article/details/163279903)<br>[11] [Vue项目的SSR处理教程 - CSDN博客](https://blog.csdn.net/qq_44642822/article/details/147604434)<br>[12] [带你五步学会Vue SSR - 腾讯云](https://cloud.tencent.com/developer/article/2385451)<br>[13] [页面加载优化:架构选型、Tree-shaking 与代码分割 - 51CTO](https://www.51cto.com/article/851625.html)<br>[14] [Nuxt.js 框架,前端SSR项目方便SEO/GEO, 实践-CSDN博客 - CSDN博客](https://blog.csdn.net/bsklhao/article/details/163273583)<br>[15] [Vue 3 服务端渲染 (SSR) 实战指南 - CSDN博客](https://blog.csdn.net/blue_698/article/details/158463384)<br>[16] [SSR是什么?Vue中怎么实现? - CSDN博客](https://blog.csdn.net/He_9a9/article/details/134728495)<br>[17] [简述Vue SSR 的实现原理 ? - CSDN博客](https://blog.csdn.net/youhebuke225/article/details/139954140)<br>[18] [Vue.js服务端渲染优化: 使用Webpack进行SSR打包优化 - 简书社区](https://www.jianshu.com/p/28453697a3c5)<br>[19] [nuxt3+ts+vue3的ssr项目总结_vue3 ssr-CSDN博客 - CSDN博客](https://blog.csdn.net/qq_45799465/article/details/132590507)<br>[20] [Vue SSR服务器端渲染: 优化首屏加载性能与SEO - 简书社区](https://www.jianshu.com/p/e86f3240a897)<br>[21] [17-Vue3 服务端渲染与 Nuxt3 入门 - CSDN博客](https://blog.csdn.net/qq_39302722/article/details/162492563)<br>[22] [深挖vue3基本原理之五 —— 性能优化机制 - CSDN博客](https://blog.csdn.net/ZhooooYuChEnG/article/details/145564791)<br>[23] [教你使用 koa2 + vite + ts + vue3 + pinia 构建前端 SSR 企业级项目 - 搜狐](https://it.sohu.com/a/558042672_121124378)<br>[24] [手摸手搭建 Vue+TS+Express 全栈 SSR 博客系统——项目架构和技术选型篇 - 思否开发者社区](https://segmentfault.com/a/1190000043281316)<br>[25] [32岁,做了8年Web前端,建议大家别想太多 - 码农最前线](http://zhuanlan.zhihu.com/p/1996588810377635588)<br>[26] [Vue3项目选型:TS还是JS - 小程玩编程](http://mbd.baidu.com/newspage/data/dtlandingsuper?nid=dt_5158468362990194690)<br>[27] [Vue前端篇——项目目录结构介绍 - 腾讯云](https://cloud.tencent.com/developer/article/2433480)<br>[28] [Vue 3 极速上手:从环境搭建到实战避坑指南 - CSDN博客](https://blog.csdn.net/weixin_30768881/article/details/163317766)<br>[29] [vue项目中各个文档的含义以及作用 - CSDN博客](https://blog.csdn.net/Jclam/article/details/129333363)<br><br>百度AI生成，内容仅供参考




---


我们可以结合 Vue 官方 SSR 文档中对‌**激活（hydrate）过程**‌的描述，完整拆解这个按钮点击从无效到生效的底层原理：

---
### 扩展：Vue Ssr 部分原理
#### 一、水合之前的状态：服务端返回纯静态 HTML

在没有接入客户端代码前，服务端通过 `renderToString` 直接将 Vue 应用实例渲染成了纯字符串的 button 标签，返回给浏览器的页面是完全静态的。这个生成的 HTML 里只包含渲染后的最终 DOM 结构，没有任何 Vue 的运行时代码，也没有绑定任何事件监听器，所以用户点击按钮时，浏览器不会触发任何交互逻辑，自然不会出现数字自增的效果。

---

#### 二、水合的完整执行流程

接入客户端代码后，Vue 会在浏览器端执行完整的激活步骤，让静态的 HTML 变为可交互的应用，整个过程分为3个核心阶段：

##### 1. 客户端创建和服务端完全一致的应用实例

客户端入口文件不会直接用 `createApp`，而是和服务端一样调用 `createSSRApp()`，传入完全相同的组件配置、初始状态和模板代码：它会重新创建一个拥有相同初始 `count` 状态的 button 虚拟 DOM 树，这一步是水合的前提——保证客户端的虚拟 DOM 和服务端生成的真实 DOM 结构完全匹配。

##### 2. Vue 执行 DOM 匹配与节点关联

客户端不会像普通 SPA 那样直接清空原有页面、重新创建所有 DOM 节点，而是走专门的激活挂载逻辑：Vue 从根节点开始遍历已经存在的静态真实 DOM，同时遍历自己刚创建的客户端虚拟 DOM 树，将每一个虚拟节点和对应的真实 DOM 节点一一绑定关联起来。  
针对这个 button 的例子，Vue 会精准找到服务端渲染好的那个按钮 DOM，把它标记为当前组件的受控节点，不会重新生成一个新的按钮覆盖原有内容。

##### 3. 绑定事件监听器，注入交互逻辑

完成 DOM 节点关联后，Vue 会把组件模板中定义的 `@click="count++"` 事件处理器，绑定到刚才匹配到的真实 button DOM 节点上，同时把之前声明的响应式状态 `count` 和这个节点做双向关联：此时点击按钮，触发绑定好的点击回调，就能修改响应式的 count 数值，Vue 检测到状态变化后自动更新按钮上显示的文本，最终实现点击数字自增的效果。

---

#### 三、这个水合逻辑的核心优势

不同于普通 SPA 完全重写 DOM 的加载模式，SSR 的水合过程可以直接复用服务端已经渲染好的静态 HTML：用户打开页面就能立刻看到按钮内容，不需要等待客户端 JS 全部下载解析完成才能看到页面，既保留了 SSR 首屏秒开、SEO 友好的优势，又通过客户端激活给静态页面补上了完整的交互能力。