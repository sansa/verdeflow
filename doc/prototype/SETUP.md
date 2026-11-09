# VerdeFlow — Setup Guide

## Prerequisites

| Tool | Version | Purpose |
|------|---------|---------|
| Docker + Docker Compose | v25+ | Run all services |
| Node.js | v20+ | UI development |
| pnpm | v9+ | Backend monorepo |
| Rust + Cargo | v1.78+ | Build the agent locally |
| Git | any | Required for webhooks |

---

## Quick Start (full stack via Docker)

```bash
# 1. Clone / unzip the codebase
cd verdeflow

# 2. Copy env file
cp .env.example .env
# Edit .env if needed — defaults work for local dev

# 3. Start everything
docker compose up --build -d

# 4. Verify all services are healthy
docker compose ps
```

Services will be available at:

| Service | URL |
|---------|-----|
| UI Dashboard | http://localhost:3000 |
| Gateway API | http://localhost:4000 |
| Ingest API | http://localhost:4001 |
| Scheduler | http://localhost:4002 |
| Estimator | http://localhost:4003 |
| MinIO Console | http://localhost:9001 |
| Sample API | http://localhost:8000 |

---

## Managing services (`scripts/vf.sh`)

A small wrapper around `docker compose` for starting, stopping, and inspecting
the stack. Runs from any directory.

```bash
./scripts/vf.sh <command> [service ...]
```

| Command | What it does | Default target |
|---------|--------------|----------------|
| `start   [svc...]`  | `up -d` — create/run (applies `docker-compose.yml` / `.env` changes) | all |
| `stop    [svc...]`  | Stop the container(s) | all |
| `restart [svc...]`  | Bounce the container(s) (keeps existing config) | all |
| `rebuild [svc...]`  | `build --no-cache` then `up -d` | all |
| `status`            | `docker compose ps` | — |
| `logs    <svc>`     | Follow one service's logs (Ctrl-C to exit) | — |
| `health`            | Curl `/health` for gateway, ingest, scheduler | — |

Service aliases: **`backend`** = `gateway ingest estimator scheduler`;
**`all`** (or omitted) = every service.

```bash
./scripts/vf.sh start backend        # (re)create the 4 apps with latest config
./scripts/vf.sh restart gateway      # bounce just the gateway
./scripts/vf.sh logs estimator       # tail estimator logs
./scripts/vf.sh health               # quick 200 check
./scripts/vf.sh stop                 # stop everything
```

> **`start` vs `restart`:** after editing `docker-compose.yml` or `.env`, use
> **`start`** (`up -d`) so the new config/env is applied. Plain `restart` bounces
> the process but keeps the old container config.

### The ngrok tunnel (`scripts/ngrok.sh`)

Only needed for GitHub App OAuth (see **docs/GITHUB_APP_SETUP.md**). The tunnel
is **not** a Docker service, so it does **not** auto-start on reboot — start it
separately with the helper:

```bash
./scripts/ngrok.sh start     # background gateway tunnel (port 4000)
./scripts/ngrok.sh status    # running? + public URL
./scripts/ngrok.sh health    # curl /health through the public domain
./scripts/ngrok.sh stop      # stop it
```

---

## First Login

Default dev credentials (seeded by the migration):
- **Email:** admin@verdeflow.local
- **Password:** verdeflow_dev

---

## Create Your First Project

```bash
# Get a JWT
TOKEN=$(curl -s -X POST http://localhost:4000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@verdeflow.local","password":"verdeflow_dev"}' \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['token'])")

# Create a project pointing at the sample API
curl -s -X POST http://localhost:4000/api/projects \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Sample API",
    "targetApiUrl": "http://sample-api:8000",
    "repositoryUrl": "http://localhost/sample-api",
    "defaultBranch": "main"
  }' | python3 -m json.tool
```

Save the `id` from the response — that's your `PROJECT_ID`.

---

## Add Endpoint Profiles

```bash
for EP in \
  '{"method":"GET","path":"/api/users","weight":30}' \
  '{"method":"GET","path":"/api/products","weight":30}' \
  '{"method":"POST","path":"/api/compute","weight":20,"requestBody":{"n":10000}}' \
  '{"method":"GET","path":"/api/stats","weight":20}'; do
  curl -s -X POST "http://localhost:4000/api/projects/$PROJECT_ID/endpoints" \
    -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    -d "$EP"
done
```

---

## Register a Device (Agent)

```bash
# Create an API key for the agent
KEY_RESP=$(curl -s -X POST http://localhost:4000/api/auth/api-keys \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Local Dev Agent","scopes":["device:register","device:poll","ingest:write"]}')

DEVICE_KEY=$(echo $KEY_RESP | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['key'])")
echo "Device key: $DEVICE_KEY"
```

Add `DEVICE_API_KEY=$DEVICE_KEY` to your `.env`, then restart the agent:
```bash
docker compose restart agent
```

---

## Run the 5-Variant Demo

```bash
cd sample-api
export VERDE_GATEWAY=http://localhost:4000
export VERDE_PROJECT_ID=<your-project-id>
bash scripts/simulate-commits.sh
```

This will:
1. Create 5 git commits (baseline → heavy → cached → compressed → refactored)
2. Trigger a profiling run for each via the webhook endpoint
3. Wait ~60s between runs for profiling to complete

Then open http://localhost:3000 and navigate to your project to see the energy trend chart.

---

## Install the Git Hook (for your own repo)

```bash
# Inside your own API repo
curl -O http://localhost:4000/webhooks/hook-installer.sh
# Or manually:
cp /path/to/verdeflow/sample-api/.git-hooks/post-commit .git/hooks/post-commit
chmod +x .git/hooks/post-commit

# Set env vars
export VERDE_GATEWAY=http://localhost:4000
export VERDE_PROJECT_ID=<your-project-id>
```

Every `git commit` will now automatically trigger a profiling run.

---

## Deploying the Agent on Raspberry Pi

```bash
# Cross-compile on your dev machine (requires cross-rs or Docker buildx)
cd verde-flow-agent

# Option A: Docker buildx (recommended)
docker buildx build --platform linux/arm64 -t vf-agent:arm64 --load .

# Option B: native cross-compile (requires arm64 toolchain)
rustup target add aarch64-unknown-linux-musl
cargo build --release --target aarch64-unknown-linux-musl

# Copy to Pi
scp target/aarch64-unknown-linux-musl/release/vf-agent pi@<PI_IP>:~/vf-agent

# On the Pi:
VERDEFLOW_GATEWAY_URL=http://<YOUR_LAPTOP_IP>:4000 \
VERDEFLOW_INGEST_URL=http://<YOUR_LAPTOP_IP>:4001 \
VERDEFLOW_DEVICE_API_KEY=<device-key> \
VERDEFLOW_DEVICE_NAME=pi-4-lab \
./vf-agent
```

Install PowerJoular on the Pi:
```bash
# On Raspberry Pi OS
sudo apt-get install -y powerjoular
# Or build from source: https://github.com/joularjx/powerjoular
```

---

## Development Workflow with Claude Code

```bash
# Backend (inside verde-flow-api/)
cd verde-flow-api && claude "add GraphQL support to gateway"

# Frontend (inside verde-flow-ui/)
cd verde-flow-ui && claude "add a comparison view between two commits"

# Agent (inside verde-flow-agent/)
cd verde-flow-agent && claude "add retry logic with exponential backoff in runner.rs"

# Full stack
claude "add multi-profiler support — allow JoularJX as an alternative to PowerJoular"
```

---

## Architecture Reminder

```
Developer commits
      │
      ▼
Git post-commit hook
      │ POST /webhooks/git
      ▼
Gateway (port 4000)
  ├── Creates Run record
  ├── Enqueues ScheduledJob
      │
      ▼ (long-poll)
Agent (Raspberry Pi)
  ├── Starts PowerJoular
  ├── Executes workload (HTTP requests to target API)
  ├── Stops PowerJoular
  ├── POSTs raw samples → Ingest (port 4001)
  └── POSTs run-complete → Ingest
            │
            ▼
        Estimator (port 4003)
  ├── Aggregates power samples
  ├── Computes energy, carbon, latency metrics
  ├── Writes to measurements table
  ├── Checks for regressions → creates alerts
            │
            ▼
        Dashboard (port 3000)
  └── Shows commit-level energy trend chart
```
