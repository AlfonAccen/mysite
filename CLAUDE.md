# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> Full project standards and architecture are in [AGENTS.md](./AGENTS.md). This file summarizes the most critical points.

## Commands

```bash
npm install                                                    # install deps
npx -y @adobe/aem-cli up --no-open --forward-browser-logs     # dev server at http://localhost:3000
npm run lint                                                   # ESLint + Stylelint
npm run lint:fix                                               # auto-fix lint issues
```

No build step. No transpilation. Code runs directly in the browser.

## Architecture

### Page Loading Pipeline (`scripts/scripts.js`)

Three sequential phases drive every page:
1. **Eager** — decorate main, load first section → triggers LCP
2. **Lazy** — load header, footer, remaining sections
3. **Delayed** — `delayed.js` fires after 3 s (martech, analytics)

`scripts/aem.js` is the core AEM library — **never modify it**.

### Block System (`blocks/`)

Each block is a folder with `{name}.js` + `{name}.css`. The JS exports a single `default async function decorate(block)` that transforms the server-rendered HTML in place using DOM APIs. Blocks are loaded on demand by `aem.js`.

Current blocks: `hero`, `cards`, `columns`, `header`, `footer`, `fragment`, `widget`.

### Auto-blocking (`buildAutoBlocks` in `scripts.js`)

Automatically wraps certain link patterns in blocks without explicit authoring:
- Links containing `/fragments/` → `fragment` block
- Links containing `/widgets/` → `widget` block

### Button decoration (`decorateButtons` in `scripts.js`)

Links inside `<p>` are promoted to buttons based on wrapping markdown formatting:
- `**[text](url)**` → `.button.primary`
- `*[text](url)*` → `.button.secondary`
- `***[text](url)***` → `.button.accent`

### CSS Conventions

- Selectors must be scoped to the block: `.hero .title`, not `.title`
- Mobile-first; breakpoints at `600px`, `900px`, `1200px`
- Global styles split: `styles.css` (eager/LCP), `lazy-styles.css` (post-LCP), `fonts.css`
- Do not use `.{blockname}-container` or `.{blockname}-wrapper` — reserved by AEM for sections

### Content & Markup

The backend delivers semantic HTML with blocks as `<div class="{blockname}">` containing table-derived rows/cells. Inspect the actual markup before writing decoration logic:

```bash
curl http://localhost:3000/path/to/page.plain.html
```

Place static test pages in `drafts/` and start the server with `--html-folder drafts`.

## Deployment URLs

Given `owner/repo` from `gh repo view --json nameWithOwner` and current `branch`:

- Preview: `https://{branch}--{repo}--{owner}.aem.page/`
- Live: `https://main--{repo}--{owner}.aem.live/`
