# Architecture Profile — схема (контракт абстракции)

> **Зачем.** Раньше `code-tutor` был зашит под FastAPI: порядок слоёв фичи (`schema→domain→repo→usecase→router→test`), скелет глав и вся база знаний жили в хардкоде. Чтобы поддержать **любой стек** (Python/Go/Java + React/Vue/…) и **fullstack**, эта конкретика вынесена в *профиль архитектуры* — декларативный документ на семейство технологий.
>
> `/tutor-stack` **резолвит** один профиль на каждый трек проекта (backend → один профиль, frontend → другой, fullstack → оба). `book-architect` строит outline по выбранным профилям; `book-author` пишет шаги, опираясь на профиль + Stack Knowledge Pack.

## Что обязан декларировать профиль

Каждый профиль (`profiles/<name>.md`) описывает следующие поля. Если для конкретного стека поле отличается — оно уточняется в Stack Knowledge Pack (`book/_knowledge/<tech>/`), но **структура** берётся из профиля.

| Поле профиля | Что задаёт | Пример (backend_layered) | Пример (frontend_spa) |
|--------------|-----------|--------------------------|------------------------|
| `track` | к какому треку относится | `backend` | `frontend` |
| `layers` | **порядок слоёв одной фичи** (= порядок шагов «один слой = один шаг» для первой фичи) | `schema → domain → repository_interface → repository_impl → (migration) → usecase → router → test` | `api_client → store/state → hooks → components → test` |
| `chapter_skeleton` | рекомендованный порядок глав «от пустой папки до результата» | setup tooling → first runnable → БД/миграции → домен+фичи → тесты → CI → деплой | setup tooling → first screen → routing → state → фичи → тесты → build → deploy |
| `state_snapshot_file` | файл-манифест зависимостей для `recap_and_state` | `requirements.txt` / `pyproject.toml` | `package.json` |
| `tooling` | пакетный менеджер, запуск, dev-сервер | `pip`/`venv`, `uvicorn` | `npm`/`pnpm`, `vite` |
| `test_runner` | как пишутся и запускаются тесты | `pytest` | `vitest` + Testing Library |
| `build` | как собирается артефакт | Docker-образ | `vite build` → статика |
| `deploy_model` | как доставляется на прод | контейнер за reverse-proxy | статика на CDN/nginx, либо тот же контейнер |
| `first_feature_rule` | как раскладывается ПЕРВАЯ фича | один слой = один шаг (полный код + разбор) | один слой = один шаг |
| `mental_model_ref` | какой обзор открывает доменную главу | поток запроса `request_flow_overview` | поток данных/рендера `data_flow_overview` |
| `glossary_seed` | базовые термины для разбора | DI, декоратор, ORM, миграция | компонент, состояние, хук, реактивность |

## Правила применения профиля

1. **Профиль задаёт структуру, пак задаёт конкретику.** Профиль говорит «у фичи есть слой api_client»; *как именно* он пишется на React + axios/fetch — берётся из `book/_knowledge/<frontend_tech>/`.
2. **Один слой = один шаг — только для первой фичи трека** (см. `delivery_pipeline.md`, «убывающая детализация»). Вторая фича — слои сгруппированы; третья+ — «по образцу со ссылкой».
3. **Доменная глава открывается обзором** (`mental_model_ref`): для бэка — как запрос проходит слои; для фронта — как данные текут от API-клиента к UI и как идёт рендер.
4. **Каждая глава завершается** `recap_and_state` с полным `state_snapshot_file` и деревом проекта.
5. **Каждая фича-секция обрамляется** `start_feature_branch` / `commit_push_open_mr` (git-flow един для всех профилей, см. `book-author/.../git_workflow.md`).
6. **Если курируемого профиля под стек нет** — `/tutor-stack` заполняет `_generic_template.md` на основе Stack Knowledge Pack (исследование Context7/веба) и фиксирует его как профиль проекта.

## Резолв профиля по стеку (для `/tutor-stack`)

| Стек / роль | Профиль |
|-------------|---------|
| Серверный фреймворк со слоистой архитектурой (FastAPI, Django REST, Spring, Express/Nest, Go-chi/Gin, Laravel…) | `backend_layered` |
| Клиентское SPA (React, Vue, Svelte, Angular, SolidJS…) | `frontend_spa` |
| Иное / экзотика / гибрид (SSR-фреймворк типа Next/Nuxt, мобильное, CLI…) | `_generic_template` (заполнить из пака) |

> Профиль — **не** копипаст кода. Это карта «какие слои, в каком порядке, чем проверяется». Реальный код всегда берётся/синтезируется по Stack Knowledge Pack под конкретные библиотеки версии проекта.
