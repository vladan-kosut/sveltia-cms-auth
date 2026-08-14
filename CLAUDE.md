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

**`.nvmrc` is upstream's — do not touch it.** It currently reads `v26`, which looks wrong next to the Node 24 the rest of the fleet pins, and a cross-repo sweep will flag it (one did on 2026-08-14). It is not ours: the file is authored by the upstream maintainer and rewritten by the daily sync, and `deploy.yml` — the only workflow here — never reads it (no `setup-node`, no `node-version`). Editing it buys nothing and costs a conflict with the cron. The Node-24 migration rule in `DevProjekty/CLAUDE.md` applies to the Astro forks, not here.

### What the force-push means for a local clone

Every time upstream publishes, the cron rewrites the SHAs of our fork commits. A local clone that has not been touched since then keeps the **old** SHAs, so `git status` reports the branch as diverged — typically "N ahead, M behind". **This is expected and no work is at risk.**

Do not try to resolve it by pushing, and do not measure it with `git rev-list --left-right --count`; that compares SHAs and will report our own already-published commits as unpushed work. Compare **patches**:

```bash
git fetch origin
git cherry @{u} HEAD     # '-' = content already on remote, '+' = genuinely missing
```

All lines `-` → the clone is merely stale. Bring it in line with the remote:

```bash
git switch -C main origin/main
```

(`git reset --hard` does the same thing but is blocked by the `block-dangerous.sh` hook.) Only a `+` line means real undelivered work — investigate that one before touching the branch.

**Why this is written down:** on 2026-08-10 a close-check measured this repo with `rev-list` and reported "5 commits unpushed since March, oldest work unbacked for five months" as a risk. `git cherry` showed all five as `-`; the patches had been on the remote the whole time under rewritten SHAs.

## Worker Environment Variables (CF Dashboard)

- `GITHUB_CLIENT_ID` / `GITHUB_CLIENT_SECRET` — GitHub OAuth app credentials
- `ALLOWED_DOMAINS` — comma-separated whitelist (supports wildcards `*.example.com`)

## Client-side Config

Each Sveltia CMS project references this worker in `public/admin/config.yml`:

```yaml
backend:
  base_url: https://sveltia-cms-auth.vibecore.workers.dev
```
