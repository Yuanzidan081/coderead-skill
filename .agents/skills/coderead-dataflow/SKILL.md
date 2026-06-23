---
name: coderead-dataflow
description: 梳理模块的动态数据流（Data Flow）与状态变化。适用于"数据流分析"、"追踪状态变化"、分析数据载体从触发源经处理节点到终点的完整生命周期等场景。通过 `/coderead-dataflow` 命令手动触发，或当用户提及数据流追踪、状态跃迁分析时自动激活。
---

# 数据流追踪 (Data Flow Trace)

## 触发方式

- **手动触发**：通过 `/coderead-dataflow` 斜杠命令。
- **自动激活**：当用户提及「数据流分析」「追踪状态变化」「数据流转」「状态跃迁」等关键词时自动加载。

## 目标

梳理用户指定模块的动态数据流（Data Flow）与状态变化——追踪数据从「诞生」到「消亡」的完整生命周期，捕获路径上的每一次变形、拷贝与状态跃迁。

## 标准作业流程 (SOP)

按以下 5 个阶段顺序执行，前一阶段的输出作为后一阶段的输入：

---

### 阶段 1: 数据载体定义 (Payload Definition)

**目标**：定义流转的核心数据载体及其初始状态下的核心字段与内存/结构表达。

**执行步骤**：

1. **定位核心数据结构**：从用户指定的模块中，找到在流程中流转的核心数据载体（struct/class/interface/type alias/protobuf message）。常见的载体类型：

   | 载体类型 | 典型命名模式 | 举例 |
   |---------|------------|------|
   | **Payload/Message** | `*Payload`, `*Message`, `*Request`, `*Response` | `LoginRequest`, `OrderPayload` |
   | **Context** | `*Context`, `*Session`, `*Scope` | `RequestContext`, `RenderContext` |
   | **Command** | `*Command`, `*Cmd`, `*Action` | `PlaceOrderCommand`, `RenderCmd` |
   | **Event** | `*Event`, `*Notification`, `On*` | `PaymentCompletedEvent`, `FrameTickEvent` |
   | **Entity/Model** | 业务实体名 | `User`, `Order`, `Transaction` |

2. **提取字段定义**：列出核心数据结构的字段名、类型和含义。标注哪些字段是**不可变的**（只读）、哪些是**可变的**（在流转中会被修改）。

3. **分析内存表达**（如适用）：
   - 该数据结构是否做了内存对齐优化（`#[repr(C)]` / `__attribute__((packed))`）？
   - 是栈分配还是堆分配？
   - 是否使用了 Small Buffer Optimization (SBO) 或变长结构？

**产出**：数据载体规格表 + 字段可变性标注 + 内存布局简述。

---

### 阶段 2: 数据流转管道追踪 (Pipeline Tracing)

**目标**：以**函数/方法级别**的精度，追踪数据载体从「触发源」出发，历经每个处理函数的完整调用链，并用 Mermaid 序列图或流程图直观呈现。

> ⚠️ **与 `/coderead-birdview` 阶段 4 的区别**：birdview 追踪的是「模块间」交互（粒度：组件），本阶段追踪的是「函数间」调用链（粒度：精确到文件和行号）。

**执行步骤**：

1. **定位数据第一站 (Source)**：
   - 数据从何处诞生，它的**第一站是哪个函数**？常见源头：网络回调、用户输入、定时器、消息队列消费、文件读取。
   - 记录入口函数/方法的完整签名及其所在文件 + 行号。

2. **追踪精确调用链 (Pipeline)**：
   - 从 Source 函数开始，沿调用链向下追踪**每一个被调用的函数**，直到数据到达最终 Sink。
   - 每个节点必须标注：**文件名 + 行号 + 函数签名**。
   - 节点类型包括但不限于：
     - **Filters**：数据校验、权限检查、格式转换
     - **Transformers**：数据变形、字段映射、编解码
     - **Handlers**：业务逻辑执行
     - **Dispatchers**：事件分发、路由转发
     - **Sinks**：数据最终写入（DB、文件、网络响应）

3. **记录数据形态变化**：
   - 在每个处理节点，数据是否发生了类型转换（如 DTO → Entity → VO）？
   - 是否发生了拷贝/序列化/反序列化？

4. **绘制流转图**：
   - 使用 Mermaid `sequenceDiagram`（强调参与者间的消息传递）或 `graph LR`（强调数据处理管道）。
   - 图中每个 participant 代表一个**函数**（而非模块），标注其所属文件。

**产出**：
- 处理节点清单（含节点类型、文件 + 行号、函数签名）
- Mermaid 数据流转全景图

---

### 阶段 3: 状态机跃迁矩阵 (State Transition Matrix)

**目标**：捕获流转过程中伴随的系统状态机变化。

**执行步骤**：

1. **识别状态机**：
   - 在数据载体或处理流程中，是否存在枚举/标志位代表系统状态？（如 `enum Status { Pending, Processing, Completed, Failed }`）
   - 是否有显式状态机类（如 `StateMachine<State>`、`fsm` 库）？

2. **枚举状态空间**：列出所有可能的状态值及其含义。

3. **追踪每一次跃迁**：
   对每一次状态变化，记录：

   | 维度 | 含义 |
   |------|------|
   | **处理节点** | 状态变化发生在哪个处理阶段 |
   | **触发事件** | 什么条件触发了状态变化 |
   | **原始状态 (Src)** | 变化前的状态 |
   | **目标状态 (Dst)** | 变化后的状态 |
   | **副作用 (Side Effects)** | 状态变化的同时还触发了什么（日志、通知、资源申请/释放） |

4. **绘制状态图**（可选）：
   - 使用 Mermaid `stateDiagram-v2` 绘制状态机全貌。

**产出**：状态跃迁矩阵（Markdown 表格）+ 可选 Mermaid 状态图。

---

### 阶段 4: 变异热点分析 (Mutation Hotspots)

**目标**：标出数据被修改（Mutation）、拷贝（Copy/Marshalling）或销毁（Free/Dispose）的关键控制点，并评估性能或并发安全隐患。

**执行步骤**：

1. **标记所有 Mutation 点**：
   - 在数据流路径上，找出所有对数据载体进行**写操作**的位置。
   - 标注每次写入修改了哪些字段。

2. **识别隐式拷贝**：
   - 是否存在因值传递（`pass-by-value`）导致的非预期拷贝？
   - 是否存在 Marshalling 边界（跨进程、跨语言 FFI、序列化/反序列化）导致的完整数据复制？
   - 是否存在不必要的 `clone()` 或 `copy()` 调用？

3. **追踪生命周期终点**：
   - 数据在哪里被释放/销毁？
   - 是确定性释放（RAII、`defer`、`Dispose`）还是依赖 GC？
   - 是否存在循环引用导致的内存泄漏风险？

4. **并发安全隐患评估**：
   - 数据载体是否在多线程/协程间共享？
   - 是否有竞态条件（Race Condition）的可能？
   - 锁定粒度是否合适？

5. **量化评估**：
   按热点严重程度分级：

   | 级别 | 标志 | 含义 |
   |------|------|------|
   | 🔴 Hot | 高频写入 + 共享访问 | 高并发瓶颈，需优先关注 |
   | 🟡 Warm | 中频写入 或 隐式拷贝 | 随规模放大可能成为瓶颈 |
   | 🟢 Cold | 低频写入，确定性释放 | 风险可控 |

**产出**：Mutation 热点标注表 + 生命周期图 + 并发隐患清单（含严重等级）。

---

### 阶段 5: 输出数据流报告 (Output)

**目标**：将前 4 个阶段的分析结果汇总为一份结构清晰的 Markdown 文档。

**文档必须包含以下章节**：

#### 文档模板

```markdown
# 🔄 [业务场景名] Dynamic Data Flow & State Trace Report

> 分析日期：YYYY-MM-DD
> 分析范围：[模块名 / 文件列表]
> 追踪场景：[场景简述]

---

## 1. 📦 数据载体定义 (Data Payload Specification)

### 核心数据结构

\`\`\`[语言]
// [数据结构定义]
struct / class / interface PayloadName {
    field1: Type1,  // [含义] [可变/不可变]
    field2: Type2,  // [含义] [可变/不可变]
    ...
}
\`\`\`

### 字段说明

| 字段名 | 类型 | 可变性 | 含义 |
|--------|------|--------|------|
| `field1` | `Type1` | 🔒 不可变 | [说明] |
| `field2` | `Type2` | ✏️ 可变 | [说明] |

### 内存布局（如适用）

- **分配方式**：[栈 / 堆 / 自定义 Allocator]
- **对齐策略**：[自然对齐 / 紧凑打包 / 缓存行对齐]
- **预估大小**：[sizeof / 近似值]

---

## 2. 🗺️ 数据流转全景拓扑 (Data Pipeline Topology)

### 处理节点清单

| 序号 | 节点名称 | 类型 | 所在文件 | 职责 |
|------|---------|------|---------|------|
| 1 | `entry_point()` | Source | `src/main.rs` | [触发源] |
| 2 | `validate()` | Filter | `src/validator.rs` | [校验] |
| 3 | `transform()` | Transformer | `src/mapper.rs` | [变形] |
| 4 | `handle()` | Handler | `src/handler.rs` | [业务逻辑] |
| 5 | `persist()` | Sink | `src/repo.rs` | [持久化] |

### 流转全景图

\`\`\`mermaid
sequenceDiagram
    participant Source
    participant Filter
    participant Transformer
    participant Handler
    participant Sink

    Source->>Filter: raw_data
    Filter->>Transformer: validated_data
    Transformer->>Handler: transformed_data
    Handler->>Sink: final_entity
    Sink-->>Handler: ack
\`\`\`

---

## 3. 🚦 状态机跃迁矩阵 (State Transition Matrix)

### 状态空间

| 状态 | 含义 |
|------|------|
| `Init` | [初始状态] |
| `Processing` | [处理中] |
| `Completed` | [成功完成] |
| `Failed` | [失败] |

### 跃迁矩阵

| 处理节点 | 触发事件 | 原始状态 | 目标状态 | 副作用 (Side Effects) |
|---------|---------|---------|---------|---------------------|
| `entry_point()` | 请求到达 | (无) | `Init` | 创建 Payload |
| `validate()` | 校验通过 | `Init` | `Validated` | 记录审计日志 |
| `handler()` | 业务处理完成 | `Validated` | `Completed` | 发送通知事件 |
| `error_handler()` | 任何异常 | `*` | `Failed` | 回滚事务、告警 |

### 状态图（可选）

\`\`\`mermaid
stateDiagram-v2
    [*] --> Init
    Init --> Validated: 校验通过
    Validated --> Processing: 开始处理
    Processing --> Completed: 处理成功
    Processing --> Failed: 异常
    Validated --> Failed: 校验失败
    Failed --> [*]
    Completed --> [*]
\`\`\`

---

## 4. ⚡ 变异热点与突变分析 (Mutation & Lifecycle Hotspots)

### 变异点标注

| 序号 | 位置 | 操作类型 | 修改字段 | 严重等级 | 说明 |
|------|------|---------|---------|---------|------|
| 1 | `handler.rs:42` | ✏️ Write | `status`, `updated_at` | 🟡 Warm | 双写，需考虑原子性 |
| 2 | `mapper.rs:18` | 📋 Copy | (全量拷贝) | 🟡 Warm | DTO → Entity 转换导致二次分配 |

### 隐式拷贝分析

| 位置 | 拷贝原因 | 触发频率 | 影响评估 |
|------|---------|---------|---------|
| `mapper.rs:18` | DTO → Entity 转换 | 每次请求 | 随 payload 增大逐渐凸显 |

### 生命周期终点

| 数据载体 | 释放位置 | 释放方式 | 风险 |
|---------|---------|---------|------|
| `Payload` | `handler.rs:100` | RAII / `defer` | ✅ 安全 |
| `Entity` | `repo.rs:55` | GC 托管 | ⚠️ 延迟回收 |

### 并发隐患

| 位置 | 风险描述 | 严重等级 | 建议 |
|------|---------|---------|------|
| `handler.rs:42` | 多协程并发写入 `status` 字段 | 🔴 Hot | 加锁或改用原子操作 |

---

## 📊 总结与建议

### 数据流健康度评估

[对数据流设计做整体评价：是否清晰、有无反模式]

### 优化建议

1. [建议一]
2. [建议二]

### 关注点

- [后续需要重点关注的性能/安全领域]
```

**文档应保存到**：`coderead/<task-name>/dataflow.md`（若已通过 `/coderead-propose` 创建任务；否则回退为 `coderead/dataflow.md`）。

> 📝 **增量追加规则**：若目标文件已存在，**禁止覆盖**。在文件末尾追加新章节 `## 📝 补充分析 YYYY-MM-DD HH:MM`，将本次分析结果写入该章节。仅当用户明确说「重写」「覆盖」「重新生成」时才覆盖原文件。

---

## 执行准则

1. **以用户指定模块为锚点**：若用户未指定追踪的具体模块/函数，主动询问：「请提供要追踪数据流的代码范围（文件路径 + 函数名、或模块名）」。
2. **先载体后管道**：必须先确定数据结构的完整定义（阶段 1），再开始追踪流转路径（阶段 2），否则容易遗漏关键字段的变化。
3. **每个节点都要有证据**：处理节点清单中的每一项都必须能对应到具体的代码位置（文件 + 行号）。
4. **关注隐式成本**：阶段 4 中不仅标注显式的写操作，更要警惕隐式拷贝（值传递、序列化边界、Rc/Arc 的 clone）——这些往往是性能问题的真凶。
5. **Mermaid 图表必须可渲染**：`sequenceDiagram` 用于参与方交互，`graph LR` 用于管道处理流程，`stateDiagram-v2` 用于状态机。不做额外样式定制。
6. **善用并行工具调用**：阶段 1 和阶段 2 中，多个独立的文件读取可以并行进行。
