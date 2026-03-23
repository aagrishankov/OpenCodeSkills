---
name: kotlin-safe-model-mapper
description: Generate safe Kotlin application models and deterministic mappers from unsafe transport models: create architecture-agnostic safe models, implement internal root mappers toData()/toRequest() with block bodies only, enforce trailing commas, normalization, aggregation with source-data preservation, private nested helpers, one root mapper per file, and explicit validation checklist with good/bad examples and edge-case templates.
metadata:
  short-description: Safe model and mapper generator
---

# Safe Model & Mapper Generator (Kotlin, architecture-agnostic)

Generate deterministic, reusable, safe-by-default Kotlin models and mapper functions for converting unsafe transport models into safe application models.

This skill must **not** generate transport models (`Request`/`Response`/`DTO` classes).
Transport models are assumed to be provided by another skill.

## Global Constraints

- Architecture-agnostic only.
- No assumptions about:
  - Clean Architecture, MVVM, MVI.
  - data/domain/presentation layers.
  - repositories, use-cases.
  - package/module naming.
  - framework/UI/network/serialization stack.
- Deterministic output.
- Predictable structure and formatting.
- Safe-by-default modeling.

## Hard Function Rules

- All functions MUST use full block bodies with `{}`.
- Expression bodies with `=` are forbidden.

Forbidden:
```kotlin
internal fun UserResponse.toData(): User = User(...)
```

Required:
```kotlin
internal fun UserResponse.toData(): User {
    val id = id ?: error("User.id is required")

    return User(
        id = id,
    )
}
```

## Trailing Comma Rule (Hard Requirement)

- Trailing commas are mandatory in every multiline structure:
  - data classes,
  - constructors,
  - function parameters,
  - function calls,
  - collections,
  - enums,
  - nested objects.

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

## Naming Rules

- Safe model names represent entities only.
- Do not use suffixes or layer markers.

Forbidden:
- `UserModel`
- `UserData`
- `UserDomain`
- `UserEntity`

Required:
- `User`
- `Chat`
- `Message`
- `Profile`

## Mapper Contract Rules

- Allowed root mapper names:
  - `toData()`
  - `toRequest()`
- No other root mapper names allowed.
- Root mapper visibility: `internal`.
- Nested helper visibility: `private`.
- Exactly one root mapper per mapper file.
- Nested helpers are allowed in the same file.

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

## Safe Model Rules

Safe model must:
- be usable without constant null checks,
- not mirror transport shape blindly,
- avoid nullable propagation caused by transport weakness,
- preserve source data when aggregating,
- allow derived values for application use.

## Nullability Policy

- Nullable in safe model must represent business meaning only.
- `transport nullable != safe nullable`.
- Nullable must be intentional, never automatic.

## Collection Policy

- Collections are non-null by default.
- Nullable collections allowed only with explicit business meaning.

Required:
```kotlin
val items: List<Item>
```

Forbidden:
```kotlin
val items: List<Item>?
```

## String Normalization Policy

For string-like inputs:
- apply `trim()`,
- detect blank values,
- map blank to `null` or explicit safe fallback based on business rule.

## Critical Field Policy

For critical fields:
- do not use `orEmpty()`,
- do not silently fallback,
- use fail-fast strategy (`error`, validation failure, explicit guard).

## Aggregation & Preservation Policy

When combining fields:
- preserve original source fields,
- add derived field(s),
- prefer dedicated nested aggregated model.

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

## Aggregation Class Rule

Create a separate class when:
- 2+ related fields exist,
- derived values exist,
- fields represent a logical group.

Typical groups:
- name,
- money,
- period,
- coordinates,
- phone,
- address.

## Root Model Structure Rule

- Keep root model readable.
- Avoid uncontrolled flattening.
- Group logically related data.

## Type Transformation Rule

Allowed when useful:
- `String -> enum`,
- `String -> date/time`,
- `String -> structured value object`,
- multi-field -> aggregated object.

Raw/source information must remain accessible.

## toData() Rules

`toData()` must:
- convert unsafe -> safe,
- validate input,
- normalize values,
- aggregate related fields,
- filter invalid nested entries,
- stay readable (not monolithic),
- delegate nested logic to private helpers.

## toRequest() Rules

`toRequest()`:
- generate only when needed,
- convert safe model -> request model deterministically,
- no hidden defaults,
- must be in a separate file from `toData()` root mapper.

## Ordering Rules (Inside Mapper File)

1. root mapper
2. private nested helpers
3. private normalization helpers
4. private aggregation helpers

---

## Section 1 — Rules Summary

- Architecture-neutral and deterministic.
- One root mapper per file (`internal`).
- Root names only: `toData()` / `toRequest()`.
- Private nested/utility helpers only.
- Full block bodies only, no expression bodies.
- Mandatory trailing commas in multiline structures.
- Safe model is business-oriented, not DTO mirror.
- Preserve original fields when aggregating.
- Validate critical fields explicitly.

## Section 2 — Model Template

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

## Section 3 — Mapper Template

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

## Section 4 — Aggregation Template

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

## Section 5 — toRequest Template

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

## Section 6 — Good Example

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

## Section 7 — Bad Example

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

## Section 8 — Validation Rules

Before returning output, verify:
- only one root mapper per file,
- root mapper name is only `toData()` or `toRequest()`,
- root mapper visibility is `internal`,
- nested helpers are `private`,
- all functions use block bodies with `{}`,
- no expression body `=`,
- trailing commas exist in every multiline structure,
- model names have no suffix/layer markers,
- safe model does not mirror transport nullability blindly,
- critical fields use explicit validation/fail-fast,
- original fields preserved when aggregation is present.

## Section 9 — Edge Case Handling

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
