# devops

Автодеплой сервисов на удаленную машину через GitHub Actions при merge в `main`.

## Что уже настроено

- `services/n8n/docker-compose.yml` — отдельный compose для `n8n`.
- `services/postgres/docker-compose.yml` — отдельный compose для `postgres`.
- `.github/workflows/deploy-services.yml` — отдельные job для каждого сервиса, запуск только из ветки `main`.
- `services/*/env.example` — шаблоны переменных окружения.

## Как это работает

1. Вы делаете merge в `main`.
2. GitHub Actions подключается к вашей машине по SSH.
3. Копирует файлы в `/opt/telemanga-devops`.
4. Job `deploy_n8n` поднимает `n8n` через его отдельный compose-файл.
5. Job `deploy_postgres` поднимает `postgres` через его отдельный compose-файл (если заданы `POSTGRES_*` secrets).

## Подготовка машины (192.168.0.33)

На целевой машине должны быть:

- Docker
- Docker Compose plugin (`docker compose`)
- Открыт SSH доступ снаружи (белый IP / проброшенный порт)

Создайте рабочую папку:

```bash
sudo mkdir -p /opt/telemanga-devops
sudo chown -R "$USER":"$USER" /opt/telemanga-devops
```

## GitHub Secrets

В репозитории `Settings -> Secrets and variables -> Actions` добавьте:

- `SSH_HOST` — белый IP или DNS вашей машины.
- `SSH_PORT` — обычно `22`.
- `SSH_USER` — пользователь на сервере.
- `SSH_PRIVATE_KEY` — приватный ключ для SSH (лучше отдельный deploy key).
- `N8N_BASIC_AUTH_USER` — логин для входа в n8n.
- `N8N_BASIC_AUTH_PASSWORD` — пароль для входа в n8n.
- `N8N_ENCRYPTION_KEY` — длинный случайный ключ для шифрования данных n8n.
- `GENERIC_TIMEZONE` — например `Asia/Tashkent`.
- `POSTGRES_DB` — имя базы для postgres.
- `POSTGRES_USER` — пользователь postgres.
- `POSTGRES_PASSWORD` — пароль postgres.

## Переменные окружения сервисов

`.env` для каждого сервиса создается автоматически в workflow из GitHub Secrets при каждом деплое:

- `/opt/telemanga-devops/services/n8n/.env`
- `/opt/telemanga-devops/services/postgres/.env`

Локально `services/*/env.example` используйте только как шаблон.

## Как добавить новый сервис

1. Создать `services/<name>/docker-compose.yml`.
2. Добавить `services/<name>/env.example`.
3. Добавить отдельный job `deploy_<name>` в workflow.
4. Добавить нужные `<NAME>_*` secrets в GitHub.