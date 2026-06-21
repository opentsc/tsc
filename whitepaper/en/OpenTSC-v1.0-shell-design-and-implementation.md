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
├── soul/                         # soul (Law 2: can be fully exported/revived)
│   ├── _genesis.md               # genesis layer, write-once (Law 1)
│   ├── _rule_codex.md            # rule codex: genesis/constitution/operation three layers (Law 7)
│   ├── _judgment_codex.md        # judgment codex: scoring/trust dimensions, skill definitions, deployability formula (Law 7)
│   ├── _schema.md                # Field ontology live dictionary (K5)
│   ├── events/                   # global event stream, append-only (Law 4)
│   │   └── YYYY-MM/evt_*.md      # one node per event, with links/caused_by
│   └── calibration/              # Calibrate ledger (Law 8)
│       └── YYYY-MM/
├── shell/                        # Shell (rewritable, replaceable)
│   ├── kernel/                   # Core K1–K8, stable
│   ├── modules/                  # module = Agent Skills (interface implementation)
│   │   ├── _registry.md          # module registry (progressive disclosure index)
│   │   ├── collector.*/SKILL.md
│   │   ├── scoring.*/SKILL.md
│   │   ├── advisor.*/SKILL.md
│   │   └── ...                   # See Section 4 module family
│   ├── professions/              # Occupation definition (contract)
│   │   └── *.md                  # See Section 6
│   └── genesis_engine/           # autopoietic engine (see Section 5)
├── world/                        # world model (entity, Laws 3/4/9)
│   ├── players/p_*/
│   ├── npcs/
│   │   ├── agents/a_*/           # Agent NPC (controllable)
│   │   └── humans/p_*/           # human NPC (uncontrollable, observed and inferred)
│   ├── orgs/o_*/                 # organizational entity (recursive TSC, Law 12)
│   ├── operations/op_*/          # Task as entity
│   └── roles/                    # Role assignment (function, not entity)
├── raw/                          # raw material with manifest+hash (Law 6)
├── inbox/                        # auto-extracted draft, pending confirmation (Law 6)
└── reports/YYYY-MM/              # Temporal artifact
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
  events: {                                                 // K3 + event graph
    append(event: IntelEvent): EventId                      // K2/K3 validation: must include source + Admiralty
    link(eventId: EventId, targets: EntityId[]): void       // One event links multiple entities
    cause(from: EventId, to: EventId): void                 // Causal edge
    timeline(filter: GraphQuery): IntelEvent[]
    deriveView(id: EntityId): EntityView                    // Current state = derived, not stored
    neighborhood(id: EntityId): EventGraph                  // Pull full event network of an entity
  }
  judgment: {                                               // K7 ★
    onEvent(eventId: EventId): AttributePatch[]             // derives attribute changes per judgment_codex
    attribute(id: EntityId, dim: string): AttributeClaim    // {value,confidence,provenance,reviewed,decay}
    compare(idA: EntityId, idB: EntityId, dim: string): Comparison   // 'who is stronger' directly queried, no need to re-read history
    explain(id: EntityId, dim: string): ReasoningChain      // By which events this attribute
  }
  feedback: {                                               // K4
    stage(prediction: Prediction): PredictionId             // must include due_date + reasoning_chain
    due(): Prediction[]
    fulfill(id: PredictionId, outcome: Outcome): void
    stats(filter?: FeedbackFilter): FeedbackStats           // hit rate decomposed by context/knowledge source
  }
  codex: {
    rule(): RuleCodex                                       // Read-only, change requires governance process
    judgment(): JudgmentCodex
    proposeAmendment(target: "rule"|"judgment", change: Amendment): ProposalId  // Self-iterating
  }
  schema: { register(field: FieldDef): void; known(): FieldDef[] }   // K5
  genesis: {                                                // K8 ★
    detectGaps(): CapabilityGap[]                           // VSM mandatory occupations + domain occupation gaps
    spawn(spec: AgentSpec | SkillSpec | ProfessionSpec): DraftId   // Draft, pending approval
    validate(draftId: DraftId): ValidationResult            // Validation gate
    register(draftId: DraftId): ModuleRef                   // Register + sunset clause
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
description: Task-personnel matching suggestion engine. Used when players ask "who is suitable for X" or "arrange today's tasks".
vsm: S3                       # maps to VSM subsystem (S1–S5), determines orchestration layer
profession: commander            # served by/which occupation calls it (can be empty)
reads:  [entities, events, knowledge, judgment]    # Authorized by K6
writes: [feedback]                                 # Authorized by K6
binding: { kind: tool, ref: advisor_match }
model_tier: high              # high|mid|low — orchestration allocates models by this to control cost
trigger: { kind: invoke }     # invoke|event|hook(session.created/idle)
sunset: { unused_after_months: 6 }
genesis: { template: advisor, domain: generic }    # generated by which autopoietic template (domain can be empty)
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
Gap (CapabilityGap: what VSM function / what domain / which profession to serve)
 → Select template: genesis_engine/templates/{collector|scoring|advisor|...}.skill-template.md
 → Read Soul: genesis (bottom-line constraints) + judgment codex (evaluation dimensions) + environment (domain vocabulary)
 → LLM generates candidate SKILL.md according to template (frontmatter contract + instructions + optional script)
 → K8 Validation gate：
     · Is the frontmatter contract complete? Are reads/writes within authorization scope? Is vsm valid?
     · Does it violate any Law? (especially Law 8/9/10: no built-in dimensions, no direct attribute writing, no auto-activation)
     · Duplicate with existing modules? (if duplicate, suggest reuse rather than new creation)
 → Output draft, status draft, write to inbox
 → Player approval (invariant 10) → K8 register → write _registry.md → set sunset
 → Reject → record reason, provide feedback for next generation
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
display: sentinel
vsm: S2/S4
holder_type: agent              # agent | human | player | any
model_tier: low
required_skills:                # skill tree node (skill definition from judgment_codex)
  - anomaly_detection: ">=Lv.2"
  - deadline_tracking: ">=Lv.1"
attribute_thresholds:           # Base attribute threshold
  reliability: ">=0.6"
reads:  [entities, events, feedback]
writes: []                      # sentinel read-only, leaves mark in vault on discovery (brand coordination)
triggers: [session.created, session.idle, schedule.hourly]
evolves_to: [scout]             # evolvable subclass (LitRPG class evolution)
---
# sentinel
Monitor anomalies, monitor deadlines, monitor pattern changes. When found, leave `needs_*` markers in the event stream, do not handle directly (brand coordination). Must be able to produce unpleasant items.
```**Preset Essential Professions (VSM-Complete Minimal Set)**: `founder` · `commander` · `sentinel` · `ingestor` · `distiller` · `oracle` · `herald` · `coordinator` · `operator` · `steward` · `recruiter`. For each profession's VSM affiliation, default holder, and whether it can be held by an agent, see Table in Part 4 of *Soul*.

---

## 7. Role Attribute Schema (World Model)

Entity frontmatter (human NPC example; agent NPC uses the same schema, with attribute source marked as `configured`):```yaml
---
id: p_1c2e
type: human_npc                 # player | human_npc | agent_npc | org | operation
names: { real: TODO(user), aliases: [{value: "Grace", platform: x, status: confirmed}] }
professions: [operator]         # multi-class (fluid class)
base:                           # base attribute layer (innate, slow-changing)
  execution_ceiling: { value: 0.7, confidence: 0.5, provenance: [evt_a1], reviewed: 2026-05, decay: 0.02 }
  resilience:        { value: 0.5, confidence: 0.4, provenance: [evt_b3], reviewed: 2026-05, decay: 0.02 }
skills:                         # skill tree layer (learned, event-driven)
  - id: client_psych
    level: 3
    prereq_met: true
    leveled_by: [evt_c1, evt_c9]    # Which events triggered upgrade
states:                         # state layer (buff/debuff with countdown)
  - tag: "Read but no reply · reliability questionable"
    kind: debuff
    expires: 2026-06-08
    on_repeat: solidify         # Repeat offense solidifies to permanent
    source: evt_d2
trust: { reviewed: 2026-06-01 } # trust dimension, read from judgment_codex
source_mode: inferred           # inferred (human) | configured (agent)
tags: []
---
```Relationship attributes are attached to edges (not node fields):```text
[current] collaboration →→ p_0f3a : "Dave" · quotation restructuring lead · 2026-present · mobilizability:{value:0.8,confidence:0.6} · cost:medium
```The complete `AttributeClaim` form of an attribute (each value is a probabilistic statement with evidence, subject to decay, and predictable) is detailed in Part 3 of *Soul*. **It is forbidden to manually fill in attribute values; only events may be appended, and K7 shall derive them** (Invariant 8).

---

## 8. Event Graph Schema

Each event `soul/events/YYYY-MM/evt_*.md`:```yaml
---
id: evt_0a1f
date: 2026-06-01
admiralty: B2                   # source reliability B × information credibility 2 (Law 5)
content: "Carol proactively mentioned during the call with Eve that the team was established only 3 months ago"
source: "Meeting transcript #5"
raw: raw_2026-06-01_call5
status: active                  # active | corrected | soft_deleted
links: [p_carol, p_eve, op_001]      # one event links multiple entities (no longer tied to one person)
caused_by: []                   # Upstream causality
causes: [evt_0a23]              # downstream causality: → 'Eve trust damaged'
judgment_triggered: [p_carol.trust, p_carol.skills.client_psych]   # Which attributes K7 updated accordingly
---
```**Implementation Note**: Start with markdown frontmatter + reverse index (K3 maintains `id → {events, edges}` mapping) to implement the event graph, without jumping straight to a graph database. The `neighborhood(id)` query goes through the reverse index. Switch storage when data volume grows, but keep the contract unchanged.

---

## 9. Dual Codex Format

- `soul/_genesis.md`: Genesis layer. Contains the existence manifesto + immutable bottom line + players and their irreversible permissions. **Write-once**, K2 rejects any modification writes.
- `soul/_rule_codex.md`: Three-layer rule codex. Genesis layer (inherits genesis) / Constitutional layer (modification requires high threshold, records threshold and procedure) / Operational layer (modification requires simple majority, each entry carries a 12-month automatic sunset).
- `soul/_judgment_codex.md`: Judgment codex. Structure:```markdown
## scoring dimensions
- negotiation: Definition / Trigger event type / Default decay
- reliability:  Definition / Trigger rule / Buff-debuff mapping
## trust dimensions
- ...
## skill tree
- client_psych: Judgment criteria per level / Prerequisites / Upgrade trigger conditions
## leverage formula
leverage = Value × Accessibility × Relationship health − Mobilization cost   # Dimensions defined here
## iteration records
- 2026-06: Dimension negotiation hit rate 41%, split into quote_game + relationship_keeping
```K7 and all scoring/advisor modules read dimensions from here at runtime, not built-in (invariant 9). Low hit rate triggers `codex.proposeAmendment` (Law 8 → Law 11).

---

## 10. Genesis Flow Implementation (Code Path for Creating a TSC)```
intake = Seven collection questions (see Part 5 of the Soul) — Refuse to start if any is missing
 ├─ 1 Genesis Manifesto → write _genesis.md (write-once)
 ├─ 2 Initial Objective → world/operations/op_root
 ├─ 3 Player → world/players/p_*, record irreversible permissions
 ├─ 4 Environment → decide which collector.platform.* to activate + which domain profession slots to grow
 ├─ 5 Judgment Seed → write _judgment_codex.md initial dimensions
 ├─ 6 Resource Boundary → write each profession model_tier budget
 └─ 7 Member Type → pre-build NPC classification

gaps = kernel.genesis.detectGaps()        # Cross-reference VSM required occupation set
for gap in gaps:
    if gap.fillable_by_agent:
        draft = genesis_engine.spawn(AgentSpec(gap)); await player_approve(draft)
    elif gap.candidate_human_exists:
        propose_candidate(gap)             # Nominate registered candidates + mobilization cost
    else:
        recruiter.emit_intent(gap)         # outputs recruitment intent, or prompts player: 'You need someone who can do X'

init_soul()                                # genesis + judgment seed + empty events + empty calibration
mount_shell()                              # kernel + essential modules (progressive disclosure registration)
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
