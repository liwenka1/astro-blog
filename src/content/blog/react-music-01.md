---
author: 李文凯
pubDatetime: 2023-08-20T17:03:24+08:00
title: 从零开始的Web端React音乐播放器（一）
postSlug: react-music-01
featured: false
draft: false
tags:
  - React + Vite
  - ESLint + Prettier + Stylelint
  - Husky + lint-staged + commitlint
ogImage: ""
description: 通过 Vite 创建项目，并且利用 ESLint + Prettier + Stylelint 来校验与格式化代码，Husky + lint-staged + commitlint 来对 git commit 来进行校验
canonicalURL: ""
---

<a name="sI8XM"></a>

## 前言

本系列将从零开始制作一个 Web 端的音乐播放器，前端采用 React 相关的技术栈，后端采用开源的网易云音乐 api<br />这一节将通过 Vite 创建项目，并且利用 ESLint + Prettier + Stylelint 来校验与格式化代码，Husky + lint-staged + commitlint 来对 git commit 来进行校验
<a name="NYNsa"></a>

### 项目地址

前端地址：[react-music](https://github.com/liwenka1/react-music)<br />后端地址：[NeteaseCloudMusicApi](https://github.com/Binaryify/NeteaseCloudMusicApi)
<a name="eFze0"></a>

### 预览图

![推荐-dark.png](/assets/reactMusic/home-dark.png)![推荐-light.png](/assets/reactMusic/home-light.png)
<a name="X7Sa2"></a>

## 技术选型

react@18 + vite@4 + react-router@6 + zustand + tailwindcss + shadcn/ui + axios + react-query
<a name="cCuUX"></a>

## 创建 vite 项目

```
pnpm create vite react-music --template react-swc-ts
```

这时候我们在根目录添加一个 .npmrc 文件来改变我们 pnpm 的镜像地址

```
shamefully-hoist=true
registry=https://registry.npmmirror.com
# registry=https://registry.npmjs.com
```

然后将项目重新打开输入命令启动项目

```
pnpm i
pnpm run dev
```

![01.png](/assets/reactMusic/01.png)
<a name="fvLW2"></a>

## 代码规范

<a name="a84ZM"></a>

### ESLint

输入下面的指令进行 ESlint 的初始化

```
npm init @eslint/config
```

![02.png](/assets/reactMusic/02.png)
<a name="SY3VR"></a>

### Prettier

我们通常利用 ESLint 检测代码风格代码规范，而代码格式化使用 Prettier<br />安装 Prettier

```
pnpm install prettier -D
```

根目录下创建 .prettierrc.cjs 文件（规范可以自行按需修改）

```
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

```
{
  "script": {
    "lint:prettier": "prettier --write \"src/**/*.{js,ts,json,tsx,css,html}\""
  }
}
```

<a name="o1p8c"></a>

### ESLint + Prettier

在 Eslint 中导入 Prettier 相关配置，首先安装依赖

```
pnpm install eslint-config-prettier eslint-plugin-prettier -D
```

然后修改 .eslintrc.cjs （规范可以自行按需修改）

```
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

```
{
  "script": {
    "lint:eslint": "eslint --ext .js,.jsx,.ts,.tsx --fix --quiet ./"
  }
}
```

控制台运行测试是否正常

```
pnpm run lint
```

<a name="btbzC"></a>

### Vite 中引入 ESLint

安装相关依赖

```
pnpm install vite-plugin-eslint -D
```

在 vite.config.ts 中引入插件（根据自身需求自定义配置即可）

```
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

```
pnpm install stylelint stylelint-config-standard stylelint-prettier -D
```

根目录下创建 .stylelintrc.cjs 文件（规范可以自行按需修改）

```
module.exports = {
  extends: ['stylelint-config-standard', 'stylelint-prettier/recommended']
}
```

同时在 package.json 添加命令

```
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

```
pnpm install husky -D
```

在 package.json 中添加脚本 prepare 并运行

```
npm pkg set scripts.prepare="husky install"

npm run prepare
```

运行命令后会在项目根目录创建 .husky 文件夹并给 Husky 添加一个 Hook

```
npx husky add .husky/pre-commit "npm run lint"
```

这样就实现了 git commit 前都会运行 npm run lint 校验代码
<a name="gbqzr"></a>

### lint-staged

添加 lint-staged 来保证我们提交代码时候只会对暂存区的代码进行校验<br />安装相关依赖

```
pnpm lint-staged -D
```

在 package.json 添加相关配置

```
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

```
pnpm install @commitlint/cli @commitlint/config-conventional -D
```

在根目录创建配置文件 .commitlintrc.cjs

```
module.exports = {
  extends: ["@commitlint/config-conventional"]
}
```

把 commitlint 命令也添加 Husky Hook

```
npx husky add .husky/commit-msg "npx --no-install commitlint -e $HUSKY_GIT_PARAMS"
```

<a name="eRr3K"></a>

## **EditorConfig**

下载 EditorConfig for VS Code 插件并且添加.editorconfig 文件来处理一些编写问题（如果你使用的是 VS Code 的话）

```typescript
# http://editorconfig.org
root = true

# 表示所有文件适用
[*]
charset = utf-8 # 设置文件字符集为 utf-8
end_of_line = lf # 控制换行类型(lf | cr | crlf)
indent_style = tab # 缩进风格（tab | space）
insert_final_newline = true # 始终在文件末尾插入一个新行

# 表示仅 md 文件适用以下规则
[*.md]
max_line_length = off # 关闭最大行长度限制
trim_trailing_whitespace = false # 关闭末尾空格修剪
```

<a name="RUD7P"></a>

## 添加 @ 别名

修改 vite.config.ts

```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react-swc";
import viteEslint from "vite-plugin-eslint";
import path from "path";

// https://vitejs.dev/config/
export default defineConfig({
	plugins: [react(), viteEslint()],
	resolve: {
		alias: {
			"@": path.resolve(__dirname, "./src"),
		},
	},
});
```

修改 tsconfig.json

```json
{
	"compilerOptions": {
		"target": "ES2020",
		"useDefineForClassFields": true,
		"lib": ["ES2020", "DOM", "DOM.Iterable"],
		"module": "ESNext",
		"skipLibCheck": true,

		/* Bundler mode */
		"moduleResolution": "bundler",
		"allowImportingTsExtensions": true,
		"resolveJsonModule": true,
		"isolatedModules": true,
		"noEmit": true,
		"jsx": "react-jsx",

		/* Linting */
		"strict": true,
		"noUnusedLocals": true,
		"noUnusedParameters": true,
		"noFallthroughCasesInSwitch": true,
		"baseUrl": ".",
		"paths": {
			"@/*": ["./src/*"]
		}
	},
	"include": ["src"],
	"references": [{ "path": "./tsconfig.node.json" }]
}
```
