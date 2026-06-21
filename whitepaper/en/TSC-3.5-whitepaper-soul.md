# TSC 3.5 White Paper · *The Soul*

**The Thin-Shell Company 3.5 — The Soul: Laws of a Self-Evolving Organization**

> Version v3.5 · Predecessor v3.0 (Autopoiesis · VSM · DAO · Recursion)
> Positioning: This is a **law**, not a manual. It defines why a TSC organization exists, what counts as good, and how it evolves. It must be readable by humans and executable by AI.
> Companion Document: *The Shell* (OpenTSC v1.0 Design Philosophy and Technical Implementation) — the operational container for the Soul.

---

## Preamble: Soul and Shell

v3.0 described TSC as an organizational entity capable of self-evolution. v3.5 splits it into two halves, making the distinction clearer:

> **TSC is the Soul, OpenTSC is the Shell.**
> **Soul = This set of laws + all accumulated judgments and memories of the organization.** It is abstract, portable, and determines the "why" and "what counts as good."
> **Shell = The container that runs this set of laws.** It is composed of modules, agents, and skills; it is modular, replaceable, and can extend to hardware, determining the "how."

Why this split? Because the entire bet of the Thin-Shell Company is "one person driving a large organization." The thinner the Shell, the thicker the Soul must be, and the more capable it must be of surviving independently of any specific person or any specific software. If an organization's judgment exists only in the founder's mind and its memory exists only in a particular software suite, then when its Shell breaks, its Soul dies with it.

v3.5 requires: **The Soul can be completely extracted from one Shell and injected into another Shell to be resurrected.** The software can be rewritten, the team can change, the models can be upgraded—as long as the Soul (the Genesis commitment + the judgment codex + the event stream + the calibration memory) remains, this TSC is still the same TSC. This is the answer to the Ship of Theseus problem at the organizational level, and it carries the full weight of the name "Soul/Shell."

```
       Soul — Portable, survives across Shells
   ┌──────────────────────────────────┐
   │  Genesis Layer   Immutable commitment to exist   │
   │  Judgment Codex  What counts as good/trustworthy/worth acting on │
   │  Event Stream    Complete trajectory with evidence        │
   │  Calibration Memory Ledger of right and wrong judgments      │
   └──────────────┬───────────────────┘
                  │ Injection / Resurrection
   ┌──────────────▼───────────────────┐
   │  Shell — OpenTSC, replaceable, rewritable │
   │  Kernel · Module Interfaces · Agents · Skills · Hardware │
   └──────────────────────────────────┘
```

---

# Part One: The Twelve Laws

The laws are divided into four families, each with three laws. Each law is given in three statements: **Human Language** (understandable by anyone), **Machine Language** (executable by AI), and **Root** (the originating principle). These twelve laws form the Genesis layer of a TSC and cannot be violated; to change them, one must first amend this law.

## Family of Existence — What a TSC Is

### Law I · Immutable Genesis

- **Human Language**: The reason a TSC exists and the bottom line it would rather die than cross, once set, cannot be changed. If changed, it becomes a different TSC.
- **Machine Language**: Fields in the `genesis` layer are write-once, never modified; any change to genesis is equivalent to destroying the current entity and creating a new `tsc_id`. Day-to-day decisions do not touch genesis; genesis only appears as the supreme arbiter when other decisions conflict with it.
- **Root**: Autopoiesis Condition 4 (Core unchangeable) · VSM System 5 (Identity) · Ship of Theseus.

### Law II · Soul/Shell Separation

- **Human Language**: The law and memory (Soul) are separate from the software that runs them (Shell). If the Shell is broken, the Soul can be poured into a new Shell and come back to life.
- **Machine Language**: `soul = {genesis, judgment_codex, event_stream, calibration_memory}` is physically stored separately from `shell = {kernel, modules, agents, skills}`. The Soul is a pure data contract, independent of any specific Shell implementation; any compliant Shell can load any Soul and continue running. Resurrection = initializing a new Shell with an old Soul.
- **Root**: Autopoiesis (Self-reproduction) · SoulShell persistent memory and resurrection mechanism.

### Law III · Identity by ID, Not by Name

- **Human Language**: Every person, every organization, every task has an internally permanent, unchangeable number. Names, aliases, and accounts are merely labels under this number.
- **Machine Language**: Every entity has a unique, immutable `id` (e.g., `p_0f3a` / `o_*` / `op_*` / `tsc_*`). Names, aliases, and platform accounts are attributes under the `id`, with a `confirmed | suspected` status. File/path anchors use IDs, never names.
- **Root**: Identity continuity is defined by the Genesis layer, not by any specific component.

> **Holder Responsibility (Pervasive across the Family of Existence)**: The Soul holds extreme information asymmetry—you know the background of every NPC, and the other party does not know they are being recorded. Therefore, local storage, encryption at rest, offline-first design, and access control are the foundation, not an option. The strength of this point is equivalent to a law, written here as a reminder: **The more valuable the Soul, the tighter it must be locked.**

## Family of Perception — How a TSC Sees the World

### Law IV · Store Trajectories, Not Snapshots

- **Human Language**: Intelligence is fluid. The system records a stream of "what happened when." The current state is calculated from this stream, not a field that is repeatedly overwritten.
- **Machine Language**: The event stream is append-only. The current state is a derived view, not a persistent field. Corrections are expressed as new events (old events are marked as soft-deleted, preserving the record), never physically overwritten.
- **Root**: Autopoiesis Condition 1 (Internal state observable) · Generative agent "memory stream."

### Law V · Evidence Precedes Judgment

- **Plain language**: Evidence first, judgment second. Every piece of intelligence must state "who said it and how credible it is"; every recommendation must state "based on which pieces of intelligence and what logic."
- **Machine language**: Every event must carry `source` + Admiralty dual-axis rating (source reliability A–F × information credibility 1–6, stored separately). Every recommendation/prediction must carry `reasoning_chain` + `due_date`, otherwise it cannot be stored. Hostile/negative judgments must carry a reviewable evidence chain; a one-time characterization must not contaminate all subsequent interpretations.
- **Root**: Data contract · Anti-manipulation · The "hallucinated memory" failure mode of generative agents (blocked by the evidence gate).

### Law 6 · Draft Before Registry

- **Plain language**: Automatically extracted items are first treated as "drafts"; they become formal intelligence only after confirmation.
- **Machine language**: Collector auto-outputs go into `inbox/` as candidates, with status `draft | pending_verification`; entry into the formal event stream requires user confirmation or explicit pending-verification marking. Raw materials remain in `raw/`, with manifest and hash.
- **Root**: Avoid echo chambers and hallucination contamination · Holder responsibility.

## The Judgment Family (Laws of Judgment) — How TSC Decides What Counts as Good

### Law 7 · Dual Codex

- **Plain language**: An organization that evolves needs two codes. One governs "what is permitted" (rules), the other governs "what counts as good, who counts as credible, what is worth mobilizing" (judgment criteria). The latter used to exist only in your mind; now it must be written into the system.
- **Machine language**: `rule_codex` (Genesis/Constitution/Operational three layers, defining permissions and prohibitions, amendment thresholds and procedures) and `judgment_codex` (scoring dimensions, trust dimensions, skill level definitions, mobilizability formula) are established as two co-equal constitutions, sharing the same three-layer structure and self-iteration mechanism. The system's evaluation dimensions must not be hardcoded internally; all must be read from `judgment_codex`.
- **Root**: OpenTSC practical exposure — "Evaluation criteria exist in model weights, not in the system; change the model and the conclusion changes." This is the biggest conceptual increment of 3.5 over 3.0.

### Law 8 · Reality Calibration, Not Emotion

- **Plain language**: The system adjusts itself based only on "whether predictions are accurate and useful," never on "whether it makes the founder comfortable." A system that dares not speak unwelcome judgments is useless.
- **Machine language**: Distinguish two types of feedback — `useful/accurate` (can be used to adjust generation) and `comfortable` (recorded only, prohibited from use in adjustment). Every prediction is forcibly backfilled with actual results upon expiration; hit rate is decomposed by context/knowledge source and is the sole legitimate learning signal. Manipulation sources include the founder's own preferences.
- **Root**: Evolutionary game theory · Evolutionarily Stable Strategy (ESS) · Defensive design (against corruption).

### Law 9 · Attributes Derived from Events, Not Manual Entry

- **Plain language**: You cannot directly score a person. You can only record what they did, and the system automatically calculates their attributes, skills, and status according to the judgment codex. This way, no matter who reads it, the conclusion is the same.
- **Machine language**: All character attributes (basic/skill/status) are derived values; each value carries `provenance` (the event_ids that triggered it) + `confidence` + `reviewed_date` + `decay`. Direct writing of attributes is prohibited; writing can only occur by appending events, from which the judgment engine derives updates according to `judgment_codex`.
- **Root**: Generative agent "reflection" (synthesizing higher-level inferences from memory) · Requirement for judgment reproducibility.

## The Evolution Family (Laws of Evolution) — How TSC Grows

### Law 10 · Grow on Demand

- **Plain language**: Default to minimal. An entity starts as a file; it grows into a folder only when it becomes important. An organization starts with the fewest agents; new agents grow only when a wall is hit. Do not pre-stack architecture.
- **Machine language**: Entities default to lightweight nodes, splitting into sub-documents based on dimensional importance. Agents/skills/modules are generated by the autopoiesis engine only when a **confirmed capability gap** appears (see Law 11). Orchestration complexity follows load complexity, not ahead of it. Long-unused modules enter the sunset process.
- **Root**: Autopoiesis (boundary expansion on demand) · Multi-agent engineering lessons (40% of multi-agent projects fail due to over-architecture; single agents are not inferior to multi-agent on 64% of tasks — so keep it simple first).

### Law 11 · Fourfold Autopoiesis

- **Plain language**: The organization can grow what it needs by itself. When new rules are needed, propose new rules; when new capabilities are needed, find people or create agents; when better goals are discovered, change goals; when tasks require it, expand boundaries, and retract them when done.
- **Machine language**: All four autopoiesis loops must satisfy "observable → triggerable response → feedback loop → do not touch the Genesis core":
  - **Rule Autopoiesis**: Rules repeatedly violated/cited → auto-propose amendment to rule/judgment codex.
  - **Member (Agent) Autopoiesis**: Capability gap detected → generate agent or dispatch recruiter to find people (see Part 6 Genesis Checklist).
  - **Goal Autopoiesis**: Higher-value opportunity discovered during execution → converted into a goal amendment proposal, following governance process.
  - **Boundary Autopoiesis**: Dynamically include/withdraw members based on tasks (including fork and merge protocols).
- **Root**: Four levels of autopoiesis · Autonomous synthesis and combinatorial acquisition of agent skills.

### Law 12 · Recursive Isomorphism

- **Plain language**: A person is a Shell, a team is a Shell, a company is a Shell, the entire ecosystem is too. Each layer uses the same cycle, the same attributes, the same law. The map can drill down infinitely.
- **Machine language**: `person ⊂ team ⊂ org ⊂ ecosystem` are all TSC entities, recursively nested. The same "Observe→Judge→Decide→Act" cycle and the same attribute ontology (basic/skill/status) run independently at each layer and can be aggregated upwards. Each System 1 operational unit is itself a complete viable system (VSM recursiveness).
- **Root**: VSM recursiveness · Emergence · TSC^n.

> **Imprint Coordination (Collaboration rules for the Evolution Family)**: Multiple agents in the organization do not call each other via a central commander, but coordinate by reading and writing marks in a shared environment (vault/event stream) — one agent leaves a mark, another agent reads it and acts. This allows any agent to be replaced, taken offline, or have its model switched at any time without breaking the cycle. This is the realization of "stable kernel, variable edge" at the collaboration level, and a prerequisite for scaling without bottlenecks.

---

# Part 2: Profession · Player · NPC Ontology

3.5 Introduce a game ontology so that "who is who and who can do what" is crystal clear to both humans and AI.

## Three Types of Entities

### Player

- **Definition**: A person who holds a Soul. Makes irreversible decisions, holds the Genesis layer, and is the ultimate beneficiary the system serves.
- **Characteristics**: Can be one or more (co-founders). The Player is the only role that can modify the Genesis layer, approve goal amendments, confirm drafts for inclusion in the codex, and declare tasks complete or abandoned.
- **Key Point**: The Player is not "scored" by the system—the system is an extension of the Player's cognition, not their examiner. However, the Player's own attributes (e.g., execution ability, stress tolerance) can be calibrated by their own events (the Player is also an entity within the world), used for self-management, not for scheduling by others.

### NPC (Non-Player Character)

NPC = All actors other than the Player, divided into two **fundamentally different** subclasses. The system's relationship with each is completely distinct:

| | Agent NPC (Endogenous / System-Controllable) | Human NPC (Exogenous / Uncontrollable) |
|---|---|---|
| What it is | An AI agent instantiated and controlled by TSC | A real person (member, client, competitor, potential collaborator) |
| System's relationship to it | Can generate, configure, reset, deactivate | Can only **observe, judge, attempt to mobilize**; cannot control |
| Attribute source | Deterministic configuration + runtime performance | **Inferred** from event streams, with uncertainty, requiring calibration |
| Free will | None (acts according to prompt + skills) | Yes (can change, lie, drop the ball, betray) |
| Modeling architecture | Memory-Reflection-Planning (controllable version) | Memory-Reflection-Planning (observational inference version) |

**This distinction is the hardcore essence of 3.5**: Human NPCs are always part of the "external environment." The system's total power over them is limited to "seeing clearly" and "mobilizing," never "controlling." Treating a Human NPC like an agent by "assigning tasks" is a category error—you can only record their events, infer their attributes, predict their behavior, and calculate the cost of mobilizing them.

> Both subclasses share the same modeling architecture (Memory Stream → Reflection → Planning → Action, derived from generative agent research). This allows the system to describe all NPCs with a single contract; the only difference is whether attributes are "configured" or "inferred."

### Profession (Class)

- **Definition**: A role archetype. It is a **contract** stipulating: "To hold this profession, one must possess these skills, meet these attribute thresholds, correspond to which VSM function, and read/write which data."
- **Characteristics**:
  - Profession ≠ Person. A profession can be held by an Agent NPC (cheap, resettable), a Human NPC (expensive, uncontrollable), or the Player themselves.
  - An NPC can hold multiple professions simultaneously (fluid class system, referencing Cyberpunk's fluid class).
  - Professions can evolve: upgrade when conditions are met, spawning sub-classes (referencing LitRPG class evolution).
  - A profession is the smallest unit of **self-creation** within a Shell—if a new TSC lacks a profession, the system generates a corresponding agent or dispatches a person to fill it (Law 11: Member Autopoiesis).

---

# Part 3: Character Attributes — The Most Advanced Architecture

Attributes are the world model of this game. 3.5 structures them as follows, aiming to be "the most advanced" and "directly computable by any AI."

## Three-Layer Attributes

```
Character Profile = Base Attribute Layer + Skill Tree Layer + State Layer
```

- **Base Attribute Layer (innate, slow-changing, semi-annual scale)**: Execution ceiling, learning speed, stress tolerance, reliability baseline, autonomy. Relatively stable, representing "talent." Configured values for Agent NPCs; long-term inferred values for Human NPCs.
- **Skill Tree Layer (learned, event-driven, node-based)**: Has levels and prerequisites; upgrades are **triggered by events** (not bought with points). Example: `Quote Restructuring Lv.5` requires `Industry Insight Lv.3` + `Client Psychology Lv.3`. References dynamic skill trees—abilities unlock through real events, not subjective scoring.
- **State Layer (state, buffs/debuffs with countdowns)**: Temporary bonuses/penalties. Example: `Left on Read: Reliability Questionable` debuff, one-week observation period; auto-clears if not repeated, solidifies into a permanent trait if repeated. Structured fields, queryable, comparable, auto-filterable.

## Six Advanced Design Principles

These six principles define what makes the attribute architecture "advanced," each making attributes more powerful than traditional database fields or RPG stats:

1. **Attributes are derived values, not stored values** (Law 9). You cannot modify an attribute; you can only add events. The system calculates the attribute. This guarantees objectivity and reproducibility.

2. **Every attribute is a probabilistic statement with evidence, not a single number**. The complete form of an attribute value:
   ```yaml
   negotiation:
     value: 0.72            # Current estimate
     confidence: 0.6        # How certain this estimate is
     provenance: [evt_0a1, evt_0b9, evt_112]   # Based on which events
     reviewed: 2026-06-01   # Last updated
     decay: 0.03/month      # Decays if no new evidence
     source_admiralty: B2   # Average credibility of evidence
   ```
   Every belief is auditable, has an expiration, and decays. This is the first principle of being "advanced."

3. **Agent NPCs and Human NPCs share the same schema but have different sources**. The same attribute structure is populated by configuration + performance for Agents, and by observation + inference for Humans. The system uses the same query interface for all characters but always knows which are "configured certainties" and which are "inferred uncertainties."

4. **Attributes are fractal—the same ontology is reused at every level** (Law 12). An individual has execution ability; a team also has execution ability (aggregation of members); a company does too. Querying Stanley Team's negotiation ability = aggregating member negotiation attributes × relationship health. One schema, recursively applied to person/team/org/ecosystem.

5. **Relationship attributes are edges, not node fields**. Trust, mobilizability, and rapport cost are attributes "between you and them," attached to the relationship edge, with direction, time, and requiring calibration. They are not a number on a single person. `advisor.leverage` is calculated as `Value × Reachability × Relationship Health − Mobilization Cost`, all sourced from edges.

6. **Attributes are predictive, not merely descriptive**. Beyond "X's current negotiation capability," the system stores "X's expected performance in context C"—a probability distribution fed by the Prophet and calibration. An attribute is simultaneously a **generative model of future behavior**, not just a historical portrait. This is the endpoint toward which the attribute architecture points: moving from "recording who this person is" to "predicting what this person will do."

---

# Part IV: VSM Essential Professions (Preset Minimum Professions)

For a TSC to be "viable" (in the VSM sense of surviving in a complex environment), it must cover the functions of all five subsystems. Below is the **minimum set of professions that must be in place when creating any TSC**. Each profession is annotated with: VSM affiliation, default role holder, whether it can be held by an Agent, and core responsibilities.

| Profession | VSM | Default Holder | Agent-able | Core Responsibility |
|---|---|---|---|---|
| **Genesis / Player** | S5 | Human (Player) | No | Holds the Genesis layer and judgment codex; supreme arbiter; confirms drafts; declares mission termination |
| **Commander** | S3/S5 | Player or Top-tier Agent | Partial | Strategic decision-making; invokes all lower-level outputs; triggers at critical moments |
| **Sentinel** | S2/S4 | Agent (cheap, high-frequency) | Yes | Monitors anomalies, deadlines, pattern shifts; read-only; leaves markers in the vault upon detection |
| **Intake Officer** | S4 | Agent (mid-tier) | Yes | Receives new material; classifies and segments; identifies participants; multimodal; produces inbox drafts |
| **Distiller** | S3 | Agent (top-tier, low-frequency) | Yes | Extracts intelligence from raw material; updates attributes; builds causal edges—the core of turning raw material into intelligence |
| **Prophet** | S4 | Agent (top-tier, on-demand) | Yes | Predicts human/project trajectories; stores predictions with expiration dates; backfills calibration |
| **Briefer** | S3 | Agent (mid-tier, scheduled) | Yes | Generates reports and task assignments; surfaces actionable items upon activation; **must include dissenting items** |
| **Coordinator** | S2 | Agent | Yes | Detects and resolves resource conflicts and interface conflicts; coordinates interfaces only, does not manage units |
| **Executor** | S1 | Human or Agent | Yes | Directly produces the TSC's core output; autonomous within their scope of responsibility |
| **Attributor / Treasurer** | S3 | Agent + Player confirmation | Partial | Contribution attribution; resource (compute/attention) allocation; contribution token issuance; evidence-based, not relationship-based |
| **Recruiter / Headhunter** | S4 | Agent | Yes | Detects capability gaps; searches for talent in the capability market; produces recruitment intent or prompts the player to find someone |

**Notes**:
- These 11 constitute the minimum VSM-complete set. When a single-person TSC starts, the player personally serves as both Genesis and Commander, delegating the rest to cheap Agents as much as possible, upgrading only when hitting a wall (Law X).
- The Recruiter is the executor of Law XI "Member Autopoiesis": when an essential profession has neither a suitable Agent to fill it nor a suitable human NPC on the roster, the Recruiter produces a recruitment intent stating "need someone who can do X," and the system **designates an Agent to search public channels or prompts the player to find someone**.
- Different TSCs will **autopoietically grow domain-specific professions** on top of this base set: a trading TSC grows "Risk Controller" and "Market Intake Officer"; a community TSC (like HerName.AI) grows "Member Activity Sentinel" and "Emotional Health Advisor"; a content TSC grows "Topic Prophet" and "Viral-hit Distiller." The base set is constant; domain professions grow as needed.

---

# Part V: Genesis Checklist—What Must Be Told to the System Before Creating a TSC

The Law requires: **Matters that must be decided when creating a TSC must be told to the system in advance.** Based on this, the system calculates gaps and then decides whether to "send an Agent to fill it" or "tell the player to find someone." This is the "character creation interface" for this game.

## The Seven Questions (All are indispensable; the system refuses to start if any are missing)

1. **Declaration of Existence**: Why does this TSC exist? What is the bottom line it would rather die than cross? → Written into the Genesis layer (Law I), immutable forever.
2. **Initial Goal**: What is to be achieved now? → Evolvable (goal autopoiesis), but must have a starting point.
3. **Player**: Who holds the Soul? One person or several? What are the boundaries of each person's irrevocable authority?
4. **Environment**: In what world does it operate (market / community / content / trading / government-enterprise…)? → Determines which collectors/sensors are needed and which domain professions will grow.
5. **Judgment Codex Seed**: Within this TSC, what do "good / credible / worth mobilizing" mean respectively? → Initial scoring dimensions and trust dimensions (Law VII, IX).
6. **Resource Boundaries**: Model budget, player attention budget? → Determines which professions use top-tier Agents, which use cheap Agents, and which must conserve human involvement.
7. **Expected Member Types**: Which NPCs will it interact with (which types of Agents + which types of humans)?

## What the System Does Automatically After the Seven Questions

```
Read the Seven Questions
 → Calculate whether all VSM essential professions have suitable holders (gap analysis)
 → For each gapped profession:
     · Can it be held by an Agent?      → Autopoiesis engine generates the agent (draft, pending player approval)
     · Must be human, and a suitable candidate is on the roster? → Nominate candidate + estimate mobilization cost
     · Must be human, and no suitable candidate is on the roster?   → Dispatch Recruiter to produce recruitment intent; or directly prompt the player: "You need to find someone who can do X"
 → Grow domain-specific profession slots corresponding to the environment (initially empty, filled as needed)
 → Initialize the Soul: write genesis, judgment_codex seed, empty event_stream, empty calibration_memory
 → Start the Shell: mount kernel + essential modules
```

This step transforms "member mechanism autopoiesis" from an abstract principle into the first concrete action when creating an organization. The system knows what it lacks from the very first moment of its birth and proactively works to fill the gaps.

---

# Part VI: Lifecycle—Health, Death, Resurrection

## Health Indicators (Is the system living well?)

- **S5 is rarely triggered** = Judgment standards are stable, values haven't drifted (the Genesis layer being invisible in daily operations is a sign of health).
- **Calibration hit rate increases over time** = The Soul is truly learning, not becoming an echo chamber.
- **The judgment codex is iterating** = Judgment standards are evolving, not ossifying.
- **The Briefer can proactively surface dissenting items** = Has not degenerated into a machine that merely pleases the player.

## Death Conditions (System Failure)

- Calibration hit rate stops updating, predictions are no longer backfilled → perception and judgment become disconnected, degrading into a labeled folder.
- The judgment codex hasn't been touched in a year → judgment has petrified.
- The system only says what players want to hear → manipulated by player preferences, reduced to an expensive echo chamber (Law 8 breached).
- The Genesis layer is arbitrarily tampered with → identity continuity is broken; it is already a different TSC.

## Resurrection (Fulfillment of Law 2)

The Shell is broken, needs rewriting, requires a model change, or must be migrated—the Soul is unaffected. Resurrection process:

```
Export Soul = {genesis, judgment_codex, event_stream, calibration_memory}
 → Verify the contractual integrity of the Soul (Can the current state be derived from the event stream? Is the calibration ledger continuous?)
 → Initialize a brand-new Shell with the Soul (which can be a completely different software implementation)
 → The Shell rebuilds according to the Soul: recalculate all attributes from the event stream, restore judgment from calibration memory, restore evaluation standards from the judgment codex
 → Continue operating under the same tsc_id
```

As long as the Genesis layer remains unchanged, the event stream is continuous, and the calibration ledger is complete, the resurrected TSC is the same TSC. This is the commitment of the "Soul/Shell" architecture to organizational perpetuity: **The body can be rebuilt again and again; as long as the Soul is not lost, the organization does not die.**

---

# Appendix: Conceptual Increments of 3.5 over 3.0

| Increment | Origin | In One Sentence |
|---|---|---|
| Soul/Shell Separation (Law 2) | OpenTSC practice + SoulShell | Law and memory can survive and be resurrected independent of software |
| Dual Codex (Law 7) | OpenTSC hitting a wall | A judgment codex supplements the rule codex |
| Attributes Derived from Events (Law 9) | Generative agent reflection | Judgments are reproducible; conclusions remain consistent across model changes |
| Profession/Player/NPC Ontology | RPG systems + generative agents | Enables both humans and AI to understand "who is who and what they can do" |
| Advanced Attribute Architecture (Part 3) | RPG + probabilistic modeling | Attributes are probabilistic statements with evidence, decay, predictability, and fractal nature |
| Perception Layer as a First-Class Citizen | OpenTSC event stream | The first condition of autopoiesis, "observable," finally has an organ |
| Genesis Checklist (Part 5) | Member autopoiesis implementation | Creation is gap analysis; the system knows from birth what it lacks and what to supplement |

> This Law defines the Soul. For how to build it into a Shell that can run, scale, and autonomously generate agents—see the companion document, *The Shell*.
