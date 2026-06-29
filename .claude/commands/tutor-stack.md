---
description: Шаг #3 code-tutor — выбирает/предлагает стек, резолвит профили архитектуры, собирает и замораживает Knowledge Pack, генерит API-контракт (stack_decision.md)
---

## Использование

```
/tutor-stack [project_brief.md] [requirements.md]
```

**Параметры:** пути к Файлу №1 и Файлу №2 (по умолчанию `project_brief.md` и `requirements.md`).

## Пример

```
/tutor-stack
/tutor-stack project_brief.md requirements.md
```

**Ожидаемый результат:**
- `stack_decision.md` (Файл №3) — утверждённый стек + версии + обоснования, резолв Architecture Profile(ей), индекс Knowledge Pack;
- `book/_knowledge/<tech>/*.md` — собранный и **замороженный** Stack Knowledge Pack (на каждый трек);
- `book/contracts/<domain>.yaml` — API-контракт OpenAPI (для fullstack / backend с API).

---

## Инструкция для LLM

### Этап 1: Загрузка роли
1. Прочитай `.claude/skills/stack-advisor/SKILL.md` — твоя персона и принципы.
2. Загрузи `.claude/skills/stack-advisor/resources/knowledge_pack_spec.md` — что и как собирать в пак.
3. Загрузи `.claude/skills/stack-advisor/resources/stack_selection_heuristics.md` — как выбирать/валидировать стек.
4. Загрузи `.claude/skills/stack-advisor/resources/contract_first.md` — генерация контракта (если есть API).
5. Загрузи `.claude/skills/stack-advisor/resources/build_smoke_gate.md` — проверка пака сборкой до заморозки.
6. Загрузи `.claude/skills/book-architect/resources/profiles/_profile_schema.md` — как резолвить профиль.
7. Сверь термины с `.claude/skills/book-architect/resources/naming_conventions.md`.

### Этап 2: Чтение входов
Прочитай Файл №1 и Файл №2. Если какого-то нет — сообщи и предложи предыдущий шаг (`/tutor-brief` или `/tutor-requirements`). Выпиши: **тип проекта** (backend/frontend/fullstack), заданный стек (если есть), сущности, эндпоинты, ключевые NFR.

### Этап 3: Решение по стеку (на каждый трек)
1. Если стек задан в Файле №1 — **прими и валидируй** (предупреди о рисках, не подменяй).
2. Если не задан — **предложи** с обоснованием (эвристики + дефолты).
3. Зафиксируй **версии** ключевых библиотек: Context7 MCP (если подключён) → иначе WebSearch/WebFetch.

### Этап 4: Резолв профиля(ей)
1. По таблице резолва в `_profile_schema.md` выбери профиль на каждый трек (`backend_layered` / `frontend_spa` / `_generic_template`).
2. Если `_generic_template` — **заполни** его из исследования и впиши в Файл №3.

### Этап 5: Сборка Knowledge Pack
1. По `knowledge_pack_spec.md` собери `book/_knowledge/<tech>/*.md` на каждый стек (бэк/фронт — отдельные подпапки).
2. **Источник фактов №1 — Context7 MCP** (сконфигурирован для проекта в `.mcp.json`, пред-одобрен в `.claude/settings.json`). Для каждой ключевой библиотеки стека:
   - `resolve-library-id` (имя библиотеки → Context7-ID);
   - `get-library-docs` по этому ID (при необходимости с `topic`: `routing`, `migrations`, `testing`, …) — актуальные доки под версию;
   - в файле пака помечай `> source: context7:<resolved-id>`.
   Если инструменты Context7 в сессии недоступны (headless/cron/сервер не поднялся) — **fallback** на WebSearch/WebFetch, помечай `> source: web:<url>`.
3. Фиксируй версии; непроверенное → `[NEEDS_VERIFICATION]`. Покрой обязательные темы пака под трек.
4. Запиши `00_index.md` с датой, версиями, источниками (что из Context7, что из веба). **`STATUS` пока НЕ `FROZEN`** — заморозка только после Этапа 5.5.

> Инструменты Context7 (`resolve-library-id`, `get-library-docs`) могут быть отложенными — найди их схемы через ToolSearch (запрос `context7` или `resolve-library-id`) перед первым вызовом.

### Этап 5.5: Build-smoke gate (обязателен до заморозки)
По `build_smoke_gate.md` собери минимальный скаффолд по паку в скрэтч-каталоге (вне `book/`) на каждый трек и **реально запусти**:
1. Backend-срез: `project_setup` + манифест с пинами → install; `/health`; при БД — модель+async-сессия+миграция (ловит `D2` greenlet); при auth — хэш+токен (ловит `D1` argon2≠bcrypt).
2. Frontend-срез: scaffold → install → **сверка фактических версий с `00_index.md`** (ловит `D5`); `npm run build` (не голый `npx tsc` — `D6`); `npm test`.
3. **Fullstack — docker-smoke «один up — три слоя»:** `docker compose up -d --build` → проверить API `/health`, nginx отдаёт SPA, сквозняк `web→api→БД`; затем `docker compose down`. `docker compose up` обязан поднимать всё **без ручных команд** (миграции — в entrypoint, единый compose со всеми сервисами) — иначе `D7`.
4. При падении — чини **пак** (файлы/версии в `_knowledge/`), не скаффолд; пересобирай. Предел — 5 итераций. Таблица «симптом → правка пака» — в `build_smoke_gate.md`.
5. Зелёный gate → запиши в `00_index.md` строку `BUILD_SMOKE: PASS (<дата>, <что проверено>)` и **только теперь** `STATUS: FROZEN`. После заморозки пак — read-only для `/tutor-write`.
6. Если Docker/сеть недоступны (headless/cron) — выполни доступную часть, отметь `BUILD_SMOKE: PARTIAL — <что пропущено>`; не маскируй пропуск под успех.

### Этап 6: API-контракт (если fullstack или backend с внешним API)
1. По `contract_first.md` сгенерируй `book/contracts/<domain>.yaml` (OpenAPI 3.0.3) **вперёд** из Файла №2 (сущности → схемы, эндпоинты → операции, краевые случаи → коды).
2. Прогони валидатор через Bash (redocly → openapi_spec_validator → yaml), цикл самокоррекции, предел 5 итераций, добейся 0 ошибок.
3. Неоднозначности помечай `[NEEDS_DECISION]`.

### Этап 7: Файл №3 и чекпоинт
1. Сохрани `stack_decision.md`: стек+версии+обоснования (таблица на трек), резолв профиля(ей), индекс Knowledge Pack (что собрано/источники/FROZEN), ссылка на контракт.
2. Дружелюбный чекпоинт: покажи предложенный стек и профиль(и), дай изменить; дефолт — «принять и продолжить».
3. Выведи следующий шаг по типу проекта:
   - **frontend / fullstack:** «Файл №3 создан, Knowledge Pack заморожен, контракт сгенерирован. Следующий шаг: `/tutor-design`» (проектирование UI до плана — иначе фронт будет белой страницей);
   - **backend-only:** «… Следующий шаг: `/tutor-plan`» (дизайн-шаг не нужен).

> **Важно:** книгу нельзя планировать/писать против незамороженного пака. Этот шаг — обязательный фундамент между требованиями и планом. Стек пользователя уважается; предложенное помечается как предложение ИИ.
