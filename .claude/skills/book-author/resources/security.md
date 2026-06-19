# База знаний: Безопасность (пароли, JWT, секреты)

> Источник истины для шагов про безопасность. Дата актуальности: 2026-06.

## 1. Хэширование паролей (passlib + bcrypt)

```python
# app/core/security.py
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(plain: str) -> str:
    return pwd_context.hash(plain)

def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)
```

> Пароль в открытом виде **никогда** не хранится и не логируется. В БД — только `password_hash`.

## 2. JWT (выпуск и проверка токена)

```python
# app/core/security.py (продолжение)
from datetime import datetime, timedelta, timezone
import jwt   # pyjwt
from app.core.config import settings

def create_access_token(sub: str, expires_min: int = 30) -> str:
    payload = {"sub": sub, "exp": datetime.now(timezone.utc) + timedelta(minutes=expires_min)}
    return jwt.encode(payload, settings.jwt_secret, algorithm="HS256")

def decode_token(token: str) -> str:
    data = jwt.decode(token, settings.jwt_secret, algorithms=["HS256"])
    return data["sub"]
```

## 3. Зависимость защиты роутов

```python
# app/api/deps.py
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from app.core.security import decode_token

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/v1/auth/login")

async def get_current_user(token: str = Depends(oauth2_scheme)) -> str:
    try:
        return decode_token(token)
    except Exception:
        raise HTTPException(status.HTTP_401_UNAUTHORIZED, "Invalid token")
```

## 4. Секреты и конфигурация

```python
# app/core/config.py
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    database_url: str
    jwt_secret: str
    model_config = SettingsConfigDict(env_file=".env")

settings = Settings()
```

- Секреты — только из переменных окружения / `.env` (в `.gitignore`); в CI — из защищённых переменных GitLab (см. `gitlab.md`).
- На проде — HTTPS обязателен (TLS на reverse-proxy, см. `deploy.md`).

## 5. Правила для шагов книги

1. Минимум: хэширование паролей, секреты в env, проверка токена на защищённых роутах.
2. Доменные ошибки авторизации → HTTP-коды 401/403 в роутере, не в usecase.
3. Рекомендовать rate limit на `/auth` (напр. slowapi) как NFR.
4. Проверка результата шага: запрос к защищённому роуту без токена → 401, с валидным токеном → 200.
