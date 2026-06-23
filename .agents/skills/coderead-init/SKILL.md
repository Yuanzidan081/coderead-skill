---
name: coderead-init
description: 在任意项目目录下初始化 CodeRead 工作空间，创建 `coderead` 输出文件夹及基础结构。通过 `/coderead-init` 命令手动触发。
disable-model-invocation: true
---

# CodeRead 初始化器 (Workspace Initializer)

## 触发方式

- **手动触发**：通过 `/coderead-init` 斜杠命令。
- **自动激活**：否——仅在用户明确要求初始化 CodeRead 工作空间时手动执行，避免在非目标项目中误创建。

## 目标

在当前项目根目录下创建 `coderead/` 根文件夹，为后续任务和分析报告提供存放位置。

> **与 `/coderead-propose` 的关系**：`coderead-init` 创建全局的 `coderead/` 根目录；`coderead-propose` 在其中按任务创建子文件夹。实际分析报告存储在任务子文件夹中，而非直接散落在 `coderead/` 根目录。

## 执行步骤

### 1. 确认目标目录

向用户确认初始化位置：

```markdown
将在当前项目根目录下创建 `coderead/` 文件夹，作为所有代码阅读分析报告的根目录。

目录结构预览：
[项目根目录]/
└── coderead/                    ← CodeRead 输出根目录
    └── <task-name-1>/           ← 由 /coderead-propose 创建
    └── <task-name-2>/           ← 每个任务一个独立子文件夹
        ├── birdview.md
        ├── dataflow.md
        ├── intent.md
        └── verify.md

确认创建？(y/n)
```

### 2. 创建目录

收到用户确认后，使用 `create_directory` 工具创建 `coderead/` 文件夹。

### 3. 创建 README

在 `coderead/` 目录下生成 `README.md`：

```markdown
# CodeRead Analysis Reports

此目录由 CodeRead Agent 自动生成，存放所有代码分析报告。

## 使用方式

1. 使用 `/coderead-propose` 创建一个分析任务
2. 使用分析命令生成报告：
   - `/coderead-birdview` — 宏观架构全景分析
   - `/coderead-dataflow` — 数据流与状态变化追踪
   - `/coderead-intent` — 设计意图与权衡分析
   - `/coderead-verify` — 掌握度反向验证
3. 报告将自动存入对应的任务子文件夹

## 目录结构

每个任务（由 `/coderead-propose` 创建）拥有独立子文件夹，其中包含该任务相关的全部分析报告。这种设计允许对同一项目的不同模块或不同分析角度进行独立追踪。
```

### 4. 输出确认

创建完成后，向用户展示结果并引导下一步：

```markdown
✅ `coderead/` 工作空间已就绪。

下一步：使用 `/coderead-propose <你的问题>` 开启第一个分析任务。
```

**产出**：`coderead/` 根目录 + README。

---

## 执行准则

1. **必须确认**：在创建任何文件前，必须先向用户展示目标结构并获得确认。
2. **轻量设计**：仅创建 `coderead/` 根目录和 README。具体的任务子文件夹由 `/coderead-propose` 按需创建。
3. **幂等设计**：如果 `coderead/` 已存在，只补充缺失的 README，不覆盖已有内容。
