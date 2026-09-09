---
title: 原始对话 01：Git 到 GitHub
tags:
  - 原始对话
  - Git
source_thread: 名词解释
source_turns: 1-13
---

# 原始对话 01：Git 到 GitHub

> 从“解释 git, worktree, multica, linear, dashi-taskboard……”开始，按原始时间顺序存档。文字保持原貌；对话附图只保留附件标记。

## 对话 1

### 用户

解释git,worktree,multica,linear,dashi-taskboard分别是干什么，怎么操作，工作逻辑，和优劣对比

### AI

可以把这 5 个东西理解成**不同层级的开发工具**，它们不是互相替代关系，而是可以叠在一起用：

> **Git 管代码历史 → Worktree 让多个任务并行开发 → Linear / Dashi 管任务 → Multica 让 AI Agent 自动接任务干活。**

你写的 **multica** 我查到的是现在这个开源的 **Multica AI**：它专门把 Codex、Claude Code、Cursor 等编码 Agent 当成“团队成员”来分配任务。citeturn614552search0turn614552search1

---

## 一、先用一张表理解它们

| 工具 | 本质 | 解决什么问题 | 是否直接写代码 |
|---|---|---|---|
| **Git** | 版本控制系统 | 代码修改、版本、回退、合并 | ❌ |
| **Git Worktree** | Git 的多工作目录功能 | 同一个项目同时开发多个任务 | ❌ |
| **Multica** | AI Agent 管理/调度平台 | 把任务自动分给 Codex/Claude 等 Agent | ✅ Agent 会写 |
| **Linear** | 专业项目/Issue 管理 | 管 Bug、需求、优先级、Sprint、Roadmap | ❌ |
| **Dashi Taskboard** | 本地优先的 AI/Codex 任务板 | 管 Codex 开发任务，并和本地代码关联 | ✅ 可配合 Codex |

最关键的一点：

**Git / Worktree 是“代码层”。**

**Linear / Dashi 是“任务层”。**

**Multica 是“Agent 执行层”。**

所以并不是：

> Git vs Linear vs Multica

而更像：

```text
项目管理
    ↓
Linear / Dashi
    ↓
任务
    ↓
Multica / Codex
    ↓
Git Worktree
    ↓
代码修改
    ↓
Git Commit
    ↓
Merge
```

---

# 二、Git 是什么

Git 是一个**分布式版本控制系统**。

它最核心的作用其实不是“上传 GitHub”，而是：

> **记录你的代码每一次变化。**

Git 会把项目历史保存成一个个：

```text
Commit
```

你可以把 Commit 理解成：

> 游戏存档点。

例如你开发 GH 插件：

```text
v0
创建项目

↓ commit

v1
加入 Data Tree Analyzer

↓ commit

v2
加入 Path Inspector

↓ commit

v3
加入 AI Debug Agent
```

哪天 v3 出问题了：

```text
回到 v2
```

就可以。

Git 官方对它的定义就是分布式版本控制系统，每个完整仓库都包含项目和历史记录。citeturn718573search4turn718573search5

### 最基本操作

典型流程：

```bash
git init
```

把目录变成 Git 项目。

修改代码之后：

```bash
git status
```

看看改了什么。

然后：

```bash
git add .
```

准备提交。

```bash
git commit -m "Add tree analyzer"
```

生成一次版本记录。

查看历史：

```bash
git log
```

创建新分支：

```bash
git switch -c feature/tree-analyzer
```

完成后：

```bash
git switch main
git merge feature/tree-analyzer
```

### Git 的工作逻辑

```text
文件
 ↓
Working Directory
 ↓ git add
Staging Area
 ↓ git commit
Repository
 ↓ git push
GitHub
```

特别要注意：

> **Git ≠ GitHub**

Git 是版本控制软件。

GitHub 是托管 Git 仓库的网站。

### 优点

非常成熟、可靠、生态巨大；可以回退；可以开分支；可以多人协作；几乎所有 AI 编码工具都默认围绕 Git 工作。

缺点是：

**Git 本身不负责“任务管理”。**

你不能指望 Git 告诉你：

```text
今天该做什么
哪个 Bug 最重要
Codex 正在处理哪个任务
这个功能做到多少了
```

这就是 Linear / Dashi / Multica 出现的原因。

---

# 三、Worktree 是什么

这个非常重要，尤其适合你现在这种 **Codex + 多任务开发**。

Worktree 不是另一个软件。

它其实是：

> **Git 自带的功能。**

官方定义就是：

> 一个 Git 仓库可以同时拥有多个 Working Tree，从而同时 checkout 多个 branch。citeturn718573search0turn718573search1

正常情况下，你只有：

```text
H:\GH-Agent
```

然后你正在开发：

```text
feature-A
```

突然想开发：

```text
feature-B
```

通常要：

```text
git switch feature-B
```

问题来了。

feature-A 可能还有一堆没完成的代码。

于是你经常：

```text
stash
switch
开发
switch
stash pop
```

很麻烦。

Worktree 就是解决这个问题的。

---

## Worktree 的逻辑

假设主项目：

```text
H:\GH-Agent
```

你可以创建：

```text
H:\GH-Agent-tree-analyzer
H:\GH-Agent-ui
H:\GH-Agent-hotfix
```

它们分别对应：

```text
main
feature/tree-analyzer
feature/ui
hotfix/crash
```

于是：

```text
                Git Repository
                     │
         ┌───────────┼───────────┐
         ↓           ↓           ↓
      main        feature-A   feature-B
         ↓           ↓           ↓
H:\GH-Agent   H:\GH-A     H:\GH-B
```

但它们仍然属于**同一个 Git Repository**。

---

## 实际操作

创建：

```bash
git worktree add ../GH-Agent-ui -b feature/ui
```

Git 会同时：

1. 创建 `feature/ui`
2. 创建新目录
3. 把这个分支放进去

查看：

```bash
git worktree list
```

删除：

```bash
git worktree remove ../GH-Agent-ui
```

---

# 四、Worktree 为什么特别适合 AI Agent

假设你同时让三个 Codex 工作：

```text
Codex A
开发 Tree Analyzer

Codex B
开发 UI

Codex C
修复 Crash
```

千万不要让它们同时操作：

```text
H:\GH-Agent
```

因为它们很可能同时修改：

```text
Plugin.cs
TreeAnalyzer.cs
UI.cs
```

最终：

```text
Agent A 修改文件
          ↓
Agent B 又修改
          ↓
Agent C 再修改
          ↓
互相覆盖 / context 混乱
```

Worktree 后：

```text
Agent A
H:\worktrees\tree-analyzer

Agent B
H:\worktrees\ui

Agent C
H:\worktrees\hotfix
```

完全隔离。

最后：

```text
feature/tree-analyzer ─┐
feature/ui ────────────┼→ main
hotfix/crash ──────────┘
```

这其实是现在 AI Coding 并行开发非常重要的一种基础设施。

### Worktree 优点

最大的优点是：

**便宜、原生、稳定、隔离清楚。**

而且不像重新 `git clone` 三份仓库那么浪费。

### 缺点

它只解决：

> **“代码目录隔离”**

它不知道：

```text
任务是什么
谁负责
优先级多少
做到哪里
Agent 是否卡住
```

所以 Worktree 一般需要搭配任务系统。

---

# 五、Linear 是什么

Linear 是一个专业的：

> **软件项目 / Issue 管理系统。**

它主要服务：

```text
产品经理
开发人员
设计师
测试
团队负责人
```

Linear 里的最基本单位叫：

> **Issue**

官方的概念也是：Issue 是 Linear 最基本的工作单元，可以代表 Bug、Feature、内部任务等，并可以关联负责人、项目、Cycle、标签、优先级。citeturn614552search7

例如：

```text
GH-101
Tree Analyzer 无法识别空 Branch

Type:
Bug

Priority:
High

Status:
Todo

Assignee:
Tianyi
```

然后状态：

```text
Backlog
   ↓
Todo
   ↓
In Progress
   ↓
In Review
   ↓
Done
```

---

## Linear 的核心逻辑

```text
公司目标
   ↓
Initiative
   ↓
Project
   ↓
Issues
   ↓
Sub Issues
```

再配合：

```text
Cycle
Priority
Label
Assignee
Roadmap
```

所以 Linear 更像：

> **开发团队的指挥中心。**

---

## Linear 怎么操作

比如开发你的 GH 插件：

创建 Project：

```text
Data Tree Debug Agent v1.0
```

然后 Issue：

```text
DT-01
实现 Tree Snapshot

DT-02
实现 Branch Diff

DT-03
实现 Path Inspector

DT-04
实现 AI Error Explanation

DT-05
修复 Null Tree Crash
```

设置：

```text
Priority
Assignee
Due Date
Status
```

开发的时候：

```text
Todo
 ↓
In Progress
 ↓
Review
 ↓
Done
```

Linear 还能和 GitHub 等工具连接，让开发状态和 PR/Issue 管理关联起来。citeturn614552search6

---

## Linear 优劣

优势非常明显：

**项目管理能力强。**

特别适合：

```text
Roadmap
Sprint/Cycle
Bug
Feature
Release
多人协作
长期项目
```

UI 和任务组织也很成熟。

但是：

它本质上不是：

> AI Agent Runtime。

也就是说 Linear 告诉你：

```text
“Tree Analyzer 要做”
```

但它本身并不等于：

```text
“Codex 已经自动打开代码开始写了。”
```

虽然现在很多 Agent 平台可以和 Linear 集成。

---

# 六、Dashi Taskboard 是什么

这个和你现在使用 Codex 的场景关系非常大。

Dashi Taskboard 的项目 README 对自己的定义是：

> **local-first issue board**

也就是一个：

> **本地优先的开发任务板。**

它运行在浏览器里，同时有 HTTP API 和 `taskctl` CLI，可以被 Codex Skill 操作。citeturn614552search8

简单说：

> **它有点像“专门围绕 Codex 做的轻量版 Linear”。**

---

# 七、Dashi 工作逻辑

例如：

```text
Project
GH Data Tree Debug Agent

│
├─ Task 01
│  Tree Snapshot
│
├─ Task 02
│  Path Analyzer
│
├─ Task 03
│  UI
│
└─ Task 04
   Crash Fix
```

Dashi 可以给任务记录：

```text
Status
Priority
Label
Branch
Worktree
Start Date
Due Date
```

它的 CLI 甚至直接支持：

```text
--git-branch

--worktree-path

--worktree-branch
```

也就是说它天然考虑了：

> **Task ↔ Git Branch ↔ Worktree**

这种开发模型。citeturn718573search8

这就是它对于 Codex 很有意思的地方。

---

## Dashi 操作方式

启动：

```bash
npm start
```

默认打开：

```text
127.0.0.1:47823
```

数据存在本地：

```text
.data/taskboard.sqlite
```

创建项目可以类似：

```bash
taskctl project create \
  --id gh-agent \
  --name "GH Data Tree Agent" \
  --workspace-path H:\GH-Agent
```

创建任务：

```bash
taskctl issue create \
  --project gh-agent \
  --title "Implement Tree Analyzer"
```

Codex 可以通过 Skill 操作这些任务。

---

# 八、Dashi 和 Linear 最大区别

这个非常重要。

Linear 更像：

```text
公司项目管理系统
```

Dashi 更像：

```text
开发者 + Codex 的本地任务控制面板
```

所以：

| | Linear | Dashi |
|---|---|---|
| 项目管理 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Roadmap | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| 团队协作 | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| 本地代码关联 | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Codex 工作流 | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Worktree 关联 | 间接 | 很直接 |
| 安装 | SaaS 为主 | 本地 |
| 数据 | 云端为主 | SQLite 本地 |
| 简洁程度 | 中等 | 高 |
| 适合个人 AI 开发 | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

---

# 九、Multica 是什么

Multica 就开始进入更高级的一层了。

它解决的是：

> **“我不是只有一个 Codex，而是有很多 AI Agent，怎么管理它们？”**

Multica 支持把 Codex、Claude Code、Cursor 等编码工具作为 Agent，然后像给人分配 Issue 一样给 Agent 分配任务。citeturn614552search0turn614552search13

例如：

```text
Issue #101
Tree Analyzer

Assignee:
Codex-Agent-A
```

然后 Agent：

```text
领取任务
   ↓
读取 Issue
   ↓
打开项目
   ↓
运行 Codex
   ↓
修改代码
   ↓
测试
   ↓
汇报进度
   ↓
完成
```

Multica 本身负责的是：

```text
Issue
Agent
Run
Runtime
Status
Comments
Skills
Automation
```

真正运行 Codex 的还是你的电脑/runtime。Multica 官方架构也是：

```text
Multica Server
记录 / 调度

        ↓

Connected Computer
Agent Runtime

        ↓

Codex / Claude Code

        ↓

本地代码
``` citeturn614552search1


---

# 十、Multica + Worktree 才真正有意思

Multica 甚至本身就支持两种执行模式：

```text
Direct
```

Agent 直接修改当前目录。

和：

```text
Parallel / Worktree
```

每个 Run 创建自己的 Git Worktree。

这样多个任务就可以同时执行，而且不会写到同一份 Working Copy。citeturn614552search14

于是：

```text
                 Multica
                    │
          ┌─────────┼─────────┐
          ↓         ↓         ↓
      Agent A    Agent B    Agent C
          │         │         │
          ↓         ↓         ↓
     Worktree A Worktree B Worktree C
          │         │         │
        Codex     Claude    Codex
          │         │         │
       Feature A  Feature B   Bug Fix
```

这才是真正的：

> **多 Agent 并行开发。**

---

# 十一、Multica 的优缺点

### 优点

它真正解决了：

```text
多个 Agent 怎么管理
哪个 Agent 做哪个任务
Agent 卡住了吗
Run 执行到哪里
结果是什么
如何复用 Agent Skill
如何自动化
```

而且它是 open-source/self-hostable，并支持多种 Coding Agent。citeturn614552search5turn614552search13

特别适合：

```text
Codex Agent A
Codex Agent B
Claude Agent C
Review Agent
Test Agent
Documentation Agent
```

形成 AI 开发团队。

### 缺点

相对复杂。

你需要理解：

```text
Git
Branch
Worktree
Agent
Runtime
Issue
Run
Merge
```

才能真正发挥价值。

如果现在只有：

```text
你 + 一个 Codex
```

Multica 可能稍微有点重。

---

# 十二、所以这五个东西真正的层级关系

我建议你记住这张图：

```text
┌────────────────────────────┐
│        Linear              │
│    产品 / 项目 / Roadmap    │
└────────────┬───────────────┘
             │
             │ Issues
             ↓
┌────────────────────────────┐
│   Dashi / Multica          │
│     任务执行管理层           │
└────────────┬───────────────┘
             │
             │ Assign
             ↓
┌────────────────────────────┐
│     Codex / Claude         │
│       AI Agent             │
└────────────┬───────────────┘
             │
             ↓
┌────────────────────────────┐
│        Worktree            │
│       独立开发目录           │
└────────────┬───────────────┘
             │
             ↓
┌────────────────────────────┐
│           Git              │
│ Branch / Commit / Merge    │
└────────────┬───────────────┘
             │
             ↓
           GitHub
```

---

# 十三、对你现在最实用的理解

你现在开发 **GH / Grasshopper 插件 + Codex**，我不建议一开始把所有东西全部堆上去。

你可以分三个阶段。

### 第一阶段：先掌握

```text
Git
+
Branch
+
Worktree
+
Dashi
+
Codex
```

工作流程：

```text
① Dashi 创建任务

       ↓

② 每个 Task 创建 Branch

       ↓

③ 每个 Branch 创建 Worktree

       ↓

④ Codex 在 Worktree 开发

       ↓

⑤ Codex 测试

       ↓

⑥ Git Commit

       ↓

⑦ 人工 Review

       ↓

⑧ Merge main

       ↓

⑨ Dashi Task → Done
```

这已经是一个非常好的 **AI 编程工作流**。

---

## 第二阶段

当任务越来越多：

```text
Codex A
Codex B
Codex C
```

再加入：

> **Multica**

变成：

```text
Task
 ↓
Multica
 ↓
自动选 Agent
 ↓
自动建立 Worktree
 ↓
Codex 开发
 ↓
测试
 ↓
Review
 ↓
Merge
```

---

# 十四、什么时候才需要 Linear

如果你的 GH 插件以后真的开始：

```text
商业化
```

有：

```text
产品 Roadmap
用户反馈
Bug
Feature Request
版本计划
设计
开发
测试
市场
```

这时再上 Linear 非常合适。

例如：

```text
Linear

Data Tree Debug Agent
│
├── v0.1 Prototype
├── v0.5 Beta
├── v1.0 Release
└── v2.0 AI Agent
```

下面再拆：

```text
Project
 ↓
Issue
 ↓
Dashi / Multica
 ↓
Codex
 ↓
Worktree
 ↓
Git
```

---

# 十五、如果只问“哪个最好”

其实不存在一个“最好”。

如果按你的当前阶段，我会这样选：

| 需求 | 推荐 |
|---|---|
| 保存代码版本 | **Git 必须** |
| 同时开发多个功能 | **Worktree 强烈推荐** |
| 管自己的 Codex 任务 | **Dashi 很合适** |
| 多 AI Agent 并行 | **Multica** |
| 商业项目/团队/路线图 | **Linear** |

所以你现在最值得建立的是：

> **Git + Worktree + Dashi + Codex**

而不是一上来：

> Linear + Dashi + Multica + GitHub + 5 个 Agent

否则工具本身会变成新的管理负担。

等你对这套工作流熟悉后，再加入 **Multica**，把：

```text
“我给 Codex 下任务”
```

升级成：

```text
“任务系统自动调度多个 Codex/Claude Agent”
```

这会更自然。

顺带一提，**Dashi 和 Multica其实存在一定功能重叠**：两者都涉及 Issue/Agent 工作流；Dashi 更轻、更本地、更偏 Codex，Multica则明显更偏“多 Agent 团队管理和自动执行”。因此你未来未必需要两个同时长期使用。citeturn614552search1turn614552search8

我也可以给你直接画一张 **「Git → Worktree → Dashi → Codex → Multica → Linear」完整工作逻辑架构图**，把每一步谁负责什么画清楚。

---

## 对话 2

### 用户

给我一个http文件，让我看得明白

### AI

我已经给你做成了一个**单文件 HTML 图解页面**，可以直接双击用浏览器打开，里面把 Git、Worktree、Dashi、Multica、Linear 的关系、流程和对比都画清楚了。

:chatgpt-content-reference{index="0"}

建议你打开后先看第 **1、4、7、9** 部分，基本就能快速理解整套工作逻辑。

---

## 对话 3

### 用户

这里的每一个agent可以理解为codex里面的一个对话框吗

[User attached 1 image; image contents were not included]

### AI

对，**为了理解这张图，你可以暂时把每个 Agent 理解成 Codex 里的一个独立对话/任务**。但更准确地说：

> **Agent ≈ 一个独立执行任务的 Codex 实例/会话，而 Worktree ≈ 这个 Codex 实例专属的代码工作区。**

比如你同时开 3 个 Codex 任务：

```text
你的 GH-Agent 项目
│
├─ Codex 对话 A
│   任务：开发 UI
│   ↓
│   Worktree：H:\GH-Agent-ui
│   Branch：feature/ui
│
├─ Codex 对话 B
│   任务：开发 Tree Analyzer
│   ↓
│   Worktree：H:\GH-Agent-tree
│   Branch：feature/tree
│
└─ Codex 对话 C
    任务：修复崩溃 Bug
    ↓
    Worktree：H:\GH-Agent-hotfix
    Branch：hotfix/crash
```

这时候就可以近似理解成：

**1 个 Codex 对话/任务 → 1 个 Agent → 1 个 Worktree → 1 个 Git Branch**

### 为什么要这么做？

假设 A 对话告诉 Codex：

> “帮我重构 UI。”

与此同时 B 对话告诉 Codex：

> “帮我修改 Tree Analyzer。”

如果两个 Codex 都操作：

```text
H:\GH-Agent
```

它们看到的是**同一批正在变化的文件**。A 刚修改完一个文件，B 马上就能看到，甚至可能继续修改它。

用了 Worktree：

```text
Codex A → H:\GH-Agent-ui
Codex B → H:\GH-Agent-tree
```

两个 Codex 就像分别坐在两个房间里工作。

最后通过 Git：

```text
feature/ui ────────┐
                   ├──→ main
feature/tree ──────┘
```

把成果汇总回来。

### 不过有一个重要区别

**“Codex 对话框”不严格等于“Agent”。**

“对话框”更多是你和 Codex 的**交互界面/会话**；Agent 是背后真正执行任务的角色。以后使用 Multica 之类的系统时，一个 Agent 甚至可以自动领取任务、运行 Codex、操作 Worktree，而不需要你手工一直开着一个聊天窗口。

所以你现阶段可以先这样记：

> **Codex 对话 = 工人**  
> **Dashi Task = 工单**  
> **Worktree = 每个工人的独立工作台**  
> **Branch = 这个工人正在做的版本线**  
> **Git = 所有施工记录和版本档案**  
> **main = 最终正式产品**

这个比喻基本就能把你现在的 **Dashi + Codex + Worktree + Git** 工作逻辑串起来了。

---

## 对话 4

### 用户

所以只要开新的codex对话就是新的workflow是吗？还是说可以在一个人物运行一半的时候分出一个workflow

### AI

不是。这里最好把 **Codex 对话、任务、Workflow、Worktree、Branch** 分开理解。

**开一个新的 Codex 对话，不等于自动创建一个新的 Workflow/Worktree。** 新对话只是多了一个独立的上下文；它是否成为新的并行开发任务，取决于你有没有给它独立的 Branch / Worktree。

而且你说的第二种情况完全可以：**一个任务做到一半，可以从当前进度“分叉”出另一个 Workflow。**

例如你的主任务正在做到这里：

```text
main
 │
 A
 │
 B
 │
 C  ← 当前已经开发到这里
```

现在你突然发现：

> UI 和 Data Tree 核心逻辑可以分开同时做。

就可以从当前 C 分出去：

```text
                    ┌→ feature/ui
                    │   ↓
main → A → B → C ───┤   Codex 对话 B
                    │
                    └→ feature/tree
                        ↓
                        Codex 对话 C
```

这两个新的 Codex 对话都可以**继承 C 时刻已经存在的代码**，然后分别继续开发。

比如物理目录变成：

```text
H:\GH-Agent
        │
        │ 当前项目
        │
        ├──────────────┐
        ↓              ↓
H:\GH-Agent-ui     H:\GH-Agent-tree
feature/ui         feature/tree
    │                  │
Codex B             Codex C
    │                  │
开发 UI             开发 Tree
```

这里最关键的是：**真正决定代码从哪里分叉的，不是“新建 Codex 对话”，而是 Git Branch / Worktree。**

所以有两种很不同的操作：

```text
只开新 Codex 对话
        ↓
只是新的 AI 上下文
        ↓
如果仍指向 H:\GH-Agent
        ↓
两个 Codex 仍可能操作同一批文件
```

而：

```text
创建 Branch
      +
创建 Worktree
      +
新 Codex 对话指向这个 Worktree
        ↓
真正形成独立并行开发线
```

因此你以后可以把规则简化成一句：

> **“新对话”是新脑子；“新 Worktree + Branch”才是新施工现场。**

对于你现在做 GH 插件，我更推荐采用 **“需要并行时再分叉”**，而不是每开一个 Codex 对话就创建 Worktree。

例如：

```text
任务：开发 Data Tree Debug Agent
              │
        Codex 主任务开发
              │
       做到基础架构完成
              │
      ┌───────┼────────┐
      ↓       ↓        ↓
   UI开发   Tree算法   Hotfix
      ↓       ↓        ↓
 Worktree A Worktree B Worktree C
      ↓       ↓        ↓
 Codex A   Codex B   Codex C
      └───────┼────────┘
              ↓
            Merge
              ↓
             main
```

**这才是 Worktree 最有价值的地方：不是“每个聊天都隔离”，而是当一个开发任务进行到某个节点后，可以从这个节点复制出多个平行开发世界。**

另外还有一个很重要的问题：**什么时候应该继续原 Codex 对话，什么时候应该新开对话 + Worktree？** 这个判断搞清楚之后，你用 Codex 做复杂项目会顺很多。

---

## 对话 5

### 用户

Git Branch / Worktree 有什么区别？分别怎么操作呢

### AI

这是你理解 **Codex 多任务并行开发**最关键的一组概念。

最简单的一句话：

> **Branch（分支）= 一条独立的代码版本线。**  
> **Worktree（工作树）= 把某条 Branch 真正放到一个独立文件夹里，让你可以同时操作它。**

它们不是二选一，而经常是 **Branch + Worktree 配套使用**。

---

## 1. 先理解 Branch

假设你的项目：

```text
H:\GH-Agent
```

目前正式代码是：

```text
main

A ── B ── C
```

现在你想开发 UI，但又不希望影响正式代码。

于是创建：

```bash
git switch -c feature/ui
```

Git 就产生：

```text
             ┌── feature/ui
             ↓
A ── B ── C
             ↑
            main
```

此时两个分支内容完全一样。

接下来你修改 UI 并 Commit：

```text
main
A ── B ── C
             \
              D ── E
                  feature/ui
```

所以 Branch 可以理解成：

> **从某个版本节点开始，开出一条新的开发路线。**

---

# 2. Branch 有一个问题

假设你现在有：

```text
main
feature/ui
feature/tree
hotfix/crash
```

但是电脑上只有：

```text
H:\GH-Agent
```

这一个项目文件夹。

你可以：

```bash
git switch feature/ui
```

此时：

```text
H:\GH-Agent
      ↓
feature/ui
```

然后又：

```bash
git switch feature/tree
```

就变成：

```text
H:\GH-Agent
      ↓
feature/tree
```

注意：

**文件夹还是同一个。**

只是里面的代码被 Git 切换成不同 Branch 的状态。

所以：

```text
一个文件夹

H:\GH-Agent
     │
     ├── switch main
     │
     ├── switch feature/ui
     │
     └── switch feature/tree
```

你一次通常只在这个目录里操作其中一个分支。

---

# 3. Worktree 就是解决这个问题

Worktree 可以让：

> **多个 Branch 同时出现在电脑上的多个文件夹里。**

例如：

```text
Git 仓库
│
├── main
│      ↓
│   H:\GH-Agent
│
├── feature/ui
│      ↓
│   H:\GH-Agent-ui
│
└── feature/tree
       ↓
    H:\GH-Agent-tree
```

现在三个目录可以**同时存在**。

你不需要不停：

```bash
git switch
```

了。

---

# 4. 用房子的比喻最好理解

假设你的 GH 插件是一套建筑设计。

### Branch 是“设计方案”

```text
方案 A：main

方案 B：feature/ui

方案 C：feature/tree
```

这些是不同的**设计版本线**。

而 Worktree 是：

> **真正给这些方案准备独立的工作室。**

于是：

```text
设计方案                     工作室

main                 →    H:\GH-Agent

feature/ui           →    H:\GH-Agent-ui

feature/tree         →    H:\GH-Agent-tree
```

所以：

> **Branch 是逻辑上的分叉。**
>
> **Worktree 是物理文件夹上的分离。**

这是最核心的区别。

---

# 5. 为什么 Codex 特别需要 Worktree

假设你开两个 Codex 对话。

### Codex A

任务：

> 开发 UI

### Codex B

任务：

> 开发 Tree Analyzer

如果只有 Branch，但都操作：

```text
H:\GH-Agent
```

你会遇到一个麻烦：

Codex A：

```text
git switch feature/ui
```

正在开发。

Codex B 又：

```text
git switch feature/tree
```

那么这个目录的 Branch 就被切走了。

甚至两个 Codex 同时修改同一批文件。

这就很危险。

---

## 6. Worktree 后就漂亮很多

直接：

```text
Codex A
   ↓
H:\GH-Agent-ui
   ↓
feature/ui


Codex B
   ↓
H:\GH-Agent-tree
   ↓
feature/tree


Codex C
   ↓
H:\GH-Agent-hotfix
   ↓
hotfix/crash
```

三个 Codex 可以：

**同时工作。**

互相基本不碰对方的工作目录。

这就是：

> **Codex 并行开发 + Git Worktree**

非常重要的原因。

---

# 7. Branch 怎么操作？

假设你当前在：

```text
H:\GH-Agent
```

先看看当前分支：

```bash
git branch
```

可能显示：

```text
* main
```

`*` 表示当前 Branch。

---

### 创建新 Branch

```bash
git switch -c feature/ui
```

意思是：

```text
创建：
feature/ui

并且：

切换过去
```

查看：

```bash
git branch
```

变成：

```text
main
* feature/ui
```

---

### 切回 main

```bash
git switch main
```

---

### 再切回来

```bash
git switch feature/ui
```

所以普通 Branch 工作模式就是：

```text
H:\GH-Agent

      ↓

main
 ↑
 │ git switch
 ↓
feature/ui
 ↑
 │ git switch
 ↓
feature/tree
```

**一个目录，不断切 Branch。**

---

# 8. Worktree 怎么操作？

现在重点来了。

假设：

```text
H:\GH-Agent
```

目前是：

```text
main
```

你想让 Codex 开一个独立 UI 开发任务。

直接：

```bash
git worktree add H:\GH-Agent-ui -b feature/ui
```

这条命令一次做两件事：

```text
① 创建 Branch

feature/ui


② 创建 Worktree

H:\GH-Agent-ui
```

最终：

```text
H:\GH-Agent
     ↓
    main


H:\GH-Agent-ui
     ↓
feature/ui
```

---

# 9. 再开一个 Tree 开发任务

执行：

```bash
git worktree add H:\GH-Agent-tree -b feature/tree
```

现在：

```text
                  GH-Agent Git Repository
                           │
              ┌────────────┼────────────┐
              │            │            │
              ↓            ↓            ↓

             main      feature/ui   feature/tree
              │            │            │
              ↓            ↓            ↓

H:\GH-Agent      H:\GH-Agent-ui    H:\GH-Agent-tree
     │                 │                  │
     ↓                 ↓                  ↓
主 Codex            Codex A             Codex B
```

这时候就可以真正并行开发。

---

# 10. 怎么查看所有 Worktree？

运行：

```bash
git worktree list
```

可能看到：

```text
H:/GH-Agent       abc123 [main]

H:/GH-Agent-ui    abc123 [feature/ui]

H:/GH-Agent-tree  abc123 [feature/tree]
```

一眼就知道：

```text
哪个文件夹
    ↓
对应哪个 Branch
```

---

# 11. 开发完成以后怎么办？

假设 Codex A 完成 UI。

进入：

```text
H:\GH-Agent-ui
```

Codex 做：

```bash
git add .
git commit -m "Add new UI"
```

于是：

```text
feature/ui
      ↓
已经保存成果
```

然后回主目录：

```bash
cd H:\GH-Agent
```

确保：

```bash
git switch main
```

然后：

```bash
git merge feature/ui
```

变成：

```text
                  feature/ui
                 /
A ── B ── C ── D
              ↑
             main
```

UI 功能正式进入主线。

---

# 12. Worktree 用完可以删除

确认内容已经 Commit / Merge 后：

```bash
git worktree remove H:\GH-Agent-ui
```

注意：

**删除 Worktree ≠ 删除 Branch。**

此时：

```text
H:\GH-Agent-ui
```

这个工作目录没了。

但是：

```text
feature/ui
```

Branch 仍然可以存在。

如果确认 Branch 也不要了：

```bash
git branch -d feature/ui
```

完整生命周期就是：

```text
创建 Branch + Worktree
        ↓
Codex 开发
        ↓
Commit
        ↓
Merge main
        ↓
删除 Worktree
        ↓
删除 Branch
```

---

# 13. 还有一种情况：Branch 已经存在

比如已经有：

```text
feature/ui
```

只是还没有 Worktree。

那就不要用：

```text
-b feature/ui
```

因为 `-b` 是“创建新 Branch”。

应该：

```bash
git worktree add H:\GH-Agent-ui feature/ui
```

意思就是：

> 把已经存在的 `feature/ui` 放到 `H:\GH-Agent-ui`。

所以记两个命令就够了：

```bash
# Branch 不存在：创建 Branch + Worktree
git worktree add H:\GH-Agent-ui -b feature/ui
```

```bash
# Branch 已存在：只创建 Worktree
git worktree add H:\GH-Agent-ui feature/ui
```

---

# 14. Branch 和 Worktree 对比

| | Branch | Worktree |
|---|---|---|
| 中文 | 分支 | 工作树/工作目录 |
| 本质 | Git 的版本路线 | 实际文件目录 |
| 会创建文件夹吗 | ❌ | ✅ |
| 能保存独立开发历史吗 | ✅ | 本身不是历史 |
| 能并行打开吗 | 逻辑上可以 | ✅ 物理上真正隔离 |
| Codex 多任务 | 必要 | **非常推荐** |
| 是否属于 Git | ✅ | ✅ |

所以不要理解成：

> Branch **VS** Worktree

而应该理解成：

> **Branch + Worktree**

---

# 15. 对你现在最重要的工作方式

假设你的 GH 插件在：

```text
H:\GH-Agent
```

以后你给 Codex 一个比较大的独立任务：

> 开发 Tree Visualizer

不要让它直接在 main 上乱改。

创建：

```bash
git worktree add H:\GH-Agent-tree-visualizer -b feature/tree-visualizer
```

然后**新开一个 Codex 任务/对话，把工作目录指定为：**

```text
H:\GH-Agent-tree-visualizer
```

于是：

```text
Dashi
  │
  │ Task
  ↓
Tree Visualizer
  │
  ↓
Codex 对话
  │
  ↓
H:\GH-Agent-tree-visualizer     ← Worktree
  │
  ↓
feature/tree-visualizer         ← Branch
  │
  ↓
修改代码
  │
  ↓
Commit
  │
  ↓
Review
  │
  ↓
Merge
  │
  ↓
main
```

**这套逻辑一旦理解，你后面用 2 个、3 个甚至更多 Codex 并行开发就很好理解了。**

而且你前面问的“**一个 Codex 做到一半能不能分出另一个 Workflow**”，答案也就清楚了：**可以——在那个代码节点创建新 Branch，再给它创建 Worktree，然后让新的 Codex 对话进入那个 Worktree。**

---

## 对话 6

### 用户

所以branch是代码层面的，worktree是物理层面的文件夹，workflow=branch+worktree是吗？

### AI

对，**前两句基本完全正确，但第三句需要稍微修正一下**：

> **Branch = 代码版本层面的分支**  
> **Worktree = 物理文件夹层面的独立工作区**  
> **Workflow ≠ Branch + Worktree，但在你的 Codex 并行开发场景里，可以暂时这样理解。**

更准确地说，**Workflow（工作流）是整个任务从开始到完成的一套流程**，Branch + Worktree 只是其中的代码执行环境。

比如你的 GH 插件：

```text
Workflow：开发 Tree Analyzer
│
├── Task
│   └── “实现 Tree Analyzer”
│
├── Codex 对话 / Agent
│   └── 负责执行
│
├── Branch                  ← 代码版本线
│   └── feature/tree
│
├── Worktree                ← 物理工作目录
│   └── H:\GH-Agent-tree
│
├── Coding
│   └── Codex 修改代码
│
├── Test
│
├── Commit
│
├── Review
│
└── Merge → main
```

所以你可以建立一个很实用的心智模型：

```text
一个独立开发任务
       ↓
    Workflow
       │
       ├── Codex 对话/Agent   ← 谁干
       │
       ├── Branch            ← 在哪条代码线上干
       │
       └── Worktree          ← 在哪个文件夹里干
```

### 举个非常具体的例子

你的主项目：

```text
H:\GH-Agent
└── main
```

现在同时开发三个东西：

```text
任务 1：开发 UI
Codex 对话 A
Branch   → feature/ui
Worktree → H:\GH-Agent-ui

任务 2：开发 Tree
Codex 对话 B
Branch   → feature/tree
Worktree → H:\GH-Agent-tree

任务 3：修 Bug
Codex 对话 C
Branch   → hotfix/crash
Worktree → H:\GH-Agent-hotfix
```

于是你**日常使用时**完全可以把它们口语化地理解成：

```text
Workflow A
= Codex A
+ feature/ui
+ H:\GH-Agent-ui

Workflow B
= Codex B
+ feature/tree
+ H:\GH-Agent-tree

Workflow C
= Codex C
+ hotfix/crash
+ H:\GH-Agent-hotfix
```

这比单纯说 `Workflow = Branch + Worktree` 更准确，因为还有一个非常重要的东西：**执行这个任务的 Codex 会话/Agent**。

所以最终建议你记成一句：

> **Branch 是“哪条代码线”，Worktree 是“哪个文件夹”，Codex 是“谁在干活”，三者组合起来构成一个独立开发工作流的核心。**

而 **Dashi** 再往上一层，就是负责看：

> **“现在有多少个这样的 Workflow？每一个做到哪了？”**

这几个概念你现在已经基本串起来了。

---

## 对话 7

### 用户

我需要记住创建work tree或者branch的代码吗？

### AI

**不需要死记。** 尤其你现在主要是用 **Codex 来开发**，更重要的是理解概念，而不是背 Git 命令。

你真正需要记住的是这三个东西：

> **Branch = 代码分支**  
> **Worktree = 独立文件夹**  
> **并行任务 = 给任务独立 Branch + Worktree**

实际操作时，你完全可以直接告诉 Codex：

> “从当前 main 创建一个 `feature/ui` 分支，并创建对应的独立 Worktree，用于开发 UI。”

让 Codex替你执行即可。

不过有 **4 个命令看熟就行，不必背**：

```bash
# 看有哪些分支
git branch

# 看有哪些 Worktree
git worktree list

# 创建 Branch + Worktree
git worktree add H:\GH-Agent-ui -b feature/ui

# 完成后删除 Worktree
git worktree remove H:\GH-Agent-ui
```

真正值得你掌握的是**什么时候该创建**。

比如：

```text
我要让 Codex 改一个小东西
        ↓
通常不用 Worktree

我要开发一个独立功能
        ↓
Branch

我要同时让两个 Codex 并行开发
        ↓
Branch + Worktree

任务完成
        ↓
Commit → Review → Merge main
        ↓
删除 Worktree / Branch
```

所以以后你甚至可以把操作交给 Codex，自己负责做**“项目经理式的判断”**：

> “这个任务需要独立开发 → 给它建 Branch + Worktree。”

这比记住 `git worktree add ... -b ...` 的参数顺序重要得多。

等你开始用 **Dashi + Codex** 后，还可以进一步把 **创建 Branch → 创建 Worktree → 启动任务 → 完成后合并 → 清理 Worktree** 做成固定流程，很多时候连命令都不需要自己输入。

---

## 对话 8

### 用户

这一步操作和我在codex里面创建新的codex任务是一样的是吗

[User attached 1 image; image contents were not included]

### AI

**很像，但不是完全一样。** 你可以把 Multica 图里的 `Agent A / B / C` 暂时理解成“你在 Codex 里开的多个独立任务”，但 Multica 多了一层**自动调度和管理**。

你现在手动使用 Codex，大概是：

```text
你
│
├─ 新建 Codex 任务 A
│    └─ 开发 UI
│
├─ 新建 Codex 任务 B
│    └─ 开发 Tree
│
└─ 新建 Codex 任务 C
     └─ 修复 Bug
```

你自己充当“调度员”：决定开什么任务、给什么提示词、在哪个目录工作、什么时候检查结果。

而图里的 Multica 是：

```text
你
 ↓
Dashi / Linear
 ↓
任务池
 ↓
Multica              ← 调度员
 ├─ Agent A
 │    ↓
 │  Worktree A
 │    ↓
 │  Codex
 │
 ├─ Agent B
 │    ↓
 │  Worktree B
 │    ↓
 │  Claude
 │
 └─ Agent C
      ↓
    Worktree C
      ↓
    Codex
```

所以最大的区别是：

> **你现在：你亲自创建和管理多个 Codex 任务。**  
> **Multica：用一个系统统一创建、分配、运行和跟踪多个 Agent 任务。**

而且图里的：

```text
Agent A → Worktree A → Codex
```

不要理解成三个东西都是“Agent”。

更准确是：

```text
Agent A
= 一个任务执行者
       │
       ├── 大脑/执行工具 → Codex
       │
       ├── 工作地点 → Worktree A
       │
       └── 工作目标 → Task A
```

### 对你现在来说

如果你只有 2～3 个并行开发任务，其实可以**暂时完全不用 Multica**：

```text
Dashi
 ↓
Task A ─→ Codex任务 A ─→ Worktree A ─→ Branch A
Task B ─→ Codex任务 B ─→ Worktree B ─→ Branch B
Task C ─→ Codex任务 C ─→ Worktree C ─→ Branch C
```

你自己就是 **“人肉 Multica”**。

等以后出现这种情况：

```text
10 个任务
+
5 个 Codex/Claude Agent
+
多个 Worktree
+
需要自动分配/执行/追踪
```

Multica 的价值才会明显。

**所以你现在可以先把 Multica 忽略掉。把 `Dashi → Codex任务 → Branch + Worktree → Commit → Merge` 这一条链跑熟，已经足够强了。**

---

## 对话 9

### 用户

把以上所有的有关工作流和工具的内容总结成一份逻辑清晰、重点突出、图文并茂的http交互网页

### AI

*此轮任务记录中没有可读取的文字回复；原对话在此处生成或交付了文件/图片。*

---

## 对话 10

### 用户

你怎么给我了一张图片？我要的是http的网页

### AI

这次已经给你做成了真正的 **HTML 交互网页**，不是图片。

里面包含了我们前面聊到的全部核心内容：Git、Branch、Worktree、Workflow、Codex 对话/Agent、Dashi、Multica、Linear、GitHub，以及“任务运行一半怎么分叉”“多 Codex 如何并行”“什么时候该创建 Worktree”等。

页面里还有可点击切换、折叠命令、场景选择器，以及“完整工作流”的动态高亮演示。

:chatgpt-content-reference{index="0"}

下载后直接双击，用 Chrome / Edge 打开即可，**不需要联网**。

---

## 对话 11

### 用户

github是一个什么样的存在？

### AI

你可以先把 [GitHub](https://github.com/?utm_source=chatgpt.com) 理解成：

> **程序员的“代码云盘 + 协作平台 + 项目主页”。**

但它和 Git 是两个不同的东西。

### Git 和 GitHub 的关系

假设你正在开发 GH 插件：

```text
你的电脑
H:\GH-Agent
│
├── Git
│   ├── main
│   ├── feature/ui
│   ├── feature/tree
│   └── 一堆 Commit
│
└── 项目代码
```

这些东西**没有 GitHub 也完全可以工作**。

Git 是安装在你电脑上的**版本控制系统**：

```text
代码
 ↓
Git
 ↓
Branch
 ↓
Commit
 ↓
Merge
```

而 GitHub 是互联网上的一台“项目服务器”：

```text
你的电脑                         互联网

H:\GH-Agent
    │
   Git
    │
 git push
    │
    └────────────────────→ GitHub
                            │
                            └─ GH-Agent Repository
                                ├─ main
                                ├─ feature/ui
                                ├─ Commit 历史
                                ├─ Issues
                                ├─ Pull Requests
                                └─ Releases
```

所以最核心的区别是：

> **Git 管你电脑里的代码版本。**  
> **GitHub 把 Git 项目放到云端，并提供协作、审核、发布等功能。**

### GitHub 不只是“代码云盘”

它最核心的对象叫 **Repository（仓库 / Repo）**。

比如以后你的：

```text
Data Tree Debug Agent
```

可以有一个 GitHub Repository：

```text
Data-Tree-Debug-Agent
│
├── src/
├── docs/
├── README.md
├── LICENSE
│
├── Branches
│
├── Commits
│
├── Issues
│
├── Pull Requests
└── Releases
```

这里面几个东西非常重要：

**Repository** = 整个项目的云端仓库。

**Issues** = Bug、需求、任务。例如：

```text
#23
空 Data Tree 导致插件崩溃
```

**Pull Request（PR）** = “我这个 Branch 做完了，请检查，确认后合并到 main。”

比如：

```text
feature/tree
      │
      │ Pull Request
      ↓
    Review
      ↓
    Merge
      ↓
     main
```

**Releases** = 正式发布软件版本，例如：

```text
v0.1 Alpha
v0.5 Beta
v1.0
v1.0.1 Hotfix
```

用户可以从这里获取正式发布版本。

### 放回你现在学的整套体系

现在整个关系就完整了：

```text
                 Dashi
                   │
              “有什么任务？”
                   ↓
              Codex 对话
                   │
                “谁来做？”
                   ↓
        Branch + Worktree
        │              │
   哪条代码线       哪个文件夹
        │              │
        └──────┬───────┘
               ↓
             Codex
          修改 / 测试代码
               ↓
              Git
               ↓
            Commit
               ↓
          git push
               ↓
            GitHub
               ↓
         Pull Request
               ↓
             Review
               ↓
             Merge
               ↓
              main
```

以后如果加入 Multica：

```text
Dashi / Linear
      ↓
    Multica
      ↓
多个 Codex / Claude Agent
      ↓
多个 Branch + Worktree
      ↓
     Git
      ↓
   GitHub
```

所以你可以用一个非常简单的比喻记住它们：

> **Dashi = 工单板**  
> **Codex = 程序员**  
> **Branch = 独立施工方案**  
> **Worktree = 独立施工现场**  
> **Git = 施工记录系统**  
> **GitHub = 云端项目总部**

而且 GitHub 还有一个非常重要的意义：**换电脑以后，你的代码不会只存在某一台电脑里。**只要已经 `push` 到 GitHub，另一台电脑就可以 `clone/pull` 下来继续开发。

这也正好和你之前问的“**一台电脑上的 Codex 任务，另一台电脑能不能继续**”联系起来：**Codex 对话/任务同步**和**代码同步**其实是两个不同的问题；GitHub 主要解决的是后者——代码、Branch、Commit 和协作历史的同步。

---

## 对话 12

### 用户

push到github上的文件会被别人破坏吗？可以不给别人看吗？/

### AI

可以，**GitHub 完全可以不给别人看**。这也是个人开发商业插件时很常见的做法。

在 [GitHub](https://github.com/?utm_source=chatgpt.com) 创建 Repository 时，主要有两种可见性：

- **Public（公开）**：别人可以看到和下载你的代码，但一般不能直接修改你的仓库；想贡献通常要通过 Fork / Pull Request，最终是否合并由你决定。
- **Private（私有）**：只有你以及**你明确授权的人**能访问代码。别人正常情况下看不到，也不能下载或修改。

所以如果你的 **Grasshopper 插件准备商业化、源码不准备公开**，建议直接：

> **Private Repository**

另外，“别人能看到”和“别人能破坏”是两回事。即使是 Public 仓库，普通陌生人通常也**没有直接写入权限**；真正需要重点保护的是你的 GitHub 账号、访问令牌，以及仓库协作者权限。开启 **2FA（双重验证）**也很值得。

你现在这套工作流可以理解成：

```text
你的电脑
H:\GH-Agent
     │
     ├─ main
     ├─ feature/ui
     └─ feature/tree
          │
          ↓
         Git
          │
       git push
          ↓
┌───────────────────────┐
│ GitHub Private Repo   │ 🔒
│                      │
│ 只有你/授权成员能访问 │
└───────────────────────┘
          │
          ↓
     云端代码备份
```

而且还有一个非常重要的安全点：**Private ≠ 可以把所有秘密都提交进去。**

像下面这些东西仍然不要 Commit：

```text
API Key
密码
OpenAI Key
数据库密码
私钥
.env 中的秘密
```

应该通过 `.gitignore`、环境变量或 GitHub Secrets 等方式管理。因为一旦秘密被 Commit，即使后来删除文件，它也可能仍存在于 Git 历史中。

对你目前的 GH 插件项目，我会建议：

> **本地 Git + GitHub Private Repository + 每个功能独立 Branch/Worktree + 定期 push。**

这样既能保留完整版本历史，又有远程备份，同时不会默认把你的插件源代码公开。

---

## 对话 13

### 用户

使用github要钱吗

### AI

对于你现在这种 **个人开发 GH 插件 + Codex + GitHub 私有仓库** 的场景，通常**不用花钱**。

GitHub 的免费账户已经支持：

- 创建 **Public 公共仓库**
- 创建 **Private 私有仓库**
- Git `push / pull / clone`
- Branch / Pull Request / Issue
- 和 Codex 配合进行代码开发
- 基本的 GitHub Actions 等功能有一定免费额度

所以你完全可以这样：

```text
你的电脑
H:\GH-Agent
     ↓
    Git
     ↓
Branch / Worktree
     ↓
   Codex 开发
     ↓
   Commit
     ↓
   Push
     ↓
GitHub Private Repository 🔒
     ↓
    免费
```

GitHub 主要在你需要**更高级的团队管理、安全功能、更高的 Actions/存储额度等**情况下才可能需要付费。

对你目前来说，**GitHub Free 基本足够，不需要买 Pro**。

如果你准备正式把现在的 `H:\GH-Agent` 项目放到 GitHub，我可以下一步带你从 **“创建 Private Repository → 连接本地 Git → 第一次 Push → 确认源码只有自己可见”**完整走一遍。

---

