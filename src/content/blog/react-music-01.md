---
author: 李文凯
pubDatetime: 2023-08-05T22:58:24+08:00
title: 从零开始的Web端音乐播放器（一）
postSlug: react-music-01
featured: true
draft: false
tags:
  - React + Vite
  - ESLint + Prettier + Stylelint
  - Husky + lint-staged + commitlint
ogImage: ""
description:
  通过 Vite 创建项目，并且利用 ESLint + Prettier + Stylelint 来校验与格式化代码，Husky + lint-staged + commitlint 来对 git commit 来进行校验
canonicalURL: ""
---
<a name="sI8XM"></a>
## 前言
最近准备学习 React 无奈找不到一个很适合我这样低水平的来练手，于是乎准备自己写一个简单的项目，也是前端玩家都很熟悉的音乐播放器了，后端还是用的网易云的接口来做的<br />前端地址：[react-music](https://github.com/liwenka1/react-music)<br />后端地址：[NeteaseCloudMusicApi](https://github.com/Binaryify/NeteaseCloudMusicApi)
<a name="X7Sa2"></a>
## 技术选型
react@18 + vite@4 + react-router@6 + zustand + tailwindcss + axios + react-query
<a name="cCuUX"></a>
## 创建vite项目
```bash
pnpm create vite react-music --template react-swc-ts
```
这时候我们在根目录添加一个 .npmrc 文件来改变我们 pnpm 的镜像地址
```js
shamefully-hoist=true
registry=https://registry.npmmirror.com
# registry=https://registry.npmjs.com
```
然后将项目重新打开输入命令启动项目
```bash
pnpm i
pnpm run dev
```
![01.png](/assets/reactMusic/01.png)
<a name="fvLW2"></a>
## 代码规范
<a name="a84ZM"></a>
### ESLint
输入下面的指令进行 ESlint 的初始化
```bash
npm init @eslint/config
```
![02.png](/assets/reactMusic/02.png)
<a name="SY3VR"></a>
### Prettier
我们通常利用 ESLint 检测代码风格代码规范，而代码格式化使用 Prettier<br />安装 Prettier
```bash
pnpm install prettier -D
```
根目录下创建 .prettierrc.cjs 文件（规范可以自行按需修改）
```js
module.exports = {
  printWidth: 80,
  tabWidth: 2,
  useTabs: false,
  singleQuote: true,
  semi: false,
  trailingComma: "none",
  bracketSpacing: true
}
```
同时在 package.json 添加命令
```js
{
  "script": {
    "lint:prettier": "prettier --write \"src/**/*.{js,ts,json,tsx,css,html}\""
  }
}
```
<a name="o1p8c"></a>
### ESLint + Prettier
在 Eslint 中导入 Prettier 相关配置，首先安装依赖
```bash
pnpm install eslint-config-prettier eslint-plugin-prettier -D
```
然后修改 .eslintrc.cjs （规范可以自行按需修改）
```js
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true
  },
  extends: [
    'eslint:recommended',
    'plugin:react/recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:prettier/recommended',
    'plugin:react/jsx-runtime'
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaFeatures: {
      jsx: true
    },
    ecmaVersion: 'latest',
    sourceType: 'module'
  },
  plugins: ['react', '@typescript-eslint', 'prettier'],
  rules: {
    'prettier/prettier': 'error',
    'arrow-body-style': 'off',
    'prefer-arrow-callback': 'off'
  },
  settings: {
    react: {
      version: 'detect'
    }
  }
}
```
修改 package.json 中 script 的 lint 命令
```js
{
  "script": {
    "lint:eslint": "eslint --ext .js,.jsx,.ts,.tsx --fix --quiet ./"
  }
}
```
控制台运行测试是否正常
```bash
pnpm run lint 
```
<a name="btbzC"></a>
### Vite 中引入 ESLint
安装相关依赖
```bash
pnpm install vite-plugin-eslint -D
```
在 vite.config.ts 中引入插件（根据自身需求自定义配置即可）
```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react-swc'
import viteEslint from 'vite-plugin-eslint'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react(), viteEslint()]
})

```
<a name="NEgI7"></a>
### Stylelint
添加 Stylelint 配置用于检查 css 格式，由于该项目准备利用 tailwindcss 所以不进行 less 等的配置，只对 css 进行简单的配置<br />安装相关的依赖
```bash
pnpm install stylelint stylelint-config-standard stylelint-prettier -D
```
根目录下创建 .stylelintrc.cjs 文件（规范可以自行按需修改）
```js
module.exports = {
  extends: ['stylelint-config-standard', 'stylelint-prettier/recommended']
}
```
同时在 package.json 添加命令
```js
{
  "script": {
    "lint:stylelint": "stylelint \"**/*.css\" --fix"
  }
}
```
<a name="oHCvm"></a>
## Husky + lint-staged
<a name="h5zIP"></a>
### Husky
利用 Husky 在 Git commit 时进行代码校验（在进行下面的环节前请先关联你的远程仓库）<br />安装相关依赖
```bash
pnpm install husky -D
```
在 package.json 中添加脚本 prepare 并运行
```bash
npm pkg set scripts.prepare="husky install"

npm run prepare
```
运行命令后会在项目根目录创建 .husky 文件夹并给 Husky 添加一个 Hook
```bash
npx husky add .husky/pre-commit "npm run lint"
```
这样就实现了 git commit 前都会运行 npm run lint 校验代码
<a name="gbqzr"></a>
### lint-staged
添加 lint-staged 来保证我们提交代码时候只会对暂存区的代码进行校验<br />安装相关依赖
```bash
pnpm lint-staged -D
```
在 package.json 添加相关配置
```js
{
  "lint-staged": {
    "*.{js,jsx,tsx,ts}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{cjs,json}": [
      "prettier --write"
    ],
    "*.{css}": [
      "stylelint --fix",
      "prettier --write"
    ]
  }
}
```
并且修改 .husky\pre-commit 中的 npm run lint 为 npx lint-staged
<a name="P2Jyo"></a>
## commitlint
添加 commitlint 对提交信息进行校验<br />相关的规范可见 [commitlint](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fconventional-changelog%2Fcommitlint%23what-is-commitlint)<br />安装相关依赖
```bash
pnpm install @commitlint/cli @commitlint/config-conventional -D
```
在根目录创建配置文件 .commitlintrc.cjs
```js
module.exports = {
  extends: ["@commitlint/config-conventional"]
}
```
把 commitlint 命令也添加 Husky Hook
```bash
npx husky add .husky/commit-msg "npx --no-install commitlint -e $HUSKY_GIT_PARAMS"
```
