---
title: 'Nuxt3+Tailwindcss implements dark mode switching'
date: '2023-05-24'
---

最近掘金网站上线了深色模式，看 jym 都挺青睐深色模式，想着我之前也实现过深色模式的切换，不过是在使用 `VUE SPA` 做的网站上实现的，而掘金网站使用的是 `VUE SSR` 框架 `Nuxt`，我想着实现原理都是大同小异，那我们也使用 `Nuxt` 来实现一下深色模式的切换试试。

## 深色模式切换原理之一

为什么是原理之一，是因为实现深色模式有多种方式，我尝试的是通过 `css class` 切换来控制深色模式切换。
大概是这么个原理：

*   定义两套样式：你需要为深色模式和浅色模式分别定义两套样式

*   切换 CSS 类：通过 JS 动态地切换特定的 CSS 类

*   CSS 样式优先级：确保在 CSS 中正确设置深色模式和浅色模式的样式优先级。为了确保样式正确应用，你可以使用合适的 CSS 选择器，例如使用 .dark 或 .light 类来选择对应的样式。

*   存储用户选择：如果你希望记住用户的模式选择，可以使用浏览器的本地存储（如 localStorage）或其他方式将用户的偏好保存起来。这样，用户下次访问网页时，可以根据之前的选择自动应用正确的模式。

## 项目搭建

这里我用的是 `Nuxt3+Tailwindcss` 来搭建的项目，用 `Tailwindcss` 是因为这个库可以帮助我方便的实现深色模式，而且我现在的项目基本都是集成了 `Tailwindcss` 的，十分推荐大家使用它，如果你不了解它的话，现在了解一下也不迟哦！

*   第一步，在 [Nuxt 官网](https://nuxt.com/docs/getting-started/installation)，跟着文档三下五除二就新建好一个项目
*   第二步，然后在跟着 [Tailwindcss 官网](https://tailwindcss.com/docs/guides/nuxtjs#3)，三五步就能把 `Tailwindcss` 集成到 `Nuxt` 中
*   第三步，我习惯于在前端项目安装了一些插件来做规范化限制，比如在这个项目用到了这些插件

    ```json
    "eslint": "^8.40.0",
    "eslint-config-prettier": "^8.8.0",
    "eslint-plugin-prettier": "^4.2.1",
    "eslint-plugin-vue": "^9.13.0",
    "prettier": "^2.8.8",
    "prettier-plugin-tailwindcss": "^0.2.8",
    ```

我搭建完的项目在这里 [Github Repo Commit](https://github.com/yxb94616/nuxt-theme-demo/commit/ea71afb171f72215aba8bb383705038050d897db)

## 具体实现

### tailwind 深色模式配置

首先，在 `tailwind.config.js` 配置 [Dark Mode](https://tailwindcss.com/docs/dark-mode)，然后自定义一个深色模式下的颜色 (我深色模式不想要纯黑色 😂)，配置后代码如下：

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  darkMode: "class",
  content: [
    "./components/**/*.{js,vue,ts}",
    "./layouts/**/*.vue",
    "./pages/**/*.vue",
    "./plugins/**/*.{js,ts}",
    "./nuxt.config.{js,ts}",
    "./app.vue",
  ],
  theme: {
    extend: {},
    extend: {
      colors: {
        dark: "rgb(18,18,18)",
      },
    },
  },
  plugins: [],
};
```

### 新增深色模式切换组件

在根目录下新增 `components/ThemeToggle.vue` ，找两个图标分别代表深色和浅色模式，定义一个变量来控制图标在对应模式下的显隐，并且在切换模式的时候，往根标签 `<html>` 上增删 `class`，同时也在 `localstorage` 中存储一下深色模式变量，方便后边我们做网页刷新保持深色模式功能，代码如下

```html
<template>
  <div
    class="absolute right-5 top-3 h-8 w-8 cursor-pointer select-none rounded bg-slate-100 text-gray-500 shadow dark:bg-slate-700 dark:text-gray-400"
    @click="handleToggleTheme"
  >
    <span v-show="!dark">
      <svg
        t="1684833132212"
        class="icon"
        viewBox="0 0 1024 1024"
        version="1.1"
        xmlns="http://www.w3.org/2000/svg"
        p-id="8388"
        width="32"
        height="32"
      >
        <path
          d="M501.48 493.55m-233.03 0a233.03 233.03 0 1 0 466.06 0 233.03 233.03 0 1 0-466.06 0Z"
          fill="#707070"
          p-id="8389"
        ></path>
        <path
          d="M501.52 185.35H478.9c-8.28 0-15-6.72-15-15V87.59c0-8.28 6.72-15 15-15h22.62c8.28 0 15 6.72 15 15v82.76c0 8.28-6.72 15-15 15zM281.37 262.76l-16 16c-5.86 5.86-15.36 5.86-21.21 0l-58.52-58.52c-5.86-5.86-5.86-15.36 0-21.21l16-16c5.86-5.86 15.36-5.86 21.21 0l58.52 58.52c5.86 5.86 5.86 15.35 0 21.21zM185.76 478.48v22.62c0 8.28-6.72 15-15 15H88c-8.28 0-15-6.72-15-15v-22.62c0-8.28 6.72-15 15-15h82.76c8.28 0 15 6.72 15 15zM270.69 698.63l16 16c5.86 5.86 5.86 15.36 0 21.21l-58.52 58.52c-5.86 5.86-15.36 5.86-21.21 0l-16-16c-5.86-5.86-5.86-15.36 0-21.21l58.52-58.52c5.85-5.86 15.35-5.86 21.21 0zM486.41 794.24h22.62c8.28 0 15 6.72 15 15V892c0 8.28-6.72 15-15 15h-22.62c-8.28 0-15-6.72-15-15v-82.76c0-8.28 6.72-15 15-15zM706.56 709.31l16-16c5.86-5.86 15.36-5.86 21.21 0l58.52 58.52c5.86 5.86 5.86 15.36 0 21.21l-16 16c-5.86 5.86-15.36 5.86-21.21 0l-58.52-58.52c-5.86-5.85-5.86-15.35 0-21.21zM802.17 493.59v-22.62c0-8.28 6.72-15 15-15h82.76c8.28 0 15 6.72 15 15v22.62c0 8.28-6.72 15-15 15h-82.76c-8.28 0-15-6.72-15-15zM717.24 273.44l-16-16c-5.86-5.86-5.86-15.36 0-21.21l58.52-58.52c5.86-5.86 15.36-5.86 21.21 0l16 16c5.86 5.86 5.86 15.36 0 21.21l-58.52 58.52c-5.86 5.86-15.35 5.86-21.21 0z"
          fill="#707070"
          p-id="8390"
        ></path>
      </svg>
    </span>
    <span v-show="dark">
      <svg
        t="1684833077751"
        class="icon"
        viewBox="0 0 1024 1024"
        version="1.1"
        xmlns="http://www.w3.org/2000/svg"
        p-id="7162"
        width="32"
        height="32"
      >
        <path
          d="M335.22 240.91c0-57.08 10.68-111.66 30.15-161.87-167.51 64.86-286.3 227.51-286.3 417.92 0 247.42 200.58 448 448 448 190.34 0 352.95-118.71 417.85-286.13-50.16 19.42-104.69 30.08-161.71 30.08-247.41 0-447.99-200.57-447.99-448z"
          fill="#dbdbdb"
          p-id="7163"
        ></path>
      </svg>
    </span>
  </div>
</template>

<script lang="ts" setup>
const dark = ref(false);

const handleToggleTheme = () => {
  const currentTheme = !dark.value;
  dark.value = currentTheme;
  if (currentTheme) {
    localStorage.setItem("theme", "dark");
    document.documentElement.classList.add("dark");
  } else {
    localStorage.removeItem("theme");
    document.documentElement.classList.remove("dark");
  }
};
</script>
```

### 添加测试代码

在 `pages/index.vue` 中添加测试代码，引入 `ThemeToggle` 组件，代码如下：

```html
<template>
  <div
    class="dark:bg-dark flex h-screen w-screen flex-col items-center justify-center bg-white"
  >
    <h1 class="text-6xl font-semibold text-sky-400">2023 ikun</h1>
    <p class="mt-4 text-9xl font-bold text-gray-600">只因你太美</p>
    <ThemeToggle />
  </div>
</template>
```

启动项目，访问网站就能看到如下效果

![n1.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/afda17b016b94f6c869b9ca63ead5b96~tplv-k3u1fbpfcp-watermark.image?)

### 新增持久化深色模式功能

能看到已经实现深色模式切换了，但是在深色模式下刷新网站的时候，深色模式会丢失，
上面提到在 `localstorage` 中存储了深色模式变量，那怎么去恢复深色模式呢，

*   在 `components/ThemeToggle.vue` 的 `onMounted` 里获取 `localstorage` 做判断，如果是深色模式，那就做一下处理，在组件里新增下面这段代码：

```js
onMounted(() => {
  if (localStorage.getItem("theme") === "dark") {
    dark.value = true;
    document.documentElement.classList.add("dark");
  }
});
```

然后我们再去测试一下，深色模式下刷新网站，效果如图

![n2.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42ec555012444f109344066fedce4826~tplv-k3u1fbpfcp-watermark.image?)

的确是持久化了深色模式，但是发现刷新网站的时候会闪一下，这是为什么呢？

*   这是因为 nuxt 框架生成的 js 代码插入到了 DOM 文档的尾部，造成 js 执行晚了一步，所以获取 `localstorage` 就延迟了一点点，造成设置深色模式的时候会闪一下

怎么解决呢？

*   那就把获取 `localstorage` 的代码插入到 DOM 文档头部不就行了，而且如果没有设置 defer 或 async 属性， js 加载还会造成浏览器将阻塞渲染，更加的能防止闪烁出现

### 解决闪烁问题

*   `Nuxt` 提供了在 DOM 文档头部插入代码的方法，请阅读 [Nuxt Configuration Reference](https://nuxt.com/docs/api/configuration/nuxt-config#head)

*   新建脚本文件 `public/script.js`，代码如下（这段代码来源是 [tailwindcss](https://tailwindcss.com/docs/dark-mode#supporting-system-preference-and-manual-selection)）：

```js
if (
  localStorage.getItem("theme") === "dark" ||
  (!localStorage.getItem("theme") &&
    window.matchMedia("(prefers-color-scheme: dark)").matches)
) {
  document.documentElement.classList.add("dark");
} else {
  document.documentElement.classList.remove("dark");
}

```

*   然后在 `nuxt.config.js` 增加配置：

```js
// https://nuxt.com/docs/api/configuration/nuxt-config
export default defineNuxtConfig({
  app: {
    head: {
      script: ["/script.js"],
    },
  },
  css: ["~/assets/css/main.css"],
  postcss: {
    plugins: {
      tailwindcss: {},
      autoprefixer: {},
    },
  },
  devServer: {
    port: 5005,
  },
});
```

*   `components/ThemeToggle.vue` 的 `onMounted` 还是需要的，需要保持变量也同步，改一下

```js
onMounted(() => {
  if (localStorage.getItem("theme") === "dark") {
    dark.value = true;
  }
});
```

*   这里还有一个小问题，就是图标也会跟着闪烁的，原因还是js执行顺序的问题，我的解决办法是改为使用 `css class` 来控制图标的显隐，修改 `components/ThemeToggle.vue` 代码如下：

```html
- <span v-show="!dark">
+ <span class="block dark:hidden">

- <span v-show="dark">
+ <span class="hidden dark:block">
```

ok，现在我们再来在深色模式下刷新网页，么得问题，达到了想要的效果。

因为在正常效果下录制 GIF 的话，刷新也看不出来，所以这里就不放效果图了，最后的代码在上面我提到的 [github 仓库](https://github.com/yxb94616/nuxt-theme-demo)。

> 如有错误，欢迎指正