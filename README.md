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
| `wrangler.jsonc` | Cloudflare Worker config: upload the repository root as static assets, no Worker script, the two Custom Domains |
| `.assetsignore` | The files in this repository that are not part of the site, so wrangler skips them |
| `.github/workflows/deploy.yml` | The GitHub Actions workflow that deploys the site on every push to `main` |
| `docs/screenshots/` | Review screenshots for pull requests, not part of the site |
| `AGENTS.md`, `CLAUDE.md` | Notes for coding agents, not part of the site |
| `.gitignore` | Keeps wrangler's local cache directory out of git |

There is no JavaScript except a few inline lines that turn on a copy button for the install command when a clipboard is available. The page renders fully without it.

## Deploy

The repository deploys as a Cloudflare Worker named `zachsibert-com` that serves static assets. There is no Worker script and no build step. The GitHub Actions workflow in `.github/workflows/deploy.yml` owns every deploy; nothing in the Cloudflare dashboard builds or deploys the site.

- `wrangler.jsonc` tells wrangler to upload the repository root as the site (`assets.directory` is `./`), to serve `index.html` for `/` and drop `.html` from addresses (`html_handling`), and to answer a missing path with `404.html` and a 404 status (`not_found_handling`). Its `routes` block attaches the Worker to `zachsibert.com` and `www.zachsibert.com` as Custom Domains: on deploy, Cloudflare creates the DNS records and the certificates for both hostnames on the zone and points them straight at the Worker, so the DNS tab needs no hand edits. `workers_dev` keeps the `zachsibert-com.<account>.workers.dev` address as a fallback. The empty `previews` block is required by `wrangler preview` and would hold any settings a Preview needs that differ from production.
- `.assetsignore` sits in the assets directory, which is the repository root, and lists what wrangler must not upload: this README, the agent notes, the screenshots, the config files, the workflow. Add any new non-site file there.
- The workflow checks out the repository, sets up Node 22, and runs wrangler 4. The production job uses [cloudflare/wrangler-action](https://github.com/cloudflare/wrangler-action) at its `v4` tag; the Preview jobs run `npx wrangler` directly, the way Cloudflare's Previews guide does, because the action's `v4` tag does not expose the Preview address. No `package.json` is needed.

What runs when:

- A push to `main` runs `wrangler deploy`. That uploads the site, attaches the Custom Domains, and is the only thing that changes what https://zachsibert.com/ serves.
- Opening or pushing to a pull request runs `wrangler preview --name pr-<number>`. A [Preview](https://developers.cloudflare.com/workers/previews/) is an isolated copy of the Worker with the branch's files, served at `pr-<number>-zachsibert-com.<account>.workers.dev`. The address stays the same for every push, and the workflow keeps one comment on the pull request up to date with it. Previews do not touch the Custom Domains or production. Preview addresses are public.
- Closing a pull request, merged or not, runs `wrangler preview delete` for its Preview. Cloudflare also deletes the least recently deployed Preview on its own once a Worker has 100 of them.
- Only pull requests get Previews. A branch with no open pull request is never deployed anywhere. Pull requests opened from a fork are skipped because GitHub withholds the secret from them.
- One run per branch or pull request at a time. Two pushes in a row queue instead of racing each other.

Custom Domains need each hostname to be free: Cloudflare refuses to attach a hostname that already has a DNS record. If `zachsibert.com` or `www.zachsibert.com` ever gets a record of its own, delete it in the DNS tab before the next deploy.

### One-time setup

The workflow reads one repository secret and one repository variable. Until both exist, every job fails at authentication. That is expected.

1. In the Cloudflare dashboard, open Manage Account, then Account API Tokens, and create a token with three permission policies. First, `Workers` at the `All Workers` scope with the `Admin` role. `Editor` is enough to deploy an existing Worker, but a token with less than `Admin` is refused when it creates a Preview (HTTP 403, "No access to the specified resource"), and a token scoped to one Worker cannot manage Custom Domains. Second, `Zone / Workers Routes / Edit` scoped to the `zachsibert.com` zone, which attaching the Custom Domains needs. Third, `Zone / DNS / Edit` on the same zone; Cloudflare creates the Custom Domain records itself, so this one is only a safety margin. Copy the token; it is shown once.
2. Copy the account id. It is on the right side of the Workers & Pages overview, and in the address bar of any dashboard page as the long hex string after `dash.cloudflare.com/`.
3. In this repository on GitHub, open Settings, then Secrets and variables, then Actions. On the Secrets tab add `CLOUDFLARE_API_TOKEN` with the token. On the Variables tab add `CLOUDFLARE_ACCOUNT_ID` with the account id. The account id is a variable, not a secret, because it is not sensitive and this way it shows in the run logs. The names must match the workflow exactly.
4. Re-run the last failed `Deploy` run from the Actions tab, or push to `main`. When it goes green, https://zachsibert.com/ resolves within a minute or two, once the new DNS records and certificate are live.
5. Disconnect the Workers Builds integration so nothing is deployed twice. In the Cloudflare dashboard, open Workers & Pages, select `zachsibert-com`, then Settings, then Build, and disconnect the GitHub repository. Do this after the first green Actions deploy, so there is never a moment with no deploy path. Until then, every pull request also gets a second Preview from Workers Builds, named after the branch instead of the pull request number.

### Switching to Cloudflare Pages instead

Pages serves the same files with no change to the site itself:

1. Delete `wrangler.jsonc`, `.assetsignore` and `.github/workflows/deploy.yml`. Pages does not read the `assets` config, and a leftover wrangler file can confuse its build.
2. In the dashboard, create a Pages project connected to this repository: framework preset none, build command empty, build output directory `/`.
3. Delete the `zachsibert-com` Worker, which releases the two Custom Domains, then add `zachsibert.com` as a custom domain on the Pages project.

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

To check that `wrangler.jsonc` still parses and the asset list is what you expect, run the same dry run the pull request check runs. It needs no Cloudflare account:

```sh
npx wrangler deploy --dry-run
```

## Rules

- No frameworks, no CDN scripts, no analytics, no external fonts.
- ASCII only in source.
- Keep the first load under 100 KB excluding the screenshot.
