# Running the VerdeFlow agent on a Raspberry Pi 5

The agent turns the Pi into a measurement bench: for each run it **checks out**
the API repo at a commit, **builds & runs** it on the Pi, drives the endpoints,
measures power with a **user-selected method**, reports results, then tears the
API back down. It runs **natively** (systemd) — not in Docker — because reading
the Pi 5 PMIC (`vcgencmd`) and building/running arbitrary APIs need host access.

## What the Pi measures

The profiler measures the power of **the machine the agent runs on**. So the
API-under-test runs *on the Pi*, and the Pi's draw reflects that API's work.
Measurement method is chosen per run in the UI:

| Method | How | Needs |
|--------|-----|-------|
| **Pi 5 PMIC** | real board power from `vcgencmd pmic_read_adc` (Σ rail V×A) | nothing extra ✓ |
| **PowerJoular** | CPU-power model from PowerJoular's CSV | `powerjoular` installed |
| **External meter** | *stub* — returns an error until wired to real hardware | INA260 / USB meter |

## Quick setup (one script)

`scripts/setup-agent.sh` does steps 1, 2 and 4 (deps + Rust + build + install, and
optionally the systemd unit) on a Pi or any Debian/Ubuntu server:

```bash
# From a copy of this repo on the box:
./scripts/setup-agent.sh --service --with-powerjoular

# …or clone + build on a fresh box (private repos need git auth configured):
./scripts/setup-agent.sh --repo git@github.com:you/verdeflow.git --branch main --service
```

Then do step 3 (register the device) and step 5 (link a repo + build config), set
`/etc/verdeflow/vf-agent.env`, and `sudo systemctl enable --now vf-agent`. The
manual steps below are the detailed fallback.

## Updating an installed agent

To pull the latest code, rebuild, reinstall the binary, and restart the service
in one step, use `scripts/update-agent.sh` (leaves your env file and unit edits
untouched):

```bash
./scripts/update-agent.sh                # pull current branch, rebuild, restart
./scripts/update-agent.sh --branch dev   # switch branch, rebuild, restart
```

Equivalent manual sequence:

```bash
cd ~/verdeflow && git pull
cd verde-flow-agent && cargo build --release
sudo install -m 0755 target/release/vf-agent /usr/local/bin/vf-agent
sudo systemctl restart vf-agent
```

## 1. Prerequisites on the Pi

```bash
sudo apt-get update
sudo apt-get install -y git build-essential curl pkg-config libssl-dev
# Rust (to build the agent on the Pi — the Pi 5 compiles it fine):
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
# Docker — REQUIRED for runs: apps are built and run from their repo's Dockerfile
# so no runtimes (Node/Python/…) get installed on the bench:
curl -fsSL https://get.docker.com | sh && sudo usermod -aG docker $USER
# Optional, only for the PowerJoular method:
#   sudo apt-get install -y powerjoular
```

After installing Docker, let the agent service use it by adding the `docker` group
to the unit (only do this once Docker is installed, or systemd fails with
`216/GROUP`):

```bash
sudo sed -i 's/^SupplementaryGroups=video$/SupplementaryGroups=video docker/' \
  /etc/systemd/system/vf-agent.service
sudo systemctl daemon-reload && sudo systemctl restart vf-agent
```

Confirm the PMIC works: `vcgencmd pmic_read_adc` should print `*_A current(...)`
and `*_V volt(...)` lines. The `pi` user is already in the `video` group it needs.

> **Every target repo must contain a `Dockerfile`** (the default run mode builds
> and runs it). A repo without one fails the run at the *Build image* step with a
> clear message; either add a Dockerfile or set the service's build mode to
> "Host command".

## 2. Build & install the agent

```bash
# On the Pi, in a checkout of this repo:
cd verde-flow-agent
~/.cargo/bin/cargo build --release
sudo install -m 0755 target/release/vf-agent /usr/local/bin/vf-agent
```
(Or cross-compile from your laptop with `docker buildx build --platform linux/arm64`.)

## 3. Register the device (get an API key)

In the VerdeFlow UI → **Devices → Register device** → name it (e.g. `pi5-lab`) →
copy the one-time **device API key** (`vfd_…`).

## 4. Configure & start the service

```bash
sudo mkdir -p /etc/verdeflow /var/lib/verdeflow
sudo cp verde-flow-agent/deploy/vf-agent.env.example /etc/verdeflow/vf-agent.env
sudo nano /etc/verdeflow/vf-agent.env    # set DEVICE_API_KEY + GATEWAY/INGEST URLs
sudo cp verde-flow-agent/deploy/vf-agent.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now vf-agent
journalctl -u vf-agent -f                # watch it poll
```

`VERDEFLOW_GATEWAY_URL` / `VERDEFLOW_INGEST_URL` must be reachable **from the Pi**
— not `localhost`. On the same LAN use the host's IP (gateway :4000, ingest :4001).
Across networks, the gateway **reverse-proxies `/api/ingest`**, so tunnel only the
gateway and point *both* URLs at that one public address (e.g. the ngrok static
domain) — no separate ingest tunnel needed.

## 5. Point a project at a repo + set build config

1. UI → **Settings → GitHub** → connect the App and link the private repo to a project.
2. UI → open the project → **Edit** → set **Build/run**:
   - has a `Dockerfile`? leave build/run blank — the agent builds & runs it.
   - otherwise set **Build command** (e.g. `npm ci && npm run build`), **Run command**
     (e.g. `node dist/index.js` — must listen on `$PORT`), **Port**, **Health path**.
3. Add the endpoints to profile (Endpoints tab / OpenAPI import).

## 6. Trigger a run and verify

UI → project → **Trigger run** → pick **Pi 5 PMIC** (and optionally the device) → run.

The agent log should show: `clone-token → cloning → build/run → Target API is ready
→ Starting profiler (pi5_pmic) → workloads → results → Job complete`. In the UI the
run flips to **completed** with real power/energy, and the method is recorded per
measurement. Re-run the same commit with **PowerJoular** to compare instruments.

### Troubleshooting
- **Run stays queued** → no online device for the org; check `journalctl -u vf-agent`.
- **`clone-token` 404** → the repo isn't linked to the project in Settings → GitHub.
- **PMIC falls back to simulated** → `vcgencmd pmic_read_adc` failed; check the `video`
  group / that you're on a Pi 5.
- **API never ready** → wrong Port/Health path, or the app doesn't honor `$PORT`.
