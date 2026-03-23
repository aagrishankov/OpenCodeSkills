# OpenCodeSkills

## Skills

- `skill-creator` - создание и улучшение скиллов для OpenCode: помогает собрать требования, написать `SKILL.md`, настроить структуру ресурсов (`scripts/`, `references/`, `assets/`), улучшить описание для корректного триггера и итеративно доработать скилл по обратной связи.
- `kotlin-response-request` - генерация Kotlin `Request`/`Response` DTO на `kotlinx.serialization`: все поля всегда nullable (опциональные) и со значением по умолчанию `null`, обязательны аннотации `@Serializable` для классов и `@SerialName` для полей, используются только `data class`, допускается только вложенность `root -> nested`, вложенность `nested -> nested` запрещена, и в одном `.kt` файле допускается только одна root-модель.
