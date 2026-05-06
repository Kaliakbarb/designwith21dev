# designwith21dev

A Claude Code skill that connects Claude to [21st.dev](https://21st.dev/home) — a community registry of production-ready UI components. Instead of hand-writing boilerplate UI, Claude searches the registry, shows you options, drops the chosen component into your project, **wires it into the actual page, theme-aligns it to your design tokens, and verifies it renders in a real browser** before declaring success.

---

## What's new in v2.1

v2.1 corrects the architecture: **community grid + shadcn registry first, MCP as fallback.**

The previous version did vector-similarity searches via Magic MCP — useful, but worse than the community-voted grids 21st.dev publishes at `/community/components/s/<slug>`. The grids surface components ranked by **real user votes** (e.g. Banner by nur/ui · 141 votes), and every component on the site is a shadcn registry item installable in one command:

```bash
npx shadcn@latest add https://21st.dev/r/<creator>/<component>
```

That single command handles files, peer deps, missing primitives, and tailwind config. No scraping. No Magic MCP API key required for the common path.

### Full v2.1 feature set

- **Community-grid first** — fetches `/community/components/s/<slug>` for the user's category, presents the top 3 by votes
- **Registry-install primary** — uses `npx shadcn add <21st.dev/r/...>` as the canonical install path
- **Magic MCP fallback** — kept for off-grid requests ("an unusual hybrid of X and Y") and for the **Refine** + **Logo** flows
- **Theme-aligned** — fixes common post-install issues: components referencing `--brand` vars you don't have, custom Tailwind classes like `animate-appear`, or `Button asChild` patterns that break on `@base-ui/react`-style buttons
- **Wired into the page** — edits `app/page.tsx` (or your detected home page) to import and render the component, replaces boilerplate so the new section is visible
- **Verified in a real browser** — Playwright MCP screenshot + console scan
- **Logged to DESIGN.md** — date, component, source URL, deps, theme tweaks
- **Refine flow** — `21st_magic_component_refiner` redesigns existing components in place
- **Logo flow** — `logo_search` (or SVGL fallback) fetches brand SVGs as TSX/JSX/SVG

---

## What it does

Say something like:

- *"add a hero section"*
- *"I need an animated pricing table"*
- *"find me a glassmorphism card component"*
- *"refine components/Hero.tsx — make it more minimal"*
- *"add GitHub and Discord logos"*
- *"make this site look good"*

Claude routes to the right sub-flow:

| Intent | Flow |
|---|---|
| Add a section/component | **Add** (7-phase, see below) |
| Improve an existing file | **Refine** (Magic MCP `component_refiner`) |
| Fetch a brand logo | **Logo** (Magic MCP `logo_search` via SVGL) |

---

## Setup

### 1. Install the skill

```bash
git clone https://github.com/Kaliakbarb/designwith21dev.git ~/designwith21dev
ln -s ~/designwith21dev/skills/21st-dev ~/.claude/skills/21st-dev
```

(Symlink so updates pull automatically. Or `cp -r` if you prefer a standalone copy.)

### 2. Install Magic MCP (recommended)

The skill works without it (scrape mode), but Magic MCP is dramatically more reliable.

```bash
npx @21st-dev/cli@latest install claude --api-key <your-key>
```

Get a key at <https://21st.dev/magic/console> — free tier available. Restart Claude Code after install.

Verify with `claude mcp list` — you should see `@21st-dev/magic`.

### 3. (Optional) Playwright MCP for verification

The verify phase uses Playwright MCP to screenshot the result and scan console errors. If it's not installed, the skill skips verification and tells you so. Install per Playwright MCP's docs.

---

## Usage

```
/21st-dev animated pricing table
/21st-dev hero section with gradient background
/21st-dev glassmorphism navbar
/21st-dev refine components/sections/Hero.tsx — more minimal
/21st-dev logo GitHub Discord Slack
```

It also triggers naturally mid-conversation on phrases like *"add a component"*, *"find a nice X"*, *"make the site look good"*.

---

## The 7-phase Add workflow (v2.1)

| Phase | What happens |
|---|---|
| 1. Detect & clarify | Read `package.json`, find target page, check for `components.json` (shadcn → registry-install available); resolve which category slug the request maps to |
| 2. Browse community grid | `WebFetch https://21st.dev/community/components/s/<slug>` — parse cards (name, creator, vote count, registry URL) |
| 3. Present top picks | Top 3 by votes, plus a stylistic outlier if the user mentioned style intent |
| 4. Install via shadcn registry | `npx shadcn@latest add https://21st.dev/r/<creator>/<component>` — handles files, deps, primitives, and tailwind config in one command |
| 5. Theme-align | Fix post-install issues: missing `--brand` vars, custom `animate-appear` classes, `asChild` mismatch on base-ui buttons, hardcoded colors |
| 6. Wire-in | Move/rename if needed, edit target page to import and render, replace boilerplate so the new section is visible |
| 7. Verify | `npm run dev` (background), Playwright `browser_navigate` + `browser_take_screenshot` + `browser_console_messages`. Append `DESIGN.md` entry on success |

### When does the skill use Magic MCP?

Only when:
- The user's intent doesn't map to a category slug ("an unusual hybrid of a hero and a chat thread"), OR
- `npx shadcn add` fails (registry 404, project not shadcn-compatible), OR
- The user explicitly wants a generated/customized component, not a community pick

Magic MCP is also the engine for the **Refine** flow (`21st_magic_component_refiner`) and the **Logo** flow (`logo_search`).

---

## Supported stacks

| Stack | Support |
|---|---|
| Next.js (App Router & Pages Router) | Native — full wire-in |
| React (Vite, CRA) | Native — full wire-in |
| Astro | React components in `.astro` islands |
| Vue 3 | Installs React version, flags conversion |
| Svelte / SvelteKit | Installs React version, flags conversion |
| Angular | Flags full manual port needed |

---

## Skill files

```
skills/
└── 21st-dev/
    ├── SKILL.md      ← 7-phase workflow, refine flow, logo flow
    └── REFERENCE.md  ← Magic MCP tool schemas, theme-align cheatsheet, DESIGN.md template, scrape fallback
```

### `SKILL.md`
The main workflow file. Covers the three sub-flows (Add / Refine / Logo), the 7 phases of Add, quality rules, and the final success format.

### `REFERENCE.md`
Lookup tables and details that don't need to live in the workflow:
- Magic MCP install (CLI + manual config) and the four tool schemas
- Theme-align cheatsheet — token mappings, where each framework keeps its tokens
- `DESIGN.md` entry template
- Stack & target-page detection cheatsheets
- Common peer dependencies
- Scrape-mode fallback (URL patterns, query construction, fallback chain)

---

## Quality rules

- Never claims success without a screenshot or an explicit "verification skipped because <reason>"
- Never overwrites an existing component file without confirming
- Skips paid/locked components and offers a free alternative
- Always runs `npm install` (and `npx shadcn add` when needed) — doesn't just print the commands
- Always edits the target page — installing the file alone is not "done"
- Reuses an already-running dev server instead of spawning a second one
- Shows install commands as code blocks before running them

---

## Requirements

- [Claude Code](https://claude.ai/code) CLI or desktop app
- A frontend project (React, Next.js, Vue, Svelte, or Astro)
- Recommended: 21st.dev API key for Magic MCP (<https://21st.dev/magic/console>)
- Optional: Playwright MCP (for the verify phase)

---

## License

MIT
