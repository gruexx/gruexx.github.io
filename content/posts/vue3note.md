---
title: Vue3笔记
date: 2023-12-20T11:10:06+08:00
subtitle: 笔记自用 仅供参考
draft: false
description: "Vue3笔记"
tags: [ "vue3" ]
categories: [ "学习笔记" ]
---

# npm 换源

```shell
// 查询源
npm config get registry

// 更换国内源
npm config set registry https://registry.npmmirror.com

// 恢复官方源
npm config set registry https://registry.npmjs.org

// 删除注册表
npm config delete registry
```

# Tailwind CSS

[快速安装](https://www.tailwindcss.cn/docs/guides/vite#vue)

```shell
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```
编辑 `tailwind.config.js`
```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{vue,js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```
引入
```css
@tailwind base; /*似乎会影响到原有的css,可以不引入*/
@tailwind components;
@tailwind utilities;
```
执行以上步骤后但是tailwind没有生效可以在`vite.config.js`中增加配置

```javascript
css: {
  postcss: {
    plugins: [require("tailwindcss"), require("autoprefixer")]
  }
}
```
