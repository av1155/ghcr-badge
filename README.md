# ghcr-badge

A small Cloudflare Worker that exposes the download count of a public GitHub
Container Registry package so [shields.io](https://shields.io) can render it
as a badge.

Self-hosted for [Houndarr](https://github.com/av1155/houndarr) so the badge
does not depend on a third-party service remaining online. Forked from
[eliasbenb/ghcr-badge](https://github.com/eliasbenb/ghcr-badge) (MIT).

Live deployment: `https://houndarr-ghcr-badge.<your-cf-subdomain>.workers.dev`

## Endpoints

| Method | Path | Description |
| ------ | ---- | ----------- |
| `GET` | `/api/:owner/:repo/:pkg` | JSON download stats for a repo-scoped package |
| `GET` | `/api/:owner/:pkg` | JSON download stats for a user-scoped package |
| `GET` | `/shield/:owner/:repo/:pkg` | Redirects to a default shields.io Docker Pulls badge |
| `GET` | `/shield/:owner/:pkg` | Same, for user-scoped packages |

Responses are cached for 3 hours. Append `?no-cache` to bypass.

## Example

```bash
curl https://houndarr-ghcr-badge.<subdomain>.workers.dev/api/av1155/houndarr
# {"downloadCount":"48.3K","downloadCountRaw":48321,"success":true,...}
```

For full control over badge style, point shields.io `dynamic/json` at the API:

```markdown
[![GHCR pulls](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fhoundarr-ghcr-badge.<subdomain>.workers.dev%2Fapi%2Fav1155%2Fhoundarr&query=downloadCount&label=ghcr%20pulls&color=2496ed&logo=docker&logoColor=white&style=flat)](https://github.com/av1155/houndarr/pkgs/container/houndarr)
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
