# site-identity-snapshot

A Claude Code skill that captures the complete visual identity of any public website into a structured reference folder — perfect for **rebranding**, **competitive analysis**, or **design system extraction**.

## What it does

Given a URL, produces a self-contained reference folder with:

- **All static assets** — HTML, CSS, JS, images, fonts (downloaded via `curl`)
- **Screenshots** — desktop + mobile, full-page + viewport + key sections (via Chrome DevTools MCP)
- **Design tokens (JSON)** — colors, typography, spacing, radii, shadows, transitions, keyframes, breakpoints — extracted from the live DOM
- **Human-readable visual identity guide** — narrative covering brand tone, palette, type system, components, animations, page structure, and rebranding direction
- **README** — methodology, folder map, and how to use the snapshot as reference

**Scope: capture-only.** This skill does NOT implement anything in your target project. It produces reference artifacts that feed *into* implementation (in any stack: Tailwind, CSS vars, Figma, etc.).

## Output structure

```
<DEST>/
├── README.md                       ← snapshot documentation + how to use
├── analysis/
│   ├── visual-identity-guide.md    ← human-readable guide
│   └── design-tokens.json          ← structured tokens
├── html/index.html
├── css/                            ← all linked stylesheets
├── js/                             ← site's own scripts only (no analytics/chat)
├── images/                         ← organized by use (landing/, graphics/, icons/)
├── fonts/                          ← @font-face self-hosted, grouped by family
└── screenshots/
    ├── desktop-fullpage.png
    ├── desktop-hero.png
    ├── mobile-fullpage.png
    ├── mobile-hero.png
    └── section-*.png
```

## Installation

```bash
git clone https://github.com/eduardodotai/site-identity-snapshot.git ~/.claude/skills/site-identity-snapshot
```

## Requirements

- [Claude Code](https://claude.ai/code) CLI or IDE extension
- [Chrome DevTools MCP](https://github.com/anthropics/claude-code/blob/main/docs/mcp.md) plugin enabled — **required** for screenshots and live token extraction
- `curl` available in shell

## Usage

```bash
# Basic — defaults to reference/<domain>/
snapshot do site https://example.com

# In English
capture the visual identity of https://competitor.com

# With custom destination
extrair design tokens de https://startup.io para reference/inspiration/

# Multiple triggers
referência visual de https://brand.com para o rebranding
design scraper de https://landing.page
```

## How it works

5 sequential phases:

| Phase | What it does | Tooling |
|-------|--------------|---------|
| **1. Reconnaissance** | Maps all assets (CSS, JS, images, fonts), classifies own vs third-party | Chrome DevTools MCP |
| **2. Download** | `curl` all own assets, organized by type and use | `curl` |
| **3. Screenshots** | Desktop + mobile, full-page + viewport + key sections | Chrome DevTools MCP |
| **4. Token Extraction** | JS injected via `evaluate_script` extracts computed styles, `@keyframes`, breakpoints from the live DOM | Chrome DevTools MCP |
| **5. Documentation** | Generates `design-tokens.json`, `visual-identity-guide.md`, `README.md` | Write |

## What's captured vs skipped

**Captured:**
- HTML (main page)
- All linked CSS files (including @font-face stylesheets)
- Site's own JS files
- Images (organized by use)
- Self-hosted fonts (`.woff2`, `.woff`, `.otf`, `.ttf`)
- Font CDN files (Google Fonts, Bunny Fonts, etc.)

**Skipped (referenced in README only):**
- Analytics & tracking (GTM, GA4, Segment, Mixpanel)
- Chat widgets (Crisp, Intercom, Drift, Zendesk)
- A/B testing (Optimizely, VWO)
- Video players (YouTube embeds, Vimeo, Wistia)
- Cloudflare protection scripts
- Cookie consent

## Design tokens structure

The generated `analysis/design-tokens.json` follows this schema:

```json
{
  "_meta": { "source", "captured_at", "method", "purpose" },
  "brand": { "name", "tagline", "tone", "stack_observada" },
  "colors": { "primary": {}, "dark": {}, "neutral": {}, "borders": {} },
  "typography": { "primary_display": {}, "secondary_body": {}, "scale_observada": {} },
  "spacing": { "unidade", "max_width", "section_padding_*", "gap_componentes" },
  "radii": {},
  "shadows": {},
  "transitions": {},
  "animations": {},
  "breakpoints": {},
  "naming_convention": {}
}
```

Colors are grouped by **semantic role** (not raw value), tokens have **semantic names** (`neon_green`, `deep_teal`, `text_muted`), and animations distinguish CSS `@keyframes` from JS-driven (IntersectionObserver, requestAnimationFrame).

## Visual guide structure

The generated `analysis/visual-identity-guide.md` covers, in order:

1. Positioning & tone
2. Color palette (grouped by role)
3. Typography (families + scale)
4. Spacing & grid
5. Component tokens & patterns (buttons, badges, cards, nav, footer, modals, accordions)
6. Animations & micro-interactions
7. Page structure (top-to-bottom)
8. CSS naming convention
9. Stack & integrations observed
10. File inventory
11. Rebranding direction (actionable cheat-sheet)

## Edge cases handled

- **SPAs (React/Vue/Angular)** — waits for hydration before extraction
- **Cloudflare protection** — handles email obfuscation, challenge pages
- **CSS Custom Properties** — extracts `:root` variables with resolved values
- **Dark mode** — captures default theme, notes toggle if present
- **JS-heavy rendering** — falls back to `outerHTML` if static HTML is empty
- **Long pages** — captures section screenshots in addition to full-page

## Example output

See [the OnProfit reference snapshot](https://github.com/eduardodotai/site-identity-snapshot/tree/main/examples) (coming soon) for what a complete output looks like — built from `https://onprofit.com.br/`, 233-line visual guide, 186-line design-tokens.json.

## License

MIT
