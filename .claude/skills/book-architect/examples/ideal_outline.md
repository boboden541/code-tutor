# Книга: <Название проекта>

> Сгенерировано `/tutor-plan` из `project_brief.md` + `requirements.md`.
> Формат строки шага (контракт для `/tutor-write`): `- [ ] <путь>.md — <назначение>`
> `[ ]` — не написан, `[x]` — написан и принят. Разделитель назначения — « — » (em dash).
> Файлы-шаги имеют 2-значный префикс порядка `<KK>_` (`01_`, `02_`, …) — порядок виден в папке.
> Глава 0 — только если профиль читателя junior/«чистая машина». Первая доменная фича — по слою на шаг.

## chapter_0_prerequisites

### section_1_setup_environment
- [ ] book/chapter_0_prerequisites/section_1_setup_environment/01_install_python.md — установить Python 3.12 под свою ОС и проверить версию
- [ ] book/chapter_0_prerequisites/section_1_setup_environment/02_install_docker.md — установить Docker и docker compose, проверить запуск
- [ ] book/chapter_0_prerequisites/section_1_setup_environment/03_install_git_and_ssh.md — установить git, создать SSH-ключ и добавить в GitLab
- [ ] book/chapter_0_prerequisites/section_1_setup_environment/04_recap_and_state.md — рекап окружения: какие версии должны быть установлены

## chapter_1_starting_project

### section_1_create_project
- [ ] book/chapter_1_starting_project/section_1_create_project/01_create_project_in_gitlab.md — создать репозиторий в GitLab, README, .gitignore, первый коммит
- [ ] book/chapter_1_starting_project/section_1_create_project/02_create_codebase_architecture.md — создать структуру каталогов чистой архитектуры
- [ ] book/chapter_1_starting_project/section_1_create_project/03_setup_python_env.md — настроить venv и зависимости (FastAPI, uvicorn)
- [ ] book/chapter_1_starting_project/section_1_create_project/04_how_we_work_git_flow.md — памятка: одна фича = ветка feat/ = Merge Request

### section_2_make_first_api
- [ ] book/chapter_1_starting_project/section_2_make_first_api/01_create_first_api.md — поднять FastAPI и эндпоинт /health
- [ ] book/chapter_1_starting_project/section_2_make_first_api/02_setup_docker.md — Dockerfile и docker-compose, запуск в контейнере
- [ ] book/chapter_1_starting_project/section_2_make_first_api/03_test_first_api.md — первый тест на /health и запуск pytest
- [ ] book/chapter_1_starting_project/section_2_make_first_api/04_recap_and_state.md — рекап главы 1 + полный requirements.txt + дерево проекта

## chapter_2_<домен_первой_фичи>

### section_0_request_flow_overview
- [ ] book/chapter_2_<домен>/section_0_request_flow_overview/01_request_flow_overview.md — обзор: как запрос проходит через слои (router→usecase→repository→БД)

### section_1_<первая_фича>
> Первая фича — ОДИН СЛОЙ = ОДИН ШАГ, полный код + «Разбор кода» в каждом шаге.
- [ ] book/chapter_2_<домен>/section_1_<фича>/01_start_feature_branch.md — ветка feat/<фича> от main
- [ ] book/chapter_2_<домен>/section_1_<фича>/02_<фича>_schema.md — Pydantic-схемы запроса/ответа (DTO)
- [ ] book/chapter_2_<домен>/section_1_<фича>/03_<фича>_domain.md — доменная сущность и бизнес-правила
- [ ] book/chapter_2_<домен>/section_1_<фича>/04_<фича>_repository_interface.md — интерфейс репозитория (Protocol)
- [ ] book/chapter_2_<домен>/section_1_<фича>/05_<фича>_repository_impl.md — реализация репозитория на SQLAlchemy
- [ ] book/chapter_2_<домен>/section_1_<фича>/06_<фича>_migration.md — ORM-модель и миграция Alembic
- [ ] book/chapter_2_<домен>/section_1_<фича>/07_<фича>_usecase.md — usecase сценария (создание/список/удаление)
- [ ] book/chapter_2_<домен>/section_1_<фича>/08_<фича>_router.md — роутер и сборка зависимостей (Depends), маппинг ошибок в HTTP
- [ ] book/chapter_2_<домен>/section_1_<фича>/09_<фича>_test.md — тесты фичи (happy-path + краевые случаи)
- [ ] book/chapter_2_<домен>/section_1_<фича>/10_commit_push_open_mr.md — commit, push, Merge Request, merge в main
- [ ] book/chapter_2_<домен>/section_1_<фича>/11_recap_and_state.md — рекап главы + полный requirements.txt + дерево

## chapter_3_delivery

### section_1_ci_pipeline
- [ ] book/chapter_3_delivery/section_1_ci_pipeline/01_start_feature_branch.md — ветка chore/gitlab-ci от main
- [ ] book/chapter_3_delivery/section_1_ci_pipeline/02_setup_gitlab_ci.md — .gitlab-ci.yml: lint, тесты, сборка образа
- [ ] book/chapter_3_delivery/section_1_ci_pipeline/03_configure_secrets_and_env.md — переменные окружения и секреты в GitLab
- [ ] book/chapter_3_delivery/section_1_ci_pipeline/04_commit_push_open_mr.md — commit, push, MR, merge в main

### section_2_deploy_to_prod
- [ ] book/chapter_3_delivery/section_2_deploy_to_prod/01_prepare_prod_server.md — VPS: Docker, Nginx reverse-proxy, HTTPS
- [ ] book/chapter_3_delivery/section_2_deploy_to_prod/02_deploy_and_smoke_test.md — выкатить контейнер, миграции, проверить /health из интернета
- [ ] book/chapter_3_delivery/section_2_deploy_to_prod/03_recap_and_state.md — финальный рекап: что построено, как развивать дальше

> **Вторая и последующие фичи** — слои сгруппированы в 1–2 шага и «по образцу» со ссылкой на первую фичу (см. `resources/delivery_pipeline.md`).
