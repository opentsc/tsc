# OpenTSC v1.0 ·《壳》

**The Shell — 设计理念与技术实现规格**

> 这是魂的运行容器的施工规格。读这份文档的可能是 Claude Code、Codex、opencode 或任何 coding agent。
> 目标：**把这份文档 + 《魂》丢给一个 AI，它就能自己写出 OpenTSC。**
> 配套文档：《魂》（TSC 3.5 白皮书）——本文档实现的那部法。本文档中"法则 N""创世层""判断法典"等术语均以《魂》为准。

---

## 0. 给执行 AI 的元指令

1. 本文档分**公理（不可违背）/ 契约（必须满足）/ 指引（可自主优化）** 三级。冲突时公理 > 契约 > 指引。公理即《魂》的十二法则。
2. 任何破坏十二法则或下面内核不变量的实现，视为错误实现，必须重做。
3. 不确定时**停下来问，不要猜**。这是处理真实人际情报的私人系统，错误默认行为代价高。
4. 全程单玩家、本地、离线优先。不引入把魂发往外部服务器的依赖，除非显式授权（持有者责任）。
5. **先建消化系统，再喂食。** 判断引擎（中段）没建好之前，不要先堆采集口——否则只是堆更多原料。施工顺序见第 11 节，不要跳。
6. **不要过度架构。** 默认单 agent + 最小职业集。只在出现已证实的能力缺口时才长出新 agent（法则十）。多 agent 项目的主要死法是为不存在的复杂度提前造基础设施。

---

## 1. 壳的三条设计理念

### 理念一 · 一切皆接口

壳里没有"功能"，只有"接口 + 填接口的实现"。内核定义一组**接口槽**（采集、评分、知识、建议、校准、报告、操作、简报、角色、招募…），每个槽是一份契约（读什么、写什么、对应 VSM 哪个功能）。具体的 agent 和技能是填进槽里的可替换实现。

好处：换模型、换实现、加领域能力，都只是换填充物，内核不动。这是法则二（魂壳分离）在软件层的兑现——内核稳定，边缘全可替换。

### 理念二 · 模块即 Agent Skill

每个模块用 **Agent Skill 开放标准**（SKILL.md：YAML frontmatter 机器可读 + Markdown 人可读 + 可选 scripts/references）实现，享受**渐进披露**：启动只加载模块的 name+description（~80 tokens/个），命中才加载完整 SKILL.md（~2000 tokens），需要才执行 scripts。一个壳可以"知道"几十个模块，代价小于激活一个。这让壳能挂海量职业技能而不撑爆上下文。

> 选 Agent Skill 而非私有插件格式的理由：它是 2025-12 起的开放标准（微软/OpenAI/GitHub 已采纳），SKILL.md 既能被 AI 执行又能被人读懂（法则要求"人和 AI 都读懂"），且存在 **skill-creator** 这一现成的"技能生成技能"——正是自创生引擎的地基。

### 理念三 · 自创生而非预装

壳不预装一个 TSC 需要的全部能力。它装一个**自创生引擎**：读魂（目标/环境/缺口），用 skill-creator 模式生成这个 TSC 此刻需要的 agent 和技能（SKILL.md 文件），经校验门入册，带日落。不同的魂长出不同的壳。这是法则十一（成员自创生）的执行机构。

---

## 2. 仓库结构

```text
opentsc/
├── soul/                         # 魂（法则二：可整体导出/复活）
│   ├── _genesis.md               # 创世层，write-once（法则一）
│   ├── _rule_codex.md            # 规则法典：创世/宪法/操作三层（法则七）
│   ├── _judgment_codex.md        # 判断法典：评分/信任维度、技能定义、可调动度公式（法则七）
│   ├── _schema.md                # 字段本体活字典（K5）
│   ├── events/                   # 全局事件流，append-only（法则四）
│   │   └── YYYY-MM/evt_*.md      # 每条事件一节点，带 links/caused_by
│   └── calibration/              # 校准账本（法则八）
│       └── YYYY-MM/
├── shell/                        # 壳（可重写、可替换）
│   ├── kernel/                   # 内核 K1–K8，稳定
│   ├── modules/                  # 模块 = Agent Skills（接口实现）
│   │   ├── _registry.md          # 模块注册表（progressive disclosure 索引）
│   │   ├── collector.*/SKILL.md
│   │   ├── scoring.*/SKILL.md
│   │   ├── advisor.*/SKILL.md
│   │   └── ...                   # 见第 4 节模块族
│   ├── professions/              # 职业定义（契约）
│   │   └── *.md                  # 见第 6 节
│   └── genesis_engine/           # 自创生引擎（见第 5 节）
├── world/                        # 世界模型（实体，法则三/四/九）
│   ├── players/p_*/
│   ├── npcs/
│   │   ├── agents/a_*/           # Agent NPC（可控）
│   │   └── humans/p_*/           # 人类 NPC（不可控，观察推断）
│   ├── orgs/o_*/                 # 组织实体（递归 TSC，法则十二）
│   ├── operations/op_*/          # 任务即实体
│   └── roles/                    # 角色指派（职能，非实体）
├── raw/                          # 原始素材，带 manifest+hash（法则六）
├── inbox/                        # 自动提取草案，待确认（法则六）
└── reports/YYYY-MM/              # 时间性产物
```

**铁律**：`soul/` 与 `shell/` 物理分离。导出 `soul/` 整个目录 = 导出魂，可注入任意合规的新壳复活（法则二）。

---

## 3. 内核（Kernel）· K1–K8

内核稳定，守护法则，不调用任何模块功能；模块只通过 K6 总线访问数据。相对 v0.4，新增 K7（判断引擎）、K8（自创生注册），它们正是过去断掉的中段。

| 模块 | 职责 | 守护法则 |
|---|---|---|
| **K1 身份服务** | 分配/解析稳定 ID，别名管理，实体消解（去重）与认证 | 法则三 |
| **K2 数据契约** | 校验实体/关系边/事件条目结构与目录形态 | 法则四、十 |
| **K3 事件流引擎** | append-only 事件流，软删除，当前状态视图推导，**事件图谱（节点+边+因果链）** | 法则四 |
| **K4 反馈契约** | 强制建议/预测带到期日+推理链，分离 useful/comfortable | 法则八 |
| **K5 字段本体** | 维护 `_schema.md`，新字段登记，防漂移 | 法则九、十 |
| **K6 模块总线** | 模块注册、生命周期、读写授权，保证不绕过内核 | 法则二、十一 |
| **K7 判断引擎** ★新 | 每来新事件，按 judgment_codex 自动更新相关实体的属性/技能/状态，留推理痕迹 | 法则九 |
| **K8 自创生注册** ★新 | 接收 genesis_engine 生成的 agent/技能/职业草案，过校验门，注册或拒绝，挂日落 | 法则十一 |

### 内核 API 形态（实现语言自选，契约必须保留）

```typescript
interface KernelAPI {
  identity: {
    resolve(nameOrId: string): EntityId
    create(type: EntityType, seed: unknown): EntityId      // person|agent|org|operation|tsc
    suggestMerges(seed: unknown): MergeCandidate[]
    confirmAlias(id: EntityId, alias: AliasRef): void
  }
  events: {                                                 // K3 + 事件图谱
    append(event: IntelEvent): EventId                      // K2/K3 校验：必须带 source + Admiralty
    link(eventId: EventId, targets: EntityId[]): void       // 一条事件连多个实体
    cause(from: EventId, to: EventId): void                 // 因果边
    timeline(filter: GraphQuery): IntelEvent[]
    deriveView(id: EntityId): EntityView                    // 当前状态 = 推导，非存储
    neighborhood(id: EntityId): EventGraph                  // 拉一个实体的全部事件网络
  }
  judgment: {                                               // K7 ★
    onEvent(eventId: EventId): AttributePatch[]             // 按 judgment_codex 推导属性变更
    attribute(id: EntityId, dim: string): AttributeClaim    // {value,confidence,provenance,reviewed,decay}
    compare(idA: EntityId, idB: EntityId, dim: string): Comparison   // "谁更强"直接查，不重读历史
    explain(id: EntityId, dim: string): ReasoningChain      // 这个属性凭哪些事件
  }
  feedback: {                                               // K4
    stage(prediction: Prediction): PredictionId             // 必须带 due_date + reasoning_chain
    due(): Prediction[]
    fulfill(id: PredictionId, outcome: Outcome): void
    stats(filter?: FeedbackFilter): FeedbackStats           // 命中率按情境/知识来源分解
  }
  codex: {
    rule(): RuleCodex                                       // 只读，改需治理流程
    judgment(): JudgmentCodex
    proposeAmendment(target: "rule"|"judgment", change: Amendment): ProposalId  // 自迭代
  }
  schema: { register(field: FieldDef): void; known(): FieldDef[] }   // K5
  genesis: {                                                // K8 ★
    detectGaps(): CapabilityGap[]                           // VSM 必备职业 + 领域职业缺口
    spawn(spec: AgentSpec | SkillSpec | ProfessionSpec): DraftId   // 草案，待批准
    validate(draftId: DraftId): ValidationResult            // 校验门
    register(draftId: DraftId): ModuleRef                   // 入册 + 挂日落
    sunset(ref: ModuleRef): void
  }
  bus: { register(module: ModuleManifest): void; invoke(ref: ModuleRef, ctx: Ctx): Promise<Result> }  // K6
}
```

### 内核不变量（违反即错误实现）

1. 每个实体一个不可变 `id`；路径锚点用 ID，不用名字。
2. 事件流 append-only；修正追加、删除软删。
3. 事件必须带 Admiralty 评级与 source。
4. 建议/预测必须带到期日与推理链才能 stage。
5. 新字段先登记 `_schema.md` 再用。
6. 模块走总线，不得直接改实体身份或事件流内部。
7. 自动提取先进 `inbox/`，正式入册需确认。
8. **属性只能由 K7 从事件推导写入，禁止任何模块直接写属性值**（法则九）。
9. **judgment_codex 不得硬编码进任何模块；所有评价维度运行时从魂读取**（法则七、九）。
10. **genesis_engine 生成的任何 agent/技能必须过 K8 校验门 + 玩家批准才入册**；不得自动激活（法则十一 + 持有者责任）。
11. 任务完成/放弃仅玩家可宣告；关闭任务强制触发复盘 + 校准回填。
12. `soul/` 可独立导出且自洽：仅凭 `soul/` 能重算出全部当前属性（法则二复活前提）。

---

## 4. 模块接口契约（接口即模块）

每个模块是一个 Agent Skill，其 SKILL.md frontmatter 必须声明这份契约：

```yaml
---
name: advisor.match
description: 任务-人选匹配建议引擎。当玩家问"谁适合做X""安排今天任务"时使用。
vsm: S3                       # 对应 VSM 子系统（S1–S5），决定它在编排里的层
profession: 指挥官            # 它服务/由哪个职业调用（可空）
reads:  [entities, events, knowledge, judgment]    # 经 K6 授权
writes: [feedback]                                 # 经 K6 授权
binding: { kind: tool, ref: advisor_match }
model_tier: high              # high|mid|low — 编排按此分配模型，控成本
trigger: { kind: invoke }     # invoke|event|hook(session.created/idle)
sunset: { unused_after_months: 6 }
genesis: { template: advisor, domain: generic }    # 由哪个自创生模板生成（领域可空）
---
```

### 模块族（接口槽，初始集）

- **collector.\***（采集，S4 感知）：manual / import / contacts / transcript / osint / **platform.\***（X、Slack、Mattermost、邮件、日历）/ **hardware.\***（录音、IoT、可穿戴）。统一产出 `inbox/` 草案事件。**硬件子契约：边缘脱敏，只送结构化事件，原始数据不出设备**（持有者责任）。
- **scoring.\***（评分，S3）：维度全部从 judgment_codex 读，不内置。
- **knowledge.\***（知识三层，S3）：facts / methods / principles，存带触发条件的可执行颗粒，建索引+显式链接检索；颗粒必须被 advisor 自动命中，不靠模型重读原料；未被引用超 N 月进日落。
- **advisor.\***（建议引擎，S3，系统的判断输出）：match / relation / capability / intake / leverage。每条建议经 K4 落库，附完整推理链；玩家打分针对推理链不针对结论。
- **calibration.\***（校准，S4，防回音壁）：review（挂 session.idle）/ stats / feedback（仅用 useful/accurate，绝不用 comfortable）。
- **operations.\***（任务生命周期，S1）：plan / track / review（关闭强制复盘）。
- **briefing.\***（启动简报，S3，挂 session.created）：startup，限量分级，必含逆耳项，只读。
- **role.\***（角色指派，职能非实体）。
- **recruiter.\***（招募，S4）：检测缺口 → 搜寻候选 / 产出招募意图 / 提示玩家找人。

---

## 5. 自创生引擎（Genesis Engine）— 壳如何长出 agent 与技能

这是法则十一的执行机构，也是"根据 TSC 组织不同自创生不同 agent 和技能"的具体实现。

### 触发条件（只在已证实缺口时，法则十）

- 创世清单缺口分析（第 10 节）发现某必备/领域职业无合适担任者；
- 蒸馏师/简报官在运行中发现重复出现、现有模块处理不了的情境（在 vault 留 `needs_capability` 标记）；
- calibration 发现某领域命中率持续低，需要更专门的判断模块。

### 生成流程（skill-creator 模式）

```
缺口（CapabilityGap: 需要什么 VSM 功能 / 什么领域 / 服务哪个职业）
 → 选模板：genesis_engine/templates/{collector|scoring|advisor|...}.skill-template.md
 → 读魂：genesis（底线约束）+ judgment_codex（评价维度）+ 环境（领域词表）
 → LLM 按模板生成候选 SKILL.md（frontmatter 契约 + 指令 + 可选 script）
 → K8 校验门：
     · frontmatter 契约完整？reads/writes 在授权范围？vsm 合法？
     · 是否违反任一法则？（尤其法则八/九/十：不得内置维度、不得直接写属性、不得自动激活）
     · 与现有模块重复？（重复则建议复用而非新建）
 → 产出草案，状态 draft，写 inbox
 → 玩家批准（不变量 10）→ K8 register → 写 _registry.md → 挂 sunset
 → 拒绝 → 记录原因，反馈给下次生成
```

### 同一个壳，不同的魂，长出不同的东西

| TSC（魂的环境） | 自创生长出的领域模块/职业（示例） |
|---|---|
| 量化交易（example-quant） | `collector.platform.exchange`、`advisor.risk`（风控官）、`scoring.signal_quality` |
| 女性社群（HerName.AI） | `collector.platform.community`、`advisor.member_health`（情绪健康顾问）、`scoring.engagement` |
| 内容/增长（@example_user） | `collector.platform.x`、`advisor.topic`（选题预言家）、`scoring.virality`（爆款蒸馏师） |
| 团队协作（Stanley Team） | `collector.platform.mattermost`、`advisor.workload`、`scoring.reliability_team`（团队级聚合） |

基础职业集恒定（第 6 节），领域模块全由自创生引擎按魂生成。**这就是壳的"开放世界"：地图（职业/技能/agent）不是预写死的，是按你这个 TSC 的目标和环境自己长出来的。**

---

## 6. 职业定义格式（含预设必备职业）

职业是一份契约文档 `shell/professions/{name}.md`：

```yaml
---
profession: sentinel
display: 哨兵
vsm: S2/S4
holder_type: agent              # agent | human | player | any
model_tier: low
required_skills:                # 技能树节点（来自 judgment_codex 的技能定义）
  - anomaly_detection: ">=Lv.2"
  - deadline_tracking: ">=Lv.1"
attribute_thresholds:           # 基础属性门槛
  reliability: ">=0.6"
reads:  [entities, events, feedback]
writes: []                      # 哨兵只读，发现就在 vault 留标记（烙印协调）
triggers: [session.created, session.idle, schedule.hourly]
evolves_to: [scout]             # 可进化的子职业（LitRPG class evolution）
---
# 哨兵
盯异常、盯 deadline、盯模式变化。发现就在事件流里留 `needs_*` 标记，不直接处理（烙印协调）。必须能产出逆耳项。
```

**预设必备职业（VSM-完整最小集）**：`founder`(创世者/玩家) · `commander`(指挥官) · `sentinel`(哨兵) · `ingestor`(摄入官) · `distiller`(蒸馏师) · `oracle`(预言家) · `herald`(简报官) · `coordinator`(协调员) · `operator`(执行者) · `steward`(归因官/司库) · `recruiter`(招募官)。各职业的 VSM 归属、默认担任者、是否可由 agent 担任，见《魂》第四部分表。

---

## 7. 角色属性 schema（世界模型）

实体 frontmatter（人类 NPC 示例；agent NPC 同 schema，属性来源标 `configured`）：

```yaml
---
id: p_1c2e
type: human_npc                 # player | human_npc | agent_npc | org | operation
names: { real: TODO(user), aliases: [{value: "Grace", platform: x, status: confirmed}] }
professions: [operator]         # 可多职业（fluid class）
base:                           # 基础属性层（innate，慢变）
  execution_ceiling: { value: 0.7, confidence: 0.5, provenance: [evt_a1], reviewed: 2026-05, decay: 0.02 }
  resilience:        { value: 0.5, confidence: 0.4, provenance: [evt_b3], reviewed: 2026-05, decay: 0.02 }
skills:                         # 技能树层（learned，事件驱动）
  - id: client_psych
    level: 3
    prereq_met: true
    leveled_by: [evt_c1, evt_c9]    # 哪些事件触发了升级
states:                         # 状态层（buff/debuff，带倒计时）
  - tag: "已读不回·可靠性存疑"
    kind: debuff
    expires: 2026-06-08
    on_repeat: solidify         # 再犯固化为永久
    source: evt_d2
trust: { reviewed: 2026-06-01 } # 信任维度，从 judgment_codex 读
source_mode: inferred           # inferred(人类) | configured(agent)
tags: []
---
```

关系属性挂在边上（不是节点字段）：

```text
[现] 合作 →→ p_0f3a : "Dave" · 报价重构主力 · 2026-至今 · mobilizability:{value:0.8,confidence:0.6} · cost:中
```

属性的完整 `AttributeClaim` 形态（每个值都是带证据、会衰减、可预测的概率声明）见《魂》第三部分。**禁止手填属性值；只能追加事件，由 K7 推导**（不变量 8）。

---

## 8. 事件图谱 schema

每条事件 `soul/events/YYYY-MM/evt_*.md`：

```yaml
---
id: evt_0a1f
date: 2026-06-01
admiralty: B2                   # 来源可靠性B × 信息可信度2（法则五）
content: "Carol 在与 Eve 通话中主动提及团队成立仅3个月"
source: "会议逐字稿 #5"
raw: raw_2026-06-01_call5
status: active                  # active | corrected | soft_deleted
links: [p_carol, p_eve, op_001]      # 一条事件连多个实体（不再只挂一个人）
caused_by: []                   # 上游因果
causes: [evt_0a23]              # 下游因果：→「Eve 信任受损」
judgment_triggered: [p_carol.trust, p_carol.skills.client_psych]   # K7 据此更新了哪些属性
---
```

**实现提示**：先用 markdown frontmatter + 反向索引（K3 维护 `id → {events, edges}` 映射）实现事件图谱，不要一上来上图数据库。`neighborhood(id)` 查询走反向索引。数据量大了再换存储，契约不变。

---

## 9. 双法典格式

- `soul/_genesis.md`：创世层。存在宣言 + 不可变底线 + 玩家及其不可逆权限。**write-once**，K2 拒绝任何修改写入。
- `soul/_rule_codex.md`：规则法典三层。创世层（继承 genesis）/ 宪法层（改需高门槛，记门槛与程序）/ 操作层（改需简单多数，每条带 12 月自动日落）。
- `soul/_judgment_codex.md`：判断法典。结构：

```markdown
## 评分维度（scoring dimensions）
- negotiation: 定义 / 触发事件类型 / 默认 decay
- reliability:  定义 / 触发规则 / buff-debuff 映射
## 信任维度（trust dimensions）
- ...
## 技能定义（skill tree）
- client_psych: 各等级判定标准 / 前置 / 升级触发条件
## 可调动度公式（leverage）
leverage = 价值 × 可达性 × 关系健康度 − 调动成本   # 各项维度在此定义
## 迭代记录
- 2026-06: 维度 negotiation 命中率41%，拆分为 quote_game + relationship_keeping
```

K7 与所有 scoring/advisor 模块运行时从这里读维度，不内置（不变量 9）。命中率低触发 `codex.proposeAmendment`（法则八→法则十一）。

---

## 10. 创世流程实现（创建 TSC 的代码路径）

```
intake = 采集七问（见《魂》第五部分）— 缺一拒绝启动
 ├─ 1 存在宣言  → 写 _genesis.md（write-once）
 ├─ 2 初始目标  → world/operations/op_root
 ├─ 3 玩家      → world/players/p_*，记不可逆权限
 ├─ 4 环境      → 决定激活哪些 collector.platform.* + 长哪些领域职业槽
 ├─ 5 判断种子  → 写 _judgment_codex.md 初始维度
 ├─ 6 资源边界  → 写各职业 model_tier 预算
 └─ 7 成员类型  → 预建 NPC 分类

gaps = kernel.genesis.detectGaps()        # 对照 VSM 必备职业集
for gap in gaps:
    if gap.fillable_by_agent:
        draft = genesis_engine.spawn(AgentSpec(gap)); await player_approve(draft)
    elif gap.candidate_human_exists:
        propose_candidate(gap)             # 提名在册人选 + 调动成本
    else:
        recruiter.emit_intent(gap)         # 产出招募意图，或提示玩家："你需要找一个能做X的人"

init_soul()                                # genesis + judgment 种子 + 空 events + 空 calibration
mount_shell()                              # kernel + 必备模块（progressive disclosure 注册）
```

系统从出生第一刻就做缺口分析，主动决定"派 agent 补"还是"叫玩家找人"。这是法则十一在创建时的第一个具体动作。

---

## 11. 多 Agent 编排与成本

**模式选择（按 2026 生产经验，复杂度决定架构，不超前）**：

- 起步：**单 agent + 必备职业模块**。多数 TSC 起步阶段单 agent 足够（单 agent 在 64% 任务上不输多 agent）。
- 长大：**supervisor（指挥官）+ fan-out（采集/分析并行）**。指挥官用顶配模型，worker 用廉价模型，省 40–60% 成本。
- 协作：**烙印（stigmergy）而非中央调度**。哨兵在 vault 留 `needs_*` 标记 → 蒸馏师空闲扫到处理 → 预言家扫到 debuff 产出预测 → 简报官启动扫到到期预测浮出。agent 之间不直接互调，全靠读写共享环境。任一 agent 可替换、下线、换模型而循环不断。
- 质检：高风险决策用 **maker-checker**（一个 agent 出，另一个 review），降幻觉。

**VSM ↔ 模型档位 ↔ 频率**：

| VSM | 职业 | model_tier | 频率 |
|---|---|---|---|
| S1 | 执行者 | low | 高频 |
| S2 | 哨兵/协调员 | low | 高频/事件 |
| S3 | 蒸馏师/简报官/归因官 | high(蒸馏) / mid | 低频/定时 |
| S4 | 摄入官/预言家/招募官 | mid / high | 中/按需 |
| S5 | 指挥官/创世者 | high | 关键时刻 |

**成本铁律**：贵模型少用、便宜模型多用；属性快照避免重复读原料（事件越多，查询成本不涨——因为查的是 K7 推导的属性，不是重读事件历史）。

---

## 12. 施工里程碑（按此顺序，不要跳）

> 总原则：先把断掉的中段（判断）建出来，再扩感知与执行。先有消化系统，再喂食。

- **M0 · 魂壳骨架**：`soul/` 与 `shell/` 目录分离；K1 身份、K2 数据契约、K5 字段本体。能创建实体、能导出魂。
- **M1 · 感知地基**：K3 事件流 + 事件图谱（markdown + 反向索引）；collector.manual + collector.transcript；inbox/raw 草案流。
- **M2 · 判断引擎（最关键）**：K7 + judgment_codex 格式 + 属性三层 schema。每来新事件自动推导属性。`compare(A,B,dim)` 能直接回答"谁更强"而不重读历史。**做完这步，系统从带标签的文件夹变成情报系统。**
- **M3 · 校准闭环**：K4 + calibration.*；预测带到期日，session.idle 提醒回填；命中率统计。系统开始有"经验"。
- **M4 · 建议引擎**：advisor.match/relation/capability/leverage/intake；knowledge 三层管线（颗粒被自动命中）。
- **M5 · 职业与编排**：职业定义格式 + 预设 11 职业；supervisor + fan-out + 烙印协调；briefing.startup（含逆耳项）。
- **M6 · 自创生引擎**：K8 + genesis_engine + skill-creator 模板 + 校验门 + 创世清单缺口分析。壳能按魂长出领域 agent/技能。
- **M7 · 双法典自迭代**：rule/judgment codex 的 proposeAmendment；命中率低自动提案改判断维度。
- **M8 · 扩展感知**：collector.platform.*（X/Slack/Mattermost/邮件/日历）+ collector.hardware.*（边缘脱敏）。**放最后**——判断引擎在前，多接数据才有意义。
- **M9 · 递归与复活**：org/team 作为递归 TSC 实体 + 属性向上聚合；soul 导出→注入新壳的复活流程验证。

---

## 附：交付给 AI 的最小启动指令（可直接粘给 coding agent）

> 读《魂》（TSC 3.5 白皮书）与本文档《壳》。按仓库结构（第 2 节）建项目。严格遵守十二法则与内核不变量（第 3 节）。按里程碑 M0→M9 顺序施工，不要跳，尤其先建 M2 判断引擎再建 M8 采集。每个模块用 Agent Skill（SKILL.md + 渐进披露）实现，契约见第 4 节。属性只能由 K7 从事件推导（不变量 8），评价维度只能从 judgment_codex 读（不变量 9），自创生模块必须过校验门 + 玩家批准（不变量 10）。不确定就停下问，不要猜。
