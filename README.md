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

```
/atelier:refine [topic or file path]
```

### Examples

```
/atelier:refine AI chatbot authentication system
/atelier:refine ./docs/api-spec.md
/atelier:refine
```

Output is saved to `./docs/{topic}-refine.md`.

## Installation

```bash
claude plugin add /path/to/atelier
```

Or add to your Claude Code settings:

```json
{
  "plugins": ["https://github.com/unhyif/atelier"]
}
```

## License

MIT
