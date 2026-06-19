# База знаний: итеративная разработка по фичам (Git-flow)

> Источник истины для итеративной работы. Дата актуальности: 2026-06.
> Главный принцип книги: **одна бизнес-фича = одна ветка = один Merge Request**. Проект собирается не «одним большим коммитом», а серией маленьких проверяемых итераций.

## 1. Цикл одной фичи (feature loop)

```
получил фичу на вход
  └─ создать ветку от main         git checkout main && git pull && git checkout -b feat/<name>
       └─ реализовать по слоям      schema → domain → repository → usecase → router
            └─ покрыть тестами       pytest -q (зелёный)
                 └─ коммит + пуш     git add -A && git commit && git push -u origin feat/<name>
                      └─ открыть MR   GitLab → Merge Request → дождаться зелёного CI → merge
                           └─ вернуться на main и взять следующую фичу
```

Каждая фича проходит цикл целиком и **изолирована** в своей ветке: `main` всегда остаётся рабочим и зелёным.

## 2. Именование веток

| Тип работы | Префикс | Пример |
|------------|---------|--------|
| Новая бизнес-фича | `feat/` | `feat/incomes`, `feat/expenses`, `feat/goals-summary` |
| Инфраструктура/настройка | `chore/` | `chore/docker`, `chore/gitlab-ci` |
| Исправление | `fix/` | `fix/login-401` |

Имя — `kebab-case`, латиница, отражает фичу (как и `snake_case` для файлов из `naming_conventions.md`, но в ветках принят kebab-case).

## 3. Шаблон git-блоков для шага

**Открытие фичи (первый шаг секции-фичи):**

```bash
git checkout main
git pull --ff-only
git checkout -b feat/<feature-name>
```

**Закрытие фичи (последний шаг секции-фичи, обычно шаг с тестами):**

```bash
# убедиться, что тесты зелёные
pytest -q
# зафиксировать и отправить
git add -A
git commit -m "feat(<feature>): <короткое описание>"
git push -u origin feat/<feature-name>
# открыть Merge Request в GitLab, дождаться зелёного pipeline, выполнить merge в main
git checkout main && git pull --ff-only
```

## 4. Conventional Commits (сообщения коммитов)

Формат: `<type>(<scope>): <summary>`. Типы: `feat`, `fix`, `test`, `chore`, `docs`, `refactor`.
Примеры: `feat(incomes): CRUD доходов с изоляцией владельца`, `test(goals): расчёт сводки`.

## 5. Правила для шагов книги

1. **Каждая бизнес-фича оформляется как итерация:** секция-фича открывается шагом создания ветки и закрывается шагом «тесты → commit → push → MR → merge».
2. Внутри ветки шаги по слоям чистой архитектуры — как обычно; новые шаги ветку **не создают** (она уже активна).
3. В `main` напрямую не коммитим — только через MR с зелёным CI (CI настроен в главе delivery).
4. Проверка результата git-шагов: `git branch --show-current` показывает ожидаемую ветку; после merge `git log --oneline main` содержит коммит фичи.
5. Инфраструктурные секции (Docker, CI) — ветки `chore/...` по тому же циклу.
