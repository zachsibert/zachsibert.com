# zachsibert.com

Personal site. Pure HTML and CSS, no build step, served by Cloudflare Workers as static assets straight from the repository root.

The landing page is about [firstmate-tui](https://github.com/zachsibert/firstmate-tui), a terminal board over a fleet of coding agents.

## Layout

| Path | What it is |
| --- | --- |
| `index.html` | The one page: hero, the six panes, the keys, how it fits, the screenshot, about, footer |
| `assets/site.css` | The one stylesheet. System font stacks, one monospace face, light and dark via `prefers-color-scheme` |
| `assets/fm-board.png` | The board screenshot, copied from `docs/fm-board.png` in the firstmate-tui repository |
| `assets/favicon.svg` | A pixel anchor, 16 by 16 |
| `404.html` | The not-found page, served with a 404 status for any missing path |
| `robots.txt` | Allows everything |
| `wrangler.jsonc` | Cloudflare Worker config: upload the repository root as static assets, no Worker script |
| `.assetsignore` | The files in this repository that are not part of the site, so wrangler skips them |
| `docs/screenshots/` | Review screenshots for pull requests, not part of the site |
| `AGENTS.md`, `CLAUDE.md` | Notes for coding agents, not part of the site |
| `.gitignore` | Keeps wrangler's local cache directory out of git |

There is no JavaScript except a few inline lines that turn on a copy button for the install command when a clipboard is available. The page renders fully without it.

## Deploy

The repository deploys as a Cloudflare Worker that serves static assets. There is no Worker script and no build step.

- `wrangler.jsonc` tells wrangler to upload the repository root as the site (`assets.directory` is `./`), to serve `index.html` for `/` and drop `.html` from addresses (`html_handling`), and to answer a missing path with `404.html` and a 404 status (`not_found_handling`).
- `.assetsignore` sits in the assets directory, which is the repository root, and lists what wrangler must not upload: this README, the agent notes, the screenshots, the config files. Add any new non-site file there.
- Workers Builds is connected to this repository in the Cloudflare dashboard as the Worker `zachsibert-com`. Its settings: build command empty, deploy command `npx wrangler deploy` (the default), root directory `/`. No `package.json` is needed; `npx` fetches wrangler.

Every push to `main` deploys production. A push to any other branch runs a preview build, and its result shows up as the `Workers Builds: zachsibert-com` check on the pull request.

### Switching to Cloudflare Pages instead

Pages serves the same files with no change to the site itself:

1. Delete `wrangler.jsonc` and `.assetsignore`. Pages does not read the `assets` config, and a leftover wrangler file can confuse its build.
2. In the dashboard, create a Pages project connected to this repository: framework preset none, build command empty, build output directory `/`.
3. Disconnect or delete the `zachsibert-com` Worker so the two do not both build on every push.

Pages serves `404.html` from the output root on its own. This README and the `docs/` folder would then be public at their paths, which is harmless.

## Preview locally

Any static file server works for the page itself. With Python:

```sh
python3 -m http.server 8000
```

Then open http://localhost:8000/. To see the real routing, including the 404 page and dropped `.html` extensions, run wrangler's local server instead:

```sh
npx wrangler dev
```

## Rules

- No frameworks, no CDN scripts, no analytics, no external fonts.
- ASCII only in source.
- Keep the first load under 100 KB excluding the screenshot.
