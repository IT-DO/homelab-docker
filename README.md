# homelab-docker

Стек self-hosted сервисов для домашней инфраструктуры на **Docker Compose**: reverse proxy, сбор метрик и дашборды. Поднимается одной командой, конфиги — в репозитории.

## Что внутри

| Сервис | Порт | Назначение |
|---|---|---|
| `nginx` | 80 / 443 | reverse proxy, статика, единая точка входа |
| `prometheus` | 9090 | сбор метрик, `scrape_interval: 15s` |
| `grafana` | 3000 | дашборды и алерты |
| `node-exporter` | 9100 | метрики хоста (CPU / RAM / диск / сеть) |

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
export GF_SECURITY_ADMIN_PASSWORD='change-me'   # пароль админа Grafana
docker compose up -d
docker compose ps
```

- Grafana: http://localhost:3000 — логин `admin`, пароль из `GF_SECURITY_ADMIN_PASSWORD`
- Prometheus: http://localhost:9090

## Проверка

```bash
curl -s localhost:9090/-/healthy            # Prometheus жив
curl -s localhost:9100/metrics | head       # метрики хоста
docker compose logs -f --tail=50            # логи всех сервисов
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

## Заметки

- Пароль Grafana приходит из окружения и в репозиторий не коммитится.
- Алерты: Prometheus Alertmanager → Telegram webhook (в планах).
- Бэкап: снапшот томов `prometheus_data` и `grafana_data`.
- HTTPS: сертификаты кладутся в `nginx/conf`, порт 443 уже проброшен.
