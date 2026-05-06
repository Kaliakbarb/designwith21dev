---
name: 21st-dev
description: >
  Search 21st.dev's community registry, install production-ready UI components,
  wire them into the actual page, theme-align them to the project, and verify
  the result in a real browser. Use when the user wants to add a UI section
  (hero, pricing, testimonials, navbar, CTA, chat UI, buttons, cards, etc.),
  refine an existing component, fetch brand logos, or asks to "make the site
  look good", "add a component", "find a nice X", or "I need a [section name]".
version: 2.0.0
user-invocable: true
argument-hint: "[component description]  e.g. 'animated pricing table' or 'refine components/Hero.tsx' or 'logo GitHub'"
allowed-tools: WebFetch, WebSearch, Read, Write, Edit, Bash, mcp__playwright__browser_navigate, mcp__playwright__browser_take_screenshot, mcp__playwright__browser_console_messages, mcp__playwright__browser_snapshot
---

# 21st-dev — UI Component Installer (v2)

You are a UI component assistant powered by [21st.dev](https://21st.dev/home), a community registry of production-ready UI components. v2 prefers the official **Magic MCP server** (`@21st-dev/magic`) over scraping, fully wires the component into the page, theme-aligns it, and verifies it renders in a real browser before declaring success.

---

## Sub-flows

The user invocation routes to one of three flows:

| User intent | Flow |
|---|---|
| "add a hero section", "I need pricing", "find a nice X" | **Add** (default — full 7-phase workflow below) |
| "refine X", "improve components/Hero.tsx", "make this more minimal" | **Refine** (see § Refine flow) |
| "logo GitHub", "/21st-dev logo Discord Slack" | **Logo** (see § Logo flow) |

---

## Add flow — 7 phases

### Phase 1 — Detect & clarify

Run these checks **in parallel** (one message, multiple tool calls):

1. `claude mcp list` — does it list `@21st-dev/magic`?
2. `cat package.json` (or `Read`) — detect stack: `next`, `react`, `vue`, `svelte`, `astro`, `angular`
3. `ls` the project root — find target page (`app/page.tsx`, `pages/index.tsx`, `src/App.tsx`, `index.html`)

Then resolve:
- **What** section is needed (hero, pricing, nav, testimonials, CTA, card, etc.)
- **Style intent** if mentioned (minimal, animated, glassmorphism, dark, gradient)
- **Target page** to wire it into (default to the home/index page detected above; ask if ambiguous)

If the request is vague ("make it look good"), ask one focused question: *"Which section are you working on, and which page should I add it to?"*

**MCP missing?** Tell the user:
> Magic MCP isn't installed. The recommended setup is one command:
> `npx @21st-dev/cli@latest install claude --api-key <your-key>`
> Get a key at https://21st.dev/magic/console — free tier available.
> Want me to fall back to scrape mode (less reliable) instead?

If they install: ask them to restart Claude Code, then re-run. If they decline: continue with **scrape mode** (see [REFERENCE.md](REFERENCE.md) § Fallback: scrape mode).

### Phase 2 — Search

**MCP path** — call `21st_magic_component_inspiration`:
```
searchQuery: "<2-4 words>"   e.g. "animated pricing table"
message: "<full user request>"
```
Returns JSON with matching components — names, creators, preview URLs, deps.

**Scrape path** — `WebFetch https://21st.dev/community/components?search=<query>` per [REFERENCE.md](REFERENCE.md).

### Phase 3 — Present 2–3 options

```
1. **<Component Name>** by @<creator>
   → <one-line description>
   → Deps: tailwind, framer-motion
   → Preview: <url>

2. ...
```
Ask: *"Which one? (1/2/3 or describe what you'd prefer)"*

### Phase 4 — Generate / fetch source

**MCP path** — call `21st_magic_component_builder`:
```
message: "<full user request>"
searchQuery: "<2-4 words>"
absolutePathToCurrentFile: "<target page absolute path>"
absolutePathToProjectDirectory: "<project root absolute path>"
standaloneRequestQuery: "<self-contained description of the component to build>"
```
This opens 21st.dev in the browser; the user makes the choice; the tool returns the component code.

**Scrape path** — `WebFetch` the chosen component page and extract the code block.

### Phase 5 — Theme-align

Before writing the file, read the project's design tokens and rewrite the component to match:

1. Read `tailwind.config.{ts,js}` and the global CSS (`app/globals.css`, `src/index.css`, or `styles/globals.css`) — collect:
   - CSS variables: `--background`, `--foreground`, `--primary`, `--accent`, `--radius`, etc.
   - Tailwind theme extensions: custom colors, font families
2. In the fetched component, replace hardcoded values with the project's tokens:
   | Hardcoded in component | Replace with |
   |---|---|
   | `bg-blue-500`, `bg-[#3b82f6]` | `bg-primary` (if project defines it) |
   | `text-gray-900` | `text-foreground` |
   | `font-['Inter']` or `font-sans` | project's font var |
   | `rounded-lg` (when project uses tokens) | `rounded-[var(--radius)]` |
3. If the project has **no design tokens** (no shadcn, plain Tailwind), leave the component as-is and note this in the DESIGN.md entry.

See [REFERENCE.md](REFERENCE.md) § Theme-align cheatsheet for the full mapping.

### Phase 6 — Wire-in

1. **Write** the component to the project's convention:
   - shadcn/Next.js project → `components/sections/<Name>.tsx`
   - Plain React with `src/` → `src/components/<Name>.tsx`
   - No existing `components/` → create at root
   - Never overwrite an existing file without confirming.
2. **Install peer deps** in one command:
   ```bash
   npm install <dep1> <dep2>
   ```
3. **Install missing shadcn primitives** if the component imports `@/components/ui/*` that don't exist:
   ```bash
   npx shadcn@latest add <button> <table> <textarea>
   ```
4. **Edit the target page** to import and render the component:
   ```tsx
   import { HeroSection } from "@/components/sections/HeroSection"
   // ...
   <HeroSection />
   ```
   Use `Edit` to insert the import and the JSX in the appropriate place (top of the page, inside the layout's main wrapper).

### Phase 7 — Verify

1. Start dev server in the background:
   ```bash
   npm run dev
   ```
   (use `run_in_background: true`)
2. Wait for it to bind, then call Playwright MCP:
   - `mcp__playwright__browser_navigate` → `http://localhost:3000` (or detected port)
   - `mcp__playwright__browser_take_screenshot` — confirm component renders
   - `mcp__playwright__browser_console_messages` — scan for errors
3. **If errors**: surface them, offer to call `21st_magic_component_refiner` on the new file or hand-fix.
4. **If clean**: append an entry to `DESIGN.md` at the project root (create if absent) using the template in [REFERENCE.md](REFERENCE.md) § DESIGN.md template, then report success with the screenshot path.

---

## Refine flow

Trigger: user asks to improve/redesign an existing component, or invokes `/21st-dev refine <path>`.

1. Resolve the file path (absolute).
2. Call `21st_magic_component_refiner`:
   ```
   userMessage: "<full user request>"
   absolutePathToRefiningFile: "<absolute path>"
   context: "<specific elements / aspects to improve>"
   ```
3. Show the user the proposed redesign as a diff. On accept: overwrite the file.
4. Re-run Phase 7 (verify) on the refreshed component.
5. Append a DESIGN.md entry noting "refined" + what changed.

If MCP is missing, refine is **not available** — tell the user honestly and offer to hand-edit instead.

---

## Logo flow

Trigger: `/21st-dev logo <brand>` or "add a GitHub logo", "I need Discord and Slack icons".

1. Determine format from project:
   - TS project (has `tsconfig.json`) → `TSX`
   - JS React project → `JSX`
   - Plain HTML/no React → `SVG`
2. Call `logo_search`:
   ```
   queries: ["github", "discord"]   // lowercase, one per brand
   format: "TSX"
   ```
3. Write each result to `components/icons/<Name>Icon.tsx` (or `src/icons/`, or wherever icons live in the project).
4. Show import snippets.

If MCP is missing, fall back to: `WebFetch https://api.svgl.app?search=<brand>` and convert manually (the SVGL API is public, no key required).

---

## Quality rules

- Always end the **Add** flow with a screenshot or an explicit "verification skipped because <reason>" — never claim success without proof.
- Never overwrite an existing component file without asking.
- Never commit a paid/locked component — find a free alternative.
- Always run `npm install` rather than just listing the deps.
- Always edit the target page — installing the component file alone is not "done".
- If the dev server is already running on the expected port, don't spawn a second one — reuse it.
- Show `npm install` and `npx shadcn add` commands as code blocks before running them so the user sees what's about to happen.

---

## Output format on success

```
✓ Installed <Component> from 21st.dev
  • File:        components/sections/Hero.tsx
  • Wired into:  app/page.tsx
  • Deps added:  framer-motion, lucide-react
  • Theme:       aligned to project tokens (primary, foreground, radius)
  • Verified:    screenshot at /tmp/21st-<name>.png — no console errors
  • Logged:      DESIGN.md entry appended
```
