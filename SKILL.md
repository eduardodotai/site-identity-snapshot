---
name: site-identity-snapshot
description: Captures complete visual identity of any public website into a structured reference folder — HTML, CSS, JS, images, fonts, screenshots, design tokens (JSON), and a human-readable visual identity guide. Use for rebranding, competitive analysis, or design system extraction. Requires Chrome DevTools MCP.
license: MIT
user-invocable: true
metadata:
  version: 1.0.0
  author: Eduardo Santos
  domains: [design, scraping, visual-identity, design-tokens, rebranding]
---

# Site Identity Snapshot

Captures the complete visual identity of any public website into a structured reference folder.

**Output:** ready-to-consume design tokens (JSON) + human-readable visual guide + all assets + screenshots.
**Scope:** capture-only — does NOT implement anything in the target project.

---

## Quick Start

```
User: snapshot do site https://example.com
User: captura a identidade visual de https://competitor.com.br
User: extrair design tokens de https://startup.io para referência
User: referência visual de https://brand.com para o rebranding
User: design scraper de https://landing.page
```

---

## Triggers

Use this skill when the user asks to:
- "snapshot do site X", "site snapshot of X"
- "captura a identidade visual de X", "capture visual identity"
- "extrair design tokens de X", "extract design tokens"
- "referência visual de X", "visual reference for rebranding"
- "scraper de design de X", "design scraper"
- "clonar visual de X para reference"
- "analisar identidade visual de X"
- "baixar assets de X para referência"

---

## Quick Reference

| Input | Output |
|-------|--------|
| URL do site (obrigatório) | Pasta estruturada com todos os artefatos |
| Pasta destino (opcional, default: `reference/<domínio>/`) | README.md + analysis/ + html/ + css/ + js/ + images/ + fonts/ + screenshots/ |

| Dependency | Required |
|------------|----------|
| Chrome DevTools MCP (`chrome-devtools` plugin) | **Yes** — screenshots + live token extraction |
| `curl` | **Yes** — static asset download |

---

## Output Structure

```
<DEST>/
├── README.md                       ← snapshot documentation + how to use
├── analysis/
│   ├── visual-identity-guide.md    ← human-readable guide (brand, palette, type, animations, components)
│   └── design-tokens.json          ← structured tokens (colors, type, radii, shadows, spacing, breakpoints, motion)
├── html/
│   └── index.html                  ← main page markup
├── css/                            ← all linked stylesheets (preserve original names)
├── js/                             ← site's own UI scripts only (NO analytics/chat/tracking)
├── images/                         ← organized by use (landing/, graphics/, icons/, etc.)
├── fonts/                          ← @font-face self-hosted files, grouped by family
└── screenshots/
    ├── desktop-fullpage.png        ← 1440px width, full page
    ├── desktop-hero.png            ← 1440x900 viewport
    ├── mobile-fullpage.png         ← 390px width, full page
    ├── mobile-hero.png             ← 390x844 viewport
    └── section-*.png               ← key individual sections (hero, footer, distinctive blocks)
```

---

## How It Works

```
URL ──► Phase 1: Recon ──► Phase 2: Download ──► Phase 3: Screenshots ──► Phase 4: Extract Tokens ──► Phase 5: Document
         │                   │                    │                        │                          │
         ▼                   ▼                    ▼                        ▼                          ▼
    Discover assets     curl all assets     Chrome DevTools MCP      evaluate_script on         Generate:
    Map CSS/JS/fonts    Organize by type    Desktop + mobile         live DOM computed          - design-tokens.json
    Identify CDNs       Resolve rel URLs    Full-page + viewport     styles + keyframes         - visual-identity-guide.md
    Detect 3rd-party    Preserve names      Key sections                                        - README.md
```

---

## Phase 1: Reconnaissance

**Goal:** Understand the site's asset structure before downloading anything.

1. **Open the site in Chrome DevTools MCP:**
   ```
   new_page → navigate_page(url) → wait for full load
   ```

2. **Extract the asset map** via `evaluate_script`:
   ```javascript
   (() => {
     const assets = {
       stylesheets: [...document.querySelectorAll('link[rel="stylesheet"]')].map(l => l.href),
       scripts: [...document.querySelectorAll('script[src]')].map(s => s.src),
       images: [...new Set([...document.querySelectorAll('img')].map(i => i.src).filter(Boolean))],
       fonts_css: [...document.querySelectorAll('link[rel="stylesheet"]')]
         .map(l => l.href).filter(h => /font|figtree|bunny|google/i.test(h)),
       meta: {
         title: document.title,
         description: document.querySelector('meta[name="description"]')?.content,
         generator: document.querySelector('meta[name="generator"]')?.content,
         charset: document.characterSet
       }
     };
     return JSON.stringify(assets, null, 2);
   })()
   ```

3. **Classify assets** into:
   - **Own assets** (same domain or asset CDN) → download
   - **Font CDNs** (Google Fonts, Bunny Fonts, Adobe Fonts) → download CSS + font files
   - **Icon libraries** (Font Awesome, Material Icons) → download minified CSS
   - **Third-party scripts** (GTM, Analytics, Crisp, Intercom, Hotjar, AB testing) → **DO NOT download**, note in README
   - **Embedded players** (YouTube, Vimeo, Converte AI, Wistia) → **DO NOT download**, note in README

4. **Detect @font-face declarations** via `evaluate_script`:
   ```javascript
   (() => {
     const fonts = [];
     for (const sheet of document.styleSheets) {
       try {
         for (const rule of sheet.cssRules) {
           if (rule instanceof CSSFontFaceRule) {
             fonts.push({
               family: rule.style.fontFamily,
               weight: rule.style.fontWeight,
               style: rule.style.fontStyle,
               src: rule.style.src
             });
           }
         }
       } catch(e) { /* cross-origin sheet, skip */ }
     }
     return JSON.stringify(fonts, null, 2);
   })()
   ```

---

## Phase 2: Asset Download

**Goal:** Download all own assets, organized by type.

### 2.1 Create directory structure
```bash
mkdir -p <DEST>/{html,css,js,images,fonts,screenshots,analysis}
```

### 2.2 Download with curl

**HTML:**
```bash
curl -sL "<URL>" -o <DEST>/html/index.html
```

**CSS files** (each linked stylesheet that's own/font):
```bash
curl -sL "<CSS_URL>" -o <DEST>/css/<original-filename>.css
```

**JS files** (own scripts ONLY — skip GTM, analytics, chat widgets):
```bash
curl -sL "<JS_URL>" -o <DEST>/js/<original-filename>.js
```

**Images** — organize by detected use:
```bash
# landing page images → images/landing/
# logos/brand graphics → images/graphics/
# favicons → images/
# partner/integration logos → images/integrations/ or images/partners/
curl -sL "<IMG_URL>" -o <DEST>/images/<subdir>/<filename>
```

**Fonts** — group by family:
```bash
# e.g., fonts/inter/, fonts/roboto/, fonts/custom-display/
curl -sL "<FONT_URL>" -o <DEST>/fonts/<family>/<filename>
```

### 2.3 Resolution rules
- **Relative URLs**: resolve against the page's base URL
- **Protocol-relative** (`//`): prefix with `https:`
- **Preserve original filenames** wherever possible
- **Deduplicate**: same file referenced multiple times → download once
- **SVG inline**: if SVGs are inlined in HTML, they're already in `html/index.html` — no separate download needed unless also referenced as `<img src="...">`

### 2.4 What NOT to download (reference in README only)
| Category | Examples | Why skip |
|----------|----------|----------|
| Analytics/tags | GTM, GA4, Segment, Mixpanel | Privacy + irrelevant to design |
| Chat widgets | Crisp, Intercom, Drift, Zendesk | Dynamic, auth-gated |
| A/B testing | Optimizely, VWO, Google Optimize | Dynamic, ephemeral |
| Video players | YouTube embeds, Vimeo, Wistia, Converte AI | Heavy, auth/CDN-gated |
| CDN protection | Cloudflare challenge scripts, email obfuscation | Not design-related |
| Cookie consent | OneTrust, Cookiebot | Not design-related |

---

## Phase 3: Screenshots

**Goal:** Visual reference at key viewports and sections.

Use Chrome DevTools MCP:

### 3.1 Desktop full-page
```
resize_page(width=1440, height=900)
take_screenshot(fullPage=true) → screenshots/desktop-fullpage.png
```

### 3.2 Desktop hero (viewport only)
```
take_screenshot(fullPage=false) → screenshots/desktop-hero.png
```

### 3.3 Mobile full-page
```
resize_page(width=390, height=844)
take_screenshot(fullPage=true) → screenshots/mobile-fullpage.png
```

### 3.4 Mobile hero (viewport only)
```
take_screenshot(fullPage=false) → screenshots/mobile-hero.png
```

### 3.5 Key sections
Identify visually distinctive sections (hero, footer, pricing, features, testimonials) and capture each:
```
evaluate_script: document.querySelector('<section-selector>').scrollIntoView()
take_screenshot(fullPage=false) → screenshots/section-<name>.png
```

**Minimum screenshots:** desktop-fullpage, desktop-hero, mobile-fullpage, mobile-hero, section-hero, section-footer.
**Additional:** any section with distinctive visual treatment (gradients, dark/light alternation, cards, carousels).

---

## Phase 4: Extract Design Tokens

**Goal:** Extract live computed styles from the DOM into structured JSON.

### 4.1 Color extraction
```javascript
(() => {
  const elements = document.querySelectorAll('*');
  const colors = new Set();
  const colorProps = ['color', 'background-color', 'border-color', 'outline-color', 'box-shadow'];
  elements.forEach(el => {
    const cs = getComputedStyle(el);
    colorProps.forEach(prop => {
      const val = cs.getPropertyValue(prop);
      if (val && val !== 'rgba(0, 0, 0, 0)' && val !== 'transparent') {
        colors.add(val);
      }
    });
  });
  return JSON.stringify([...colors].sort(), null, 2);
})()
```

### 4.2 Typography extraction
```javascript
(() => {
  const elements = document.querySelectorAll('h1,h2,h3,h4,h5,h6,p,a,button,span,li,label,input,textarea,th,td');
  const typeMap = {};
  elements.forEach(el => {
    const cs = getComputedStyle(el);
    const key = `${cs.fontFamily}|${cs.fontSize}|${cs.fontWeight}|${cs.lineHeight}|${cs.letterSpacing}`;
    if (!typeMap[key]) {
      typeMap[key] = {
        family: cs.fontFamily,
        size: cs.fontSize,
        weight: cs.fontWeight,
        lineHeight: cs.lineHeight,
        letterSpacing: cs.letterSpacing,
        tag: el.tagName.toLowerCase(),
        className: el.className?.toString().slice(0, 80) || '',
        count: 0
      };
    }
    typeMap[key].count++;
  });
  return JSON.stringify(Object.values(typeMap).sort((a,b) => parseInt(b.size) - parseInt(a.size)), null, 2);
})()
```

### 4.3 Spacing, radii, shadows
```javascript
(() => {
  const elements = document.querySelectorAll('[class]');
  const data = { radii: new Set(), shadows: new Set(), paddings: new Set(), margins: new Set() };
  elements.forEach(el => {
    const cs = getComputedStyle(el);
    const br = cs.borderRadius;
    if (br && br !== '0px') data.radii.add(br);
    const bs = cs.boxShadow;
    if (bs && bs !== 'none') data.shadows.add(bs);
    const p = cs.padding;
    if (p && p !== '0px') data.paddings.add(p);
  });
  return JSON.stringify({
    radii: [...data.radii].sort(),
    shadows: [...data.shadows],
    common_paddings: [...data.paddings].sort()
  }, null, 2);
})()
```

### 4.4 Keyframes and animations
```javascript
(() => {
  const keyframes = [];
  const transitions = new Set();
  for (const sheet of document.styleSheets) {
    try {
      for (const rule of sheet.cssRules) {
        if (rule instanceof CSSKeyframesRule) {
          const frames = [];
          for (const kf of rule.cssRules) {
            frames.push({ key: kf.keyText, style: kf.style.cssText });
          }
          keyframes.push({ name: rule.name, frames });
        }
      }
    } catch(e) {}
  }
  document.querySelectorAll('[class]').forEach(el => {
    const t = getComputedStyle(el).transition;
    if (t && t !== 'all 0s ease 0s' && t !== 'none 0s ease 0s') transitions.add(t);
  });
  return JSON.stringify({ keyframes, transitions: [...transitions] }, null, 2);
})()
```

### 4.5 Breakpoints (from CSS)
```javascript
(() => {
  const breakpoints = new Set();
  for (const sheet of document.styleSheets) {
    try {
      for (const rule of sheet.cssRules) {
        if (rule instanceof CSSMediaRule) {
          breakpoints.add(rule.conditionText);
        }
      }
    } catch(e) {}
  }
  return JSON.stringify([...breakpoints].sort(), null, 2);
})()
```

### 4.6 Assemble design-tokens.json

Combine all extracted data into a structured JSON following this schema. See [references/design-tokens-schema.md](references/design-tokens-schema.md) for the full template.

**Required top-level keys:**
```json
{
  "_meta": { "source", "captured_at", "method", "purpose" },
  "brand": { "name", "tagline", "tone", "stack_observada" },
  "colors": { "primary": {}, "dark": {}, "neutral": {}, "borders": {} },
  "typography": { "primary_display": {}, "secondary_body": {}, "icons": {}, "scale_observada": {} },
  "spacing": { "unidade", "section_padding_*", "max_width", "card_padding", "button_padding", "gap_componentes" },
  "radii": {},
  "shadows": {},
  "gradients_e_imagens_de_fundo": {},
  "transitions": {},
  "animations": {},
  "breakpoints": {},
  "naming_convention": {}
}
```

**Critical rules for token assembly:**
- Group colors **by semantic role** (primary, dark/background, neutral, border), not by raw value
- Name tokens **semantically** (`neon_green`, `deep_teal`, `text_muted`), not by hex
- Include **hover/glow/alpha variants** as sub-tokens of their parent
- Document the **usage** of each token (which CSS class or component uses it)
- For typography, always note the **source** (self-hosted, Google Fonts, CDN)
- For animations, distinguish **CSS @keyframes** from **JS-driven** (requestAnimationFrame, IntersectionObserver)

---

## Phase 5: Documentation

**Goal:** Generate the three documentation artifacts.

### 5.1 visual-identity-guide.md

Human-readable narrative covering (in this order):

1. **Posicionamento e tom** — product, core promise, key vocabulary, personality, recurring CTAs
2. **Paleta de cores** — tables with Token | Hex | Usage for each group (primary, dark, neutral)
3. **Tipografia** — families table (where, weights, source) + observed scale table (level, size, family, context)
4. **Espaçamentos e grid** — unit, max-width, padding patterns, common gaps, dominant layout pattern
5. **Componentes — tokens e padrões** — for each recurring component (buttons, badges, cards, nav, footer, modals, accordions): CSS snippet + behavioral notes
6. **Animações e micro-interações** — table of all animations (name, type, timing, where used)
7. **Estrutura da página** — ordered table of all page sections (class, content, approximate height)
8. **Convenção de naming CSS** — prefix, BEM style, utility pattern
9. **Stack & integrações** — framework, CDN, analytics, third-party services observed
10. **Inventário de arquivos baixados** — table of all downloaded files by folder
11. **Direção sugerida pro rebranding** — actionable cheat-sheet: what defines the brand's visual signature, what to preserve vs. break

See [references/visual-guide-template.md](references/visual-guide-template.md) for the full template.

### 5.2 design-tokens.json

Assembled in Phase 4. Review and refine:
- Ensure all colors have semantic names
- Ensure typography scale is complete (h1 through caption)
- Ensure breakpoints are sorted
- Add any missing gradients or background patterns

### 5.3 README.md

Must answer:
1. **What** — what is this snapshot, date of capture, source URL
2. **Structure** — folder tree (use markdown code block)
3. **How it was made** — brief methodology (curl + Chrome DevTools MCP)
4. **How to use as reference** — step-by-step (start with guide → tokens → screenshots as visual diff → HTML/CSS for markup copying)
5. **Important notes** — uncaptured external assets, CSS peculiarities (monolithic, no vars, no build step, etc.), obfuscation, anything unusual

---

## Anti-Patterns

| Avoid | Why | Instead |
|-------|-----|---------|
| Downloading third-party scripts | Privacy, irrelevant, bloat | Reference in README |
| Flat image dump | Hard to navigate | Organize by use (landing/, graphics/, icons/) |
| Raw hex values as token names | Not semantic, hard to use | Name by role (`primary_accent`, `text_muted`) |
| Skipping mobile screenshots | Mobile-first is standard | Always capture 390px + 844px |
| Inlining templates in output | Inconsistent quality | Follow the reference templates |
| Running site JS in extraction | May trigger analytics, CORS errors | Extract from static CSS + computed DOM |
| Implementing in target project | Out of scope, project-specific | Stop at reference artifacts |

---

## Verification

After completion, verify:

- [ ] All 7 top-level folders exist (`html/`, `css/`, `js/`, `images/`, `fonts/`, `screenshots/`, `analysis/`)
- [ ] `analysis/design-tokens.json` has all required top-level keys (`_meta`, `brand`, `colors`, `typography`, `spacing`, `radii`, `shadows`, `transitions`, `animations`, `breakpoints`)
- [ ] `analysis/visual-identity-guide.md` covers all 11 sections
- [ ] `README.md` has folder tree + methodology + how-to-use
- [ ] At least 6 screenshots: desktop-fullpage, desktop-hero, mobile-fullpage, mobile-hero, section-hero, section-footer
- [ ] No analytics/tracking scripts in `js/`
- [ ] Images organized in subdirectories by use
- [ ] Fonts grouped by family
- [ ] All color tokens have semantic names (not raw hex)
- [ ] Typography scale is complete (h1 through caption/small)

Report the final folder tree and a summary of what was captured.

---

<details>
<summary><strong>Deep Dive: Handling Edge Cases</strong></summary>

### Single Page Apps (React/Vue/Angular)
- Wait for hydration before extracting (`evaluate_script` with setTimeout or MutationObserver)
- May need to navigate to multiple routes to capture all styles
- Check for CSS-in-JS (styled-components, emotion) — styles may be in `<style>` tags, not `.css` files

### Sites with Cloudflare Protection
- Email obfuscation: note in README, decoded emails are not critical for design
- Challenge pages: if blocked, try with different user-agent via curl
- Rate limiting: add small delays between curl requests

### Sites with Heavy JS Rendering
- If `curl` gets an empty shell, the HTML in `html/index.html` will be minimal
- Use `evaluate_script` to get `document.documentElement.outerHTML` as the rendered HTML instead
- Note in README that the HTML is post-render, not source

### Dark Mode / Theme Variants
- Capture the default theme first
- If the site has a theme toggle, note it in the guide
- Optionally capture both themes in separate screenshot sets

### Very Long Pages
- Full-page screenshots may be very large (>10MB)
- Chrome DevTools MCP handles this, but note the file size
- Section screenshots become more important for quick reference

### CSS Custom Properties (Variables)
- Extract with: `getComputedStyle(document.documentElement)` for `:root` vars
- Include raw variable names AND their resolved values in tokens
- Note the variable naming convention in the guide

### Multi-Page Sites
- Default: capture the landing/home page only
- If the user specifies additional pages, repeat Phases 1-4 for each
- Use `html/<page-name>.html` naming and note in README

</details>

<details>
<summary><strong>Deep Dive: CSS Variables Extraction</strong></summary>

If the site uses CSS custom properties, extract them:

```javascript
(() => {
  const root = getComputedStyle(document.documentElement);
  const vars = {};
  for (const sheet of document.styleSheets) {
    try {
      for (const rule of sheet.cssRules) {
        if (rule.selectorText === ':root' || rule.selectorText === 'html') {
          for (const prop of rule.style) {
            if (prop.startsWith('--')) {
              vars[prop] = root.getPropertyValue(prop).trim();
            }
          }
        }
      }
    } catch(e) {}
  }
  return JSON.stringify(vars, null, 2);
})()
```

Include these in `design-tokens.json` under a `css_variables` key, preserving original variable names.

</details>
