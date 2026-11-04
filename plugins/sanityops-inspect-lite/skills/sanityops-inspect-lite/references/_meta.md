---
title: 统一 Checklist 共享元文档（_meta）
---

# 统一 Checklist 共享元文档

> 本文件是 8 个域 checklist 的**共享定义**：字段模型、check_engine 映射、applicability 结构事实门、severity 表达、打分规则、域文件组织约定。**加载一次、各域复用**。域文件只承载"检查项本身"，不重复本文件。
>
> 状态：v2 统一引擎版（已定稿）。与 [checklist-unification-design.md](../../../../../checklist-unification-design.md)（设计决策记录）配套。

## 1. 字段模型（全字段，三端共用）

> 原则：字段以最全消费者（platform）为上限；"用不用"裁剪发生在消费端，不发生在字段端。

| 字段 | 含义 | skill | CLI | platform |
|---|---|---|---|---|
| `id` / `name` / `definition` | 溯源锚点 + 判定定义 | ✓ | ✓ | ✓ |
| `check_engine` | `program` / `model` / `hybrid` | 忽略（全模型） | ✓ 路由 | ✓ 路由 |
| `applicability` | 结构事实（决定"考不考"，不写 L） | ✓ | ✓ | ✓ |
| `severity` | 按 L 完整映射（决定"多严重"） | ✓ | ✓ | ✓ |
| `program_rule` | 程序判定规则（program/hybrid 的初筛） | 忽略 | ✓ | ✓ |
| `model_prompt` | 模型判定要点（model/hybrid 的确认） | ✓ | ✓ | ✓ |
| `score_weight` | 打分权重/是否计入分母 | 忽略 | 忽略 | ✓ |
| `examples` | 正/反例（挂起，待补） | ✓ | ✓ | ✓ |
| `cross_refs` | 同源去重/合并/转移关系 | ✓ | ✓ | ✓ |

## 2. check_engine 三值与映射

- `program`：确定性规则，程序引擎可执行（如字段缺失、语法解析）。
- `model`：语义判断，必须模型推理（如指令矛盾、描述质量）。
- `hybrid`：程序初筛 + 模型确认。

**源 `Method` 列 → check_engine 基线映射**：`Automated→program`、`Human→model`、`Semi-automated→hybrid`。

**纠源原则（优先收敛）**：能纯 model 就纯 model，初筛规则进程序块、确认要点进正文。当源 Method 列与自身定义矛盾时，按此原则纠正，并在域文件内留重分类日志。

## 3. applicability（结构事实门）

- "考不考"由**可观测结构事实**决定（`has_tool` / `has_workflow` / `has_write_semantics` / `is_tool_schema` 等），**不由 L 决定**。
- **点1悖论（必守）**：结构事实判断"工件声称要干嘛/出现什么迹象"，**不能**判断"声明是否完整"。否则自己取消自己的题——"工具未声明"若判"有没有声明工具"就永远抓不到。
  - 因此"缺 X 字段 / 未声明 X"类检查的 applicability 用 `is_*`（类型事实，恒真）而非 `has_X`（会自我取消）。例：`QD-T-1.1 Base Field Missing` 用 `is_tool_schema`，不用 `has_schema`。
- **失效安全默认**：
  - `on_absent` → N/A（结构事实不存在，必写依据）
  - `on_unknown` → judge（无法判定，默认查）
- **N/A 纪律**：N/A 是"结构不适用"的出口，**不是缺失**；必须写明依据。

## 4. severity 表达约定

- `P0/P1/P2` = 该 L 下的固定取值
- `∅` = 该 L 下原规范不分配 P → **参考级**（仍检查，但不计该 L 的 severity）；N/A 由 `applicability.fact` 单独决定，与 L 无关。
- `P1→P0(条件)` = 默认 P1，命中括号内条件升级 P0
- `P1·P2` = 同项内两个子缺陷分列不同级
- `P0(style P2)` = 主缺陷 P0、风格性子项 P2
- 源规范 `*`（by-feature，按特征触发）的语义已并入 `applicability.fact`，severity 只存"命中后取值"。

**域特有 L 命名**：Prompt 域用 AC-L、Skill/Tool 域用 OR-L、Permission 用 max；severity 表头显式标前缀。

## 5. 打分（platform 专用）

- 全局权重：`P0 : P1 : P2 = 5 : 3 : 1`，按项最高严重度扣一次。
- `score_weight` 字段保留；skill/CLI 不激活打分（行为裁剪在消费端）。
- change-check（如 QD-T-5.x 变更管理）与 tombstone 不计入静态分母。
- **分母** = 该 L 下 `applicability.fact` 命中的活跃项数；各域文件 §6 的 L1/L2/L3 固定计数是原 tier-scoped 规格的历史参考值，platform 实现以 applicability 命中为准。

## 6. 域文件组织约定

每个域文件（如 `prompt-checklist.v2.md`）结构：

1. 状态头（来源 + 本文件只承载检查项，共享定义见 `_meta.md`）
2. §1 域内重分类日志（如有）
3. §2 检查项索引（ID / name / check_engine / applicability.fact / severity / 计入打分）
4. §3 模型推理项（§3.1 纯 model / §3.2 hybrid 确认）
5. §4 程序判定项（`<!-- program-check -->` JSON 块，只放 program + hybrid 初筛）
6. §5 tombstone（merged / transferred）
7. §6 打分字段说明（域特有分母）

**三栏闭合约束**：§2 索引 / §3.1 model / §3.2 hybrid / §4 program 块 的 ID 集合一致。
