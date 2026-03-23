---
name: kotlin-client-usecase-generator
description: Generate client-side Kotlin UseCase interface and internal implementation when the user asks to create/refactor Android or KMP client use cases. Enforce execute-only suspend contract, DispatcherProvider with withContext (IO by default, Default only for CPU-bound work), Russian KDoc, strict naming/formatting rules, and deterministic output without repositories/mappers/DI generation.
metadata:
  short-description: Kotlin client UseCase generation
  version: 2.0.0
  owner: opencode
  updated_at: 2026-03-24
---

# Kotlin Client UseCase Generator

## 1) Purpose
- Goal: Generate deterministic client-side Kotlin use case code (`interface` + `internal Impl`) for Android/KMP with strict execution, dispatcher, and formatting constraints.
- Non-goals: Do not generate repository/datasource/mapper/DI/module wiring, architecture placement, Flow-based contracts, or extra abstraction layers.

## 2) Trigger Contract (MUST)
- Primary triggers: User asks to create or refactor a Kotlin use case for Android or Kotlin Multiplatform client code.
- Secondary/indirect triggers: User requests `execute`-style orchestration logic, `DispatcherProvider` usage, or use case templates with Russian KDoc.
- Must-not-trigger cases: Backend/server handlers, repository implementation requests, DI module generation, API client generation, architecture/package planning.

## 3) Inputs Required
- Required inputs: Use case intent (feature/action), input parameters, and expected return contract (`Unit`, model, nullable, primitive, sealed result, etc.).
- Optional inputs: Dependency names/types, CPU-bound vs IO-bound clarification, explicit error-handling requirement, and preferred naming.
- Missing-input fallback: Infer `FeatureActionUseCase` naming, default dispatcher to `dispatcherProvider.IO`, preserve provided return contract, and avoid adding try/catch unless explicitly required.

## 4) Output Contract (MUST)
- Required deliverables: Kotlin `interface` with exactly one public `suspend fun execute(...)`, plus `internal` implementation with constructor dependencies, orchestration logic, and optional private helpers.
- Output format: Kotlin code with Russian KDoc (short on interface, detailed on implementation), named arguments in calls, trailing commas in multiline blocks, and no extra generated entities.
- Acceptance criteria: No `operator invoke`; no Flow return type; `DispatcherProvider + withContext(...)` always present; `dispatcherProvider.Main` forbidden; implementation exposes no additional public/internal API beyond `execute`.

## 5) Workflow (Step-by-step)
1. Parse requested use case intent and return contract.
2. Build `FeatureActionUseCase` interface with one public `suspend fun execute(...)`.
3. Build `internal FeatureActionUseCaseImpl` with only required dependencies and strict lowerCamelCase names.
4. Wrap main business logic in `withContext(...)` using dispatcher policy.
5. Implement orchestration logic (branching, aggregation, read/write, side effects) without adding forbidden abstractions.
6. Add private helper methods only when they add semantic value.
7. Add Russian KDoc per policy.
8. Validate naming, contract, formatting, and restriction checklist before final output.

## 6) Decision Rules
- If parameter count is 1 -> single-line signature allowed.
- If parameter count is 2+ -> multiline signature required.
- If dependency count is 2+ -> multiline constructor list required.
- If logic is single-step post-processing -> `also` is allowed.
- If logic has multiple steps -> use local variables and explicit return.
- If workload is clearly CPU-bound -> use `dispatcherProvider.Default`; otherwise use `dispatcherProvider.IO`.
- If try/catch is not explicitly requested by contract/task -> do not add try/catch.
- Default path: keep orchestration in `execute`, extract only meaningful private helpers, preserve return contract exactly.

## 7) Resource Usage
- Use `references/README.md` as the index for extended examples.
- Use `references/examples.md` for canonical, unit-return, multi-branch, bad, and fixed examples.
- Use `assets/...` only if repeated response artifacts are needed.
- Use `scripts/...` only for deterministic repeated transforms; avoid scripts for one-off generation.

## 8) Error Handling and Fallbacks
- Common failure: Generated code adds `invoke`, public impl, `Flow`, wrong dispatcher, or unnecessary try/catch.
- Recovery action: Rewrite to strict `execute` contract, force `internal Impl`, enforce dispatcher policy, and remove forbidden constructs.
- Hard-stop conditions: Do not output code that changes return contract, adds extra entities, uses `dispatcherProvider.Main`, or violates public API constraints.

## 9) Safety Boundaries (MUST/NEVER)
- MUST: Generate client-side Android/KMP use case code only; keep architecture-neutral file placement.
- MUST: Use `DispatcherProvider` with `withContext(...)` for main logic.
- NEVER: Generate repositories/datasources/mappers/DI or enforce package/layer placement.
- NEVER: Add `operator invoke`, `Flow`, extra public API, public impl class, or default try/catch without explicit requirement.

## 10) Validation Checklist (BLOCKING)
- [ ] Frontmatter complete and valid
- [ ] Trigger contract includes should-trigger and should-not-trigger examples
- [ ] Output contract is testable and unambiguous
- [ ] Workflow is executable with available tools
- [ ] Safety boundaries are explicit
- [ ] Use case name matches `FeatureActionUseCase`
- [ ] Implementation name matches `FeatureActionUseCaseImpl`
- [ ] Interface is public and implementation is `internal`
- [ ] Exactly one public method exists and it is `suspend fun execute(...)`
- [ ] `DispatcherProvider + withContext(...)` is used
- [ ] Dispatcher selection is valid (`IO` default, `Default` for CPU-only, never `Main`)
- [ ] No `operator invoke`
- [ ] No `Flow` in contract
- [ ] No try/catch unless explicitly required
- [ ] Named arguments are used for all calls
- [ ] Multiline formatting and trailing commas are correct
- [ ] KDoc is in Russian (short in interface, detailed in impl)
- [ ] `SKILL.md` includes concise practical examples and links to extended examples

## 11) Test Prompts
- Should trigger:
  - Create `AuthOtpVerifyUseCase` for Android client with execute contract and token save side effect.
  - Generate a KMP client use case interface + internal impl with DispatcherProvider and Russian KDoc.
  - Refactor this Kotlin use case to remove `invoke` and keep only `suspend execute`.
- Should not trigger:
  - Generate backend application service and controller for OTP verification.
  - Build repository + datasource + DI module for this feature.
  - Design package architecture and module boundaries for the app.
- Edge cases:
  - Request includes CPU-heavy hashing only -> use `dispatcherProvider.Default` and explain why.
  - Request asks for try/catch but contract is silent -> do not add try/catch; keep original error contract.

Inline examples (concise):

Canonical snippet:

```kotlin
/**
 * Проверяет OTP-код пользователя и возвращает результат верификации.
 */
interface AuthOtpVerifyUseCase {

    /**
     * Выполняет проверку OTP-кода
     * по идентификатору сессии верификации.
     */
    suspend fun execute(
        code: String,
        verificationSessionId: String,
    ): PhoneVerifyResult
}

/**
 * Реализация проверки OTP-кода.
 *
 * Выполняет удаленную верификацию через репозиторий,
 * затем при успешной проверке сохраняет токены.
 *
 * Основная логика выполняется в IO dispatcher,
 * так как сценарий включает сетевой вызов
 * и локальное сохранение данных.
 */
internal class AuthOtpVerifyUseCaseImpl(
    private val dispatcherProvider: DispatcherProvider,
    private val authRepository: AuthRepository,
    private val authTokenSaver: AuthTokenSaver,
) : AuthOtpVerifyUseCase {

    /**
     * Выполняет удаленную проверку OTP-кода
     * и при успешном результате сохраняет токены.
     */
    override suspend fun execute(
        code: String,
        verificationSessionId: String,
    ): PhoneVerifyResult {
        return withContext(dispatcherProvider.IO) {
            authRepository.authOtpVerify(
                code = code,
                verificationSessionId = verificationSessionId,
            ).also { saveTokens(result = it) }
        }
    }

    /**
     * Сохраняет токены после успешной проверки.
     */
    private suspend fun saveTokens(
        result: PhoneVerifyResult,
    ) {
        when (result) {
            is PhoneVerifyResult.PhoneVerifyFailed -> Unit
            is PhoneVerifyResult.PhoneVerifySuccess -> {
                val tokens = AuthTokens(
                    accessToken = result.accessToken,
                    refreshToken = result.refreshToken,
                )

                authTokenSaver.save(
                    tokens = tokens,
                )
            }
        }
    }
}
```

Bad snippet (do not generate):

```kotlin
interface XUseCase {
    suspend operator fun invoke(id: String): User
}
```

Extended example library: `skills/kotlin-client-usecase-generator/references/examples.md`
Reference index: `skills/kotlin-client-usecase-generator/references/README.md`

## 12) Iteration Log
- v2.0.0:
  - Observed issue: Legacy skill had strong constraints and rich examples, but structure was non-canonical, rules were duplicated, and trigger boundaries were implicit.
  - Change made: Refactored to canonical 12-section template, made trigger/output contracts explicit, centralized decision/safety rules, and moved extended code examples into references while preserving inline samples.
  - Expected impact: More consistent triggering, easier maintenance, and lower risk of rule drift without losing example coverage.
