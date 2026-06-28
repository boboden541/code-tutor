---
description: Шаг #6 code-tutor — проверяет целостность книги, стек-aware (только чтение, идемпотентно)
---

## Использование

```
/tutor-validate [book/outline.md]
```

**Параметр:** путь к плану (по умолчанию `book/outline.md`).

## Пример

```
/tutor-validate book/outline.md
```

**Ожидаемый результат:** отчёт о расхождениях в чат + вердикт PASS / FAIL. Файлы книги НЕ изменяются.

---

## Инструкция для LLM

### Этап 1: Загрузка чек-листа
Прочитай `.claude/skills/book-architect/resources/book_validation.md` — критерии проверки (разделы 1–10).
Сверь термины/формат с `.claude/skills/book-architect/resources/naming_conventions.md`.

### Этап 2: Сбор данных (только чтение)
1. Прочитай `book/outline.md`, распарси шаги регэкспом `^- \[( |x)\] (\S+\.md) — (.+)$`.
2. Получи фактический список файлов в `book/**`.
3. Прочитай `requirements.md` (Файл №2) — для покрытия требований.
4. Прочитай `stack_decision.md` (Файл №3) — тип проекта, стек(и), профиль(и), путь к паку и контракту.
5. Прочитай индекс(ы) Knowledge Pack (`book/_knowledge/<tech>/00_index.md`) и контракт(ы) (`book/contracts/*.yaml`), если есть.

### Этап 3: Проверки
Для каждого пункта чек-листа `book_validation.md` (разделы 1–10) собери расхождения. Типы — из таблицы чек-листа, включая стек-aware:
- `missing` / `orphan` / `broken-link` / `numbering` / `uncovered-requirement`;
- `missing-branch-step` / `missing-mr-step` / `placeholder` / `unexplained-code` / `orphan-symbol`;
- `missing-recap` / `missing-overview` / `deps-drift` / `step-order` / `step-count-mismatch`;
- `pack-not-frozen` / `profile-incomplete` / `profile-layer-mismatch` / `part-structure` / `contract-drift` / `pack-drift`.

> Если есть контракт — можно прогнать линтер OpenAPI через Bash (redocly → openapi_spec_validator → yaml) для объективной проверки его валидности; расхождения «код шага vs контракт» проверяются чтением.

### Этап 4: Отчёт
1. Выведи таблицу расхождений: `Тип | Путь | Детали`.
2. Вердикт: **PASS** (расхождений нет) или **FAIL** (есть).
3. Не модифицируй файлы книги (валидатор только читает).
4. При FAIL — подскажи: исправить стек/пак `/tutor-stack`, план `/tutor-plan` или дописать `/tutor-write`.
