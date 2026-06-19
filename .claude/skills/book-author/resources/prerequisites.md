# База знаний: подготовка окружения (глава 0)

> Источник истины для шагов главы 0 «Подготовка окружения». Дата актуальности: 2026-06.
> Используется, если профиль читателя — junior / «чистая машина». ОС берётся из Файла №2.
> Каждый шаг главы 0 заканчивается **проверкой версии** (команда + ожидаемый вывод).

## 1. Python 3.12

| ОС | Установка | Проверка |
|----|-----------|----------|
| macOS | `brew install python@3.12` (Homebrew) | `python3.12 --version` → `Python 3.12.x` |
| Ubuntu/Debian | `sudo apt update && sudo apt install -y python3.12 python3.12-venv` | `python3.12 --version` |
| Windows | установщик с python.org, галочка «Add to PATH» | `py -3.12 --version` |

Что объяснить новичку: **интерпретатор** Python запускает код; версия важна, т.к. проект использует синтаксис 3.12 (например `dict[str, str]`).

## 2. Docker и docker compose

| ОС | Установка | Проверка |
|----|-----------|----------|
| macOS / Windows | Docker Desktop (docker.com) | `docker --version` и `docker compose version` |
| Ubuntu | `curl -fsSL https://get.docker.com | sh`, затем `sudo usermod -aG docker $USER` (перелогиниться) | `docker run hello-world` → «Hello from Docker!» |

Что объяснить: **Docker** запускает приложение и БД в изолированных контейнерах — одинаково на любой машине; **образ** — шаблон, **контейнер** — запущенный экземпляр.

## 3. Git

| ОС | Установка | Проверка |
|----|-----------|----------|
| macOS | `brew install git` (или Xcode CLT) | `git --version` |
| Ubuntu | `sudo apt install -y git` | `git --version` |
| Windows | git-scm.com (Git for Windows) | `git --version` |

Первичная настройка: `git config --global user.name "..."` и `git config --global user.email "..."`.

## 4. SSH-ключ для GitLab

```bash
ssh-keygen -t ed25519 -C "you@example.com"   # Enter на все вопросы
cat ~/.ssh/id_ed25519.pub                     # скопировать вывод
# GitLab → Preferences → SSH Keys → вставить → Add key
ssh -T git@gitlab.com                         # проверка
```

Ожидаемый вывод проверки: `Welcome to GitLab, @<username>!`.

Что объяснить: **SSH-ключ** — пара «приватный (на вашей машине) + публичный (в GitLab)»; позволяет пушить без ввода пароля и подтверждает вашу личность.

## 5. Правила для шагов главы 0

1. Один инструмент = один шаг (`install_python`, `install_docker`, `install_git_and_ssh`), затем `recap_and_state`.
2. В каждом шаге — раздел под ОС читателя (из Файла №2), не вываливать все ОС, если ОС известна.
3. Каждый шаг заканчивается **проверкой версии** с ожидаемым выводом.
4. Глава 0 не содержит кода приложения — только установка и проверка инструментов.
