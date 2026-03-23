---
name: skill-creator
description: Create or refactor OpenCode skills into a strict, repeatable template. Use when a user asks to build, standardize, adapt, audit, or improve a skill workflow, trigger behavior, or SKILL.md structure.
metadata:
  short-description: Create or improve skills
---

# Skill Creator (Template-Enforced)

Create and improve skills with a single standardized template so different skills stay consistent, testable, and easy to maintain.

## Non-Negotiable Rules

1. Every created or updated skill MUST contain `SKILL.md`.
2. `SKILL.md` MUST follow the exact section order from `Canonical SKILL.md Template`.
3. Trigger behavior MUST be defined in frontmatter `description` and validated with positive and negative prompts.
4. Use optional folders only when they have clear value:
   - `references/` for long domain material
   - `assets/` for reusable output templates
   - `scripts/` for repeated deterministic tasks
5. Keep guidance compact; move long examples to `references/`.
6. Favor practical defaults and fallback paths over brittle branching.

---

## Standard Skill Package

Use this default folder structure:

```text
<skill-name>/
├── SKILL.md                 # required
├── references/README.md     # recommended baseline stub
├── assets/templates.md      # recommended if output format is structured
└── scripts/                 # optional (only if repeated deterministic tasks exist)
```

Rules:

- `SKILL.md` is mandatory.
- `references/README.md` is recommended as a lightweight index, even if short.
- `assets/templates.md` is recommended when the skill must emit repeatable output shapes.
- `scripts/` is optional and should not be added for one-off operations.

---

## Canonical SKILL.md Template

Use this exact structure and section order:

```md
---
name: <kebab-case-skill-name>
description: <what it does + when it should trigger + adjacent trigger cases>
metadata:
  short-description: <short summary>
  version: <semver-like, e.g. 1.0.0>
  owner: <team-or-person>
  updated_at: <YYYY-MM-DD>
---

# <Skill Title>

## 1) Purpose
- Goal:
- Non-goals:

## 2) Trigger Contract (MUST)
- Primary triggers:
- Secondary/indirect triggers:
- Must-not-trigger cases:

## 3) Inputs Required
- Required inputs:
- Optional inputs:
- Missing-input fallback:

## 4) Output Contract (MUST)
- Required deliverables:
- Output format:
- Acceptance criteria:

## 5) Workflow (Step-by-step)
1. ...
2. ...
3. ...

## 6) Decision Rules
- If X -> do A
- If Y -> do B
- Default path:

## 7) Resource Usage
- Use `references/...` for ...
- Use `assets/...` for ...
- Use `scripts/...` for ...

## 8) Error Handling and Fallbacks
- Common failure:
- Recovery action:
- Hard-stop conditions:

## 9) Safety Boundaries (MUST/NEVER)
- MUST:
- NEVER:

## 10) Validation Checklist (BLOCKING)
- [ ] Frontmatter complete and valid
- [ ] Trigger contract includes should-trigger and should-not-trigger examples
- [ ] Output contract is testable and unambiguous
- [ ] Workflow is executable with available tools
- [ ] Safety boundaries are explicit

## 11) Test Prompts
- Should trigger:
- Should not trigger:
- Edge cases:

## 12) Iteration Log
- v<version>:
  - Observed issue:
  - Change made:
  - Expected impact:
```

Do not skip sections. If unknown, fill with explicit defaults.

---

## Input Intake Template (Required Before Writing)

Collect this brief before creating or refactoring a skill:

```text
Skill name:
Primary outcome:
Trigger contexts (direct wording):
Trigger contexts (indirect wording):
Must-not-trigger contexts:
Expected output format:
Hard constraints (tools/policy/style):
Known failure modes:
```

If data is missing, infer safe defaults from the conversation and note assumptions.

---

## Trigger Matrix Template (Required)

For each skill, define at least:

- 3 should-trigger prompts
- 3 should-not-trigger prompts
- 2 edge prompts with expected behavior

Use this format:

```text
[Should Trigger]
1) <prompt>
2) <prompt>
3) <prompt>

[Should Not Trigger]
1) <prompt>
2) <prompt>
3) <prompt>

[Edge]
1) <prompt> -> <expected handling>
2) <prompt> -> <expected handling>
```

---

## Quality Gates (Blocking)

A skill is not complete until all gates pass:

1. Structure gate: canonical section order is exact.
2. Trigger gate: description is specific and matrix is present.
3. Execution gate: workflow is actionable with available tools.
4. Output gate: deliverables and acceptance criteria are concrete.
5. Safety gate: prohibited behavior is explicit.
6. Consistency gate: no conflicting rules across sections/resources.

If any gate fails, fix before final delivery.

---

## Workflow

### 1) Capture Intent

Extract outcome, trigger scenarios, constraints, and failure modes from the conversation.

### 2) Produce Intake Brief

Fill `Input Intake Template` with concrete values and assumptions.

### 3) Draft Trigger Contract

Write frontmatter `description` with explicit action + context + adjacent phrasing.

### 4) Scaffold Canonical SKILL.md

Create all required sections in the exact order and fill defaults where needed.

### 5) Add Support Resources (Only If Justified)

Add `references/`, `assets/`, and `scripts/` only if they reduce repeated effort or improve reliability.

### 6) Run Trigger Matrix Check

Validate should-trigger/should-not-trigger behavior and refine wording.

### 7) Run Quality Gates

Pass all blocking gates before final output.

### 8) Record Iteration Notes

Update `Iteration Log` with observed issue -> change -> expected impact.

---

## Updating Existing Skills

When refactoring an existing skill:

- Preserve `name` unless a rename is explicitly requested.
- Keep effective existing instructions; standardize structure around them.
- Remove or rewrite ambiguous rules that cannot be tested.
- Normalize the skill to the canonical template without changing user intent.

---

## Safety Boundaries

Do not create or improve skills for malware, unauthorized access, stealth abuse, fraud, or deception.

Allow defensive security and compliance automation only with explicit benign intent and clear constraints.

---

## Output Contract

When asked to create or update a skill, deliver:

1. Final folder structure
2. Final `SKILL.md` using the canonical template
3. Optional `references/assets/scripts` files only when justified
4. Trigger matrix (or test prompts section) with positive/negative examples
5. Short rationale explaining key design decisions and what was standardized
