---
name: kotlin-safe-model-mapper
description: Generate architecture-agnostic safe Kotlin application models and deterministic mappers from unsafe transport models when the user asks for safe model design, transport-to-safe mapping, nullability normalization, aggregation with source preservation, and strict mapper contracts (`toData()`/`toRequest()`).
metadata:
  short-description: Safe model and mapper generator
  version: 2.0.0
  owner: opencode
  updated_at: 2026-03-24
---

# Safe Model & Mapper Generator (Kotlin)

## 1) Purpose
- Goal: Generate deterministic, reusable, safe-by-default Kotlin models and mapper functions for converting unsafe transport models into safe application models.
- Non-goals: Do not generate transport models (`Request`/`Response`/`DTO`), architecture layers, repositories/use-cases, framework/network code, or package/module structure decisions.

## 2) Trigger Contract (MUST)
- Primary triggers: User asks to build safe Kotlin models from transport models, add `toData()`/`toRequest()` mappers, or refactor nullable transport shape into business-safe models.
- Secondary/indirect triggers: User requests normalization (`trim`, blank handling), critical field validation, aggregation objects (name/period/phone/address), or deterministic mapper contracts.
- Must-not-trigger cases: Pure DTO generation from JSON/OpenAPI, networking client generation, architecture scaffolding, or non-mapper Kotlin tasks.

## 3) Inputs Required
- Required inputs: Transport model shape (existing classes, schema, or field list) and target safe model intent.
- Optional inputs: Critical field list, preferred fail-fast policy, desired aggregation groups, and whether reverse mapping (`toRequest`) is required.
- Missing-input fallback: Infer safe defaults from field semantics, keep assumptions explicit, and favor strict safety over permissive mirroring.

## 4) Output Contract (MUST)
- Required deliverables: Safe model(s), one root mapper per file, private helpers, and explicit validation/normalization behavior.
- Output format: Kotlin code with block-body functions only, required trailing commas in multiline structures, and deterministic ordering.
- Acceptance criteria: Root mapper names restricted to `toData()`/`toRequest()`, root visibility `internal`, helpers `private`, no expression bodies, no blind nullable propagation, and aggregation preserves source data.

## 5) Workflow (Step-by-step)
1. Parse transport shape and mark critical vs optional business fields.
2. Design safe models with intentional nullability and non-null collections by default.
3. Identify logical groups for aggregation and preserve raw fields when deriving values.
4. Generate one root mapper per file (`toData()` or `toRequest()`), `internal` visibility.
5. Add private nested/normalization/aggregation helpers in canonical order.
6. Enforce formatting and contract constraints (block bodies, trailing commas, naming rules).
7. Validate output against the blocking checklist before returning.

## 6) Decision Rules
- If transport field is nullable but business requires value -> fail fast (`error` or explicit guard), do not use `orEmpty()`.
- If string-like input is present -> apply `trim()`, then treat blank as `null` or explicit fallback.
- If collection exists -> default to non-null collection in safe model; use nullable collection only when business semantics require it.
- If 2+ related fields or derived value exist -> create a dedicated aggregation class and keep source fields accessible.
- If generating `toRequest()` -> keep it deterministic and place root `toRequest()` in a file separate from root `toData()`.
- Default path: prefer strict, explicit, business-safe mappings over transport-shape mirroring.

## 7) Resource Usage
- Use `references/README.md` as the index for extended guidance and examples.
- Use `references/examples.md` for full templates, good/bad examples, and edge-case snippets.
- Keep inline examples in this file concise; keep large examples in references.

## 8) Error Handling and Fallbacks
- Common failure: Input encourages blind nullable propagation, silent defaults for critical fields, or monolithic unreadable mapper bodies.
- Recovery action: Split logic into private helpers, enforce fail-fast for critical fields, normalize strings consistently, and rebuild aggregations with source preservation.
- Hard-stop conditions: Do not return output that violates root mapper naming/visibility, one-root-per-file, block body only, or trailing comma requirements.

## 9) Safety Boundaries (MUST/NEVER)
- MUST: Stay architecture-agnostic, deterministic, and explicit about assumptions when input is incomplete.
- MUST: Preserve source information when deriving aggregated fields.
- NEVER: Generate transport models, introduce architecture-coupled dependencies in mapper signatures, or use expression-body mappers.
- NEVER: Use hidden defaults for critical fields or relax required formatting/contract constraints.

## 10) Validation Checklist (BLOCKING)
- [ ] Frontmatter complete and valid
- [ ] Trigger contract includes should-trigger and should-not-trigger examples
- [ ] Output contract is testable and unambiguous
- [ ] Workflow is executable with available tools
- [ ] Safety boundaries are explicit
- [ ] Exactly one root mapper exists per mapper file
- [ ] Root mapper name is only `toData()` or `toRequest()`
- [ ] Root mapper visibility is `internal`
- [ ] Nested helpers are `private`
- [ ] All functions use full block bodies (`{}`), no expression bodies
- [ ] Trailing commas are present in every multiline Kotlin structure
- [ ] Safe model names do not use suffix/layer markers (`Model`, `Data`, `Domain`, `Entity`)
- [ ] Safe model nullability is intentional (not transport-mirrored blindly)
- [ ] Critical fields use explicit validation/fail-fast, never silent fallback
- [ ] Aggregations preserve original source fields while adding derived values
- [ ] `SKILL.md` contains concise examples and references `references/examples.md`

## 11) Test Prompts
- Should trigger:
  - Build safe Kotlin models and `toData()` mappers from these nullable response classes.
  - Refactor this mapper so critical fields fail fast and strings are normalized.
  - Create aggregation objects (name/period/phone) while preserving raw fields.
- Should not trigger:
  - Generate `@Serializable` DTO `Request`/`Response` classes from JSON.
  - Implement Retrofit API interfaces and repositories.
  - Design Clean Architecture modules for network/domain layers.
- Edge cases:
  - Deeply nullable nested transport input -> keep safe root readable, filter invalid nested entries, and document assumptions.
  - User asks for both `toData()` and `toRequest()` in one file -> split root mappers into separate files.

Inline examples (concise):

Required block body (good):

```kotlin
internal fun UserResponse.toData(): User {
    val id = id ?: error("User.id is required")

    return User(
        id = id,
    )
}
```

Forbidden expression body (bad):

```kotlin
internal fun UserResponse.toData(): User = User(...)
```

Required trailing comma (good):

```kotlin
data class User(
    val id: String,
    val name: PersonName,
    val avatarUrl: String?,
)
```

Forbidden nullable collection mirroring (bad):

```kotlin
data class User(
    val items: List<Item>?,
)
```

Extended example library: `skills/kotlin-safe-model-mapper/references/examples.md`
Reference index: `skills/kotlin-safe-model-mapper/references/README.md`

## 12) Iteration Log
- v2.0.0:
  - Observed issue: Legacy skill was strong in content but overloaded in a single file and not aligned to the canonical 12-section structure.
  - Change made: Refactored into canonical template, kept strict rules, moved full examples into references, and preserved concise inline guardrail snippets.
  - Expected impact: More reliable triggering, easier maintenance, and retained example coverage without losing enforcement clarity.
