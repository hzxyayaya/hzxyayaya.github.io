---
title: deploy_blog
published: 2025-07-07
description: '第一次部署个人博客'
image: ''
tags: [blog]
category: '个人'
draft: false 
lang: 'zh'
---
## 使用fuwari模板
省略nodejs配置
运行
```sh
npm create fuwari@latest
```
根据指引完善内容

## 使用astro和github Page部署
1. 修改`astro.config.mjs`
添加
```js
export default defineConfig({

site: 'https://astronaut.github.io',

base: 'my-repo',

})
```
修改`site`后面的值
- 使用基于你的用户名的地址：`https://<username>.github.io`
`base`可以默认为你的项目的根目录`/`
2. 配置GitHub Action
>在你的项目中的 `.github/workflows/` 目录创建一个新文件 `deploy.yml`，并粘贴以下 YAML 配置信息

```yml
name: Deploy to GitHub Pages

on:
  # 每次推送到 `main` 分支时触发这个“工作流程”
  # 如果你使用了别的分支名，请按需将 `main` 替换成你的分支名
  push:
    branches: [ main ]
  # 允许你在 GitHub 上的 Actions 标签中手动触发此“工作流程”
  workflow_dispatch:

# 允许 job 克隆 repo 并创建一个 page deployment
permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout your repository using git
        uses: actions/checkout@v4
      - name: Install, build, and upload your site
        uses: withastro/action@v3
        # with:
          # path: . # 存储库中 Astro 项目的根位置。（可选）
          # node-version: 20 # 用于构建站点的特定 Node.js 版本，默认为 20。（可选）
          # package-manager: pnpm@latest # 应使用哪个 Node.js 包管理器来安装依赖项和构建站点。会根据存储库中的 lockfile 自动检测。（可选）

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
3. 在GitHub上新建一个<your_username>.github.io的仓库
4. 按照指引将本地的项目上传到刚刚创建的仓库中
5. 在 GitHub 上，跳转到存储库的 **Settings** 选项卡并找到设置的 **Pages** 部分
6. 选择 **GitHub Actions** 作为你网站的 **Source**，然后按 **Save**

你的网站现在应该已完成发布了！当你将更改推送到 Astro 项目的存储库时，GitHub Action 将自动为你部署它们。

## 在部署的时候出现一些小问题
1. 注意astro文件的格式问题
2. 在`markdown.css`文件中修改两处地方
第一处
```css
22 - a:not(.no-styling) {
23 -       @apply relative bg-none link font-medium text-[var(--primary)]
```
删除`a`和`link`
```css
22 +      :not(.no-styling) {
23 +       @apply relative bg-none font-medium text-[var(--primary)]
```
第二处
```css
    .copy-btn {
        all: initial;
63 -    @apply btn-regular-dark opacity-0 shadow-lg shadow-black/50 absolute active:scale-90 h-8 w-8 top-3 right-3 text-sm rounded-lg transition-all ease-in-out z-20 cursor-pointer;
    }
```

```css
    .copy-btn {
        all: initial;
63 +    @apply bg-gray-800 text-white px-4 py-2 rounded shadow opacity-0 shadow-lg shadow-black/50 absolute active:scale-90 h-8 w-8 top-3 right-3 text-sm rounded-lg transition-all ease-in-out z-20 cursor-pointer;
    }
```

## 参考资料
[fuwari](https://github.com/saicaca/fuwari?tab=readme-ov-file)
[astro.build/zh-cn](https://docs.astro.build/zh-cn/guides/deploy/github/)