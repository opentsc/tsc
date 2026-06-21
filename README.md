<div align="center">

# TSC — The Thin-Shell Company

**A whitepaper on how one person can run a large, self-evolving organization with AI agents — and keep its judgment and memory alive no matter which software, team, or model happens to run it.**

🌐 **English** · [中文](README.zh-CN.md)

by **dashen** (「AI 最严厉的父亲」) · [dashen.wang](https://dashen.wang) · [@dashen_wang](https://x.com/dashen_wang)

[![License: CC BY-SA 4.0](https://img.shields.io/badge/whitepaper-CC%20BY--SA%204.0-lightgrey.svg)](LICENSE) · ![version](https://img.shields.io/badge/version-v3.5-blue.svg) · [Changelog](CHANGELOG.md) · [Implementation → OpenTSC](https://github.com/opentsc/opentsc)

</div>

---

## What is TSC?

TSC (the **Thin-Shell Company**) is not software — it's a set of ideas, almost a constitution, for a new kind of organization: **thin on people and tooling, thick on portable judgment.**

The bet behind it: AI now lets *one person drive an organization that used to need a hundred.* But there's a catch — if that organization's judgment lives only in the founder's head, and its memory lives only in one piece of software, then the moment the founder burns out or the software dies, the whole thing dies with it.

TSC's answer is one core idea:

> **Separate the "soul" from the "shell."**
> The **soul** = the organization's laws + everything it has learned + its memory of what worked. Abstract, portable.
> The **shell** = whatever runs the soul today — the apps, the agents, the team, the AI models. Replaceable.
> Keep them separate, and you can rewrite the software, swap the team, or upgrade the model — and it's still the *same* organization. Export the soul, and you've moved the whole thing.

It's the [Ship of Theseus](https://en.wikipedia.org/wiki/Ship_of_Theseus) problem, answered at the level of an organization.

## Who is this for?

- **Founders and solo operators** who want serious leverage from AI without becoming a single point of failure.
- **People building (or thinking about) AI-agent organizations** — multi-agent systems that need to be governed, not just wired together.
- **Systems and org-design thinkers** interested in autopoiesis, cybernetics (VSM), and self-evolving institutions.

You don't need to code to get value from it — it's a way of *thinking* about organizations. The code that runs it is a separate project, [OpenTSC](https://github.com/opentsc/opentsc).

## What's inside

| Version | What it covers | Read |
|---|---|---|
| **v3.5 · The Soul** | The latest. The twelve laws, the judgment codex, soul/shell separation. | [TSC-3.5-白皮书-魂.md](whitepaper/TSC-3.5-白皮书-魂.md) |
| v3.0 | Autopoiesis · the Viable System Model · DAO integration · recursion | [v3.0/](whitepaper/v3.0/) |
| v2.0 | Organizational architecture | [v2.0/](whitepaper/v2.0/) |
| v1.0 | The founding case | [v1.0/](whitepaper/v1.0/) |
| The Shell | OpenTSC design & implementation rationale | [OpenTSC-v1.0-壳…md](whitepaper/OpenTSC-v1.0-壳-设计理念与技术实现.md) |

> The whitepapers are being translated to English — see [`whitepaper/en/`](whitepaper/en/).

## The four ideas it's built on

1. **Genesis is fixed.** Why the organization exists is set once and never edited — everything else can change, this can't.
2. **Judgment comes from evidence, not vibes.** Every claim carries a source and a confidence; predictions carry a due date and get checked against reality.
3. **It grows only where it's proven it needs to.** New rules, agents, and goals can self-generate — but only through a governed process, never sprawl.
4. **The same pattern repeats at every scale.** A person, a team, a company, an ecosystem all run on the same cycle and the same vocabulary.

## From idea to running system

This repo is the **soul** (the doctrine). The **shell** that actually runs it — a local, offline-first tool you talk to through an AI agent — is **[OpenTSC](https://github.com/opentsc/opentsc)**.

## License

This whitepaper is published under [CC BY-SA 4.0](LICENSE): share and adapt freely, but **credit dashen (dashen.wang)** and keep derivatives under the same license. "TSC" / "Thin-Shell Company" are trademarks. To cite, see [CITATION.cff](CITATION.cff).

---

<div align="center">
TSC · by dashen「AI 最严厉的父亲」· <a href="https://dashen.wang">dashen.wang</a> · <a href="https://x.com/dashen_wang">@dashen_wang</a>
</div>
