# Design Tokens JSON Schema

Reference schema for `analysis/design-tokens.json`. Every snapshot must produce a JSON file matching this structure.

## Full Schema

```json
{
  "_meta": {
    "source": "https://example.com (page description)",
    "captured_at": "YYYY-MM-DD",
    "method": "Live DOM computed styles + <filename>.css static analysis",
    "purpose": "Referência de identidade visual para <purpose>"
  },

  "brand": {
    "name": "Brand Name",
    "tagline_pt": "Tagline in Portuguese (if applicable)",
    "tagline_en": "Tagline in English (if applicable)",
    "tone": "descriptors / separated / by / slash",
    "stack_observada": "Framework + libraries observed in source"
  },

  "colors": {
    "primary": {
      "token_name": "#HEX or rgba()",
      "token_name_hover": "#HEX",
      "token_name_alpha_variant": "rgba(r, g, b, alpha)"
    },
    "dark": {
      "background_name": "#HEX",
      "overlay_variant": "rgba(r, g, b, alpha)"
    },
    "neutral": {
      "white": "#FFFFFF",
      "off_white": "#F5F5F5",
      "text_variant": "#HEX"
    },
    "semantic": {
      "success": "#HEX",
      "error": "#HEX",
      "warning": "#HEX",
      "info": "#HEX"
    },
    "borders": {
      "descriptive_name": "rgba() or #HEX"
    }
  },

  "typography": {
    "primary_display": {
      "family": "Font Name",
      "weights_used": ["Light", "Regular (400)", "Medium (500)", "Bold (700)"],
      "format": "OTF/WOFF2/TTF self-hosted | Google Fonts | CDN",
      "letter_spacing": "value or normal",
      "uso": "Where used (titles, badges, numbers)"
    },
    "secondary_body": {
      "family": "Font Name",
      "fallback": "sans-serif | serif | monospace",
      "weights_used": [400, 500, 600, 700],
      "source": "Google Fonts | Bunny Fonts | Adobe Fonts | self-hosted",
      "uso": "Where used (body text, buttons, paragraphs)"
    },
    "icons": {
      "library": "Library name + version",
      "cdn": "CDN domain"
    },
    "scale_observada": {
      "h1_hero": "size / family weight / line-height",
      "h2_section_title": "size / family weight / line-height",
      "h3_card_title": "size / family weight",
      "subtitle": "size / family weight",
      "body_lg": "size / family",
      "body": "size / family weight / line-height",
      "body_sm": "size / family",
      "badge": "size / family weight / letter-spacing",
      "button": "size / family weight",
      "caption": "size / family weight"
    }
  },

  "spacing": {
    "unidade": "rem | px | em (base value)",
    "section_padding_y_desktop": "values observed",
    "section_padding_x_desktop": "values observed",
    "max_width": "value (class name) | breakpoint fallbacks",
    "card_padding": "value",
    "button_padding": "value / height",
    "gap_componentes": "common values"
  },

  "radii": {
    "component_name": "value (description)"
  },

  "shadows": {
    "component_shadow_name": "full box-shadow value"
  },

  "gradients_e_imagens_de_fundo": {
    "descriptive_name": "gradient or url() value"
  },

  "transitions": {
    "descriptive_name": "property duration easing"
  },

  "animations": {
    "animation_name": {
      "tipo": "@keyframes | JS (requestAnimationFrame | IntersectionObserver)",
      "tempo": "duration timing-function iteration",
      "uso": "selector or component",
      "description": "what it does"
    }
  },

  "breakpoints": {
    "name": "media query value"
  },

  "naming_convention": {
    "prefixo": "prefix used (e.g., on-, tw-, etc.)",
    "estilo": "BEM | utility-first | custom description",
    "exemplos": ["class examples"],
    "utilities_observadas": ["utility class examples"]
  }
}
```

## Rules

1. **Colors by role, not value** — group as primary/dark/neutral/semantic/borders
2. **Semantic token names** — `neon_green`, `deep_teal`, `text_muted` (not `color_1`, `#50FFB1`)
3. **Include variants** — hover, glow, alpha as sub-tokens of parent
4. **Document usage** — every token should note which CSS class or component uses it
5. **Typography source** — always note origin (self-hosted, Google, Bunny, CDN)
6. **Animation type** — distinguish CSS @keyframes from JS-driven
7. **Breakpoints sorted** — smallest to largest
8. **Stack inference** — note framework, libraries, build tool observations
