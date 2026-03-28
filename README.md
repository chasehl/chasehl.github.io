# ChaseLog

> 记录折腾的地方 — 云服务器、AI 工具、技术学习

**Live:** [chasehl.github.io](https://chasehl.github.io)

---

## 技术栈

| 组件 | 版本 / 说明 |
|------|------------|
| 框架 | [Hexo](https://hexo.io) 8.x |
| 主题 | [Butterfly](https://butterfly.js.org) 5.x |
| 托管 | GitHub Pages（`gh-pages` 分支） |
| 部署 | GitHub Actions 自动构建 |
| 评论 | [Giscus](https://giscus.app)（基于 GitHub Discussions） |
| 搜索 | hexo-generator-searchdb（本地搜索） |
| 统计 | 不蒜子（UV / PV） |

---

## 项目结构

```
.
├── source/
│   ├── _posts/          # 博客文章（Markdown）
│   ├── _data/
│   │   └── link.yml     # 友链数据
│   ├── about/           # 关于页
│   ├── categories/      # 分类页
│   ├── tags/            # 标签页
│   ├── links/           # 友链页
│   ├── css/
│   │   └── custom.css   # 自定义样式
│   └── img/             # 图片资源
├── _config.yml          # Hexo 主配置
├── _config.butterfly.yml # Butterfly 主题配置
└── .github/
    └── workflows/
        └── deploy.yml   # 自动部署 CI
```

---

## 本地开发

**环境要求：** Node.js 18+

```bash
# 安装依赖
npm install

# 启动本地预览（http://localhost:4000）
npm run server

# 构建静态文件
npm run build

# 清理缓存
npm run clean
```

---

## 写文章

```bash
# 新建文章
npx hexo new post "文章标题"

# 文件生成在 source/_posts/文章标题.md
```

Front Matter 模板：

```yaml
---
title: 文章标题
date: 2026-03-28
categories:
  - 分类名
tags:
  - 标签1
  - 标签2
cover: /img/covers/1.jpg
---
```

写完后 push 到 `main` 分支，GitHub Actions 自动构建并发布，约 1 分钟生效。

---

## 部署流程

```
push to main
    └─► GitHub Actions (deploy.yml)
            ├── npm install
            ├── hexo clean && hexo generate
            └── 发布 ./public → gh-pages 分支
                    └─► GitHub Pages 对外提供访问
```

无需本地执行 `hexo deploy`，所有构建在 CI 中完成。

---

## 主要配置说明

| 配置项 | 位置 | 说明 |
|--------|------|------|
| 站点信息、URL、语言 | `_config.yml` | 基础 Hexo 配置 |
| 主题色、导航、侧边栏 | `_config.butterfly.yml` | Butterfly 主题功能 |
| 字体、卡片样式、深色模式 | `source/css/custom.css` | 自定义 CSS 覆盖 |
| 友链数据 | `source/_data/link.yml` | 与页面解耦，单独维护 |

---

## License

文章内容采用 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) 协议。
代码部分采用 [MIT](https://opensource.org/licenses/MIT) 协议。
