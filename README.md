# homelab-docker

Docker Compose stack for my self-hosted homelab: Nginx, Prometheus, Grafana, Node Exporter.

## Services

| Service | Port | Purpose |
|---|---|---|
| nginx | 80 / 443 | reverse proxy + static |
| prometheus | 9090 | metrics collection |
| grafana | 3000 | dashboards |
| node-exporter | 9100 | host metrics |

## Run

```bash
export GF_SECURITY_ADMIN_PASSWORD=change-me
docker compose up -d
```

Grafana: http://localhost:3000 · Prometheus: http://localhost:9090

## Notes

- Alerting: Prometheus Alertmanager → Telegram webhook.
- Backups: snapshot of `prometheus_data` and `grafana_data` volumes.
