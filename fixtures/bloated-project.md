# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## 项目概述

这是一个用 Vue 3 构建的电商管理后台系统。项目帮助商家管理商品、订单、用户和营销活动。
系统采用前后端分离架构，前端负责 UI 展示和用户交互，后端提供 RESTful API。

## 技术栈

本项目使用以下技术：

- Vue 3.4.21 - 渐进式 JavaScript 框架，使用 Composition API
- Vue Router 4.2.5 - 官方路由管理器
- Pinia 2.1.7 - Vue 的状态管理库，替代 Vuex
- TypeScript 5.2.2 - JavaScript 的超集，添加静态类型检查
- Vite 5.1.4 - 下一代前端构建工具，基于 ES modules
- Element Plus 2.5.6 - 基于 Vue 3 的组件库
- Axios 1.6.2 - Promise based HTTP client
- Tailwind CSS 3.4.1 - 实用优先的 CSS 框架
- dayjs 1.11.10 - 轻量级日期处理库
- ECharts 5.4.3 - 数据可视化图表库
- lodash-es 4.17.21 - 实用工具函数库

## 项目结构

```
src/
├── main.ts           # 应用入口文件，创建 Vue 应用实例并挂载
├── App.vue           # 根组件，包含全局布局和 router-view
├── router/           # 路由配置目录
│   └── index.ts      # 路由定义，包含所有页面的路由规则和导航守卫
├── stores/           # Pinia 状态管理
│   ├── user.ts       # 用户状态管理，处理登录、用户信息、权限
│   ├── cart.ts       # 购物车状态，管理购物车商品列表
│   └── app.ts        # 应用全局状态，侧边栏折叠、主题等
├── views/            # 页面组件目录
│   ├── Home.vue      # 首页，展示数据概览仪表盘
│   ├── Login.vue     # 登录页，处理用户认证
│   ├── Products.vue  # 商品管理列表页
│   ├── Orders.vue    # 订单管理页
│   └── Users.vue     # 用户管理页
├── components/       # 可复用组件
│   ├── DataTable.vue # 通用数据表格组件
│   ├── SearchBar.vue # 搜索栏组件
│   └── Modal.vue     # 弹窗组件
├── api/              # API 请求层
│   ├── request.ts    # Axios 实例封装，统一错误处理
│   ├── product.ts    # 商品相关 API 请求
│   └── order.ts      # 订单相关 API 请求
├── utils/            # 工具函数
│   ├── format.ts     # 格式化工具（日期、金额、数字）
│   └── auth.ts       # 认证工具（token 存取）
├── assets/           # 静态资源
│   └── styles/       # 全局样式
└── types/            # TypeScript 类型定义
    └── api.d.ts      # API 响应类型定义
```

## 开发命令

```bash
# 安装依赖
npm install

# 启动开发服务器（默认端口 5173）
npm run dev

# 构建生产版本
npm run build

# 预览生产构建
npm run preview

# 代码检查
npm run lint
```

## 代码风格

- 必须使用 TypeScript 编写所有代码，充分发挥类型系统的优势
- 组件统一使用 `<script setup lang="ts">` 语法
- 使用 Composition API，不要使用 Options API
- 变量命名使用 camelCase，组件命名使用 PascalCase
- 写干净的、可维护的代码，保持代码整洁
- 每个函数应该只做一件事（单一职责原则）
- 处理好错误，不要忽略异常
- 写有意义的注释，解释为什么而不是什么

## Vue 组件开发指南

Vue 3 的 Composition API 是现在推荐的方式。使用 `<script setup>` 语法可以让你写出更简洁的组件代码。下面是组件开发的基本模式：

```vue
<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

// 定义 props
const props = defineProps<{
  title: string
  count?: number
}>()

// 响应式状态
const data = ref<string[]>([])

// 计算属性
const total = computed(() => data.value.length)

// 生命周期
onMounted(() => {
  fetchData()
})

// 方法
async function fetchData() {
  // 获取数据
}
</script>
```

Pinia 是 Vue 的官方状态管理库，它提供了更简洁的 API 和更好的 TypeScript 支持。定义 store 使用 `defineStore`：

```ts
export const useUserStore = defineStore('user', () => {
  const token = ref('')
  const userInfo = ref<UserInfo | null>(null)
  return { token, userInfo }
})
```

## API 请求规范

本项目使用 Axios 进行 HTTP 请求。所有请求都经过 `src/api/request.ts` 中封装的实例，该实例配置了：

- baseURL: 从环境变量读取 `VITE_API_BASE_URL`
- 请求拦截器：自动添加 Authorization header，从 localStorage 读取 token
- 响应拦截器：统一处理错误码，401 跳转登录页
- 超时时间：10000ms

Axios 的基本用法如下：`axios.get(url, config)` 发起 GET 请求，`axios.post(url, data, config)` 发起 POST 请求。请求返回 Promise 对象，可以用 `.then()` 或 `async/await` 处理。

## 环境变量

需要在项目根目录创建 `.env` 文件：

```
VITE_API_BASE_URL=https://api.example.com
VITE_APP_TITLE=电商管理后台
```

开发环境用 `.env.development`，生产环境用 `.env.production`。

## 重要约定

- 提交信息遵循 Conventional Commits 规范：`feat:` / `fix:` / `refactor:` 等
- 分支命名：`feature/xxx`、`fix/xxx`、`hotfix/xxx`
- PR 需要至少一个 reviewer 审批通过

## 注意事项

Element Plus 的 el-table 在大数据量下会卡顿，超过 500 行的数据请使用虚拟滚动。这个坑我们踩过好几次，不要再用普通表格渲染大量数据。

## 路由配置说明

路由使用 Vue Router 的 createRouter 创建。路由配置定义在 `src/router/index.ts` 中。每个路由对象包含 path、name、component 三个必填字段，以及 meta 可选字段用于存储路由元信息。导航守卫通过 `router.beforeEach` 注册，常用于权限校验。

## 数据可视化

图表使用 ECharts 渲染。ECharts 是一个功能强大的数据可视化库，支持折线图、柱状图、饼图等多种图表类型。使用时需要先初始化 echarts 实例，然后通过 setOption 方法设置配置项。配置项是一个复杂的树状结构对象，包含 series、xAxis、yAxis、tooltip 等字段。
