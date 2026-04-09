# CLAUDE.md

Sveltia CMS OAuth proxy — Cloudflare Worker providing GitHub/GitLab OAuth for Sveltia CMS. Fork of [sveltia/sveltia-cms-auth](https://github.com/sveltia/sveltia-cms-auth).

## Architecture

- Single-file Worker: `src/index.js` (vanilla JS, no build step)
- Routes: `/auth` (start OAuth flow) → `/callback` (exchange code for token)
- CSRF protection via HttpOnly cookie
- Config: `wrangler.toml` (name + compatibility_date only, no secrets)

## Commands

```bash
pnpm install          # Install deps (pnpm is project's package manager)
pnpm run start        # Local dev (wrangler dev)
pnpm run deploy       # Deploy to CF Workers (wrangler deploy)
pnpm run check        # Lint + format + audit + spellcheck
```

## Deployment

- **Worker URL:** `https://sveltia-cms-auth.vibecore.workers.dev` (shared for all CF projects)
- **CI/CD:** `.github/workflows/deploy.yml` — push to main triggers deploy via `wrangler-action`
- **Package manager in CI:** npm (not pnpm) — wrangler-action default
- **GitHub Secrets:** `CF_API_TOKEN`, `CF_ACCOUNT_ID`

## Upstream Sync

- Daily cron (5:30 UTC): fetches `sveltia/sveltia-cms-auth` main, rebases fork commits, force-pushes
- Push after rebase triggers deploy job automatically
- Fork-specific commits (CLAUDE.md, deploy.yml, package-lock.json) stay on top via rebase

## Worker Environment Variables (CF Dashboard)

- `GITHUB_CLIENT_ID` / `GITHUB_CLIENT_SECRET` — GitHub OAuth app credentials
- `ALLOWED_DOMAINS` — comma-separated whitelist (supports wildcards `*.example.com`)

## Client-side Config

Each Sveltia CMS project references this worker in `public/admin/config.yml`:

```yaml
backend:
  base_url: https://sveltia-cms-auth.vibecore.workers.dev
```
