# designwith21dev

A Claude Code skill that connects Claude to [21st.dev](https://21st.dev/home) — a community registry of production-ready UI components. Instead of hand-writing boilerplate UI, Claude searches the registry, shows you options, drops the chosen component into your project, **wires it into the actual page, theme-aligns it to your design tokens, and verifies it renders in a real browser** before declaring success.

---

## What's new in v2

The previous version stopped at "write file + list deps". v2 actually finishes the job:

- **MCP-first** — uses the official `@21st-dev/magic` MCP server when installed (far more reliable than scraping). Falls back to `WebFetch` only if MCP isn't available.
- **Theme-aligned** — reads your `tailwind.config.{ts,js}` and `globals.css`, then rewrites hardcoded colors / fonts / radii in the fetched component to use your project's design tokens (`bg-primary`, `text-foreground`, `rounded-[var(--radius)]`, etc.).
- **Wired into the page** — not just `components/<Name>.tsx` — also edits `app/page.tsx` (or your detected home page) to import and render the component.
- **Auto-installs deps** — runs `npm install` for peer deps and `npx shadcn@latest add` for missing primitives. No manual copy-paste.
- **Verified in a real browser** — starts the dev server, navigates with Playwright MCP, takes a screenshot, scans console for errors. If anything's broken, surfaces it.
- **Logged to DESIGN.md** — appends a short entry per install (date, component, source URL, deps, theme tweaks) so the project keeps a running design history.
- **Refine flow** — `21st_magic_component_refiner` redesigns existing components in place.
- **Logo flow** — `logo_search` fetches brand SVGs as TSX/JSX/SVG via SVGL.

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

## The 7-phase Add workflow

| Phase | What happens |
|---|---|
| 1. Detect & clarify | `claude mcp list`, read `package.json`, detect target page; ask one question if vague |
| 2. Search | MCP: `21st_magic_component_inspiration`. Fallback: `WebFetch` 21st.dev search |
| 3. Present | Show 2–3 options (name, creator, deps, preview URL) |
| 4. Generate | MCP: `21st_magic_component_builder` (opens 21st.dev canvas → returns code). Fallback: scrape |
| 5. Theme-align | Rewrite hardcoded colors/fonts/radii to use the project's CSS vars and Tailwind tokens |
| 6. Wire-in | Write component file, `npm install` deps, `npx shadcn add` missing primitives, edit the target page |
| 7. Verify | `npm run dev` (background), Playwright `browser_navigate` + `browser_take_screenshot` + `browser_console_messages`. Append `DESIGN.md` entry on success |

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
