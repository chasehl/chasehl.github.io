---
title: Github Pages 博客搭建指南
date: 2026-03-12 00:42:38
tags:
---

# GitHub Pages 博客搭建指南

> Hexo + Butterfly 主题 + GitHub Actions 自动部署
> 个人备忘，记录首次搭建流程及后续更换博客的方法

---

## 目录

1. [技术栈说明](#一技术栈说明)
2. [首次搭建完整步骤](#二首次搭建完整步骤)
3. [日常写文章工作流](#三日常写文章工作流)
4. [我的情况：已有博客，重新搭建替换](#四我的情况已有博客重新搭建替换)
5. [常见报错处理](#五常见报错处理)
6. [文件结构说明](#六文件结构说明)

---

## 一、技术栈说明

| 工具           | 作用                          | 安装位置             |
| -------------- | ----------------------------- | -------------------- |
| Node.js        | 运行环境，Hexo 依赖它         | C 盘（系统级）       |
| Hexo CLI       | 博客框架命令行工具            | C 盘（全局 npm 包）  |
| Butterfly      | Hexo 主题，决定博客外观       | 项目 node_modules 内 |
| Git            | 版本管理，推送到 GitHub       | C 盘（系统级）       |
| GitHub Pages   | 免费静态网站托管              | 云端                 |
| GitHub Actions | 自动编译部署，push 后自动触发 | 云端                 |

**核心理解：**

- Hexo CLI 是"工具"，装在 C 盘，装一次不用管
- 博客项目是"内容"，可以放在任意盘（如 `E:\blog`），路径不要有中文和空格
- GitHub Actions 配置好后，本地只需写文章 + `git push`，编译部署全在云端自动完成

---

## 二、首次搭建完整步骤

### 前置准备

1. 安装 Node.js（LTS 版本，需 18+）：[https://nodejs.org](https://nodejs.org)
2. 安装 Git：[https://git-scm.com](https://git-scm.com)
3. 注册 GitHub 账号：[https://github.com](https://github.com)

验证安装：
```powershell
node --version   # v18.x.x 或更高
npm --version
git --version
```

---

### 第 1 步：在 GitHub 创建仓库

仓库名必须为 `你的用户名.github.io`（固定格式，不能改）。

1. 登录 GitHub → 右上角 `+` → New repository
2. Repository name 填写：`你的用户名.github.io`
3. 设为 Public
4. 不要勾选 Initialize README
5. 点击 Create repository

---

### 第 2 步：安装 Hexo CLI
```powershell
npm install -g hexo-cli
```

验证：
```powershell
hexo --version
```

---

### 第 3 步：初始化博客项目
```powershell
# 切换到 E 盘（或你想放的位置）
E:

# 初始化项目，会创建 E:\blog 文件夹
hexo init blog
cd blog
npm install
```

---

### 第 4 步：安装 Butterfly 主题
```powershell
npm install hexo-theme-butterfly hexo-renderer-pug hexo-renderer-stylus --save
```

---

### 第 5 步：配置 `_config.yml`

用 VS Code 打开 `E:\blog\_config.yml`，修改以下字段：
```yaml
title: 你的博客名
subtitle: '副标题'
description: '博客描述'
author: 你的名字

url: https://你的用户名.github.io
root: /

theme: butterfly
```

可以删除 `_config.landscape.yml`（默认主题配置，用不到）：
```powershell
del _config.landscape.yml
```

---

### 第 6 步：创建 Butterfly 主题配置

在 `E:\blog\` 根目录新建 `_config.butterfly.yml`：
```yaml
# 深色模式
dark_mode:
  enable: true
  default: light

# 代码块
highlight_theme: mac
highlight_copy: true
highlight_lang: true

# 文章封面
cover:
  index_enable: true
  aside_enable: true
  archives_enable: true
  position: both
  default_cover:

# 侧边栏
aside:
  enable: true
  position: right
  card_author:
    enable: true
    description: 记录折腾过程的地方
  card_tags:
    enable: true
  card_archives:
    enable: true
  card_recent_post:
    enable: true

# 页脚
footer:
  owner:
    enable: true
    since: 2024

# 关闭暂不需要的功能
aplayer:
  enable: false
```

---

### 第 7 步：本地预览
```powershell
hexo clean
hexo server
```

打开浏览器访问 `http://localhost:4000`，确认主题和样式正常。

> `Ctrl+C` 停止预览服务器。

---

### 第 8 步：初始化 Git 并关联远程仓库
```powershell
git init
git add .
git commit -m "init: hexo + butterfly"
git branch -M main
git remote add origin https://github.com/你的用户名/你的用户名.github.io.git
```

---

### 第 9 步：配置 GitHub Actions 自动部署

在项目根目录创建文件（路径如下，需手动创建文件夹）：
```
E:\blog\.github\workflows\deploy.yml
```

文件内容：
```yaml
name: Deploy Hexo Blog

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: |
          npm install hexo-cli -g
          npm install

      - name: Build
        run: |
          hexo clean
          hexo generate

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
          publish_branch: gh-pages
```

---

### 第 10 步：推送到 GitHub
```powershell
git add .
git commit -m "deploy: add github actions"
# 首次推送需要 -u 指定上游，之后直接 git push 即可
git push -u origin main
```

---

### 第 11 步：设置 GitHub Pages 分支

推送成功后（等 Actions 跑完，约 1~2 分钟）：

1. 打开 GitHub 仓库页面
2. Settings → Pages
3. Source 选择 **Deploy from a branch**
4. Branch 选 **gh-pages**，目录选 `/ (root)`
5. 点击 Save

稍等片刻，访问 `https://你的用户名.github.io` 即可看到博客上线。

---

## 三、日常写文章工作流
```powershell
# 进入项目目录
cd E:\blog

# 新建文章（会在 source/_posts/ 下生成 .md 文件）
hexo new post "文章标题"

# 用 VS Code 编辑文章
# source/_posts/文章标题.md

# 本地预览（可选）
hexo clean
hexo server

# 满意后推送，GitHub Actions 自动编译部署
git add .
git commit -m "新文章：文章标题"
git push
```

### 文章头部格式（Front Matter）
```markdown
---
title: 文章标题
date: 2026-03-12
tags:
  - 标签1
  - 标签2
categories:
  - 分类名
cover: https://可选的封面图片URL
---

正文内容从这里开始，支持标准 Markdown 语法。
```

---

## 四、我的情况：已有博客，重新搭建替换

> 适用场景：已有 `用户名.github.io` 仓库，但本地文件丢失，想全部重来换新主题。

### 与首次搭建的区别

| 步骤              | 首次搭建    | 替换重建               |
| ----------------- | ----------- | ---------------------- |
| 创建 GitHub 仓库  | 需要新建    | 仓库已存在，跳过       |
| 本地初始化        | `hexo init` | `hexo init`（一样）    |
| 关联远程仓库      | 正常 push   | 需要**强制覆盖**旧仓库 |
| GitHub Pages 设置 | 首次配置    | 可能需要重新确认分支   |

---

### 完整操作步骤

**第 1~7 步与首次搭建完全相同**（安装工具、初始化项目、配置主题、本地预览），按照第二章操作即可。

---

**第 8 步：关联已有的远程仓库**
```powershell
git init
git add .
git commit -m "init: hexo + butterfly"
git branch -M main
git remote add origin https://github.com/你的用户名/你的用户名.github.io.git
```

---

**第 9 步：添加 GitHub Actions 配置**

同首次搭建第 9 步，创建 `.github/workflows/deploy.yml`，内容完全相同。

---

**第 10 步：强制推送覆盖旧仓库**

> ⚠️ 因为旧仓库有历史提交记录，普通 `git push` 会被拒绝，需要强制覆盖。
```powershell
git push -f -u origin main
```

`-f` 表示强制覆盖，`-u` 设置上游分支。**之后写文章正常 `git push` 即可，不需要再加 `-f`。**

---

**第 11 步：确认 GitHub Pages 分支设置**

等 GitHub Actions 跑完后（仓库 Actions 标签页查看进度）：

1. Settings → Pages
2. 确认 Source 为 **Deploy from a branch**，Branch 为 **gh-pages**
3. 如果之前设置过其他分支，重新选择 gh-pages 并保存

---

## 五、常见报错处理

### `&&` 不是有效语句分隔符

**原因**：PowerShell 不支持 `&&`，这是 CMD/bash 的写法。

**解决**：分开执行，或用分号：
```powershell
# 方式一：分开执行
hexo clean
hexo server

# 方式二：分号分隔（PowerShell 语法）
hexo clean; hexo server
```

---

### `fatal: The current branch main has no upstream branch`

**原因**：本地分支第一次推送，还没有指定远程上游。

**解决**：首次推送加 `-u` 参数：
```powershell
git push -u origin main
```

之后直接 `git push` 即可。

---

### `error: failed to push some refs`（推送被拒绝）

**原因**：远程仓库有本地没有的提交（如替换旧博客时）。

**解决**：确认要覆盖旧内容后，强制推送：
```powershell
git push -f origin main
```

---

### GitHub Actions 运行失败

1. 打开 GitHub 仓库 → Actions 标签页
2. 点击失败的工作流，查看具体报错信息
3. 常见原因：`deploy.yml` 缩进格式错误（YAML 对缩进严格，只能用空格，不能用 Tab）

---

## 六、文件结构说明
```
E:\blog\
├── .github\
│   └── workflows\
│       └── deploy.yml          ← GitHub Actions 自动部署配置
├── node_modules\               ← 依赖包，自动生成，不提交 git
├── public\                     ← 编译产物，自动生成，不提交 git
├── scaffolds\                  ← 文章模板，一般不用动
├── source\
│   ├── _posts\                 ← ★ 所有文章放这里（.md 文件）
│   └── about\
│       └── index.md            ← 关于页
├── themes\                     ← 主题（用 npm 安装时在 node_modules 里）
├── .gitignore                  ← 忽略 node_modules 和 public 等
├── _config.yml                 ← Hexo 全局配置（标题、URL、主题等）
├── _config.butterfly.yml       ← Butterfly 主题配置（样式、功能开关）
└── package.json                ← 项目依赖清单
```

**平时只需要动的文件：**

- `source/_posts/*.md` — 写文章
- `_config.butterfly.yml` — 调整博客样式和功能
- `_config.yml` — 修改博客标题、作者等基础信息
