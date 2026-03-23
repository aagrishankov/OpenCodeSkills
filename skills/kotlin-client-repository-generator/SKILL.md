---
name: kotlin-client-repository-generator
description: Generate compile-ready Kotlin repository interfaces and implementations for client applications (Android and Kotlin Multiplatform): create Repository/RepositoryImpl, repository methods, only-used dependencies (featureDao, httpClient), mapper calls toData()/toRequest()/toEntity(), required KDoc in Russian, imports/package, and deterministic templates for DAO-only, HTTP-only, DAO+HTTP, Flow, and 204-no-body scenarios. Trigger for prompts about generating or updating repository layer code for client-side Kotlin apps.
metadata:
  short-description: Kotlin client repository generator
---

# Kotlin Client Repository Generator (Android / KMP)

Generates repository layer only for **client Kotlin applications**:
- Android
- Kotlin Multiplatform (client side)

Never generate backend/server code.

## 1) Responsibility Scope

This skill is responsible only for:
- Repository interface
- Repository implementation
- Data-source interaction
- Mapping usage
- KDoc generation
- Imports generation
- Full compile-ready Kotlin file (`package + imports + interface + implementation`)

## 2) Out Of Scope

Do not do:
- architecture file placement
- layer selection (data/domain/presentation)
- DTO/Entity/safe model generation
- mapper function generation
- DI configuration
- dispatcher/threading management
- cache policy / retry / fallback
- paging
- websocket-specific logic
- multipart upload
- expect/actual
- server-side implementations (Ktor Server, Spring, etc.)

## 3) Naming Rules

### Repository naming
- `FeatureRepository`
- `FeatureRepositoryImpl`

Example:
- `UserRepository`
- `UserRepositoryImpl`

### DAO dependency naming
- format: `featureDao`
- examples: `userDao`, `orderDao`, `messageDao`

### HTTP dependency naming
- always: `httpClient`

## 4) Dependency Injection Rule

Add only actually used dependencies.

Allowed:
```kotlin
class UserRepositoryImpl(
    private val userDao: UserDao,
) : UserRepository
```

Forbidden:
```kotlin
class UserRepositoryImpl(
    private val userDao: UserDao,
    private val httpClient: HttpClient, // unused
) : UserRepository
```

## 5) Model Contract Rule

Repository works only with safe models.

Forbidden in interface:
- `DTO`
- `Request`
- `Response`
- `Entity`

Allowed in interface:
- `User`
- `Order`
- `Message`
- `Result` (only if explicitly required by input contract)

## 6) Mapper Usage Rules

Use only existing mapper calls:
- `Response.toData()`
- `Data.toRequest()`
- `Data.toEntity()`
- `Entity.toData()`

Never generate:
- DTO/Entity/Request/Response
- mapper functions
- safe models

## 7) List Mapping Rules

If list mapper exists:
```kotlin
List<Entity>.toData()
```
use it.

If not, use:
```kotlin
.map { it.toEntity() }
```
or
```kotlin
.map { it.toData() }
```

Inline mapping is allowed, but intermediate variable is preferred.

## 8) Supported Scenarios

### DAO-only
```kotlin
override suspend fun getUsers(): List<User> {
    return userDao
        .getUsers()
        .toData()
}
```

### HTTP-only
```kotlin
override suspend fun getUsers(): List<User> {
    return httpClient
        .get("url")
        .body<UserResponse>()
        .toData()
}
```

### DAO + HTTP (only on explicit scenario)
```kotlin
override suspend fun getUsers(
    isForce: Boolean,
): List<User> {
    if (isForce) {
        val users = httpClient
            .get("url")
            .body<UserResponse>()
            .toData()

        val entities = users.map {
            it.toEntity()
        }

        userDao.saveUsers(entities)

        return users
    }

    return userDao
        .getUsers()
        .toData()
}
```

## 9) HTTP 204 / No Body Rule

If response body is absent, do not call `.body<Unit>()`.

Correct:
```kotlin
httpClient.post("url") {
    setBody(body)
}
```

## 10) Flow Rule

Use `Flow` only when explicitly required by contract.

```kotlin
override fun observeUsers(): Flow<List<User>> {
    return userDao
        .observeUsers()
        .map {
            it.toData()
        }
}
```

Paging/WebSocket are not supported by this skill.

## 11) Error Handling Rule

By default, do not add error handling.

Automatically forbidden:
- `runCatching`
- `try/catch`
- forced `Result` wrapping

Add only when explicitly requested.

## 12) Threading Rule

Forbidden:
- `withContext(...)`
- `Dispatchers`

Threading belongs to use-case layer.

## 13) Helper Functions Rule

Private helper functions are allowed if they reduce duplication.

```kotlin
private suspend fun fetchUsers(): List<User> {
    return httpClient
        .get("url")
        .body<UserResponse>()
        .toData()
}
```

## 14) KDoc Rule (Mandatory)

### Interface KDoc is short
```kotlin
/**
 * Получает список пользователей.
 */
suspend fun getUsers(): List<User>
```

### Implementation KDoc is detailed and in Russian
Must describe:
- what it does
- how it works
- data source
- return value

Example:
```kotlin
/**
 * Получает список пользователей.
 *
 * Выполняет запрос через httpClient.
 * Ответ преобразуется из UserResponse
 * в список User через mapper toData().
 *
 * @return список пользователей
 */
```

## 15) Formatting Rules (Hard)

- Expression body is forbidden:
  - do not use `fun x(): T = ...`
  - use block body only `{ return ... }`
- If parameters count is >= 2, always place each parameter on a new line.
- Trailing comma is mandatory in all multiline structures:
  - function params
  - constructors
  - calls
- Constructor format:
```kotlin
class UserRepositoryImpl(
    private val userDao: UserDao,
    private val httpClient: HttpClient,
) : UserRepository
```

## 16) Forbidden Constructs

- DTO/Entity/Request/Response in interface
- unused constructor dependencies
- `withContext`, `Dispatchers`
- DI annotations (`@Inject`, `@Singleton`, `@Factory`)
- region comments

## 17) Batch Operations

Allowed:
- `saveAll`
- `deleteAll`
- `upsertAll`
- `replaceAll`

## 18) Nullability Rule

Keep nullability exactly as defined by input contract.
Do not change nullability implicitly.

## 19) Supported Return Types

- `T`
- `List<T>`
- `Flow<T>`
- `Unit`
- `Result<T>` (only when explicitly required)

## 20) Output Contract

Always output compile-ready Kotlin including:
- `package`
- `imports`
- `interface`
- `implementation`

## 21) Templates

### 21.1 Interface template
```kotlin
package com.example.feature.data.repository

import com.example.feature.domain.model.User

interface UserRepository {

    /**
     * Получает список пользователей.
     */
    suspend fun getUsers(): List<User>

    /**
     * Обновляет пользователя.
     */
    suspend fun updateUser(
        user: User,
    ): Unit
}
```

### 21.2 DAO-only template
```kotlin
package com.example.feature.data.repository

import com.example.feature.data.local.UserDao
import com.example.feature.data.mapper.toData
import com.example.feature.domain.model.User

class UserRepositoryImpl(
    private val userDao: UserDao,
) : UserRepository {

    /**
     * Получает список пользователей.
     *
     * Берет данные из локального источника userDao.
     * Результат в виде List<UserEntity> преобразует в List<User>
     * через существующий list-mapper toData().
     *
     * @return список пользователей
     */
    override suspend fun getUsers(): List<User> {
        return userDao
            .getUsers()
            .toData()
    }
}
```

### 21.3 HTTP-only template
```kotlin
package com.example.feature.data.repository

import io.ktor.client.HttpClient
import io.ktor.client.call.body
import io.ktor.client.request.get
import com.example.feature.data.remote.model.UserResponse
import com.example.feature.data.mapper.toData
import com.example.feature.domain.model.User

class UserRepositoryImpl(
    private val httpClient: HttpClient,
) : UserRepository {

    /**
     * Получает список пользователей.
     *
     * Выполняет HTTP-запрос через httpClient.
     * Полученный UserResponse преобразует в List<User>
     * через mapper toData().
     *
     * @return список пользователей
     */
    override suspend fun getUsers(): List<User> {
        return httpClient
            .get("users")
            .body<UserResponse>()
            .toData()
    }
}
```

### 21.4 DAO + HTTP template
```kotlin
package com.example.feature.data.repository

import io.ktor.client.HttpClient
import io.ktor.client.call.body
import io.ktor.client.request.get
import com.example.feature.data.local.UserDao
import com.example.feature.data.mapper.toData
import com.example.feature.data.mapper.toEntity
import com.example.feature.data.remote.model.UserResponse
import com.example.feature.domain.model.User

class UserRepositoryImpl(
    private val userDao: UserDao,
    private val httpClient: HttpClient,
) : UserRepository {

    /**
     * Получает список пользователей.
     *
     * Если isForce = true, запрашивает данные из сети через httpClient,
     * преобразует ответ в safe-модель User и сохраняет результат в userDao.
     * Если isForce = false, читает данные из userDao и преобразует их в User.
     *
     * @param isForce признак принудительного обновления из сети
     * @return список пользователей
     */
    override suspend fun getUsers(
        isForce: Boolean,
    ): List<User> {
        if (isForce) {
            val users = httpClient
                .get("users")
                .body<UserResponse>()
                .toData()

            val entities = users.map {
                it.toEntity()
            }

            userDao.saveAll(entities)

            return users
        }

        return userDao
            .getUsers()
            .toData()
    }
}
```

### 21.5 Flow template
```kotlin
package com.example.feature.data.repository

import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map
import com.example.feature.data.local.UserDao
import com.example.feature.data.mapper.toData
import com.example.feature.domain.model.User

class UserRepositoryImpl(
    private val userDao: UserDao,
) : UserRepository {

    /**
     * Наблюдает за списком пользователей.
     *
     * Подписывается на поток данных из userDao.
     * Каждый элемент потока преобразует в safe-модель User
     * через mapper toData().
     *
     * @return поток списка пользователей
     */
    override fun observeUsers(): Flow<List<User>> {
        return userDao
            .observeUsers()
            .map {
                it.toData()
            }
    }
}
```

### 21.6 HTTP 204 template
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

## 22) Validation Checklist

Before returning output, verify:
- interface contains only safe models
- interface has no DTO/Entity/Request/Response
- implementation implements interface
- dependencies are only used ones
- only existing mapper calls are used
- every function has KDoc
- implementation KDoc is detailed and in Russian
- no expression bodies
- params >= 2 are multiline
- trailing comma is present
- imports are present
- output is compile-ready
- generated code is only for client Android/KMP scenarios
