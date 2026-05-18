# Visual Identity Guide Template

Reference template for `analysis/visual-identity-guide.md`. Adapt section depth to what the site actually offers — skip sections that don't apply, expand sections with rich content.

---

## Required Sections (in order)

### 1. Header Block
```markdown
# {Brand Name} – Guia de Identidade Visual (referência para {purpose})

> Snapshot capturado em **{YYYY-MM-DD}** a partir de {URL}.
> Esta pasta espelha a {page description}: HTML, CSS, JS, imagens, fontes e screenshots.
> Use como **fonte de verdade visual** durante {context}.
```

### 2. Posicionamento e tom
- Product name and category
- Core promise / tagline
- Key vocabulary and brand language
- Personality descriptors (e.g., "tech / fintech / dark-elegante / acento neon")
- Recurring CTAs (text + style)

### 3. Paleta de cores
Use tables grouped by role:

```markdown
### Primárias
| Token | Hex | Uso |
| --- | --- | --- |
| **Token Name** | `#HEX` | Where used (CSS class, component) |

### Escuras (background e texto)
| Token | Hex | Uso |
...

### Neutras
| Token | Hex | Uso |
...
```

Note important background images/gradients at the end.

### 4. Tipografia
Two tables:

**Families table:**
| Família | Onde | Pesos | Origem |
| --- | --- | --- | --- |

**Scale table:**
| Nível | Tamanho | Família | Onde |
| --- | --- | --- | --- |

Cover: h1 through caption, buttons, badges, input text.

### 5. Espaçamentos e grid
- Base unit (rem/px)
- Container max-width + breakpoint fallbacks
- Section padding (vertical + horizontal)
- Common gaps between components
- Dominant layout pattern (e.g., "50/50 alternating bg white/grey")

### 6. Componentes — tokens e padrões
For each recurring component, provide:
- CSS snippet showing key properties
- Behavioral notes (hover, active, responsive)

Common components to look for:
- Buttons (primary, secondary, ghost)
- Badges / pills
- Cards (pricing, feature, testimonial)
- Navigation bar (fixed, scroll behavior, mobile)
- Footer
- Modal / dialog
- Accordion / FAQ
- Form inputs
- Carousel / slider

### 7. Animações e micro-interações
| Nome | Tipo | Tempo / curva | Onde |
| --- | --- | --- | --- |

Distinguish: CSS @keyframes vs JS (IntersectionObserver, rAF, GSAP, Framer Motion).
Note dominant easing and duration patterns.

### 8. Estrutura da página
| # | Seção (classe) | Conteúdo | Altura desktop |
| - | --- | --- | --- |

Ordered top-to-bottom. Include CSS class, brief content description, approximate desktop height.

### 9. Convenção de naming CSS
- Prefix used
- Methodology (BEM, utility-first, custom)
- Example classes
- Utility patterns observed

### 10. Stack & integrações
- Backend/framework detected
- CDN
- Analytics/tags (referenced, not downloaded)
- Third-party services
- Font sources
- Icon library

### 11. Inventário de arquivos baixados
| Pasta | Conteúdo |
| --- | --- |
| `html/index.html` | description |
| `css/*.css` | description |
| ... | ... |

### 12. Direção sugerida pro rebranding (cheat-sheet)
Actionable list of 5-8 items answering:
- What defines this brand's visual signature?
- What to preserve vs. consciously break?
- Substitutes for key typefaces with similar vibes
- Animation cadence to maintain or change
- Layout patterns that are easy to port

---

## Tone
- Write in the language of the site (Portuguese for BR sites, English for EN sites)
- Be specific and evidence-based (reference CSS classes, hex values, pixel sizes)
- Include CSS snippets for component patterns — don't just describe, show
- Be opinionated in the rebranding section — give actionable direction
