# 统一 Checklist 改造 — 设计决策记录

> 状态：**进行中**。本文档是统一 checklist 改造的唯一设计事实源，供 skill / CLI / platform 三端改造时查阅，也用于跨对话/跨窗口延续上下文。
>
> 配套样例：[tool-checklist.v2.md](plugins/sanityops-inspect-lite/skills/sanityops-inspect-lite/references/tool-checklist.v2.md)（已产出，待审）。

## 1. 背景与目标

**现状问题**：skill 的 8 个 `references/*.md` 各自浓缩源规范，字段/口径不统一，且"源模型改动需同步 checklist、SKILL.md、README 多份文档"，漂移风险高（历史教训已在项目记忆中记录）。

**目标**：维护**一份统一 checklist 作为单一事实源**，skill / CLI / platform 三端归一化共用，从它派生各自所需。根治漂移。

**范围**：本次只改 skill 与 CLI；platform 复用同一份 checklist 但实现已定，不在此改造范围（字段设计仍以 platform 全功能为上限）。

## 2. 已定决策（共识清单）

### 2.1 单一事实源与文件结构

- checklist 用 **md 一份文件承载，无生成、无同步、无副本**。
- 物理结构分两层：

```
_meta.md          — 共享元文档：字段定义、判定口径、N/A 纪律、severity 规则、失效安全（小，加载一次）
prompt.md         ┐
skill.md          │
tool.md           ├ 域检查项（纯检查项，按阶段加载）
code-contract.md  │
cross.md          │
permission.md     │
explicit.md       │
relevance.md      ┘
id-index.md       — 精简 ID 清单（Stage G 自检 back-closure 用）
```

- 铁律：**md 就是真源，任何改动只改这一处**。

### 2.2 字段模型（全字段，三端共用）

原则：**字段以最全消费者（platform）为上限；"用不用"裁剪发生在消费端，不发生在字段端。**

| 字段 | 含义 | skill | CLI | platform |
|---|---|---|---|---|
| `id` / `name` / `definition` | 溯源锚点 + 判定定义 | ✓ | ✓ | ✓ |
| `check_engine` | `program` / `model` / `hybrid` | 忽略 | ✓ 路由 | ✓ 路由 |
| `applicability` | 结构事实（决定"考不考"） | ✓ | ✓ | ✓ |
| `severity` | 按 L 完整映射（决定"多严重"） | ✓ | ✓ | ✓ |
| `program_rule` | 程序判定规则 | 忽略 | ✓ | ✓ |
| `model_prompt` | 模型判定要点 | ✓ | ✓ | ✓ |
| `score_weight` | 打分权重/是否计入分母 | 忽略 | 忽略 | ✓ |
| `examples` | 正/反例 | ✓ | ✓ | ✓ |
| `cross_refs` | 同源去重/合并/转移关系 | ✓ | ✓ | ✓ |

### 2.3 ID 体系：不可变

- 检查项 ID 沿用 `QD-*` / `QC-*` / `EX-*` / `AS-*` / `FM-*` 及 `P0/P1/P2` 标签，**一字不改**。
- 合并/转移的项用状态标记（`merged` / `transferred`）记录，编号保留，不重编。

### 2.4 P 与 L：解耦，L 只喂 P

- 原始框架里 P 与 L 绑定（tier-specific severity），需解耦。
- 新设计：**L 只决定 P 的取值，不再决定"查不查"**。
- L 的判定**前置到 pre-inspect-extract**，一次判出。

### 2.5 L 的融合：引擎层分离，产品层合并

- **引擎层**：AC-L（复杂度）与 OR-L（操作风险）**两维度保留，不合成一个**（合成会导致 P 算错，如 AC-L1 + OR-L3 无法表达）。
  - AC-L：单轮/多轮/复杂/多 Agent，决定"查哪些项"（原职责，现被结构事实门接管大部分）+ Prompt 域 P。
  - OR-L：只读/可恢复写/不可逆，决定 Skill/Tool 域 P。
- **产品层**：合并成**一个统一标签**给用户看（Prompt→AC-L；Skill/Tool→OR-L；Permission→max）。
- 一句话：融合发生在"怎么给用户看"，不发生在"引擎怎么算"。

### 2.6 结构事实门（不丢项，核心）

- "考不考"由**可观测结构事实**决定（`has_tool` / `has_workflow` / `has_write_semantics` 等），**不由 L 决定**。
- 由此保证：**L 判错，最坏 P 偏一档，永不丢项**。
- 失效安全默认：`on_absent → N/A`（必写依据）、`on_unknown → judge`（默认查）。
- N/A 是"结构不适用"的出口，**不是缺失**。
- **关键前提（点1悖论，必须遵守）**：结构事实判断的是"工件声称要干嘛/出现什么迹象"，**不能**是"声明是否完整"。否则会自己把自己的题取消——例：`QD-P-2.4.2 工具未声明` 的适用性若判"有没有声明工具"，就永远抓不到"未声明"这个缺陷。

### 2.7 IR 两层（前置提取产物）

pre-inspect-extract 输出**两层** IR：

- **对象层**：实际提取到、有完整签名的对象（几个 P / S / T）。
- **引用层**：工件里被"提到 / 依赖"但未成形的引用。

跨对象检查（如"工具存在性"）看**引用层**，避免漏掉"该有却没有"的缺陷。

- **落盘**：IR 是 pre-inspect-extract 的产物，**落盘为 Markdown 文件**（默认 `{输入名}.ir.md`），供审计回溯（区分"提取错"还是"检查错"）与 CLI/platform 复用；同时保留一份在上下文里供链式交接。
- **机器可读核心**：IR 内嵌一个 `<!-- ir-core -->` JSON 块，只承载 `objects[].{type,name,L,completeness}` 与 `references[].{name,referenced_by,relationship}`，程序引擎只 parse 这一块（与 checklist 的 `program-check` 同一套路）。

### 2.8 check_engine（三值）

- `program`：确定性规则，程序引擎可执行（如字段缺失、语法解析）。
- `model`：语义判断，必须模型推理（如指令矛盾、描述质量）。
- `hybrid`：程序初筛 + 模型确认。**开发时优先收敛**——能纯 model 就纯 model，或初筛规则进程序块、确认要点进正文。

三端消费：
- skill：忽略 check_engine，全模型一次推理（program 项也由模型"模拟判定"）。
- CLI / platform：program 走程序引擎，model 走模型引擎，hybrid 两段式。

### 2.9 打分

- skill 与 CLI **不打分、不做 AI 应用级别对应**，只输出"检查项 → P → 证据"。
- 但 `score_weight` 字段**保留**（平台专用），不删。
- 全局权重：P0 : P1 : P2 = 5 : 3 : 1。

### 2.10 参考级展示（避免虚假恐吓）

- 原规范因 L 略去的检查项，**降级为参考级**展示（展示层概念，不进数据层）。
- **必须可追溯原始 P**：标注"此项在 L3 下为 P0"，避免"降级 = 隐性漏检"。

### 2.11 自检（防幻觉，分层）

- **程序自检**（仅 CLI）：只防"形式错误"（ID 是否存在、P 是否合法、证据非空、格式）。**防不了"张冠李戴"**（语义错配）。
- **模型三角核对**（skill + CLI 的模型引擎）：
  - 正向：`工件 + checklist → 缺陷ID`；反向：`缺陷ID + 缺陷内容 → checklist 定义是否匹配`。
  - **必须另起任务 / 独立上下文**才准——混在检测上下文里自检会退化成走过场。
  - 只对已检出的少量 ID 逐项核对。
- 结论：**程序管形式、模型管语义，互补非替代**。skill 无程序，只能软自检（三角核对 + ID back-closure）。

### 2.12 漏检

- 无系统解法（不能全量重扫）。
- 两条路径：① 实践积累反例；② **把更多项 program 化**（程序项天然全量枚举、不漏）。

### 2.13 加载方式（token 效率）

- **元文档加载一次**，各阶段复用。
- **按阶段、按工件类型增量加载**：只加载当前要用的域，不预加载无关域。
- **域内检查项保持完整，不做"适用性预筛删项"**（否则漏"该声明却没声明"的缺陷）；靠结构事实门在检查时产生 N/A（快速判、不深挖）。
- **CLI 剔除 program 项**（走程序引擎），skill 不剔除（全模型）。
- IR 引用层作为**输入**加载，不进 checklist。

### 2.14 逐域转换机械清单（铺全量时逐条照做）

1. ID/name/severity 从源 verbatim 提取；name 以「规则定义表」为正源。
2. check_engine：`Automated→program`、`Human→model`、`Semi-automated→hybrid`；源 Method 与定义矛盾时按"优先收敛"纠源并留日志。
3. applicability.fact 用可观测结构事实；"缺 X 字段/未声明"类检查用 `is_*`（类型事实，恒真）而非 `has_*`（防点1悖论）。
4. severity 记号：`P0/P1/P2`、`∅`(N/A)、`P1→P0(条件)`、`P1·P2`(双子缺陷)、`P0(style P2)`；源 `*`(by-feature) 并入 applicability.fact。
5. program 项用 `<!-- program-check -->` 包裹 JSON，只放 program + hybrid 初筛 summary；model 项绝不进入。
6. 三栏闭合：§2 索引 / §3.1 model / §3.2 hybrid 确认 / §4 program 块，ID 集合一致。
7. tombstone（merged/transferred）单独一节，编号保留。
8. 打分：`score_weight` 保留；change-check 与 tombstone 不计入分母。
9. examples 挂起，占位标注。
10. 域特有 L：Prompt 用 AC-L、Skill/Tool 用 OR-L、Permission 用 max；severity 表头显式标前缀。

## 3. 关键修正轨迹（教训，防重蹈覆辙）

1. **格式选型：YAML → md**。曾倾向 YAML 单源 + 程序生成 md，后纠正为"迁就少数程序项"的过度设计。正确：md 就是真源，程序项用 `<!-- program-check -->` 内嵌 JSON 块单独承载，程序引擎只 parse 这几块。
2. **打分字段：勿删**。曾把"skill/CLI 不打分"错误落成"删 score_weight 字段"，违背"字段以 platform 为上限"。正确：字段保留，行为裁剪在消费端。
3. **P 与 L**：原始框架里 P 是 tier-specific（与 L 绑定），不是独立固定值；解耦后靠"结构事实门 + L 只喂 P"保留场景敏感性。
4. **点1悖论**：适用性判断"工件声称要干嘛"，不能判断"声明是否完整"。
5. **`*`(by-feature) 语义归属**：源里 3.2 三档全带 `*`、3.4 却在 OR-L1/L2 标 `P0*(by feature)`、OR-L3 标 `P0`（无星），自相矛盾。判定：`*` 语义统一并入 `applicability.fact`（3.2→has_executable_param、3.4→has_authz_param），severity 只存"命中后取值"（3.4 取 P0/P0/P0）。依据：3.4 本质按特征触发（有 authz 参数才有缺陷），L3 无星属源简写。
6. **name 正源**：源「规则定义表」与「Appendix B 索引表」名称不一致（1.7/2.5/2.6/2.7/3.1）。判定：以**规则定义表**为 name 正源（信息更全，符合 name=溯源锚点）。
7. **Method→check_engine 纠源**：1.4(Automated→hybrid)、4.1/4.6/5.2(Semi-automated→model) 四处偏离源 Method 列，动因是源 Method 列与自身定义矛盾。接受纠源，样例内留重分类日志。

## 4. 待定 / 挂起项

- **反例（examples）**：挂起，后续再讨论，暂不处理。
- **L 统一尺子是否设 L4**（多 Agent 场景）：未定，按是否涉及多 Agent 决定 L1–L3 或 L1–L4。

## 5. 落地进度与下一步

**已完成**：
- [x] tool 域样例 `tool-checklist.v2.md`（已按自检修订：1.1 适用性点1悖论、1.2 子项补齐、`*` 归属、name 正源、Method 纠源、program-check 标记；待用户审阅）

**待办（按顺序）**：
1. 审 tool 样例，确认字段模型后铺全量其余 7 个域（prompt / skill / code-contract / cross / permission / explicit / relevance）。
2. 产出 `_meta.md`（共享元文档）+ `id-index.md`。
3. 改 pre-inspect-extract：加 L 前置定级 + IR 两层输出。
4. 改主 SKILL.md：流程引用新 checklist 方式、Stage G 自检升级为三角核对、移除打分、加参考级展示。
5. 同步 README / GET_STARTED。

**CLI 侧**（另一 repo `sanityops-cli`）按本决策记录 + 开发指导另行实现，不在本仓库。

## 6. 相关文件

- 样例：[tool-checklist.v2.md](plugins/sanityops-inspect-lite/skills/sanityops-inspect-lite/references/tool-checklist.v2.md)
- 现有旧 checklist（待替换）：`plugins/sanityops-inspect-lite/skills/sanityops-inspect-lite/references/*.md`
- pre 层：`plugins/sanityops-inspect-lite/skills/pre-inspect-extract/SKILL.md`
- 主 skill：`plugins/sanityops-inspect-lite/skills/sanityops-inspect-lite/SKILL.md`
