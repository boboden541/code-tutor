# База знаний: БД и миграции (SQLAlchemy + Alembic)

> Источник истины для шагов про базу данных. Дата актуальности: 2026-06.
> Адаптируй под СУБД/ORM из Файла №1.

## 1. Подключение (infrastructure-слой)

```python
# app/infrastructure/db.py
from collections.abc import AsyncGenerator
from sqlalchemy.ext.asyncio import AsyncSession, async_sessionmaker, create_async_engine
from app.core.config import settings

engine = create_async_engine(settings.database_url, echo=False, pool_size=10, max_overflow=20)
SessionLocal = async_sessionmaker(engine, expire_on_commit=False)

async def get_session() -> AsyncGenerator[AsyncSession, None]:
    async with SessionLocal() as session:
        yield session
```

> Сессия отдаётся в роутеры/usecase через `Depends(get_session)`. Бизнес-логика не создаёт движок сама.

## 2. ORM-модель (infrastructure, отдельно от доменной сущности)

```python
# app/infrastructure/models.py
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column

class Base(DeclarativeBase):
    pass

class UserModel(Base):
    __tablename__ = "users"
    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(unique=True, index=True)
    password_hash: Mapped[str]
```

## 3. Alembic (миграции)

```bash
pip install alembic
alembic init -t async migrations        # async-шаблон
# в migrations/env.py: target_metadata = Base.metadata, url из settings
alembic revision --autogenerate -m "create users"
alembic upgrade head
```

Откат: `alembic downgrade -1`.

## 4. Репозиторий: интерфейс + реализация

```python
# app/repositories/user_repository.py  (интерфейс — application знает только его)
from typing import Protocol
from app.domain.user import User

class UserRepository(Protocol):
    async def add(self, user: User) -> User: ...
    async def get_by_email(self, email: str) -> User | None: ...
```

```python
# app/infrastructure/user_repository_impl.py  (реализация — здесь SQL/ORM)
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession
from app.domain.user import User
from app.infrastructure.models import UserModel

class SqlUserRepository:
    def __init__(self, session: AsyncSession) -> None:
        self._s = session
    async def add(self, user: User) -> User:
        m = UserModel(email=user.email, password_hash=user.password_hash)
        self._s.add(m); await self._s.flush()
        return User(id=m.id, email=m.email, password_hash=m.password_hash)
    async def get_by_email(self, email: str) -> User | None:
        m = (await self._s.execute(select(UserModel).where(UserModel.email == email))).scalar_one_or_none()
        return User(id=m.id, email=m.email, password_hash=m.password_hash) if m else None
```

## 5. Правила для шагов книги

1. ORM-модель ≠ доменная сущность: домен в `app/domain`, ORM в `app/infrastructure`.
2. Любое изменение схемы — только через миграцию Alembic (никаких ручных ALTER).
3. Транзакции — на уровне usecase (commit после успешного сценария).
4. Проверка результата шага: `alembic upgrade head` без ошибок + таблица существует (`\dt` в psql).
