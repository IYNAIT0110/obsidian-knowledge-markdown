---
title: Git、GitHub、Branch、Worktree 与 Workflow
tags:
  - Git
  - GitHub
  - Branch
  - Worktree
  - Workflow
level: 入门
---

# Git、GitHub、Branch、Worktree 与 Workflow

## 一、先用一句话区分

| 概念 | 最短解释 |
|---|---|
| Git | 本地代码版本历史系统 |
| GitHub | 放 Git 仓库并协作的网络平台 |
| Branch | 仓库中的逻辑开发路线 |
| Worktree | 某条 Branch 对应的实际工作目录 |
| Workflow | 完成任务的一套步骤 |
| Codex 任务/对话 | Agent 的一段独立工作上下文 |

## 二、Git 是什么

Git 记录项目随时间的变化，帮助回退、追踪改动、尝试不同方案、合并工作和形成可审查记录。

```text
工作目录 → 暂存区 → Commit → Branch → Repository
```

新手先掌握四个动作：查看变化、暂存需要的变化、提交清晰版本、在合适时推送或合并。

## 三、GitHub 是什么

GitHub 是托管 Git 仓库的在线平台，提供 Pull Request、Issues、Actions、权限、审计和 Releases。

- 没有写权限的人不能直接修改仓库；
- 有写权限的人可能 Push；
- 分支保护可要求 PR、测试和审核；
- Private Repository 只对获授权者可见；
- 免费与收费范围会变化，应查看官方定价。

## 四、Branch：逻辑路线

```text
main: A --- B --- C
                    \
feature:             D --- E
```

Branch 不是复制整个文件夹，而是 Git 对提交历史路线的引用。

优点：轻量、历史清楚、容易合并与回滚。  
局限：一个普通工作目录一次只检出一条 Branch；未提交修改会影响切换；多个 Agent 同目录容易干扰。

## 五、Worktree：物理目录

```text
同一个 Git 仓库历史
├─ D:\project-main       → main
├─ D:\project-feature-a  → feature/a
└─ D:\project-bugfix     → bugfix/tree-null
```

优点：多 Branch 同时打开、适合多 Agent、减少切换、隔离文件和构建状态。  
缺点：多占磁盘、目录需要管理、同一 Branch 通常不能同时检出到两个 Worktree。

## 六、准确关系

> **Branch 是逻辑路线，Worktree 是这条路线在硬盘上的工作房间。**

不要写成 `Workflow = Branch + Worktree`。

```text
Workflow
可能使用
├─ Codex 对话
├─ Branch
├─ Worktree
├─ Task
└─ Review / Test / Merge
```

Branch + Worktree 是 Workflow 的基础设施，不是 Workflow 本身。

## 七、Codex 新任务是否等于创建 Worktree

不一定。新任务可能直接在当前目录工作、创建独立 Worktree、使用无项目目录，或只讨论不修改代码。要看创建任务时选择的工作环境。

## 八、一个功能是否都要独立组合

建议拆开的情况：可独立验收、范围较大、需要并行、风险高、需要单独 Review、多个 Agent 同时编辑。

不必拆开的情况：几分钟小修、同一功能的紧密步骤、需要连续理解大量上下文、拆分成本更高。

> [!tip]
> 拆分单位是“可独立完成、验证、合并的工作包”，不是每个函数或按钮。

## 九、操作时需要背命令吗

不需要死记，命令可查，工作逻辑必须懂。

```bash
git status
git switch -c feature/example
git branch
git worktree add ../project-example -b feature/example
git worktree list
git worktree remove ../project-example
```

## 十、完整分层

```text
Linear / Dashi Taskboard：要做什么、谁负责、进度
Codex / Multica / Agent：谁来分析与实现
Branch / Worktree：怎样隔离并行工作
Git：改了什么、如何回退和合并
GitHub：怎样共享、审核、测试和发布
```

## 十一、在一个 Task 中怎样使用 Branch / Worktree

假设任务是“实现 Tree Analyzer Core”，完整关系是：

```text
Task：实现 Tree Analyzer Core
→ 选择 GH Coding Agent
→ 启动 Run
→ Branch：feature/tree-core
→ Worktree：独立 Core 目录
→ 编码、Build、Test、Commit、Review、Merge
```

整个链条才属于 Workflow。Branch 和 Worktree只是本次代码工作的版本线与施工现场。

### 新 Branch 是否必须交给新 Agent

**不需要。Branch 不属于 Agent，而属于一次具体代码工作。**

同一个 GH Coding Agent 配置可以连续或并行处理：

```text
GH Coding Agent
├─ Task A → Run A → feature/tree-core → Worktree A
├─ Task B → Run B → feature/tree-ui   → Worktree B
└─ Task C → Run C → feature/export    → Worktree C
```

创建新 Branch 不代表必须新建 Agent。只有任务性质由 Coding 变成 Debug、Review、Architecture 等，并且规则和技能确实不同，才考虑换 Agent 配置。

### Task 做到一半怎样分叉 Core 与 UI

当 Core 开发中发现 UI 可以独立并行时，推荐从共同基点拆成两个工作包：

```text
共同基点
├─ Task：Core → Run A → feature/core → Worktree A
└─ Task：UI   → Run B → feature/ui   → Worktree B
```

两个 Task 可以仍使用同一个 GH Coding Agent 配置。不要让两个并行执行实例在同一个目录里同时修改同一批文件。

### 是否需要额外 Worktree

按顺序判断：

1. 这是不是独立可提交的代码工作？
2. 若是，创建独立 Branch。
3. 是否与其他代码工作并行？
4. 若并行，创建独立 Worktree。
5. 再让相应 Run 在该目录执行。

串行小任务可以只有 Branch，不必机械创建 Worktree。


## 十二、自测

- [ ] 我能区分 Git 和 GitHub。
- [ ] 我知道 Branch 是逻辑路线，Worktree 是实际目录。
- [ ] 我不会把 Workflow 简化成 Branch + Worktree。
- [ ] 我知道新建 Codex 对话不一定新建 Worktree。
- [ ] 我能判断任务是否值得独立 Branch / Worktree。
