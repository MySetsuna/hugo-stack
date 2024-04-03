---
title: "从头搭建一个vite+TailwindCSS+React+redux项目"
description: 
date: 2024-04-03T13:03:21+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: true
---

## 初始化项目

### 使用vite react-ts模板

```shell
# 使用vite react-ts模板
yarn create vite xox --template react-ts

# 安装依赖
yarn
```

### 初始化TailwindCss

#### 执行命令行

```shell
# 进入项目文件
cd xox

yarn add -D tailwindcss
# npx tailwindcss init
yarn run tailwindcss init

```

```js
// 执行命令后，生成tailwind.config.js文件
/** @type {import('tailwindcss').Config} */
export default {
  content: [],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

这里自动生成了js文件，某强迫症改成ts文件，修改内容为

```ts
import { Config } from "tailwindcss";
const tailwindConfig: Config = {
  content: [],
  theme: {
    extend: {},
  },
  plugins: [],
};
export default tailwindConfig;

```

#### 配置模板文件的路径

```ts
const tailwindConfig: Config = {
// content: ["./src/**/*.{html,js}"],
  // 在react中使用，处理的文件类型如下
  content: ["./src/**/*.{js,jsx,ts,tsx}"],
  ...
};
```

#### 将加载 Tailwind 的指令添加到CSS 文件中

在你的主 CSS 文件中通过 @tailwind 指令添加每一个 Tailwind 功能模块。

```css:src/input.css
// file: src/input.css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

#### 开启 Tailwind CLI 构建流程

```shell
npx tailwindcss -i ./src/input.css -o ./src/output.css --watch
```

#### 记得下载vscode Tailwind CSS IntelliSense插件！然后在jsx代码中愉快地使用Tailwind！



