# Kotlin Safe Model Mapper Examples

This file preserves the extended examples used by the skill, including good/bad snippets, templates, and edge-case handling.

## 1) Hard function rule examples

Forbidden (expression body):

```kotlin
internal fun UserResponse.toData(): User = User(...)
```

Required (block body):

```kotlin
internal fun UserResponse.toData(): User {
    val id = id ?: error("User.id is required")

    return User(
        id = id,
    )
}
```

## 2) Trailing comma rule examples

Required:

```kotlin
data class User(
    val id: String,
    val name: PersonName,
    val avatarUrl: String?,
)
```

Forbidden:

```kotlin
data class User(
    val id: String,
    val name: PersonName,
    val avatarUrl: String?
)
```

## 3) Mapper contract examples

Forbidden (two root mappers in one file):

```kotlin
internal fun UserResponse.toData(): User { ... }
internal fun ChatResponse.toData(): Chat { ... }
```

Required (one root mapper per file):

```kotlin
internal fun UserResponse.toData(): User { ... }

private fun AddressResponse?.toData(): Address? { ... }
```

## 4) Collection policy examples

Required:

```kotlin
val items: List<Item>
```

Forbidden:

```kotlin
val items: List<Item>?
```

## 5) Aggregation and preservation examples

Required:

```kotlin
data class User(
    val name: PersonName,
)

data class PersonName(
    val firstName: String,
    val lastName: String,
    val fullName: String,
)
```

Forbidden:

```kotlin
data class User(
    val fullName: String,
)
```

## 6) Model template

```kotlin
data class Entity(
    val id: String,
    val title: String,
    val details: EntityDetails?,
    val items: List<EntityItem>,
)

data class EntityDetails(
    val rawValue: String?,
    val normalizedValue: String?,
)

data class EntityItem(
    val code: String,
    val label: String?,
)
```

## 7) Mapper template

```kotlin
internal fun SourceResponse.toData(): Entity {
    val id = id?.trim()?.takeIf { it.isNotEmpty() } ?: error("SourceResponse.id is required")
    val title = normalizeOptionalText(
        raw = title,
        fallback = "Untitled",
    )
    val details = details.toData()
    val items = items
        .orEmpty()
        .mapNotNull { it.toDataOrNull() }

    return Entity(
        id = id,
        title = title,
        details = details,
        items = items,
    )
}

private fun SourceDetailsResponse?.toData(): EntityDetails? {
    if (this == null) {
        return null
    }

    val normalizedValue = rawValue
        ?.trim()
        ?.takeIf { it.isNotEmpty() }

    return EntityDetails(
        rawValue = rawValue,
        normalizedValue = normalizedValue,
    )
}

private fun SourceItemResponse?.toDataOrNull(): EntityItem? {
    if (this == null) {
        return null
    }

    val code = code
        ?.trim()
        ?.takeIf { it.isNotEmpty() }
        ?: return null

    val label = label
        ?.trim()
        ?.takeIf { it.isNotEmpty() }

    return EntityItem(
        code = code,
        label = label,
    )
}

private fun normalizeOptionalText(
    raw: String?,
    fallback: String,
): String {
    val normalized = raw
        ?.trim()
        ?.takeIf { it.isNotEmpty() }
        ?: fallback

    return normalized
}
```

## 8) Aggregation template

```kotlin
data class PersonName(
    val firstName: String,
    val lastName: String,
    val fullName: String,
)

private fun buildPersonName(
    firstNameRaw: String?,
    lastNameRaw: String?,
): PersonName {
    val firstName = firstNameRaw
        ?.trim()
        ?.takeIf { it.isNotEmpty() }
        ?: error("firstName is required")

    val lastName = lastNameRaw
        ?.trim()
        ?.takeIf { it.isNotEmpty() }
        ?: error("lastName is required")

    val fullName = "$firstName $lastName"

    return PersonName(
        firstName = firstName,
        lastName = lastName,
        fullName = fullName,
    )
}
```

## 9) toRequest template

```kotlin
internal fun Entity.toRequest(): EntityRequest {
    val request = EntityRequest(
        id = id,
        title = title,
        details = details?.toRequest(),
        items = items.map { it.toRequest() },
    )

    return request
}

private fun EntityDetails.toRequest(): EntityDetailsRequest {
    val request = EntityDetailsRequest(
        rawValue = rawValue,
        normalizedValue = normalizedValue,
    )

    return request
}

private fun EntityItem.toRequest(): EntityItemRequest {
    val request = EntityItemRequest(
        code = code,
        label = label,
    )

    return request
}
```

## 10) Full good example

```kotlin
data class User(
    val id: String,
    val name: PersonName,
    val email: String?,
    val phones: List<Phone>,
    val addresses: List<Address>,
)

data class PersonName(
    val firstName: String,
    val lastName: String,
    val fullName: String,
)

data class Phone(
    val countryCodeRaw: String?,
    val numberRaw: String?,
    val normalized: String?,
)

data class Address(
    val city: String,
    val street: String?,
    val postalCode: String?,
)

internal fun UserResponse.toData(): User {
    val id = id?.trim()?.takeIf { it.isNotEmpty() } ?: error("UserResponse.id is required")
    val name = buildPersonName(
        firstNameRaw = firstName,
        lastNameRaw = lastName,
    )
    val email = email
        ?.trim()
        ?.takeIf { it.isNotEmpty() }
    val phones = phones
        .orEmpty()
        .mapNotNull { it.toDataOrNull() }
    val addresses = addresses
        .orEmpty()
        .mapNotNull { it.toDataOrNull() }

    return User(
        id = id,
        name = name,
        email = email,
        phones = phones,
        addresses = addresses,
    )
}

private fun PhoneResponse?.toDataOrNull(): Phone? {
    if (this == null) {
        return null
    }

    val countryCodeRaw = countryCode
        ?.trim()
        ?.takeIf { it.isNotEmpty() }
    val numberRaw = number
        ?.trim()
        ?.takeIf { it.isNotEmpty() }
    val normalized = buildPhoneNormalized(
        countryCodeRaw = countryCodeRaw,
        numberRaw = numberRaw,
    )

    if (countryCodeRaw == null && numberRaw == null) {
        return null
    }

    return Phone(
        countryCodeRaw = countryCodeRaw,
        numberRaw = numberRaw,
        normalized = normalized,
    )
}

private fun AddressResponse?.toDataOrNull(): Address? {
    if (this == null) {
        return null
    }

    val city = city
        ?.trim()
        ?.takeIf { it.isNotEmpty() }
        ?: return null
    val street = street
        ?.trim()
        ?.takeIf { it.isNotEmpty() }
    val postalCode = postalCode
        ?.trim()
        ?.takeIf { it.isNotEmpty() }

    return Address(
        city = city,
        street = street,
        postalCode = postalCode,
    )
}

private fun buildPersonName(
    firstNameRaw: String?,
    lastNameRaw: String?,
): PersonName {
    val firstName = firstNameRaw
        ?.trim()
        ?.takeIf { it.isNotEmpty() }
        ?: error("UserResponse.firstName is required")
    val lastName = lastNameRaw
        ?.trim()
        ?.takeIf { it.isNotEmpty() }
        ?: error("UserResponse.lastName is required")
    val fullName = "$firstName $lastName"

    return PersonName(
        firstName = firstName,
        lastName = lastName,
        fullName = fullName,
    )
}

private fun buildPhoneNormalized(
    countryCodeRaw: String?,
    numberRaw: String?,
): String? {
    val countryCode = countryCodeRaw
        ?.trim()
        ?.takeIf { it.isNotEmpty() }
    val number = numberRaw
        ?.trim()
        ?.takeIf { it.isNotEmpty() }

    if (countryCode == null && number == null) {
        return null
    }

    val normalized = listOfNotNull(countryCode, number).joinToString(separator = " ")

    return normalized
}
```

## 11) Bad examples

Bad: missing trailing comma.

```kotlin
data class User(
    val id: String
)
```

Bad: expression body.

```kotlin
internal fun UserResponse.toData(): User = User(id = id.orEmpty())
```

Bad: blind nullable propagation.

```kotlin
data class User(
    val items: List<Item>?,
)
```

Bad: field loss during aggregation.

```kotlin
data class User(
    val fullName: String,
)
```

Bad: architecture dependency inside mapper contract.

```kotlin
internal fun UserResponse.toData(repository: UserRepository): User {
    // forbidden: architecture-coupled dependency
}
```

## 12) Edge case handling

Nullable nested model:

```kotlin
private fun MetaResponse?.toDataOrNull(): Meta? {
    if (this == null) {
        return null
    }

    val code = code?.trim()?.takeIf { it.isNotEmpty() } ?: return null

    return Meta(
        code = code,
    )
}
```

Empty list from transport:

```kotlin
val items = items
    .orEmpty()
    .mapNotNull { it.toDataOrNull() }
```

Invalid nested elements:

```kotlin
private fun ItemResponse?.toDataOrNull(): Item? {
    if (this == null) {
        return null
    }

    val id = id?.trim()?.takeIf { it.isNotEmpty() } ?: return null

    return Item(
        id = id,
    )
}
```

Aggregation fallback:

```kotlin
private fun buildPeriod(
    startRaw: String?,
    endRaw: String?,
): Period? {
    val start = startRaw?.trim()?.takeIf { it.isNotEmpty() }
    val end = endRaw?.trim()?.takeIf { it.isNotEmpty() }

    if (start == null && end == null) {
        return null
    }

    val label = listOfNotNull(start, end).joinToString(separator = " - ")

    return Period(
        startRaw = startRaw,
        endRaw = endRaw,
        label = label,
    )
}
```
