# База знаний: Деплой на прод (VPS + reverse-proxy + HTTPS)

> Источник истины для шагов про продакшн-деплой. Дата актуальности: 2026-06.
> Адаптируй под хостинг из Файла №1 (VPS / облако / PaaS).

## 1. Прод-команда запуска (gunicorn + uvicorn workers)

```dockerfile
# Прод-CMD в Dockerfile (вместо dev-uvicorn)
CMD ["gunicorn", "app.main:app", \
     "-k", "uvicorn.workers.UvicornWorker", \
     "-w", "4", "-b", "0.0.0.0:8000"]
```

- Без `--reload`. Число воркеров ≈ `2 * CPU + 1`.
- Приложение stateless → масштабируется горизонтально.

## 2. Подготовка VPS

```bash
# на сервере (Ubuntu)
curl -fsSL https://get.docker.com | sh
mkdir -p /opt/app && cd /opt/app
# скопировать docker-compose.prod.yml и .env (secrets) на сервер
```

## 3. docker-compose.prod.yml (app + Nginx + БД)

```yaml
services:
  app:
    image: registry.gitlab.com/<user>/<project>:latest
    env_file: .env
    restart: always
    expose: ["8000"]
  nginx:
    image: nginx:1.27
    ports: ["80:80", "443:443"]
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - ./certs:/etc/nginx/certs:ro
    depends_on: [app]
    restart: always
```

## 4. Nginx reverse-proxy + HTTPS

```nginx
server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}
server {
    listen 443 ssl;
    server_name example.com;
    ssl_certificate     /etc/nginx/certs/fullchain.pem;
    ssl_certificate_key /etc/nginx/certs/privkey.pem;
    location / {
        proxy_pass http://app:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

TLS-сертификат: Let's Encrypt (`certbot`) или certbot-контейнер; автопродление по cron.

## 5. Выкат и smoke-тест

```bash
docker compose -f docker-compose.prod.yml pull
docker compose -f docker-compose.prod.yml up -d
docker compose -f docker-compose.prod.yml run --rm app alembic upgrade head
curl -fsS https://example.com/health    # ожидаем {"status":"ok"}
```

Откат: задеплоить предыдущий тег образа (`:<prev_sha>`) и `up -d`.

## 6. Правила для шагов книги

1. Прод — без reload, с gunicorn-воркерами, за reverse-proxy, по HTTPS.
2. Миграции применяются на проде отдельной командой перед/при выкате.
3. Секреты только из `.env`/менеджера секретов на сервере, не в образе.
4. Проверка результата (финал книги): `curl https://<домен>/health` из интернета возвращает 200 — продукт открыт на проде.
