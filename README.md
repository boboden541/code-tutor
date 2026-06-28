<p align="center">
  <strong>code-tutor</strong><br>
  <em>Генератор обучающих книг по разработке: от пустой папки до прод-стенда — на любом стеке</em>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/pipeline-6%20шагов-4a9eff" alt="6 шагов">
  <img src="https://img.shields.io/badge/навыки-5-green" alt="5 навыков">
  <img src="https://img.shields.io/badge/стек-backend%20%C2%B7%20frontend%20%C2%B7%20fullstack-purple" alt="backend/frontend/fullstack">
  <img src="https://img.shields.io/badge/MCP-Context7-orange?logo=anthropic&logoColor=white" alt="Context7 MCP">
</p>

---

> **code-tutor** превращает идею проекта в **пошаговую книгу-туториал**, по которой новичок своими руками строит приложение — от установки окружения до деплоя на прод.
>
> Стек **не зашит**: backend, frontend или fullstack на любом языке/фреймворке (Python/Go/Java/Node + React/Vue/…). ИИ либо берёт стек из вашего описания, либо предлагает обоснованный сам.
>
> Ключевой принцип — **код без объяснения запрещён**: после каждого блока кода идёт «Разбор кода» и «Под капотом». Глубина и отсутствие галлюцинаций держатся на **замороженной базе знаний под выбранный стек** (Stack Knowledge Pack), собираемой из актуальных доков через **Context7**.

---

## Содержание

- [Быстрый старт](#-быстрый-старт)
- [Процесс (пайплайн)](#-процесс-пайплайн)
- [Доступные команды](#-доступные-команды)
- [Артефакты пайплайна](#-артефакты-пайплайна)
- [Ключевые абстракции](#-ключевые-абстракции)
- [Навыки](#-навыки)
- [Структура репозитория](#-структура-репозитория)
- [Применение на практике](#применение-на-практике)
- [Context7 MCP](#-context7-mcp)

---

## ⚡ Быстрый старт

code-tutor — это набор **команд** (`/tutor-*`) и **навыков** для Claude Code, лежащих в `.claude/`. Два способа использовать:

**A. Работать прямо в этом репозитории** (книга генерируется в `book/`):

```bash
git clone https://github.com/boboden541/code-tutor.git
cd code-tutor
```

**B. Подключить инструментарий к своему проекту** — скопировать управляемые папки и MCP-конфиг:

```bash
git clone --depth 1 https://github.com/boboden541/code-tutor.git /tmp/code-tutor
cp -R /tmp/code-tutor/.claude/commands ./.claude/commands
cp -R /tmp/code-tutor/.claude/skills   ./.claude/skills
cp    /tmp/code-tutor/.mcp.json         ./.mcp.json
cp    /tmp/code-tutor/.claude/settings.json ./.claude/settings.json   # пред-одобрение Context7
rm -rf /tmp/code-tutor
```

> **После установки перезагрузите IDE** (Reload Window), чтобы команды `/tutor-*` появились в чате, а MCP-сервер `context7` подключился.

Затем запустите цепочку (по одному шагу):

```text
/tutor-brief Хочу fullstack «список задач»: пользователи, проекты, задачи. Бэкенд + веб-интерфейс.
/tutor-requirements
/tutor-stack
/tutor-plan
/tutor-write
/tutor-validate
```

---

## 🔄 Процесс (пайплайн)

```mermaid
flowchart TD
    A["/tutor-brief"] -->|"project_brief.md (Файл №1)"| B["/tutor-requirements"]
    B -->|"requirements.md (Файл №2)"| C["/tutor-stack"]
    C -->|"stack_decision.md (Файл №3)<br>+ _knowledge/ (заморожен)<br>+ contracts/*.yaml"| D["/tutor-plan"]
    D -->|"book/outline.md"| E["/tutor-write"]
    E -->|"book/**/*.md (шаги)"| F["/tutor-validate"]
    F -->|"PASS"| G(("Готово"))
    F -->|"FAIL"| E

    C -.->|"источник доков"| X[["Context7 MCP"]]

    style A fill:#4a9eff,color:#fff
    style C fill:#ffd43b,color:#000
    style E fill:#ffd43b,color:#000
    style F fill:#ff6b6b,color:#fff
    style G fill:#51cf66,color:#fff
    style X fill:#9b59b6,color:#fff
```

**Идея:** требования (что строим) отделены от стека (на чём строим). Шаг `/tutor-stack` выбирает стек, резолвит профиль архитектуры и **замораживает** базу знаний — после этого книга пишется строго против неё, поэтому она воспроизводима и без галлюцинаций.

---

## 🛠 Доступные команды

| Команда | Навык | Описание | Результат |
|:--------|:------|:---------|:----------|
| `/tutor-brief` | project-brief | Идея → бриф (тип проекта, домен, сценарии; стек опционален) | `project_brief.md` |
| `/tutor-requirements` | requirements-interview | Опросник FR/UR/IR/NFR (стек-агностично) | `requirements.md` |
| `/tutor-stack` | stack-advisor | Выбор/предложение стека, профиль, **сборка+заморозка пака**, контракт | `stack_decision.md`, `book/_knowledge/`, `book/contracts/` |
| `/tutor-plan` | book-architect | План книги по профилю (для fullstack — 2 трека) | `book/outline.md` |
| `/tutor-write` | book-author | Запись шагов против пака (идемпотентно, возобновляемо) | `book/**/*.md` |
| `/tutor-validate` | book-architect | Проверка целостности (стек-aware, только чтение) | отчёт PASS/FAIL |

---

## 📦 Артефакты пайплайна

| Артефакт | Создаёт | Что внутри |
|:---------|:--------|:-----------|
| **Файл №1** `project_brief.md` | `/tutor-brief` | Цель, **тип проекта** (backend/frontend/fullstack), домен, сценарии, профиль читателя; стек — опционально |
| **Файл №2** `requirements.md` | `/tutor-requirements` | Требования по 4 категориям с дефолтами ИИ |
| **Файл №3** `stack_decision.md` | `/tutor-stack` | Утверждённый стек + версии + обоснования, резолв профиля(ей), индекс пака |
| **Knowledge Pack** `book/_knowledge/<tech>/` | `/tutor-stack` | Замороженная база знаний под стек (источник истины для записи) |
| **Контракт** `book/contracts/<domain>.yaml` | `/tutor-stack` | OpenAPI 3.0.3 — шов фронт↔бэк |
| **План** `book/outline.md` | `/tutor-plan` | Машиночитаемый план (части → главы → секции → шаги) |
| **Книга** `book/**` | `/tutor-write` | Сами шаги-уроки с полным кодом и разборами |

---

## 🧠 Ключевые абстракции

| Абстракция | Зачем | Где |
|:-----------|:------|:----|
| **Architecture Profile** | Декларативный профиль на технологию: порядок слоёв фичи, скелет глав, файл-снимок состояния, тест-раннер, деплой. Делает план стек-независимым | `.claude/skills/book-architect/resources/profiles/` |
| **Stack Knowledge Pack** | Замораживаемая база знаний под выбранные библиотеки. Переносит «единый источник истины, без галлюцинаций» на любой стек | `book/_knowledge/<tech>/` |
| **API Contract (OpenAPI)** | Генерится *вперёд* из требований. Бэк реализует ровно его, фронт строит типы ровно под него | `book/contracts/` |

**Профили из коробки:** `backend_layered` (FastAPI/Spring/Nest/Go…), `frontend_spa` (React/Vue/Svelte…), `_generic_template` (заполняется для экзотики).

**Стандарт подачи (убывающая детализация):** 1-я фича — один слой = один шаг с полным кодом и разбором; 2-я — слои сгруппированы; 3-я+ — «по образцу со ссылкой». Каждая фича = ветка `feat/*` = Merge Request.

---

## 📂 Навыки

Каждый навык содержит **SKILL.md** (ролевая модель + принципы), **resources/** (стандарты, профили, чек-листы), **examples/** (шаблоны-эталоны).

| Навык | Назначение |
|:------|:-----------|
| `project-brief/` | Идея → структурированный бриф (Файл №1) |
| `requirements-interview/` | Бриф → опросник требований (Файл №2) |
| `stack-advisor/` | Выбор стека, резолв профиля, сборка Knowledge Pack, генерация контракта (Файл №3) |
| `book-architect/` | Проектирование плана книги + чек-лист валидации + профили архитектуры |
| `book-author/` | Написание шагов книги против пака (полный код + «Разбор кода» + «Под капотом») |

---

## 🗂 Структура репозитория

```
code-tutor/
├── .claude/
│   ├── commands/                 ← 6 команд /tutor-*
│   ├── skills/                   ← 5 навыков (роли + ресурсы + эталоны)
│   │   ├── project-brief/
│   │   ├── requirements-interview/
│   │   ├── stack-advisor/
│   │   ├── book-architect/       ← resources/profiles/ — профили архитектуры
│   │   └── book-author/
│   └── settings.json             ← пред-одобрение MCP context7
├── .mcp.json                     ← MCP-сервер context7 (актуальные доки библиотек)
├── project_brief.md              ← Файл №1 (пример: Personal Finance Assistant)
├── requirements.md               ← Файл №2
└── book/                         ← сгенерированная книга
    ├── outline.md                ← план (контракт для /tutor-write)
    ├── _knowledge/               ← Stack Knowledge Pack (заморожен)
    ├── contracts/                ← API-контракты OpenAPI
    └── chapter_*/section_*/NN_*.md
```

---

# Применение на практике

### Backend на Python (стек задан)

```text
/tutor-brief Сервис заметок с тегами и пользователями. Стек: FastAPI, PostgreSQL, SQLAlchemy, JWT, Docker, GitLab CI.
/tutor-requirements          # принять дефолты ИИ или дозаполнить
/tutor-stack                 # подтвердит стек, соберёт пак из доков (Context7), сгенерит контракт
/tutor-plan                  # план книги по профилю backend_layered
/tutor-write                 # пишет шаги; запускайте повторно — допишет оставшиеся
/tutor-validate              # PASS/FAIL
```

### Fullstack (стек предлагает ИИ)

```text
/tutor-brief Fullstack «трекер привычек»: пользователи, привычки, отметки выполнения, статистика. Бэкенд + веб-интерфейс. Стек на ваше усмотрение.
/tutor-requirements
/tutor-stack                 # предложит, напр., FastAPI + React; профили backend_layered + frontend_spa; контракт OpenAPI
/tutor-plan                  # две части: part_1_backend / part_2_frontend, контракт-first срезы по фичам
/tutor-write 5               # написать 5 ближайших шагов за вызов
/tutor-validate
```

> **Возобновляемость и идемпотентность:** `/tutor-write` пишет только незавершённые шаги (`[ ]`) и никогда не перезаписывает готовые (`[x]`). Запускайте сколько угодно раз.

---

## 🔌 Context7 MCP

`/tutor-stack` собирает Knowledge Pack из **актуальной документации библиотек** через [Context7](https://github.com/upstash/context7). Сервер уже сконфигурирован в репозитории:

- `.mcp.json` — сервер `context7` (`npx -y @upstash/context7-mcp`)
- `.claude/settings.json` — `enabledMcpjsonServers: ["context7"]` (подключается без ручного подтверждения)

**Как используется:** для каждой ключевой библиотеки стека — `resolve-library-id` → `get-library-docs` (под нужную версию), происхождение фактов помечается в паке (`source: context7:<id>`).

> **Fallback:** если Context7 недоступен (headless/cron/сервер не поднялся) — `/tutor-stack` берёт факты из WebSearch/WebFetch и помечает это в индексе пака. Книга соберётся в любом окружении.

Проверить подключение: команда `/mcp` (должен быть `context7` → connected). API-ключ не обязателен (работает с базовыми лимитами); для больших лимитов можно добавить `--api-key` в `.mcp.json`.

---
