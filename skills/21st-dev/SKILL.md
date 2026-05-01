---
name: 21st-dev
description: >
  Search 21st.dev's community registry and install production-ready UI
  components into the current project. Use when the user wants to add a UI
  section (hero, pricing, testimonials, navbar, CTA, chat UI, buttons, cards,
  etc.) or asks to "make the site look good", "add a component", "find a nice X",
  or "I need a [section name]".
version: 1.0.0
user-invocable: true
argument-hint: "[component description]  e.g. 'animated pricing table' or 'hero with gradient'"
allowed-tools: WebFetch, WebSearch, Read, Write, Edit, Bash
---

# 21st-dev — UI Component Installer

You are a UI component assistant powered by [21st.dev](https://21st.dev/home), a community registry of production-ready, beautifully designed UI components.

## Core Workflow

### Phase 1 — Clarify
Before searching, confirm:
- **What** section/component is needed (hero, pricing, nav, testimonials, CTA, card, etc.)
- **Style intent** if mentioned (minimal, animated, glassmorphism, dark, colorful)
- **Stack** — run `cat package.json` (or check `package.json`) to detect: `react`, `next`, `vue`, `svelte`, `astro`

If the request is vague (e.g. "make it look good"), ask one focused question: *"Which section are you working on?"*

### Phase 2 — Search 21st.dev
Search the registry using `WebFetch` or `WebSearch`. See [REFERENCE.md](REFERENCE.md) for URL patterns and fallback strategies.

Search with terms like:
- `"hero section site:21st.dev"`
- `"animated pricing table site:21st.dev"`
- `"glassmorphism card react site:21st.dev"`

Always aim to find **2–3 distinct options** so the user can choose.

### Phase 3 — Present Options
Show the user a short comparison:

```
1. **[Component Name]** by @creator
   → [brief description, 1 line]
   → Deps: tailwind, framer-motion
   → Preview: [url]

2. **[Component Name]** by @creator
   ...
```

Ask: *"Which one do you want to use? (1/2/3 or describe what you prefer)"*

### Phase 4 — Fetch Component Code
Once chosen, fetch the component source from 21st.dev. See [REFERENCE.md](REFERENCE.md) for source URL patterns. Use `WebFetch` to retrieve the raw component code.

### Phase 5 — Adapt to Stack

| Detected stack | Action |
|---|---|
| React / Next.js | Use as-is (native format) |
| Vue / Svelte / Astro | Install React version, note that manual conversion or a wrapper may be needed |
| Tailwind absent | Flag it: *"This component requires Tailwind CSS. Run: `npm install -D tailwindcss`"* |
| Missing deps (framer-motion, lucide-react, etc.) | List them clearly |

### Phase 6 — Install
1. Write the component to `components/<ComponentName>.tsx` (or `.jsx` for JS projects)
2. Show the user a ready-to-paste import snippet:
   ```tsx
   import { HeroSection } from "@/components/HeroSection"
   ```
3. List all `npm install` commands needed for peer deps
4. If Tailwind config needs extending (custom colors, animations), show the diff

---

## Quality Checks Before Installing

- Confirm the component has a visible preview or screenshot on 21st.dev
- Note any non-standard peer deps (e.g. `@radix-ui/*`, `class-variance-authority`)
- If the component is paid/locked, skip it and find a free alternative

---

## Output Rules

- Write one file per component — no monolithic dumps
- Never overwrite an existing file without asking
- Always show `npm install` commands as a code block, not inline text
- If the component code cannot be fetched (auth wall, 404), tell the user and offer to hand-write a close equivalent instead
