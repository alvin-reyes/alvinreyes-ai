# alvinreyes.ai — v1 build spec

**Date:** 2026-05-21
**Owner:** Alvin Reyes
**Status:** Approved, ready for implementation plan

## 1. Goal

Recreate the warm-dark editorial design in `design_handoff_alvinreyes_ai/references/Personal Site.html` as a static Astro site, deploy it to Vercel at `alvinreyes.ai`, and enable Vercel Analytics. The visual design is fully locked by the handoff README — this spec covers engineering execution only.

## 2. Scope

**In scope (v1):**
- Static single-page site, pixel-accurate to `Personal Site.html`
- 10 content sections + sticky top bar + footer (per handoff §5)
- Design tokens as CSS custom properties, exact values from handoff §4
- Spotlight case-study layout (only — drop cards/list variants)
- Per-section Astro components for maintainability
- SEO: title, meta description, canonical, OpenGraph, Twitter card, JSON-LD (Person + Organization), `robots.txt`, sitemap
- Vercel Analytics
- Vercel deploy config
- Favicon (SVG + 32×32 PNG fallback)

**Out of scope (v1):**
- Blog, RSS, contact form, newsletter, light/dark toggle, CMS, embedded Calendly
- Tweaks panel from the prototype (dev tool only)
- Cards / list case-study layouts (only spotlight ships)
- Real case-study screenshots — placeholder backgrounds ship; real images come later
- OG image asset — placeholder URL in meta tags, real 1200×630 PNG to be added before launch (open item, see §8)

**Decisions made during brainstorming:**
- Stack: Astro, latest stable major (handoff recommended Astro 4; install the current latest at scaffold time)
- Direction: warm dark editorial (`Personal Site.html`, the handoff's primary)
- Copy: placeholder copy from the handoff verbatim — including Koneksi/MetaDev placeholders, approximate Arc dates, four placeholder case studies, `alvin@ardata.tech` email
- Hosting: Vercel
- Analytics: Vercel Analytics
- Component structure: per-section split (not single flat `index.astro`)
- Repo location: Astro project at the root of `/Users/alvin-reyes/Project/personal-site`, alongside the existing design handoff folder

## 3. Repository layout

```
personal-site/
├── astro.config.mjs
├── package.json
├── tsconfig.json
├── .gitignore                           # design_handoff_alvinreyes_ai/, dist/, node_modules/, .env, .DS_Store, .vercel/
├── public/
│   ├── favicon.svg                      # burnt-orange circle + "AR"
│   ├── favicon-32.png                   # fallback
│   ├── robots.txt
│   └── og-image.png                     # TODO before launch (1200×630)
├── src/
│   ├── pages/
│   │   └── index.astro                  # composes all section components
│   ├── layouts/
│   │   └── Base.astro                   # <html>, <head>, fonts, meta, JSON-LD, analytics, global CSS
│   ├── styles/
│   │   ├── tokens.css                   # :root CSS custom properties
│   │   └── global.css                   # resets, base typography, body bg, .reveal, @media prefers-reduced-motion
│   └── components/
│       ├── TopBar.astro
│       ├── Hero.astro
│       ├── About.astro
│       ├── Holdings.astro
│       ├── Arc.astro
│       ├── Workflow.astro
│       ├── WATED.astro
│       ├── BMAD.astro
│       ├── Principles.astro
│       ├── Work.astro                   # spotlight layout only
│       ├── Contact.astro
│       └── Footer.astro
├── docs/
│   └── superpowers/
│       └── specs/
│           └── 2026-05-21-alvinreyes-ai-site-design.md  # this file (tracked)
└── design_handoff_alvinreyes_ai/        # gitignored, kept on disk for reference
```

The design handoff folder stays on disk but is excluded from git and from the build output so it does not ship.

## 4. Design tokens (verbatim from handoff §4)

Defined once on `:root` in `src/styles/tokens.css`. All component CSS reads via `var(--token)`.

**Colors:**
- `--bg: #1c1917`
- `--paper: #24201c`
- `--paper-2: #2c2722`
- `--ink: #ede1c8`
- `--ink-soft: #c9b896`
- `--muted: #8a7a60`
- `--rule: #38312a`
- `--accent: #d97757`
- `--accent-soft: #e8a07d`
- `--ok: #7fc99d`

**Typography (Google Fonts, single `<link>`):**
- Instrument Serif 400 / 400 italic — display, headlines, quotes
- Geist 300 / 400 / 500 / 600 — body, UI
- JetBrains Mono 400 / 500 — labels, eyebrows, mono accents

Type scale, line-heights, and letter-spacings: exactly per handoff §4.2. `text-wrap: balance` on headlines, `text-wrap: pretty` on body.

**Layout:**
- `--col: 1180px`
- `--pad-x: clamp(20px, 5vw, 64px)`
- `--row-gap: 132px` default

**Radius:** 4px cards, 6px contact panel, 8px notable cards, 999px pills. No drop shadows.

**Motion:** `.15s ease` on hover (bg/color/border), `.2s ease` on transforms. Hero entrance with staggered `.d1`–`.d4` delays. Signature 14s drift. Status pulse 2.4s. All gated on `prefers-reduced-motion: reduce`.

## 5. Section components

Each component owns its markup and scoped `<style>`. All sections wrapped in `<section id="…" class="section">` with shared section padding from `global.css`. Last section drops the bottom border.

| Component | Anchor | Source in reference | Notes |
| --- | --- | --- | --- |
| `TopBar.astro` | — | `.topbar` | Sticky, backdrop-blur, status pulse on `.ok` dot |
| `Hero.astro` | `#top` | hero block | H1 with `<span class="ital">` on "ship.", signature `.sig` absolute circle with drift |
| `About.astro` | `#about` | About — "The AR is me" | 2-col grid 1fr 1.4fr, collapses <820px |
| `Holdings.astro` | `#holdings` | The Group | Parent card + trunk + 5 branches (3×2 grid); JV uses diagonal-stripe bg |
| `Arc.astro` | `#arc` | The Arc | 7-row table, `<span class="pin">●</span>` on current role |
| `Workflow.astro` | `#workflow` | The Loop | 12-col grid with arrow rows; `.accent` modifier on EXECUTE |
| `WATED.astro` | `#wated` | Five Pillars | 5-col grid of pillars, giant accent letters |
| `BMAD.astro` | `#bmad` | The Agent Crew | 2-col intro + 6-col agent grid |
| `Principles.astro` | `#principles` | Principles | 3×2 grid, item 5 spans full width via `.wide` |
| `Work.astro` | `#work` | Selected Work | Spotlight layout only: first card 2-col (image left, body right), items 2-4 compact rows |
| `Contact.astro` | `#contact` | Contact | Paper panel, Calendly CTA + vertical contact list |
| `Footer.astro` | — | Footer | Top border, AR mini-mark, copyright + colophon |

Copy (eyebrows, headlines, paragraphs, table rows, pillar text, agent text, principle blockquotes, case-study descriptions) is taken verbatim from the handoff README §6 sub-sections. Email is `alvin@ardata.tech`. Calendly URL is `https://calendly.com/alvin-ardata/discovery-ai-partnership`.

## 6. Responsive

Fluid-by-default via `clamp()` on type and padding. Discrete breakpoints from handoff §8 implemented per component:

- ≤1100px: Holdings 5→3 cols
- ≤980px: BMAD 6→3 cols, WATED 5→2 cols
- ≤820px: About 2→1 col, Arc rows stack, Workflow cells full width, Principles full width
- ≤760px: Holdings 3→2 cols, BMAD intro 1 col, Contact panel 1 col
- ≤600px: Contact lines 1 col
- ≤540px: Holdings, WATED, BMAD all 1 col

## 7. SEO and metadata

All in `src/layouts/Base.astro`.

- `<title>`: "Alvin Reyes — Production AI & Agentic Workflows | AR Data"
- `<meta name="description">`: ~155 chars, derived from the hero subhead
- `<link rel="canonical" href="https://alvinreyes.ai/">`
- `<html lang="en">`, viewport meta (Astro default)
- OpenGraph: `og:title`, `og:description`, `og:url`, `og:type=website`, `og:image=https://alvinreyes.ai/og-image.png`
- Twitter: `twitter:card=summary_large_image`, `twitter:creator=@alvinjayreyes`
- JSON-LD: a single `<script type="application/ld+json">` containing a `Person` for Alvin (`sameAs` to Twitter, GitHub, LinkedIn; `jobTitle`, `worksFor`) and an `Organization` for AR Data Intelligence Solutions (`parentOrganization` ARCE Holdings Inc.)
- Favicon: `favicon.svg` + `favicon-32.png`
- `public/robots.txt`: allow all
- Sitemap via `@astrojs/sitemap` integration (single URL, but signals intent)

Fonts: `<link rel="preconnect" href="https://fonts.googleapis.com">` and `<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>` before the Google Fonts CSS link. `font-display: swap` is in the Google Fonts URL by default.

## 8. Open items (do not block v1 build)

These ship as the placeholders specified, to be resolved before public launch:

1. **OG image** (`public/og-image.png`, 1200×630) — needs a real asset; Alvin to review
2. **Koneksi / MetaDev** scope, copy, and domains — currently placeholder per handoff §11
3. **Operating company domains** — verify `ardata.tech`, `caiden.ai`, `selfvision.ai`, `koneksi.*`, `metadev.*`
4. **Arc dates and cities** — approximate per handoff §11
5. **Case studies (CS-01 through CS-04)** — all placeholders with plausible metrics; replace before claiming any of them are real engagements
6. **Email** — `alvin@ardata.tech` assumed
7. **Real case-study screenshots** — placeholder repeating-line backgrounds ship in v1

## 9. Deploy

- **Hosting:** Vercel, static output (no SSR, no edge functions needed)
- **Astro adapter:** none — pure static build (`output: 'static'`, the default)
- **Build command:** `astro build` (Vercel auto-detects)
- **Output dir:** `dist/`
- **Analytics:** `@vercel/analytics/astro` — single `<Analytics />` component in `Base.astro`
- **Custom domain:** `alvinreyes.ai` configured in Vercel project settings (manual step, done post-deploy)

## 10. What we explicitly do not build

From the brainstorming, these were considered and rejected for v1:
- Blog or RSS — not requested, no content to seed
- Contact form — Calendly link is the CTA
- Newsletter — out of scope
- Light/dark toggle — handoff is intentionally one direction
- CMS — content is in code, edits are PRs
- Embedded Calendly widget — link opens in a new tab
- Tweaks panel — dev tool from the prototype
- Cards and list case-study layouts — only spotlight ships
- Per-section JS interactivity — there is none; the site is HTML + CSS only (Astro ships zero JS by default)

## 11. Success criteria

- Visual diff vs. `Personal Site.html` is pixel-equivalent at desktop (1440px), tablet (820px), and mobile (375px) widths
- Lighthouse mobile score ≥95 across Performance / Accessibility / Best Practices / SEO
- All sections present in the order specified in handoff §5
- No JS shipped (other than the Vercel Analytics script)
- Calendly link opens in a new tab and points at the discovery URL
- `prefers-reduced-motion: reduce` disables hero entrance, signature drift, and status pulse
- Site builds and deploys to a Vercel preview URL successfully

## 12. Reference

The visual source of truth is `design_handoff_alvinreyes_ai/references/Personal Site.html`. The handoff README in the same folder contains exact copy, token values, and section-by-section specs that this document points at rather than duplicates.
