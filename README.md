# ghcr-badge

A small Cloudflare Worker that exposes the download count of a public GitHub
Container Registry package so [shields.io](https://shields.io) can render it
as a badge.

Forked from [eliasbenb/ghcr-badge](https://github.com/eliasbenb/ghcr-badge)
(MIT). Ships with an owner allowlist (`ALLOWED_OWNERS` in `src/index.ts`) so
a deployment only serves badges for the owners you choose, preventing others
from piggybacking on your Worker's free-tier quota.

Live deployment: `https://ghcr-badge.<your-cf-subdomain>.workers.dev`

## Endpoints

| Method | Path | Description |
| ------ | ---- | ----------- |
| `GET` | `/api/:owner/:repo/:pkg` | JSON download stats for a repo-scoped package |
| `GET` | `/api/:owner/:pkg` | JSON download stats for a user-scoped package |
| `GET` | `/shield/:owner/:repo/:pkg` | Redirects to a default shields.io Docker Pulls badge |
| `GET` | `/shield/:owner/:pkg` | Same, for user-scoped packages |

Responses are cached for 3 hours. Append `?no-cache` to bypass.

## Example

Replace `<owner>` and `<package>` with an allowlisted owner and the package name:

```bash
curl https://ghcr-badge.<subdomain>.workers.dev/api/<owner>/<package>
# {"downloadCount":"48.3K","downloadCountRaw":48321,"success":true,...}
```

For full control over badge style, point shields.io `dynamic/json` at the API:

```markdown
[![GHCR pulls](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fghcr-badge.<subdomain>.workers.dev%2Fapi%2F<owner>%2F<package>&query=downloadCount&label=ghcr%20pulls&color=2496ed&logo=docker&logoColor=white&style=flat)](https://github.com/<owner>/<package>/pkgs/container/<package>)
```

## Develop and deploy

```bash
pnpm install
pnpm dev          # wrangler dev on 127.0.0.1
pnpm run deploy   # wrangler deploy to Cloudflare (the explicit `run` avoids pnpm's reserved `deploy` builtin)
```

`pnpm deploy` requires `wrangler login` once; subsequent deploys reuse the cached credentials.

## GitHub package URL styles

| Style | URL pattern | Endpoint path |
| ----- | ----------- | ------------- |
| Repo-scoped | `https://github.com/:owner/:repo/pkgs/container/:pkg` | `/api/:owner/:repo/:pkg` |
| User-scoped | `https://github.com/users/:owner/packages/container/package/:pkg` | `/api/:owner/:pkg` |
