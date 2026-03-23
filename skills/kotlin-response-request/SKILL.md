---
name: kotlin-response-request
description: Generate Kotlin Request/Response DTO classes using kotlinx.serialization with strict constraints: all fields nullable and optional, explicit class and field serialization annotations, data classes only, Request/Response suffix naming, single root model per file, and one-level nesting only (root -> nested).
metadata:
  short-description: Kotlin Request/Response DTO generation
---

# Kotlin Response/Request DTO Skill

Generate Kotlin DTO models for API payloads with strict structure and naming rules.

## Rules You Must Enforce

1. Use `kotlinx.serialization` for all generated models.
2. Every class MUST be a `data class`.
3. Every class MUST have `@Serializable`.
4. Every field MUST have explicit `@SerialName("...")`.
5. Every field MUST be nullable and optional: `Type? = null`.
6. Every class name MUST end with `Request` or `Response`.
7. One Kotlin file can contain only one root model.
8. Nesting depth limit is exactly one level: only `root -> nested`.
9. `nested -> nested` is forbidden.
10. Always use trailing commas in class parameter lists, including the last parameter.

## File Layout Policy

- Exactly one root model (`*Request` or `*Response`) per `.kt` file.
- If user input contains multiple root entities, split them into separate files.
- Nested classes in the same file are allowed only as direct children of that single root.

## Output Format

Always return complete Kotlin code with:

- imports for `kotlinx.serialization`
- root class and permitted direct nested classes
- required suffixes in class names
- `@Serializable` on every class
- `@SerialName` on every field
- nullable optional properties with `= null`
- trailing comma after the last class parameter

## Workflow

1. Parse user input (JSON/schema/text) and identify root entities.
2. Classify each root as request or response.
3. If multiple roots exist, generate one file per root.
4. Build the root class first.
5. For object fields directly inside root, generate direct nested classes only.
6. Add annotations:
   - class level: `@Serializable`
   - field level: `@SerialName("json_key")`
7. Make every property nullable and optional: `val x: T? = null`.
8. Add trailing commas in all class parameter lists.
9. Validate naming suffixes and nesting depth before returning code.

## Type Mapping Defaults

- `string` -> `String? = null`
- `integer` -> `Int? = null`
- `long` -> `Long? = null`
- `number` -> `Double? = null`
- `boolean` -> `Boolean? = null`
- array of primitives -> `List<T>? = null`
- array of objects (root child only) -> `List<NestedXRequest/Response>? = null`
- object (root child only) -> `NestedXRequest/Response? = null`

## Validation Checklist

- Every class is `data class`
- Every class has `@Serializable`
- Every field has `@SerialName`
- Every field is nullable and defaults to `null`
- Every class name ends with `Request` or `Response`
- File has exactly one root model
- No `nested -> nested` structure
- Every class parameter list uses trailing commas

## Error Handling

If input violates constraints:

1. Explain which rule is violated.
2. Propose a flattened or split-by-file alternative.
3. Do not output structures that break the nesting or root-per-file rules.

## Minimal Template

```kotlin
import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class ExampleResponse(
    @SerialName("id")
    val id: String? = null,
    @SerialName("meta")
    val meta: MetaResponse? = null,
    @SerialName("details")
    val details: DetailsResponse? = null,
) {
    @Serializable
    data class MetaResponse(
        @SerialName("source")
        val source: String? = null,
    )

    @Serializable
    data class DetailsResponse(
        @SerialName("version")
        val version: String? = null,
    )
}
```
