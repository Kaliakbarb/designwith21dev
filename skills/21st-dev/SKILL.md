---
name: 21st-dev
description: >
  Search 21st.dev's community registry, install production-ready UI components,
  wire them into the actual page, theme-align them to the project, and verify
  the result in a real browser. Use when the user wants to add a UI section
  (hero, pricing, testimonials, navbar, CTA, chat UI, buttons, cards, etc.),
  refine an existing component, fetch brand logos, or asks to "make the site
  look good", "add a component", "find a nice X", or "I need a [section name]".
version: 2.1.0
user-invocable: true
argument-hint: "[component description]  e.g. 'animated pricing table' or 'refine components/Hero.tsx' or 'logo GitHub'"
allowed-tools: WebFetch, WebSearch, Read, Write, Edit, Bash, mcp__playwright__browser_navigate, mcp__playwright__browser_take_screenshot, mcp__playwright__browser_console_messages, mcp__playwright__browser_snapshot
---

# 21st-dev — UI Component Installer (v2.1)

You are a UI component assistant powered by [21st.dev](https://21st.dev/home), a community registry of production-ready UI components. v2.1 prefers **community-curated picks** (the human-voted grids at `/community/components/s/<slug>`) installed via the **shadcn registry** (`npx shadcn@latest add https://21st.dev/r/<creator>/<component>`) — the canonical, programmatic install path. Magic MCP is used only when the request doesn't map cleanly to a category, or the registry install fails. Theme-aligns the component, wires it into the page, and verifies it renders in a real browser before declaring success.

## Why community-first, not MCP-first

The community grid surfaces components ranked by **votes from real users** (e.g. Banner by nur/ui · 141 votes, Upgrade Banner by Victor Welander · 71). That's a far stronger quality signal than vector similarity on a free-text search query. And every component on 21st.dev is a shadcn registry item — `npx shadcn add <url>` handles peer deps, files, and tailwind config automatically. **Use the registry path whenever the user's intent maps to a category.** Fall back to Magic MCP only for off-grid requests (e.g. "an unusual hybrid of a hero and a chat thread").

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

1. `cat package.json` (or `Read`) — detect stack: `next`, `react`, `vue`, `svelte`, `astro`, `angular`
2. `ls` the project root — find target page (`app/page.tsx`, `pages/index.tsx`, `src/App.tsx`, `index.html`)
3. Check for `components.json` (shadcn config) — if present, the **shadcn registry install path is available**, which is the preferred install method
4. `claude mcp list` — does it list `@21st-dev/magic`? (only matters as a fallback for off-grid requests)

Then resolve:
- **What** section is needed (hero, pricing, nav, testimonials, CTA, card, etc.) → maps to a category slug like `/s/hero`, `/s/pricing`, `/s/announcement` (see [REFERENCE.md](REFERENCE.md) § Category slugs)
- **Style intent** if mentioned (minimal, animated, glassmorphism, dark, gradient)
- **Target page** to wire it into (default to the home/index page detected above; ask if ambiguous)

If the request is vague ("make it look good"), ask one focused question: *"Which section are you working on, and which page should I add it to?"*

### Phase 2 — Browse the community grid

This is the **default path** — community-voted picks beat free-text search.

`WebFetch https://21st.dev/community/components/s/<slug>` (e.g. `/s/hero`, `/s/announcement`). The page lists components ranked by votes. Parse the listing:

- Each card has: component name, creator handle, vote count, preview thumbnail, and a detail-page URL of the form `/community/components/<creator>/<component>/<variant>`
- The shadcn registry URL for each component is `https://21st.dev/r/<creator>/<component>` — this is the install URL

**If the user's intent doesn't map to any category slug** (e.g. "an unusual hybrid of a hero and a chat thread"), skip to the **MCP fallback** at the bottom of this section.

See [REFERENCE.md](REFERENCE.md) § Category slugs for the full list.

### Phase 3 — Present top community picks

Show the user the **top 3 by votes**, plus 1 stylistic outlier if the user mentioned style intent:

```
Top community picks for "announcement":

1. **Banner** by @nur/ui — 141 votes
   → https://21st.dev/r/nurui/banner

2. **Upgrade Banner** by @victorwelander — 71 votes
   → animated banner with floating gear icons, hover effects, dismiss button
   → https://21st.dev/r/victorwelander/upgrade-banner

3. **Animated Shiny Text** by @magicui — 67 votes
   → https://21st.dev/r/magicui/animated-shiny-text
```

Ask: *"Which one? (1/2/3, or describe what you'd prefer)"*

### Phase 4 — Install via shadcn registry

This is the killer step. **One command** installs the component, all peer deps, all primitives, and any tailwind config edits:

```bash
npx shadcn@latest add https://21st.dev/r/<creator>/<component>
```

Run it from the project root. shadcn handles:
- Writing the component file to `components/ui/<name>.tsx` (or wherever `components.json` says)
- Installing npm peer deps (`framer-motion`, `lucide-react`, `@radix-ui/*`, etc.)
- Installing any missing shadcn primitives the component depends on
- Extending `tailwind.config` if the registry item declares it

**No MCP. No scraping. No manual `npm install`.** This is the canonical 21st.dev install path.

If `npx shadcn add` fails (no internet, registry 404, paid component, project not shadcn-compatible), fall through to the **MCP fallback** below.

### Phase 5 — Theme-align

Even after `shadcn add`, the component may use tokens or classes the project doesn't have. Before wiring in:

1. Open the freshly-installed file
2. Read `tailwind.config.{ts,js}` and the global CSS (`app/globals.css`, `src/index.css`, or `styles/globals.css`) — collect:
   - CSS variables: `--background`, `--foreground`, `--primary`, `--accent`, `--radius`, etc.
   - Tailwind theme extensions: custom colors, font families
3. In the installed component, fix mismatches:
   | Issue | Action |
   |---|---|
   | Component references `--brand` / `--brand-foreground` and project doesn't have them | Drop those references — don't invent the vars; replace with `--primary` / `--primary-foreground` or remove the styling |
   | Component uses a custom Tailwind class like `animate-appear` not in our config | Inline the keyframes via `<style>{...}</style>` rather than editing `tailwind.config` |
   | Component uses `Button asChild` (Radix Slot) but project's button is `@base-ui/react`-based | Swap to anchor + `buttonVariants(...)` — `asChild` doesn't exist on base-ui buttons |
   | Hardcoded `bg-blue-500`, `bg-[#3b82f6]` | Replace with `bg-primary` if defined |
   | `text-gray-900`, `text-black` | `text-foreground` |
   | `text-gray-500`, `text-gray-600` | `text-muted-foreground` |
   | `font-['Inter']` or hardcoded font | project's font var (`font-sans` if mapped via `next/font`) |

See [REFERENCE.md](REFERENCE.md) § Theme-align cheatsheet for the full mapping. If the project has **no design tokens**, leave the component as-is and note it in DESIGN.md.

### Phase 6 — Wire-in

1. **Move/rename** if needed — `shadcn add` writes to `components/ui/<name>.tsx`. Sections (hero, pricing, footer) are conventionally in `components/sections/`. Move and update imports if so.
2. **Edit the target page** to import and render the component:
   ```tsx
   import { Hero } from "@/components/sections/Hero"
   // ...
   <Hero />
   ```
   Use `Edit` to insert the import and the JSX in the appropriate place (top of the page, inside the layout's main wrapper).
3. **Replace boilerplate** — if `app/page.tsx` is still the default Next.js scaffold, replace it entirely so the new section is visible above the fold.

### MCP fallback (when registry install isn't viable)

Use Magic MCP only when:
- The user's intent doesn't map to a category slug, OR
- `npx shadcn add` fails (registry 404, project not shadcn-compatible), OR
- The user explicitly wants a generated/customized component, not a community pick

**Search**: `21st_magic_component_inspiration` — returns JSON of matching components by similarity.
**Generate**: `21st_magic_component_builder` — opens 21st.dev's canvas in the browser, user picks, returns code.

Then proceed to Phase 5 (theme-align) and Phase 6 (wire-in).

**MCP missing?** Suggest install (`npx @21st-dev/cli@latest install claude --api-key <key>` from https://21st.dev/magic/console) and ask the user to restart Claude Code — but don't block the registry path on it.

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
