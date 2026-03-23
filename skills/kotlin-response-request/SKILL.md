---
name: kotlin-response-request
description: Generate Kotlin Request/Response DTOs with kotlinx.serialization when users ask to create API models from JSON/schema/examples, including adjacent asks like "convert payload to Kotlin" or "make request/response classes"; do not trigger for domain/service/business logic tasks.
metadata:
  short-description: Kotlin Request/Response DTO generation
  version: 2.0.0
  owner: OpenCodeSkills
  updated_at: 2026-03-23
---

# Kotlin Response/Request DTO Skill

## 1) Purpose
- Goal: Generate strict Kotlin DTO models for API payloads using `kotlinx.serialization` with deterministic naming, nullability, and nesting limits.
- Non-goals: Generating Retrofit/Ktor clients, validation/business logic, database entities, mappers, or deep object hierarchies beyond one nested level.

## 2) Trigger Contract (MUST)
- Primary triggers: Requests to convert JSON/schema payloads into Kotlin `Request`/`Response` DTO classes.
- Secondary/indirect triggers: Phrases like "сделай Kotlin модели", "convert API body", "generate DTO from OpenAPI snippet", "make request and response classes".
- Must-not-trigger cases: Requests focused on repository/service/controller logic, app architecture, UI models, or non-Kotlin DTO targets.

## 3) Inputs Required
- Required inputs: Source payload shape (JSON/schema/text fields) and whether each root model is request or response.
- Optional inputs: Preferred class names, file names, field naming nuances, and primitive type hints.
- Missing-input fallback: Infer from payload keys and safe defaults; if request/response intent is ambiguous, default to `Response` and explicitly note the assumption.

## 4) Output Contract (MUST)
- Required deliverables: Complete Kotlin DTO code for each root entity, including imports, root class, and allowed direct nested classes.
- Output format: Kotlin code block(s) only, with `@Serializable` on every class, `@SerialName` on every field, `Type? = null` on every property, trailing commas, and `Request`/`Response` suffixes.
- Acceptance criteria: Exactly one root model per file, no `nested -> nested`, no non-null required fields, and no missing serialization annotations.

## 5) Workflow (Step-by-step)
1. Parse user input and identify root entities plus each root intent (`Request` or `Response`).
2. Build one file output per root entity and enforce root suffix naming.
3. Generate the root data class first with all fields nullable and optional (`? = null`).
4. For object fields directly under root, generate direct nested classes only; forbid deeper nesting.
5. Add `kotlinx.serialization` imports and annotate every class/field with `@Serializable` and `@SerialName`.
6. Apply type mapping defaults (`string -> String?`, `integer -> Int?`, `number -> Double?`, arrays -> nullable `List<...>?`).
7. Validate rule compliance (suffixes, one-root-per-file, one-level nesting, trailing commas) before returning.

## 6) Decision Rules
- If multiple root entities are present -> split into separate Kotlin files, one root class each.
- If nested objects appear under a nested class -> flatten structure or split into additional root files and explain why.
- Default path: Produce strict DTO-only code that satisfies all hard constraints even if the input schema is incomplete.

## 7) Resource Usage
- Use `references/...` for extended Kotlin serialization notes or mapping edge cases when the core skill becomes too long.
- Use `assets/...` for reusable output stubs (for example, canonical Request/Response file templates).
- Use `scripts/...` for deterministic transformations only if repeated conversion automation is introduced.

## 8) Error Handling and Fallbacks
- Common failure: Input implies forbidden deep nesting or multiple roots in one target file.
- Recovery action: Explain violated rule, then provide a compliant flattened/split alternative with explicit file separation.
- Hard-stop conditions: Never emit DTO code that breaks one-root-per-file, annotation coverage, nullability default, or nesting-depth constraints.

## 9) Safety Boundaries (MUST/NEVER)
- MUST: Keep output strictly limited to DTO declarations and serialization annotations aligned with provided payload intent.
- NEVER: Invent transport/business logic, silently remove fields, or generate structures that violate mandatory constraints.

## 10) Validation Checklist (BLOCKING)
- [ ] Frontmatter complete and valid
- [ ] Trigger contract includes should-trigger and should-not-trigger examples
- [ ] Output contract is testable and unambiguous
- [ ] Workflow is executable with available tools
- [ ] Safety boundaries are explicit

## 11) Test Prompts
- Should trigger:
  - "Сгенерируй Kotlin Response DTO из этого JSON ответа."
  - "Convert this OpenAPI schema into Kotlin request classes with kotlinx.serialization."
  - "Нужны `CreateUserRequest` и `CreateUserResponse` модели из примера payload."
- Should not trigger:
  - "Напиши Ktor endpoint для создания пользователя."
  - "Refactor service layer and repository interfaces."
  - "Сделай UI модель для Compose экрана профиля."
- Edge cases:
  - "Schema has nested object inside nested object" -> refuse deep nesting and return flattened/split compliant DTO plan.
  - "No explicit request/response type provided" -> default to `Response`, mark assumption, and continue.

## 12) Iteration Log
- v2.0.0:
  - Observed issue: Legacy skill format was useful but inconsistent with the new canonical template and trigger validation requirements.
  - Change made: Rebuilt the skill into the required 12-section structure with explicit trigger contract, output contract, decision rules, and test prompts.
  - Expected impact: More predictable skill activation and higher compliance when generating Kotlin DTO outputs.
