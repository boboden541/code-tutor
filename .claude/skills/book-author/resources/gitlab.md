# База знаний: GitLab (репозиторий и CI/CD)

> Источник истины для шагов про GitLab. Дата актуальности: 2026-06.

## 1. Создание репозитория и первый пуш

```bash
git init
git add .
git commit -m "init: project skeleton"
git remote add origin git@gitlab.com:<user>/<project>.git
git push -u origin main
```

`.gitignore` (минимум): `.venv/`, `__pycache__/`, `.env`, `*.pyc`, `.pytest_cache/`.

## 2. .gitlab-ci.yml (lint → test → build)

```yaml
stages: [lint, test, build]

variables:
  PIP_CACHE_DIR: "$CI_PROJECT_DIR/.pip"

cache:
  paths: [.pip]

lint:
  stage: lint
  image: python:3.12-slim
  script:
    - pip install ruff
    - ruff check app

test:
  stage: test
  image: python:3.12-slim
  services:
    - postgres:16
  variables:
    POSTGRES_DB: app
    POSTGRES_USER: app
    POSTGRES_PASSWORD: app
    DATABASE_URL: "postgresql+psycopg://app:app@postgres:5432/app"
  script:
    - pip install -r requirements.txt pytest pytest-asyncio httpx
    - pytest -q

build:
  stage: build
  image: docker:27
  services: [docker:27-dind]
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA .
    - echo "$CI_REGISTRY_PASSWORD" | docker login -u "$CI_REGISTRY_USER" --password-stdin $CI_REGISTRY
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
  only: [main]
```

## 3. Секреты и переменные

- Settings → CI/CD → Variables: `JWT_SECRET`, `DATABASE_URL` (prod), реестр-креды.
- Помечать как **Masked** и **Protected**. Никогда не коммитить `.env`.

## 4. Правила для шагов книги

1. CI обязан гонять линт и тесты на каждый push, образ собирать только на `main`.
2. Все секреты — через переменные GitLab, не в `.gitlab-ci.yml`.
3. Проверка результата шага: пайплайн зелёный, образ появился в Container Registry.
