---
name: kotlin-client-repository-generator
description: Generate client-side Kotlin repository interface and implementation for Android/KMP when the user asks to create or refactor repository layer code. Enforce safe-model repository contracts, only-used dependencies (featureDao/httpClient), existing mapper calls, Russian KDoc, strict formatting, and deterministic templates for DAO-only, HTTP-only, DAO+HTTP, Flow, and HTTP 204-no-body scenarios.
metadata:
  short-description: Kotlin client repository generator
  version: 2.0.0
  owner: opencode
  updated_at: 2026-03-24
---

# Kotlin Client Repository Generator

## 1) Purpose
- Goal: Generate deterministic, compile-ready client Kotlin repository code (`package + imports + interface + implementation`) for Android/KMP.
- Non-goals: Do not generate DTO/Entity/safe model classes, mapper functions, DI config, architecture placement, threading policy, paging/websocket/multipart logic, expect/actual, or any backend/server code.

## 2) Trigger Contract (MUST)
- Primary triggers: User asks to create, refactor, or regenerate Kotlin repository interface/impl for Android or KMP client app.
- Secondary/indirect triggers: User asks for repository templates with DAO/HTTP data source wiring, mapper-based conversion (`toData()/toRequest()/toEntity()`), or Russian KDoc for repository methods.
- Must-not-trigger cases: Requests for backend services/controllers, use-case generation, DI module wiring, model/mapper generation, or architecture/package planning only.

## 3) Inputs Required
- Required inputs: Feature/entity intent, repository method contracts (params + return type), and data-source behavior (DAO, HTTP, or explicit DAO+HTTP strategy).
- Optional inputs: Exact class names, endpoint/path placeholders, mapper availability (list mapper vs element mapper), and explicit error contract (`Result<T>` vs throwing).
- Missing-input fallback: Use `FeatureRepository`/`FeatureRepositoryImpl`, infer dependency names (`featureDao`, `httpClient`), keep safe models in interface, and avoid adding error handling/threading unless explicitly required.

## 4) Output Contract (MUST)
- Required deliverables: Public repository interface + implementation class with only required constructor dependencies, valid imports, Russian KDoc, and compile-ready Kotlin.
- Output format: Block-body functions only (no expression bodies), multiline params when count >= 2, trailing commas in all multiline structures, explicit mapper calls only from allowed set.
- Acceptance criteria: Interface contains only safe models; implementation uses only used dependencies; no `withContext`/`Dispatchers`/DI annotations; Flow and `Result<T>` only when explicitly required by input contract.

## 5) Workflow (Step-by-step)
1. Parse repository contract: methods, params, return types, and data-source strategy.
2. Build interface with safe-model-only method signatures and short Russian KDoc.
3. Build implementation with only used dependencies (`featureDao`, `httpClient` as needed).
4. Implement method bodies with existing mapper calls and required source interactions.
5. Apply scenario-specific rules (DAO-only, HTTP-only, DAO+HTTP, Flow, HTTP 204/no-body).
6. Add detailed Russian KDoc on implementation methods.
7. Validate strict formatting and forbidden constructs.
8. Return full compile-ready Kotlin output.

## 6) Decision Rules
- If method uses only local persistence -> DAO-only template.
- If method uses only network request -> HTTP-only template.
- If both DAO and HTTP are required -> use DAO+HTTP flow only when explicitly requested by contract (for example, `isForce`).
- If return body is absent (HTTP 204/no-content) -> do not call `.body<Unit>()`.
- If contract requires `Flow<T>` -> use DAO observation + `map { ...toData() }`.
- If list mapper exists (`List<Entity>.toData()`) -> prefer list mapper; otherwise map elements.
- If error handling is not explicitly requested -> do not add `try/catch`, `runCatching`, or forced `Result` wrapping.
- Default path: keep repository thin, deterministic, and mapping-driven.

## 7) Resource Usage
- Use `references/README.md` as index for extended guidance and examples.
- Use `references/examples.md` for full compile-ready templates and scenario snippets.
- Use `assets/...` only if future output formatting artifacts are introduced.
- Use `scripts/...` only for deterministic repeated transforms; avoid for one-off code generation.

## 8) Error Handling and Fallbacks
- Common failure: Interface leaks DTO/Entity/Request/Response or implementation includes unused dependencies.
- Recovery action: Rewrite interface to safe models only and remove unused constructor dependencies.
- Common failure: Generated code adds forbidden threading/error wrappers.
- Recovery action: Remove `withContext`, `Dispatchers`, `try/catch`, `runCatching`, and implicit `Result` wrapping unless explicitly required.
- Hard-stop conditions: Do not output code that is server-side, non-compile-ready, or violates formatting/contract constraints.

## 9) Safety Boundaries (MUST/NEVER)
- MUST: Generate repository layer only for client Android/KMP scenarios.
- MUST: Keep nullability exactly as defined by input contract.
- MUST: Use only existing mapper calls (`Response.toData()`, `Data.toRequest()`, `Data.toEntity()`, `Entity.toData()`).
- NEVER: Generate DTO/Entity/Request/Response classes, mapper functions, DI wiring, or server/backend code.
- NEVER: Add unused constructor dependencies or forbidden annotations (`@Inject`, `@Singleton`, `@Factory`).
- NEVER: Use expression-body functions, region comments, `withContext`, or `Dispatchers`.

## 10) Validation Checklist (BLOCKING)
- [ ] Frontmatter complete and valid
- [ ] Trigger contract includes should-trigger and should-not-trigger examples
- [ ] Output contract is testable and unambiguous
- [ ] Workflow is executable with available tools
- [ ] Safety boundaries are explicit
- [ ] Interface contains only safe models (no DTO/Entity/Request/Response)
- [ ] Implementation correctly implements interface contracts
- [ ] Constructor includes only actually used dependencies
- [ ] Allowed mapper calls only; no mapper generation
- [ ] Russian KDoc present: short in interface, detailed in implementation
- [ ] No expression bodies; multiline params (>=2) and trailing commas are correct
- [ ] Imports/package are present; output is compile-ready
- [ ] Scenario constraints respected (DAO-only / HTTP-only / DAO+HTTP / Flow / HTTP 204)
- [ ] Code examples preserved and synchronized with rules

## 11) Test Prompts
- Should trigger:
  - Generate `UserRepository` and `UserRepositoryImpl` for Android client: read users from DAO and return safe model list.
  - Refactor this KMP repository to use only `httpClient` and map `UserResponse` to `List<User>` with `toData()`.
  - Create repository method with `isForce` that fetches from network and caches entities in DAO.
- Should not trigger:
  - Build Ktor server repository and service layer for users.
  - Generate DTO classes and mappers for user API.
  - Create DI module bindings for repository implementations.
- Edge cases:
  - API returns 204 for delete endpoint -> do not call `.body<Unit>()`, keep `Unit` contract.
  - Contract asks `Flow<List<User>>` observe method -> generate Flow mapping path, not suspend list fetch.

Inline examples (concise):

DAO-only:

```kotlin
override suspend fun getUsers(): List<User> {
    return userDao
        .getUsers()
        .toData()
}
```

HTTP-only:

```kotlin
override suspend fun getUsers(): List<User> {
    return httpClient
        .get("users")
        .body<UserResponse>()
        .toData()
}
```

HTTP 204/no-body:

```kotlin
override suspend fun deleteUser(
    userId: String,
): Unit {
    val body = userId.toRequest()

    httpClient.post("users/delete") {
        setBody(body)
    }

    return Unit
}
```

Extended example library: `skills/kotlin-client-repository-generator/references/examples.md`
Reference index: `skills/kotlin-client-repository-generator/references/README.md`

## 12) Iteration Log
- v2.0.0:
  - Observed issue: Previous version had comprehensive constraints and templates but mixed structure, duplicated rules, and less explicit trigger boundaries.
  - Change made: Refactored to canonical 12-section template, clarified trigger/output/safety contracts, and moved full template set into references while keeping concise inline examples.
  - Expected impact: More predictable triggering, easier maintenance, and lower rule drift without losing code-example coverage.
