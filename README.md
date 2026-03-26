# Atelier

Spec refinement through adversarial iteration — find and eliminate blind spots in any plan, spec, or architecture.

## How it works

```
Round N:
  Phase 1: BUILD  → Create/update spec (ask user if unclear)
  Phase 2: BREAK  → Spawn breaker agent (isolated context)
  Phase 3: JUDGE  → Classify gaps by severity
  Phase 4: ASK    → Question user about critical/important gaps
  → Loop to Round N+1

Exit when: zero critical and zero important gaps remain
```

The core principle: **the creator and the destroyer must be separate.** The refine skill builds the spec. The breaker agent attacks it with fresh context, free from confirmation bias.

## Components

| Type | Name | Purpose |
|------|------|---------|
| Skill | `refine` | Orchestrates the BUILD → BREAK → ASK loop |
| Agent | `breaker` | Adversarial reviewer — sees only the spec, no conversation history |

## Usage

### Standard mode

Every gap is presented to the user as a question.

```
/atelier:refine [topic or file path]
```

### Autonomous mode

AI resolves implementation-level gaps silently. Only asks user for intent/direction decisions.

```
/atelier:refine --auto [topic or file path]
```

### Examples

```
/atelier:refine API authentication system
/atelier:refine --auto ./docs/api-spec.md
/atelier:refine
```

## Output

| File | Purpose |
|------|---------|
| `./docs/{topic}-refine.md` | Final refined spec |
| `./docs/{topic}-refine-log.tsv` | Round-by-round history (gaps found, resolution method) |

The TSV log tracks each round's gap counts and how they were resolved (by AI or by user), useful for analyzing blind spot patterns over time.

## Installation

```
/plugin marketplace add unhyif/atelier
/plugin install atelier@atelier
```

## License

MIT
