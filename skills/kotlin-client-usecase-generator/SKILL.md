---
name: kotlin-client-usecase-generator
description: Generate client-side Kotlin UseCase interface and internal implementation for Android/Kotlin Multiplatform apps: enforce execute-only suspend contract, DispatcherProvider with withContext (IO by default, Default only for CPU-bound work), orchestration business logic with optional private helpers, Russian KDoc, strict naming/formatting rules, and deterministic templates/examples without repositories/mappers/DI generation.
metadata:
  short-description: Kotlin client use case generator
---

# Kotlin Client UseCase Generator (Android / KMP client)

Генерирует use case только для клиентских Kotlin-приложений:
- Android
- Kotlin Multiplatform (client side)

Никогда не генерирует backend/server код.

---

## 1) Назначение

Скил генерирует:
- UseCase interface
- UseCaseImpl implementation
- зависимости конструктора (только необходимые)
- orchestration бизнес-логику
- execute метод
- private helper methods (при необходимости)
- KDoc (на русском языке)

---

## 2) Scope

### Генерируемые сущности
- UseCase interface
- UseCaseImpl implementation
- execute method
- private helper methods
- KDoc документацию

### Не генерируется
- repository
- datasource
- mapper
- DI конфигурация
- module wiring
- Flow-based use case
- архитектурное размещение
- дополнительные abstraction layers

---

## 3) Naming Rules

### UseCase naming
Использовать:
- FeatureActionUseCase
- FeatureActionUseCaseImpl

Примеры:
- AuthOtpVerifyUseCase
- AuthOtpVerifyUseCaseImpl
- UserProfileLoadUseCase
- UserProfileLoadUseCaseImpl

### Dependency naming
- lowerCamelCase
- имя переменной от типа

Примеры:
- dispatcherProvider
- authRepository
- authTokenSaver
- profileRepository
- sessionManager
- analyticsTracker

Запрещенные generic-имена:
- repository
- manager
- data
- repo
- helper
- util

Пример:
- тип AuthRepository -> имя authRepository

---

## 4) Contract Rules

### Interface contract
- Ровно один публичный метод.
- Имя метода только execute.
- Сигнатура только suspend fun execute(...).
- Дополнительный публичный API запрещен.

### execute method
- Всегда suspend.
- Возвращаемый тип: любой по контракту задачи (Unit, model, primitive, nullable, sealed result и т.д.).
- operator fun invoke запрещен.
- Разрешен только execute.

---

## 5) Implementation Rules

### Visibility
- interface — public
- implementation — internal

Пример:
```kotlin
internal class AuthOtpVerifyUseCaseImpl(
    private val dispatcherProvider: DispatcherProvider,
    private val authRepository: AuthRepository,
) : AuthOtpVerifyUseCase
```

### Public API
Запрещено:
- дополнительные публичные методы
- дополнительные entry points
- любые public/internal API кроме execute

Разрешено:
- private methods

---

## 6) Dispatcher Policy (Hard Rule)

Каждый use case обязан:
- использовать DispatcherProvider
- выполнять основную логику внутри withContext(...)

Dispatcher selection:
- по умолчанию: dispatcherProvider.IO
- dispatcherProvider.Default только для явно вычислительных задач
- dispatcherProvider.Main запрещен

---

## 7) Business Logic Rules

Use case — orchestration unit, не прокси.

Допустимо:
- использовать несколько repository
- использовать зависимости разных feature
- читать/писать данные
- выполнять post-processing
- сохранять локальные данные
- агрегировать источники
- ветвить логику (if, when, early return)
- валидировать входные параметры
- мультишаговые сценарии

Side-effects допустимы:
- сохранение токенов
- обновление кэша
- запись локальных данных
- связка remote + local

---

## 8) Error Handling Policy

По умолчанию запрещено:
- самовольно добавлять try/catch
- самовольно менять error contract
- самовольно вводить Result wrapper

Разрешено:
- try/catch только когда явно требуется задачей/контрактом

---

## 9) Private Methods Policy

Private methods:
- разрешены при смысловой пользе
- могут быть suspend
- могут содержать бизнес-логику
- могут использовать named arguments

Не создавать private methods:
- ради структуры
- ради красоты
- без логической необходимости

---

## 10) Logic Composition Rules

### also usage
Разрешено только для простой одношаговой post-processing логики:

```kotlin
repository.verify(
    code = code,
).also { saveTokens(result = it) }
```

### Local variable usage
Обязательно, если логика сложнее одного шага:

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

## 11) KDoc Policy (на русском)

Обязательно:
- краткий KDoc у interface
- краткий KDoc у execute в interface
- развернутый KDoc у implementation и execute в implementation

Implementation KDoc должен описывать:
- что делает use case
- как работает логика
- почему выбран конкретный dispatcher
- что делают private methods (если есть)

---

## 12) Formatting Specification (Hard Rules)

### Parameter declaration
- если параметр 1: можно в одну строку
- если параметров 2+: только перенос строк

Пример:
```kotlin
suspend fun execute(
    code: String,
    sessionId: String,
): Result
```

### Named arguments
Всегда использовать named arguments в вызовах:
- даже если аргумент один

```kotlin
repository.loadUser(
    userId = userId,
)
```

### Trailing comma
Обязательна в multiline:
- параметры
- аргументы
- конструкторы

### Constructor formatting
Если зависимостей 2+, только вертикальный список:

```kotlin
internal class ExampleUseCaseImpl(
    private val dispatcherProvider: DispatcherProvider,
    private val repository: ExampleRepository,
) : ExampleUseCase
```

---

## 13) Abstraction Restrictions

Запрещено:
- лишние abstraction layers
- wrapper без необходимости
- factory без причины
- executor без причины
- helper-классы без задачи

---

## 14) Architectural Neutrality

Скил:
- не определяет архитектуру
- не определяет пакеты
- не определяет слои
- не распределяет файлы по структуре проекта

---

## 15) Explicit Restrictions (Never)

Скил никогда не должен:
- использовать operator invoke
- делать Impl публичным
- добавлять дополнительный public API
- добавлять Flow
- добавлять try/catch без причины
- изменять return contract
- создавать лишние сущности
- размещать файлы по пакетам/слоям по собственной инициативе

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

Неправильно:
```kotlin
interface XUseCase {
    suspend operator fun invoke(id: String): User
}
```

Неправильно:
```kotlin
class XUseCaseImpl(...) : XUseCase // Impl должен быть internal
```

Неправильно:
```kotlin
override suspend fun execute(id: String): Flow<User> // Flow запрещен
```

Неправильно:
```kotlin
override suspend fun execute(id: String): User {
    return try {
        ...
    } catch (e: Exception) {
        ...
    }
}
```
(если try/catch не был явно запрошен контрактом)

Неправильно:
```kotlin
override suspend fun execute(id: String): User {
    return withContext(dispatcherProvider.Main) { ... }
}
```

---

## 19) Validation Checklist

Перед выдачей результата проверить:
- код только для client-side Android/KMP
- use case name соответствует FeatureActionUseCase
- impl name соответствует FeatureActionUseCaseImpl
- interface public, impl internal
- ровно один public метод execute
- execute всегда suspend
- нет operator invoke
- используется DispatcherProvider + withContext
- выбран корректный dispatcher (IO default / Default CPU-only / no Main)
- нет Flow
- нет лишних сущностей (repo/mapper/datasource/DI)
- нет try/catch без явной причины
- named arguments используются всегда
- multiline правила и trailing comma соблюдены
- KDoc на русском: краткий у interface, развернутый у implementation
- архитектурная нейтральность соблюдена (без навязывания пакетов/слоев)
