# GitHub App Setup for VerdeFlow

## User experience

Users connecting their GitHub account only need to:

1. Click "Connect GitHub" in VerdeFlow Settings
2. Approve access on GitHub (select account, choose repos)
3. That's it — no manual webhook setup required

GitHub App installations handle webhooks automatically for all repos
the user grants access to. Pushes to connected repos trigger energy
profiling runs.

## How the flow works

1. User clicks "Connect GitHub" → gateway `GET /api/github/connect`
   returns the install URL `github.com/apps/verdeflow-dev/installations/new`
2. User selects account/repos → GitHub redirects to the App's **Setup URL**
   → `https://<public-url>/api/github/callback?installation_id=XXX&state=YYY`
3. Gateway validates `state`, generates an App JWT (signed with the private
   key), exchanges it for an installation access token, stores the connection
4. Gateway redirects the browser back to `FRONTEND_URL/settings?github=connected`
5. Pushes to connected repos hit `POST /api/github/webhook` → runs are queued

The callback and webhook are **gateway** routes (port 4000). GitHub must
reach them over the public internet, so the gateway is exposed via an ngrok
tunnel. The `/api/github/callback` **path is required** — pointing GitHub at
a bare tunnel root will just land the user on a 404.

---

## Public URL — ngrok static domain (recommended)

ngrok's free tier rotates the tunnel URL on every restart, which breaks the
GitHub App config each time. Reserve **one free static domain** so you set the
GitHub App URLs once and never touch them again.

1. Reserve a domain at https://dashboard.ngrok.com/domains → **New Domain**
   (free plan gives one, e.g. `hydrothermal-reda-nonmineralogically.ngrok-free.dev`).
2. Add a named endpoint to `~/Library/Application Support/ngrok/ngrok.yml`
   (config schema v3) that tunnels the domain to the **gateway on port 4000**:

   ```yaml
   version: "3"
   agent:
       authtoken: <your-agent-authtoken>
   endpoints:
     - name: gateway
       url: https://YOUR-STATIC-DOMAIN.ngrok-free.dev
       upstream:
         url: http://localhost:4000
   ```

3. Start the tunnel with the helper (runs it in the background, waits for the
   tunnel, and prints the live URL):

   ```bash
   ./scripts/ngrok.sh start
   ```

   (Equivalent to `ngrok start gateway` — see **Managing the tunnel** below.)

   Verify it reaches the gateway:
   `./scripts/ngrok.sh health` → `.../health -> 200`

> The tunnel **must** point at port **4000** (gateway), not 3000 (UI).
> The callback/webhook routes live on the gateway.

### Managing the tunnel (`scripts/ngrok.sh`)

ngrok is **not** a Docker service, so it does **not** auto-start on reboot the
way the containers do (`restart: unless-stopped`). After a reboot the apps come
back but the tunnel does not — start it manually with the helper:

| Command | What it does |
|---------|--------------|
| `./scripts/ngrok.sh start`   | Start the tunnel in the background (no-op if already running) |
| `./scripts/ngrok.sh stop`    | Stop the tunnel |
| `./scripts/ngrok.sh restart` | Stop then start |
| `./scripts/ngrok.sh status`  | Running? + current public URL |
| `./scripts/ngrok.sh url`     | Print just the public URL (for scripting) |
| `./scripts/ngrok.sh health`  | Curl `/health` through the public domain |
| `./scripts/ngrok.sh logs`    | Tail the tunnel log |

Typical post-reboot startup:

```bash
./scripts/vf.sh start        # containers (usually already up via unless-stopped)
./scripts/ngrok.sh start     # the tunnel — the manual step
```

---

## Creating the GitHub App

1. Go to https://github.com/settings/apps/new
2. Fill in:
   - **App name**: `verdeflow-dev` (this becomes the slug / `GITHUB_APP_SLUG`)
   - **Homepage URL**: `http://localhost:3000`
   - **Callback URL**: `https://YOUR-STATIC-DOMAIN.ngrok-free.dev/api/github/callback`
   - **Setup URL (post-installation)**: same as the Callback URL, and check
     **"Redirect on update"** so re-installs also trigger the callback
   - **Webhook URL**: `https://YOUR-STATIC-DOMAIN.ngrok-free.dev/api/github/webhook`
   - **Webhook secret**: generate a random secret
3. Permissions:
   - Repository → Contents → Read-only
   - Repository → Metadata → Read-only
4. Subscribe to events: **Push**
5. Where can this app be installed? **Any account**
6. Click "Create GitHub App", then generate a **private key** and save it
   to repo-root `.secrets/` (gitignored — see below).

---

## Environment variables

The **Dockerized** gateway reads the **repo-root `.env`** (via docker-compose
`env_file: .env`) — *not* `apps/gateway/.env` (that one is only used by local
`pnpm dev`). Set these in repo-root `.env`:

```
GITHUB_APP_ID=3713652
GITHUB_APP_CLIENT_ID=Iv23li...
GITHUB_APP_CLIENT_SECRET=a494d3...
GITHUB_APP_SLUG=verdeflow-dev
GITHUB_APP_PRIVATE_KEY_PATH=./.secrets/verdeflow-dev.<date>.private-key.pem
GITHUB_WEBHOOK_SECRET=your-webhook-secret
PUBLIC_URL=https://YOUR-STATIC-DOMAIN.ngrok-free.dev
FRONTEND_URL=http://localhost:3000
```

Notes:
- **`FRONTEND_URL` must be `http://localhost:3000`** — that's the UI container's
  port. The callback redirects the browser here; a wrong port lands the user
  on a dead page even after a successful connect.
- `GITHUB_APP_PRIVATE_KEY_PATH` is resolved relative to the gateway's working
  directory (`/app` in the container), so `./.secrets/...` → `/app/.secrets/...`.

---

## Private key & the `.secrets` mount

The private key lives in repo-root **`.secrets/`**, which is **gitignored** and
**outside the gateway's Docker build context** (`./verde-flow-api`). It is
therefore *not* baked into the image. `docker-compose.yml` mounts it read-only
into the gateway so `generateAppJwt()` can read it:

```yaml
  gateway:
    volumes:
      - ./.secrets:/app/.secrets:ro
```

Without this mount, every callback fails with `ENOENT` on the key file.
(This is a narrow subdirectory mount — it does not shadow the built `dist/`.)

---

## Applying config changes

`.env` and compose `volumes` changes require **recreating** the container, not
just restarting it:

```bash
./scripts/vf.sh start gateway      # docker compose up -d — reloads env_file + mounts
# (plain `restart` keeps the old container config and will NOT pick up .env changes)
```

Verify inside the container:

```bash
docker exec vf-gateway sh -c 'env | grep -E "PUBLIC_URL|FRONTEND_URL|GITHUB_APP_SLUG"'
docker exec vf-gateway ls -l /app/.secrets   # private key should be present
```

---

## End-to-end test

```bash
./scripts/ngrok.sh start           # background tunnel; ./scripts/ngrok.sh status to check
```

UI → Settings → **Connect GitHub** → approve on GitHub → you land back on
`http://localhost:3000/settings?github=connected`. Because the domain is
static, this keeps working across ngrok and container restarts.
