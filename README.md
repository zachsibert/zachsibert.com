# zachsibert.com

Personal site. Pure HTML and CSS, no build step, served by Cloudflare Pages from the repository root.

The landing page is about [firstmate-tui](https://github.com/zachsibert/firstmate-tui), a terminal board over a fleet of coding agents.

## Layout

| Path | What it is |
| --- | --- |
| `index.html` | The one page: hero, the six panes, the keys, how it fits, the screenshot, about, footer |
| `assets/site.css` | The one stylesheet. System font stacks, one monospace face, light and dark via `prefers-color-scheme` |
| `assets/fm-board.png` | The board screenshot, copied from `docs/fm-board.png` in the firstmate-tui repository |
| `assets/favicon.svg` | A pixel anchor, 16 by 16 |
| `robots.txt` | Allows everything |

There is no JavaScript except a few inline lines that turn on a copy button for the install command when a clipboard is available. The page renders fully without it.

## Deploy

Cloudflare Pages serves the repository root as-is.

| Setting | Value |
| --- | --- |
| Framework preset | None |
| Build command | (empty) |
| Build output directory | `/` |
| Root directory | `/` |

Every push to `main` deploys. Pull request branches get a preview URL.

## Preview locally

Any static file server works. With Python:

```sh
python3 -m http.server 8000
```

Then open http://localhost:8000/.

## Rules

- No frameworks, no CDN scripts, no analytics, no external fonts.
- ASCII only in source.
- Keep the first load under 100 KB excluding the screenshot.
