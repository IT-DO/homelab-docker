# homelab-docker

Стек self-hosted сервисов для домашней инфраструктуры на **Docker Compose**: reverse proxy, сбор метрик и дашборды. Поднимается одной командой, конфиги — в репозитории.

## Что внутри

| Сервис | Порт | Назначение |
|---|---|---|
| `nginx` | 80 / 443 | reverse proxy, статика, единая точка входа |
| `prometheus` | внутренний 9090 | сбор метрик, `scrape_interval: 15s` |
| `grafana` | `127.0.0.1:3000` | дашборды и алерты, только локально |
| `node-exporter` | внутренний 9100 | метрики хоста (CPU / RAM / диск / сеть) |

## Схема

```
        :80/:443
            │
       ┌────▼─────┐
       │  nginx   │  reverse proxy
       └────┬─────┘
            │
    ┌───────┴────────┐
    ▼                ▼
┌──────────┐   ┌──────────────┐
│ grafana  │◄──│  prometheus  │──► node-exporter :9100
│  :3000   │   │    :9090     │
└──────────┘   └──────────────┘
```

## Запуск

```bash
export GF_SECURITY_ADMIN_PASSWORD='change-me'   # пароль админа Grafana, обязателен
docker compose up -d
docker compose ps
```

- Grafana: http://localhost:3000 — логин `admin`, пароль из `GF_SECURITY_ADMIN_PASSWORD`
- Prometheus и node-exporter наружу не публикуются: доступ к ним — только изнутри compose-сети

## Проверка

```bash
curl -s localhost:3000/api/health            # Grafana: {"database":"ok"}
docker compose ps                            # все сервисы Up
docker compose logs -f --tail=50             # логи всех сервисов
```

## Структура

```
docker-compose.yml          сервисы, порты, тома, restart: unless-stopped
prometheus/prometheus.yml   scrape-конфиг (Prometheus + node-exporter)
nginx/conf/default.conf     пример reverse proxy (замените апстримы на свои)
```

## Что демонстрирует

- сервис-ориентированный compose-стек с зависимостями (`depends_on`) и политикой перезапуска;
- сбор метрик Prometheus и визуализацию в Grafana;
- единую точку входа через nginx;
- персистентность данных в именованных томах и их бэкап.

## Безопасность

- **Порт Grafana привязан к `127.0.0.1`** — снаружи и из LAN он недоступен. Нужен доступ с другого хоста: `ssh -L 3000:localhost:3000 user@host`.
- **Prometheus и node-exporter не публикуют порты** (`expose` вместо `ports`): метрики собираются внутри compose-сети. Открытый `9100` — это утечка сведений о хосте, открытый `9090` — доступ к метрикам без авторизации.
- **Пароль Grafana обязателен**: `${GF_SECURITY_ADMIN_PASSWORD:?...}` — без переменной `docker compose up` падает, а не поднимает админку с дефолтом.
- **Самописные пароли в compose не хранятся**: только переменные окружения, `.env` — в `.gitignore`.
- Перед публикацией портов наружу — reverse proxy с TLS и авторизацией (nginx уже в стеке).

## Заметки

- Пароль Grafana приходит из окружения и в репозиторий не коммитится.
- Алерты: Prometheus Alertmanager → Telegram webhook (в планах).
- Бэкап: снапшот томов `prometheus_data` и `grafana_data`.
- HTTPS: сертификаты кладутся в `nginx/conf`, порт 443 уже проброшен.
