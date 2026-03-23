---
name: kotlin-client-usecase-generator
description: Generate client-side Kotlin UseCase interface and internal implementation for Android/Kotlin Multiplatform apps: enforce execute-only suspend contract, DispatcherProvider with withContext (IO by default, Default only for CPU-bound work), orchestration business logic with optional private helpers, Russian KDoc, strict naming/formatting rules, and deterministic templates/examples without repositories/mappers/DI generation.
metadata:
  short-description: Kotlin client use case generator
---

# Kotlin Client UseCase Generator (Android / KMP client)

Generate use cases only for client-side Kotlin applications:
- Android
- Kotlin Multiplatform (client side)

Never generate backend/server code.

---

## 1) Purpose

This skill generates:
- UseCase interface
- UseCaseImpl implementation
- constructor dependencies (only required ones)
- orchestration business logic
- execute method
- private helper methods (when needed)
- KDoc (in Russian)

---

## 2) Scope

### Generated entities
- UseCase interface
- UseCaseImpl implementation
- execute method
- private helper methods
- KDoc documentation

### Never generated
- repository
- datasource
- mapper
- DI configuration
- module wiring
- Flow-based use case
- architectural placement
- extra abstraction layers

---

## 3) Naming Rules

### UseCase naming
Use:
- FeatureActionUseCase
- FeatureActionUseCaseImpl

Examples:
- AuthOtpVerifyUseCase
- AuthOtpVerifyUseCaseImpl
- UserProfileLoadUseCase
- UserProfileLoadUseCaseImpl

### Dependency naming
- lowerCamelCase
- variable name based on dependency type

Examples:
- dispatcherProvider
- authRepository
- authTokenSaver
- profileRepository
- sessionManager
- analyticsTracker

Forbidden generic names:
- repository
- manager
- data
- repo
- helper
- util

Example:
- type AuthRepository -> variable authRepository

---

## 4) Contract Rules

### Interface contract
- Exactly one public method.
- Method name must be execute.
- Signature must be suspend fun execute(...).
- Additional public API is forbidden.

### execute method
- Always suspend.
- Return type can be any task-defined contract type (Unit, model, primitive, nullable, sealed result, etc.).
- operator fun invoke is forbidden.
- Only execute is allowed.

---

## 5) Implementation Rules

### Visibility
- interface: public
- implementation: internal

Example:
```kotlin
internal class AuthOtpVerifyUseCaseImpl(
    private val dispatcherProvider: DispatcherProvider,
    private val authRepository: AuthRepository,
) : AuthOtpVerifyUseCase
```

### Public API
Forbidden:
- additional public methods
- additional entry points
- any public/internal API except execute

Allowed:
- private methods

---

## 6) Dispatcher Policy (Hard Rule)

Each use case must:
- use DispatcherProvider
- run main logic inside withContext(...)

Dispatcher selection:
- default: dispatcherProvider.IO
- use dispatcherProvider.Default only for clearly CPU-bound work
- dispatcherProvider.Main is forbidden

---

## 7) Business Logic Rules

Use case is an orchestration unit, not a proxy.

Allowed:
- use multiple repositories
- use cross-feature dependencies
- read/write data
- apply post-processing
- save local data
- aggregate data sources
- branch logic (if, when, early return)
- validate input
- implement multi-step scenarios

Allowed side effects:
- token saving
- cache updates
- local writes
- remote + local orchestration

---

## 8) Error Handling Policy

By default, forbidden:
- adding try/catch without explicit need
- changing error contract
- introducing Result wrappers on your own

Allowed:
- try/catch only when explicitly required by task/contract

---

## 9) Private Methods Policy

Private methods:
- are allowed when they add semantic value
- can be suspend
- can contain business logic
- can use named arguments

Do not create private methods:
- for structure only
- for aesthetics only
- without logical need

---

## 10) Logic Composition Rules

### also usage
Allowed only for simple single-step post-processing:

```kotlin
repository.verify(
    code = code,
).also { saveTokens(result = it) }
```

### Local variable usage
Required when logic is more than one step:

```kotlin
val result = repository.verify(
    code = code,
)

saveTokens(
    result = result,
)

return result
```

---

## 11) KDoc Policy

KDoc must be written in Russian.

Required:
- short KDoc for interface
- short KDoc for execute in interface
- detailed KDoc for implementation and execute in implementation

Implementation KDoc must explain:
- what the use case does
- how logic works
- why this dispatcher is used
- what private methods do (if present)

---

## 12) Formatting Specification (Hard Rules)

### Parameter declaration
- if method has 1 parameter: single-line is allowed
- if method has 2+ parameters: multiline only

Example:
```kotlin
suspend fun execute(
    code: String,
    sessionId: String,
): Result
```

### Named arguments
Always use named arguments in calls:
- even for one argument

```kotlin
repository.loadUser(
    userId = userId,
)
```

### Trailing comma
Mandatory in all multiline structures:
- parameters
- arguments
- constructors

### Constructor formatting
If dependencies count is 2+, use vertical list only:

```kotlin
internal class ExampleUseCaseImpl(
    private val dispatcherProvider: DispatcherProvider,
    private val repository: ExampleRepository,
) : ExampleUseCase
```

---

## 13) Abstraction Restrictions

Forbidden:
- unnecessary abstraction layers
- wrappers without purpose
- factories without reason
- executors without reason
- helper classes without a concrete task

---

## 14) Architectural Neutrality

This skill:
- does not define architecture
- does not define package structure
- does not define layers
- does not place files in project structure

---

## 15) Explicit Restrictions (Never)

Never:
- use operator invoke
- make Impl public
- add extra public API
- add Flow
- add try/catch without explicit reason
- change return contract
- create extra entities
- enforce package/layer placement

---

## 16) Canonical Template

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

---

## 17) Additional Templates

### 17.1 Unit-return use case

```kotlin
/**
 * Обновляет профиль пользователя.
 */
interface UserProfileRefreshUseCase {

    /**
     * Выполняет обновление профиля пользователя.
     */
    suspend fun execute(
        userId: String,
    ): Unit
}

/**
 * Реализация обновления профиля пользователя.
 *
 * Сценарий оркестрирует удаленную загрузку и локальное сохранение.
 * Использует IO dispatcher, так как выполняются операции чтения/записи данных.
 */
internal class UserProfileRefreshUseCaseImpl(
    private val dispatcherProvider: DispatcherProvider,
    private val profileRepository: ProfileRepository,
    private val profileCache: ProfileCache,
) : UserProfileRefreshUseCase {

    /**
     * Загружает профиль пользователя и сохраняет его в кэш.
     */
    override suspend fun execute(
        userId: String,
    ): Unit {
        return withContext(dispatcherProvider.IO) {
            val profile = profileRepository.loadProfile(
                userId = userId,
            )

            profileCache.save(
                profile = profile,
            )

            Unit
        }
    }
}
```

### 17.2 Multi-branch orchestration template

```kotlin
/**
 * Загружает профиль пользователя с учетом политики обновления.
 */
interface UserProfileLoadUseCase {

    /**
     * Выполняет загрузку профиля пользователя.
     */
    suspend fun execute(
        userId: String,
        forceRefresh: Boolean,
    ): UserProfile
}

/**
 * Реализация загрузки профиля пользователя.
 *
 * Если forceRefresh = true, use case запрашивает удаленные данные,
 * сохраняет их локально и возвращает актуальный профиль.
 * Если forceRefresh = false, use case использует локальный источник.
 *
 * Основная логика выполняется в IO dispatcher, так как сценарий
 * сочетает сетевые и локальные операции.
 */
internal class UserProfileLoadUseCaseImpl(
    private val dispatcherProvider: DispatcherProvider,
    private val profileRepository: ProfileRepository,
    private val profileCache: ProfileCache,
    private val analyticsTracker: AnalyticsTracker,
) : UserProfileLoadUseCase {

    /**
     * Выполняет ветвление сценария загрузки профиля
     * по признаку принудительного обновления.
     */
    override suspend fun execute(
        userId: String,
        forceRefresh: Boolean,
    ): UserProfile {
        return withContext(dispatcherProvider.IO) {
            if (forceRefresh) {
                val remoteProfile = profileRepository.fetchRemoteProfile(
                    userId = userId,
                )

                profileCache.save(
                    profile = remoteProfile,
                )

                analyticsTracker.trackProfileRefresh(
                    userId = userId,
                )

                return@withContext remoteProfile
            }

            val cachedProfile = profileCache.get(
                userId = userId,
            )

            return@withContext cachedProfile
        }
    }
}
```

---

## 18) Bad Examples (Do Not Generate)

Wrong:
```kotlin
interface XUseCase {
    suspend operator fun invoke(id: String): User
}
```

Wrong:
```kotlin
class XUseCaseImpl(...) : XUseCase // Impl must be internal
```

Wrong:
```kotlin
override suspend fun execute(id: String): Flow<User> // Flow is forbidden
```

Wrong:
```kotlin
override suspend fun execute(id: String): User {
    return try {
        ...
    } catch (e: Exception) {
        ...
    }
}
```
(if try/catch was not explicitly required by contract)

Wrong:
```kotlin
override suspend fun execute(id: String): User {
    return withContext(dispatcherProvider.Main) { ... }
}
```

---

## 19) Validation Checklist

Before returning output, verify:
- code is client-side Android/KMP only
- use case name matches FeatureActionUseCase
- impl name matches FeatureActionUseCaseImpl
- interface is public, impl is internal
- exactly one public method execute
- execute is always suspend
- no operator invoke
- DispatcherProvider + withContext is used
- dispatcher is correct (IO default / Default CPU-only / no Main)
- no Flow
- no extra entities (repo/mapper/datasource/DI)
- no try/catch without explicit reason
- named arguments are always used
- multiline rules and trailing comma are respected
- KDoc is in Russian (short in interface, detailed in implementation)
- architectural neutrality is preserved (no forced package/layer structure)
