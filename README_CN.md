---

# 🧭 模块化个人生产力系统 (M-PPS)

### **一个极简的、可执行的 AI 原生个人操作系统**

### *构建于 Structure DNA → M-PPS → LLC → Claude Skills 之上*

---

## 🧩 概述

**M-PPS (模块化个人生产力系统)** 是一个轻量级但完整的、可执行的个人操作系统，完全通过**语言 → 结构 → 调度**构建而成。

本仓库提供：

- **四个核心结构协议**
- **四个 Claude Skills** 实现 M-PPS 运行时
- **示例账本** (初始 JSON 模板)
- **用户工作流示例**
- **一个最小化的、自洽的认知循环**

该系统将大语言模型转换为：

1. **语言解释器** — 意图识别
2. **结构编译器** — JSON Structure DNA 条目
3. **调度运行时** — 执行生命周期状态机
4. **反思引擎** — 生成反馈与学习

> 每个指令都成为结构。每个结构都成为行动。每个行动都成为学习。这是 AI 原生个人操作系统的基础。
> 

---

# 📐 系统架构

```
语言输入 (自然语言)
        ↓
Structure DNA (统一 JSON 架构)
        ↓
M-PPS 模块 (目标、日程、反思、画像)
        ↓
Claude Skills (执行与调度运行时)
        ↓
账本系统 (goal.json, schedule.json, reflection.json, profile.json)
        ↓
LLC (语言逻辑核心: 反馈 → 学习)

```

- **Structure DNA** 定义字段、时间戳规则和状态机。
- **M-PPS** 提供模块化的生活运营模型。
- **Claude Skills** 实现调度和分派。
- **账本** 充当操作系统的内存。
- **LLC** 将循环闭合为一个活的认知系统。

---

# 📁 仓库结构

```
.
├── protocols/
│   ├── structure-dna/v1.0/spec.md
│   ├── m-pps/v1.0/spec.md
│   ├── profile-generator/v1.1.md
│   └── language-logic-core/v1.0.md
│
├── skills/
│   ├── goal-manager/SKILL.md
│   ├── personal-schedule-manager/SKILL.md
│   ├── reflection-manager/SKILL.md
│   └── ledger-registry/SKILL.md
│
├── example-ledgers/
│   ├── goal.json
│   ├── schedule.json
│   ├── reflection.json
│   ├── profile.json
│   └── manifest.json
│
└── README.md

```

---

# 🔧 核心协议 (规范)

以下**四个基础协议**定义了 M-PPS 生态系统。

| 协议 | 描述 |
| --- | --- |
| **Structure DNA v1.0** | 通用架构：字段、时间戳、生命周期状态机、调度语义 |
| **M-PPS Specification v1.0** | 定义操作模块：目标、日程、反思、画像 |
| **Profile Generator v1.1** | 个人偏好：能量周期、时间块、调度约束 |
| **Language Logic Core (LLC) v1.0** | 元认知循环：意图 → 结构 → 行动 → 反思 → 学习 |

所有 Skills 和账本必须符合这些规范契约。

---

# 🧩 核心 Skills

本仓库包含**四个 Claude Skills**，它们共同构成一个可运行的执行循环。

| Skill | 模块 | 职责 |
| --- | --- | --- |
| **goal.manager** | 目标 | 解析意图 → 生成 G-条目 |
| **personal.schedule.manager** | 日程 | 将目标转换为可调度的 S-条目 |
| **reflection.manager** | 反思 | 捕获结果、学习、反馈 (R-条目) |
| **ledger.registry** | 注册表 | 维护清单、验证账本结构 |

### Skill 概要

---

### **1. goal.manager (目标管理器)**

- 理解自然语言意图
- 创建结构化的目标条目 (`G-xxx`)
- 分配标签和语义锚点
- 在准备好时将项目分派到调度

---

### **2. personal.schedule.manager (个人日程管理器)**

- 读取开放的目标
- 生成日程条目 (`S-xxx`)
- 应用时间语义
- 跟踪状态转换 (`open → scheduled → in_progress → done`)
- 完成时触发反思

---

### **3. reflection.manager (反思管理器)**

- 生成反馈条目 (`R-xxx`)
- 更新目标成功率
- 生成摘要字段
- 闭合认知循环

---

### **4. ledger.registry (账本注册器)**

- 读取 `manifest.json`
- 验证所有账本文件
- 协助其他 Skills 进行一致的文件写入

---

# 📘 示例账本 (模板)

所有示例账本存储在：

```
/example-ledgers/

```

它们包括：

- `goal.json`
- `schedule.json`
- `reflection.json`
- `profile.json`
- `manifest.json`

这些文件**仅作为初始模板**，不包含**任何个人数据**。

---

# ▶️ 如何使用

## **1. 将 Skills 导入 Claude**

从 `/skills/` 加载每个 `SKILL.md`。

## **2. 初始化你的账本**

所有账本文件在 Skills 初始化时自动创建。

你**不需要**手动复制模板 — 系统在首次使用时会基于 Structure DNA v1.0 生成干净的、符合架构的账本。

如果需要，你仍然可以参考 `/example-ledgers/` 作为参考模板。

## **3. 创建你的第一个目标**

示例：

```
添加一个新目标：在 8 周内学会基础日语。

```

Claude 将：

- 解析 → 结构化 → 创建 `G-001`
- 更新目标账本
- 如果指示则进行调度

---

## **4. 请调度器进行规划**

```
为接下来的 7 天安排所有开放的目标。

```

---

## **5. 标记任务为已完成**

```
将今天的学习课程标记为完成并生成反思。

```

系统将：

- 更新 S-条目
- 创建 R-条目
- 更新清单

---

## **6. 查询你的系统**

示例：

```
显示所有活跃的目标。
总结本周的所有进展。
显示与生产力相关的反思。

```

---

# 🔄 认知循环

```
意图 → 结构 → 调度 → 行动 → 反思 → 学习 → 再目标

```

这创建了一个**自我更新、持续学习的个人操作系统**。

---

# 📜 许可证

MIT License

© Entropy Control Theory (Susan STEM)

---

# 🌱 哲学

> 语言在结构化和调度后成为生命。
> 
> 
> 在 M-PPS 中，语言不仅仅是描述 —
> 
> 它成为**结构**、**行动**、**记忆**和**认知**。
> 

---

# 🙋 贡献

本仓库欢迎：

- 新的 Skills
- 协议扩展
- 账本格式
- 测试场景
- 可视化工具

每个贡献都扩展了 M-PPS 结构生态系统。

---

