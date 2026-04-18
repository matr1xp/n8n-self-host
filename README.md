# n8n Self-Hosted

This repository serves two purposes:

1. **Landing page** — static HTML site deployed to GitHub Pages at [n8n.ml1.app](https://n8n.ml1.app)
2. **n8n stack** — Docker Compose configuration for running the self-hosted n8n instance on a VPS

## Stack

- **[n8n](https://n8n.io)** — workflow automation platform
- **PostgreSQL 16** — persistent database
- **Traefik v3** — reverse proxy with automatic HTTPS via Let's Encrypt
- **n8n-runner** — isolated task executor for code nodes

## Deployment

### Prerequisites

- VPS running Ubuntu 24.04 LTS (minimum 2 vCPU / 2 GB RAM)
- Docker with the Compose plugin installed
- DNS A record for `n8n.ml1.app` pointing to the VPS IP

### First-time setup

```bash
git clone https://github.com/matr1xp/n8n-self-host.git
cd n8n-self-host

cp .env.example .env
```

Edit `.env` and fill in all blank values. Generate secrets with:

```bash
openssl rand -hex 32     # N8N_ENCRYPTION_KEY, N8N_USER_MANAGEMENT_JWT_SECRET, RUNNERS_AUTH_TOKEN
openssl rand -base64 24  # POSTGRES_PASSWORD, POSTGRES_NON_ROOT_PASSWORD
```

Then start the stack:

```bash
docker compose up -d
```

n8n will be available at `https://n8n.ml1.app`. The first visit creates the owner account.

### Updating n8n

```bash
docker compose pull
docker compose up -d
docker image prune -f
```

### Useful commands

```bash
docker compose logs -f n8n       # stream n8n logs
docker compose logs -f traefik   # stream Traefik logs
docker compose stop              # stop all services
docker compose down              # stop and remove containers (volumes are preserved)
```

## Important Notes

- The `n8n_storage` Docker volume (`/home/node/.n8n` inside the container) holds the encryption key. If this volume is lost, all saved credentials become unrecoverable. Back it up regularly.
- Never commit `.env` — it is gitignored.
- Pin `N8N_VERSION` in `.env` to a specific release (e.g. `1.70.0`) for controlled upgrades in production.
