---
title: openclaw使用方法备忘
date: 2026-03-12 00:32:51
tags:
---

# OpenClaw 使用方法记录

> 记录个人使用 OpenClaw 的完整过程，涵盖云服务器选购、安装配置、大模型接入、Channel 接入及技能管理，供个人备忘或博客参考。

---

## 目录

1. [云服务器选购与配置](#一云服务器选购与配置)
2. [安装 OpenClaw](#二安装-openclaw)
3. [大模型 API 接入](#三大模型-api-接入)
4. [接入 Channel](#四接入-channel)
   - [接入飞书](#接入飞书)
   - [接入 QQ 机器人](#接入-qq-机器人)
5. [Web UI 访问（SSH 隧道）](#五web-ui-访问ssh-隧道)
6. [ClawHub 登录问题](#六clawhub-登录问题)
7. [无桌面 Linux 服务器使用浏览器（X11 转发）](#七无桌面-linux-服务器使用浏览器x11-转发)
8. [OpenClaw 技能管理（Skills）](#八openclaw-技能管理skills)

---

## 一、云服务器选购与配置

**优点：** 可以 24 小时在线，不影响本地电脑性能，操作相对安全隔离

**缺点：** 操作便利性不如本地；无桌面环境时，浏览器调用、Web UI 访问、网页登录验证都比较麻烦

### 推荐选择海外节点

海外服务器可以直接访问外网，调用 OpenAI、Anthropic 等国外大模型无需额外代理。

> ⚠️ 缺点：海外节点普遍比国内贵；免费套餐通常需要 Visa 或信用卡验证。

### 最低配置建议

| 配置项 | 最低要求      | 推荐配置              |
| ------ | ------------- | --------------------- |
| CPU    | 2 核          | 2 核                  |
| 内存   | 2 GB          | 4 GB                  |
| 系统   | Ubuntu 20.04+ | Ubuntu 22.04 LTS      |
| 网络   | 有公网 IP     | 独立公网 IP（非 NAT） |

> 💡 **NAT 转发 IP 的坑**：部分廉价云服务器使用 NAT 公网 IP，端口映射受限，可能导致 WebSocket 长连接不稳定、ClawHub 登录回调失败等问题（本人踩坑记录）。

### 本次测试使用

- **服务商**：[沛千云](https://www.peiqianyun.com/)（4 元/月测试机）
- **系统**：Ubuntu（无桌面版）
- **注意**：无桌面系统无法直接调用浏览器，详见[第七节](#七无桌面-linux-服务器使用浏览器x11-转发)

---

## 二、安装 OpenClaw

> **官方文档**：[https://docs.openclaw.dev](https://docs.openclaw.dev)（请以最新官方文档为准）

### 前置依赖

确保服务器已安装 Node.js 18+ 和 npm：

```bash
# 检查版本
node --version   # 需要 v18.x.x 或更高
npm --version
```

如未安装，执行以下命令安装 Node.js LTS（Ubuntu）：

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### 安装 OpenClaw

```bash
npm install -g openclaw@latest
```

验证安装：

```bash
openclaw --version
```

### 初始化配置

安装完成后，执行以下命令进行初始化引导：

```bash
openclaw onboard --install-daemon
```

该命令会自动：

- 检测运行环境
- 安装后台守护进程（daemon）
- 引导填写大模型 API Key 等初始配置

按照终端提示逐步完成配置即可。

---

## 三、大模型 API 接入

OpenClaw 需要通过 API Key 调用大模型。

### 国内可用服务

| 服务商                 | 特点                           | 官网                                                         |
| ---------------------- | ------------------------------ | ------------------------------------------------------------ |
| 阿里云百炼（通义千问） | 免费额度多，支持多模型         | [https://bailian.console.aliyun.com](https://bailian.console.aliyun.com) |
| 火山引擎（豆包）       | Coding Plan 专项套餐，性价比高 | [https://www.volcengine.com](https://www.volcengine.com)     |
| DeepSeek               | 价格极低，代码能力强           | [https://platform.deepseek.com](https://platform.deepseek.com) |
| 月之暗面（Kimi）       | 长上下文能力强                 | [https://platform.moonshot.cn](https://platform.moonshot.cn) |

### 本次使用

**火山引擎 Coding Plan**（¥8.9/月）进行测试，适合代码类任务，性价比较高。

获取 API Key 步骤：

1. 登录 [https://console.volcengine.com/ark](https://console.volcengine.com/ark)
2. 进入「API Key 管理」，创建并复制 API Key
3. 在 `openclaw onboard` 引导步骤中填入，或后续通过配置文件修改

---

## 四、接入 Channel

OpenClaw 支持接入飞书、Telegram、QQ、企业微信、钉钉等 IM 平台，实现通过聊天软件直接与 AI 交互，方便日常使用。

---

### 接入飞书

#### 飞书开放平台侧配置

##### 1. 创建企业自建应用

登录 [https://open.feishu.cn/app](https://open.feishu.cn/app)，点击「创建企业自建应用」（注意：选「企业自建应用」而非「商店应用」），填写应用名称和描述。

![飞书创建企业自建应用](https://pic3.zhimg.com/v2-304b909ecfaa36a7a48fb39702505cd8_1440w.jpg)

##### 2. 获取应用凭证

进入应用详情页，在「凭证与基础信息」中记录 **App ID** 和 **App Secret**，后续配置 OpenClaw 时需要用到。

![获取 App ID 和 App Secret](https://pica.zhimg.com/v2-2747e6ec8f9c34e831cc007dd54b33fe_1440w.jpg)

##### 3. 添加机器人能力

进入「应用能力」→「添加应用能力」→「按能力添加」，选择【机器人】并点击配置。

![添加机器人能力](https://pic3.zhimg.com/v2-bce7cef95f071c6032e6387191b2361e_1440w.jpg)

##### 4. 设置事件订阅为长连接（WebSocket）

在「事件与回调」→「事件配置」中，将订阅方式改为**长连接**，并添加「接收消息」订阅事件。

> 💡 使用长连接模式无需公网 IP 和域名，服务器主动建立 WebSocket 连接到飞书服务器，对国内服务器更友好。

![设置长连接事件订阅 1](https://pic2.zhimg.com/v2-2d172519e54ed320a87a56094d268fad_1440w.jpg)

![设置长连接事件订阅 2](https://pic2.zhimg.com/v2-358b6de9ea88ee21a5d16a6232caa95f_1440w.jpg)

![添加「接收消息」事件](https://pica.zhimg.com/v2-12c61ce934e90f2d2a63967a7fb4b09a_1440w.jpg)

##### 5. 配置长连接回调

切换到右侧「回调配置」Tab，同样设为长连接，并添加回调「卡片回传交互」，用于处理卡片按钮点击等交互事件。

![回调配置长连接 1](https://pic4.zhimg.com/v2-549b063df1a55adb4e29899e4f62df21_1440w.jpg)

![回调配置长连接 2](https://pic4.zhimg.com/v2-4abb2e5d969b1c9def87feab0bcf3a7d_1440w.jpg)

##### 6. 批量导入权限

进入「权限管理」，清空默认内容，将以下权限配置 JSON **整体粘贴进去**，点击下一步确认。

> ✅ 以下权限均为**免审权限**，无需飞书管理员审核，可直接生效。

```json
{
  "scopes": {
    "tenant": [
      "application:bot.menu:write",
      "calendar:timeoff",
      "cardkit:card:write",
      "contact:contact.base:readonly",
      "contact:user.base:readonly",
      "directory:employee.base.background_image:read",
      "docs:document:import",
      "im:chat",
      "im:chat.access_event.bot_p2p_chat:read",
      "im:chat.announcement:read",
      "im:chat.announcement:write_only",
      "im:chat.managers:write_only",
      "im:chat.members:bot_access",
      "im:chat.members:read",
      "im:chat.members:write_only",
      "im:chat.menu_tree:read",
      "im:chat.menu_tree:write_only",
      "im:chat.moderation:read",
      "im:chat.tabs:read",
      "im:chat.tabs:write_only",
      "im:chat.top_notice:write_only",
      "im:chat.widgets:read",
      "im:chat.widgets:write_only",
      "im:chat:create",
      "im:chat:delete",
      "im:chat:moderation:write_only",
      "im:chat:operate_as_owner",
      "im:chat:read",
      "im:chat:readonly",
      "im:chat:update",
      "im:message",
      "im:message.group_at_msg:readonly",
      "im:message.group_msg",
      "im:message.p2p_msg:readonly",
      "im:message.pins:read",
      "im:message.pins:write_only",
      "im:message.reactions:read",
      "im:message.reactions:write_only",
      "im:message.urgent",
      "im:message.urgent.status:write",
      "im:message:readonly",
      "im:message:recall",
      "im:message:send_as_bot",
      "im:message:send_multi_depts",
      "im:message:send_multi_users",
      "im:message:send_sys_msg",
      "im:message:update",
      "im:resource",
      "im:url_preview.update",
      "im:user_agent:read",
      "optical_char_recognition:image"
    ],
    "user": [
      "directory:employee.base.background_image:read",
      "docs:document:import",
      "im:chat",
      "im:chat.access_event.bot_p2p_chat:read",
      "im:chat.announcement:read",
      "im:chat.announcement:write_only",
      "im:chat.managers:write_only",
      "im:chat.members:read",
      "im:chat.members:write_only",
      "im:chat.moderation:read",
      "im:chat.tabs:read",
      "im:chat.tabs:write_only",
      "im:chat.top_notice:write_only",
      "im:chat:delete",
      "im:chat:moderation:write_only",
      "im:chat:read",
      "im:chat:readonly",
      "im:chat:update",
      "im:message",
      "im:message.pins:read",
      "im:message.pins:write_only",
      "im:message.reactions:read",
      "im:message.reactions:write_only",
      "im:message.urgent.status:write",
      "im:message:readonly",
      "im:message:recall",
      "im:message:update"
    ]
  }
}
```

##### 7. 创建版本并发布

创建版本号（如 `v1.0.0`）并提交发布，等待审核通过（自建应用通常即时生效）。

![创建版本](https://picx.zhimg.com/v2-634f6752b40d4018f211fe9de040c4f1_1440w.jpg)

![发布应用](https://pic1.zhimg.com/v2-65dfa339de09bb824136be9bf5895ad2_1440w.jpg)

#### OpenClaw 侧配置

在 OpenClaw 中添加飞书 Channel，填入上面获取的 App ID 和 App Secret：

```bash
openclaw channels add --channel feishu --app-id "你的AppID" --app-secret "你的AppSecret"
```

重启服务生效：

```bash
openclaw gateway restart
```

---

### 接入 QQ 机器人

> 本方式使用原生方法（非腾讯云一键部署），适合使用非腾讯云服务器的用户。

##### 第 1 步：创建 QQ 机器人

访问 [https://q.qq.com](https://q.qq.com) —— QQ 开放平台，创建机器人应用。

记录好以下信息（后续配置需要用到）：

- **AppID**
- **AppSecret**

##### 第 2 步：安装 QQ Bot 插件

```bash
openclaw plugins install @tencent-connect/openclaw-qqbot@latest
```

##### 第 3 步：绑定 QQ 机器人

```bash
# 将 AppID:AppSecret 替换为你的实际凭证
openclaw channels add --channel qqbot --token "你的AppID:你的AppSecret"
```

##### 第 4 步：重启 OpenClaw 服务

```bash
openclaw gateway restart
```

验证是否接入成功：在 QQ 中 @ 机器人发送消息，观察是否有响应。

---

## 五、Web UI 访问（SSH 隧道）

OpenClaw 自带 Web UI 管理面板，默认监听 `127.0.0.1:18789`（仅本地访问）。

在无公网 IP 或不想暴露端口的情况下，可通过 **SSH 隧道**将云服务器的端口映射到本地访问。

### 建立 SSH 隧道

在**本地电脑**终端（Windows PowerShell / macOS Terminal）执行：

```bash
# 格式：ssh -L 本地端口:远端地址:远端端口 -p SSH端口 用户名@服务器IP
ssh -L 18789:127.0.0.1:18789 -p 10022 root@你的服务器IP
```

> 将 `10022` 替换为你服务器的 SSH 端口，`你的服务器IP` 替换为实际 IP。

隧道建立后，打开本地浏览器访问：

```
http://localhost:18789
```

即可看到 OpenClaw Web UI 管理面板。

### 保持隧道后台运行（可选）

```bash
# 在后台静默运行隧道（-N 不执行远程命令，-f 后台运行）
ssh -N -f -L 18789:127.0.0.1:18789 -p 10022 root@你的服务器IP
```

---

## 六、ClawHub 登录问题

### 为什么需要登录 ClawHub？

未登录状态下从 ClawHub 下载技能（Skills）有速率限制：几分钟内只能下载一个，频繁下载还会被封 IP，报错信息类似：

```
Error: Rate limit exceeded. Please login to download more skills.
```

### 登录方式

```bash
npx clawhub@latest login
```

正常情况下会自动打开浏览器完成 OAuth 登录。

### 问题：无桌面服务器登录失败

无桌面 Linux 服务器没有浏览器，执行登录命令会报错：

```
Error: spawn xdg-open ENOENT
```

**解决方案一：SSH 隧道 + 本地浏览器（推荐）**

先建立 SSH 隧道（见第五节），然后在命令中指定本地回调：

```bash
npx clawhub@latest login --no-browser
```

复制输出的授权 URL，在本地浏览器中打开完成登录。

**解决方案二：手动输入 Token（最终有效方案）**

如果 SSH 隧道方式也不成功（如服务器使用 NAT 转发 IP、端口受限），可以手动获取 Token 登录：

1. 在本地电脑上执行 `npx clawhub@latest login`，完成登录

2. 查看本地生成的 Token：

   ```bash
   # macOS / Linux 本地
   cat ~/.clawhub/token
   
   # Windows PowerShell 本地
   type $env:USERPROFILE\.clawhub\token
   ```

3. 在云服务器上手动写入 Token：

   ```bash
   mkdir -p ~/.clawhub
   echo "你复制的token内容" > ~/.clawhub/token
   ```

4. 验证登录状态：

   ```bash
   npx clawhub@latest whoami
   ```

> 📝 **踩坑记录**：上述 X11 和 SSH 隧道方式在本机测试均未成功，怀疑原因是云服务器使用了 NAT 转发公网 IP，端口映射存在限制。最终通过手动写入 Token 解决。

---

## 七、无桌面 Linux 服务器使用浏览器（X11 转发）

适用场景：需要在服务器上打开浏览器（如完成网页登录验证）时使用。

> ⚠️ 此方案需要 Windows 本地安装 [MobaXterm](https://mobaxterm.mobatek.net/download-home-edition.html)（免费版即可），其内置 X11 Server。

### 第 1 步：MobaXterm 开启 X11 转发

1. 打开 MobaXterm，右键编辑你的服务器 Session
2. 进入 **「Advanced SSH settings」** 标签页
3. 勾选 **「X11-forwarding」**（必须勾选）
4. 保存后**重新连接服务器**

> 连接成功后，MobaXterm 底部状态栏会显示 `X11-forwarding enabled`，说明 X Server 已启动。

### 第 2 步：服务器安装 Firefox

```bash
# Ubuntu / Debian
apt update && apt install -y firefox
```

验证安装：

```bash
firefox --version
# 输出类似：Mozilla Firefox 124.0
```

> 安装过程中若出现图形库需要重启的提示，按 Tab 键选择「OK」回车即可继续。

### 第 3 步：启动 Firefox（X11 转发到本地）

```bash
# root 用户必须加 --no-sandbox，否则会报权限错误
firefox --no-sandbox --display $DISPLAY
```

如果 X11 转发配置正确，**你的 Windows 本地桌面上会弹出一个 Firefox 窗口**，这个窗口实际运行在服务器上，但显示在本地屏幕。

> 💡 首次启动较慢，耐心等待 3-5 秒。

### 第 4 步：在弹出的 Firefox 中完成登录

浏览器弹出后，访问 ClawHub 或其他需要网页登录的地址，完成扫码或账号登录即可。

---

## 八、OpenClaw 技能管理（Skills）

> 参考：[https://www.cnblogs.com/informatics/p/19679935](https://www.cnblogs.com/informatics/p/19679935)

### ClawHub CLI 常用命令

```bash
# 安装 ClawHub CLI（如未安装）
npm i -g clawhub

# 搜索技能
clawhub search "browser automation"

# 安装技能
clawhub install <skill-slug>

# 查看已安装的技能列表
clawhub list

# 更新所有技能到最新版
clawhub update --all

# 卸载技能
clawhub uninstall <skill-slug>
```

### Skills 加载优先级

OpenClaw 按以下优先级加载技能，**高优先级同名技能会覆盖低优先级**：

```
优先级 1（最高）：工作区技能   <workspace>/skills/
优先级 2         ：用户技能     ~/.openclaw/skills/
优先级 3（最低）：内置技能     随 OpenClaw 安装包附带
```

> 💡 可以将任意 Skill Fork 到工作区目录（`<workspace>/skills/`）下进行自定义修改，不影响原版。

### ⚠️ 安全警告：ClawHavoc 供应链攻击事件

2025 年末至 2026 年初，ClawHub 曾遭遇大规模供应链投毒攻击（**ClawHavoc 事件**），安全公司 Koi Security 识别出 341~1184 个恶意 Skills，感染率约 12%。攻击者主要伪装成**加密货币工具、YouTube 工具**等热门类目投放恶意技能。

**安装任何 Skill 前，务必检查以下几项：**

- ✅ 优先选择**高星数、高安装量**的 Skills
- ✅ 确认 ClawHub 页面上 **Security Scan 结果为 "Benign"**（绿色标签）
- ✅ 查看 ClawHub 页面上的 **VirusTotal 扫描结果**
- ✅ 阅读 `SKILL.md` 源码，检查是否有可疑的「前置条件」或不明网络请求
- ✅ 使用 ClawdStrike 工具进行安全审计（详见 ClawHub 安全文档）

---

### 推荐技能列表

#### 🟢 低风险必装

| 技能名      | 功能                                                         | 安装命令                    |
| ----------- | ------------------------------------------------------------ | --------------------------- |
| `summarize` | 对网页、PDF、播客、长文进行智能摘要。40 分钟播客 20 秒出摘要，据用户反馈每周可节省 5 小时以上 | `clawhub install summarize` |
| `weather`   | 获取本地天气预报，免费无需 API Key，即装即用，配合每日简报效果佳 | `clawhub install weather`   |

#### 🔴 高风险（谨慎使用）

| 技能名     | 功能                                           | 说明                                                         |
| ---------- | ---------------------------------------------- | ------------------------------------------------------------ |
| `himalaya` | 通用 IMAP/SMTP 邮件收发，适合非 Gmail 邮箱用户 | 需要存储邮箱账号密码，存在凭证泄露风险，请仅在可信环境中使用 |

---

### 实时搜索：Tavily Search

**功能说明**：OpenClaw 默认知识有截止日期，安装此技能后 AI 可实时搜索全网信息（新闻、天气、最新动态等），是摆脱"幻觉"和知识过期的第一步。

**安装命令：**

```bash
clawhub install tavily-search
```

**API Key 配置：**

1. 访问 [https://tavily.com](https://tavily.com) 注册账号（免费额度对个人使用完全够用）
2. 在 Dashboard 中获取 API Key
3. 在终端配置：

```bash
clawhub config set TAVILY_API_KEY 你的TavilyAPIKey
```

**使用示例：**

```
用户：帮我查询一下佛山今天的天气。
用户：最近有哪些 AI 新闻？
用户：查一下比特币现在的价格。
```

---

## 附录：常用命令速查

```bash
# 查看 OpenClaw 版本
openclaw --version

# 查看运行状态
openclaw status

# 重启后台服务
openclaw gateway restart

# 查看已接入的 Channel 列表
openclaw channels list

# 查看日志（排查问题用）
openclaw logs --tail 100

# SSH 隧道（本地执行，访问 Web UI）
ssh -L 18789:127.0.0.1:18789 -p 10022 root@你的服务器IP

# ClawHub 登录
npx clawhub@latest login

# ClawHub 查看登录状态
npx clawhub@latest whoami
```
