---
title: 多 Agent 任务拆分与 Token 优化
tags:
  - 多Agent
  - Token
  - 任务拆分
  - 成本优化
level: 实践
---

# 多 Agent 任务拆分与 Token 优化

## 一、多 Agent 为什么可能更耗 Token

每个 Agent 都可能重复接收系统规则、角色说明、工具描述、背景、历史、输入材料和工具结果。

```text
总 Token
= 各 Agent 固定上下文
+ 各自输入输出
+ 工具结果
+ Agent 间交接
+ 失败重试与重复阅读
```

## 二、多 Agent 不一定更贵

合理拆分可能更省：

- 小模型处理简单重复任务；
- 强模型只处理架构和复杂判断；
- 每个 Agent 只读取相关文件；
- 确定性工具先筛选，再给模型小摘要；
- 专门 Agent 的提示更短、更聚焦。

## 三、按“可交付工作包”拆分

好的工作包应输入清楚、输出明确、边界独立、可单独验证、文件重叠小、容易合并。

```text
GH Agent MVP
├─ GH 文档读取器
├─ Data Tree 摘要器
├─ Empty Branch 检测
├─ 上游追踪
└─ 测试与样例
```

不要让不同 Agent 分别写一个函数名、一个 if 和一条文案；交接会比工作本身更贵。

## 四、一个编程 Agent 还是每个功能一个 Agent

同一对话适合紧密相关、共享大量上下文、规模小、串行、文件高度重叠的工作。

独立对话适合可并行、可独立测试、边界清楚、工作包足够大、需要不同角色或独立审查的任务。

推荐中间方案：

```text
一个主 Agent：架构、接口、验收、汇总
若干执行 Agent：独立交付模块
Review Agent（必要时）：跨模块兼容、测试、安全
```

## 五、最省 Token 的分工

### 最小充分上下文

只给目标、相关文件、接口、验收、风险和上游摘要；不传无关历史。

### 摘要交接

```markdown
## 目标
实现 Empty Branch 检测。

## 输入
TreeSnapshot 数据结构。

## 输出
实现代码、测试、已知限制。

## 不要做
不要修改 UI 和模型路由。

## 验收
三组样例准确返回空 Branch 路径。
```

### 强弱模型分层

| 工作 | 推荐 |
|---|---|
| 架构、复杂根因 | 强模型 |
| 分类、格式化、简单测试 | 快模型/小模型 |
| 规则可准确完成 | 不调用模型 |
| 长日志 | 程序筛选后再给模型 |

### 减少工具输出

不要把整个仓库、完整日志或所有 GH 数据塞进上下文。先搜索、截取错误附近、统计 Tree、压缩成摘要、只返回变化。

### 防止重复工作

主 Agent 维护谁在做什么、查过什么、已有决定、文件归属和被否定的假设。

## 六、Branch / Worktree 与 Agent 配对

高并行代码工作中的实用约定：

```text
一个独立代码任务
≈ 一个 Agent 对话 + 一条 Branch + 一个 Worktree
```

这是经验规则，不是绝对真理。只读研究任务通常不需要 Worktree。

## 七、单 Agent 与多 Agent 的成本

单 Agent 固定上下文和交接少，但历史可能越来越长。多 Agent 固定上下文与交接更多，但每个上下文更短、更专业。

应比较：

```text
单位正确成果成本
= Token + 时间 + 返工 + 人工协调 + 合并风险
```

## 八、GH Agent 项目建议

- MVP：一个主对话和主 Worktree，先做数据结构、规则检测、测试。
- 模块化：Runtime、Plugin、Analyzer、Tests 可独立后再分 Worktree。
- 规模化：长期 Agent 角色、模型路由、Token 预算、集中日志，再引入 Multica 或 Linear。

## 九、Core 与 UI：什么时候并行更好

Core 与 UI 可以使用同一个 GH Coding Agent 配置。并行真正依赖的是两个独立 Run，而不是两个不同 Agent 名称。

### 适合并行

- Core 与 UI 的职责边界清楚；
- 接口 Contract 已经稳定；
- 数据结构和错误状态已约定；
- 两边修改文件重叠少；
- 可以分别测试和合并。

```text
先确定 Contract
├─ TreeAnalysisResult
├─ EmptyBranches
├─ NullItems
├─ Warnings
└─ Summary

然后并行
├─ Run A → Core → Branch / Worktree A
└─ Run B → UI   → Branch / Worktree B
```

### 不适合立即并行

若 UI 还不知道 Core 返回什么结构、接口叫什么、错误如何表示，两个 Run 会分别作出不同假设，最终产生冲突、重写和更多 Token 消耗。

这时应：

1. 先完成一个很小的接口设计任务；
2. 固定 Core ↔ UI Contract；
3. 再启动两个并行 Run。

### 什么时候拆成不同 Agent 配置

同一个 Agent 配置已经足够的情况：

- 都属于普通 Coding；
- 使用相同模型、Skills 和 Runtime；
- 规则差异只与当前任务有关。

值得拆成 Core Agent 与 UI Agent 的情况：

- 长期职责不同；
- Skills 不同；
- 修改权限不同；
- 模型或 Thinking 不同；
- UI Agent 必须长期禁止修改 Core。

> [!tip]
> **任务独立性比 Agent 数量更重要；接口稳定性决定并行是否真正提速。**

## 十、从一个任务拆成两个 Run 的成本判断

并行能缩短时间，但可能增加：

- 两份上下文；
- 两次模型调用链；
- 接口误解；
- Merge 冲突；
- 主 Agent 汇总与 Review。

因此先用一个小设计步骤换取稳定 Contract，往往比盲目并行更省 Token。


## 十一、自测

- [ ] 我知道多 Agent 的固定上下文与交接成本。
- [ ] 我会按可独立验收的工作包拆分。
- [ ] 我知道只读研究不一定需要 Worktree。
- [ ] 我会用摘要交接。
- [ ] 我会先用规则和工具缩小数据。
