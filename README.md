# SentriNotes

Сервис заметок на **FastAPI + PostgreSQL + Redis Sentinel**, упакованный как цельная инфраструктурная работа с автоматизированным деплоем через **Ansible** и **GitHub Actions CI/CD**.

## Проект в двух словах

`SentriNotes` демонстрирует, как собрать прикладной backend и эксплуатационный контур вокруг него:
- API слой на FastAPI,
- персистентность в PostgreSQL,
- кэширование через Redis,
- отказоустойчивый доступ к Redis через Sentinel,
- базовая наблюдаемость через Redis Exporter,
- автоматизированный деплой и провижининг.

## CI/CD

В репозитории реализован pipeline: [`.github/workflows/ansible-cicd.yml`](/Users/sergeyromanov/api_notes_infra/.github/workflows/ansible-cicd.yml).

Что делает workflow:
- триггерится на `push` в `main` и вручную через `workflow_dispatch`,
- поднимает окружение runner’а (`ubuntu-latest`, Python 3.11, Ansible),
- настраивает SSH-доступ к серверу через GitHub Secrets,
- при `push` запускает `ansible-lab/deploy.yml` (обновление приложения),
- при ручном запуске поднимает полный стек через `ansible-lab/site.yml`.

Используемые секреты:
- `SSH_PRIVATE_KEY`
- `SERVER_HOST`
- `SERVER_USER`

Это превращает репозиторий из «набора playbook’ов» в воспроизводимый deployment-процесс.

## Архитектура

### Application Layer

Файл: [`app.py`](/Users/sergeyromanov/api_notes_infra/app.py)

- CRUD API для заметок (`/notes`, `/notes/{id}`),
- health endpoint (`/health`),
- SQLAlchemy engine для PostgreSQL,
- кэш чтений в Redis с TTL,
- инвалидация кэша при изменении данных.

### Data & Cache Layer

- PostgreSQL: источник истины,
- Redis: ускорение чтения (single note + list cache),
- Redis Sentinel: discovery мастера и failover-ready конфигурация.

### Reliability Pattern

При недоступности Redis/Sentinel сервис продолжает работу через PostgreSQL (degraded mode), а не падает целиком. Это важное эксплуатационное поведение, заложенное прямо в приложении.

## Инфраструктурная часть

Каталог: [`ansible-lab/`](/Users/sergeyromanov/api_notes_infra/ansible-lab)

Роли:
- `postgresql`: установка, создание БД/пользователя, nightly backup + retention.
- `redis`: мастер + реплики как отдельные systemd-инстансы, backup + retention.
- `sentinel`: 3 sentinel-инстанса с кворумом failover.
- `redis_exporter`: метрики Redis для мониторинга.

Отдельно реализован `deploy.yml` для жизненного цикла приложения (update/start/stop/restart).

## Что именно показывает этот проект как портфолио

- Умение проектировать не только API, но и окружение вокруг него.
- Практика IaC-подхода через Ansible roles/playbooks.
- Интеграция CI/CD с удалённым сервером по SSH.
- Базовые SRE-паттерны: healthcheck, fallback/degraded mode, резервное копирование, systemd-управление сервисами, мониторинговый exporter.

## Структура репозитория

- [`app.py`](/Users/sergeyromanov/api_notes_infra/app.py) — backend API.
- [`requirements.txt`](/Users/sergeyromanov/api_notes_infra/requirements.txt) — Python dependencies.
- [`ansible-lab/`](/Users/sergeyromanov/api_notes_infra/ansible-lab) — инфраструктура и деплой.
- [`.github/workflows/ansible-cicd.yml`](/Users/sergeyromanov/api_notes_infra/.github/workflows/ansible-cicd.yml) — CI/CD pipeline.
