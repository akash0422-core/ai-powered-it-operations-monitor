# AI-powered IT Operations Monitor

A self-hosted monitoring starter for CPU, memory, and disk utilization, robust outlier detection, alert history, log summaries, and operational reporting.

## Run it

1. Copy `.env.example` to `.env` and optionally set `LLM_API_KEY` (an OpenAI-compatible chat completions API key). With no key, log analysis uses a local keyword-based fallback.
2. Start Docker Desktop, then run `docker compose up --build`.
3. Open the application at http://localhost:8000, interactive API documentation at http://localhost:8000/docs, and Grafana at http://localhost:3000 (default login `admin` / `admin`). Change Grafana's default password for shared deployments.

With Docker Compose, the app stores metric samples, alerts, and submitted logs in PostgreSQL. If `DATABASE_URL` is unset, it keeps recent metrics and alerts in memory instead; those are cleared when the process restarts. It samples every 15 seconds by default. Configure `CPU_THRESHOLD`, `MEMORY_THRESHOLD`, and `DISK_THRESHOLD` as percentages; alerts can be sent to a JSON webhook using `ALERT_WEBHOOK_URL`. The anomaly detector compares samples against a rolling 30-sample median and median absolute deviation after it has at least 10 baseline samples.

## Render demo deployment

The included `render.yaml` configures a free Render web service. In the Render dashboard, create a Blueprint from this repository and select the `render.yaml` file. The demo runs without a database and keeps its most recent metrics and alerts in memory, so a restart clears its history. Render's free web services may sleep after inactivity. For durable history, deploy with PostgreSQL and set `DATABASE_URL` to the database's internal connection string.

## API

- `GET /health`
- `GET /api/metrics/latest`
- `GET /api/metrics/history?limit=100`
- `GET /api/alerts?status=open`
- `POST /api/alerts/{id}/resolve`
- `POST /api/logs/analyze` with `{"host":"web-01","logs":["ERROR ..."]}`
- `GET /api/reports/daily`

## Notes

The collector monitors the container's visible resources. For host-level visibility when running in Docker, platform-specific host mounts or a host agent may be needed; Linux host deployments can use host PID and filesystem mounts with the corresponding `MONITOR_DISK_PATH`. The included Grafana dashboard queries the provisioned PostgreSQL datasource. Protect the dashboard/API and replace sample credentials before exposing the service beyond a trusted local network. LLM summaries send submitted log text to the configured LLM provider; avoid sending secrets or personal data.
