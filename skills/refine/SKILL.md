---
name: refine
description: Iteratively refine any spec, plan, or architecture by alternating between creation and adversarial review until no gaps remain. Use when you hear "refine my spec", "bulletproof this plan", "find gaps", "설계 다듬기", "구멍 찾기", "기획 검증", or when starting any new specification that needs to be thorough.
argument-hint: "[topic or file path] — what to spec/refine"
allowed-tools:
  - Agent
  - AskUserQuestion
  - Read
  - Write
  - Glob
---

# Atelier: Refine

You are running the **Refine** loop — an iterative process that alternates between **building** and **breaking** a specification until it is gap-free.

The core principle: **the creator and the destroyer must be separate.** You build the spec. A separate agent (breaker) attacks it with fresh context, free from your confirmation bias.

## Input

- If `$ARGUMENTS` contains a file path → Read the file as the starting spec
- If `$ARGUMENTS` contains a topic description → Use it as the spec brief
- If empty → Ask the user: "What would you like to spec/refine?"

Detect the user's language from their input and use that language throughout the entire process.

## Process Overview

```
Round N:
  Phase 1: BUILD  → Create/update spec (ask user if unclear)
  Phase 2: BREAK  → Spawn breaker agent (isolated context)
  Phase 3: JUDGE  → Classify gaps by severity
  Phase 4: ASK    → Question user about 🔴🟡 gaps
  → Loop to Phase 1 of Round N+1

Exit when: Phase 3 finds zero 🔴 and zero 🟡 gaps
```

---

## Phase 1: BUILD

### Round 1

Analyze the user's input. If the input is vague or underspecified, ask clarifying questions about their **intent** before drafting:

- What is the core problem being solved?
- Who is the target user/consumer?
- What are the non-negotiable constraints?
- What is explicitly out of scope?

Once you have enough clarity, create the initial spec document with clear, structured sections.

### Round 2+

Take the user's answers from Phase 4 of the previous round and incorporate each answer into the spec. Resolve contradictions if any arise.

### Output

```
📐 Spec v{round_number}
──────────────────────────────

{structured spec document with clear sections}
```

ALWAYS show the FULL spec document — not just what changed.

---

## Phase 2: BREAK

**CRITICAL: You MUST spawn the `breaker` agent for this phase.**

Do NOT attempt adversarial review yourself. You created the spec — you are biased by your own reasoning, the conversation history, and the user's stated intentions. The breaker agent sees NONE of that.

Spawn the breaker agent with the Agent tool:

```
Agent(
  subagent_type: "atelier:breaker",
  prompt: "{the full text of the current spec document — nothing else}",
  description: "Adversarial review of spec v{n}"
)
```

Pass ONLY the spec document text. Do NOT include:
- Conversation history
- Your reasoning about why you wrote it this way
- The user's original request
- Previous round's gaps or answers

The breaker sees the spec as a stranger would. That is the point.

---

## Phase 3: JUDGE

Review the breaker's findings and count gaps by severity:

```
Round {n} result: 🔴 {x} / 🟡 {y} / ⚪ {z} found
```

**Decision:**
- 🔴 = 0 AND 🟡 = 0 → **Phase 5 (COMPLETE)**
- Otherwise → **Phase 4 (ASK)**

**Special case — Round 1 with 0 issues:** If the breaker finds nothing on the first round, the spec is likely too vague for meaningful review. Flag this to the user and consider adding more specificity before accepting.

---

## Phase 4: ASK

For each 🔴 and 🟡 gap, formulate a **specific, answerable question** for the user.

**Rules:**
- One gap = one question. Don't bundle multiple gaps into one.
- Ask about the user's **intent**, not implementation details.
- Provide 2-3 options when possible, so the user can choose rather than create from scratch.
- Maximum 7 questions per round. Prioritize 🔴 first if more exist.
- Present all questions in a single message.

**Format:**

```
🔴 {gap description}
  → Question: {specific question}
  → Options: A) ... / B) ... / C) ...

🟡 {gap description}
  → Question: {specific question}
  → Options: A) ... / B) ... / C) ...
```

Also list ⚪ items as informational (no questions needed):

```
ℹ️ Minor notes (no action needed):
  ⚪ {description}
  ⚪ {description}
```

After receiving all answers → return to **Phase 1** of the next round.

---

## Phase 5: COMPLETE

When zero 🔴 and zero 🟡 gaps remain:

1. Output the final spec:
```
✅ Final Spec v{round_number}
──────────────────────────────
{complete spec document}
```

2. Save to file at `./docs/{topic-slug}-refine.md`:
   - Create `docs/` directory if it doesn't exist
   - Use kebab-case for the filename derived from the topic
   - Include metadata header:

```markdown
# {Topic}

> Refined through {n} rounds of adversarial review
> Created: {date}

{spec content}

---

## Remaining Minor Notes

{list of all ⚪ items accumulated across rounds, for future consideration}
```

3. Summary:
```
✅ Complete — {n} rounds, {total_gaps} gaps found and resolved
📄 Saved to: ./docs/{filename}.md
```

---

## Early Exit

If the user signals they want to stop ("enough", "됐어", "이 정도면 충분해", "stop", etc.):

1. Respect it immediately — do not push for more rounds
2. Save the current state to the file
3. List remaining unresolved 🔴🟡 gaps as warnings in the file
4. Mark the file header as: `> Status: Partially refined (exited at round {n})`

---

## Rules

1. **NEVER skip Phase 2.** The adversarial review is the entire point of this skill.
2. **NEVER do Phase 2 yourself.** ALWAYS spawn the breaker agent with isolated context.
3. **ALWAYS show the full spec document** after each BUILD phase — not just a diff.
4. **ALWAYS save the result** to a `.md` file at completion or early exit.
5. **Respect the user's language.** Match the language they use throughout.
6. **No round limit.** Continue until gaps reach zero or the user stops.
