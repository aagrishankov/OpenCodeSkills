---
name: skill-description-generator
description: Create and refresh repository README skill catalogs when the user asks to generate, standardize, or update README.md from skill metadata/frontmatter. Enforce a single markdown template, extract data from SKILL.md meta blocks first, and present human-readable Russian descriptions.
metadata:
  short-description: README catalog generator for skills
  version: 1.0.0
  owner: opencode
  updated_at: 2026-03-24
---

# Skill Description Generator

## 1) Purpose
- Goal: Deterministically generate and update `README.md` with a unified skill catalog based on metadata in skill `SKILL.md` files.
- Non-goals: Do not rewrite internal logic of unrelated skills and do not invent features that are absent from source metadata/content.

## 2) Trigger Contract (MUST)
- Primary triggers: User asks to create/update `README.md`, build a skill catalog, normalize README format, or regenerate skill descriptions from metadata/frontmatter.
- Secondary/indirect triggers: User asks to keep one README template for all skills, translate skill descriptions to Russian, or sync README with newly added skills.
- Must-not-trigger cases: Requests to implement business/application code, generate Kotlin DTO/domain code, or refactor non-documentation files unrelated to the skill catalog.

## 3) Inputs Required
- Required inputs: Target README path (default `README.md`) and directory roots where skills are stored.
- Optional inputs: Include/exclude patterns, section title override, custom ordering, and whether to include metadata fields (`version`, `owner`, `updated_at`).
- Missing-input fallback: Use defaults `README.md` and skill roots `skills/` plus `skills/.opencode/skill/`.

## 4) Output Contract (MUST)
- Required deliverables: Updated `README.md` with a unified skill section and one consistent block format per skill.
- Output format: Markdown in Russian, with skill names preserved in code formatting and metadata-driven descriptions.
- Acceptance criteria: Every listed skill has a block generated from metadata/frontmatter first; Russian text is natural and concise; entry layout is identical for all skills.

## 5) Workflow (Step-by-step)
1. Discover skill files under `skills/**/SKILL.md` and optional custom roots.
2. Parse frontmatter from each `SKILL.md` and extract `name`, `description`, `metadata.short-description`, `metadata.version`, `metadata.owner`, `metadata.updated_at`.
3. Build Russian purpose text using priority: `metadata.short-description` -> first sentence of `description` -> first meaningful line under `## 1) Purpose`.
4. Normalize purpose text to Russian while preserving technical tokens (`Request`, `Response`, `DTO`, file paths, code identifiers).
5. Render each skill with the unified README template from `assets/readme-entry-template.md`.
6. Sort skills by `name` (ascending) unless explicit user ordering is provided.
7. Replace or create the `## Skills` section in `README.md` without breaking unrelated README sections.
8. Validate: no duplicate entries, no missing names, and all generated entries follow one format.

## 6) Decision Rules
- If `metadata.short-description` exists -> use it as the primary source for Russian "Назначение".
- If `metadata.short-description` is missing -> derive from `description` and translate/simplify.
- If frontmatter is missing or invalid -> fallback to section text in `SKILL.md` and mark assumptions briefly.
- If both `skills/` and `skills/.opencode/skill/` contain the same skill name -> prefer the path explicitly requested by user.
- Default path: prioritize metadata consistency over preserving legacy README phrasing.

## 7) Resource Usage
- Use `references/README.md` for section boundaries and update strategy.
- Use `references/examples.md` for positive/negative examples of generated README entries.
- Use `assets/readme-entry-template.md` as the single canonical per-skill block template.
- Use `scripts/` only if deterministic README regeneration is requested explicitly.

## 8) Error Handling and Fallbacks
- Common failure: Missing frontmatter fields or mixed old/new README formats.
- Recovery action: Apply fallback extraction chain and normalize output to the canonical template.
- Hard-stop conditions: Do not produce output that mixes multiple entry templates in one README update.

## 9) Safety Boundaries (MUST/NEVER)
- MUST: Preserve unrelated README sections and keep generated skill facts grounded in source files.
- NEVER: Fabricate unsupported skill capabilities, silently drop discovered skills, or alter non-target documentation files without request.

## 10) Validation Checklist (BLOCKING)
- [ ] Frontmatter complete and valid
- [ ] Trigger contract includes should-trigger and should-not-trigger examples
- [ ] Output contract is testable and unambiguous
- [ ] Workflow is executable with available tools
- [ ] Safety boundaries are explicit
- [ ] Unified README entry template is applied to all generated skills
- [ ] Russian descriptions are derived from metadata/frontmatter first
- [ ] `README.md` keeps non-skill sections intact

## 11) Test Prompts
- Should trigger:
  - "Обнови README по всем SKILL.md и сделай единый шаблон описаний."
  - "Собери список скиллов в README из метаданных и переведи описания на русский."
  - "Добавь новый скилл в README в том же формате, что и остальные."
- Should not trigger:
  - "Сгенерируй Kotlin DTO из JSON."
  - "Напиши репозиторий и usecase для Android."
  - "Оптимизируй SQL-запросы в backend."
- Edge cases:
  - "У части SKILL.md нет metadata.short-description" -> fallback к `description` и пометка предположений.
  - "README уже содержит кастомные разделы" -> обновить только `## Skills`, остальные разделы не трогать.

## 12) Iteration Log
- v1.0.0:
  - Observed issue: README entries for skills were maintained manually and drifted in style.
  - Change made: Introduced metadata-first, Russian-localized, single-template generation rules.
  - Expected impact: Faster and more consistent README updates with less manual editing.
