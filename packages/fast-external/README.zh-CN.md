# vite-plugin-fast-external

[![npm package](https://nodei.co/npm/vite-plugin-fast-external.png?downloads=true&downloadRank=true&stars=true)](https://www.npmjs.com/package/vite-plugin-fast-external)
<br/>
[![NPM version](https://img.shields.io/npm/v/vite-plugin-fast-external.svg?style=flat)](https://npmjs.org/package/vite-plugin-fast-external)
[![NPM Downloads](https://img.shields.io/npm/dm/vite-plugin-fast-external.svg?style=flat)](https://npmjs.org/package/vite-plugin-fast-external)

不使用语法转换, 支持自定义 external 代码段

**[English](https://github.com/caoxiemeihao/vite-plugins/tree/main/packages/fast-external#readme) | 简体中文**

- 类似 webpack 的 externals，支持浏览器、Node.js、Electron 等多环境

- 本质上是通过 `resolve.alias` 实现的模块重定向加载

- 默认使用的 window 作为宿主对象，你也可以通过函数返回字符串的形式任意定制代码段 -- 十分灵活！🎉

## 安装

```bash
npm i vite-plugin-fast-external -D
```

## 使用

```js
import external from 'vite-plugin-fast-external';

export default defineConfig({
  plugins: [
    external({
      // 基本使用
      // 默认会生成 const Vue = window['Vue']; export { Vue as default }
      vue: 'Vue',

      // 支持包命名空间，通过函数可以自定义返回任何代码段 - 但你要知道 vite 开发期只支持 ESM
      '@scope/name': () => `const Lib = window.ScopeName.Member; export default Lib;`,

      // 还支持返回 Promise<string> 很容易配合文件、网络等 IO
      externalId: async () => await require('fs').promises.readFile('path', 'utf-8'),

      // 在 Electron 渲染进程中使用
      electron: () => `const { ipcRenderer } = require('electron'); export { ipcRenderer }`,
    })
  ]
})
```

## API

### external(externals[, options])

#### externals

```ts
export type Externals = Record<string, string | ((args: { dir: string; }) => string | Promise<string>)>;
```

#### options

```ts
export interface ExternalOptions {
  /**
   * 相对或绝对路径
   * @default ".vite-plugin-fast-external"
   */
  dir?: string;
}
```

## 工作原理

**用 Vue 来举个 🌰**

```js
external({
  vue: 'Vue',
})
```

1. 创建 `node_modules/.vite-plugin-fast-external/vue.js` 文件并包含下面的代码

```js
const vue = window['Vue']; export { vue as default }
```

2. 创建一个 `vue` 的别名项，并且添加到 `resolve.alias`

```js
{
  resolve: {
    alias: [
      {
        find: 'vue',
        replacement: 'User/work-directory/node_modules/.vite-plugin-fast-external/vue.js',
      },
    ],
  },
}
```

3. 默认会将 `vue` 添加到 `optimizeDeps.exclude` 中  
  你可以通过 `optimizeDeps.include` 绕开

```js
export default {
  optimizeDeps: {
    exclude: ['vue'],
  },
}
```
