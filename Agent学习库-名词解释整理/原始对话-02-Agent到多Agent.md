---
title: 原始对话 02：Agent 到多 Agent
tags:
  - 原始对话
  - Agent
  - Multica
  - Token
source_thread: 名词解释
source_turns: 14-29
---

# 原始对话 02：Agent 到多 Agent

> 按原始时间顺序存档，覆盖 Agent 运行位置、API、API Key、本地模型、多 Agent 分工、Token、Multica 与任务拆分。

## 对话 14

### 用户

所以agent的运行逻辑到底是什么？是在本地跑代码还是云端跑代码？为什么输入apikey其他软件就可以调用agent，其他软件会泄露我的apikey吗？

### AI

可以把 **Agent 的运行逻辑**拆成三层看，就很清楚了：

> **模型在哪跑**、**代码在哪跑**、**密钥放在哪**，这三件事不一定在同一个地方。

## 1. Agent 到底是在本地跑，还是云端跑？

答案是：**两种都可能，而且经常是“混合运行”。**

最典型的是这种结构：

```text
你的电脑
│
├─ Agent 客户端 / Codex / 第三方软件
│
├─ 本地项目文件
│   H:\GH-Agent
│
├─ 本地终端
│   dotnet build
│   npm test
│   git
│
└───────────────┐
                │ API 请求
                ↓
          云端 AI 模型
          OpenAI / Claude
                │
                ↓
          返回推理/代码结果
                │
                ↓
你的电脑继续：
修改文件
运行代码
测试
Git commit
```

所以很多 Coding Agent 实际上是：

> **AI 大脑在云端，代码执行环境在本地。**

例如 Agent 可以把你的代码片段发送给云端模型，请模型判断“下一步改什么”，然后本地 Agent 再执行：

```text
修改 TreeAnalyzer.cs
↓
dotnet build
↓
读取报错
↓
再次把报错发给模型
↓
继续修改
```

这就是一个循环。

但也有另一种：

```text
你的电脑
   ↓
云端 Agent 平台
   ↓
云端虚拟机 / Container
   ↓
clone GitHub Repo
   ↓
在云端修改 / 编译 / 测试
```

这时**代码也在云端执行**。

所以以后看到一个 Agent 工具，要先问：

> ① 模型在哪里？  
> ② 项目代码在哪里？  
> ③ Shell / 编译器在哪里执行？

这比单纯问“Agent 是本地还是云端”更准确。

---

# 2. 为什么输入 API Key，别的软件就能调用 AI？

因为 API Key 本质上就是一张：

> **“这个程序有权代表我调用 API” 的通行证。**

比如你有：

```text
OpenAI API Key
sk-xxxxxxxx
```

第三方软件调用模型时，大致做的是：

```text
第三方软件
   │
   │ 请求：
   │ “帮我分析这段 C# 代码”
   │
   │ + API Key
   ↓
OpenAI API
   │
   │ 验证：
   │ 这个 Key 是否有效？
   │ 有什么权限？
   ↓
模型处理
   ↓
返回答案
```

API 服务端看到这个 Key，就知道：

> “这是某个有权限的 API 凭据发来的请求。”

然后用对应账户/项目额度进行计费和权限检查。

所以 API Key 有点类似：

> **银行卡 + 门禁卡的结合体。**

别人只要拿到它，在它允许的权限范围内，就可能**以你的身份调用 API，并消耗你的额度**。OpenAI 也明确建议不要共享 API Key，并且可以给 Key 设置权限、轮换和限制。citeturn483159search0turn483159search7

---

# 3. 那第三方软件会不会泄露我的 API Key？

**有可能。**

这取决于这个软件怎么保存和处理你的 Key。

最安全的情况：

```text
你的 API Key
     ↓
只保存在你电脑本地
     ↓
第三方软件本地读取
     ↓
直接请求 OpenAI
```

例如：

```text
你的电脑
│
├─ Dashi / Agent 程序
│
├─ OPENAI_API_KEY 环境变量
│
└─────────────→ OpenAI
```

这种情况下，理论上第三方公司的服务器甚至**不需要看到你的 Key**。

但是也可能是：

```text
你的电脑
   ↓
把 API Key 发送到
   ↓
第三方服务器
   ↓
第三方服务器再请求 OpenAI
```

这时候第三方理论上就**能够接触你的 API Key**。

所以真正需要看的不是：

> “这个软件支持 OpenAI API Key 吗？”

而是：

> **“它把我的 Key 存在哪里？请求是从我的电脑直接发给 OpenAI，还是先经过它自己的服务器？”**

OpenAI 官方也明确提醒：对于要求你提供 API Key 的第三方软件、库和框架，应谨慎评估，因为即使工具看起来可靠，仍然存在泄露或滥用风险。citeturn483159search2turn483159search3

---

# 4. 有一种情况特别危险

比如网页直接要求你输入：

```text
OpenAI API Key:
[ sk-xxxxxxxxxxxxx ]
```

然后网页 JavaScript 直接保存。

如果它：

```text
localStorage.setItem("api_key", ...)
```

风险就比较高。

更严重的是：

```text
API Key
 ↓
被上传到第三方网站服务器
```

你实际上是在把“钥匙”交给别人。

OpenAI 特别建议**不要把 API Key 部署在浏览器或移动 App 等客户端环境里**，因为用户端暴露的 Key 可能被窃取和滥用。citeturn483159search0turn483159search4

---

# 5. 比较好的做法是什么？

你自己开发 Agent 时，推荐结构：

```text
Windows
│
├─ 环境变量
│
│  OPENAI_API_KEY=*****
│
├─ Codex / Agent
│       │
│       └─ 读取环境变量
│
└──────────────→ OpenAI API
```

代码里只写：

```python
api_key = os.environ["OPENAI_API_KEY"]
```

而不是：

```python
api_key = "sk-xxxxxxxxxxxxxxxx"
```

这样 GitHub 里面就不会有你的真实 Key。

OpenAI 目前仍然建议通过**环境变量或 Secret 管理工具**保存 API Key，并避免把 Key 写入源码或提交到 Git 仓库。citeturn483159search0turn483159search6

---

# 6. 这和 GitHub Private 有什么关系？

即使你的 GitHub 是：

```text
Private 🔒
```

也不要 Commit：

```text
.env
config.json
apikey.txt
```

里面如果包含：

```text
OPENAI_API_KEY=sk-xxxxx
```

因为：

> **Private Repo 降低了暴露概率，但不是 Secret Manager。**

更标准的是：

```text
项目
│
├─ .env             ← 本地，包含真正 Key
├─ .env.example     ← 可以进 Git
├─ .gitignore
└─ src/
```

`.gitignore`：

```text
.env
```

这样：

```text
真正 API Key
      ↓
只留在你电脑
      ✕
不会 push 到 GitHub
```

---

# 7. 对你现在这套系统，可以这么理解

如果以后你用：

```text
Dashi
+
Multica
+
Codex
+
Git Worktree
```

可能形成：

```text
                 AI 模型云端
              OpenAI / Claude
                     ↑
                     │ API
                     │
               API Key 验证
                     │
                     ↓
┌─────────────────────────────────┐
│            你的电脑              │
│                                 │
│   Multica / Codex Agent         │
│        │                        │
│        ├─ Agent A               │
│        │    ↓                   │
│        │  Worktree A            │
│        │    ↓                   │
│        │  dotnet build          │
│        │                        │
│        └─ Agent B               │
│             ↓                   │
│           Worktree B            │
│             ↓                   │
│           git / test            │
│                                 │
└─────────────────────────────────┘
```

这里可能是：

**模型推理：云端**

**代码文件：本地**

**编译运行：本地**

**Git：本地**

**GitHub：云端备份**

**API Key：最好只保存在本地安全环境**

---

# 8. 你以后看到第三方 Agent，可以先检查这 6 件事

| 要检查 | 理想情况 |
|---|---|
| API Key 存哪里 | 本地环境变量 / Secret Store |
| Key 会不会上传 | 最好不会经过第三方服务器 |
| 是否开源 | 能审查代码更好 |
| Key 权限 | 尽可能 Restricted |
| 是否设置消费限制 | 建议有预算/额度提醒 |
| GitHub 是否含 Key | 绝对不要 |

另外，如果某个 API Key 你怀疑已经泄露，最稳妥的处理不是“观察一下”，而是：

> **立即撤销旧 Key，生成新 Key。**

OpenAI 也建议怀疑泄露时进行 Key rotation，并设置合理的消费阈值。citeturn483159search2turn483159search3

---

你可以把整个 Agent 最后压缩成一句：

> **Agent 本质上是一个循环：读取环境 → 把问题交给模型判断 → 调用工具执行 → 读取执行结果 → 再交给模型判断 → 直到任务完成。**

而 **API Key** 就是第三方程序调用云端模型时的“授权钥匙”。

如果你接下来要用 **Multica / Dashi / 某个具体 Agent 工具**输入 API Key，我可以直接帮你判断它到底是**本地直连 OpenAI，还是 Key 会经过第三方服务器**，这样你就知道是否值得信任。

---

## 对话 15

### 用户

可以本地部署大模型吗？本地部署和云端模型有什么区别呢？

### AI

可以，而且现在**本地部署大模型已经非常成熟**。但“本地模型”和你现在用的 ChatGPT / Codex 云端模型，在能力、成本、隐私和硬件要求上差别很大。

你可以先记住一句：

> **本地模型 = 模型的大脑也下载到你的电脑里运行。**  
> **云端模型 = 你的电脑负责交互，真正的大模型运行在厂商的数据中心。**

## 1. 什么叫本地部署大模型？

比如你下载一个开源/开放权重模型：

```text
Qwen
Llama
DeepSeek 系列开放模型
Mistral
Gemma
……
```

再通过 Ollama、LM Studio 等运行时加载。

整个过程可以是：

```text
你的电脑
┌──────────────────────────────┐
│                              │
│  Agent                       │
│    ↓                         │
│  本地模型                    │
│    ↓                         │
│  GPU / 显存                  │
│    ↓                         │
│  推理                        │
│    ↓                         │
│  Agent                       │
│    ↓                         │
│  修改 H:\GH-Agent            │
│                              │
└──────────────────────────────┘

甚至可以完全不访问互联网
```

这和使用云端 API 有本质区别：

```text
                 Internet

你的电脑 ───────────────→ OpenAI 云端
 Agent                  大模型
   ↑                       │
   └───────────────────────┘
          返回结果
```

---

## 2. 本地模型最大的优势：隐私和控制权

假设你的 GH 商业插件以后包含核心算法：

```text
Data Tree Debug Agent
│
├─ 核心算法
├─ 商业逻辑
├─ 用户项目
└─ 私有代码
```

使用本地模型时可以做到：

```text
代码
 ↓
本地 Agent
 ↓
本地模型
 ↓
本地 GPU
 ↓
结果

整个过程
不需要把代码发给模型供应商
```

如果机器断网仍然能运行，那么数据甚至可以做到物理意义上的不离开电脑。

这对于：

**公司机密代码、建筑项目数据、客户模型、企业内部知识库、医疗/法律等敏感资料**

会非常有价值。

---

## 3. 第二个优势：通常不需要按 Token 付 API 费

云端 API 通常是：

```text
Agent
 ↓
API
 ↓
输入 Token
+
输出 Token
 ↓
产生 API 成本
```

如果 Agent 一天调用模型几百次，成本可能不断积累。

本地模型则更像：

```text
先买 GPU / 电脑
       ↓
下载模型
       ↓
大量运行
       ↓
不按 Token 给模型供应商付 API 费
```

但注意：

> 本地 ≠ 免费。

你实际上把成本从：

**API 费用**

换成：

**GPU + 显存 + 内存 + SSD + 电费 + 维护成本。**

---

# 4. 最大的问题：本地模型通常没有顶级云模型那么强

这是非常关键的一点。

假设：

```text
复杂 GH 插件开发
↓
阅读几十个 C# 文件
↓
理解 Grasshopper SDK
↓
发现 Bug
↓
修改多个文件
↓
编译
↓
分析报错
↓
继续修复
```

强大的云端 Coding Agent / 模型通常更适合这种复杂任务。

本地模型则非常依赖：

```text
模型本身
+
参数规模
+
量化方式
+
你的 GPU
+
上下文长度
+
Agent 框架
```

所以不能简单理解成：

> “我下载一个模型，就相当于在自己电脑里装了一个 ChatGPT/Codex。”

通常不是。

---

# 5. 为什么显卡那么重要？

因为大模型需要把大量参数放进：

> **GPU VRAM（显存）**

你前面问过“显卡内存是什么”，这里就连接起来了。

非常粗略地理解：

```text
小模型
7B / 8B
↓
普通消费级 GPU
比较容易运行

中型模型
14B / 32B
↓
需要更多显存
或者量化

大型模型
70B+
↓
消费级单卡开始困难
多 GPU / 大内存 / 量化
```

而且模型能“塞进去”：

**不代表跑得快，也不代表拥有舒服的长上下文。**

所以如果以后你跟我说：

> “我的显卡是 RTX XXXX，显存 XX GB，内存 XX GB。”

我可以直接帮你判断：

```text
你适合跑多大的模型
↓
哪些模型适合 Coding
↓
速度大概属于什么级别
↓
是否值得本地部署
```

---

# 6. 本地 vs 云端可以这样比较

| | 本地模型 | 云端模型 |
|---|---|---|
| 模型在哪里 | 你的电脑 | 数据中心 |
| GPU | 用你的 | 用服务商的 |
| 是否需要联网 | 可以不需要 | 通常需要 |
| API Key | 本地直接推理可不需要 | API 使用通常需要凭据 |
| 数据控制 | ⭐⭐⭐⭐⭐ | 取决于服务与配置 |
| 前期成本 | 高 | 低 |
| 按 Token API 成本 | 通常没有 | API 通常有 |
| 维护 | 自己负责 | 厂商负责 |
| 顶级模型能力 | 通常较弱 | 通常更强 |
| 升级 | 自己下载/配置 | 厂商更新 |
| 硬件要求 | 高 | 很低 |
| 离线使用 | 可以 | 通常不行 |

---

# 7. Agent 和本地模型可以组合

这点对你尤其重要。

你现在理解的是：

```text
Codex / Agent
      ↓
 OpenAI 云端模型
      ↓
决定下一步
      ↓
本地执行
```

以后完全可以变成：

```text
       Agent
         │
         ↓
     本地模型
         │
         ↓
   Git / Worktree
         │
         ↓
H:\GH-Agent-tree
         │
         ↓
dotnet build / test
```

Agent 本身不一定关心背后是谁。

只要 Agent 支持对应的模型接口，就可能：

```text
                Agent
                  │
        ┌─────────┴─────────┐
        ↓                   ↓
    OpenAI API           本地模型
        ↓                   ↓
   云端 GPU              你的 GPU
```

甚至可以混合。

---

# 8. 混合模式其实很适合你

例如以后你的 GH Agent 项目：

```text
                 Dashi
                   ↓
                Multica
                   ↓
        ┌──────────┼──────────┐
        ↓          ↓          ↓
     Agent A    Agent B     Agent C
        │          │          │
        ↓          ↓          ↓
     本地模型    本地模型     云端模型
        │          │          │
      简单任务    简单任务    困难任务
```

例如：

**本地模型负责：**

```text
代码分类
生成注释
简单重构
日志分析
文档整理
简单 Bug
```

强云模型负责：

```text
复杂架构
困难 Bug
跨几十个文件修改
复杂 Agent 规划
GH SDK 深层问题
```

这样就变成：

> **便宜/隐私敏感任务 → Local**
>
> **困难任务 → Cloud**

这通常比“100% 本地”或“100% 云端”更灵活。

---

# 9. 还有一个容易混淆的地方

**本地模型 ≠ 本地 Agent。**

这是两个维度：

```text
Agent 在哪里运行？
        │
        ├─ 本地
        └─ 云端

模型在哪里运行？
        │
        ├─ 本地
        └─ 云端
```

所以实际上可能出现四种组合：

```text
① 本地 Agent + 本地模型

② 本地 Agent + 云端模型

③ 云端 Agent + 云端模型

④ 云端 Agent + 本地模型
   （需要网络打通，较特殊）
```

你现在最应该理解的是 **②**：

```text
┌────── 你的电脑 ──────┐

Codex / Agent
     ↓
读取代码
     ↓
调用云端模型 ─────────────→ ☁️ AI
     ↑                         │
     └─────────────────────────┘
     ↓
修改代码
     ↓
dotnet build
     ↓
Git Commit

└──────────────────────┘
```

而本地大模型方案主要是把这里：

```text
☁️ AI
```

搬到：

```text
🖥️ 你的 GPU
```

---

## 对你现在 GH 插件开发的建议

现阶段我**不建议你因为担心 API Key 就急着全部改成本地模型**。

你现在更值得先掌握：

```text
Dashi
  ↓
Codex
  ↓
Branch + Worktree
  ↓
开发 / Test
  ↓
Git
  ↓
GitHub Private
```

等这个流程稳定以后，再增加：

```text
              Agent
                ↓
        ┌───────┴───────┐
        ↓               ↓
    本地模型          云端模型
   低成本/隐私       高难度任务
```

这样你的整个 AI 开发体系就会逐渐完整，而不是一次把复杂度全部堆上去。

如果你把**电脑的 CPU、显卡型号、显存、内存**告诉我，我可以直接按你的机器判断“能不能本地部署、适合多大的模型、哪些 Coding 模型值得跑”。

---

## 对话 16

### 用户

api是什么意思？全称是什么

### AI

**API** 的全称是：

> **Application Programming Interface**  
> 中文通常翻译为：**应用程序编程接口**

不过“接口”这个词第一次听比较抽象。你可以直接把 API 理解成：

> **两个软件之间沟通和调用功能的标准通道。**

比如你使用一个 Agent 调用大模型：

```text
你的 Agent
    │
    │ ① 发送请求
    │ “帮我分析这段 C# 代码”
    ↓
┌─────────────────────┐
│     OpenAI API      │ ← 接口
└─────────────────────┘
    ↓
OpenAI 大模型
    ↓
生成结果
    ↓
┌─────────────────────┐
│     OpenAI API      │
└─────────────────────┘
    ↓
你的 Agent 得到结果
```

### API 就像餐厅的“服务窗口”

可以这样类比：

```text
你                 Agent 软件
↓                      ↓
顾客                   程序

服务员                 API
↓                      ↓
厨房                 AI 模型
```

你不能自己跑进厨房：

```text
顾客 → 厨房 ❌
```

而是：

```text
顾客
 ↓
服务员
 ↓
厨房
 ↓
服务员
 ↓
顾客
```

软件也是：

```text
Agent
 ↓
API
 ↓
大模型
 ↓
API
 ↓
Agent
```

所以你刚才问的 **API Key** 就更容易理解了：

```text
API
= 接口 / 通道

API Key
= 使用这个接口的身份凭证 / 钥匙
```

例如：

```text
你的软件
   ↓
拿着 API Key
   ↓
调用 API
   ↓
OpenAI 验证：
“这个 Key 有权限吗？”
   ↓
有
   ↓
允许调用模型
```

因此这几个概念可以一次记住：

> **AI Model = 大脑**  
> **API = 软件访问这个大脑的接口**  
> **API Key = 使用接口的钥匙/凭证**  
> **Agent = 拿着钥匙，通过 API 调用大脑，再操作工具完成任务的执行者**

这四个概念搞清楚以后，你理解 Codex、Multica、本地模型以及第三方 Agent 会容易很多。

---

## 对话 17

### 用户

再把上述的概念补充到之前的http网站里，保持逻辑清晰，重点突出，并且配色清新

### AI

已经补充进之前的交互网页，并把整体配色改成了更清新的浅色系。

这次新增了 **Agent / AI Model / API / API Key / 本地模型 / 云端模型** 这一整套逻辑，并加入了：

- Agent 的完整运行循环
- API 全称和作用
- API Key 为什么能授权第三方软件
- API Key 泄露风险和安全存放方式
- 本地模型 vs 云端模型
- 本地 Agent ≠ 本地模型
- 云端 / 本地 / 混合模式互动切换
- 与 Git、Worktree、Codex、Dashi、Multica、Linear 的整体关系
- 原有章节重新整理编号，保持前后逻辑连续

你可以直接覆盖之前那份使用：

:chatgpt-content-reference{index="0"}

---

## 对话 18

### 用户

逻辑顺序不太对，对于初学者应该是先理解agent的运行逻辑和api之类的，等她明白了agent怎么工作的再去学习工作流之类的相关知识，重新整理一下这个http，并且重点要加粗放大或者其他颜色的字标出，现在看完全没有重点

[User attached 1 image; image contents were not included]

### AI

我已经按“**初学者应该先理解 Agent，再进入开发工作流**”的顺序重新整理了，而且这次把重点做得更明显了：大字号、加粗、蓝/绿/橙/紫色强调、关键结论框、学习顺序提示。

新的逻辑顺序是：

**Agent 怎么工作 → AI Model → API → API Key → 本地/云端模型 → Agent/模型四种组合 → 为什么要学 Git → Git → Branch / Worktree → Workflow → Codex 对话 → 分叉 → 多 Codex → Dashi / Multica / Linear → GitHub → 完整流程 → GH 推荐路线**

:chatgpt-content-reference{index="0"}

这一版我也保留了交互切换、流程播放、左侧导航和场景选择器。

---

## 对话 19

### 用户

如果使用multica会不会很消耗token？怎么样给不同的任务分配不同的agent最省token

### AI

会，但要看你怎么用。**Multica 本身不是“额外吃很多 token 的模型”**，它主要是调度层；真正大量消耗 token 的，是它启动的 Codex / Claude 等 Agent 的每次运行。Multica 的 Agent 配置里可以指定模型、thinking level、skills、instructions 和并发数；每次任务触发后，都会形成一次独立 run，运行时再调用对应的 AI 编程工具。citeturn315692search0turn315692search1

最容易出现 token 暴涨的情况不是“用了 Multica”，而是下面这些：

- 同一个任务被多个 Agent 重复分析
- 每个 Agent 都拿到过多无关上下文
- 小任务也使用最强模型 + 高 thinking
- 任务切得太碎，频繁启动新 run，反复重新读项目
- Agent 之间互相评论、互相调用太多轮
- 并发开很多 Agent，但任务其实互相依赖，最后大量返工

所以你真正应该优化的是：

> **任务 → Agent → 模型 → 上下文 → thinking level**

而不是单纯减少 Agent 数量。

## 最省 token 的分配原则

我建议你以后把 Agent 分成 **4 个等级**，而不是“每个功能建一个 Agent”。

| Agent 类型 | 负责什么 | 模型策略 | Token 策略 |
|---|---|---|---|
| **轻量 Agent** | 查文件、简单改名、文档、小修复 | 便宜/快模型 | 低 |
| **Coding Agent** | 常规功能开发、单模块修改 | 中等 coding 模型 | 中 |
| **Debug Agent** | 编译错误、Bug、测试失败 | 较强模型 | 中高 |
| **Architect Agent** | 架构、跨模块重构、复杂设计 | 最强模型 | 高，但少用 |

也就是说，不应该：

```text
任务 A → 最强 Agent
任务 B → 最强 Agent
任务 C → 最强 Agent
任务 D → 最强 Agent
```

而应该：

```text
简单任务
   ↓
轻量 Agent

普通开发
   ↓
Coding Agent

困难 Bug
   ↓
Debug Agent

重大架构问题
   ↓
Architect Agent
```

这叫**模型路由**。

Multica 本身允许 Agent 使用不同的 model 和 thinking level，所以这种分层是可以配置出来的。citeturn315692search3turn315692search8

---

# 对你的 GH 插件，我建议这样配

不要一开始搞 10 个 Agent。

你现在可以只建 **3 个**。

```text
Multica
│
├─ Agent 1：GH Coding
│      普通开发
│
├─ Agent 2：GH Debug
│      Bug / Build / Test
│
└─ Agent 3：GH Architect
       架构 / 复杂 Data Tree 问题
```

然后按任务复杂度分配。

例如：

```text
“按钮颜色调整”
       ↓
GH Coding
       ↓
低 thinking
```

```text
“新增 Tree Analyzer Component”
       ↓
GH Coding
       ↓
中 thinking
```

```text
“为什么空 Data Tree 会导致运行时崩溃？”
       ↓
GH Debug
       ↓
中 / 高 thinking
```

```text
“重新设计 Data Tree Debug Agent 整体架构”
       ↓
GH Architect
       ↓
高 thinking
```

这样比：

```text
所有任务
↓
最强模型
↓
High Thinking
```

省很多。

---

# 一个特别重要的 token 原则：不要让 Agent 什么都读

假设你的项目：

```text
GH-Agent
│
├─ Core
├─ UI
├─ TreeAnalyzer
├─ AI
├─ Tests
├─ Docs
└─ 其他几十个文件
```

任务只是：

> 修改 UI 按钮。

如果你给 Agent 的任务写：

```text
检查整个项目，
分析架构，
理解所有模块，
然后修改按钮。
```

它可能会：

```text
读取 Core
读取 AI
读取 TreeAnalyzer
读取 Tests
读取 Docs
……
```

大量 token 被浪费。

更好的任务是：

```text
只修改 UI 模块中的按钮布局。

目标文件优先检查：
src/UI/xxx.cs

不要修改：
Core/
TreeAnalyzer/
AI/

完成后只运行相关 build/test。
```

于是变成：

```text
相关上下文
   ↓
模型
   ↓
完成任务
```

而不是：

```text
整个项目
   ↓
模型
   ↓
先花大量 token 理解无关内容
```

这通常比“换一个便宜模型”还有效。

---

# Task 应该切多大？

这个很关键。

不要切得太大：

```text
❌ 开发整个 Data Tree Debug Agent
```

因为 Agent 会不停探索、规划、修改，token 很容易越来越多。

但也不要切成：

```text
❌ 创建一个类
❌ 增加一个函数
❌ 改一个变量
❌ 加一个按钮
```

因为每个 run 都可能重新加载上下文。

比较好的粒度是：

```text
✅ 实现 Tree 输入扫描功能

✅ 添加 Empty Branch 检测

✅ 创建 Tree Analyzer UI 面板

✅ 修复空 Tree 导致的崩溃

✅ 为 Tree Analyzer 添加单元测试
```

也就是：

> **一个 Task 最好对应一个明确、可以独立验收的成果。**

---

# 还有一个经常被忽略的浪费：重复启动 Agent

Multica 中一个 Issue 可以有多次 run，而且同一个任务可以重新运行或者交给不同 Agent。每次运行都有自己的记录。citeturn315692search9

所以：

```text
Run 1
Agent A 分析
消耗 50k token
↓
不满意
↓
Run 2
Agent B 从头分析
又 50k
↓
Run 3
Agent C
又重新理解项目
```

很容易出现：

```text
50k + 50k + 50k
```

但真正有价值的工作可能只有一次。

更好的方式：

```text
Agent A
↓
完成 80%
↓
在原 Issue 中补充：
“现在只修复 XX，不要重做已有部分”
↓
继续
```

而不是不断：

> 换 Agent → 从零开始。

---

# Multica 的 Skills 反而可以帮你省 token

Multica 的 Agent 可以配置 reusable Skills，长期复用方法、参考资料和支持文件。citeturn315692search1

例如你的 GH 项目以后可以有：

```text
Skill：GH Plugin Coding Rules

里面固定写：
- Grasshopper SDK 版本
- 项目目录结构
- 命名规范
- build 命令
- test 命令
- Data Tree 规则
- 哪些目录不能修改
```

这样你不用每一个 Task 都重复输入：

```text
我们用 Grasshopper……
项目位于……
SDK 是……
编译方式……
代码规范……
```

注意：Skill 本身也可能进入上下文，因此**不要把 Skill 写成一本几万字的百科全书**。

应该做到：

> **短、稳定、高频复用。**

---

# Thinking Level 也很重要

Multica 的 Agent 配置支持 thinking level。citeturn315692search1turn315692search5

你可以理解成：

```text
Low
↓
少想一些
快
便宜
适合简单任务

Medium
↓
大多数 Coding
比较平衡

High
↓
复杂 Debug / 架构
更慢
通常消耗更多
```

所以推荐：

```text
文档 / 简单 UI
→ Low

常规编码
→ Medium

复杂 Bug
→ Medium / High

重大架构
→ High
```

不要：

```text
所有 Agent = High
```

---

# 我最推荐你的 Multica 架构

现阶段你可以这样：

```text
                       Multica
                          │
            ┌─────────────┼─────────────┐
            ↓             ↓             ↓
      GH Coding       GH Debug     GH Architect
            │             │             │
        普通模型        强模型        最强模型
         Medium        Medium/High      High
            │             │             │
         80%任务         15%任务        5%任务
```

核心思想就是：

> **80% 的普通工作不要交给最贵、最强的 Agent。**

复杂任务才“升级”。

这就像公司里：

```text
普通工作
→ 普通工程师

难 Bug
→ 高级工程师

架构决策
→ 架构师
```

不是所有事情都找 CTO。

---

# 如果你追求最低 Token，可以再加一层“Router”

以后任务多了，可以变成：

```text
                  Task
                   ↓
              Router Agent
          “先判断任务类型”
                   ↓
        ┌──────────┼──────────┐
        ↓          ↓          ↓
      简单        普通        困难
        ↓          ↓          ↓
   Cheap Agent  Coding     Architect
```

但你现在**不建议先做这个**。

因为 Router 自己也需要调用模型，会增加一次 token 消耗。

在你现在任务量还不大的情况下：

> **你自己当 Router 最省钱。**

也就是你看一眼任务，然后手动决定：

```text
这个简单 → Coding
这个 Bug → Debug
这个很难 → Architect
```

零额外 token。

---

## 最后给你一个非常实用的判断公式

以后每创建一个任务，先问自己三个问题：

```text
① 这个任务有多难？
      ↓
决定模型

② 它需要看多少代码？
      ↓
限制上下文

③ 能否独立验收？
      ↓
决定 Task 大小
```

然后：

```text
小任务
→ 小上下文
→ 低 / 中 thinking
→ 普通 Agent

复杂任务
→ 必要上下文
→ 高 thinking
→ 强 Agent
```

这通常就是 **Multica 最省 token、同时又不明显牺牲质量** 的方式。

另外，Multica CLI 目前还提供了按 Issue 查看聚合 token 用量的 `usage` 功能，所以你完全可以跑一两周后看真实数据，再调整哪个 Agent 太贵。citeturn315692search5

如果你准备正式上 Multica，我建议第一版只建 **GH Coding / GH Debug / GH Architect 三个 Agent**，先别建更多。这样最容易看清每类任务到底消耗多少。

---

## 对话 20

### 用户

这一步实现每一个功能应该分别创建新的agent/codex对话，还是创建一个编程agent/一个codex对话呢?那么每一个功能是不是也应该创建一个新的brunch和workflow呢

[User attached 1 image; image contents were not included]

### AI

这里最容易犯的错误，就是把“任务拆得细”误解成“每一项都要新 Agent + 新 Codex 对话 + 新 Branch + 新 Worktree”。

**不是这样。**

你截图里的 5 项：

```text
① 实现 Tree 输入扫描功能
② 添加 Empty Branch 检测
③ 创建 Tree Analyzer UI 面板
④ 修复空 Tree 导致的崩溃
⑤ 为 Tree Analyzer 添加单元测试
```

我更建议你按照“**能不能作为一个独立成果被开发、测试、Review、Merge**”来分，而不是按照每一个小功能机械地分。

## 先把 4 个东西分开

最关键的是：

```text
Agent
= 什么类型的程序员

Codex 对话
= 这一次具体工作的上下文

Branch
= 这次开发对应的代码路线

Workflow
= 这项任务从开始到结束的全过程
```

所以比如你可以长期只有一个：

> **GH Coding Agent**

它就像你团队里的“Grasshopper C# 程序员”。

但是这个 Agent 可以执行很多不同任务：

```text
GH Coding Agent
│
├─ Codex 对话 A
│   → Tree Analyzer 核心功能
│
├─ Codex 对话 B
│   → UI
│
└─ Codex 对话 C
    → Bug 修复
```

因此：

> **不要每个功能创建一个新的 Agent。**

Agent 更像“岗位/能力配置”，应该长期复用。

---

# 对你截图这 5 项，我会怎么拆？

我大概率不会拆成 5 个 Workflow。

而是先拆成 **2～3 个真正的开发任务**。

### Workflow A：Tree Analyzer 核心逻辑

把这些放在一起：

```text
Tree Analyzer Core

├─ 实现 Tree 输入扫描
├─ 添加 Empty Branch 检测
└─ 添加对应单元测试
```

为什么？

因为它们高度相关：

```text
输入扫描
   ↓
分析 Tree
   ↓
判断 Empty Branch
   ↓
测试这些行为
```

测试本身通常就应该跟功能一起提交。

所以可以是：

```text
Dashi Task:
实现 Tree Analyzer Core

Codex 对话 A
        ↓
GH Coding Agent
        ↓
Branch:
feature/tree-analyzer-core
        ↓
Worktree:
H:\GH-Agent-tree-core
```

这是一条完整 Workflow。

---

# Workflow B：Tree Analyzer UI

UI 可以独立出来：

```text
创建 Tree Analyzer UI 面板
```

因为：

> UI 和核心分析逻辑虽然相关，但代码职责明显不同。

可以：

```text
Codex 对话 B
        ↓
GH Coding Agent
        ↓
Branch:
feature/tree-analyzer-ui
        ↓
Worktree:
H:\GH-Agent-tree-ui
```

这样以后 Review 也清楚：

```text
Core PR
→ 看算法逻辑

UI PR
→ 看界面逻辑
```

而且如果两个 Codex 同时开发：

```text
Codex A
↓
Core Worktree

Codex B
↓
UI Worktree
```

可以并行。

---

# Workflow C：空 Tree 崩溃 Bug

这一项要看具体情况：

```text
修复空 Tree 导致的崩溃
```

有两种情况。

如果这个 Bug **本来就是 Tree Analyzer Core 开发过程中发现的**，而且修复非常小：

```text
Core
├─ 输入扫描
├─ Empty Branch
├─ 修复 Empty Tree crash
└─ Tests
```

直接放进 Workflow A 就好。

没必要再：

```text
新 Codex
新 Branch
新 Worktree
新 Workflow
```

这会增加管理成本和 token 消耗。

但是如果：

```text
main 当前正式版本
已经存在严重崩溃
```

而且需要马上修：

那么应该单独：

```text
Hotfix Workflow

Codex 对话 C
     ↓
GH Debug Agent
     ↓
hotfix/empty-tree-crash
     ↓
H:\GH-Agent-hotfix
```

因为它有独立的目的：

> **修复生产 Bug，而不是开发新功能。**

---

# 所以你这个例子，我推荐的实际结构是

```text
                    Tree Analyzer
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
     Workflow A      Workflow B    Workflow C？
      Core 功能          UI         Hotfix
          │              │              │
      Codex A        Codex B        Codex C
          │              │              │
   GH Coding Agent  GH Coding Agent  GH Debug Agent
          │              │              │
feature/tree-core  feature/tree-ui  hotfix/empty-tree
          │              │              │
 Worktree Core     Worktree UI     Worktree Hotfix
```

其中 C 只有在 Bug 值得独立修复的时候才单独存在。

---

# 那“一项功能 = 一个 Branch”吗？

**也不是。**

更准确的规则是：

> **一个可以独立 Review、独立测试、独立 Merge 的开发成果 ≈ 一个 Branch。**

例如：

```text
实现输入扫描
+
Empty Branch 检测
+
对应测试
```

虽然是 3 个 checklist：

```text
☑ 输入扫描
☑ Empty Branch
☑ Tests
```

但它们共同构成：

> **Tree Analyzer Core**

所以一个 Branch 就够：

```text
feature/tree-analyzer-core
```

而不是：

```text
feature/input-scan
feature/empty-branch
feature/tree-tests
```

后一种对你现在来说太碎了。

---

# 测试尤其不要轻易单独 Branch

例如：

```text
开发 Empty Branch Detection
```

最好的开发过程本来就应该：

```text
实现功能
   ↓
写测试
   ↓
运行测试
   ↓
Commit
```

所以：

```text
功能代码 + 对应测试
```

通常属于同一个 Workflow。

不要：

```text
Agent A 写代码
Agent B 专门给这一点写测试
```

除非项目已经很大，需要专门的 QA/Test Agent。

你现在没必要。

---

# Codex 对话什么时候应该新开？

可以用一个非常简单的判断：

### 同一个目标，连续工作

不必新开。

例如 Codex 正在：

```text
Tree Analyzer Core
│
├─ 输入扫描
├─ Empty Branch
└─ Tests
```

保持一个 Codex 对话反而更好。

因为它已经知道上下文：

```text
为什么这么设计
哪些文件改过
测试结构是什么
```

节省重新理解代码的 token。

---

### 目标发生明显变化

建议新开。

比如：

```text
从 Core 算法
↓
切换到 UI 设计
```

可以开 Codex B。

或者：

```text
正常 Feature 开发
↓
变成紧急 Crash Debug
```

可以开 Codex C。

---

### 要真正同时并行

强烈建议新开：

```text
Codex A
↓
Core

Codex B
↓
UI
```

并且分别配：

```text
Branch A + Worktree A
Branch B + Worktree B
```

否则两个 Codex 都操作同一个目录，就容易互相干扰。

---

# 一个很好用的判断标准

以后你看到 checklist：

```text
☐ A
☐ B
☐ C
☐ D
☐ E
```

不要马上想：

> 五个任务 = 五个 Agent。

先问：

### ① 它们是不是实现同一个目标？

是：

```text
放在同一个 Workflow
```

### ② 它们是不是必须共享大量上下文？

是：

```text
尽量同一个 Codex 对话
```

### ③ 它们能不能独立 Review / Merge？

能，而且比较大：

```text
可以拆 Branch
```

### ④ 它们需要同时开发吗？

需要：

```text
独立 Branch + Worktree + Codex 对话
```

---

# 你现在最适合的粒度

对你的 GH 项目，我建议控制在：

```text
一个 Workflow
≈ 0.5 天 ～ 2 天左右可以完成的
一个明确、可验收的成果
```

例如：

**太小：**

```text
❌ 添加一个函数
❌ 改一个按钮
❌ 写一个 test
```

**太大：**

```text
❌ 完成整个 Data Tree Debug Agent
```

**比较合适：**

```text
✅ 实现 Tree Analyzer Core

✅ 实现 Tree Analyzer UI

✅ 实现 Tree 问题可视化提示

✅ 修复 Empty Tree Crash

✅ 实现自动修复建议模块
```

---

## 所以你截图里的内容，我建议最终这样组织

```text
Dashi

Task 1
Tree Analyzer Core
├─ 输入扫描
├─ Empty Branch Detection
└─ Unit Tests

      ↓

Codex 对话 A
GH Coding Agent
feature/tree-analyzer-core
Worktree A


Task 2
Tree Analyzer UI

      ↓

Codex 对话 B
GH Coding Agent
feature/tree-analyzer-ui
Worktree B


Task 3（如果值得独立处理）
Empty Tree Crash Fix

      ↓

Codex 对话 C
GH Debug Agent
hotfix/empty-tree-crash
Worktree C
```

这里有一个非常重要的地方：

> **Agent 可以复用，Codex 对话按任务分，Branch 按独立开发成果分，Worktree 按“是否要并行”决定。**

这四者并不是永远 1:1:1:1。

你现在阶段可以先用一个非常实用的近似规则：

> **1 个中等大小、可独立验收的 Task ≈ 1 个 Codex 对话 + 1 个 Branch；如果并行开发，再给它 1 个 Worktree。**

这个粒度比较适合你现在的 GH + Codex + Dashi 工作方式。

---

## 对话 21

### 用户

我说错了，不是workflow而是worktree

### AI

明白，你问的是：

> **截图里每一个功能，是不是都应该创建一个新的 Agent / Codex 对话，以及新的 Branch + Worktree？**

答案还是：**不需要一项功能就配一套。**

更适合你的规则是：

```text
一个相对独立、可以单独开发和验收的任务
≈
1 个 Codex 对话
+
1 个 Branch
+
如果要并行开发，再加 1 个 Worktree
```

注意这里：

> **Branch 比 Worktree 更“应该存在”。**  
> **Worktree 主要是为并行开发服务的。**

所以你截图里的 5 项，我更建议这样分：

```text
Task A：Tree Analyzer Core
├─ 实现 Tree 输入扫描
├─ 添加 Empty Branch 检测
└─ 添加单元测试

Task B：Tree Analyzer UI
└─ 创建 UI 面板

Task C：Empty Tree Crash
└─ 修复空 Tree 崩溃
```

然后对应：

```text
Task A
→ Codex 对话 A
→ feature/tree-analyzer-core
→ Worktree A   （如果和别的任务并行）

Task B
→ Codex 对话 B
→ feature/tree-analyzer-ui
→ Worktree B   （如果并行）

Task C
→ Codex 对话 C
→ hotfix/empty-tree-crash
→ Worktree C   （如果并行）
```

如果你是**一个一个顺序做**，其实可以：

```text
主项目目录
H:\GH-Agent
```

然后不断切 Branch：

```text
feature/tree-analyzer-core
↓ 完成

feature/tree-analyzer-ui
↓ 完成

hotfix/empty-tree-crash
```

这种情况下，**甚至不需要多个 Worktree**。

只有当你要：

```text
Codex A 同时开发 Core
+
Codex B 同时开发 UI
+
Codex C 同时修 Bug
```

才非常适合：

```text
H:\GH-Agent-core
H:\GH-Agent-ui
H:\GH-Agent-hotfix
```

每个目录对应一个 Branch。

所以一句话记住：

> **Branch 用来隔离“代码任务”，Worktree 用来隔离“同时进行的工作目录”。**

你现在不要把 Worktree 理解成“每个功能都必须创建一个文件夹”。它更像是：

> **只有多个 Codex / 多个任务要同时开工时，才给每条 Branch 配一个独立施工现场。**

---

## 对话 22

### 用户

所以这里的对话a\b\c就是一个codex,三个对话框三个brunch，三个worktree

[User attached 1 image; image contents were not included]

### AI

**对，你现在这个理解基本正确。** 但我帮你把措辞再校准一下，以后就不容易混淆。

假设你只有一个长期配置好的 **GH Coding Agent**：

```text
GH Coding Agent
│
├─ Codex 对话 A
│    → Tree Analyzer Core
│
├─ Codex 对话 B
│    → UI
│
└─ Codex 对话 C
     → Bug 修复
```

如果这三个任务要**同时并行开发**，最清晰的配置就是：

```text
                    GH Coding Agent
                          │
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
   Codex 对话 A       Codex 对话 B       Codex 对话 C
        │                 │                 │
   Core 功能             UI              Bug Fix
        │                 │                 │
        ↓                 ↓                 ↓
 Branch A            Branch B            Branch C
feature/core       feature/ui        hotfix/empty-tree
        │                 │                 │
        ↓                 ↓                 ↓
 Worktree A          Worktree B          Worktree C
H:\GH-core          H:\GH-ui           H:\GH-hotfix
```

所以在这种**并行开发场景**下，你完全可以先记成：

> **3 个 Codex 对话 = 3 个独立开发任务 = 3 个 Branch = 3 个 Worktree。**

### 但“一个 Codex”这个说法要稍微修正

不是：

> 一个 Codex 里面有三个对话框。

更准确地说，是同一个 **GH Coding Agent 配置/角色**，启动了 **3 个独立的 Codex 任务/会话**。

可以把它想成：

```text
GH Coding Agent
= “GH 程序员”这个岗位模板

Codex A
= 这个程序员的工作台 A

Codex B
= 这个程序员的工作台 B

Codex C
= 这个程序员的工作台 C
```

如果三个同时工作，就给三个工作台各自一个施工现场：

```text
Codex A → Branch A → Worktree A
Codex B → Branch B → Worktree B
Codex C → Branch C → Worktree C
```

这样就不会互相踩代码。

### 为什么 Agent 不需要创建三个？

因为 **Agent 描述的是“能力和角色”**，不是具体任务。

例如：

```text
GH Coding Agent
────────────────
能力：
• C#
• Grasshopper SDK
• RhinoCommon
• Git
• 编译
• 测试
• 项目编码规范
```

这个配置可以反复使用：

```text
             GH Coding Agent
                   │
          使用同一套能力配置
                   │
       ┌───────────┼───────────┐
       ↓           ↓           ↓
     Task A      Task B      Task C
       ↓           ↓           ↓
    Codex A     Codex B     Codex C
```

这样反而比创建：

```text
GH Coding Agent A
GH Coding Agent B
GH Coding Agent C
```

更合理，也更容易维护。

---

## 什么时候才需要不同 Agent？

如果**工作性质变了**，才值得创建不同 Agent。

例如以后可以是：

```text
Multica
│
├─ GH Coding Agent
│    ├─ Codex A → Core
│    └─ Codex B → UI
│
├─ GH Debug Agent
│    └─ Codex C → Crash Bug
│
└─ GH Architect Agent
     └─ Codex D → 架构设计
```

这里 Agent 的区别不是“任务 A/B/C”，而是**专业能力和模型配置不同**。

例如：

```text
GH Coding Agent
→ 常规 Coding
→ Medium Thinking
→ 普通模型

GH Debug Agent
→ 疑难 Bug
→ High Thinking
→ 更强模型

GH Architect Agent
→ 架构设计
→ High Thinking
→ 最强模型
```

这也正好对应你前面问的**如何节省 Token**：不是每个任务都用最强 Agent。

---

### 最终可以把整个关系记成这一张图

```text
                    Multica
                 （任务调度）
                      │
          ┌───────────┼───────────┐
          ↓           ↓           ↓
      Coding Agent  Debug Agent  Architect Agent
          │
      ┌───┴───┐
      ↓       ↓
   Codex A   Codex B
      │       │
   Task A   Task B
      │       │
 Branch A  Branch B
      │       │
Worktree A Worktree B
```

这里最重要的是：

> **Agent = 什么类型的人来干。**  
> **Codex 对话/任务 = 这一次具体干什么。**  
> **Branch = 这次任务使用哪条代码线。**  
> **Worktree = 这次任务在哪个独立文件夹施工。**

而你截图里的 **A / B / C 如果确实需要同时开发**，那么 **3 个 Codex 会话 + 3 个 Branch + 3 个 Worktree** 是非常清晰、合理的组织方式。

---

## 对话 23

### 用户

这种情况在multica中对应什么情况？

[User attached 1 image; image contents were not included]

### AI

对，你这张图如果映射到 **Multica**，可以理解为：

> **1 个 Agent 配置 + 3 个独立 Task/Issue + 3 次独立 Run + 3 个独立 Worktree/Branch。**

也就是说，你图里的“Codex 对话 A/B/C”，到了 Multica 里更准确的概念不是“你手动打开三个聊天框”，而是 **Multica 启动三次独立的 Agent Run**。

### 你的图 → Multica

```text
                    Multica
                       │
                       │
              GH Coding Agent
             （一个 Agent 配置）
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       Task A         Task B       Task C
     Core 功能          UI         Bug Fix
          │            │            │
          ↓            ↓            ↓
        Run A         Run B        Run C
          │            │            │
       Codex          Codex        Codex
          │            │            │
          ↓            ↓            ↓
   feature/core    feature/ui   hotfix/empty-tree
          │            │            │
          ↓            ↓            ↓
    Worktree A     Worktree B    Worktree C
```

这里最关键的是：

**GH Coding Agent 并没有复制成三个 Agent。**

它更像一个“模板”：

```text
GH Coding Agent

模型：某个 Coding Model
Thinking：Medium
Skills：GH / C# / RhinoCommon
Instructions：项目编码规范
执行工具：Codex
```

然后 Multica 可以拿这一个 Agent 配置去执行不同 Task。

---

## Multica 替你做掉了什么？

你不用 Multica 的时候，是你自己手动调度：

```text
你
│
├─ 打开 Codex A
│   └─ 指定 H:\GH-core
│
├─ 打开 Codex B
│   └─ 指定 H:\GH-ui
│
└─ 打开 Codex C
    └─ 指定 H:\GH-hotfix
```

所以之前我说：

> 你自己其实就是“人工 Multica”。

用了 Multica 后：

```text
                  你
                  ↓
              创建 Tasks
                  ↓
               Multica
          ┌───────┼───────┐
          ↓       ↓       ↓
        Run A   Run B   Run C
          ↓       ↓       ↓
       Codex   Codex   Codex
          ↓       ↓       ↓
         WT A    WT B    WT C
```

**Multica 负责中间的调度。**

---

## 而且这里还有一个很重要的概念：Run

这个概念你现在应该加进知识体系里。

比如：

```text
Task
= 要做什么

Agent
= 用什么类型的 AI 程序员

Run
= 让这个 Agent 真正执行一次任务
```

例如：

```text
Task A
“实现 Tree Analyzer Core”
        ↓
选择
GH Coding Agent
        ↓
点击执行
        ↓
产生
Run #1
        ↓
Codex 真正开始工作
```

如果失败了，你可能再次运行：

```text
Task A
│
├─ Run #1 ❌
│
└─ Run #2 ✅
```

所以：

> **Task 和 Run 也不是同一个东西。**

一个 Task 可以有多次 Run。

---

# 这时候“Codex 对话”发生了变化

你手动使用 Codex 时：

```text
Codex 对话
≈
你和一个 Coding Agent 的工作会话
```

所以你看到的是聊天窗口。

但是进入 Multica 后，更适合把脑子里的模型改成：

```text
❌

Multica
↓
打开三个 Codex 聊天框
```

而是：

```text
✅

Multica
↓
启动三个 Agent Run
↓
三个 Run 分别调用 Codex
↓
Codex 在各自 Worktree 工作
```

所以 UI 上不一定表现成你熟悉的三个“聊天窗口”。

---

# 举你的 GH 项目实际例子

你在 Multica 创建三个任务：

```text
Task #101
Tree Analyzer Core

Task #102
Tree Analyzer UI

Task #103
Fix Empty Tree Crash
```

然后：

```text
Task #101
↓
GH Coding Agent
↓
Run #1
↓
Codex
↓
feature/tree-core
↓
Worktree A


Task #102
↓
GH Coding Agent
↓
Run #2
↓
Codex
↓
feature/tree-ui
↓
Worktree B


Task #103
↓
GH Coding Agent
↓
Run #3
↓
Codex
↓
hotfix/empty-tree
↓
Worktree C
```

于是你的电脑里可能实际出现：

```text
H:\GH-Agent
        → main

H:\GH-core
        → feature/tree-core

H:\GH-ui
        → feature/tree-ui

H:\GH-hotfix
        → hotfix/empty-tree
```

三个任务同时工作，但是不会互相切换文件夹里的 Branch。

---

# 不过你的图还有一个地方可以优化

你现在画的是：

```text
GH Coding Agent
├─ Core
├─ UI
└─ Bug Fix
```

真正使用 Multica 后，我反而可能把 Bug Fix 分给另一个 Agent：

```text
                    Multica
                       │
          ┌────────────┴────────────┐
          ↓                         ↓
   GH Coding Agent            GH Debug Agent
          │                         │
      ┌───┴───┐                     │
      ↓       ↓                     ↓
    Core      UI               Empty Tree Bug
      │       │                     │
    Run A   Run B                  Run C
      │       │                     │
   Codex   Codex                  Codex
      │       │                     │
 Branch A Branch B              Branch C
      │       │                     │
   WT A     WT B                  WT C
```

这样才真正体现 Multica 的价值：

> **不是单纯帮你“同时开三个 Codex”，而是根据任务性质，让不同类型的 Agent 去执行不同工作。**

---

## 这又和你前面问的 Token 优化连接起来了

例如：

```text
Core
↓
GH Coding Agent
↓
Medium
↓
正常成本


UI
↓
GH Coding Agent
↓
Low / Medium
↓
更省 Token


复杂 Bug
↓
GH Debug Agent
↓
High
↓
贵一点，但只给困难任务用
```

这样：

```text
                 Multica
                    │
              判断任务性质
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
       UI          Core        Bug
        ↓           ↓           ↓
       Low        Medium       High
        ↓           ↓           ↓
      Codex       Codex       Codex
```

而不是：

```text
所有 Task
↓
同一个最强 Agent
↓
High Thinking
↓
💸💸💸
```

---

## 所以你现在可以把整个 Multica 心智模型定下来

```text
                    Multica
                  【调度中心】
                       │
       ┌───────────────┼───────────────┐
       ↓                               ↓
GH Coding Agent                  GH Debug Agent
【能力/模型配置】                 【能力/模型配置】
       │                               │
   Task A / B                       Task C
【要做什么】                     【要做什么】
       │                               │
     Run A/B                         Run C
【一次实际执行】                 【一次实际执行】
       │                               │
     Codex                           Codex
【真正编码工具】                 【真正编码工具】
       │                               │
 Branch + Worktree               Branch + Worktree
【代码线 + 独立施工现场】        【代码线 + 独立施工现场】
       │                               │
     Commit                          Commit
       └──────────────┬────────────────┘
                      ↓
                    Review
                      ↓
                  Merge main
```

你现在已经基本把最容易混淆的几个层级拆开了：

> **Multica = 调度中心**  
> **Agent = 能力/角色模板**  
> **Task = 要做什么**  
> **Run = 真正执行一次**  
> **Codex = 编程执行工具**  
> **Branch = 独立代码线**  
> **Worktree = 独立物理工作目录**

这几个概念一旦分开，后面真正开始配置 Multica 就会清晰很多。

---

## 对话 24

### 用户

所以在multica里面可以配置不同的agent做不同的工作，那我在codex里面怎么实现呢？

### AI

可以实现，但 **Codex 本身更像“一个通用程序员”**，不像 Multica 那样天然有一个“Agent 配置中心”去管理多个不同角色。

所以你在 Codex 里要实现“不同 Agent 做不同工作”，本质上有 3 种办法：

### 1. 最简单：不同 Codex 对话 = 不同角色
你可以给不同对话明确不同职责。

比如：

```text
Codex 对话 A
角色：GH Coding Agent
任务：实现 Core 功能
要求：优先写功能代码，Medium thinking

Codex 对话 B
角色：GH UI Agent
任务：实现 UI
要求：尽量少改 Core，只处理 UI

Codex 对话 C
角色：GH Debug Agent
任务：修 Bug
要求：先复现，再定位，再最小化修改
```

这就是“人工模拟多个 Agent”。

你自己相当于 Multica：

```text
你
├─ 把 Core 交给 Codex A
├─ 把 UI 交给 Codex B
└─ 把 Bug 交给 Codex C
```

---

### 2. 更规范：给每类工作准备固定 Prompt 模板
比如你长期准备三个模板：

```text
GH Coding Agent
- 专注常规 C# / Grasshopper 功能开发
- 优先最小修改
- 完成后 build + test
- 不修改无关模块
```

```text
GH Debug Agent
- 先复现 Bug
- 收集错误信息
- 定位根因
- 优先最小修复
- 必须补回归测试
```

```text
GH Architect Agent
- 不急着改代码
- 先分析模块依赖
- 输出方案和风险
- 跨模块修改前先说明
```

然后新开 Codex 对话时，把对应模板放进去。

这就相当于：

> **Agent = 固定角色说明 + 固定工作规则**

虽然 Codex UI 里未必叫“Agent 配置”，但逻辑完全一样。

---

### 3. 再进一步：每个角色绑定不同 Branch / Worktree
如果同时开发：

```text
GH Coding Agent
↓
Codex A
↓
feature/tree-core
↓
H:\GH-core
```

```text
GH UI Agent
↓
Codex B
↓
feature/tree-ui
↓
H:\GH-ui
```

```text
GH Debug Agent
↓
Codex C
↓
hotfix/empty-tree
↓
H:\GH-hotfix
```

这样你就基本手动搭出了一个“小型 Multica”。

---

## Multica 和你手动用 Codex，最大的区别

Multica 是：

```text
Task
↓
Multica 自动选择 / 启动 Agent
↓
自动建立 Run
↓
绑定执行环境
↓
跟踪状态
```

你直接用 Codex 是：

```text
Task
↓
你判断该交给哪个角色
↓
你打开对应 Codex 对话
↓
你指定 Worktree
↓
你自己跟踪进度
```

所以本质差别是：

> **Multica = 自动调度**
>
> **Codex = 你自己调度**

---

## 对你现在最实用的方式

你其实不需要现在就上 Multica。

先建立 3 个固定角色就够：

```text
1. GH Coding Agent
   → 普通功能开发

2. GH Debug Agent
   → Bug / Build / Test

3. GH Architect Agent
   → 架构 / 大重构
```

然后每个具体 Task：

```text
Task
↓
你判断属于哪一类
↓
开对应 Codex 对话
↓
指定 Branch
↓
如果并行，再指定独立 Worktree
```

你甚至可以把它简化成一句：

> **Codex 里“不同 Agent”其实就是：不同角色提示词 + 不同任务上下文 + 必要时不同 Worktree。**

等你以后觉得“我每天都在手动做这些分配，很烦”，那就是 Multica 开始有价值的时候。

---

## 对话 25

### 用户

使用multica要收费吗

### AI

目前看，**Multica 本身可以免费使用，尤其是你自己在本地 / Self-hosted 使用的方式**。它是开放源码并支持 self-hosting 的 Agent 管理平台。citeturn0search7turn0search0

但这里有一个很重要的区别：

> **Multica 免费 ≠ Multica 里面运行的 AI Agent 都免费。**

Multica 本身并不提供 Codex、Claude Code 等模型额度；它调用的是你已经安装并登录的 AI Coding CLI。官方文档明确说明，Multica 不提供这些工具的账号、订阅或模型 quota。citeturn0search8

所以你的成本结构实际上是：

```text
Multica
【调度管理】
   │
   ├── GH Coding Agent
   │       ↓
   │     Codex
   │       ↓
   │   OpenAI 模型成本/额度
   │
   ├── GH Debug Agent
   │       ↓
   │     Codex
   │       ↓
   │   OpenAI 模型成本/额度
   │
   └── GH Architect Agent
           ↓
        Claude Code
           ↓
       Anthropic 成本/额度
```

Multica 官方的定位就是：它**驱动你已经拥有的 Codex、Claude Code、Cursor 等工具，而不是自己附送这些 AI 服务**。citeturn0search1turn0search10

### 对你现在最重要的一点

如果你现在已经可以正常使用 Codex，那么安装 Multica 后，并不是：

```text
Multica
↓
重新购买一个 AI
```

而更接近：

```text
你已有 Codex
       ↓
Multica 帮你管理它
       ↓
Task A → Codex Run A
Task B → Codex Run B
Task C → Codex Run C
```

而且官方文档目前说明，Codex/Claude/Cursor 等 CLI 的登录凭据由各自工具**保存在本机**，Multica 不接收这些 CLI 的登录 token。citeturn0search8

### 但还有“商业许可”这个区别

Multica 的源码许可允许 self-host、修改和构建，但它的许可证包含额外条件：如果你要**把 Multica 作为托管服务提供给第三方**，或者**嵌入商业发行的软件产品**，可能需要商业许可。单纯自己拿来管理你的 GH 插件开发，与“把 Multica 本身卖给客户”不是一回事。citeturn0search0

所以对你目前：

> **个人开发 GH 插件 + 本机 Codex + Multica 调度**

我会先按：

**Multica 本身可以免费开始用，真正需要关注的是 Codex/Claude 等底层 AI 的额度和费用。**

而这也是为什么你刚才问的“不同任务怎么分配 Agent 才省 token”非常重要——**真正烧钱的主要是下面执行任务的模型，不是上面的 Multica 任务板。**

---

## 对话 26

### 用户

所以multica更像一个agent集合工作平台？

### AI

对，**这个理解非常接近了**。

你可以先把 Multica 理解成：

> **Multica = 多个 AI Agent 的“集合 + 分工 + 调度 + 工作管理平台”。**

但它不只是“把 Agent 放在一起”，更重要的是**管理这些 Agent 怎么干活**。

例如你的 GH 插件项目以后可能是：

```text
                     Multica
                【AI 开发工作平台】
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
 GH Coding Agent   GH Debug Agent   GH Architect Agent
   常规开发            调试              架构
        │              │              │
        ↓              ↓              ↓
   Task：Core      Task：修 Bug     Task：重构方案
        │              │              │
        ↓              ↓              ↓
      Run A           Run B           Run C
        │              │              │
      Codex           Codex           Codex
        │              │              │
        ↓              ↓              ↓
 Branch + WT      Branch + WT      Branch + WT
```

这里可以把几个东西类比成一个软件公司：

| 概念 | 类比 |
|---|---|
| **Multica** | AI 软件开发团队的管理办公室 |
| **Agent** | 不同岗位的员工模板 |
| **Task / Issue** | 工单 |
| **Run** | 某个员工真正开始执行一次工单 |
| **Codex** | 员工实际使用的编程能力/工具 |
| **Branch** | 独立代码路线 |
| **Worktree** | 独立施工桌 |
| **Git** | 版本档案系统 |
| **GitHub** | 云端代码仓库 + 协作平台 |

所以你之前单独使用 Codex：

```text
你
↓
Codex
↓
写代码
```

随着项目变复杂，会变成：

```text
你
↓
判断任务
↓
打开 Codex A / B / C
↓
告诉每个 Codex 做什么
↓
给它们 Branch
↓
给它们 Worktree
↓
检查谁完成了
↓
Review
↓
Merge
```

这时候**你本人就是调度器**。

Multica 相当于把中间这一层系统化：

```text
                你
                ↓
         “我要完成这些任务”
                ↓
        ┌──── Multica ────┐
        │                 │
    管 Task           管 Agent
    管 Run            管执行环境
    管状态            管并行任务
        │                 │
        └────────┬────────┘
                 ↓
          Codex / Claude 等
                 ↓
             实际写代码
```

### 但要特别区分一句话

**Multica ≠ Agent 本身。**

它更接近：

> **Multica 是“Agent 团队的管理平台”；Codex/Claude 等才是下面真正干活的 AI 编程工具。**

因此你可以建立：

```text
Multica
│
├── GH Coding Agent
├── GH Debug Agent
├── GH Architect Agent
├── Documentation Agent
└── Review Agent
```

然后让不同 Agent 使用不同的模型、thinking level、instructions、skills 等配置。

这也是 Multica 相比“我自己开 5 个 Codex 对话”的核心意义：

> **Codex 解决“AI 怎么写代码”。**
>
> **Multica 解决“很多 AI 编程任务怎么组织、分工、并行和管理”。**

对于你现在这个 **GH Data Tree Debug Agent 插件**，你已经可以先把 Multica 想象成未来的 **“AI 开发团队办公室”**。等项目任务多到你觉得手动开 Codex、建 Branch/Worktree、跟踪任务开始麻烦时，再引入它会非常自然。

---

## 对话 27

### 用户

那multica和harness有什么区别

### AI

如果你说的是 **Harness.io**，那它和 Multica 虽然都开始大量使用 Agent，但**关注的阶段不一样**。

最容易理解的一句话是：

> **Multica 更偏“管理 AI 程序员怎么写代码”。**  
> **Harness 更偏“代码写完以后，怎么测试、安全检查、部署、上线和运维”。**

Multica 官方把自己定位成一个管理 coding agents 的 workspace：可以把 Issue 分配给 Codex、Claude Code 等 Agent，让它们在你控制的 runtime 上执行，并跟踪进度；它还原生支持 worktree 并行模式。citeturn0search4turn0search12

Harness 当前则把自己定位成 Autonomous SDLC Platform，重点覆盖 build、test、security、deployment、infrastructure、runtime protection、cost management 等软件交付环节，而且强调企业级 policy、审批、RBAC 和 audit trail。citeturn0search0turn0search2

你可以把整个开发过程画成：

```text
需求
 │
 ▼
┌──────────────────────────────┐
│          Multica             │
│      “AI 开发团队办公室”       │
│                              │
│ Coding Agent                 │
│ Debug Agent                  │
│ Architect Agent              │
│       ↓                      │
│ Codex / Claude Code          │
│       ↓                      │
│ Branch + Worktree            │
│       ↓                      │
│ 写代码 / 修 Bug / Commit      │
└──────────────┬───────────────┘
               │
               ▼
             Git
               │
               ▼
           Pull Request
               │
               ▼
┌──────────────────────────────┐
│           Harness            │
│      “AI 交付 / 运维平台”      │
│                              │
│ Build                        │
│ Test                         │
│ Security Scan                │
│ Deployment                   │
│ Verification                 │
│ Rollback                     │
│ Infrastructure               │
│ Production                   │
└──────────────────────────────┘
```

Harness 自己现在甚至直接用一句类似的定位：**“Developers and AI write code. Harness AI does the rest.”** 它明确强调 coding agents 写完代码之后的交付链路。citeturn0search1turn0search14

### 但它们现在确实有重叠

这也是最容易让你困惑的地方。

Harness 现在也有 **Worker Agents / Expert Agents**，可以让 Agent 作为 pipeline step 执行 code review、remediation、测试等工作，所以它已经不只是传统意义上的 CI/CD。citeturn0search2turn0search5

因此不是严格的：

```text
Multica = Agent
Harness = 非 Agent
```

而更像：

```text
                    软件开发生命周期

需求 ──→ 写代码 ──→ Review ──→ Build ──→ Test ──→ Security ──→ Deploy ──→ Production
        ↑                  ↑                                      ↑
        │                  │                                      │
    Multica 强项        两者开始重叠                         Harness 强项
```

### 放到你的 GH 插件项目就非常直观

你现在开发 **Data Tree Debug Agent**：

```text
Dashi / Linear
↓
“实现 Empty Branch 检测”

Multica
↓
分配给 GH Coding Agent
↓
Codex
↓
建立 Branch + Worktree
↓
写 C# 代码
↓
编译 / 测试
↓
Commit
↓
PR
```

到这里，**Multica 的价值已经非常明显。**

Harness 更适合继续往后：

```text
PR
↓
自动 Build
↓
自动 Test
↓
安全检查
↓
生成 Artifact
↓
部署测试环境
↓
审批
↓
发布正式版本
↓
监控
↓
有问题自动处理/回滚
```

而 Harness 特别强调的正是这种“从 commit 到 production”的治理式自动化。citeturn0search2turn0search9

所以对你**目前一个人开发 Grasshopper 插件**这个阶段，我会把优先级理解成：

```text
现在
★★★★★ Codex
★★★★☆ Git
★★★★☆ Branch / Worktree
★★★☆☆ GitHub
★★★☆☆ Dashi
★★☆☆☆ Multica
★☆☆☆☆ Harness
```

不是 Harness 不强，而是**你目前还没遇到它主要解决的问题**。

等以后你的 GH 插件真正商业化，出现：

```text
多个开发者
+
大量 AI Agent
+
自动测试
+
自动 Build
+
版本发布
+
正式/测试环境
+
安全策略
+
审批
+
自动部署
+
运行监控
```

Harness 的价值才会快速上升。

所以你现在可以形成一个非常清晰的分层：

> **Codex = AI 程序员**
>
> **Multica = AI 程序员团队管理/调度**
>
> **Git/GitHub = 代码版本与协作**
>
> **Harness = AI 软件交付、治理和上线平台**

而且它们**不是四选一**，理论上完全可以组成同一条链路。Harness 官方也明确表示可以和 GitHub、GitLab、Jira 等现有工具集成，而不是要求把整个工具链替换掉。citeturn0search6

[Multica 官方项目](https://github.com/eledax/multica-ai?utm_source=chatgpt.com) · [Harness 官方网站](https://www.harness.io/?utm_source=chatgpt.com)

---

## 对话 28

### 用户

那multica和deep seek harness有什么区别

### AI

对，这两个就**非常值得放在一起比较**了。你说的是 DeepSeek 最近发布的 **DeepSeek Harness（DSH）**。它目前还是 Developer Preview，而且官方明确把它定义为开源的 **Agent Harness / Agent 运行环境**。citeturn0search0turn0search11

先给你最重要的结论：

> **Multica 更像“AI 团队管理平台”。**  
> **DeepSeek Harness 更像“制造、运行和改造 AI Agent 的底层工作台/发动机”。**

它们不是完全同一层。

### 用你的 GH 项目理解最容易

假设你有三个角色：

```text
GH Coding Agent
GH Debug Agent
GH Architect Agent
```

在 **Multica** 里，你关注的是：

```text
                    你
                    ↓
                 Multica
             【AI 团队管理】
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
     Coding       Debug      Architect
      Agent        Agent        Agent
        ↓           ↓           ↓
      Task A       Task B       Task C
        ↓           ↓           ↓
       Run         Run          Run
        ↓           ↓           ↓
   各自执行具体开发任务
```

也就是：

> “**我已经有这些 AI 员工了，怎么给他们分任务、并行执行、跟踪工作？**”

---

而 **DeepSeek Harness** 更关注 Agent **里面到底装什么**：

```text
               GH Debug Agent
                     │
             DeepSeek Harness
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
     Model          Tools         Skills
       │             │             │
 DeepSeek V4      Shell        GH Debug Skill
 Codex模型等       Files        C# Skill
                  Search       Git Skill

       + Session
       + Sandbox
       + Storage
       + Agent Loop
       + Scheduling
       + Subagents
       + UI
```

DeepSeek 官方甚至直接用：

> **Agent = Model + Harness**

来解释这个关系。它认为模型只是 Agent 的“灵魂”，Harness 才负责让模型获得工具、环境、会话、执行循环等能力。DSH 的模型、工具、Skills、Session、Sandbox、Storage、Loop、Scheduling、UI 等都采用插件化设计，可以替换和重新组合。citeturn0search0turn0search1

---

### 所以二者关注的问题不同

|  | Multica | DeepSeek Harness |
|---|---|---|
| 核心定位 | **管理多个 Agent 工作** | **构建/运行 Agent** |
| 更像什么 | AI 团队管理办公室 | Agent 操作系统/发动机 |
| 关注 | Task、Issue、Agent、Run、并行 | Model、Tool、Skill、Session、Sandbox、Loop |
| 多 Agent | 强调调度多个 Coding Agent | 支持 Subagent/调度，但重点是 Harness 本身 |
| Worktree | 很适合 Coding Agent 并行任务 | 不是其最核心定位 |
| 自定义 Agent | 配置不同工作角色 | **非常强，插件化是核心设计** |
| 模型 | 调度 Codex/Claude 等工具 | 模型本身就是可替换插件 |
| 工具 | 使用外部 Coding Agent 的能力 | 可以自己组合 Shell、Files、Search 等 |
| 适合你现在 | 管理 GH 开发任务 | **打造你自己的 GH Agent** |

而且 DSH 不只是 DeepSeek 模型的一个 UI。官方明确说模型也是插件之一，整个目标就是让开发者自由组合 Agent 能力。citeturn0search0

### 这和 Codex 的关系也一下清楚了

你之前的知识体系是：

```text
Model
  ↓
Codex
  ↓
读代码
写代码
Shell
Git
Build
Test
```

其实从架构角度，**Codex 本身就已经包含了很多 Harness 的作用**。

可以粗略理解：

```text
            AI Model
               │
       ┌───────┴────────┐
       ↓                ↓
     Codex        DeepSeek Harness
       │                │
 成熟的 Coding       开放、插件化的
 Agent 产品          Agent Harness
       │                │
       ↓                ↓
     Tools            Tools
     Files            Files
     Shell            Shell
     Git              Skills
     etc.             Subagents
```

区别在于，DSH 特别强调**“Everything is a plugin”**：甚至模型、工具、Skills、会话、沙箱、存储、循环、调度、UI 都可以组合和替换。citeturn0search0

所以你可以把它想成：

> **Codex = OpenAI 已经帮你组装好的高性能 AI 程序员。**
>
> **DeepSeek Harness = 给你一套零件和底盘，让你自己组装 AI 程序员。**

当然 DSH 自己也提供 Standard Mode，可以直接作为完整 Coding Agent 使用，支持文件编辑、Shell、搜索、Skills、规划、Goals、Subagents 和 Workflows，所以它也不只是一个纯 SDK。citeturn0search1

---

### 最有意思的是：Multica 和 DeepSeek Harness 理论上可以上下组合

你的终极架构甚至可以是：

```text
                         你
                         │
                         ▼
                      Multica
                【管理 AI 开发团队】
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
     Coding Agent    Debug Agent   Architect Agent
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                DeepSeek Harness
                【Agent 运行环境】
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
        Model           Tools          Skills
          │              │              │
      DeepSeek         Shell        Grasshopper
      其他模型          Files        RhinoCommon
                       Git          C#
                       Search       Debug
                         │
                         ▼
                     你的电脑
                         │
                         ▼
                Branch + Worktree
                         │
                         ▼
                    GH 插件代码
```

这里就是：

**Multica 管“谁干什么”。**

**DeepSeek Harness 管“这个 Agent 到底怎么干”。**

这两个概念区别非常关键。

---

### 对你的 GH Data Tree Debug Agent，DSH 反而可能比 Harness.io 更值得研究

因为你最终不只是想**用 AI 开发 GH 插件**，你本身还想做一个 **GH 专属 Agent**。

例如未来你完全可以研究：

```text
Grasshopper Debug Agent
│
├── Model
│   └── DeepSeek / OpenAI / 其他
│
├── GH Tree Skill
│   ├── Tree Structure Analysis
│   ├── Path Analysis
│   ├── Empty Branch Detection
│   └── Data Matching Analysis
│
├── RhinoCommon Skill
│
├── Tools
│   ├── 读取 GH Document
│   ├── 获取 Component
│   ├── 获取 DataTree
│   ├── 分析 Runtime Message
│   └── 修改 GH 参数
│
├── Memory / Session
│
└── Agent Loop
    ├── 观察 GH
    ├── 分析问题
    ├── 调用工具
    ├── 检查结果
    └── 继续修复
```

这已经开始接近 **“用 Harness 构建你自己的 Grasshopper Agent”**，而不仅仅是“让 Codex 帮你写一个插件”。

DSH 当前还是开发者预览版，官方明确提醒核心插件/API 会继续快速变化、可能出现兼容性破坏，所以我不会建议你现在立刻把整个 GH 产品押在它上面；但**非常值得作为你的下一阶段技术方向研究**。citeturn0search11

[DeepSeek Harness 官方页面](https://www.deepseek.com/harness/en/?utm_source=chatgpt.com)

如果把你目前学过的东西按层级排一下，现在其实已经可以形成一张很漂亮的技术栈：

```text
                 Linear / Dashi
                 【需求 / Task】
                       ↓
                    Multica
                【Agent 团队管理】
                       ↓
        Codex / DeepSeek Harness
              【Agent 执行层】
                       ↓
                    Model
                 【AI 大脑】
                       ↓
             Tools / Skills / API
                       ↓
              Git + Branch + Worktree
                       ↓
                GH Plugin Project
                       ↓
                    GitHub
```

**你前面学的那些零散概念，到这里其实已经开始连成一套完整的 AI 软件开发架构了。**

---

## 对话 29

### 用户

但是在multica里面也可以构建agent也可以给agent加skill和规则，两者的差异在哪呢

### AI

对，你抓到了真正容易混淆的地方：**Multica 也能创建 Agent、配置 instructions/rules、skills、model、runtime，所以不能简单说“Multica 管 Agent，DeepSeek Harness 造 Agent”。**

更准确的区别是：

> **Multica 主要是在“应用/管理层”定义 Agent。**  
> **DeepSeek Harness 主要是在“运行时/基础设施层”定义 Agent。**

### 用“公司员工”类比最清楚

在 Multica 里创建一个：

```text
GH Debug Agent

角色：Grasshopper 调试工程师
规则：
- 先复现问题
- 再定位原因
- 最小修改
- 必须运行测试

Skills：
- Grasshopper
- C#
- RhinoCommon

Model：
- GPT / Claude / ...
```

这相当于：

> **给一个员工配置岗位说明书、技能包和工作环境。**

Multica关心的是：

```text
这个 Agent 是谁？
擅长什么？
用什么模型？
有什么 Skills？
把哪个 Issue 给它？
现在 Run 到哪里了？
能不能同时跑多个任务？
```

然后真正执行任务时，它通常还需要下面的 coding agent/runtime。

---

DeepSeek Harness 则继续往**更底层**走。

它不仅允许你说：

```text
这个 Agent 有 GH Debug Skill
```

还允许开发者改变：

```text
这个 Agent 的工具系统怎么实现？
Session 怎么保存？
Sandbox 怎么运行？
上下文怎么注入？
Agent Loop 怎么循环？
Subagent 怎么调度？
Storage 怎么实现？
甚至 UI 怎么实现？
```

DeepSeek 官方现在明确把 **models、tools、skills、sessions、sandboxes、storage、loops、scheduling、UI** 全部设计成可组合/替换的插件。citeturn0search0turn0search1

所以差别在这里：

```text
                 ┌──── Agent ────┐

Multica 更关注       ↑
                 “配置这个 Agent”
                 “管理这个 Agent”
                 “给它派任务”
                 “跟踪它的 Run”

────────────────────────────────────
                 ↓

DeepSeek Harness
更关注           “Agent 本身怎么运行”
                 “工具系统怎么工作”
                 “Loop 怎么工作”
                 “Session 怎么工作”
                 “Sandbox 怎么工作”
                 “上下文怎么进入模型”
                 “Subagent 怎么调度”
```

## 举一个特别具体的例子

假设你要做：

> **GH Debug Agent 可以读取 Grasshopper 当前 Data Tree。**

在 **Multica** 层面，你可能做的是：

```text
GH Debug Agent
│
├─ Rules
│   └─ 遇到 Tree 问题先检查路径结构
│
├─ Skills
│   └─ Grasshopper Data Tree Debug
│
├─ Model
│   └─ GPT / Claude
│
└─ Runtime
    └─ 我的 Windows 开发机
```

重点是：

> “**让这个 Agent 知道如何使用 GH 调试能力。**”

但假如现有 Coding Agent **根本没有能力直接读取正在运行的 Grasshopper Document**，你就遇到底层问题了。

你需要真正造一个工具，例如：

```text
get_grasshopper_tree()
        ↓
连接 Rhino / Grasshopper
        ↓
读取 GH_Document
        ↓
找到 Component
        ↓
读取 GH_Structure
        ↓
转换成 Agent 能理解的数据
        ↓
返回给 Model
```

这时候 DeepSeek Harness 这种东西的价值就明显了。

你可以把：

```text
Grasshopper Tool Plugin
        ↓
注册到 Harness
        ↓
Agent Loop
        ↓
Model 判断需要检查 Tree
        ↓
调用 get_grasshopper_tree()
        ↓
获得真实 GH 数据
        ↓
Model 分析
        ↓
再调用修改/诊断 Tool
```

也就是说，**你开始改造 Agent 的“身体”了，而不只是给它写岗位说明书。**

DeepSeek Harness 官方的设计正是让工具、Skills、Session、Sandbox、Agent Loop 等都成为可替换组件。citeturn0search0

---

## 所以“Skill”这个词在两边看起来一样，但层级可能不同

这是你现在最需要注意的。

```text
                Skill
                  │
        ┌─────────┴─────────┐
        ↓                   ↓
    知识/规则型 Skill      能力/工具型 Skill

“遇到 Empty Branch       “调用 GH API
 应该检查什么？”          读取当前 Data Tree”
```

前者更像：

```text
说明书
Prompt
规则
知识
流程
```

后者可能涉及：

```text
代码
API
Tool
Runtime
Process
Sandbox
事件系统
```

Multica 很适合管理前面的**Agent 配置和工作组织**。

DeepSeek Harness 则允许你继续深入后面的**Agent Runtime 本身**。

---

## 你甚至可以这样理解三层

这对你以后做 GH Agent 特别重要：

```text
┌─────────────────────────────────────┐
│              Multica                │
│                                     │
│   谁来干？干什么？现在干到哪里？      │
│                                     │
│ Coding Agent   Debug Agent   Review │
│     ↓              ↓          ↓     │
│ Task / Issue → Run → Status          │
└─────────────────┬───────────────────┘
                  ↓
       【Agent 管理 / 调度层】


┌─────────────────────────────────────┐
│         DeepSeek Harness            │
│                                     │
│ Agent 到底是怎么工作的？             │
│                                     │
│ Model                               │
│ Tools ←→ Skills                     │
│ Session                             │
│ Context                             │
│ Agent Loop                          │
│ Sandbox                             │
│ Storage                             │
│ Subagents                           │
└─────────────────┬───────────────────┘
                  ↓
          【Agent Runtime 层】


┌─────────────────────────────────────┐
│       Grasshopper / Windows         │
│                                     │
│ GH Document                         │
│ Data Tree                           │
│ RhinoCommon                         │
│ Files                               │
│ Shell                               │
│ Git                                 │
└─────────────────────────────────────┘
           【真实执行环境】
```

而 DeepSeek Harness 自己目前就是一个 **local-first 的 Coding Agent + Agent development/runtime environment**，并不只是一个 Skill SDK。citeturn0search2

### 那为什么看起来两者功能重叠这么多？

因为**确实重叠**。

现代 Agent 平台的边界正在快速融合：

```text
                 Multica
        Agent 配置 ███████████
        Skills     ███████
        Runtime    ███████
        Task管理   █████████████
        多Agent    █████████████


             DeepSeek Harness
        Agent 配置 ███████████
        Skills     ████████████
        Runtime    █████████████
        Task管理   █████
        多Agent    ████████
```

所以不要把它们理解成完全不相干的两个东西。

**区别主要是“产品重心”和“你控制到多底层”。**

---

## 对你的 GH 项目，这个区别非常重要

如果你的目标只是：

> “我要 5 个 AI 程序员帮我开发 GH 插件。”

那么：

```text
Multica
   ↓
Coding Agent
Debug Agent
Architect Agent
   ↓
Codex / Claude
```

已经非常合理。

但你的最终目标其实还有另一层：

> **“我要开发一个真正懂 Grasshopper、能够观察 Data Tree、诊断 Data Tree、甚至操作 GH 的专属 Agent。”**

这时候你研究 Harness 的意义就出来了：

```text
              你的 GH Agent
                    │
           ┌────────┼────────┐
           ↓        ↓        ↓
         Model    GH Tools   Skills
                    │
              RhinoCommon
                    │
              Grasshopper
                    │
               Data Tree
```

你可能需要的不只是：

> “告诉 AI 怎么分析 Data Tree。”

而是：

> **“赋予 AI 真正读取和操作 Grasshopper 的能力。”**

这已经进入 **Agent Harness / Runtime / Tool architecture** 的范畴了。

所以如果压缩成两句话：

> **Multica：我有一群 Agent，怎么定义他们的岗位、给他们任务、让他们并行工作并管理结果。**
>
> **DeepSeek Harness：我要深入 Agent 内部，决定它的大脑、工具、Skill、Session、Sandbox、Loop、上下文甚至运行方式是怎么组成的。**

对于你来说，**Multica 更像“开发你 GH Agent 产品时使用的管理工具”；DeepSeek Harness 则有可能成为“你 GH Agent 产品本身的技术底座之一”**。不过 DSH 目前仍是 Developer Preview，官方也说明核心插件和 API 还会继续演进，所以现阶段更适合研究和原型验证，而不是马上锁死产品架构。citeturn0search0

[DeepSeek Harness 官方介绍](https://deepseek.com/harness/?utm_source=chatgpt.com)

---

