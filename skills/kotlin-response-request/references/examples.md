# Kotlin Request/Response DTO Examples

## 1) Happy-path example (input -> output)

Input JSON:

```json
{
  "id": "u-1",
  "profile": {
    "email": "dev@example.com"
  },
  "tags": ["a", "b"]
}
```

Expected output:

```kotlin
import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class UserResponse(
    @SerialName("id")
    val id: String? = null,
    @SerialName("profile")
    val profile: ProfileResponse? = null,
    @SerialName("tags")
    val tags: List<String>? = null,
) {
    @Serializable
    data class ProfileResponse(
        @SerialName("email")
        val email: String? = null,
    )
}
```

Why valid:
- One root per file.
- One-level nesting only.
- `@Serializable` on each class.
- `@SerialName` on each field.
- All fields are `Type? = null`.

## 2) Bad example (violates rules)

```kotlin
@Serializable
class UserDto(
    val id: String,
    val profile: Profile,
)
```

Why invalid:
- Not a `data class`.
- Missing `Request`/`Response` suffix.
- Missing `@SerialName` annotations.
- Fields are not nullable and not optional.

## 3) Fixed version of bad example

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
        @SerialName("name")
        val name: String? = null,
    )
}
```

## 4) Edge case with explicit fallback

Input JSON (deep nesting):

```json
{
  "user": {
    "profile": {
      "details": {
        "nickname": "neo"
      }
    }
  }
}
```

Fallback behavior:
- `nested -> nested` is forbidden.
- Flatten illegal depth to root-level direct nested classes or split into multiple root files.

Compliant flattened output:

```kotlin
import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class UserResponse(
    @SerialName("user")
    val user: UserNodeResponse? = null,
    @SerialName("profile")
    val profile: ProfileNodeResponse? = null,
    @SerialName("details")
    val details: DetailsNodeResponse? = null,
) {
    @Serializable
    data class UserNodeResponse(
        @SerialName("profile")
        val profile: String? = null,
    )

    @Serializable
    data class ProfileNodeResponse(
        @SerialName("details")
        val details: String? = null,
    )

    @Serializable
    data class DetailsNodeResponse(
        @SerialName("nickname")
        val nickname: String? = null,
    )
}
```

## 5) Trigger-boundary example (should not trigger)

Prompt:

```text
Generate safe domain models and mappers from these DTOs.
```

Expected handling:
- Do not trigger this skill.
- Route to a safe-model/mapper skill instead.
