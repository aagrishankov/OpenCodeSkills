# OpenCodeSkills

Коллекция специализированных скиллов для OpenCode.
Ниже — краткое описание каждого скилла и его ключевых правил.

## Skills

### `skill-creator`

**Назначение:** создание и улучшение скиллов для OpenCode.

- Помогает собрать требования.
- Формирует `SKILL.md`.
- Настраивает структуру ресурсов: `scripts/`, `references/`, `assets/`.
- Улучшает описание для корректного триггера.
- Поддерживает итеративную доработку по обратной связи.

### `kotlin-response-request`

**Назначение:** генерация Kotlin `Request`/`Response` DTO на `kotlinx.serialization`.

- Все поля — nullable (опциональные) со значением по умолчанию `null`.
- Обязательные аннотации: `@Serializable` для классов и `@SerialName` для полей.
- Используются только `data class`.
- Разрешена только вложенность `root -> nested`.
- Вложенность `nested -> nested` запрещена.
- В одном `.kt` файле допускается только одна root-модель.

### `kotlin-safe-model-mapper`

**Назначение:** генерация безопасных Kotlin-моделей и детерминированных мапперов из transport-моделей.

- Строгое именование: `toData()`/`toRequest()`.
- Один root-маппер на файл.
- Видимость: `internal` для root, `private` для helper-функций.
- Только block body.
- Обязательные trailing comma.
- Нормализация строк и фильтрация невалидных nested-элементов.
- Агрегация с сохранением исходных полей.
- Архитектурная нейтральность + встроенные шаблоны, примеры и чеклисты.

### `kotlin-client-repository-generator`

**Назначение:** генерация repository-слоя для клиентских Kotlin-приложений (Android/KMP).

- Генерирует интерфейс и реализацию репозитория.
- Подключает зависимости (`featureDao`, `httpClient`) только при использовании.
- Вызывает существующие мапперы: `toData()`/`toRequest()`/`toEntity()`.
- Добавляет KDoc на русском.
- Учитывает импорты.
- Поддерживает шаблоны: DAO-only, HTTP-only, DAO+HTTP, Flow.
- Соблюдает ограничения: без DI, threading, error-wrapping и paging.

### `kotlin-client-usecase-generator`

**Назначение:** генерация usecase-слоя для клиентских Kotlin-приложений (Android/KMP).

- Генерирует интерфейс и реализацию юзкейса.
- Соблюдает строгий контракт: `suspend execute(...)`.
- Обязательно использует `DispatcherProvider` + `withContext` (`IO` по умолчанию).
- Поддерживает orchestration-логику с optional private helper methods.
- Добавляет KDoc на русском.
- Использует named arguments и trailing comma.
- Включает шаблоны и чеклист.
