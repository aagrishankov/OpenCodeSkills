# Kotlin Client UseCase Examples

## 1) Canonical template example

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

## 2) Unit-return use case

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

## 3) Multi-branch orchestration

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

## 4) Bad examples (do not generate)

Wrong (`invoke` is forbidden):

```kotlin
interface XUseCase {
    suspend operator fun invoke(id: String): User
}
```

Wrong (Impl must be `internal`):

```kotlin
class XUseCaseImpl(...) : XUseCase
```

Wrong (`Flow` is forbidden):

```kotlin
override suspend fun execute(id: String): Flow<User>
```

Wrong (try/catch without explicit contract need):

```kotlin
override suspend fun execute(id: String): User {
    return try {
        ...
    } catch (e: Exception) {
        ...
    }
}
```

Wrong (`dispatcherProvider.Main` is forbidden):

```kotlin
override suspend fun execute(id: String): User {
    return withContext(dispatcherProvider.Main) { ... }
}
```

## 5) Fixed direction for common failures

- Replace `operator invoke` with single public `suspend fun execute(...)`.
- Keep implementation visibility as `internal`.
- Replace `Flow` contracts with explicit task-defined return type.
- Use `withContext(dispatcherProvider.IO)` by default.
- Add try/catch only when explicitly required by task contract.
