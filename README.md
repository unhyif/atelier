# Atelier

Document refinement through adversarial review — find and eliminate blind spots in any PRD, architecture, spec, or plan.

## How it works

```
Phase 1: UNDERSTAND  → Parse input, expand broad perspectives into sub-perspectives
Phase 2: BREAK       → Spawn Breaker agent (or Self-Break fallback if Agent unavailable)
Phase 3: CLASSIFY    → Show ALL findings, classify by decision owner (🔴/🟡/⚪)
Phase 4: DECIDE      → 🔴 Human / 🟡 AI decisions (5-7 items per batch)
Phase 5: GENERATE    → Produce refined document (1.5x+ depth, original structure preserved)
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
/atelier:refine prd.md 보안이랑 장애 대응 관점도 봐줘
/atelier:refine prd.md 이전 버전은 prd_refined_v1.md
/atelier:refine
```

### Broad perspective expansion

Broad keywords like "보안", "성능", "장애 대응" are automatically expanded into concrete sub-perspectives:

```
"보안 봐줘"
  → 인증/인가, 입력 검증, rate limiting, 데이터 암호화, PII 보호, abuse prevention

"장애 대응 관점도"
  → retry 전략, circuit breaker, DLQ, failover, 모니터링/alerting, 롤백
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
| ⚪ Skip | Nobody | <1% probability in production (skip reason always stated) |

AI decisions include a confidence rating (★) and rationale. The user can override any 🟡 decision.

## Output

Saves to `{name}_refined_v{n}.md` — the original document structure preserved exactly, with improvements woven in naturally. Refined documents are at least 1.5x the original length, with vague statements replaced by concrete numbers. A Decision appendix below `---` tracks all Human and AI decisions for audit.

```markdown
{original document — structure preserved, improvements integrated}

---

> 📌 Refined v1 | From: prd.md | 2026-03-31

## 🔴 Human Decisions
| # | Section | Gap | Decision |

## 🟡 AI Decisions
| # | Section | Gap | Applied Practice | ★ | Rationale |
```

## Self-Break Fallback

When the Agent tool is unavailable (e.g., subagent contexts), the skill automatically switches to **Self-Break mode** — reading `agents/breaker.md` and internalizing the Breaker's adversarial mindset to perform analysis directly. All phases continue as normal.

## Installation

```
/plugin marketplace add unhyif/atelier
/plugin install atelier@atelier
```

## License

MIT
