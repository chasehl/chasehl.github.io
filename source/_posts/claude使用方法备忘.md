---
title: claude使用方法备忘
date: 2026-03-12 00:31:56
tags:
cover: none
---

# Claude Code 使用指南

> 本文记录 Claude Code 在 Windows 系统下的三种接入方式，适合个人学习与工程使用参考。

---

## 方法一：官网直充（官方订阅）

**优点：** 体验最完整，使用官方原版模型，响应稳定，无中间商风险

**缺点：** 价格较贵，需要海外信用卡，国内网络可能受限

| 套餐        | 价格         | 适用场景               |
| ----------- | ------------ | ---------------------- |
| Claude Pro  | $20/月       | 普通开发者，日常使用   |
| Claude Max  | $100–$200/月 | 重度使用，高速率限制   |
| Console API | 按量计费     | 仅需 API，不需要网页端 |

**注册步骤：**

1. 访问 [https://claude.ai](https://claude.ai) 注册账号（建议使用 Gmail / Outlook 等国际邮箱）
2. 订阅 Pro 或 Max 套餐，需要 Visa / MasterCard 国际信用卡
3. 国内用户也可通过 [console.anthropic.com](https://console.anthropic.com) 创建 API Key，按量付费

---

## 方法二：使用中转站（推荐国内用户）

**优点：** 无需海外信用卡，无需翻墙，价格更低，操作简单

**缺点：** 存在数据隐私风险，稳定性依赖第三方服务，不适合传输敏感信息

> ⚠️ **安全提示**：禁止通过中转站传输含个人隐私、商业机密的内容。建议仅用于学习或个人开发项目。

常见中转站参考（以下均为第三方服务，请自行评估风险）：

- **AnyRouter**：[https://anyrouter.top](https://anyrouter.top)（GitHub 账号登录，有免费额度）
- **302.AI**：[https://api.302.ai](https://api.302.ai)（支持多模型中转）
- **AIHubMix**：[https://aihubmix.com](https://aihubmix.com)

---

### Windows 安装步骤（以中转站接入为例）

#### 第 1 步：安装 Node.js

Claude Code 的 npm 安装方式需要 **Node.js 18+**。

1. 访问官网下载 LTS 版本：[https://nodejs.org](https://nodejs.org)
2. 下载并运行安装包（默认选项即可）
3. 安装完成后，**关闭并重新打开 PowerShell**，验证安装：

```powershell
node --version   # 应输出 v18.x.x 或更高
npm --version    # 应输出对应 npm 版本
```

> ✅ 如果显示版本号，说明安装成功。

---

#### 第 2 步：安装 Git for Windows

Claude Code 在 Windows 上依赖 Git Bash 运行内部命令。

1. 访问官网下载：[https://git-scm.com/downloads/win](https://git-scm.com/downloads/win)
2. 运行安装包，安装选项保持默认即可
3. 验证安装（在 PowerShell 中运行）：

```powershell
git --version   # 应输出 git version x.x.x
```

---

#### 第 3 步：安装 Claude Code

打开 **PowerShell**（不需要管理员权限），运行以下命令：

```powershell
npm install -g @anthropic-ai/claude-code
```

> ⚠️ **注意**：不要使用 `sudo npm install -g`，会导致权限问题。

验证安装：

```powershell
claude --version
```

如果显示版本号，说明安装成功。

---

#### 第 4 步：配置中转站 API Key

从你使用的中转站获取 API Key（格式通常为 `sk-` 开头），然后配置环境变量。

**方式 A：通过 PowerShell 设置环境变量（永久生效）**

```powershell
# 替换为你的 API Key 和中转站地址
[Environment]::SetEnvironmentVariable("ANTHROPIC_AUTH_TOKEN", "sk-你的APIKey", [EnvironmentVariableTarget]::User)
[Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL", "https://anyrouter.top", [EnvironmentVariableTarget]::User)
```

设置完成后，**重新打开一个新的 PowerShell 窗口**，验证：

```powershell
echo $env:ANTHROPIC_AUTH_TOKEN
echo $env:ANTHROPIC_BASE_URL
```

**方式 B：通过配置文件设置（推荐，适合多项目管理）**

首先启动一次 Claude Code 让它自动创建 `.claude` 目录：

```powershell
claude
# 出现界面后按 Ctrl+C 退出
```

然后创建/编辑配置文件。路径为：`C:\Users\你的用户名\.claude\settings.json`

用记事本或 VS Code 打开该文件，写入以下内容：

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "sk-你的APIKey",
    "ANTHROPIC_BASE_URL": "https://anyrouter.top"
  }
}
```

---

#### 第 5 步：启动使用

在项目目录下打开 PowerShell，输入：

```powershell
claude
```

首次启动会出现交互引导，按提示操作即可。常用命令：

```
/help      # 查看所有命令
/config    # 查看当前配置
/model     # 切换模型
/clear     # 清空当前对话
/quit      # 退出
```

---

## 方法三：接入国内大模型

**优点：** 完全国内服务，无网络限制，延迟低，价格便宜，数据留在国内

**缺点：** 使用的不是原版 Claude，编程能力存在差异；部分接口兼容性不完美

目前支持 Anthropic API 兼容接口的国内大模型服务：

| 服务商                 | 推荐模型                           | Base URL                                        |
| ---------------------- | ---------------------------------- | ----------------------------------------------- |
| 阿里云百炼（通义千问） | `qwen3-coder-plus`、`qwen3.5-plus` | `https://dashscope.aliyuncs.com/apps/anthropic` |
| 月之暗面（Kimi K2）    | `kimi-k2-turbo-preview`            | `https://api.moonshot.cn/anthropic`             |
| DeepSeek               | `deepseek-chat`                    | `https://api.deepseek.com/anthropic`            |

---

### 以阿里云百炼为例（接入通义千问）

**第 1 步：获取 API Key**

1. 访问 [https://bailian.console.aliyun.com](https://bailian.console.aliyun.com)
2. 注册/登录阿里云账号，开通百炼服务
3. 进入 **API Key 管理** 页面，点击创建 API Key，复制保存

**第 2 步：配置环境变量（CMD 方式）**

打开 **CMD**（命令提示符），运行：

```cmd
setx ANTHROPIC_API_KEY "你的百炼APIKey"
setx ANTHROPIC_BASE_URL "https://dashscope.aliyuncs.com/apps/anthropic"
setx ANTHROPIC_MODEL "qwen3-coder-plus"
```

**或者通过 PowerShell：**

```powershell
[Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY", "你的百炼APIKey", [EnvironmentVariableTarget]::User)
[Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL", "https://dashscope.aliyuncs.com/apps/anthropic", [EnvironmentVariableTarget]::User)
[Environment]::SetEnvironmentVariable("ANTHROPIC_MODEL", "qwen3-coder-plus", [EnvironmentVariableTarget]::User)
```

重新打开终端，验证：

```powershell
echo $env:ANTHROPIC_API_KEY
echo $env:ANTHROPIC_BASE_URL
echo $env:ANTHROPIC_MODEL
```

**第 3 步：也可通过 settings.json 配置**

编辑 `C:\Users\你的用户名\.claude\settings.json`：

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "你的百炼APIKey",
    "ANTHROPIC_BASE_URL": "https://dashscope.aliyuncs.com/apps/anthropic",
    "ANTHROPIC_MODEL": "qwen3-coder-plus",
    "ANTHROPIC_SMALL_FAST_MODEL": "qwen-flash"
  }
}
```

> `ANTHROPIC_SMALL_FAST_MODEL` 用于指定处理简单任务时使用的快速模型，设置为较便宜的模型可以节省费用。

**第 4 步：启动 Claude Code**

```powershell
claude
```

对话中可用 `/model <模型名>` 命令临时切换模型。

---

## 常见问题

**Q：安装后输入 `claude` 提示 "command not found"？**

关闭当前终端，重新打开一个新的 PowerShell 窗口，再次尝试。这是因为 PATH 环境变量需要刷新。

**Q：在 Windows 上提示 "Unsupported OS"？**

使用 npm 安装方式时 Windows 需要配合 Git Bash 或 WSL。确认 Git for Windows 已安装，或考虑使用官方原生安装命令：

```powershell
irm https://claude.ai/install.ps1 | iex
```

**Q：中转站配置后提示认证失败？**

检查 `ANTHROPIC_AUTH_TOKEN` 和 `ANTHROPIC_BASE_URL` 是否设置正确，注意不同中转站有时要求用 `ANTHROPIC_API_KEY` 而非 `ANTHROPIC_AUTH_TOKEN`，参考对应平台文档。

**Q：如何诊断配置问题？**

```powershell
claude doctor
```

该命令会自动检测 Node.js 版本、认证状态、网络连接等常见问题。

---

## 参考链接

- Claude Code 官方文档：[https://docs.claude.com/en/docs/claude-code/overview](https://docs.claude.com/en/docs/claude-code/overview)
- 阿里云百炼 Claude Code 接入文档：[https://help.aliyun.com/zh/model-studio/claude-code](https://help.aliyun.com/zh/model-studio/claude-code)
- Node.js 官网：[https://nodejs.org](https://nodejs.org)
- Git for Windows：[https://git-scm.com/downloads/win](https://git-scm.com/downloads/win)
