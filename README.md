# Game-MOTD

Static MOTD (Message of the Day) pages displayed in-game on NiDE servers, deployed to **Cloudflare Workers** (static assets).

## Structure

```
public/                 # Everything in here is deployed as-is
├── index.html          # Index of available MOTDs
├── 404.html            # Fallback page
├── _redirects          # Legacy URL redirects (e.g. /css_ze/index.php -> /css_ze/)
├── _headers            # Response headers
├── css_ze/             # CS:S Zombie Escape MOTD (index.html + css/ + imgs/)
└── css_zr/             # CS:S Zombie Revival MOTD (index.html + css/ + imgs/)
wrangler.jsonc          # Worker config (assets-only, no script)
.github/workflows/      # Deploy on push to main, previews on PRs
```

To add a new MOTD, create `public/<game>_<mode>/index.html` (plus its CSS/images in the same folder) and link it from `public/index.html`.

## URLs

| MOTD | Path |
|------|------|
| CS:S Zombie Escape | `/css_ze/` |
| CS:S Zombie Revival | `/css_zr/` |

The old `/css_*/index.php` URLs redirect to the new paths, so existing server configs keep working.

## Local preview

```sh
npx wrangler dev
```

## Deployment

Every push to `main` deploys through GitHub Actions (`.github/workflows/deploy.yml`).

Required repository secrets (**Settings → Secrets and variables → Actions**):

| Secret | Description |
|--------|-------------|
| `CLOUDFLARE_API_TOKEN` | API token using the *Edit Cloudflare Workers* template |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare account ID |

The Worker is served from **`motd.nide.gg`**, declared as a custom domain in `wrangler.jsonc` and attached on every deploy. This requires:

- the `nide.gg` zone to live in the same Cloudflare account as `CLOUDFLARE_ACCOUNT_ID`;
- the API token to have access to that zone (the *Edit Cloudflare Workers* template covers it when scoped to `nide.gg` or all zones);
- no pre-existing DNS record for `motd.nide.gg` (delete it first, otherwise the deploy fails).

### Pull request previews

Every pull request opened from this repository gets its own [Worker Preview](https://developers.cloudflare.com/workers/previews/) (`.github/workflows/preview.yml`):

- URL: `https://pr-<number>-game-motd.<subdomain>.workers.dev`, posted as a comment on the PR and updated on each push;
- deleted automatically when the PR is closed or merged;
- skipped for forks and Dependabot PRs (repository secrets are not available to them).
