# OpenCodeSkills

Коллекция специализированных скиллов для OpenCode.
Ниже — краткое описание каждого скилла и его ключевых правил.

## Skills

### `skill-creator`

**Назначение:** создание и рефакторинг скиллов OpenCode по строгому шаблону с проверяемыми правилами и обязательным покрытием примерами.

- Приводит `SKILL.md` к канонической 12-секционной структуре.
- Формализует trigger-контракт (should-trigger / should-not-trigger / edge cases).
- Требует тестируемые output-критерии и явные safety-границы.
- Обязывает минимальный набор примеров (happy path, bad/fixed, edge, boundary).
- Стандартизирует ресурсы `references/`, `assets/`, `scripts/` только при реальной необходимости.

### `skill-description-generator`

**Назначение:** генерация и обновление каталога скиллов в `README.md` из метаданных `SKILL.md` по единому шаблону.

- Использует metadata/frontmatter как основной источник данных (`name`, `description`, `short-description` и др.).
- Нормализует описания на русском с сохранением технических токенов.
- Обновляет только раздел `## Skills`, не затрагивая остальные части README.
- Применяет единый формат записи для всех скиллов.
- Поддерживает fallback-цепочку при неполных метаданных.

### `kotlin-response-request`

**Назначение:** генерация Kotlin `Request`/`Response` DTO на `kotlinx.serialization`.

- Все поля обязательны в формате nullable optional: `Type? = null`.
- Обязательные аннотации: `@Serializable` на каждом классе и `@SerialName` на каждом поле.
- Только `data class`, строгие суффиксы `Request`/`Response`.
- В одном `.kt` файле — только одна root-модель.
- Допустима только вложенность `root -> nested` (nested -> nested запрещена).

### `kotlin-safe-model-mapper`

**Назначение:** генерация безопасных Kotlin-моделей и детерминированных мапперов из transport-моделей.

- Строгий контракт root-мапперов: только `toData()` или `toRequest()`.
- Один root-маппер на файл; `internal` для root и `private` для helper-функций.
- Только block body (expression body запрещен), обязательные trailing comma.
- Нормализация строк, fail-fast для критичных полей, без «тихих» дефолтов.
- Поддержка агрегаций с сохранением исходных полей и архитектурная нейтральность.

### `kotlin-client-repository-generator`

**Назначение:** генерация repository-слоя для клиентских Kotlin-приложений (Android/KMP).

- Генерирует интерфейс и реализацию репозитория с compile-ready структурой.
- Подключает только реально используемые зависимости (`featureDao`, `httpClient`).
- Использует только существующие мапперы (`toData()`, `toRequest()`, `toEntity()`).
- Добавляет KDoc на русском (краткий в интерфейсе, расширенный в реализации).
- Поддерживает сценарии DAO-only, HTTP-only, DAO+HTTP, Flow и HTTP 204/no-body.
- Не добавляет DI/threading/error-wrapping без явного запроса.

### `kotlin-client-usecase-generator`

**Назначение:** генерация usecase-слоя для клиентских Kotlin-приложений (Android/KMP).

- Генерирует `interface` + `internal`-реализацию с единственным `suspend execute(...)`.
- Обязательно использует `DispatcherProvider` и `withContext` (`IO` по умолчанию, `Default` для CPU-bound).
- Сохраняет строгий API-контракт: без `operator invoke`, без `Flow`, без лишних публичных методов.
- Поддерживает orchestration-логику и private helper methods при необходимости.
- Добавляет KDoc на русском, использует named arguments и trailing comma.
