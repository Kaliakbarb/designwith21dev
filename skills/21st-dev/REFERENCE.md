# 21st-dev Reference

## Magic MCP — install

```bash
npx @21st-dev/cli@latest install claude --api-key <your-key>
```

Get a key at https://21st.dev/magic/console (free tier available). Restart Claude Code after install.

**Manual config** for `~/.claude/settings.json` (or `.mcp.json`):

```json
{
  "mcpServers": {
    "@21st-dev/magic": {
      "command": "npx",
      "args": ["-y", "@21st-dev/magic@latest", "API_KEY=\"your-api-key\""]
    }
  }
}
```

Verify: `claude mcp list` should show `@21st-dev/magic`.

---

## Magic MCP — tool reference

The server exposes exactly four tools. Use the right one for the job.

### `21st_magic_component_inspiration`
**When**: phase 2 of Add flow — to fetch options before committing to a build.
**Returns**: JSON with matching components (no codegen).
**Schema**:
```
message: string         # full user message
searchQuery: string     # 2-4 words, e.g. "animated pricing table"
```

### `21st_magic_component_builder`
**When**: phase 4 of Add flow — to actually generate a chosen component.
**Returns**: component code (after the user picks in the 21st.dev canvas the tool opens).
**Schema**:
```
message: string                          # full user message
searchQuery: string                      # 2-4 words
absolutePathToCurrentFile: string        # the page being edited
absolutePathToProjectDirectory: string   # project root
standaloneRequestQuery: string           # self-contained component description
```

### `21st_magic_component_refiner`
**When**: Refine flow — improving an existing component file.
**Returns**: redesigned component code + integration notes.
**Schema**:
```
userMessage: string                  # full user message
absolutePathToRefiningFile: string   # absolute path to the file to redesign
context: string                      # specific aspects to improve (or "" if unclear)
```

### `logo_search`
**When**: Logo flow — fetching brand SVGs.
**Returns**: components/code in the requested format. Source: SVGL.
**Schema**:
```
queries: string[]                # lowercase company names, e.g. ["github", "discord"]
format: "JSX" | "TSX" | "SVG"
```

---

## Theme-align cheatsheet

### Where projects keep design tokens

| Project shape | Tokens live in |
|---|---|
| Next.js + shadcn/ui | `app/globals.css` (`:root { --primary: ...; }`) + `tailwind.config.ts` |
| Vite + React + Tailwind | `src/index.css` + `tailwind.config.js` |
| Plain Tailwind, no shadcn | `tailwind.config.{js,ts}` only — `theme.extend.colors` |
| CSS modules / vanilla | `:root` block in a global stylesheet |

### Common mappings (component → project token)

| Hardcoded in fetched component | Replace with (if project defines it) |
|---|---|
| `bg-blue-500`, `bg-[#3b82f6]`, `bg-indigo-600` | `bg-primary` |
| `text-white` on a brand bg | `text-primary-foreground` |
| `bg-gray-50`, `bg-white` (page bg) | `bg-background` |
| `text-gray-900`, `text-black` | `text-foreground` |
| `text-gray-500`, `text-gray-600` | `text-muted-foreground` |
| `border-gray-200` | `border-border` |
| `bg-gray-100` (subtle surface) | `bg-muted` |
| `font-['Inter']`, hardcoded font | the project's font var (`font-sans` if mapped via `next/font`) |
| `rounded-lg` (when project uses tokens) | `rounded-[var(--radius)]` |

### Algorithm

1. Read the global CSS — list all `--*` vars under `:root` and `.dark`.
2. Read the Tailwind config — list custom `theme.extend.colors` and font families.
3. For each hardcoded value in the component, find the closest matching token. If none exists, leave the original value and note it in DESIGN.md.
4. Don't invent tokens — only use ones the project actually defines.

---

## DESIGN.md template

Append this block to `<project root>/DESIGN.md` after a successful install (create the file if missing).

```markdown
## YYYY-MM-DD · <ComponentName>

- **Source:** 21st.dev · <preview-url>
- **File:** `components/sections/<Name>.tsx`
- **Wired into:** `app/page.tsx`
- **Deps added:** framer-motion, lucide-react
- **Theme tweaks:** mapped `bg-blue-500` → `bg-primary`, `text-gray-900` → `text-foreground`
- **Notes:** <anything non-obvious — e.g. "kept hardcoded gradient, no token match">
```

---

## Stack detection cheatsheet

| Key in `package.json` | Stack | Component extension |
|---|---|---|
| `"next"` | Next.js (React) | `.tsx` |
| `"react"` (no next) | Plain React | `.tsx` if `tsconfig.json` exists, else `.jsx` |
| `"vue"` | Vue 3 | flag conversion needed |
| `"svelte"` | Svelte | flag conversion needed |
| `"astro"` | Astro | `.tsx` (works in `.astro` islands) |
| `"@angular/core"` | Angular | flag full manual port needed |

### Target page detection

In priority order:
1. `app/page.tsx` (Next.js App Router)
2. `app/(home)/page.tsx`, `app/(marketing)/page.tsx` (route groups)
3. `pages/index.tsx` (Next.js Pages Router)
4. `src/App.tsx`, `src/App.jsx` (Vite/CRA)
5. `src/routes/+page.svelte` (SvelteKit)
6. `src/pages/index.astro` (Astro)
7. `index.html` (plain)

If multiple home-like pages exist, ask the user which one.

---

## File placement convention

```
components/
  ui/          ← atoms (button, badge, input)  ← never touch unless adding a primitive
  sections/    ← page sections (hero, pricing, footer)  ← default for Add flow
  layout/      ← structural (navbar, sidebar)
  icons/       ← logo_search results
```

If the project has no `components/` directory, check for `src/components/` first; otherwise create at root.

---

## Common peer dependencies

These appear frequently in 21st.dev components — check `package.json` before listing as missing:

| Package | Purpose |
|---|---|
| `tailwindcss` | Styling (nearly universal) |
| `framer-motion` | Animations |
| `lucide-react` | Icons |
| `@radix-ui/react-*` | Headless primitives (dialog, dropdown, etc.) |
| `class-variance-authority` | Variant styling |
| `clsx`, `tailwind-merge` | Conditional class names |
| `embla-carousel-react` | Carousels |
| `@react-three/fiber`, `three` | 3D / WebGL components |

---

## Verify phase — Playwright MCP usage

After `npm run dev` (background):

```
mcp__playwright__browser_navigate         url: "http://localhost:3000"
mcp__playwright__browser_take_screenshot  filename: "/tmp/21st-<component>.png"
mcp__playwright__browser_console_messages
```

Look for: red console errors, hydration mismatches, "Module not found", "X is not defined". If the dev server is on a non-standard port (3001, 5173, 4321, 8080), grep the dev-server background output for `Local:` to find it.

---

## Fallback: scrape mode (no MCP)

When Magic MCP isn't available, the skill scrapes 21st.dev directly. Less reliable — auth walls, paid components, and 404s are common.

### Browse / Search URLs
```
https://21st.dev/community/components                  # all components
https://21st.dev/community/components?search=<query>   # filtered
https://21st.dev/community/components?sort=popular     # by popularity
```

### Component pages
```
https://21st.dev/<creator>/<component-slug>
```

### Raw source attempts (try in order)
```
https://21st.dev/r/<component-slug>            # registry endpoint
https://21st.dev/<creator>/<slug>/raw          # raw fallback
```

### Fallback chain
1. `WebFetch` the registry search URL
2. If blocked, `WebSearch site:21st.dev <component name>`
3. If still nothing, hand-write a close equivalent and label it as such

### Query construction
- noun + adjective: `pricing table animated` not `animated pricing table`
- add `react` or `tailwind` to narrow
- style modifiers: `glassmorphism`, `minimal`, `dark`, `gradient`, `3d`

### Component categories

| Category | Search terms |
|---|---|
| Hero sections | `hero`, `hero section`, `landing hero` |
| Navigation | `navbar`, `navigation`, `header nav` |
| Pricing | `pricing`, `pricing table`, `pricing cards` |
| Testimonials | `testimonials`, `reviews`, `social proof` |
| CTA / Buttons | `cta`, `call to action`, `button animated` |
| Cards | `card`, `feature card`, `bento grid` |
| Chat / AI UI | `chat`, `ai chat`, `prompt input`, `message bubble` |
| Forms | `form`, `contact form`, `login form` |
| Footers | `footer`, `site footer` |
| Loaders | `loader`, `skeleton`, `spinner` |
| Backgrounds | `background`, `particle`, `gradient mesh`, `aurora` |
