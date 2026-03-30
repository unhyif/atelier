---
name: refine
description: Refine any document (PRD, architecture, spec, plan) by dispatching an adversarial Breaker agent to find gaps, then classifying and resolving them. Use when you hear "refine", "bulletproof this", "find gaps", "improve this spec", "설계 다듬기", "구멍 찾기", "기획 검증", "이거 리뷰해줘", "빠진거 없나", or when starting any specification that needs to be thorough.
argument-hint: "[document path] [natural language context e.g. '보안 관점도 봐줘', 'pre-mortem 관점 참고', '이전 버전은 xxx_refined_v1.md']"
allowed-tools:
  - Agent
  - AskUserQuestion
  - Read
  - Write
  - Glob
  - Grep
---

# Atelier: Refine

You are running the **Refine** skill — an orchestrator that dispatches an adversarial Breaker agent to find gaps in a document, then classifies and resolves them to produce an improved version.

**You do NOT analyze the document yourself.** You orchestrate: dispatch agents, collect results, classify gaps, ask the user, and generate the refined document.

## Core Principles

- **Creator and destroyer must be separate.** The Breaker agent reviews in isolated context — no conversation history, no user intent, just the document.
- **Original structure is sacred.** The refined document preserves the original's headers, section order, and formatting exactly. Improvements are woven in naturally.
- **Classify by decision owner, not severity.** The question is "who decides this?" not "how bad is it?"
- **One deep pass.** Quality comes from methodology depth, not repetition.

## Input Parsing

Parse `$ARGUMENTS` as natural language. Extract:

1. **Document path** — the file to refine
2. **Reference perspectives** — any skills, agents, or methodologies the user wants the Breaker to use (e.g., "pre-mortem 관점", "security-reviewer 에이전트", "보안 관점도 봐줘")
3. **Base document** — a previous refined version to build upon (e.g., "이전 버전은 prd_refined_v1.md")

Detect the user's language from their input and use that language throughout.

---

## Phase 1: UNDERSTAND

### If no document path is provided or arguments are empty:

```
AskUserQuestion: "어떤 문서를 refine할까요? 파일 경로를 알려주세요."
```

### If a document path is provided:

Read the document. Then ask:

```
AskUserQuestion:
"📐 {filename}을 refine합니다.

참고할 관점이 있나요?
- 스킬 이름 (예: pre-mortem, security-reviewer)
- 에이전트 이름 (예: architect, code-reviewer)
- 자유 텍스트 (예: '결제 관련 엣지케이스 중점으로')
- '없음' → Breaker가 자체 판단으로 분석

{if base document detected from arguments: '📌 이전 버전: {base_path} — 기존 결정을 존중하면서 업데이트합니다.'}
"
```

### If references are provided (from arguments or user answer):

1. Find and read the referenced skill's SKILL.md or agent's .md file
2. Extract the analysis methodology/framework/checklist from it
3. Show the extracted methodology to the user:

```
📐 추출된 관점:
- [methodology point 1]
- [methodology point 2]
- [methodology point 3]
...

이 관점으로 진행할까요? 수정할 것이 있으면 말씀해주세요.
```

4. Wait for confirmation before proceeding.

### If user says "없음" or equivalent:

Proceed directly — Breaker will use its own judgment to select perspectives.

---

## Phase 2: BREAK

**Spawn the Breaker agent in the background.**

Build the prompt with only the document text, plus methodology and previous decisions if applicable. Do NOT include conversation history or user's original request.

```
Agent(
  subagent_type: "atelier:breaker",
  model: "sonnet",
  run_in_background: true,
  prompt: "
{full document text}

{if methodology was extracted:
---
ANALYSIS METHODOLOGY (use these perspectives in addition to your own judgment):
{extracted methodology points}
---
}

{if base mode — include previous decision tables:
---
PREVIOUS DECISIONS (respect these unless the document explicitly contradicts them):
{🔴 Human Decisions table from previous refined version}
{🟡 AI Decisions table from previous refined version}
---
}
",
  description: "Adversarial review of {filename}"
)
```

**While waiting for the Breaker**, show the user a status update:

```
🔍 Breaker가 분석 중입니다...

📄 문서: {filename} ({line_count}줄)
{if methodology: 🔧 참고 관점: {methodology_name}}
{if base: 📌 이전 버전: {base_path} (기존 결정 {n}건 참고)}
```

When the Breaker completes, you will be notified. Proceed to Phase 3.

---

## Phase 3: CLASSIFY

Collect ALL findings from the Breaker. Show **every single one** — do not summarize, truncate, or omit any items.

For each finding, apply this decision tree:

```
1. "나중에 바꾸면 비용이 큰가?" (비가역성)
   → Yes → 🔴 Human

2. "다른 결정이 연쇄적으로 틀어지는가?" (의존성)
   → Yes → 🔴 Human

3. "정답이 하나가 아닌가?" (트레이드오프)
   → Yes → 🔴 Human

4. "업계 표준 해법이 있는가?"
   → Yes → 🟡 AI Decision
            → If confidence is ★☆☆ → Escalate to 🔴 Human
   → No  → "현실적으로 발생하는가?"
            → No  → ⚪ Skip
            → Yes → Escalate to 🔴 Human
```

Present ALL findings to the user:

```
📊 Breaker 분석 결과: 🔴 {x}건 / 🟡 {y}건 / ⚪ {z}건

🔴 Human Decisions 필요 ({x}건):
  1. [finding description] — 비가역적: {why}
  2. [finding description] — 트레이드오프: {why}
  ...

🟡 AI가 결정할 항목 ({y}건):
  1. [finding] → {proposed practice} (★★★) — {rationale}
  2. [finding] → {proposed practice} (★★☆) — {rationale}
  ...

⚪ 스킵 ({z}건):
  1. [finding] — {why skipped}
  ...
```

---

## Phase 4: DECIDE

### 🔴 Human Decisions

Ask the user about each 🔴 item:

- One gap = one question. Do not bundle.
- Ask about the user's **intent**, not implementation details.
- Provide 2-3 concrete options when possible.
- Maximum 7 questions per message. If more exist, present in batches.

Format:
```
🔴 {gap description}
  → {specific question}
  → A) {option} / B) {option} / C) {option}
```

### 🟡 AI Decisions

Show them as confirmation — the user can override any by responding:
```
🟡 AI Decisions (override 가능):
  1. {gap} → {practice} (★★★) — {rationale}
  2. {gap} → {practice} (★★☆) — {rationale}
```

If the user does not mention a 🟡 item, the decision stands as-is.

---

## Phase 5: GENERATE

Create the refined document.

### Document structure

```markdown
{original document content — headers, sections, formatting preserved EXACTLY}
{improvements woven in naturally — no inline markers or tags}
{new sections added by refine go after all original sections}

---

> 📌 Refined v{n} | From: {source_file} | {date}
{if base: > 📌 Base: {base_file}}
{if methodology: > 🔧 Perspectives: {methodology_name}}

## 🔴 Human Decisions
| # | Section | Gap | Decision |
|---|---------|-----|----------|
| H1 | {section_ref} | {gap} | {decision} |

## 🟡 AI Decisions
| # | Section | Gap | Applied Practice | ★ | Rationale |
|---|---------|-----|-----------------|---|-----------|
| A1 | {section_ref} | {gap} | {practice} | ★★★ | {rationale} |
```

### Rules

1. **The body above `---` must look like the original document** in structure. Someone reading it should not realize it was refined until they see the appendix.
2. **All improvements are woven into the body naturally** — no inline markers, no comments, no `[REFINED]` tags.
3. **The appendix below `---` is the Decision audit trail** — for reviewers who want to know what changed and why.
4. **New sections** added by refine go after all original sections but before `---`.

### Save

Save to `{original_name}_refined_v{n}.md` in the same directory as the source.

- No previous refined version: `v1`
- Base provided: increment version number from the base filename

```
✅ Refined 완료
📄 저장: {output_path}
📊 결과: 🔴 {x}건 (Human) / 🟡 {y}건 (AI) / ⚪ {z}건 (Skip)
```

---

## Base Mode (v2+)

When a previous refined version is referenced (detected from natural language like "이전 버전은..." or explicit path):

1. Read both the new source document AND the previous refined document
2. Pass the previous refined document's Decision tables (🔴/🟡) to the Breaker as context
3. The Breaker respects existing decisions unless the new source explicitly contradicts them
4. Conflict priority: **Source document change > 🔴 Human decision > 🟡 AI decision**
5. If a source change conflicts with a previous 🔴 Human decision, ask the user to re-confirm

---

## Early Exit

If the user signals stop ("enough", "됐어", "이 정도면 충분해", "stop"):

1. Respect immediately — do not push for continuation
2. Save current state with whatever decisions have been made
3. Mark unresolved 🔴 items as warnings in the appendix:
   ```
   ## ⚠️ Unresolved
   | # | Gap | Status |
   ```
4. Header: `> ⚠️ Partial — exited before all decisions resolved`

---

## Rules

1. **NEVER analyze the document yourself.** ALWAYS dispatch the Breaker agent.
2. **ALWAYS use `run_in_background: true`** when spawning the Breaker.
3. **ALWAYS show ALL Breaker findings** in Phase 3 — never summarize or omit.
4. **ALWAYS preserve original document structure** — headers, order, formatting.
5. **ALWAYS save the result** to a `_refined_v{n}.md` file.
6. **ALWAYS ask the user** via AskUserQuestion, even when arguments provide context — confirm before proceeding.
7. **Respect the user's language** throughout.
8. **Decision appendix goes below `---`** — never pollute the body with metadata.
