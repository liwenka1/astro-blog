---
author: 李文凯
pubDatetime: 2023-08-28T20:43:24+08:00
title: 从零开始的Web端React音乐播放器（二）
postSlug: react-music-02
featured: true
draft: false
tags:
  - react-router@6 + zustand + axios + react-query 
  - tailwindcss + shadcn/ui
ogImage: ""
description:
  将一些常用的插件进行安装并进行简单的配置(react-router@6 + zustand + axios + react-query + tailwindcss + shadcn/ui)
canonicalURL: ""
---
## 前言
上一节我们通过vite创建了项目工程，这一节我们将一些常用的插件进行安装并进行简单的配置
### 项目地址
前端地址：[react-music](https://github.com/liwenka1/react-music)
后端地址：[NeteaseCloudMusicApi](https://github.com/Binaryify/NeteaseCloudMusicApi)
## react-router@6
安装依赖
```bash
pnpm install react-router-dom
```
在 src 目录下新建 pages 和 router 文件夹
在 pages 下新增页面级组件 Home/index.tsx, About/index.tsx 
```tsx
import { Link } from 'react-router-dom'

const Home = () => {
  return (
    <div>
      这里是Home
      <Link to="/about">去about</Link>
    </div>
  )
}

export default Home
```
```tsx
import { Link } from 'react-router-dom'

const About = () => {
  return (
    <div>
      这里是About
      <Link to="/home">去Home</Link>
    </div>
  )
}

export default About
```
在 router 目录下创建 index.tsx 
```tsx
import { createBrowserRouter, Navigate } from 'react-router-dom'
import type { RouteObject } from 'react-router-dom'
import Home from '@/pages/Home'
import About from '@/pages/About'

const routes: RouteObject[] = [
  {
    path: '/',
    children: [
      {
        index: true,
        element: <Navigate to="/home" replace />,
      },
      {
        path: 'home',
        element: <Home />,
      },
      {
        path: 'about',
        element: <About />,
      },
    ],
  },
]

export default createBrowserRouter(routes, {
  basename: '/',
})
```
修改 src/App.tsx
```tsx
import { RouterProvider } from 'react-router-dom'
import router from './router'

const App = () => {
  return <RouterProvider router={router} />
}

export default App
```
router 目录下新建 lazyLoad.tsx 实现组件懒加载
```tsx
import { Suspense } from 'react'

const lazyLoad = (Component: React.LazyExoticComponent<() => JSX.Element>) => {
  return (
    <Suspense>
      <Component />
    </Suspense>
  )
}

export default lazyLoad
```
修改 router/index.tsx
```tsx
import { lazy } from 'react'
import { createBrowserRouter, Navigate } from 'react-router-dom'
import type { RouteObject } from 'react-router-dom'
import lazyLoad from './lazyLoad'

const Home = lazy(() => import('@/pages/Home'))
const About = lazy(() => import('@/pages/About'))

const routes: RouteObject[] = [
  {
    path: '/',
    children: [
      {
        index: true,
        element: <Navigate to="/home" replace />
      },
      {
        path: 'home',
        element: lazyLoad(Home)
      },
      {
        path: 'about',
        element: lazyLoad(About)
      }
    ]
  }
]

export default createBrowserRouter(routes, {
  basename: '/'
})
```
## zustand
安装依赖
```bash
pnpm install zustand
```
在 src 目录下新建 stores 文件夹并添加 counter.ts
```typescript
import { create } from 'zustand'

interface CounterState {
  counter: number
  increase: (cnt: number) => void
}

const useCounterStore = create<CounterState>()((set) => ({
  counter: 0,
  increase: (cnt) => set((state) => ({ counter: state.counter + cnt }))
}))

export default useCounterStore
```
在Home组件中进行使用
```tsx
import { Link } from 'react-router-dom'
import useCounterStore from '@/stores/counter'

const Home = () => {
  const counter = useCounterStore((state) => state.counter)
  const increase = useCounterStore((state) => state.increase)

  return (
    <div>
      这里是Home
      <Link to="/about">去about</Link>
      <button onClick={() => increase(1)}> counter: {counter} </button>
    </div>
  )
}

export default Home
```
## axios
安装依赖
```bash
pnpm install axios
```
在 src 目录下新建 utils 文件夹并添加 request.ts 并对 axios 进行简单封装（后续根据需求按需更改）
```typescript
import axios, { InternalAxiosRequestConfig, AxiosResponse } from 'axios'

// 创建 axios 实例
const service = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  maxBodyLength: 5 * 1024 * 1024,
  withCredentials: true,
  timeout: 50000
})

// 请求拦截器
service.interceptors.request.use(
  (config: InternalAxiosRequestConfig) => {
    config.params = {
      ...config.params,
      t: Date.now()
    }
    return config
  },
  (error: unknown) => {
    return Promise.reject(error)
  }
)

// 响应拦截器
service.interceptors.response.use(
  (response: AxiosResponse) => {
    return response
  },
  (error: unknown) => {
    return Promise.reject(error)
  }
)

interface Http {
  get<T>(url: string, params?: unknown): Promise<T>

  post<T>(url: string, params?: unknown): Promise<T>
}

const http: Http = {
  get(url, params) {
    return new Promise((resolve, reject) => {
      service
        .get(url, { params })
        .then((res) => {
          resolve(res.data)
        })
        .catch((err) => {
          reject(err.data)
        })
    })
  },

  post(url, params) {
    return new Promise((resolve, reject) => {
      service
        .post(url, JSON.stringify(params))
        .then((res) => {
          resolve(res.data)
        })
        .catch((err) => {
          reject(err.data)
        })
    })
  }
}

// 导出 axios 实例
export default http

```
## .env
在 src 目录下新建 .env.development 和 .env.production
```typescript
// .env.development
VITE_APP_ENV = 'development'
VITE_BASE_URL = '/'
VITE_API_URL = '/'
```
```typescript
// .env.production
VITE_APP_ENV = 'production'
VITE_BASE_URL = '/'
VITE_API_URL = '/'
```
修改 router/index.tsx 以及 utils/request.ts
```tsx
export default createBrowserRouter(routes, {
  basename: import.meta.env.VITE_BASE_URL
})
```
```typescript
const service = axios.create({
  baseURL: import.meta.env.VITE_APP_BASE_API,
  timeout: 50000
})
```
## react-query
安装依赖
```bash
pnpm install react-query
```
这里简单举个例子，具体使用后面用到在讲解
```tsx
 const [list, setList] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    setLoading(true);
    fetchList().then(res => {
      setLoading(false);
      setList(res?.data)
    }).catch(err => setLoading(false))
  }, []);
```
```tsx
 const { isLoading, error, data } = useQuery('getListData', () => fetchList().then(res => res.data));
```
这两部分的代码是等效的，我们不难看出 react-query 的好处
## tailwindcss + shadcn/ui
tailwincss 是一个实用且高度可定制的 CSS 框架。它让开发者通过简单地添加类名来轻松创建任何样式，无需编写自定义 CSS。与其他 CSS 框架相比，Tailwindcss 更加注重可定制性，因此可以更好地满足特定项目的需要。
shadcn/ui是一个使用 Radix UI 和 Tailwind CSS 构建的可重用组件库。具有优秀的设计和良好的用户体验。它提供了许多实用的组件，如日期选择器、分页控件等，而且易于使用并且高度可定制。
简单来说，我们可以更加轻松且自由的来开发我们的页面，并且不再过于的关注 ui 组件库，因为我们本身还是在书写 css
### tailwincss
安装依赖
```bash
pnpm install tailwindcss postcss autoprefixer -D 
npx tailwindcss init -p
```
修改 tailwind.config.js
```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: ['./src/**/*.{ts,tsx}'],
  theme: {
    extend: {}
  },
  plugins: []
}
```
修改 src/index.css 并在 src/main.ts 引入
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```
```typescript
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.tsx'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)
```
### **shadcn/ui**
这个ui库非常特殊，他的本质就是代码片段，和我们平常使用的方法大有不同，按照我下面的操作进行即可
利用 shadcn-uiinit 命令来设置项目
```bash
npx shadcn-ui@latest init
```
配置components.json
```bash
Would you like to use TypeScript (recommended)? no / yes
Which style would you like to use? › Default
Which color would you like to use as base color? › Slate
Where is your global CSS file? › › src/index.css
Do you want to use CSS variables for colors? › no / yes
Where is your tailwind.config.js located? › tailwind.config.js
Configure the import alias for components: › @/components
Configure the import alias for utils: › @/lib/utils
Are you using React Server Components? › no / yes (no)
```
添加组件使用，这里我们用 switch 组件实例 并且尝试添加一些 tailwindcss 的样式
```bash
npx shadcn-ui@latest add switch
```
这时候他会自动的去生成我们需要的组件到 src/components/ui 里面，我们引入使用即可
修改我们的 home 组件
```tsx
import { Link } from 'react-router-dom'
import useCounterStore from '@/stores/counter'
import { Switch } from '@/components/ui/switch'

const Home = () => {
  const counter = useCounterStore((state) => state.counter)
  const increase = useCounterStore((state) => state.increase)

  return (
    <div className="bg-pink-900 h-96">
      这里是Home
      <span className="text-lg">123</span>
      <Link to="/about">去about</Link>
      <Switch />
      <button onClick={() => increase(1)}> counter: {counter} </button>
    </div>
  )
}

export default Home
```
如果出现 'className' is missing in props validation 这个报错
只需要在 .eslintrc.cjs 添加 rules 相关修改即可
```javascript
  rules: {
    'react/prop-types': 'off'
  },
```
