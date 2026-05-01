# 21st.dev Reference

## URL Patterns

### Browse / Search
```
https://21st.dev/community/components                        # all components
https://21st.dev/community/components?search=<query>        # filtered search
https://21st.dev/community/components?sort=popular          # by popularity
https://21st.dev/community/components?sort=newest           # newest first
```

### Component Pages
```
https://21st.dev/<creator>/<component-slug>                 # component detail page
```

### Raw Component Source
21st.dev uses an open registry model. Component source can often be fetched via:
```
https://21st.dev/r/<component-slug>                         # registry endpoint (try this first)
https://21st.dev/<creator>/<slug>/raw                       # raw view fallback
```

If the above return auth walls or 404s, fall back to:
1. Scrape the component detail page with `WebFetch` and extract the code block
2. Use `WebSearch` with `site:21st.dev <component name> code` to find a cached/indexed version

---

## Search Strategies

### Primary: WebFetch the registry search
```
WebFetch: https://21st.dev/community/components?search=hero+section
```

### Fallback: WebSearch
```
WebSearch: site:21st.dev animated hero section react
WebSearch: site:21st.dev pricing table tailwind component
```

### Query construction tips
- Use **noun + adjective** order: `pricing table animated` not `animated pricing table`
- Add `react` or `tailwind` to narrow to compatible results
- For style-specific searches add: `glassmorphism`, `minimal`, `dark`, `gradient`, `3d`

---

## Common Component Categories on 21st.dev

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

---

## Common Peer Dependencies

These appear frequently in 21st.dev components — check `package.json` before listing as missing:

| Package | Purpose |
|---|---|
| `tailwindcss` | Styling (nearly universal) |
| `framer-motion` | Animations |
| `lucide-react` | Icons |
| `@radix-ui/react-*` | Headless primitives (dialogs, dropdowns) |
| `class-variance-authority` | Variant styling utility |
| `clsx` or `tailwind-merge` | Conditional class names |
| `shadcn/ui` | Component primitives (if component is shadcn-based) |

---

## Stack Detection Cheatsheet

Read `package.json` dependencies/devDependencies to detect:

| Key found | Stack |
|---|---|
| `"next"` | Next.js (React, use `.tsx`) |
| `"react"` (no next) | Plain React (use `.tsx` or `.jsx`) |
| `"vue"` | Vue 3 — note conversion needed |
| `"svelte"` | Svelte — note conversion needed |
| `"astro"` | Astro — React components work in `.astro` islands |
| `"@angular/core"` | Angular — full manual port needed |

---

## File Placement Convention

```
src/
  components/
    ui/          ← for small atoms (button, badge, input)
    sections/    ← for page sections (hero, pricing, footer)
    layout/      ← for structural components (navbar, sidebar)
```

If the project has no `src/` prefix, use `components/` directly at root.
