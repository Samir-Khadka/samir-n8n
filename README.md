# samir-n8n — free self-hosted n8n, no subscription

Your own n8n automation server. n8n is open-source, so self-hosting is free
forever — and unlike the paid cloud plans, **nothing is metered**: unlimited
workflows, unlimited executions, unlimited users, community nodes allowed,
your data in your own Postgres.

## Why this beats the subscription

| | n8n Cloud (paid) | This (self-hosted) |
|---|---|---|
| Price | $20+/mo | $0 |
| Workflow executions | capped per plan | unlimited |
| Active workflows | capped | unlimited |
| Your data | on their servers | your Postgres |
| Community nodes | restricted | fully allowed |

## Option A — cloud, always on (recommended)

Runs 24/7 on Render's free tier + Neon free Postgres.

**1. Create the free database** — [neon.tech](https://neon.tech) → New Project
→ copy **Host, Database, User, Password** (Dashboard → Connect → Parameters).

**2. Deploy the blueprint** — [render.com](https://render.com) → **New → Blueprint**
→ connect this repo → when prompted, fill the 6 empty values:
`DB_POSTGRESDB_HOST / DATABASE / USER / PASSWORD` (from Neon) and
`N8N_BASIC_AUTH_USER / N8N_BASIC_AUTH_PASSWORD` (your n8n login — pick anything).
`N8N_ENCRYPTION_KEY` is auto-generated. Deploy.

**3. Fix webhook URLs** — after the first deploy, copy your service URL
(`https://samir-n8n.onrender.com`), then Render → service → Environment:
set `N8N_HOST` = `samir-n8n.onrender.com` (no `https://`) and
`WEBHOOK_URL` = `https://samir-n8n.onrender.com` → Save (auto-redeploys).

**4. Keep it awake (important)** — free Render services sleep after 15 min idle,
which pauses scheduled workflows. In [UptimeRobot](https://uptimerobot.com)
(free) add an **HTTP monitor** on `https://<your-service>.onrender.com/healthz`
every 5 minutes. The pings keep n8n warm so schedules fire on time.

**5. Open it** — go to your URL, log in with the basic-auth credentials from
step 2, create your owner account, then **Workflows → Import** →
`workflows/starter-hourly-tick.json` from this repo and activate it.

## Option B — on your Windows laptop

No cloud, runs while the laptop is on. Good for building/testing.

1. Install [Docker Desktop](https://www.docker.com/products/docker-desktop/).
2. `copy .env.example .env` and fill the 4 secrets
   (`N8N_ENCRYPTION_KEY`: run `openssl rand -hex 24` in Git Bash).
3. `docker compose up -d`
4. Open **http://localhost:5678** — workflows and data persist in Docker volumes.

## Honest limits of the free setup

- Render free sleeps without the UptimeRobot ping trick (step 4 fixes it).
- Neon free = 100 compute-hours/month — plenty for n8n, which idles near zero.
  Old executions auto-delete after 7 days (`EXECUTIONS_DATA_MAX_AGE`) to stay small.
- If you outgrow this, the clean upgrade is Oracle Cloud's Always-Free VM
  (never sleeps, free forever) — same Docker image, same env vars.

## Files

- `render.yaml` — one-click Render Blueprint (n8n Docker + env config)
- `docker-compose.yml` — laptop setup (n8n + Postgres, persistent volumes)
- `.env.example` — secrets template for the laptop setup
- `workflows/starter-hourly-tick.json` — import to verify executions run
