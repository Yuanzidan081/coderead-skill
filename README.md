# CodeRead

> **Code can be read, but understanding must be self-proven.**
> 代码可以阅读，但理解必须自证。

CodeRead 是一套专为 **Zed Editor** 设计的智能代码阅读 Agent 技能包。它不帮你写代码——它帮你 **看懂代码**。从宏观架构到微观意图，从数据流向到设计动机，四层递进式分析 + 反向考核，让你真正消化源码而非被动接受 AI 的结论。

---

## 🧠 核心理念

大多数 AI 编程工具在帮你「写」，CodeRead 在帮你「读」。我们遵循**费曼学习法**的精神：

```
鸟瞰全局 → 追踪数据流 → 灵魂拷问设计 → 反向输出验证
```

每一步都站在**架构师**而非代码解释者的角度，追问 **Why** 而非 What。

> 灵感来源：[电话微波炉 — 如何用 AI 高效阅读源码](https://zhuanlan.zhihu.com/p/2048044532516890300)

---

## 🛠️ 技能一览

### 分析技能（自动激活）

| 命令 | 技能 | 一句话 |
|------|------|--------|
| `/coderead-birdview` | 宏观骨架 | 自顶向下构建项目架构全景、依赖拓扑与领域划分 |
| `/coderead-dataflow` | 数据流追踪 | 函数级精度追踪数据从诞生到消亡的完整生命周期 |
| `/coderead-intent` | 设计意图挖掘 | 灵魂拷问：Why this, not that? 删掉这行会怎样？ |
| `/coderead-verify` | 掌握度验证 | 盲写重构或费曼教学，无情检验你的真实理解 |

### 工作空间技能（手动触发）

| 命令 | 技能 | 一句话 |
|------|------|--------|
| `/coderead-init` | 初始化 | 创建 `coderead/` 根目录 |
| `/coderead-propose` | 创建任务 | 为分析目标建立独立任务文件夹 |
| `/coderead-switch` | 切换任务 | 在多个分析任务间切换上下文 |
| `/coderead-archive` | 内容归档 | 将指定主题内容提取存档，清理原文件 |

---

## 🚀 快速开始

### 1. 安装

将本仓库克隆到你的 Zed 项目 `.agents/skills/` 目录：

```bash
git clone https://github.com/your-org/coderead-skills.git .agents/skills/coderead
```

或者复制到全局 skills 目录（所有项目可用）：

```bash
# Windows
git clone https://github.com/your-org/coderead-skills.git %USERPROFILE%\.agents\skills\coderead

# macOS / Linux
git clone https://github.com/your-org/coderead-skills.git ~/.agents/skills/coderead
```

### 2. 初始化工作空间

在 Zed 中打开任意项目，输入：

```
/coderead-init
```

### 3. 创建第一个分析任务

```
/coderead-propose 帮我分析认证模块的架构
```

### 4. 开始分析

```
/coderead-birdview
```

---

## 📖 标准阅读流程

```
/coderead-init          ← 初始化工作空间（仅首次）
        │
/coderead-propose ...   ← 创建分析任务
        │
/coderead-birdview      ← ① 鸟瞰全局：架构全景
        │
/coderead-dataflow      ← ② 动态追踪：数据流向
        │
/coderead-intent        ← ③ 灵魂拷问：设计动机
        │
/coderead-verify        ← ④ 反向验证：自测掌握度
        │
    中途可追问 → 自动归入 notes.md
    重复分析 → 增量追加，不覆盖原报告
```

---

## 📁 输出结构

```
你的项目/
└── coderead/                          ← CodeRead 输出根目录
    └── auth-dataflow/                 ← 任务文件夹（由 propose 创建）
        ├── README.md                  ← 任务索引 + 进度追踪
        ├── birdview.md             ← birdview 报告
        ├── dataflow.md              ← dataflow 报告
        ├── intent.md                ← intent 报告
        ├── verify.md                ← verify 报告
        ├── notes.md                   ← 碎片 Q&A 笔记
        └── archive/                   ← 内容归档
            ├── birdview/
            ├── dataflow/
            ├── intent/
            ├── verify/
            └── note/
```

---

## ✨ 亮点特性

### 🔄 增量追加，不覆盖

重复触发同一 Skill 时，报告**默认追加**而非覆盖。每次分析以带时间戳的新章节形式追加到文件末尾。只有你说「重写」才覆盖。

### 📝 碎片对话自动归档

追问某行代码的意图、某个参数的含义——这些不触发完整报告，而是追加到 `notes.md`：

```markdown
### 💬 2026-06-17 14:30 — 为什么 verify() 用指数退避？

**用户追问**：AuthService.verify() 的重试为什么不用固定间隔？

**分析**：指数退避在分布式系统中能避免「惊群效应」...
```

### 🎯 Why 先于 What

`coderead-intent` 强制 Agent 先解释**设计动机**再描述行为：

> ❌ What 模式：「这段代码先校验输入，然后调用 `process()`，最后写库。」
>
> ✅ Why 模式：「两阶段校验而非一次性校验，是因为第二阶段依赖 DB 查询结果，分开可尽早拒绝无效请求，避免无效查询拖垮连接池。」

### 🪓 反向验证，拒绝幻觉

两个高压模式检验你是否真懂：

- **模式 A — 盲写盲评**：挖空核心函数，你手写实现，AI 逐行对比评分
- **模式 B — 费曼教学**：你当作者向「叛逆新人」解释设计，AI 无情追问直到你扛不住

### 📊 Mermaid 可视化

所有报告内置 Mermaid 图表——依赖拓扑图、时序图、状态机图、权衡矩阵，可直接在 GitHub / Zed 中渲染。

---

## 🗺️ 技能路由规则

CodeRead Agent 会根据你的提问**自动选择**最合适的 Skill：

| 你问... | 自动激活 |
|---------|---------|
| 「这个项目的整体架构是怎样的？」 | `/coderead-birdview` |
| 「数据从 Controller 到 DB 中间经过了几次拷贝？」 | `/coderead-dataflow` |
| 「为什么作者用自旋锁而不是 Mutex？」 | `/coderead-intent` |
| 「考考我，看我是否真理解了这段代码」 | `/coderead-verify` |

意图模糊时会主动引导你选择分析角度。

---

## 📋 依赖

- **Zed Editor**（最新版）
- 已配置的 AI Agent 功能

---

## 🤝 贡献

欢迎提交 Issue 和 PR。如果你想扩展新的分析 Skill（如 FFI 边界分析、测试覆盖率分析），可以参考现有 Skill 的 SOP 结构。

---

## 📄 许可

MIT License

---

<p align="center">
  <b>Code can be read, but understanding must be self-proven.</b><br>
  代码可以阅读，但理解必须自证。
</p>
