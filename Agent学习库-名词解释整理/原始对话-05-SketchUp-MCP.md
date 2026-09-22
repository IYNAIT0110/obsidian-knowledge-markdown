---
title: 添加 SketchUp MCP 原始对话
tags:
  - 原始对话
  - MCP
  - SketchUp
  - Node.js
  - Python
source_thread: 添加SketchUp MCP
source_date: 2026-09-22
source_turns: 3
---

# 添加 SketchUp MCP 原始对话

> [!note] 存档范围
> 收录“添加SketchUp MCP”对话中与 MCP、SketchUp 接入、Node、npm、npx、Python、uv 和 uvx 有关的全部问答。最后一轮在源对话导出时尚无 AI 回答，因此只保留用户原问。经整理和核对的学习笔记见 [[12-MCP原理与SketchUp接入]]。

## 对话 1

### 用户

我应该怎么添加su的mcp

[User attached 1 image; image contents were not included]

### AI

可以。你这里的 **SU 我理解为 SketchUp**。你截图里的 D5 Arco 这个窗口就是用来添加 **STDIO 类型 MCP Server** 的。

目前有一个比较适合你这个界面的 SketchUp MCP：`@parkhill/mcp-server-for-sketchup`。它通过 **MCP → 本地 TCP → SketchUp Ruby 插件 → SketchUp** 工作，支持创建/修改模型等操作。:chatgpt-content-reference{index="0"}

### 你在这个窗口里这样填

| D5 Arco 项目 | 填写 |
|---|---|
| **连接名称** | `SketchUp` |
| **类型** | `STDIO` |
| **启动命令** | `npx` |
| **参数** | `-y` |
| **参数** | `@parkhill/mcp-server-for-sketchup` |
| **环境变量** | 暂时不用填 |

也就是最终相当于：

```text
连接名称：SketchUp

类型：STDIO

启动命令：
npx

参数：
-y
@parkhill/mcp-server-for-sketchup
```

然后点击**保存**。

---

## 但有一个非常重要的地方

**光在 D5 Arco 里面添加 MCP 是不够的。**

SketchUp MCP 基本都是这种结构：

```text
             D5 Arco
                │
                │ MCP / STDIO
                ▼
       SketchUp MCP Server
                │
                │ TCP 127.0.0.1:9876
                ▼
        SketchUp MCP 插件
                │
                ▼
           SketchUp
                │
                ▼
           你的 .skp
```

所以你还需要给 **SketchUp 本体安装一个 MCP 插件**。

这个 `@parkhill/mcp-server-for-sketchup` 项目的说明也明确要求 SketchUp 中安装对应的 `su_mcp` 扩展；默认通过 `127.0.0.1:9876` 通信。:chatgpt-content-reference{index="1"}

### 第一步：给 SketchUp 安装 MCP 插件

需要下载这个项目的 `.rbz` 插件，然后在 SketchUp：

**Window → Extension Manager → Install Extension**

安装 `.rbz`。

安装以后，SketchUp 菜单里应该出现类似：

```text
Extensions
   └── MCP Server
         └── Start Server
```

然后启动它。

---

### 第二步：确认 SketchUp MCP Server 已经启动

正常情况下它会监听：

```text
127.0.0.1:9876
```

也就是说：

```text
SketchUp
   ↑
   │ 9876
   │
MCP Server
```

这个端口只在你自己电脑本机使用。项目文档也建议保持本地回环地址，不要把这个端口暴露到网络。:chatgpt-content-reference{index="2"}

---

### 第三步：再回到 D5 Arco

你的截图里：

**连接 → 添加 MCP**

填写：

```text
连接名称
SketchUp

类型
STDIO

启动命令
npx

参数
-y
@parkhill/mcp-server-for-sketchup
```

然后保存。

如果 D5 Arco 成功启动 MCP，应该能看到类似：

```text
SketchUp
● 已连接
```

以及一系列 MCP Tools。

---

## ⚠️ 还有一个问题和你这台电脑有关

这个 MCP 要求：

> **Node.js 20 或更高版本**

这是 `@parkhill/mcp-server-for-sketchup` 当前 npm 包给出的要求。:chatgpt-content-reference{index="3"}

而你之前电脑上出现过：

```powershell
node -v
```

提示 `node` 找不到，并且你之前安装 Node.js LTS 时遇到过 **1603 安装失败**。

所以**如果你现在 Node 还没有正常安装，这个 npx 方法会直接失败。**

你可以先在 PowerShell 执行：

```powershell
node -v
npm -v
npx -v
```

如果三个都有版本号，例如：

```text
v22.x.x
10.x.x
10.x.x
```

那么就可以直接按照上面的 D5 Arco 配置。

---

## 还有一个选择：不用 npx

其实你这个 D5 Arco 的 **STDIO 并不意味着一定要用 npx**。

目前另一个 SketchUp MCP 是 Python/`uvx` 方案，例如 `sketchup-mcp2`：

```text
启动命令：
uvx

参数：
sketchup-mcp2
```

它同样通过 SketchUp 插件的 `127.0.0.1:9876` 与 SketchUp 通信。:chatgpt-content-reference{index="4"}

所以你实际上有：

```text
方案 A：Node
D5 Arco
  ↓
npx
  ↓
@parkhill/mcp-server-for-sketchup
  ↓
SketchUp 插件
```

或者：

```text
方案 B：Python
D5 Arco
  ↓
uvx
  ↓
sketchup-mcp2
  ↓
SketchUp 插件
```

**考虑到你之前 Node 安装出过问题，我反而建议我们先检查你电脑现在到底有没有 Node / uv，再决定用哪一个。**

你现在直接把下面三个命令的结果发给我：

```powershell
node -v
npm -v
uv --version
```

我可以根据你的结果，直接告诉你**这个 D5 Arco 窗口每一栏具体填什么**，以及 **SketchUp 的 `.rbz` 插件应该装哪一个**。

---

## 对话 2

### 用户

Node / uv分别是什么

### AI

简单理解：**Node 和 uv 都是“运行其他程序的工具”**，但服务于不同的编程生态。

你现在是在配置 **D5 Arco → SketchUp MCP**，所以会碰到它们。

### 1. Node.js 是什么？

**Node.js = 让电脑能够运行 JavaScript 程序的运行环境。**

以前你可能认为：

> JavaScript = 浏览器里的网页代码

Node.js 出现以后，JavaScript 也可以直接在电脑上运行。

例如：

```text
JavaScript程序
      ↓
    Node.js
      ↓
Windows电脑执行
```

所以很多 AI 工具、MCP Server、CLI 工具都是用 Node.js 写的。

---

### 2. npm 和 npx 又是什么？

它们通常跟 Node.js 一起出现：

```text
Node.js
├── npm
└── npx
```

**npm**：负责安装 JavaScript/Node.js 软件包。

例如：

```powershell
npm install xxx
```

意思类似：

> “帮我把 xxx 这个程序安装到电脑上。”

**npx**：可以理解成：

> “帮我临时找到并运行这个 Node.js 程序。”

例如我们刚才说的：

```powershell
npx @parkhill/mcp-server-for-sketchup
```

意思大致是：

```text
npx
 ↓
找到 @parkhill/mcp-server-for-sketchup
 ↓
启动它
 ↓
它作为 MCP Server 运行
```

所以你在 D5 Arco 里面看到：

```text
启动命令：npx
参数：
-y
@parkhill/mcp-server-for-sketchup
```

实际上就是让 D5 Arco：

> **启动 npx，然后让 npx 启动 SketchUp MCP。**

---

# 3. 那 uv 是什么？

**uv 是 Python 世界里的一个高速工具。**

如果 Node.js 是：

```text
JavaScript / Node.js 生态
```

那么 uv 主要对应：

```text
Python 生态
```

它可以帮你：

- 安装 Python
- 管理 Python 版本
- 安装 Python 包
- 创建 Python 虚拟环境
- 直接运行 Python 工具

因此可以粗略理解成：

```text
Node.js 世界              Python 世界

Node.js                    Python
  │                          │
 npm                        pip
  │                          │
 npx                        uv
```

不过这个表只是方便理解，**uv 实际上比 pip 更综合**，不完全是“一一对应”。

---

# 4. 为什么 MCP 会需要这些东西？

这其实是你现在最应该理解的地方。

**MCP 本身不是一个软件。**

MCP 更像是一套“让 AI 调用外部工具的通信标准”。

例如：

```text
D5 Arco
   │
   │ MCP
   ↓
SketchUp MCP Server
   │
   ↓
SketchUp
```

但是这个 **SketchUp MCP Server 本身也得是一个真正运行起来的程序**。

这个程序可能是：

```text
JavaScript 写的
       ↓
Node.js 运行
```

也可能是：

```text
Python 写的
       ↓
Python / uv 运行
```

所以你会看到不同 MCP 的启动方式：

### Node MCP

```text
启动命令：npx

参数：
@xxx/xxx-mcp
```

### Python MCP

可能是：

```text
启动命令：uvx

参数：
xxx-mcp
```

---

# 5. 放到你的 SketchUp 场景里

你现在可以把整个系统理解成：

```text
                 ┌──────────────┐
                 │   D5 Arco    │
                 │   AI Agent   │
                 └──────┬───────┘
                        │
                       MCP
                        │
                ┌───────▼────────┐
                │  MCP Server     │
                │                 │
                │ Node / Python   │
                └───────┬─────────┘
                        │
                        │
                ┌───────▼────────┐
                │    SketchUp     │
                │                 │
                │   你的模型      │
                └─────────────────┘
```

其中：

**Node.js / uv 并不是 SketchUp。**

它们只是负责：

> **把 MCP Server 这个程序跑起来。**

---

# 6. 为什么我刚才让你检查 Node 和 uv？

因为你现在不知道应该选择哪一种 SketchUp MCP。

例如：

**如果这个 MCP 是 Node 写的：**

```text
D5 Arco
 ↓
npx
 ↓
SketchUp MCP
```

你的电脑就需要能够运行：

```powershell
node
npm
npx
```

如果它是 Python MCP：

```text
D5 Arco
 ↓
uvx
 ↓
SketchUp MCP
```

你的电脑就需要有：

```powershell
uv
uvx
```

---

### 你可以先记住这一张表

| 名称 | 属于什么 | 主要作用 |
|---|---|---|
| **Node.js** | JavaScript | 运行 JS 程序 |
| **npm** | Node.js | 安装 JS 软件包 |
| **npx** | Node.js | 运行 JS 软件包 |
| **Python** | Python | 运行 Python 程序 |
| **pip** | Python | 安装 Python 软件包 |
| **uv** | Python | 更现代的 Python 环境/包管理工具 |
| **uvx** | uv | 直接运行 Python 工具 |
| **MCP** | 协议/标准 | 让 AI 调用外部工具 |

所以你现在这个问题，本质上就是：

> **“D5 Arco 要启动 SketchUp MCP，而这个 MCP 到底是用 Node 还是 Python 跑？”**

这也是为什么你刚才看到我让你检查：

```powershell
node -v
npm -v
uv --version
```

**如果你愿意，:chatgpt-content-reference{index="0"}。**

---

## 对话 3

### 用户

可以，并且把mcp全称告诉我

> [!info] 本轮没有 AI 回答
> 源对话截至导出时只记录了用户提问。

---

