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
├── css_zr/             # CS:S Zombie Revival MOTD (index.html + css/ + imgs/)
└── css_gg/             # CS:S GunGame MOTD (index.html + css/ + imgs/)
wrangler.jsonc          # Worker config (assets-only, no script)
.github/                # Dependabot + auto-merge for GitHub Actions updates
```

To add a new MOTD, create `public/<game>_<mode>/index.html` (plus its CSS/images in the same folder) and link it from `public/index.html`.

## URLs

| MOTD | Path |
|------|------|
| CS:S Zombie Escape | `/css_ze/` |
| CS:S Zombie Revival | `/css_zr/` |
| CS:S GunGame | `/css_gg/` |

The old `/css_*/index.php` URLs redirect to the new paths, so existing server configs keep working.

## Shared logos

`public/assets/logos/` is the single source of the NiDE banners used by the other sites (forum, bans, donate, demos, stats). Each site links a **stable** URL, never a seasonal file:

| Logo | Stable URL | Used by |
|------|------------|---------|
| `forums` | `https://motd.nide.gg/assets/logos/forums.png` | forum, these MOTD pages |
| `sourcebans` | `https://motd.nide.gg/assets/logos/sourcebans.png` | bans.nide.gg |
| `ebans` | `https://motd.nide.gg/assets/logos/ebans.png` | (EBans tab of bans.nide.gg) |
| `kbans` | `https://motd.nide.gg/assets/logos/kbans.png` | (KBans tab of bans.nide.gg) |
| `donate` | `https://motd.nide.gg/assets/logos/donate.png` | donate.nide.gg |
| `demos` | `https://motd.nide.gg/assets/logos/demos.png` | demos.nide.gg |
| `demos-small` | `https://motd.nide.gg/assets/logos/demos-small.png` | |
| `rankings` | `https://motd.nide.gg/assets/logos/rankings.png` | stats.nide.gg |

`public/_redirects` rewrites each stable URL (status 200, the browser keeps the same URL) to `classic/<name>.png` or `winter/<name>.png`. To switch season, change the target folder of the relevant lines and merge: every site follows within the 5-minute cache set in `_headers`, with no redeploy on their side. Every logo has a classic and a winter file; a new logo without a winter file points at `classic/` until one is added.

Keep both variants of a logo the same size, so the sites' layout does not move when the season changes.

## Local preview

```sh
npx wrangler dev
```

## Deployment

The `game-motd` Worker is connected to this repository through [Workers Builds](https://developers.cloudflare.com/workers/ci-cd/builds/) (Cloudflare's Git integration), so there is no deploy workflow or secret on the GitHub side:

- every push to `main` runs `wrangler deploy` to production;
- every other branch gets a [Worker Preview](https://developers.cloudflare.com/workers/previews/) at `https://<branch>-game-motd.<subdomain>.workers.dev`, and Cloudflare posts its URL as a comment on the pull request.

Build settings and logs: **Workers & Pages → game-motd → Settings → Build** in the Cloudflare dashboard.

The Worker is served from **`motd.nide.gg`**, declared as a custom domain in `wrangler.jsonc` and attached on every deploy. There must be no pre-existing DNS record for `motd.nide.gg` in the `nide.gg` zone (delete it first, otherwise the deploy fails).
