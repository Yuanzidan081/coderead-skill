---
name: coderead-propose
description: 根据用户的代码阅读问题创建一个命名任务，在 `coderead` 目录下建立对应的任务子文件夹，后续所有分析报告均输出到该文件夹内。通过 `/coderead-propose` 命令手动触发。
disable-model-invocation: true
---

# CodeRead 任务提案器 (Task Proposer)

## 触发方式

- **手动触发**：通过 `/coderead-propose` 斜杠命令。
- **自动激活**：否——仅在用户明确发起一个代码阅读任务时手动执行。

## 目标

将用户的代码阅读需求封装为一个「命名任务」，在 `coderead/` 目录下创建对应的任务子文件夹。后续该任务相关的所有分析报告（birdview、dataflow、intent、verify）均输出到此文件夹内，实现按任务维度的输出隔离。

## 执行步骤

### 1. 解析用户意图

从用户的提示词中提取关键信息：
- **分析目标**：要分析的模块/文件/函数名称
- **分析维度**：用户关注的角度（架构？数据流？设计意图？）
- **项目上下文**：当前项目的名称或技术栈

### 2. 生成任务文件夹名

将用户的提示词浓缩为一个**简短、可读的英文 slug**（小写字母 + 数字 + 连字符），作为任务文件夹名。

**命名规则**：
- 仅使用小写英文字母、数字和连字符 `-`
- 长度控制在 4-40 个字符
- 体现核心分析目标

**示例**：

| 用户提示词 | 生成的任务文件夹名 |
|-----------|-----------------|
| "帮我分析认证模块的数据流" | `auth-dataflow` |
| "我想理解为什么订单模块用了事件溯源" | `order-event-sourcing` |
| "考考我对消息队列的理解" | `mq-verify` |
| "梳理整个项目的宏观架构" | `project-architecture` |

如果无法从提示词中提取有意义的关键词，回退为 `task-YYYYMMDD-HHMMSS`。

### 3. 创建任务文件夹

1. 确保 `coderead/` 根目录存在（若不存在则先创建）。
2. 在 `coderead/` 下创建任务子文件夹。
3. 在任务文件夹内创建 `README.md` 作为任务索引。

**任务 README 模板**：

```markdown
# 📋 [任务标题]

> 创建时间：YYYY-MM-DD HH:MM
> 任务文件夹：`coderead/[folder-name]/`

## 任务描述

[用户原始提示词或提炼后的描述]

## 分析目标

- **模块/文件**：[目标]
- **关注维度**：[架构 / 数据流 / 设计意图 / 掌握验证]

## 产出文件

| 文件 | 来源 | 状态 |
|------|------|------|
| `birdview.md` | `/coderead-birdview` | ⬜ 待生成 |
| `dataflow.md` | `/coderead-dataflow` | ⬜ 待生成 |
| `intent.md` | `/coderead-intent` | ⬜ 待生成 |
| `verify.md` | `/coderead-verify` | ⬜ 待生成 |
| `notes.md` | 碎片 Q&A | ⬜ 待生成 |
```

### 4. 通知并引导下一步

创建完成后，向用户展示：

```markdown
---

## ✅ 任务已创建：`[任务标题]`

**输出目录**：`coderead/[folder-name]/`

### 🔜 下一步

你可以使用以下命令开始分析，所有报告将自动保存到此目录：

| 命令 | 用途 |
|------|------|
| `/coderead-birdview` | 宏观架构全景分析 |
| `/coderead-dataflow` | 数据流与状态变化追踪 |
| `/coderead-intent` | 设计意图与权衡深挖 |
| `/coderead-verify` | 掌握度反向验证 |

> 💡 推荐分析顺序：`/coderead-birdview` → `/coderead-dataflow` → `/coderead-intent` → `/coderead-verify`

---
```

**产出**：`coderead/<task-folder>/` 任务目录 + README。

> 💡 **每次调用 `/coderead-propose` 都创建全新任务文件夹并自动设为当前活动任务。** 如需回到已有任务继续分析，使用 `/coderead-switch` 切换上下文。

---

## 执行准则

1. **slug 必须可读**：任务文件夹名不能被用户看懂的话就失去了意义。优先使用有意义的英文缩写，避免纯数字或随机字符串。
2. **命名去重**：如果目标文件夹已存在，在末尾追加 `-2`、`-3` 等序号。
3. **记录活动任务**：创建任务后，明确告知用户当前活动任务名。后续分析 skill 输出时将自动写入该任务文件夹。
4. **不自动启动分析**：创建任务后仅引导用户下一步操作，不自动触发分析 skill。
