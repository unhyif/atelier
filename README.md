# Atelier

Document refinement through adversarial review — find and eliminate blind spots in any PRD, architecture, spec, or plan.

## How it works

```
Phase 1: UNDERSTAND  → Parse input, ask for reference perspectives
Phase 2: BREAK       → Spawn Breaker agent (isolated, reads file via Read tool)
Phase 3: CLASSIFY    → Show ALL findings, classify by decision owner
Phase 4: DECIDE      → 🔴 Human / 🟡 AI decisions (5-7 items per batch)
Phase 5: GENERATE    → Produce refined document preserving original structure
```

The core principle: **the creator and the destroyer must be separate.** The Breaker agent sees only the document — no conversation history, no user intent. It selects its own analysis perspectives based on the document content.

## Components

| Type | Name | Purpose |
|------|------|---------|
| Skill | `refine` | Orchestrator — dispatches agents, classifies gaps, generates refined document |
| Agent | `breaker` | Adversarial reviewer — quality review + gap finding in isolated context |

## Usage

```
/atelier:refine prd.md
/atelier:refine prd.md pre-mortem 관점도 참고해줘
/atelier:refine prd.md 이전 버전은 prd_refined_v1.md
/atelier:refine
```

### Reference perspectives

You can inject analysis methodologies from any installed skill or agent:

```
/atelier:refine spec.md security-reviewer 관점으로 봐줘
/atelier:refine plan.md pre-mortem이랑 architect 관점 참고
```

The Breaker extracts the methodology and uses it alongside its own judgment.

### Base mode (v2+)

Build on a previous refined version — existing decisions are respected:

```
/atelier:refine prd_2.md 이전 버전은 prd_refined_v1.md
```

Conflict priority: Source document change > 🔴 Human decision > 🟡 AI decision.

## Gap Classification

Gaps are classified by **decision owner**, not severity:

| Classification | Who decides | Criteria |
|---------------|-------------|----------|
| 🔴 Human | User | Irreversible, creates dependencies, or involves tradeoffs |
| 🟡 AI | AI (with rationale) | Industry best practice exists, confidence ★★☆+ |
| ⚪ Skip | Nobody | Unrealistic scenario, not worth addressing |

AI decisions include a confidence rating (★) and rationale. The user can override any 🟡 decision.

## Output

Saves to `{name}_refined_v{n}.md` — the original document structure preserved exactly, with improvements woven in naturally. A Decision appendix below `---` tracks all Human and AI decisions for audit.

```markdown
{original document — structure preserved, improvements integrated}

---

> 📌 Refined v1 | From: prd.md | 2026-03-31

## 🔴 Human Decisions
| # | Section | Gap | Decision |

## 🟡 AI Decisions
| # | Section | Gap | Applied Practice | ★ | Rationale |
```

## Installation

```
/plugin marketplace add unhyif/atelier
/plugin install atelier@atelier
```

## License

MIT
