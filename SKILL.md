---
name: deploy-rakazo
description: "Use when deploying Rakazo (persistent AI teammates platform) on Railway with the one-click template, or when taking over, reconnecting to, or retrieving the results of an existing Rakazo deployment from another WSL/Linux host."
---

# Deploy Rakazo on Railway

## Overview

Rakazo is an open-source platform for persistent AI teammates with memory, routines, and their own computers (browser, terminal, files, graphical desktop). This skill documents the end-to-end deployment on Railway using the official template.

**Core principle:** Deploy the full stack (web, api, worker, postgres) via Railway's one-click template, configure the optional model/sandbox keys, and verify the first user registration.

## Prerequisites

- A Railway account (sign up at https://railway.com)
- (Optional) OpenRouter API Key (https://openrouter.ai) for out-of-the-box model access
- (Optional) Sandbox provider API Key (E2B, Daytona, or Box) for bot computer capabilities

## Deployment Steps

### 1. Open the Railway Template

Navigate to the Rakazo template page:

```
https://railway.com/deploy/rakazo
```

Or directly to the deploy wizard:

```
https://railway.com/new/template/rakazo
```

### 2. Sign In to Railway

Railway supports three login methods:
- **Continue with GitHub** (recommended — also grants repo access for template sources)
- **Continue with Google**
- **Log in using email**

Click "Login" on the deploy page, then choose your preferred method.

### 3. Review the Template

The template deploys four services:

| Service | Image | Role |
|---------|-------|------|
| `web` | `ghcr.io/elie222/rakazo/app:v0.3.3` | Public web app (Vite preview, port 5173) |
| `api` | `monotykamary/railway-template-rakazo` | Hono API server (port from PORT env) |
| `worker` | `monotykamary/railway-template-rakazo` | Graphile Worker (routines, wakeups) |
| `postgres` | `railwayapp-templates/postgres-ssl:18` | PostgreSQL 18 datastore |

### 4. Configure Environment Variables

#### Required (auto-generated — do not edit)
- `BETTER_AUTH_SECRET` — signing secret for auth sessions
- `ENCRYPTION_KEY` — AES key for stored credentials (never regenerate after first deploy)
- `SCREEN_PROXY_SECRET` — shared signing secret for browser-screen capabilities
- `POSTGRES_PASSWORD` — auto-generated Postgres password
- `DATABASE_URL` — auto-wired Postgres connection string

#### Optional but Recommended
- `OPENROUTER_API_KEY` — model key so bots work out of the box (users can also connect models per-user in the UI)
- `SANDBOX_PROVIDER` — `e2b`, `daytona`, `box`, or `none` (default: `e2b`)
- `E2B_API_KEY` / `DAYTONA_API_KEY` / `BOX_API_KEY` — matching key for the chosen provider
- `SIGNUP_ALLOWLIST` — comma-separated emails/domains allowed to sign up (configure before first API start to keep deployment private)
- `SMTP_URL` — optional SMTP URL for password recovery
- `EMAIL_FROM` — optional From header for outgoing email
- `COMPOSIO_API_KEY` — optional Composio key for managed plugin catalog

#### Do Not Edit (auto-wired)
- `API_URL`, `WEB_ORIGIN`, `BETTER_AUTH_URL`, `RAKAZO_HOST` — wired to the web service domain
- `API_PROXY_TARGET` — private API endpoint
- `DATABASE_URL` — Postgres connection string
- `DATA_DIR` — writable directory for bot workspace checkpoints

### 5. Deploy

Click the **"Deploy"** button. Railway will:
1. Provision all four services
2. Run database migrations (`prisma migrate deploy`)
3. Health-check the API at `GET /health`
4. Assign a public domain to the web service

### 6. Post-Deploy Verification

1. **Register the first user immediately.** The first registered user becomes the deployment owner.
2. Access the web URL provided by Railway (e.g., `https://xxx.railway.app`).
3. Sign up with your email.
4. Verify the API health endpoint: `https://xxx.railway.app/health` should return OK.
5. (Optional) Test bot creation and a simple conversation.

### 7. Important Notes

- **ENCRYPTION_KEY**: Never regenerate after the first start — stored credentials would become undecryptable.
- **First user is owner**: Register immediately after deploy to claim ownership.
- **Private deployment**: Set `SIGNUP_ALLOWLIST` before the first API start.
- **Sandbox provider**: Without a key, the product boots and bots chat, but computer tools are unavailable.
- **Railway trial**: Free tier provides $5.00 or 30 days. **Note: The free plan may not have enough resources to provision all 4 services (web, api, worker, postgres). If you see "Free plan resource provision limit exceeded", upgrade to a paid plan (Hobby $5/mo or Pro) before deploying.**
- **Resource requirements**: The template deploys 4 services. The free tier's resource limits may be insufficient. Consider the Hobby plan ($5/month) or higher.

### 8. Alternative: External Postgres (3 services, no Postgres on Railway)

If the free plan's 3-service limit is insufficient for all 4 services, use an external Postgres database:

**Free external Postgres providers:**
- [Neon](https://neon.tech) — free tier, 0.5 GB storage
- [Supabase](https://supabase.com) — free tier, 500 MB

**Steps:**
1. Create a database on the external provider
2. Get the connection string: `postgresql://user:pass@host:port/dbname?sslmode=require`
3. Deploy only `web`, `api`, `worker` on Railway (3 services)
4. Set `DATABASE_URL` in both `api` and `worker` to the external connection string
5. Ensure `ENCRYPTION_KEY` is shared between `api` and `worker`

**Resource requirements (minimum):**
| Service | Min RAM | Recommended |
|---------|---------|-------------|
| web (Vite) | 128 MB | 256 MB |
| api (Hono) | 256 MB | 512 MB |
| worker (Graphile) | 256 MB | 512 MB |

### 9. Alternative: Docker Compose Self-Hosting

For full control over resources, deploy on your own server:

```bash
git clone https://github.com/elie222/rakazo.git
cd rakazo/infra/compose
cp .env.example .env
# Edit .env: set POSTGRES_PASSWORD, OPENROUTER_API_KEY, etc.
docker compose -f docker-compose.prod.yml up -d --build
```

Requirements: Linux server, Docker + Docker Compose, 4 GB RAM / 2 CPU minimum (8 GB recommended).

### 10. Alternative: GitHub Codespaces as External Postgres

Use a GitHub Codespace to run Postgres as an external database for Railway:

**Setup:**
1. Go to https://github.com/phycomp/codespaces
2. Click "Code" → "Codespaces" → "Create codespace on main"
3. Choose 4+ cores, 8GB+ RAM machine
4. Wait for the setup script to complete (installs Rakazo + Postgres)
5. In the Codespace terminal, get the Postgres connection string:
   ```bash
   # Postgres runs on port 5432 inside the Codespace
   # Use the Codespace's forwarded port URL
   echo "postgresql://postgres:password@localhost:5432/rakazo?sslmode=disable"
   ```
6. Make port 5432 public in the Codespace "Ports" tab
7. Use the forwarded URL as `DATABASE_URL` in Railway

**Limitations:**
- Codespaces sleep after 30 min of inactivity
- Codespaces are deleted after 30 days of inactivity
- Free tier: 60 hours/month for 2-core, 120 hours for 4-core
- Not suitable for production; use for development/testing only

**Recommended:** Use Neon (https://neon.tech) or Supabase (https://supabase.com) for a permanent free Postgres database instead.

| Issue | Solution |
|-------|----------|
| Free plan resource limit exceeded | Railway free tier cannot provision 4 services; upgrade to Hobby ($5/mo) or Pro plan |
| API container exits immediately | Check `POSTGRES_PASSWORD` is set and Postgres healthcheck passes |
| Bots can't use computer tools | Set `SANDBOX_PROVIDER` and matching API key |
| Can't sign up | Check `SIGNUPS_ENABLED` is true and `SIGNUP_ALLOWLIST` if set |
| Stored credentials undecryptable | ENCRYPTION_KEY was regenerated — restore from backup |

## Retrieve an Existing Deployment (other host)

When the user says they deployed on another host and wants to **取回成果 / 接手**,
do NOT redeploy. The deployment lives in Railway, not in the local filesystem.

### 1. Sync the skill repo (own repo, not oskill)
```bash
git -C ~/.agents/skills/deploy-rakazo pull --ff-only
git -C ~/.agents/skills/deploy-rakazo status --porcelain          # must be empty
git -C ~/.agents/skills/deploy-rakazo log origin/main..HEAD       # must be empty
```
Remote is `https://github.com/phycomp/deploy-rakazo.git` (`main`).
If the directory is missing, install it with `repo-pull`, then run `audit-skill`
(first install/update requires an APPROVE/CAUTION verdict).

### 2. Reconnect to the existing project
- If `railway` is not installed (`command -v railway`), install `@railway/cli` first.
  On this machine, run `audit-skill` before installing any package (local hard rule).
- `railway login` — user completes browser auth; must be the **same Railway account**.
- `railway link <projectId>` — **never `railway init`** (that creates a new deployment).
- `railway status` — confirm services `web`, `api`, `worker`, `postgres`.
- Project ID: ask the user or read a saved handoff note. Do not guess from a domain.
- Web URL: `railway domain` or `RAILWAY_PUBLIC_DOMAIN`; do not assume `*.up.railway.app`.
- Health: `curl -sS -o /dev/null -w '%{http_code}\n' https://<web-domain>/health`
  (`/health` is the documented path; only try `/api/health` if `/health` 404s.)

### 3. Retrieve settings safely
- Secrets source of truth = **Railway Variables** (`railway variables` / dashboard).
- **Never** scrape `~/.bash_history`, old `.env`, or git remote URLs for tokens.
- Write secrets only to `~/.config/rakazo/.env` with mode `600`; never into git,
  `environment-profile.md`, chat, logs, argv, or remote URLs.
- `ENCRYPTION_KEY` must be reused as-is. If unrecoverable, stop — regenerating makes
  stored credentials undecryptable.
- Push with `gh auth setup-git` or temporary `GIT_ASKPASS`; never embed tokens in URLs.

### 4. Save a handoff note (no secrets)
In the skill repo or `~/.config/rakazo/handoff.md` (600): project ID, environment,
web domain, health path, service list, and variable **names only** — never values.

### Red flags / rationalizations
- 「先 `railway init` 比較快」→ creates a second deployment.
- 「token 可能在 history 裡」→ history scraping is out of scope; use Railway Variables.
- 「skill 應該在 oskill」→ this skill has its own repo `phycomp/deploy-rakazo`.
- 「重新產生 ENCRYPTION_KEY」→ irreversible data loss.

## Quick Reference

```
Template URL:  https://railway.com/deploy/rakazo
Deploy URL:    https://railway.com/new/template/rakazo
Station:       https://station.railway.com/all-templates/49594aec-af0d-44fd-b143-ce05be3dd1e9
Source:        https://github.com/elie222/rakazo
Image:         ghcr.io/elie222/rakazo/app:v0.3.3
```
