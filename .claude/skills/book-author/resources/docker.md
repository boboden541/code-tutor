# База знаний: Docker для FastAPI

> Источник истины для шагов про контейнеризацию. Дата актуальности: 2026-06.
> Адаптируй версии и образы под стек из Файла №1.

## 1. Dockerfile (multi-stage, prod-ready)

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.12-slim AS base
ENV PYTHONUNBUFFERED=1 PYTHONDONTWRITEBYTECODE=1
WORKDIR /app

FROM base AS deps
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

FROM deps AS runtime
COPY app ./app
EXPOSE 8000
# uvicorn для dev; на проде — gunicorn с uvicorn worker (см. deploy.md)
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## 2. docker-compose.yml (локальная разработка: app + БД)

```yaml
services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql+psycopg://app:app@db:5432/app
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - ./app:/app/app   # hot-reload кода в dev
  db:
    image: postgres:16
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app
      POSTGRES_DB: app
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app"]
      interval: 5s
      retries: 5
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
```

## 3. .dockerignore

```
__pycache__/
*.pyc
.venv/
.git/
.env
tests/
```

## 4. Базовые команды

```bash
docker compose up --build        # собрать и запустить
docker compose logs -f app       # логи приложения
docker compose exec app bash     # шелл внутри контейнера
docker compose down              # остановить
```

## 5. Правила для шагов книги

1. В dev — `--reload` и volume для горячей перезагрузки; на проде — без reload (см. `deploy.md`).
2. Секреты — только через переменные окружения, никогда в образ (см. `security.md`).
3. Образ — `slim`, multi-stage, без dev-зависимостей в runtime.
4. Проверка результата шага: `curl http://localhost:8000/health` → `{"status":"ok"}`.
