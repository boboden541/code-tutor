---
description: Шаг #4 code-tutor — проектирует UI: экраны + дизайн-система (design_spec.md, Файл №4) и утверждаемый прототип (только при наличии frontend-трека)
---

## Использование

```
/tutor-design [project_brief.md] [requirements.md] [stack_decision.md]
```

**Параметры:** пути к Файлу №1, №2 и №3 (по умолчанию `project_brief.md`, `requirements.md`, `stack_decision.md`).

## Пример

```
/tutor-design
/tutor-design project_brief.md requirements.md stack_decision.md
```

**Ожидаемый результат:**
- `design_spec.md` (Файл №4) — суть+эстетика, дизайн-система (токены/компоненты/тон), матрица «экран×эндпоинт», спецификация каждого экрана, способ стилизации, ссылка на утверждённый прототип;
- визуальный прототип в Claude Design (DesignSync) или Artifact (fallback).

> Шаг выполняется **только при наличии frontend-трека** (frontend/fullstack из Файла №3). Для backend-only — пропусти и сразу `/tutor-plan`.

---

## Инструкция для LLM

### Этап 1: Загрузка роли
1. Прочитай `.claude/skills/ui-designer/SKILL.md` — персона и принципы.
2. Загрузи `.claude/skills/ui-designer/resources/screen_derivation.md` — как выводить экраны и матрицу покрытия.
3. Загрузи `.claude/skills/ui-designer/resources/screen_spec_schema.md` — поля каждого экрана.
4. Загрузи `.claude/skills/ui-designer/resources/design_system_spec.md` — что фиксировать в дизайн-системе.
5. Загрузи `.claude/skills/ui-designer/resources/prototype_guide.md` — как собрать и дать утвердить прототип.
6. Подгрузи глобальный навык `frontend-design` для дисциплины эстетики (палитра/типографика/фирменный элемент).
7. Сверь термины с `.claude/skills/book-architect/resources/naming_conventions.md`.

### Этап 2: Чтение входов
Прочитай Файлы №1, №2, №3, API-контракт(ы) `book/contracts/*.yaml` и индекс Knowledge Pack фронт-трека (read-only). Если тип проекта backend-only — сообщи, что дизайн-шаг не нужен, и предложи `/tutor-plan`. Если нет Файла №3 — предложи `/tutor-stack`.

### Этап 3: Вывод экранов (матрица покрытия)
1. Назови суть продукта (предмет/аудитория/задача) — из неё выводится эстетика.
2. По `screen_derivation.md`: выпиши все операции контракта и ролевые флоу; разложи операции по экранам; построй матрицу «экран×эндпоинт».
3. Проверь: нет непотреблённых эндпоинтов, нет разрывов в ролевых флоу.

### Этап 4: Спецификация экранов
Для каждого экрана заполни поля `screen_spec_schema.md`: маршрут, role-gating, api_calls, формы+валидация (зеркало контракта), парсинг данных, состояния loading/empty/error и pending/success/failure, навигация, компоненты.

### Этап 5: Дизайн-система
По `design_system_spec.md`: выведи токены (палитра/типографика/spacing/радиусы/тени) из сути; перечисли компоненты и тон; зафиксируй способ стилизации (дефолт — токены + CSS Modules, без новых зависимостей).

### Этап 6: Прототип и утверждение
По `prototype_guide.md` собери прототип и дай утвердить:
- основная поверхность — **Claude Design (DesignSync)**: `list_projects` → (`create_project`) → `finalize_plan` → `write_files`; дай ссылку, попроси утвердить/дополнить;
- если DesignSync/ design-login недоступны — **fallback Artifact** (self-contained HTML); если и он недоступен — spec-only, честно отметь.
> Инструмент `DesignSync` может быть отложенным — найди его схему через ToolSearch (`DesignSync` / `design`) перед первым вызовом.
Дружелюбный чекпоинт: дефолт «утвердить и продолжить»; внеси правки пользователя.

### Этап 7: Файл №4 и чекпоинт
1. Сохрани `design_spec.md` по шаблону `.claude/skills/ui-designer/examples/ideal_design_spec.md` (суть+эстетика, дизайн-система, матрица, экраны, способ стилизации, ссылка на прототип/пометка fallback).
2. Выведи: «Файл №4 создан, прототип утверждён. Следующий шаг: `/tutor-plan`».

> **Важно:** план фронт-книги нельзя строить без утверждённого `design_spec.md` — иначе книга даст белую страницу. Этот шаг — обязательный фундамент интерфейса между стеком и планом (для frontend/fullstack).
