# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This repository serves two purposes:
1. **Static landing page** (`index.html`, `privacy_policy.html`) — deployed via GitHub Pages at `n8n.ml1.app`
2. **n8n self-hosted stack** (`docker-compose.yml`) — Docker Compose configuration for running the actual n8n instance on a VPS

The GitHub Pages site is just the public-facing landing page. The n8n application itself runs on a separate VPS using the Docker Compose stack in this repo.

## Repository Structure

```
index.html              # Landing page (GitHub Pages)
privacy_policy.html     # Privacy policy (GitHub Pages)
CNAME                   # GitHub Pages custom domain (n8n.ml1.app)
docker-compose.yml      # n8n stack: Traefik + PostgreSQL + n8n + runner
.env.example            # Environment variable template (commit this)
.env                    # Actual secrets (gitignored — never commit)
init-data.sh            # PostgreSQL init: creates non-root app user
.gitignore
.github/workflows/
  static.yml            # Auto-deploy static site to GitHub Pages on push to master
```

## Docker Stack Architecture

Four services in a single `n8n-net` bridge network:

| Service | Image | Role |
|---|---|---|
| `traefik` | `traefik:v3.3` | Reverse proxy — terminates TLS, routes `n8n.ml1.app` → n8n:5678 |
| `postgres` | `postgres:16` | Database — persisted in `db_storage` volume |
| `n8n` | `docker.n8n.io/n8nio/n8n` | Main app — persisted in `n8n_storage` volume |
| `n8n-runner` | `n8nio/runners` | Isolated task executor — communicates with n8n via port 5679 |

Port 5678 is **not** exposed to the host. All external traffic enters on 443 via Traefik. Traefik auto-provisions Let's Encrypt certificates (HTTP challenge on port 80, stored in `traefik_certs` volume).

The `n8n_storage` volume at `/home/node/.n8n` holds the encryption key. **If this volume is lost, all saved credentials become unrecoverable.** Back up this volume before any destructive operation.

## Deployment (VPS)

**First-time setup:**
```bash
cp .env.example .env
# Fill in all blank values in .env — see generation commands below
docker compose up -d
```

**Generate secret values:**
```bash
openssl rand -hex 32   # for N8N_ENCRYPTION_KEY, N8N_USER_MANAGEMENT_JWT_SECRET, RUNNERS_AUTH_TOKEN
openssl rand -base64 24  # for POSTGRES_PASSWORD, POSTGRES_NON_ROOT_PASSWORD
```

**Update n8n:**
```bash
docker compose pull
docker compose up -d
docker image prune -f
```

**Stop / restart:**
```bash
docker compose stop
docker compose up -d
```

**View logs:**
```bash
docker compose logs -f n8n
docker compose logs -f traefik
```

## Key Environment Variables

| Variable | Purpose |
|---|---|
| `DOMAIN_NAME` | Public hostname (`n8n.ml1.app`) |
| `SSL_EMAIL` | Let's Encrypt notification address |
| `N8N_VERSION` | Image tag — use `stable` or a pinned version like `1.70.0` |
| `N8N_ENCRYPTION_KEY` | Encrypts all stored credentials — treat like a master password |
| `N8N_USER_MANAGEMENT_JWT_SECRET` | Signs user session tokens |
| `RUNNERS_AUTH_TOKEN` | Shared secret between n8n and n8n-runner |
| `POSTGRES_USER` / `POSTGRES_PASSWORD` | PostgreSQL superuser (init only) |
| `POSTGRES_NON_ROOT_USER` / `POSTGRES_NON_ROOT_PASSWORD` | App-level DB user (runtime) |

`WEBHOOK_URL` is derived from `DOMAIN_NAME` in `docker-compose.yml` (`https://${DOMAIN_NAME}/`). This is required for webhooks to register the correct public URL — do not remove it.

## Static Site Deployment (GitHub Pages)

No build step. Push to `master` → GitHub Actions deploys the entire repo as a Pages artifact.

- **Live URL**: https://n8n.ml1.app (also serves as the n8n app URL via the VPS)
- The landing page CTA buttons (`Go to Your Server`, `Launch Dashboard`) link to `https://n8n.ml1.app`

## Styling Conventions (HTML pages)

- **CSS**: Tailwind CSS via CDN — no build step. Prefer utility classes over custom CSS.
- **Font**: Inter from `https://rsms.me/inter/inter.css`
- **Color scheme**: `bg-gray-900` page background, `bg-gray-800` card/section backgrounds, `indigo-600` primary CTAs
- **Layout**: `max-w-3xl` for centered text, `max-w-7xl` for grid sections
- Custom CSS goes in the `<style>` block in `<head>` only when Tailwind can't handle it
