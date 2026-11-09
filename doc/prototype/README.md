# VerdeFlow

> Commit-aware API energy profiling for green software development.

VerdeFlow integrates energy profiling directly into developer workflows — every
git commit triggers an automated energy measurement run, results appear in a
dashboard, and regressions are flagged before they ship.

## Repository Structure

```
verdeflow/
├── verde-flow-api/      TypeScript pnpm monorepo — control plane
│   ├── apps/
│   │   ├── gateway/     Public REST API  (port 4000)
│   │   ├── ingest/      Metric ingestion (port 4001)
│   │   ├── scheduler/   Job dispatch     (port 4002)
│   │   └── estimator/   Energy/carbon    (port 4003)
│   └── packages/
│       ├── domain/      Shared TypeScript types
│       ├── storage/     PostgreSQL repositories
│       ├── config/      Environment loading
│       └── telemetry/   Structured logging
├── verde-flow-ui/       React + Vite dashboard (port 3000)
├── verde-flow-agent/    Rust profiling agent (runs on Pi)
├── sample-api/          5-variant demo API for benchmarking
└── docker-compose.yml   Full stack orchestration
```

## Quick Start

See **SETUP.md** for the full guide.

```bash
cp .env.example .env
docker compose up --build -d
# Open http://localhost:3000
```

Manage the stack with the helper script (wraps `docker compose`):

```bash
./scripts/vf.sh status               # show all services
./scripts/vf.sh restart gateway      # start / stop / restart any service
./scripts/vf.sh start backend        # backend = gateway ingest estimator scheduler
./scripts/vf.sh health               # /health check the backend apps
```

To connect GitHub repos (OAuth via a GitHub App + ngrok tunnel), see
**docs/GITHUB_APP_SETUP.md**. The tunnel is not a Docker service, so start it
separately (it does not auto-start on reboot):

```bash
./scripts/ngrok.sh start             # background gateway tunnel
./scripts/ngrok.sh status            # running? + public URL
```

## Paper

Joof, M.B., Khan, M.A., Oyedeji, S., Porras, J. (2025).
_Integrating API Energy Profiling into Developer Workflows: The VerdeFlow Prototype._
LUT University, Lappeenranta, Finland.

## License

Apache 2.0 (core). See LICENSE.
