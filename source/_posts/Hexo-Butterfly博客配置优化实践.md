---
title: 从零搭建 Hexo + Butterfly 博客：配置优化与个性化实践
date: 2026-03-28
categories:
  - 折腾记录
tags:
  - Hexo
  - Butterfly
  - GitHub Pages
  - 博客搭建
cover: /img/covers/1.jpg
---

## 前言

博客搭好之后，默认配置往往只是个骨架——功能残缺、样式平淡、部署流程也没打通。这篇文章记录了我在 Hexo + Butterfly 博客上做的一轮系统性配置与优化，涵盖主配置修正、主题功能补全、自定义样式、自动化部署等方面，供有类似需求的人参考。

---

## 功能与优化总览

- 修正 `_config.yml` 基础配置（语言、时区、URL、部署）
- 补全 `_config.butterfly.yml` 主题功能（侧边栏、TOC、版权、统计等）
- 自定义 CSS 样式（卡片、代码块、深色模式、移动端）
- GitHub Actions 自动构建并部署到 `gh-pages` 分支

---

## 详细实现

### 一、修正 `_config.yml` 基础配置

初始配置有几处明显问题：语言是 `en`、时区为空、URL 填错了、deploy 没有配置。

```yaml
# _config.yml

title: ChaseLog
subtitle: '记录折腾的地方'
description: '记录云服务器、AI工具、技术学习的个人博客'
keywords: 博客,技术,AI,折腾,Hexo
author: Chase King
language: zh-CN
timezone: 'Asia/Shanghai'

url: https://chasehl.github.io

deploy:
  type: git
  repo: https://github.com/chasehl/chasehl.github.io.git
  branch: main
```

`language: zh-CN` 会让 Butterfly 自动切换为中文界面（菜单、提示文字等）。`timezone` 影响文章发布时间排序，不设置在 CI 环境下容易出现时区偏差。

---

### 二、补全 Butterfly 主题配置

Butterfly 的 `_config.butterfly.yml` 配置项非常多，默认状态下大量功能是关闭的。以下是我补全的核心部分。

#### 侧边栏

```yaml
aside:
  enable: true
  position: right
  card_author:
    enable: true
    description: 记录折腾过程的地方
    button:
      enable: true
      icon: fab fa-github
      text: GitHub
      link: https://github.com/chasehl
  card_announcement:
    enable: true
    content: 欢迎来到 ChaseLog，这里记录我的折腾历程 🚀
  card_recent_post:
    enable: true
    limit: 5
  card_tags:
    enable: true
    limit: 40
    color: true
  card_webinfo:
    enable: true
    post_count: true
    last_push_date: true
```

`card_tags` 开启 `color: true` 后标签会随机着色，视觉上比全灰好很多。

#### 文章目录（TOC）

```yaml
toc:
  post: true
  number: true
  expand: true
  scroll_percent: true
```

`scroll_percent: true` 会在右下角显示阅读进度百分比，长文章体验更好。

#### 字数统计与阅读时长

需要先安装插件：

```bash
npm install hexo-wordcount --save
```

然后在主题配置中开启：

```yaml
wordcount:
  enable: true
  post_wordcount: true
  min2read: true
  total_wordcount: true
```

#### 版权声明

```yaml
post_copyright:
  enable: true
  license: CC BY-NC-SA 4.0
  license_url: https://creativecommons.org/licenses/by-nc-sa/4.0/
```

每篇文章底部自动附上版权信息，省去手动添加的麻烦。

#### 文章过期提醒

```yaml
noticeOutdate:
  enable: true
  style: flat
  limit_day: 365
  position: top
  message_prev: 本文已超过
  message_next: 天未更新，内容可能已过时，请注意甄别。
```

超过 365 天未更新的文章顶部会自动显示提醒横幅，对技术类内容很实用。

#### 浏览器 Tab 切换提示

```yaml
change_title_bar:
  enable: true
  hidden: 你去哪了？快回来！
  visible: 欢迎回来！
```

切换到其他 Tab 再切回来时，标题会变化，小细节但有趣。

#### 访问统计（不蒜子）

```yaml
busuanzi:
  site_uv: true
  site_pv: true
  page_pv: true
```

不需要注册账号，直接引入即可，适合轻量博客。

#### 深色模式

```yaml
dark_mode:
  enable: true
  default: light
```

默认浅色，用户可手动切换，偏好会保存在 localStorage。

#### 代码块样式

```yaml
highlight_theme: mac
highlight_copy: true
highlight_lang: true
highlight_shrink: false
```

`mac` 风格代码块带红黄绿三个圆点，视觉上比默认好看。

---

### 三、自定义 CSS

Butterfly 支持通过 `inject` 注入自定义样式文件：

```yaml
# _config.butterfly.yml
inject:
  head:
    - <link rel="stylesheet" href="/css/custom.css">
```

文件放在 `source/css/custom.css`，以下是关键样式片段。

#### 字体

```css
@import url('https://fonts.googleapis.com/css2?family=Noto+Serif+SC:wght@400;700&family=JetBrains+Mono&display=swap');

body {
  font-family: 'Noto Serif SC', serif;
}

code, pre {
  font-family: 'JetBrains Mono', monospace;
}
```

正文用衬线字体增加阅读质感，代码用等宽字体保证对齐。

#### 文章卡片 hover 效果

```css
.recent-post-item {
  border-radius: 12px !important;
  box-shadow: 0 4px 20px rgba(0,0,0,0.08) !important;
  transition: transform 0.3s ease, box-shadow 0.3s ease !important;
}

.recent-post-item:hover {
  transform: translateY(-4px) !important;
  box-shadow: 0 8px 30px rgba(0,0,0,0.15) !important;
}
```

#### 深色模式适配

```css
[data-theme="dark"] .recent-post-item {
  box-shadow: 0 4px 20px rgba(0,0,0,0.3) !important;
}

[data-theme="dark"] blockquote {
  background: rgba(66, 90, 239, 0.08) !important;
}
```

Butterfly 切换深色模式时会在 `<html>` 上加 `data-theme="dark"`，直接用属性选择器覆盖即可。

#### 移动端禁用 hover 动画

```css
@media (max-width: 768px) {
  .recent-post-item:hover {
    transform: none !important;
  }
}
```

移动端没有 hover 状态，但触摸时会短暂触发，禁掉避免视觉抖动。

---

### 四、GitHub Actions 自动部署

推送到 `main` 分支后自动构建并发布到 `gh-pages`，不需要本地执行 `hexo deploy`。

```yaml
# .github/workflows/deploy.yml
name: Deploy Hexo Blog

on:
  push:
    branches: [main]

permissions:
  contents: write
  pages: write
  id-token: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'

      - run: npm install

      - run: npm install hexo-cli -g

      - run: hexo clean && hexo generate

      - uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
          publish_branch: gh-pages
```

GitHub Pages 的 Source 设置为 `gh-pages` 分支，每次 push 后约 1 分钟内生效。

---

### 五、友链配置

友链数据通过 `source/_data/link.yml` 管理，格式如下：

```yaml
- class_name: 技术博客
  class_desc: 优秀的技术内容创作者
  link_list:
    - name: 示例博客
      link: https://example.com
      avatar: https://example.com/avatar.jpg
      descr: 这是一个示例友链
```

页面文件 `source/links/index.md` 只需声明 `type: link` 即可，数据与页面解耦。

---

## 遇到的问题

**git safe.directory 报错**

在 Windows 上用 bash 执行 git 命令时遇到：

```
fatal: detected dubious ownership in repository at 'E:/blog'
```

原因是目录所有者与当前用户不一致（管理员 vs 普通用户）。解决：

```bash
git config --global --add safe.directory E:/blog
```

---

## 总结

Butterfly 主题的配置项覆盖面很广，大部分功能只需在 `_config.butterfly.yml` 中开启对应字段即可，不需要改主题源码。自定义样式通过 `inject` 注入独立 CSS 文件，升级主题时不会丢失。整套流程跑通后，写文章只需 push 到 main，其余全部自动化。
