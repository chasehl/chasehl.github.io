---
title: skill概念理解、使用与创建
date: 2026-03-12 21:36:10
tags:
---

# Skill 详解：概念、使用与创建

> 以 OpenClaw / Claude 平台为例，记录 Skill 的完整使用方式，供个人学习与备忘。

---

## 目录

1. [什么是 Skill](#一什么是-skill)
2. [Skill 能做什么：具体举例](#二skill-能做什么具体举例)
3. [Skill 的文件结构](#三skill-的文件结构)
4. [如何使用 Skill（ClawHub 安装与管理）](#四如何使用-skillclawhub-安装与管理)
5. [如何创建自己的 Skill](#五如何创建自己的-skill)
6. [OpenClaw 实战举例](#六openclaw-实战举例)
7. [安全注意事项](#七安全注意事项)
8. [常用命令速查](#八常用命令速查)

---

## 一、什么是 Skill

### 一句话定义

**Skill（技能）是一段给 AI 看的"操作说明书"**，它告诉 AI 在遇到特定任务时应该怎么做、按什么步骤做、输出什么格式。

### 类比理解

可以把 AI 理解成一个能力很强但什么都不专精的新员工。

- **没有 Skill**：你每次都要手动告诉它"帮我搜索这个关键词，用这个 API，返回这个格式……"
- **有了 Skill**：你只需要说"帮我搜索一下佛山天气"，AI 自动知道该调用哪个工具、怎么处理结果、用什么格式回复你

Skill 本质上是**将复杂的、重复的任务流程提前写好**，让 AI 每次遇到类似任务时直接按方案执行，不需要重新"教"它。

### 和"插件"有什么区别？

| 概念           | 本质                                 | 谁来写         |
| -------------- | ------------------------------------ | -------------- |
| 插件（Plugin） | 一段可执行代码，扩展软件功能         | 开发者         |
| Skill          | 一份 Markdown 说明文档，指导 AI 行为 | 任何人都可以写 |

Skill 不是代码插件，**核心文件是一个 Markdown 文本文件**（`SKILL.md`），普通用户也能创建。

---

## 二、Skill 能做什么：具体举例

### 例 1：天气查询（weather）

**没有 Skill 时：**

```
用户：佛山今天天气怎么样？
AI：我没有实时数据，无法告诉你当前天气。
```

**安装 weather Skill 后：**

```
用户：佛山今天天气怎么样？
AI：（自动调用天气 API）佛山今日多云，气温 22~28°C，东南风 3 级，
    体感舒适，无需带伞。
```

---

### 例 2：网页/PDF 摘要（summarize）

**没有 Skill 时：**

需要手动把内容复制粘贴给 AI，再告诉它"帮我总结一下"。

**安装 summarize Skill 后：**

```
用户：帮我总结这篇文章 https://example.com/long-article
AI：（自动抓取网页、提取正文、生成摘要）
    核心观点：……
    关键数据：……
    结论：……
```

---

### 例 3：实时搜索（tavily-search）

```
用户：马斯克今天发了什么推特？
AI：（调用 Tavily 搜索 API，检索最新内容）
    根据今日搜索结果：……
```

---

### 例 4：邮件管理（himalaya）

```
用户：帮我查一下有没有新邮件
AI：（通过 IMAP 协议连接邮箱，读取未读邮件列表）
    你有 3 封未读邮件：
    1. [发件人] 主题 …… (10分钟前)
    ……
```

---

## 三、Skill 的文件结构

一个完整的 Skill 由以下文件组成：

```
skill-name/
├── SKILL.md          ← 核心文件（必须），AI 的"操作说明书"
├── scripts/          ← 可选，可执行脚本（处理复杂逻辑）
├── references/       ← 可选，参考文档（AI 按需读取）
└── assets/           ← 可选，附带资源（模板、图标等）
```

### SKILL.md 的结构

`SKILL.md` 是 Skill 最重要的文件，分为两部分：

```markdown
---
name: skill-name
description: 这里描述这个 Skill 做什么、什么时候触发。
             AI 会读这段描述来决定要不要使用这个 Skill。
---

# Skill 正文

这里写具体的操作步骤、注意事项、输出格式要求……
（普通 Markdown 格式，AI 会按照这里的说明行动）
```

**`description` 字段非常关键**：AI 是根据 description 来判断当前任务是否需要调用这个 Skill 的。描述写得越准确，触发越精准。

### 三级加载机制

Skill 采用按需加载，避免占用过多 AI 上下文：

```
第 1 级：name + description   ← 始终在 AI 视野里（约 100 字）
第 2 级：SKILL.md 正文        ← Skill 触发时才加载（建议 500 行以内）
第 3 级：scripts / references ← 需要时才读取（大小不限）
```

---

## 四、如何使用 Skill（ClawHub 安装与管理）

ClawHub 是 OpenClaw 的技能市场，类似于手机的"应用商店"。

官网：[https://clawhub.com](https://clawhub.com)

### 安装 ClawHub CLI

```bash
npm i -g clawhub
```

### 常用管理命令

```bash
# 搜索技能
clawhub search "weather"
clawhub search "browser automation"

# 安装技能
clawhub install tavily-search
clawhub install summarize
clawhub install weather

# 查看已安装的技能列表
clawhub list

# 更新所有技能到最新版
clawhub update --all

# 卸载技能
clawhub uninstall <skill-name>
```

### Skill 的加载优先级

当多个位置存在同名 Skill 时，按以下顺序加载，**高优先级覆盖低优先级**：

```
优先级 1（最高）：工作区技能   <你的项目目录>/skills/
优先级 2         ：用户技能     ~/.openclaw/skills/
优先级 3（最低）：内置技能     随 OpenClaw 安装包自带
```

> 💡 **实用技巧**：想修改某个公开 Skill 的行为？把它复制到你的项目 `skills/` 目录下改，不影响原版，也不用等官方更新。

### 登录 ClawHub（下载限速解除）

未登录时下载 Skill 有频率限制（几分钟只能下一个），登录后解除限制：

```bash
npx clawhub@latest login
```

---

## 五、如何创建自己的 Skill

任何人都可以创建 Skill，不需要会编程，核心就是写一个 `SKILL.md` 文件。

### 创建流程

#### 第 1 步：明确你要解决的问题

在写 Skill 之前，先想清楚三件事：

1. **这个 Skill 要让 AI 做什么？**（目标）
2. **用户说什么话时应该触发？**（触发条件）
3. **期望 AI 输出什么格式？**（输出规范）

#### 第 2 步：创建 Skill 文件夹

```bash
# 在你的项目 skills 目录下创建
mkdir -p skills/my-skill
```

#### 第 3 步：编写 SKILL.md

```markdown
---
name: my-skill
description: |
  这个 Skill 用于做 XXX。
  当用户说"帮我XXX"、"查询XXX"、"处理XXX文件"时触发。
  适合 XXX 场景，能够自动完成 XXX 步骤。
---

# My Skill

## 功能说明

这个 Skill 的主要功能是……

## 操作步骤

1. 首先，……
2. 然后，……
3. 最后，以以下格式输出结果：

## 输出格式

\`\`\`
标题：xxx
内容：xxx
来源：xxx
\`\`\`

## 注意事项

- 如果遇到 XX 情况，应该……
- 不要……
```

#### 第 4 步：测试 Skill

把 `SKILL.md` 放到项目 `skills/` 目录后，重启 OpenClaw 服务，然后向 AI 发送触发语句，观察是否按预期执行：

```bash
openclaw gateway restart
```

#### 第 5 步：发布到 ClawHub（可选）

如果想分享给其他人使用，可以打包发布：

```bash
# 打包成 .skill 文件
npm run package-skill skills/my-skill

# 按 ClawHub 官网引导上传发布
```

---

### 一个完整的创建示例：「每日简报」Skill

**需求**：每天早上问 AI "今天有什么值得关注的新闻？"，AI 自动搜索并用固定格式输出简报。

```markdown
---
name: daily-brief
description: |
  生成每日信息简报。当用户说"今天有什么新闻"、"帮我出一份早报"、
  "今日简报"、"最近有什么大事"时触发。
  会自动搜索科技、财经、AI 领域的最新动态，整理成简洁的摘要格式。
---

# Daily Brief

## 执行步骤

1. 使用搜索工具，依次搜索以下关键词：
   - "今日科技新闻"
   - "今日 AI 进展"
   - "今日财经要闻"

2. 从结果中筛选出最近 24 小时内的内容

3. 按以下格式输出：

## 输出格式

```

📅 今日简报 - {日期}

🤖 AI 动态
• [标题]：[一句话摘要]
• [标题]：[一句话摘要]

💻 科技要闻
• [标题]：[一句话摘要]

💰 财经简讯
• [标题]：[一句话摘要]

---

数据来源：[搜索结果来源]

```
## 注意

- 每个分类最多列 3 条，保持简洁
- 摘要不超过 30 字
- 如果某分类没有近 24 小时的新内容，注明"暂无最新动态"
```

---

## 六、OpenClaw 实战举例

### 场景：搭建一个「技术助手机器人」

目标：在飞书里 @ 机器人，让它能帮你搜索技术文档、总结文章、查询天气。

#### 安装需要的 Skills

```bash
# 实时搜索（最重要，解锁联网能力）
clawhub install tavily-search

# 文章/网页摘要
clawhub install summarize

# 天气查询
clawhub install weather
```

#### 配置 Tavily API Key

```bash
# 去 https://tavily.com 注册，获取免费 API Key
clawhub config set TAVILY_API_KEY 你的TavilyAPIKey
```

#### 重启服务让 Skills 生效

```bash
openclaw gateway restart
```

#### 效果演示

```
# 在飞书中 @ 机器人：

用户：帮我搜索一下 Hexo butterfly 主题怎么配置深色模式

AI：（调用 tavily-search Skill，搜索后返回）
    根据搜索结果，Butterfly 主题配置深色模式的方法如下：
    在 _config.butterfly.yml 中添加：
    dark_mode:
      enable: true
      default: light
    ……

---

用户：总结一下这篇文章 https://example.com/article

AI：（调用 summarize Skill，抓取并总结）
    文章核心观点：……
    主要数据：……
    结论：……

---

用户：广州明天天气

AI：（调用 weather Skill）
    广州明日：晴，26~32°C，南风 2 级，紫外线较强，建议防晒。
```

### 自定义一个「代码审查」Skill（放入项目目录）

项目目录结构：

```
my-openclaw-project/
└── skills/
    └── code-review/
        └── SKILL.md
```

`SKILL.md` 内容：

```markdown
---
name: code-review
description: |
  对用户提交的代码进行审查。当用户说"帮我看看这段代码"、
  "代码 review"、"检查一下代码"、"有没有 bug" 时触发。
  会检查代码规范、潜在 bug、性能问题，并给出改进建议。
---

# Code Review Skill

## 审查维度

对收到的代码，依次从以下维度分析：

1. **正确性**：逻辑是否正确，有无明显 bug
2. **安全性**：有无 SQL 注入、XSS 等安全隐患
3. **性能**：有无不必要的循环、重复计算
4. **可读性**：命名是否清晰，注释是否充分
5. **规范性**：是否符合该语言的通用编码规范

## 输出格式

```

## 代码审查报告

**总体评分**：x/10

### 🐛 发现的问题

| 行号 | 严重程度 | 问题描述 | 修改建议 |
| ---- | -------- | -------- | -------- |
| L12  | 🔴 严重   | ……       | ……       |
| L25  | 🟡 警告   | ……       | ……       |

### ✅ 改进后的代码

（给出修改后的完整代码片段）

### 💡 其他建议

……

```

```

---

## 七、安全注意事项

> ⚠️ **2025 年末 ClawHavoc 供应链攻击事件**：ClawHub 曾出现大规模恶意 Skill 投毒，341~1184 个 Skill 受影响，感染率约 12%，主要伪装成加密货币工具和 YouTube 工具。

### 安装 Skill 前必须检查

- ✅ **Security Scan 为绿色 "Benign"**（ClawHub 页面上的安全扫描结果）
- ✅ **VirusTotal 扫描通过**（ClawHub 页面有展示）
- ✅ **高星数 + 高安装量**（社区验证过的）
- ✅ **阅读 SKILL.md 源码**，检查有无可疑的网络请求或"前置条件"
- ❌ **避免安装低星、新上架、无人维护的 Skill**，尤其是涉及钱包、密钥的

### 高风险 Skill 类型

| 类型                  | 风险原因                   |
| --------------------- | -------------------------- |
| 加密货币相关          | 可能窃取钱包私钥           |
| 邮件类（如 himalaya） | 需要存储邮箱账号密码       |
| 浏览器自动化          | 可能读取 Cookie 和账号信息 |
| 文件系统操作          | 可能读取或删除本地文件     |

---

## 八、常用命令速查

### ClawHub 技能管理

```bash
npm i -g clawhub                      # 安装 ClawHub CLI
npx clawhub@latest login              # 登录（解除下载限速）
clawhub search "关键词"               # 搜索技能
clawhub install <skill-name>          # 安装技能
clawhub list                          # 查看已安装列表
clawhub update --all                  # 更新所有技能
clawhub uninstall <skill-name>        # 卸载技能
clawhub config set KEY value          # 配置 API Key 等参数
```

### OpenClaw 服务管理

```bash
openclaw gateway restart              # 重启服务（安装新 Skill 后执行）
openclaw status                       # 查看运行状态
openclaw logs --tail 100              # 查看日志（排查问题）
```

### Skill 文件位置

```
工作区技能（最高优先级）：<你的项目目录>/skills/
用户技能：                ~/.openclaw/skills/
内置技能（最低优先级）：  随 OpenClaw 安装包自带
```
