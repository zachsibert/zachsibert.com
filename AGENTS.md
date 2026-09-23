# Project agent memory

This file is the project's committed home for project-intrinsic agent knowledge: build, test, release, architecture, and sharp-edge notes that should travel with the code.

## What this is

A one-page static site about firstmate-tui, deployed as a Cloudflare Worker that serves the repository root as static assets: `wrangler.jsonc` has no `main`. `.github/workflows/deploy.yml` owns every deploy: `wrangler deploy` on a push to `main`, a per-pull-request Cloudflare Preview otherwise, deleted when the pull request closes. `README.md` lists the files, what runs when, the one-time credential setup and how to switch to Pages.

## Rules that are easy to break

- No build step, no frameworks, no CDN scripts, no analytics, no external fonts. System font stacks only; `--mono` is the one terminal-flavored face.
- The only JavaScript is the inline copy button at the foot of `index.html`. The page must render fully with it disabled: the button starts `hidden` and JavaScript reveals it.
- ASCII only in source. Check: `LC_ALL=C grep -nP '[^\x00-\x7F]' index.html assets/site.css README.md` prints nothing.
- Do not name an employer, a client, or any real company in copy or in the mock board rows. Fictional task ids and first names only.
- A new file that is not part of the site (notes, screenshots, config) goes into `.assetsignore`, or wrangler uploads it.
- `assets/fm-board.png` is copied from `docs/fm-board.png` in the firstmate-tui repository. Refresh it from there rather than editing it.

## Verify before a PR

- HTML and CSS: `curl -s -H "Content-Type: text/html; charset=utf-8" --data-binary @index.html "https://validator.w3.org/nu/?out=json"` (and `text/css` for `assets/site.css`) must return zero messages.
- Lighthouse on a local server (`python3 -m http.server 8765`) should score 100 in every category on desktop and mobile. `chrome-devtools-axi lighthouse` works when its bridge is healthy; otherwise install `lighthouse` in a scratch directory and point `CHROME_PATH` at a Chrome binary.
- Check 360, 768 and 1280 px in both color schemes: `document.documentElement.scrollWidth` must equal `innerWidth`.
- First load stays under 100 KB excluding the screenshot.
- `npx wrangler deploy --dry-run` must pass after any change to `wrangler.jsonc` or `.assetsignore`; it needs no credentials and prints the asset list at `WRANGLER_LOG=debug`. Run `actionlint` on `.github/workflows/deploy.yml` after editing it.
- Every pull request gets a live Preview at the address in the workflow's sticky comment; use it for browser checks. It answers a missing path with a plain `Not found`, so check the 404 page with `npx wrangler dev` or on production.

## Maintaining this file

Keep this file for knowledge useful to almost every future agent session in this project.
Do not repeat what the codebase already shows; point to the authoritative file or command instead.
Prefer rewriting or pruning existing entries over appending new ones.
When updating this file, preserve this bar for all agents and keep entries concise.
