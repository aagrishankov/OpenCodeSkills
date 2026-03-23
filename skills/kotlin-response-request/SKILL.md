---
name: kotlin-response-request
description: Generate Kotlin Request/Response DTO classes with kotlinx.serialization when the user asks to convert JSON, schema, payload examples, or API contracts into models. Enforce nullable optional fields, explicit class and field serialization annotations, strict Request/Response suffix naming, one root model per file, and one-level nesting only (root -> nested).
metadata:
  short-description: Kotlin Request/Response DTO generation
  version: 2.0.0
  owner: opencode
  updated_at: 2026-03-23
---

# Kotlin Request/Response DTO Generator

## 1) Purpose
- Goal: Generate deterministic Kotlin DTO files for API payloads using `kotlinx.serialization` with strict structural rules.
- Non-goals: Do not generate business/domain models, mappers, repositories, use-cases, validators, or transport-unrelated logic.

## 2) Trigger Contract (MUST)
- Primary triggers: User asks to generate Kotlin `Request`/`Response` classes from JSON, OpenAPI-like schema snippets, API examples, or text contract descriptions.
- Secondary/indirect triggers: User asks to convert payload fields into nullable optional Kotlin DTOs with `@Serializable` and `@SerialName`.
- Must-not-trigger cases: Requests for safe domain models, mapper generation (`toData`/`toRequest`), architecture setup, endpoint client implementations, or framework-specific networking code.

## 3) Inputs Required
- Required inputs: At least one payload structure (JSON object, schema fragment, or textual field list) and intended model role (`Request` or `Response`) when inferable.
- Optional inputs: Desired root class name, preferred file names, type clarifications (`Int` vs `Long`), and explicit split preferences for multiple roots.
- Missing-input fallback: Infer sensible root names from top-level entities, default uncertain numeric types to `Double`, and clearly state assumptions.

## 4) Output Contract (MUST)
- Required deliverables: Complete Kotlin code for one or more `.kt` files where each file contains exactly one root `data class` plus optional direct nested classes, and concise in-skill examples with links to extended examples.
- Output format: Kotlin source with `kotlinx.serialization` imports, `@Serializable` on every class, `@SerialName("...")` on every field, nullable optional properties (`Type? = null`), and trailing commas in all multiline class parameter lists.
- Acceptance criteria: Every class name ends with `Request` or `Response`; no `nested -> nested`; each file has one root model; all fields are nullable with default `null`; no missing serialization annotations.

## 5) Workflow (Step-by-step)
1. Parse input payload/spec text and identify top-level entities.
2. Classify each top-level entity as `Request` or `Response` using explicit user intent or best-fit inference.
3. Split output into one file per root entity.
4. Build each root class first as a `data class` with `@Serializable`.
5. Generate direct nested classes only for object or object-array fields directly under the root.
6. Annotate every property with `@SerialName("json_key")` and make every property `Type? = null`.
7. Apply type mapping defaults and enforce class suffixes.
8. Validate root-per-file, depth limit, annotations, and trailing commas before final output.

## 6) Decision Rules
- If multiple root entities exist -> generate separate files, one root model per file.
- If object nesting exceeds one level -> flatten or split to satisfy `root -> nested` only, then explain applied fallback.
- If class naming does not end with `Request` or `Response` -> rename to compliant suffix.
- If a field lacks a stable JSON key alias -> use the source key as `@SerialName` value.
- If numeric type is ambiguous -> default to `Double? = null` unless user specifies `Int` or `Long`.
- Type defaults: `string -> String?`, `integer -> Int?`, `long -> Long?`, `number -> Double?`, `boolean -> Boolean?` (all with `= null`).
- Collection defaults: primitive arrays -> `List<T>? = null`; root-child object arrays -> `List<NestedXRequest/Response>? = null`; root-child object -> `NestedXRequest/Response? = null`.
- Default path: Prefer strict compliance over preserving invalid source shape.

## 7) Resource Usage
- Use `references/README.md` as the index for example and rule references.
- Use `references/examples.md` for full good/bad/fixed/edge/trigger-boundary examples.
- Use `assets/...` for reusable output templates only if a repeated format artifact is required.
- Use `scripts/...` only for deterministic repeated transforms; avoid scripts for one-off generation.

## 8) Error Handling and Fallbacks
- Common failure: Input implies multiple roots in one file, deep nested object trees, or missing request/response intent.
- Recovery action: Explain violated rule, split by root or flatten illegal depth, declare naming/type assumptions, then return compliant DTO code.
- Hard-stop conditions: Do not output models that violate root-per-file, suffix policy, annotation requirements, or depth constraints.

## 9) Safety Boundaries (MUST/NEVER)
- MUST: Enforce strict DTO constraints deterministically and provide assumptions when input is incomplete.
- NEVER: Emit non-nullable required fields, omit `@SerialName`, generate nested depth beyond one level, or produce non-`data class` DTOs.

## 10) Validation Checklist (BLOCKING)
- [ ] Frontmatter complete and valid
- [ ] Trigger contract includes should-trigger and should-not-trigger examples
- [ ] Output contract is testable and unambiguous
- [ ] Workflow is executable with available tools
- [ ] Safety boundaries are explicit
- [ ] Every class is a `data class`
- [ ] Every class has `@Serializable`
- [ ] Every field has explicit `@SerialName("...")`
- [ ] Every field is nullable and defaults to `null`
- [ ] Every class name ends with `Request` or `Response`
- [ ] Exactly one root model exists per file
- [ ] Nesting depth is limited to `root -> nested`
- [ ] Multiline class parameter lists use trailing commas
- [ ] `SKILL.md` includes concise practical examples and a minimal template
- [ ] `references/examples.md` contains happy-path, bad, fixed, edge, and trigger-boundary examples

## 11) Test Prompts
- Should trigger:
  - Convert this JSON payload into Kotlin `UserResponse` DTO using kotlinx.serialization.
  - Generate Kotlin `CreateOrderRequest` models from this field list and keep all fields nullable.
  - Build request/response DTO classes from this API schema with `@SerialName` on every field.
- Should not trigger:
  - Generate domain models and mappers from transport models.
  - Create Retrofit API interfaces and repository implementations.
  - Design app architecture layers for networking and caching.
- Edge cases:
  - Input has three top-level objects -> split into three files and keep one root per file.
  - Input has deep nested objects (`a.b.c`) -> flatten or split to remove nested-of-nested and explain the transformation.

Inline examples (concise):

Happy-path snippet:

```kotlin
import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class UserResponse(
    @SerialName("id")
    val id: String? = null,
    @SerialName("profile")
    val profile: ProfileResponse? = null,
) {
    @Serializable
    data class ProfileResponse(
        @SerialName("email")
        val email: String? = null,
    )
}
```

Bad snippet (forbidden nested -> nested):

```kotlin
@Serializable
data class UserResponse(
    @SerialName("profile")
    val profile: ProfileResponse? = null,
) {
    @Serializable
    data class ProfileResponse(
        @SerialName("details")
        val details: DetailsResponse? = null,
    ) {
        @Serializable
        data class DetailsResponse(
            @SerialName("name")
            val name: String? = null,
        )
    }
}
```

Fixed direction: flatten `DetailsResponse` to a direct child of root or split into a separate root file.

Minimal template:

```kotlin
import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class ExampleResponse(
    @SerialName("id")
    val id: String? = null,
    @SerialName("meta")
    val meta: MetaResponse? = null,
) {
    @Serializable
    data class MetaResponse(
        @SerialName("source")
        val source: String? = null,
    )
}
```

Extended example library: `skills/kotlin-response-request/references/examples.md`
Reference index: `skills/kotlin-response-request/references/README.md`

## 12) Iteration Log
- v2.0.0:
  - Observed issue: Legacy skill had strong rules but lacked canonical structure, trigger contract clarity, and operational metadata.
  - Change made: Reorganized into canonical 12-section template, added metadata fields, decision rules, and explicit test prompts.
  - Expected impact: More reliable triggering, clearer enforcement behavior, and easier maintainability across skill packages.
