# Kotlin Client Repository Generator - Extended Examples

This file preserves the full code examples/templates for repository generation scenarios.

## 1) Interface Template

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

## 2) DAO-only Template

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

## 3) HTTP-only Template

```kotlin
package com.example.feature.data.repository

import com.example.feature.data.mapper.toData
import com.example.feature.data.remote.model.UserResponse
import com.example.feature.domain.model.User
import io.ktor.client.HttpClient
import io.ktor.client.call.body
import io.ktor.client.request.get

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

## 4) DAO + HTTP Template

```kotlin
package com.example.feature.data.repository

import com.example.feature.data.local.UserDao
import com.example.feature.data.mapper.toData
import com.example.feature.data.mapper.toEntity
import com.example.feature.data.remote.model.UserResponse
import com.example.feature.domain.model.User
import io.ktor.client.HttpClient
import io.ktor.client.call.body
import io.ktor.client.request.get

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

## 5) Flow Template

```kotlin
package com.example.feature.data.repository

import com.example.feature.data.local.UserDao
import com.example.feature.data.mapper.toData
import com.example.feature.domain.model.User
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map

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

## 6) HTTP 204 / No Body Template

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

## 7) Focused Scenario Snippets

DAO-only snippet:

```kotlin
override suspend fun getUsers(): List<User> {
    return userDao
        .getUsers()
        .toData()
}
```

HTTP-only snippet:

```kotlin
override suspend fun getUsers(): List<User> {
    return httpClient
        .get("url")
        .body<UserResponse>()
        .toData()
}
```

DAO+HTTP snippet:

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

Flow snippet:

```kotlin
override fun observeUsers(): Flow<List<User>> {
    return userDao
        .observeUsers()
        .map {
            it.toData()
        }
}
```

HTTP 204 snippet:

```kotlin
httpClient.post("url") {
    setBody(body)
}
```

## 8) Negative Examples (Do Not Generate)

Unused dependency:

```kotlin
class UserRepositoryImpl(
    private val userDao: UserDao,
    private val httpClient: HttpClient, // unused
) : UserRepository
```

Expression body (forbidden):

```kotlin
override suspend fun getUsers(): List<User> = userDao.getUsers().toData()
```

Interface leaking transport/storage model (forbidden):

```kotlin
interface UserRepository {
    suspend fun getUsers(): UserResponse
}
```
