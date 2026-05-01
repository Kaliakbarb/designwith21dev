# designwith21dev

A Claude Code skill that connects Claude to [21st.dev](https://21st.dev/home) — a community registry of production-ready, beautifully designed UI components. Instead of hand-writing boilerplate UI, Claude searches the registry, shows you options, and drops the chosen component directly into your project.

---

## What It Does

Say something like:

- *"add a hero section"*
- *"I need an animated pricing table"*
- *"find me a glassmorphism card component"*
- *"make this site look good"*

Claude will:

1. **Clarify** the UI need and detect your frontend stack from `package.json`
2. **Search** 21st.dev's community registry for matching components
3. **Present** 2–3 options with name, creator, deps, and preview link
4. **Fetch** the source code of your chosen component
5. **Adapt** it to your stack — flags missing Tailwind, peer deps, or conversion notes for non-React projects
6. **Install** it — writes the component file, shows the import snippet, lists `npm install` commands

---

## Installation

**One-liner** — clone and symlink so updates pull automatically:

```bash
git clone https://github.com/Kaliakbarb/designwith21dev.git ~/designwith21dev
ln -s ~/designwith21dev/skills/21st-dev ~/.claude/skills/21st-dev
```

**Or copy** if you prefer a standalone install:

```bash
git clone https://github.com/Kaliakbarb/designwith21dev.git
cp -r designwith21dev/skills/21st-dev ~/.claude/skills/21st-dev
```

Restart Claude Code after installing. The skill appears immediately as `/21st-dev`.

---

## Usage

Open any frontend project in Claude Code and invoke the skill:

```
/21st-dev animated pricing table
/21st-dev hero section with gradient background
/21st-dev glassmorphism navbar
/21st-dev I need a nice testimonials section
```

You can also trigger it naturally mid-conversation — Claude will recognize phrases like *"add a component"*, *"find a nice X"*, or *"make the site look good"*.

---

## Supported Stacks

| Stack | Support |
|---|---|
| React | Native — components used as-is |
| Next.js | Native — components used as-is |
| Vue 3 | Installs React version, flags conversion needed |
| Svelte | Installs React version, flags conversion needed |
| Astro | React components work inside `.astro` islands |
| Angular | Flags full manual port needed |

---

## Skill Files

```
skills/
└── 21st-dev/
    ├── SKILL.md       ← main workflow instructions for Claude
    └── REFERENCE.md   ← URL patterns, search tips, dep map, file conventions
```

### SKILL.md — 6-Phase Workflow

| Phase | What happens |
|---|---|
| 1. Clarify | Extract the UI need; detect stack from `package.json` |
| 2. Search | Query `21st.dev/community/components?search=...` |
| 3. Present | Show 2–3 options with name, creator, deps, preview URL |
| 4. Fetch | Pull component source via `WebFetch` |
| 5. Adapt | Check Tailwind presence, peer deps, non-React stacks |
| 6. Install | Write file to `components/`, show import snippet + `npm install` |

### REFERENCE.md — Lookup Tables

- 21st.dev URL patterns for browsing, searching, and fetching raw source
- Search query construction tips
- Common component categories and search terms
- Common peer dependencies (`framer-motion`, `lucide-react`, `@radix-ui/*`, etc.)
- Stack detection cheatsheet
- File placement conventions (`components/ui/`, `components/sections/`, etc.)

---

## Quality Rules

- Never overwrites an existing file without asking
- Skips paid/locked components and finds a free alternative
- Always shows `npm install` commands as a code block
- If source can't be fetched (auth wall / 404), offers to hand-write a close equivalent

---

## Requirements

- [Claude Code](https://claude.ai/code) CLI or desktop app
- A frontend project (React, Next.js, Vue, Svelte, or Astro)
- Internet access (skill uses `WebFetch` and `WebSearch` at runtime)

---

## License

MIT
