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
.github/workflows/      # Auto-deploy on push to main
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

To serve the pages from `motd.nide.gg`, add a custom domain to the `game-motd` Worker in the Cloudflare dashboard (**Workers & Pages → game-motd → Settings → Domains & Routes**).
