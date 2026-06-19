# База знаний: Тестирование (pytest + httpx)

> Источник истины для шагов про тесты. Дата актуальности: 2026-06.

## 1. Зависимости

```bash
pip install pytest pytest-asyncio httpx
```

## 2. Фикстуры (conftest.py)

```python
# tests/conftest.py
import pytest
from httpx import ASGITransport, AsyncClient
from app.main import app

@pytest.fixture
async def client():
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url="http://test") as c:
        yield c
```

> Для БД-тестов: отдельная тестовая БД или транзакция с откатом после каждого теста; репозиторий подменяется через переопределение зависимости `app.dependency_overrides`.

## 3. Тест эндпоинта

```python
# tests/test_health.py
import pytest

@pytest.mark.asyncio
async def test_health(client):
    r = await client.get("/health")
    assert r.status_code == 200
    assert r.json() == {"status": "ok"}
```

## 4. Тест с подменой репозитория (изоляция слоёв)

```python
# usecase тестируется с фейковым репозиторием — без БД
class FakeUserRepo:
    def __init__(self): self._db = {}
    async def add(self, user): self._db[user.email] = user; return user
    async def get_by_email(self, email): return self._db.get(email)
```

## 5. Запуск

```bash
pytest -q
pytest --cov=app           # с покрытием (pip install pytest-cov)
```

## 6. Правила для шагов книги

1. Пирамида тестов: больше unit (usecase с фейками), меньше интеграционных (через ASGI-клиент).
2. Каждый API-шаг сопровождается шагом с тестом (happy-path + минимум один краевой случай: 401/403/404/409).
3. Тесты не ходят во внешний интернет и не зависят от порядка выполнения.
4. Проверка результата шага: `pytest -q` зелёный.
