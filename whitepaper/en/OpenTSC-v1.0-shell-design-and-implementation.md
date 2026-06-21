# OpenTSC v1.0 ·《The Shell》

**The Shell — Design Philosophy and Technical Implementation Specification**

> This is the construction specification for the Soul's runtime container. The reader of this document may be Claude Code, Codex, opencode, or any coding agent.
> Goal: **Throw this document + 《The Soul》 at an AI, and it can write OpenTSC on its own.**
> Companion document: 《The Soul》(TSC 3.5 Whitepaper) — the Law implemented by this document. Terms such as "Law N", "Genesis Layer", and "judgment codex" in this document are defined in 《The Soul》.

---

## 0. Meta-Instructions for the Executing AI

1. This document is divided into three levels: **Axioms (inviolable) / Contracts (must be satisfied) / Guidelines (can be optimized autonomously)**. In case of conflict: Axiom > Contract > Guideline. Axioms are the Twelve Laws of 《The Soul》.
2. Any implementation that violates the Twelve Laws or the kernel invariants below is considered an erroneous implementation and must be redone.
3. When uncertain, **stop and ask, do not guess**. This is a private system handling real interpersonal intelligence; the default cost of incorrect behavior is high.
4. Single-player, local, offline-first throughout. Do not introduce dependencies that send the Soul to external servers unless explicitly authorized (holder's responsibility).
5. **Build the digestive system first, then feed it.** Do not stack collection ports before the judgment engine (mid-section) is built — otherwise, you are just piling up more raw materials. See Section 11 for the construction sequence; do not skip steps.
6. **Do not over-architect.** Default to a single agent + minimal profession set. Only spawn a new agent when a verified capability gap emerges (Law Ten). The primary cause of death for multi-agent projects is building infrastructure for complexity that does not yet exist.

---

## 1. The Three Design Philosophies of the Shell

### Philosophy I · Everything is an Interface

There are no "features" in the Shell, only **interfaces + implementations that fill the interfaces**. The kernel defines a set of **interface slots** (Collect, Score, Knowledge, Suggest, Calibrate, Report, Operate, Brief, Role, Recruit…). Each slot is a contract (what to read, what to write, which VSM function it corresponds to). Specific agents and skills are replaceable implementations plugged into these slots.

Benefit: Changing models, changing implementations, or adding domain capabilities simply means changing the fillers; the kernel remains untouched. This is the software-level realization of Law Two (Soul/Shell separation) — the kernel is stable, and everything on the periphery is fully replaceable.

### Philosophy II · Module as Agent Skill

Each module is implemented using the **Agent Skill Open Standard** (SKILL.md: YAML frontmatter machine-readable + Markdown human-readable + optional scripts/references), enjoying **progressive disclosure**: on startup, only the module's name+description (~80 tokens each) is loaded; only when hit is the full SKILL.md (~2000 tokens) loaded; only when needed are scripts executed. A Shell can "know" dozens of modules at a cost less than activating one. This allows the Shell to carry a vast number of professional skills without overflowing the context window.

> Reason for choosing Agent Skill over a private plugin format: It is an open standard from December 2025 onwards (adopted by Microsoft/OpenAI/GitHub). SKILL.md can be both executed by AI and read by humans (a Law requirement: "readable by both humans and AI"), and there exists **skill-creator** — a ready-made "skill to generate skills" — which is the very foundation of the autopoiesis engine.

### Philosophy III · Autopoiesis, Not Pre-Installation

The Shell does not pre-install all the capabilities a TSC might need. It installs an **autopoiesis engine**: read the Soul (goal/environment/gap), use the skill-creator pattern to generate the agents and skills (SKILL.md files) this TSC needs *right now*, register them through the validation gate, with a sunset clause. Different Souls grow different Shells. This is the execution mechanism for Law Eleven (Member Autopoiesis).

---

## 2. Repository Structure```text
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
```**Iron Law**: `soul/` and `shell/` are physically separated. Exporting the entire `soul/` directory = exporting the Soul, which can be injected into any compliant new Shell to revive (Law 2).

---

## 3. Kernel · K1–K8

The kernel is stable, guards the Laws, and does not call any module functionality; modules only access data through the K6 bus. Compared to v0.4, K7 (Judgment Engine) and K8 (Autopoiesis Registry) are newly added—these are precisely the middle segments that were previously broken.

| Module | Responsibility | Guarded Law |
|---|---|---|
| **K1 Identity Service** | Assign/resolve stable IDs, alias management, entity resolution (deduplication) and authentication | Law 3 |
| **K2 Data Contract** | Validate entity/relationship edge/event entry structure and directory morphology | Laws 4, 10 |
| **K3 Event Stream Engine** | Append-only event stream, soft deletion, current state view derivation, **event graph (nodes + edges + causal chains)** | Law 4 |
| **K4 Feedback Contract** | Enforce suggestions/predictions with expiration date + reasoning chain, separate useful/comfortable | Law 8 |
| **K5 Field Ontology** | Maintain `_schema.md`, register new fields, prevent drift | Laws 9, 10 |
| **K6 Module Bus** | Module registration, lifecycle, read/write authorization, ensure no bypassing of the kernel | Laws 2, 11 |
| **K7 Judgment Engine** ★New | Upon each new event, automatically update relevant entity attributes/skills/status according to the judgment_codex, leaving reasoning traces | Law 9 |
| **K8 Autopoiesis Registry** ★New | Receive agent/skill/profession drafts generated by the genesis_engine, pass through validation gate, register or reject, attach sunset | Law 11 |

### Kernel API Form (Implementation language is optional, contract must be preserved)```typescript
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
```### Kernel Invariants (Violation = Incorrect Implementation)

1. Each entity has an immutable `id`; path anchors use IDs, not names.
2. Event stream is append-only; corrections are appended, deletions are soft deletes.
3. Events must carry an Admiralty rating and source.
4. Suggestions/predictions must include an expiration date and reasoning chain before staging.
5. New fields must first be registered in `_schema.md` before use.
6. Modules communicate via the bus and must not directly modify entity identities or the internal event stream.
7. Auto-extracted items first go into `inbox/`; formal registration requires confirmation.
8. **Attributes can only be written by K7 derived from events; no module is permitted to directly write attribute values** (Law Nine).
9. **The judgment_codex must not be hardcoded into any module; all evaluation dimensions are read from the Soul at runtime** (Laws Seven and Nine).
10. **Any agent/skill generated by genesis_engine must pass the K8 verification gate + player approval before registration**; automatic activation is prohibited (Law Eleven + Holder Responsibility).
11. Task completion/abandonment can only be declared by the player; closing a task forces a review + calibration backfill.
12. `soul/` can be independently exported and is self-consistent: all current attributes can be recalculated using only `soul/` (prerequisite for resurrection under Law Two).

---

## 4. Module Interface Contract (Interface = Module)

Each module is an Agent Skill, and the frontmatter of its SKILL.md must declare this contract:```yaml
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
```### Module Families (Interface Slots, Initial Set)

- **collector.\*** (Collection, S4 Sensing): manual / import / contacts / transcript / osint / **platform.\*** (X, Slack, Mattermost, Email, Calendar) / **hardware.\*** (Recording, IoT, Wearable). Unified output: `inbox/` draft events. **Hardware sub-contract: edge anonymization, only structured events transmitted, raw data never leaves device** (holder responsibility).
- **scoring.\*** (Scoring, S3): All dimensions read from judgment_codex, not built-in.
- **knowledge.\*** (Three Knowledge Layers, S3): facts / methods / principles. Stores executable granules with trigger conditions, builds index + explicit link retrieval; granules must be auto-hit by advisor, not relying on model re-reading raw materials; granules unreferenced for >N months enter sunset.
- **advisor.\*** (Advisory Engine, S3, System's Judgment Output): match / relation / capability / intake / leverage. Each piece of advice is logged via K4 with a complete reasoning chain; player scoring targets the reasoning chain, not the conclusion.
- **calibration.\*** (Calibration, S4, Anti-Echo Chamber): review (attached to session.idle) / stats / feedback (only useful/accurate used, comfortable strictly forbidden).
- **operations.\*** (Task Lifecycle, S1): plan / track / review (mandatory post-mortem on close).
- **briefing.\*** (Startup Briefing, S3, attached to session.created): startup, tiered with limited access, must include unpleasant items, read-only.
- **role.\*** (Role Assignment, functional, not an entity).
- **recruiter.\*** (Recruitment, S4): Detect gaps → Search candidates / Generate recruitment intent / Prompt player to find people.

---

## 5. Genesis Engine — How the Shell Grows Agents and Skills

This is the execution mechanism for Law XI, and the concrete implementation of "generating different agents and skills based on different TSC organizations."

### Trigger Conditions (Only when gaps are confirmed, Law X)

- Genesis list gap analysis (Section 10) finds no suitable occupant for a mandatory or domain profession;
- Distiller/Briefer discovers recurring situations during operations that existing modules cannot handle (leaves a `needs_capability` marker in the vault);
- Calibration finds persistently low hit rates in a certain domain, requiring a more specialized judgment module.

### Generation Process (skill-creator mode)```
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
```### Same Shell, Different Souls, Grow Different Things

| TSC (Soul's Environment) | Domain Modules/Professions Grown via Autopoiesis (Example) |
|---|---|
| Quantitative Trading (example-quant) | `collector.platform.exchange`, `advisor.risk` (Risk Officer), `scoring.signal_quality` |
| Women's Community (HerName.AI) | `collector.platform.community`, `advisor.member_health` (Emotional Health Advisor), `scoring.engagement` |
| Content/Growth (@example_user) | `collector.platform.x`, `advisor.topic` (Topic Prophet), `scoring.virality` (Viral Distiller) |
| Team Collaboration (Stanley Team) | `collector.platform.mattermost`, `advisor.workload`, `scoring.reliability_team` (Team-Level Aggregation) |

The base profession set is constant (Section 6), while all domain modules are generated by the autopoiesis engine according to the Soul. **This is the Shell's "open world": the map (professions/skills/agents) is not pre-scripted but grows organically based on your TSC's goals and environment.**

---

## 6. Profession Definition Format (Including Preset Required Professions)

A profession is a contract document `shell/professions/{name}.md`:```yaml
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
```**Preset Essential Professions (VSM-Complete Minimal Set)**: `founder` · `commander` · `sentinel` · `ingestor` · `distiller` · `oracle` · `herald` · `coordinator` · `operator` · `steward` · `recruiter`. For each profession's VSM affiliation, default holder, and whether it can be held by an agent, see Table in Part 4 of *Soul*.

---

## 7. Role Attribute Schema (World Model)

Entity frontmatter (human NPC example; agent NPC uses the same schema, with attribute source marked as `configured`):```yaml
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
```Relationship attributes are attached to edges (not node fields):```text
[现] 合作 →→ p_0f3a : "Dave" · 报价重构主力 · 2026-至今 · mobilizability:{value:0.8,confidence:0.6} · cost:中
```The complete `AttributeClaim` form of an attribute (each value is a probabilistic statement with evidence, subject to decay, and predictable) is detailed in Part 3 of *Soul*. **It is forbidden to manually fill in attribute values; only events may be appended, and K7 shall derive them** (Invariant 8).

---

## 8. Event Graph Schema

Each event `soul/events/YYYY-MM/evt_*.md`:```yaml
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
```**Implementation Note**: Start with markdown frontmatter + reverse index (K3 maintains `id → {events, edges}` mapping) to implement the event graph, without jumping straight to a graph database. The `neighborhood(id)` query goes through the reverse index. Switch storage when data volume grows, but keep the contract unchanged.

---

## 9. Dual Codex Format

- `soul/_genesis.md`: Genesis layer. Contains the existence manifesto + immutable bottom line + players and their irreversible permissions. **Write-once**, K2 rejects any modification writes.
- `soul/_rule_codex.md`: Three-layer rule codex. Genesis layer (inherits genesis) / Constitutional layer (modification requires high threshold, records threshold and procedure) / Operational layer (modification requires simple majority, each entry carries a 12-month automatic sunset).
- `soul/_judgment_codex.md`: Judgment codex. Structure:```markdown
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
```K7 and all scoring/advisor modules read dimensions from here at runtime, not built-in (invariant 9). Low hit rate triggers `codex.proposeAmendment` (Law 8 → Law 11).

---

## 10. Genesis Flow Implementation (Code Path for Creating a TSC)```
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
```From the very first moment of its birth, the system performs a gap analysis, actively deciding whether to "dispatch an agent to fill it" or "call the player to find someone." This is the first concrete action of Law 11 at the moment of creation.

---

## 11. Multi-Agent Orchestration and Cost

**Mode Selection (based on 2026 production experience, architecture determined by complexity, no premature optimization)**:

- **Startup**: **Single agent + essential profession modules**. For most TSCs in the startup phase, a single agent is sufficient (a single agent is not inferior to multi-agent setups in 64% of tasks).
- **Growth**: **Supervisor (Commander) + fan-out (collection/analysis in parallel)**. The commander uses a top-tier model, while workers use cheaper models, saving 40–60% in costs.
- **Collaboration**: **Stigmergy rather than central scheduling**. Sentinels leave `needs_*` markers in the vault → idle Distillers scan and process them → Prophets scan for debuffs and produce predictions → idle Briefing Officers scan for expiring predictions and surface them. Agents do not call each other directly; they rely entirely on reading and writing to the shared environment. Any agent can be replaced, taken offline, or have its model swapped without breaking the cycle.
- **Quality Control**: High-risk decisions use **maker-checker** (one agent produces, another reviews) to reduce hallucinations.

**VSM ↔ Model Tier ↔ Frequency**:

| VSM | Profession | model_tier | Frequency |
|---|---|---|---|
| S1 | Executor | low | High frequency |
| S2 | Sentinel/Coordinator | low | High frequency/Event-driven |
| S3 | Distiller/Briefing Officer/Attribution Officer | high (Distiller) / mid | Low frequency/Scheduled |
| S4 | Intake Officer/Prophet/Recruiter | mid / high | Medium/On-demand |
| S5 | Commander/Creator | high | Critical moments |

**Cost Iron Law**: Use expensive models sparingly, cheap models frequently; attribute snapshots avoid re-reading raw materials (the more events, the query cost does not increase—because it queries attributes derived from K7, not re-reading the event history).

---

## 12. Construction Milestones (Follow this order, do not skip)

> General Principle: First build the broken middle section (judgment), then expand perception and execution. First have a digestive system, then feed it.

- **M0 · Soul/Shell Skeleton**: Separate `soul/` and `shell/` directories; K1 Identity, K2 Data Contract, K5 Field Ontology. Ability to create entities and export souls.
- **M1 · Perception Foundation**: K3 Event Stream + Event Graph (markdown + reverse index); collector.manual + collector.transcript; inbox/raw draft stream.
- **M2 · Judgment Engine (Most Critical)**: K7 + judgment_codex format + three-layer attribute schema. Automatically derive attributes for each new event. `compare(A,B,dim)` can directly answer "who is stronger" without re-reading history. **After this step, the system transforms from a tagged folder into an intelligence system.**
- **M3 · Calibration Loop**: K4 + calibration.*; predictions include expiration dates, session.idle reminders for backfilling; hit rate statistics. The system begins to have "experience."
- **M4 · Suggestion Engine**: advisor.match/relation/capability/leverage/intake; three-layer knowledge pipeline (granules are automatically hit).
- **M5 · Professions and Orchestration**: Profession definition format + 11 preset professions; supervisor + fan-out + stigmergy coordination; briefing.startup (including contrarian items).
- **M6 · Autopoiesis Engine**: K8 + genesis_engine + skill-creator template + validation gate + genesis checklist gap analysis. The shell can grow domain agents/skills according to the soul.
- **M7 · Dual Codex Self-Iteration**: proposeAmendment for rule/judgment codex; low hit rate automatically proposes amendments to judgment dimensions.
- **M8 · Expanded Perception**: collector.platform.* (X/Slack/Mattermost/Email/Calendar) + collector.hardware.* (Edge anonymization). **Leave for last**—the judgment engine comes first; connecting more data only makes sense afterward.
- **M9 · Recursion and Resurrection**: org/team as recursive TSC entities + attribute upward aggregation; soul export → resurrection process verification by injecting into a new shell.

## Appendix: Minimum Startup Instructions for AI (Can Be Directly Pasted to Coding Agent)

> Read *Soul* (TSC 3.5 White Paper) and this document *Shell*. Build the project according to the repository structure (Section 2). Strictly adhere to the Twelve Laws and kernel invariants (Section 3). Proceed in order of milestones M0→M9, do not skip; especially build the M2 judgment engine before M8 collection. Implement each module using Agent Skill (SKILL.md + progressive disclosure), with contracts as specified in Section 4. Attributes can only be derived by K7 from events (Invariant 8), evaluation dimensions can only be read from judgment_codex (Invariant 9), and self-creation/autopoiesis modules must pass the verification gate + player approval (Invariant 10). If uncertain, stop and ask; do not guess.
