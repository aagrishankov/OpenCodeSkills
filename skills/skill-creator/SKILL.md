---
name: skill-creator
description: Create and improve reusable skills for OpenCode-compatible agents: capture intent, design robust trigger descriptions, structure SKILL.md/resources, validate quality, and iterate from real feedback. Use whenever a user asks to build, adapt, refactor, or optimize a skill workflow.
metadata:
  short-description: Create or improve skills
---

# Skill Creator (OpenCode, model-agnostic)

Build and iterate skills that work across different models in OpenCode.

## Core Loop

1. Capture intent and real usage examples
2. Draft trigger description and skill structure
3. Write SKILL.md and only necessary resources
4. Validate clarity, triggerability, and efficiency
5. Iterate from observed failures and user feedback

---

## Operating Principles

- Assume the agent is capable by default; add only non-obvious guidance.
- Keep SKILL.md compact; move heavy reference material to `references/`.
- Put all trigger logic in frontmatter `description`, not in body sections.
- Prefer practical defaults and fallback paths over rigid instructions.
- Generalize from feedback; do not overfit to one prompt.

---

## OpenCode Tooling Conventions

When authoring or updating skills:

- Use `glob` for file discovery.
- Use `grep` for content search.
- Use `read` for inspection.
- Use `apply_patch` for focused edits.
- Use `bash` for terminal operations only (validation/tests/git).
- Run independent tool calls in parallel when possible.
- Ask questions only when blocked by ambiguity, risk, or missing secrets.
- Do not commit or push unless explicitly requested.

If a preferred tool is unavailable, use the closest safe equivalent and continue.

---

## Skill Anatomy

Use this structure:

```text
<skill-name>/
├── SKILL.md                 # required
├── agents/openai.yaml       # recommended (UI metadata)
├── scripts/                 # optional deterministic helpers
├── references/              # optional long docs/schemas
└── assets/                  # optional templates/static files
```

`SKILL.md` contains:

- Frontmatter:
  - `name` (required)
  - `description` (required; primary trigger signal)
- Body:
  - Workflow instructions, decision rules, and resource usage

Avoid extra process docs unless explicitly requested.

---

## Workflow

### 1) Capture Intent

Extract from conversation first. Fill gaps only when needed.

Collect:

- What outcomes should the skill produce?
- Which prompts/contexts should trigger it?
- Output format and quality bar?
- Constraints (tools, environment, policies)?

Gather 2-5 realistic prompts that represent actual usage.

### 2) Design Trigger Description

Write frontmatter `description` so it includes:

- What the skill does
- When it should trigger
- Adjacent cases where it should still trigger even if phrasing is indirect

Avoid vague wording. Use concrete contexts and verbs.

### 3) Plan Progressive Disclosure

Decide what goes where:

- `SKILL.md`: compact workflow and decision logic
- `references/*`: detailed schemas, long examples, domain facts
- `scripts/*`: repeated deterministic operations
- `assets/*`: templates/static artifacts used in outputs

If SKILL.md grows too large, split details into `references/` and link clearly.

### 4) Author the Skill

Write in imperative style.

Include:

- Step-by-step workflow
- Decision points and defaults
- Error handling and fallback behavior
- Resource usage instructions
- Short examples for tricky edge cases

Use strict MUST/NEVER language only for safety-critical or correctness-critical rules.

### 5) Validate Quality

Perform a quick quality pass:

- Frontmatter fields exist and are valid
- Description is specific enough to trigger reliably
- Instructions are executable with available tools
- No large duplicated content across SKILL.md and references
- Safety boundaries are explicit

If repo has validation scripts, run them via `bash`.

### 6) Iterate from Real Usage

After first usage:

1. Identify repeated failures or wasted effort
2. Update SKILL.md or add reusable resources
3. Re-test with realistic prompts
4. Repeat until behavior is stable

Prioritize patterns over one-off tweaks.

---

## Updating Existing Skills

When improving an existing skill:

- Preserve `name` unless rename is explicitly requested.
- Keep working parts; modify only bottlenecks.
- Ensure `agents/openai.yaml` still matches actual skill behavior.
- If trigger quality is poor, revise description first, then body.

---

## Lightweight Evaluation Pattern

Use this when no formal eval harness exists:

1. Prepare 3-8 realistic prompts (direct + indirect wording)
2. Execute and capture outcomes
3. Score by correctness, completeness, efficiency, and instruction adherence
4. Group failures by pattern
5. Apply focused changes and rerun

For trigger checks, include:

- should-trigger prompts
- near-miss should-not-trigger prompts

---

## Safety Boundaries

Do not create or optimize skills for malware, unauthorized access, stealth abuse, or deception.

Allow defensive/security work only with explicit benign intent and clear constraints.

---

## Output Contract

When asked to create or update a skill, deliver:

1. Proposed folder structure
2. Final `SKILL.md`
3. Optional `references/scripts/assets` stubs only when justified
4. Short rationale for trigger design
5. Suggested next test prompts for iteration
