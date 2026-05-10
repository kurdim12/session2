# Palm Guard — Network Deployment Package

This directory contains everything needed to put
[`aboodhaymouni/palm-guard`](https://github.com/aboodhaymouni/palm-guard)
on a public URL or on your LAN.

The repo is a single full-stack app:

| Component | Stack | Port |
|-----------|-------|------|
| Backend   | Node 22 + Express + Socket.IO + SQLite | 4000 |
| Frontend  | React + Vite (built to `frontend/dist`) | served by backend in prod |
| Storage   | SQLite file at `backend/data/palmguard.db` (override via `PG_DB_PATH`) | persistent volume |

The backend already serves `frontend/dist` automatically when it exists, so a
single container on one port covers both.

---

## Files in this package

| File | Purpose |
|------|---------|
| `Dockerfile` | Multi-stage build: builds frontend, installs backend prod deps, runs `server.js` on port 4000. |
| `.dockerignore` | Keeps the build context small (skips PDFs, images, firmware, node_modules, etc.). |
| `docker-compose.yml` | One-command local/LAN run with a named volume for the SQLite DB. |
| `render.yaml` | Render.com Blueprint — Docker web service + 1 GB persistent disk. |

All four are designed to live at the **root of the palm-guard repo**.

---

## Step 1 — Copy these files into palm-guard

Because the deploy configs need to live in the repo being deployed, you have
two ways to get them there.

### Option A — Fork & copy (recommended)

```bash
# 1. Fork aboodhaymouni/palm-guard on GitHub (your own account).
# 2. Clone your fork:
git clone https://github.com/<YOUR_USERNAME>/palm-guard.git
cd palm-guard

# 3. Copy the four files from this package:
cp /path/to/session2/palm-guard-deploy/Dockerfile .
cp /path/to/session2/palm-guard-deploy/.dockerignore .
cp /path/to/session2/palm-guard-deploy/docker-compose.yml .
cp /path/to/session2/palm-guard-deploy/render.yaml .

# 4. Commit & push:
git add Dockerfile .dockerignore docker-compose.yml render.yaml
git commit -m "Add Docker + Render deployment config"
git push origin main
```

### Option B — PR to upstream

Open a PR against `aboodhaymouni/palm-guard` adding the same four files at the
repo root.

---

## Step 2 — Choose a deployment target

### A. Render.com (free public URL, recommended)

1. Sign in to <https://dashboard.render.com>.
2. **New → Blueprint** → connect your fork → Render auto-detects `render.yaml`.
3. Click **Apply**.
4. (Optional) Set the `PG_OPENROUTER_KEY` env var on the service to enable
   the chat feature. Leave blank to disable.
5. After ~3–5 min you get a public URL like
   `https://palm-guard.onrender.com`. The frontend, REST API, and Socket.IO
   all run on this one URL.

Notes:
- Free plan instances sleep after 15 min idle and cold-start in ~30 s. Switch
  the `plan:` field in `render.yaml` to `starter` ($7/mo) for always-on.
- The 1 GB disk at `/data` keeps the SQLite DB across restarts and deploys.

### B. Fly.io

```bash
fly launch --no-deploy --copy-config --dockerfile ./Dockerfile
fly volumes create palmguard_data --size 1 --region <region>
# In fly.toml: mount palmguard_data -> /data, set internal_port=4000
fly deploy
```

### C. Any VPS with Docker (LAN or public)

```bash
docker compose up -d --build
# App is now on http://<server-ip>:4000
```

For a public URL on a VPS, put Caddy or nginx in front of port 4000 and point
a domain at the box. Caddy one-liner:

```
yourdomain.com {
    reverse_proxy localhost:4000
}
```

Caddy auto-handles HTTPS via Let's Encrypt.

### D. LAN only (your network)

`docker compose up --build` on any machine, then visit
`http://<that-machine-IP>:4000` from any phone/laptop on the same Wi-Fi.

---

## Step 3 — Verify

Once deployed, hit the health endpoint:

```
GET https://<your-url>/api/v1/health
```

Expected response:

```json
{
  "ok": true,
  "service": "palm-guard-backend",
  "db_path": "/data/palmguard.db",
  "readings_count": 0,
  "server_ts": 1715299200,
  "uptime_s": 12
}
```

Then open the root URL in a browser to load the React dashboard.

---

## Environment variables

| Var | Default | Required | Purpose |
|-----|---------|----------|---------|
| `PORT` | `4000` | no | HTTP port the backend listens on. |
| `HOST` | `0.0.0.0` | no | Bind address. |
| `PG_DB_PATH` | `backend/data/palmguard.db` | no | SQLite file path; point at your persistent volume. |
| `PG_OPENROUTER_KEY` | _(unset)_ | no | Enables `/api/v1/chat`. Without it, chat is disabled but the rest of the app works. |
| `PG_OPENROUTER_MODEL` | `anthropic/claude-3.5-sonnet` | no | Model id for OpenRouter. |
| `PG_OPENROUTER_MAX_TOKENS` | `1024` | no | Max tokens per chat reply. |

---

## ESP32 firmware → cloud backend

Once you have a public URL, update `firmware/palmguard-esp32s3/include/secrets.h`
on each device so it posts readings to your deployment instead of `localhost`:

```cpp
#define PG_BACKEND_HOST   "palm-guard.onrender.com"
#define PG_BACKEND_PORT   443
#define PG_BACKEND_HTTPS  1
```

(Exact macro names depend on the firmware — check that file in the repo.)

---

## What this package does NOT do

- It does **not** create accounts on Render/Fly/etc. for you.
- It does **not** push to `aboodhaymouni/palm-guard`. You need write access
  (own a fork or have permission upstream) to commit these files into the
  repo Render will build from.
- It does **not** configure a custom domain — Render gives you a free
  `*.onrender.com` URL out of the box; bring-your-own-domain is a one-click
  step in the Render dashboard.
