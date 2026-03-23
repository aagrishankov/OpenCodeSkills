---
name: skill-creator
description: Create or refactor OpenCode skills into a strict, testable, example-rich template. Trigger when the user asks to build, standardize, audit, improve, adapt, or rewrite a skill package, SKILL.md structure, trigger behavior, workflow contract, validation gates, or example coverage.
metadata:
  short-description: Create or improve skills
  version: 2.0.0
  owner: opencode
  updated_at: 2026-03-23
---

# Skill Creator (Template-Enforced, Example-Driven)

Create and improve skills with a single standardized template so different skills stay consistent, testable, and easy to maintain.

This skill is blocking by default: incomplete or weak outputs must be corrected before final delivery.

## Non-Negotiable Rules

1. Every created or updated skill MUST contain `SKILL.md`.
2. `SKILL.md` MUST follow the exact section order from `Canonical SKILL.md Template`.
3. Trigger behavior MUST be defined in frontmatter `description` and validated with positive and negative prompts.
4. Example coverage is mandatory. A skill without practical examples is incomplete.
5. Use optional folders only when they have clear value:
   - `references/` for long domain material and example libraries
   - `assets/` for reusable output templates
   - `scripts/` for repeated deterministic tasks
6. Favor practical defaults and fallback paths over brittle branching.
7. If a rule cannot be validated, rewrite it into a testable rule.

---

## Standard Skill Package

Use this default folder structure:

```text
<skill-name>/
├── SKILL.md                     # required
├── references/README.md         # required index for support docs
├── references/examples.md       # required example library
├── assets/templates.md          # required only when output shape is structured
└── scripts/                     # optional (repeated deterministic tasks only)
```

Rules:

- `SKILL.md` is mandatory.
- `references/README.md` is mandatory when `references/` exists.
- `references/examples.md` is mandatory for this skill workflow.
- `assets/templates.md` is required only when the skill emits repeatable formats.
- `scripts/` is optional and must not be added for one-off operations.

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
- [ ] Example requirements are satisfied

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
Example obligations (must include):
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

## Example Requirements (BLOCKING)

The final skill package is invalid unless all example requirements pass.

Minimum required examples per created/refactored skill:

1. One complete happy-path example (input -> process -> final output).
2. One bad example with explanation of why it violates rules.
3. One fixed version of the bad example.
4. One edge-case example with explicit fallback behavior.
5. One trigger boundary example (looks similar, should not trigger).

Placement rules:

- Keep `SKILL.md` focused and short; place long examples in `references/examples.md`.
- `SKILL.md` must include short links to example sections and expected usage.
- Example snippets must be executable or directly reusable.

---

## Anti-Patterns (Reject These Outputs)

Reject and regenerate if output has any of these:

1. Template-only result with placeholders and no concrete filled examples.
2. Rules that cannot be tested or verified.
3. Trigger contract missing negative examples.
4. Vague workflow like "analyze then produce" without concrete steps.
5. Output contract with no acceptance criteria.
6. Large instruction wall in `SKILL.md` with no `references/examples.md` split.

---

## Quality Gates (Blocking)

A skill is not complete until all gates pass:

1. Structure gate: canonical section order is exact.
2. Trigger gate: description is specific and matrix is present.
3. Execution gate: workflow is actionable with available tools.
4. Output gate: deliverables and acceptance criteria are concrete.
5. Safety gate: prohibited behavior is explicit.
6. Example gate: minimum example obligations are fully present.
7. Consistency gate: no conflicting rules across sections/resources.

If any gate fails, fix before final delivery.

---

## Workflow

### 1) Capture Intent

Extract outcome, trigger scenarios, constraints, failure modes, and expected example depth.

### 2) Produce Intake Brief

Fill `Input Intake Template` with concrete values and assumptions.

### 3) Draft Trigger Contract

Write frontmatter `description` with explicit action + context + adjacent phrasing.

### 4) Scaffold Canonical SKILL.md

Create all required sections in the exact order and fill defaults where needed.

### 5) Build Example Library

Create or update `references/examples.md` with all required example types.

### 6) Add Support Resources (Only If Justified)

Add `assets/` and `scripts/` only if they reduce repeated effort or improve reliability.

### 7) Run Trigger Matrix Check

Validate should-trigger/should-not-trigger behavior and refine wording.

### 8) Run Quality Gates

Pass all blocking gates before final output.

### 9) Record Iteration Notes

Update `Iteration Log` with observed issue -> change -> expected impact.

---

## Updating Existing Skills

When refactoring an existing skill:

- Preserve `name` unless a rename is explicitly requested.
- Keep effective existing instructions; standardize structure around them.
- Remove or rewrite ambiguous rules that cannot be tested.
- Add missing example coverage until `Example Requirements` pass.
- Normalize the skill to the canonical template without changing user intent.

---

## Safety Boundaries

Do not create or improve skills for malware, unauthorized access, stealth abuse, fraud, or deception.

Allow defensive security and compliance automation only with explicit benign intent and clear constraints.

---

## Output Contract

When asked to create or update a skill, deliver:

1. Final folder structure.
2. Final `SKILL.md` using the canonical template.
3. `references/README.md` and `references/examples.md` with required examples.
4. Optional `assets/templates.md` and `scripts/` only when justified.
5. Trigger matrix (or test prompts section) with positive, negative, and edge examples.
6. Short rationale explaining key design decisions, what was standardized, and which anti-patterns were removed.
