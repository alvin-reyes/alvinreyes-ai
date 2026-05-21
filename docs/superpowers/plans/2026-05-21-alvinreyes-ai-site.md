# alvinreyes.ai Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Scaffold and ship a static Astro site recreating the warm-dark editorial design in `design_handoff_alvinreyes_ai/references/Personal Site.html`, deployed to Vercel at `alvinreyes.ai`.

**Architecture:** Static Astro project at the repo root. Design tokens as CSS custom properties on `:root`. Per-section `.astro` components composed into a single `index.astro`. Zero runtime JS (Vercel Analytics is the only script). Reference HTML is the visual + structural source of truth — extract markup and section CSS from it directly; do not redesign.

**Tech Stack:** Astro (latest stable major), TypeScript (strict), Vercel (static deploy), `@vercel/analytics`, `@astrojs/sitemap`. Google Fonts (Instrument Serif, Geist, JetBrains Mono).

**Spec:** `docs/superpowers/specs/2026-05-21-alvinreyes-ai-site-design.md`

**Reference HTML structure (line ranges in `design_handoff_alvinreyes_ai/references/Personal Site.html`):**

| Section | Lines | Notes |
| --- | --- | --- |
| `<style>` block | 12–902 | All CSS for the page |
| TopBar (`<header class="topbar">`) | 906–924 | |
| Hero (`<section class="hero" id="top">`) | 925–953 | |
| About (`<section class="about" id="about">`) | 954–984 | |
| Holdings (`<section id="holdings">`) | 985–1068 | |
| Arc (`<section class="arc" id="arc">`) | 1069–1121 | |
| Workflow (`<section class="workflow" id="workflow">`) | 1122–1194 | |
| WATED (`<section id="wated">`) | 1195–1243 | |
| BMAD (`<section id="bmad">`) | 1244–1301 | |
| Principles (`<section id="principles">`) | 1302–1334 | |
| Work (`<section id="work">`) | 1335–1414 | Use spotlight variant only |
| Contact (`<section class="contact" id="contact">`) | 1415–1451 | |
| Footer (`<footer>`) | 1452–1462 | |
| Tweaks panel scripts | 1463–1503 | **DO NOT SHIP** |

**Working directory:** `/Users/alvin-reyes/Project/personal-site`

---

## Task 1: Scaffold Astro project and install dependencies

**Files:**
- Create: `package.json`, `astro.config.mjs`, `tsconfig.json`, `src/env.d.ts`
- Modify: existing `.gitignore` (merge Astro's defaults with ours)

- [ ] **Step 1: Scaffold Astro into the existing repo root**

Run from `/Users/alvin-reyes/Project/personal-site`:

```bash
npm create astro@latest . -- --template minimal --typescript strict --install --no-git --yes
```

The directory isn't empty (we already have `.git/`, `README.md`, `docs/`, `.gitignore`, `design_handoff_alvinreyes_ai/`). The `--yes` flag tells Astro to proceed; it will not overwrite our existing files but may print a warning.

Expected: `package.json`, `astro.config.mjs`, `tsconfig.json`, `src/`, `public/`, `node_modules/` created. Existing `README.md`, `docs/`, `.gitignore`, `design_handoff_alvinreyes_ai/` untouched.

- [ ] **Step 2: Add the integrations and analytics**

Run:

```bash
npx astro add sitemap --yes
npm install @vercel/analytics
```

Expected: `@astrojs/sitemap` appears in `package.json` and is wired into `astro.config.mjs`. `@vercel/analytics` added.

- [ ] **Step 3: Configure site URL in `astro.config.mjs`**

Open `astro.config.mjs`. It should look like this after edits (preserve whatever Astro added for sitemap):

```js
// @ts-check
import { defineConfig } from 'astro/config';
import sitemap from '@astrojs/sitemap';

export default defineConfig({
  site: 'https://alvinreyes.ai',
  integrations: [sitemap()],
});
```

- [ ] **Step 4: Merge `.gitignore`**

Astro will have appended its own entries. Verify the final `.gitignore` includes (in any order):

```
node_modules/
dist/
.astro/
.vercel/
.env
.env.*
!.env.example
.DS_Store
.idea/
.vscode/
design_handoff_alvinreyes_ai/
design my personal website alvinreyes.ai.zip
```

Deduplicate any repeated lines.

- [ ] **Step 5: Delete the default scaffolded page**

The scaffold creates `src/pages/index.astro` with a "Welcome to Astro" template. Delete its contents — leave the file empty for now (Task 10 rewrites it).

Run:

```bash
> src/pages/index.astro
```

- [ ] **Step 6: Verify the project boots**

Run:

```bash
npm run dev
```

Expected: dev server starts on `http://localhost:4321/`. Loading `/` shows a blank page (because `index.astro` is empty) with no console errors. Stop the dev server with `Ctrl+C`.

- [ ] **Step 7: Commit**

```bash
git add package.json package-lock.json astro.config.mjs tsconfig.json src/ public/ .gitignore
git commit -m "Scaffold Astro project with sitemap and Vercel analytics"
```

---

## Task 2: Extract design tokens and global styles

**Files:**
- Create: `src/styles/tokens.css`, `src/styles/global.css`

The reference HTML's `<style>` block (lines 12–902) contains tokens (`:root` block), global resets, base typography, motion keyframes, and per-section selectors. Tasks 2 and 3 split out the global parts; later component tasks (4–9) pull section-specific selectors into scoped component styles.

- [ ] **Step 1: Identify the token block in the reference**

Open `design_handoff_alvinreyes_ai/references/Personal Site.html`. Find the `:root { ... }` block near the top of `<style>` (starts around line 13). It contains all CSS custom properties (--bg, --paper, --ink, etc.) and the `clamp()` font-size base.

- [ ] **Step 2: Create `src/styles/tokens.css`**

Copy the entire `:root { ... }` block (every custom property) verbatim into `src/styles/tokens.css`. Do not change any values — the spec requires exact tokens.

Verify it contains at minimum:

```css
:root {
  --bg: #1c1917;
  --paper: #24201c;
  --paper-2: #2c2722;
  --ink: #ede1c8;
  --ink-soft: #c9b896;
  --muted: #8a7a60;
  --rule: #38312a;
  --accent: #d97757;
  --accent-soft: #e8a07d;
  --ok: #7fc99d;
  --col: 1180px;
  --pad-x: clamp(20px, 5vw, 64px);
  --row-gap: 132px;
  /* …plus whatever else the reference defines */
}
```

- [ ] **Step 3: Create `src/styles/global.css`**

From the reference `<style>` block, extract everything that is NOT scoped to a specific section. That includes:

- `*, *::before, *::after { box-sizing: border-box; }` (reset)
- `html, body { ... }` rules
- `body { background: var(--bg); color: var(--ink); font-family: 'Geist', ... }` and base type
- `a { color: inherit; text-decoration: none; }` and similar utility resets
- Headings base: any default for `h1, h2, h3, p` that's not section-scoped
- `.section { padding: var(--row-gap) 0; border-bottom: 1px solid var(--rule); }`
- `.section:last-of-type { border-bottom: none; }`
- `.col { max-width: var(--col); margin: 0 auto; padding: 0 var(--pad-x); }`
- `.eyebrow { ... }` (mono uppercase label utility)
- `.ital { font-style: italic; color: var(--accent); }`
- `.reveal { opacity: 0; transform: translateY(14px); animation: ... }` plus `.d1`–`.d4` delay classes
- `@keyframes` definitions (reveal, drift, statusPulse)
- `@media (prefers-reduced-motion: reduce) { .reveal, .sig, .ok { animation: none !important; transform: none !important; } }`

Skip per-section selectors like `.hero`, `.about`, `.workflow`, `.org`, `.pillar`, etc. — those move to component-scoped styles in later tasks.

Put `@import "./tokens.css";` at the very top of `global.css`.

- [ ] **Step 4: Verify the global stylesheet works in isolation**

Temporarily add to `src/pages/index.astro`:

```astro
---
import '../styles/global.css';
---
<html lang="en">
<head><meta charset="utf-8"><title>smoke</title></head>
<body>
  <div class="col">
    <p class="eyebrow">— sample eyebrow</p>
    <h1 style="font-family: 'Instrument Serif', serif;">Tokens working</h1>
  </div>
</body>
</html>
```

Run `npm run dev` and load `/`. Expected: warm dark background (`#1c1917`), cream text, the eyebrow renders in JetBrains Mono uppercase (font will fall back to system mono until Task 3 wires Google Fonts — that's fine for this check). No console errors.

Stop the dev server. Revert `src/pages/index.astro` to empty.

- [ ] **Step 5: Commit**

```bash
git add src/styles/ src/pages/index.astro
git commit -m "Add design tokens and global styles"
```

---

## Task 3: Build the Base layout with fonts, SEO, JSON-LD, and analytics

**Files:**
- Create: `src/layouts/Base.astro`

- [ ] **Step 1: Create `src/layouts/Base.astro`**

Write the file:

```astro
---
import '../styles/global.css';
import { Analytics } from '@vercel/analytics/astro';

interface Props {
  title?: string;
  description?: string;
  canonical?: string;
  ogImage?: string;
}

const {
  title = 'Alvin Reyes — Production AI & Agentic Workflows | AR Data',
  description = "Alvin Reyes — 20-year practitioner shipping production AI and agentic systems. Founder of AR Data Intelligence Solutions. We don't prototype and disappear. We ship.",
  canonical = 'https://alvinreyes.ai/',
  ogImage = 'https://alvinreyes.ai/og-image.png',
} = Astro.props;

const personLd = {
  '@context': 'https://schema.org',
  '@type': 'Person',
  name: 'Alvin Reyes',
  jobTitle: 'Founder & Principal Engineer',
  worksFor: {
    '@type': 'Organization',
    name: 'AR Data Intelligence Solutions',
    parentOrganization: { '@type': 'Organization', name: 'ARCE Holdings Inc.' },
  },
  url: 'https://alvinreyes.ai/',
  sameAs: [
    'https://twitter.com/alvinjayreyes',
    'https://github.com/alvin-reyes',
    'https://linkedin.com/in/alvinpreyes',
  ],
};

const orgLd = {
  '@context': 'https://schema.org',
  '@type': 'Organization',
  name: 'AR Data Intelligence Solutions',
  url: 'https://ardata.tech',
  parentOrganization: { '@type': 'Organization', name: 'ARCE Holdings Inc.' },
};
---
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>{title}</title>
    <meta name="description" content={description} />
    <link rel="canonical" href={canonical} />

    <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
    <link rel="icon" type="image/png" sizes="32x32" href="/favicon-32.png" />

    <meta property="og:type" content="website" />
    <meta property="og:title" content={title} />
    <meta property="og:description" content={description} />
    <meta property="og:url" content={canonical} />
    <meta property="og:image" content={ogImage} />
    <meta name="twitter:card" content="summary_large_image" />
    <meta name="twitter:creator" content="@alvinjayreyes" />

    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    <link
      rel="stylesheet"
      href="https://fonts.googleapis.com/css2?family=Instrument+Serif:ital@0;1&family=JetBrains+Mono:wght@400;500&family=Geist:wght@300;400;500;600&display=swap"
    />

    <script type="application/ld+json" set:html={JSON.stringify(personLd)} />
    <script type="application/ld+json" set:html={JSON.stringify(orgLd)} />
  </head>
  <body>
    <slot />
    <Analytics />
  </body>
</html>
```

- [ ] **Step 2: Verify the layout boots**

Temporarily make `src/pages/index.astro`:

```astro
---
import Base from '../layouts/Base.astro';
---
<Base>
  <main class="col">
    <h1 style="font-family: 'Instrument Serif', serif; font-size: 4rem;">It ships.</h1>
  </main>
</Base>
```

Run `npm run dev`, load `/`. Expected:
- Warm dark page with cream "It ships." in Instrument Serif (Google Fonts loads)
- View source: title is "Alvin Reyes — Production AI & Agentic Workflows | AR Data"
- View source: two `<script type="application/ld+json">` blocks present with Person and Organization data
- View source: `<meta property="og:title">` and twitter card meta present
- No console errors
- Network tab: Google Fonts requests succeed; Vercel Analytics is loaded (will be a no-op locally)

Stop the dev server. Revert `src/pages/index.astro` to empty.

- [ ] **Step 3: Commit**

```bash
git add src/layouts/Base.astro src/pages/index.astro
git commit -m "Add Base layout with fonts, SEO meta, JSON-LD, and Vercel Analytics"
```

---

## Task 4: Build TopBar and Footer components

**Files:**
- Create: `src/components/TopBar.astro`, `src/components/Footer.astro`

These two are the smallest components and share the "mono 11–12px chrome" style. Building them together establishes the component-extraction pattern used by all later tasks.

**Extraction pattern (reuse in Tasks 4–9):**

1. Open `design_handoff_alvinreyes_ai/references/Personal Site.html`.
2. Copy the section's HTML markup from the line range above into the `.astro` file (everything below the frontmatter).
3. In the reference's `<style>` block (lines 12–902), find every selector that targets this section's classes/IDs (e.g., `.topbar`, `.topbar .nav a`, etc.). Copy those rules into the component's `<style>` block — Astro scopes them automatically.
4. Strip `data-screen-label="…"` attributes (dev tool, do not ship).
5. Strip any `data-cs` or `data-density` attributes if present (Tweaks-panel dev tool).
6. Replace literal token values with `var(--token)` where the reference still has inline literals (most should already be tokens — but if you see `#1c1917`, replace with `var(--bg)`, etc.).
7. Verify in dev server, then commit.

- [ ] **Step 1: Create `src/components/TopBar.astro`**

Extract markup from `Personal Site.html` lines 906–924 and the matching `.topbar*` selectors from the `<style>` block. The component should be self-contained:

```astro
---
// no props
---
<header class="topbar">
  <!-- markup extracted verbatim from reference lines 906–924,
       minus the data-screen-label attribute -->
</header>

<style>
  /* Every selector starting with .topbar from the reference <style> block */
</style>
```

Required behaviors preserved from the reference:
- Sticky `top: 0; z-index: 50`
- `backdrop-filter: blur(10px)` on a `color-mix(in srgb, var(--bg) 88%, transparent)` background
- Bottom border `1px solid var(--rule)`
- Left: AR mark (22×22 burnt-orange circle, white "AR" in mono 8px) + `ALVIN REYES / AR DATA`
- Middle: nav links (About, Holdings, Arc, Workflow, WATED, BMAD, Principles, Work, Book a call) — each `<a href="#anchor">`
- Right: green `.ok` dot (with pulse animation defined in `global.css`) + `TAKING ENGAGEMENTS / Q3 2026`

- [ ] **Step 2: Create `src/components/Footer.astro`**

Extract markup from lines 1452–1462 and matching `footer` selectors from the `<style>` block.

```astro
<footer>
  <!-- markup extracted verbatim from reference lines 1452–1462 -->
</footer>

<style>
  /* Every footer-related selector from the reference */
</style>
```

Required content:
- AR mini-mark (18px) + `© 2026 AR DATA · ALVIN REYES`
- Right: `BOOTSTRAPPED · NO VC` · `ALVINREYES.AI · v2026.05`
- Top border `1px solid var(--rule)`, mono 11px muted

- [ ] **Step 3: Smoke-test both components**

Set `src/pages/index.astro` to:

```astro
---
import Base from '../layouts/Base.astro';
import TopBar from '../components/TopBar.astro';
import Footer from '../components/Footer.astro';
---
<Base>
  <TopBar />
  <main class="col" style="min-height: 60vh; padding-top: 120px;">
    <p>placeholder</p>
  </main>
  <Footer />
</Base>
```

Run `npm run dev`. Expected:
- Sticky top bar visible with all nav items, AR mark, and pulsing green dot
- Footer at the bottom with mini-mark and colophon
- Visual match: open `Personal Site.html` in another tab — top bar and footer should look pixel-identical

Stop the dev server. Leave `src/pages/index.astro` as-is (we'll keep adding to it as we build).

- [ ] **Step 4: Commit**

```bash
git add src/components/TopBar.astro src/components/Footer.astro src/pages/index.astro
git commit -m "Add TopBar and Footer components"
```

---

## Task 5: Build Hero component

**Files:**
- Create: `src/components/Hero.astro`

- [ ] **Step 1: Extract markup**

From `Personal Site.html` lines 925–953, copy the `<section class="hero" id="top">` markup into `src/components/Hero.astro`. Strip `data-screen-label`.

The hero must include:
- Meta line (`.hero-meta`): JetBrains Mono 11px, 0.16em letter-spacing, muted color, three items separated by visual gap (`↓ ALVIN REYES`, `FOUNDER — AR DATA`, `20 YEARS / NO VC / NO BENCH`)
- H1: "We don't prototype / and disappear. / We ship." — with `<span class="ital">ship.</span>` for the accent italic on the last word
- Subhead paragraph mentioning Oracle, IBM, HP, Macquarie, Scotiabank, Protocol Labs
- Two pill CTAs: primary "Book a discovery call →" (linking to the Calendly URL with `target="_blank" rel="noopener"`) and ghost "See the workflow ↓" (linking to `#workflow`)
- Signature graphic: absolute-positioned `.sig` (burnt-orange circle, 14s drift animation)
- `.reveal .d1`, `.reveal .d2`, etc. classes on entrance-animated children

Calendly URL: `https://calendly.com/alvin-ardata/discovery-ai-partnership`.

- [ ] **Step 2: Extract scoped CSS**

From the reference `<style>` block, copy every selector starting with `.hero`, `.hero-meta`, `.hero h1`, `.hero .sub`, `.hero .ctas`, `.hero .btn`, `.hero .sig`, etc. into the component's `<style>` block.

- [ ] **Step 3: Update `src/pages/index.astro`**

```astro
---
import Base from '../layouts/Base.astro';
import TopBar from '../components/TopBar.astro';
import Hero from '../components/Hero.astro';
import Footer from '../components/Footer.astro';
---
<Base>
  <TopBar />
  <main>
    <Hero />
  </main>
  <Footer />
</Base>
```

- [ ] **Step 4: Visual check**

Run `npm run dev`. Open `/` and `Personal Site.html` side by side at desktop width (1440px). The hero should match:
- Giant headline with italic accent "ship." in burnt orange
- Signature circle drifting in the upper right
- Two CTAs at the bottom of the hero
- All text in correct fonts (Instrument Serif for headline, Geist for subhead, JetBrains Mono for meta)

Test `prefers-reduced-motion: reduce` (in Chrome DevTools → Rendering → Emulate CSS media feature `prefers-reduced-motion: reduce`). Expected: signature stops drifting, headline doesn't slide-in on refresh.

Click "Book a discovery call →". Expected: opens Calendly URL in a new tab.

Stop the dev server.

- [ ] **Step 5: Commit**

```bash
git add src/components/Hero.astro src/pages/index.astro
git commit -m "Add Hero section"
```

---

## Task 6: Build About and Holdings components

**Files:**
- Create: `src/components/About.astro`, `src/components/Holdings.astro`

- [ ] **Step 1: Build `About.astro`**

Extract lines 954–984 markup and matching `.about*` selectors from the reference. Strip `data-screen-label`.

Structure:
- 2-col grid `1fr 1.4fr`, gap 72px, collapsing to 1 col below 820px
- Left: eyebrow `01 About`, H2 "The *AR* / is me." (italic on "AR")
- Right: 3 paragraphs of ink-soft body copy with `<strong>` and `<em>` highlights

- [ ] **Step 2: Build `Holdings.astro`**

Extract lines 985–1068 markup and matching `#holdings`, `.org`, `.branch`, `.jv` selectors from the reference. Strip `data-screen-label`.

Structure:
- Eyebrow `02 Holdings`, lede H2, intro paragraph
- Centered "parent" card: ARCE Holdings Inc. with "PARENT · 100%" pill
- 1×32px vertical trunk (rule color)
- Branches grid: 3 cols × 2 rows at desktop, collapsing 3 → 2 → 1
- Six branches per spec §6.3: AR Data, Caiden, SelfVision, Koneksi (placeholder), MetaDev (placeholder), Joint Ventures (with `.jv` diagonal-stripe background)

Each branch card includes: tick label (e.g., `SUB · 01 · OPERATING`), name + small subtitle (domain), focus paragraph, status row with top border.

- [ ] **Step 3: Update `src/pages/index.astro`**

Add `About` and `Holdings` imports + render them in order:

```astro
<Base>
  <TopBar />
  <main>
    <Hero />
    <About />
    <Holdings />
  </main>
  <Footer />
</Base>
```

- [ ] **Step 4: Visual check**

Run `npm run dev`. Compare `/` and `Personal Site.html`:
- About: 2-col layout, italic "AR" in display serif
- Holdings: parent card centered, trunk line, 6 branches in 3×2 grid (or 2 cols at <760px, 1 col at <540px)
- Joint Ventures branch shows diagonal stripe pattern

Resize the browser to verify breakpoints (1100px, 760px, 540px) — branch grid should reflow correctly.

- [ ] **Step 5: Commit**

```bash
git add src/components/About.astro src/components/Holdings.astro src/pages/index.astro
git commit -m "Add About and Holdings sections"
```

---

## Task 7: Build Arc component

**Files:**
- Create: `src/components/Arc.astro`

- [ ] **Step 1: Extract markup and styles**

Extract lines 1069–1121 markup and matching `.arc*` selectors from the reference.

Structure:
- Eyebrow `03 Arc`, lede H2
- 7-row table, columns: Year (110px mono accent), Company (Instrument Serif clamp 26–38px), Role (14px ink-soft), City (90px mono muted, right-aligned)
- Current row (2024–) has `<span class="pin">●</span>` (accent dot) inside Company column
- Note below table (mono 11px muted): "↳ Dates & locations approximate — tell me which to refine."
- Mobile (<820px): rows stack into single-column blocks

Rows per spec §6.4 in this exact order (newest first):
1. 2024– · AR Data · Founder & principal engineer · Bootstrapped (with pin)
2. 2021–24 · Protocol Labs · IPFS / Filecoin · Remote / Global
3. 2018–21 · Scotiabank · Trading & core · Toronto
4. 2015–18 · Macquarie Bank · Capital markets · Sydney / NYC
5. 2010–15 · HP / DXC · Multi-continent infra · Multi-region
6. 2007–10 · IBM · Middleware · APAC
7. 2004–07 · Oracle · DB & enterprise · APAC

- [ ] **Step 2: Render and verify**

Add `<Arc />` after `<Holdings />` in `index.astro`. Run `npm run dev`. Expected:
- 7-row table, each row hover changes background slightly
- Pin dot visible on the 2024– row in accent color
- Resize to <820px: rows stack vertically (year above, company below, etc.)

- [ ] **Step 3: Commit**

```bash
git add src/components/Arc.astro src/pages/index.astro
git commit -m "Add Arc timeline section"
```

---

## Task 8: Build the Approach trilogy — Workflow, WATED, BMAD

**Files:**
- Create: `src/components/Workflow.astro`, `src/components/WATED.astro`, `src/components/BMAD.astro`

These three share an eyebrow prefix scheme (`04`, `04 / 02`, `04 / 03`) and lede prefixes ("Layer 2 of 3 · …", "Layer 3 of 3 · …") that group them under "My Approach". Build all three in this task.

- [ ] **Step 1: Build `Workflow.astro`**

Extract lines 1122–1194 markup and matching `.workflow*`, `.node*`, `.arrow*` selectors from the reference.

Structure:
- Lede H2 with `<em>` on "plan, ground, execute, verify, ship."
- 12-col grid (gap 14px) of nodes in this order:
  1. `00 · INPUT — Intent & constraints` (full width)
  2. Arrow row `↓ decompose` (full width with rule-line ::before/::after)
  3. `01 · PLAN` (cols 1–4) — Task graph
  4. `02 · GROUND` (cols 5–8) — Context layer
  5. `03 · GUARD` (cols 9–12) — Guardrails & rollback
  6. Arrow row `↓ execute`
  7. `04 · EXECUTE` (full width, `.accent` modifier — accent background, bg-color text) — Specialist agents calling typed tools
  8. EXECUTE sub-tools: 4 cards (cols 1–3, 4–6, 7–9, 10–12) — code & docs / data & metrics / on-chain & off-chain / PRs, tickets, comms
  9. Arrow row `↓ check`
  10. `05 · VERIFY` (cols 1–7) — Critic loop & evals
  11. `06 · SHIP` (cols 8–12) — Hand-off, observe, stand behind
- Mobile (<820px): all nodes collapse to full width (`grid-column: 1 / -1 !important`)

Each node: 18px padding, paper bg, 1px rule border, 4px radius. Top label row (mono 10px 0.14em uppercase muted with section label left, role right). Middle name (ink 14px weight 500). Bottom detail (ink-soft 12px).

- [ ] **Step 2: Build `WATED.astro`**

Extract lines 1195–1243 markup and matching `.wated-grid`, `.pillar` selectors from the reference.

Structure:
- Eyebrow `05 Approach · WATED`, H2 lede with `<em>` on "five pillars"
- Intro paragraph
- 5-col grid of pillars (collapses 5 → 2 → 1)
- Each pillar: position-relative, absolute `idx` (e.g., `/ 01`) top-right, giant Instrument Serif letter (W, A, T, E, D) in accent, word (Workflow / Agent / Tools / Events / Data Storage) in mono with top border, then description paragraph
- Pillar descriptions from spec §6.6, copied verbatim
- Closing line (mono 11px muted): "↳ WATED is AR Data's own framework — proven across finance, enterprise, and Web3 engagements."

- [ ] **Step 3: Build `BMAD.astro`**

Extract lines 1244–1301 markup and matching `.bmad-intro`, `.crew`, `.agent` selectors from the reference.

Structure:
- Eyebrow `06 Approach · BMAD`, H2 lede with `<em>` on "what", "how", "inside the WATED frame"
- 2-col intro: left paragraph explains BMAD = Breakthrough Method for Agile AI-Driven Development = public open-source method; right note in dashed-border box (mono 11px muted) re: replayable/reviewable/reversible hand-offs
- 6-col crew grid (collapses 6 → 3 → 1)
- Six agent cards per spec §6.7, in this exact order:
  1. `01 · DISCOVERY · Analyst` — researches domain… → OUT: **Project brief**
  2. `02 · INTENT · PM` — brief → PRD with epics… → OUT: **PRD**
  3. `03 · DESIGN · Architect` — system design across WATED pillars… → OUT: **Architecture doc**
  4. `04 · PLAN · Scrum Master` — drafts shippable stories sized for one agent → OUT: **Stories**
  5. `05 · BUILD · Dev` — implements each story (code, migrations, tests, observability) → OUT: **Pull requests**
  6. `06 · VERIFY · QA` — runs golden evals, security checks, rollback drills → OUT: **Sign-off**

Each card: 22px 18px padding, min-height 220px. `ix` index (mono 10px accent), name (Instrument Serif clamp 24–30px), role (13px ink-soft), `out` line at bottom (mono 10px muted with top border, `OUT: <b>artifact</b>`).

- [ ] **Step 4: Render all three and verify**

Update `index.astro`:

```astro
<Arc />
<Workflow />
<WATED />
<BMAD />
```

Run `npm run dev`. Expected:
- Workflow diagram renders as a 12-col grid with arrow rows between layers; EXECUTE row has accent background
- WATED: 5 giant letters W·A·T·E·D in accent color, each in its own pillar card
- BMAD: 6 agent cards in a row at desktop, collapsing to 3 then 1

Resize browser to ≤820px and verify Workflow collapses to single column; ≤980px verifies BMAD goes 6→3 and WATED goes 5→2.

- [ ] **Step 5: Commit**

```bash
git add src/components/Workflow.astro src/components/WATED.astro src/components/BMAD.astro src/pages/index.astro
git commit -m "Add Workflow, WATED, and BMAD sections (Approach trilogy)"
```

---

## Task 9: Build Principles, Work (spotlight), and Contact components

**Files:**
- Create: `src/components/Principles.astro`, `src/components/Work.astro`, `src/components/Contact.astro`

- [ ] **Step 1: Build `Principles.astro`**

Extract lines 1302–1334 markup and matching `#principles`, `.tenets`, `.tenet` (or similar) selectors from the reference.

Structure:
- Eyebrow `07 Principles`, H2 lede with `<em>` on "how AR Data picks engagements"
- 3-col × 2-row grid (last item spans full width via `.wide` class)
- 1px gap on `--rule` background, 1px outer border, 4px radius, overflow hidden
- Each cell: min-height 280px, padding 40px 32px, paper bg
- Num (mono 11px 0.16em accent): `/ 01` … `/ 05 · FOUNDATIONAL`
- Blockquote (Instrument Serif clamp 24–32px, italic accent on key phrase via `<em>`)
- Gloss paragraph at bottom (14px ink-soft)

The five principles (italic phrases in `<em>`, accent color):
1. Production is the *only* thing that counts.
2. If you can't *roll back*, you can't ship.
3. AI tooling moves fast. *Maintenance* doesn't.
4. No bench. No *prototype-and-disappear.*
5. (`.wide` full-width) Agentic workflow. *Human drives the outcome.* AI supports.

Glosses from the reference HTML — copy verbatim.

- [ ] **Step 2: Build `Work.astro` — spotlight layout only**

Extract lines 1335–1414 markup but **only the spotlight variant**. The reference contains three layouts (cards, list, spotlight) toggled via `[data-cs="…"]` on `<html>`. The spec calls for spotlight only.

To extract spotlight:
1. In the reference, find the `<section id="work">` markup. It includes the 4 case-study cards inside containers that get re-styled by the `[data-cs="spotlight"]` selector group.
2. Drop the `[data-cs]` switching entirely from the component. Just hard-code the spotlight layout: first card is large 2-col (image-slot left, body right), items 2–4 are compact horizontal rows.
3. From the reference `<style>` block, copy only the spotlight-mode selectors (the ones inside `html[data-cs="spotlight"] .work { ... }` rule groups) and rewrite them without the `html[data-cs="spotlight"]` prefix so they apply unconditionally.
4. Discard the `[data-cs="cards"]` and `[data-cs="list"]` rule blocks entirely.

Four case studies per spec §6.9 (placeholders are intentional for v1):
1. **CS-01 — Fintech** — Compliance-grade agentic review for a tier-1 bank — 8× throughput / 100% audit trail / 0 false-ship
2. **CS-02 — Web3 infra** — Filecoin / IPFS pipeline with an agent ops layer — PB-scale / −70% on-call / 99.9% retrieval SLA
3. **CS-03 — Enterprise** — Migrating a Fortune-100 chatbot to multi-agent — 2.4× resolution / −38% cost/session / p95 −1.1s
4. **CS-04 — Open** — "Your engagement here" — placeholder → discovery call

Each card has:
- Image slot (`.cs-imgslot`): aspect-ratio 16/10, repeating diagonal-line placeholder background (CSS only, no real asset)
- Tag (mono 10px accent uppercase): `CASE STUDY · 01 — FINTECH`
- H3 title (Instrument Serif clamp 24–32px)
- Body paragraph (14px ink-soft)
- Metrics row (3 metrics): big serif value + mono uppercase key

- [ ] **Step 3: Build `Contact.astro`**

Extract lines 1415–1451 markup and matching `.contact*` selectors.

Structure:
- Eyebrow `09 Contact`, large H2 "Let's build / the *production* system." (italic accent on "production")
- Sub-paragraph (~620px max-width)
- Panel (paper bg, 8px radius, 1px rule, padding 36px, 2-col grid collapsing to 1 col <760px):
  - Left: small label `BOOK A CALL`, serif headline "30-minute AI partnership discovery.", filled accent pill button "Open Calendly →" linking to `https://calendly.com/alvin-ardata/discovery-ai-partnership` (`target="_blank" rel="noopener"`)
  - Right: vertical list of contact lines (mono 12px, paper-2 bg, 1px rule between, hover ink bg). Each line is `LABEL /handle ↗`:
    - TWITTER /alvinjayreyes → `https://twitter.com/alvinjayreyes`
    - GITHUB /alvin-reyes → `https://github.com/alvin-reyes`
    - LINKEDIN /alvinpreyes → `https://linkedin.com/in/alvinpreyes`
    - EMAIL alvin@ardata.tech → `mailto:alvin@ardata.tech`

All external links: `target="_blank" rel="noopener"`. The mailto does not need those attributes.

- [ ] **Step 4: Render all three and verify**

Update `index.astro` to add `<Principles />`, `<Work />`, `<Contact />` after `<BMAD />`.

Run `npm run dev`. Expected:
- Principles: 3×2 grid, item 5 spans full width
- Work: first card is large with image-slot on the left (diagonal stripe placeholder), items 2–4 are compact rows
- Contact: 2-col panel; clicking "Open Calendly →" opens Calendly in new tab; clicking GitHub opens GitHub profile in new tab; clicking EMAIL opens mail client

- [ ] **Step 5: Commit**

```bash
git add src/components/Principles.astro src/components/Work.astro src/components/Contact.astro src/pages/index.astro
git commit -m "Add Principles, Work (spotlight), and Contact sections"
```

---

## Task 10: Finalize `index.astro` composition and add favicon + robots.txt

**Files:**
- Modify: `src/pages/index.astro`
- Create: `public/favicon.svg`, `public/favicon-32.png`, `public/robots.txt`

- [ ] **Step 1: Lock in `src/pages/index.astro`**

Final version:

```astro
---
import Base from '../layouts/Base.astro';
import TopBar from '../components/TopBar.astro';
import Hero from '../components/Hero.astro';
import About from '../components/About.astro';
import Holdings from '../components/Holdings.astro';
import Arc from '../components/Arc.astro';
import Workflow from '../components/Workflow.astro';
import WATED from '../components/WATED.astro';
import BMAD from '../components/BMAD.astro';
import Principles from '../components/Principles.astro';
import Work from '../components/Work.astro';
import Contact from '../components/Contact.astro';
import Footer from '../components/Footer.astro';
---
<Base>
  <TopBar />
  <main>
    <Hero />
    <About />
    <Holdings />
    <Arc />
    <Workflow />
    <WATED />
    <BMAD />
    <Principles />
    <Work />
    <Contact />
  </main>
  <Footer />
</Base>
```

- [ ] **Step 2: Create `public/favicon.svg`**

A 32×32 burnt-orange filled circle with white "AR" centered:

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 32">
  <circle cx="16" cy="16" r="16" fill="#d97757"/>
  <text x="16" y="20" text-anchor="middle"
        font-family="ui-monospace, 'JetBrains Mono', monospace"
        font-size="11" font-weight="500" fill="#ffffff">AR</text>
</svg>
```

- [ ] **Step 3: Generate `public/favicon-32.png` fallback**

If you have ImageMagick or `rsvg-convert`:

```bash
rsvg-convert -w 32 -h 32 public/favicon.svg -o public/favicon-32.png
```

If not, open `favicon.svg` in any browser, screenshot at 32×32, and save as `public/favicon-32.png`. Or skip and remove the `<link rel="icon" type="image/png">` from `Base.astro` — modern browsers will use the SVG.

- [ ] **Step 4: Create `public/robots.txt`**

```
User-agent: *
Allow: /

Sitemap: https://alvinreyes.ai/sitemap-index.xml
```

(The `@astrojs/sitemap` integration outputs `sitemap-index.xml` and `sitemap-0.xml` at build time.)

- [ ] **Step 5: Full visual diff**

Run `npm run build` then `npm run preview`. Open `http://localhost:4321/` and `Personal Site.html` side by side. Verify:
- Section order matches: TopBar, Hero, About, Holdings, Arc, Workflow, WATED, BMAD, Principles, Work, Contact, Footer
- No visual regressions vs. the reference
- Top-bar nav links scroll to the right anchors
- Favicon appears in the browser tab (burnt-orange circle)
- View source: `<link rel="canonical">`, OG meta, JSON-LD, sitemap link all present

Build output `dist/` should contain `index.html`, `sitemap-index.xml`, `sitemap-0.xml`, `favicon.svg`, `favicon-32.png`, `robots.txt`.

- [ ] **Step 6: Commit**

```bash
git add src/pages/index.astro public/favicon.svg public/favicon-32.png public/robots.txt
git commit -m "Compose final index.astro with favicon, robots.txt"
```

---

## Task 11: Accessibility, motion, and Lighthouse pass

**Files:** none new — verification + fixes inline

- [ ] **Step 1: `prefers-reduced-motion` audit**

In Chrome DevTools → Rendering → emulate `prefers-reduced-motion: reduce`. Reload `/`. Verify:
- Hero `.reveal` elements appear without slide-up animation
- Hero signature `.sig` is stationary (no drift)
- TopBar `.ok` green dot does not pulse
- All hover transitions still work (those are user-driven, not entrance animations)

If any animation still plays under `reduce`, fix the `@media (prefers-reduced-motion: reduce)` block in `src/styles/global.css` to add the offending selector.

- [ ] **Step 2: Lighthouse run (mobile)**

In Chrome DevTools → Lighthouse → Mobile → Performance/Accessibility/Best Practices/SEO. Run against `npm run preview` build (not dev).

Target: ≥95 on each category. Most likely deductions and fixes:
- Performance: if <95, check whether Google Fonts are loading async — they should be (we use `<link rel="stylesheet">` which is render-blocking but acceptable for a single-page editorial site)
- Accessibility: if any contrast issues are flagged on muted text, verify they're on text we're allowed to keep at `--muted` (labels only — body copy uses `--ink-soft` which has stronger contrast)
- SEO: if any link is missing accessible name, add aria-label
- Best Practices: should be 100 if no console errors

- [ ] **Step 3: Cross-browser sanity check**

Open the preview in Safari and Firefox (in addition to Chrome). Verify the layout doesn't break — particularly `backdrop-filter` on the top bar (Safari needs `-webkit-backdrop-filter`; add it to the topbar CSS if needed) and `color-mix()` (supported in modern Safari/Firefox; if a fallback is needed, use a literal hex).

- [ ] **Step 4: Commit any fixes**

```bash
git add -u
git commit -m "Accessibility, reduced-motion, and cross-browser fixes"
```

(Skip this commit if no fixes were needed.)

---

## Task 12: Deploy to Vercel

**Files:** none new

- [ ] **Step 1: Push the build-ready repo to GitHub**

```bash
git push origin main
```

- [ ] **Step 2: Connect the GitHub repo to Vercel**

This is a manual step in the Vercel dashboard:
1. Go to https://vercel.com/new
2. Import `alvin-reyes/alvinreyes-ai`
3. Framework preset: Astro (auto-detected)
4. Build command: `npm run build` (default)
5. Output directory: `dist` (default)
6. No environment variables needed for v1
7. Click Deploy

Expected: preview URL `https://alvinreyes-ai-<hash>.vercel.app` deploys successfully. Open it and verify the live site renders correctly.

- [ ] **Step 3: Verify Vercel Analytics is collecting**

Open the preview URL, click around for a few seconds, then check Vercel dashboard → Project → Analytics. You should see at least one page view recorded (allow a minute for ingestion).

- [ ] **Step 4: Configure custom domain `alvinreyes.ai`**

In Vercel project settings → Domains → Add `alvinreyes.ai`. Follow Vercel's DNS instructions (set A record or CNAME at your registrar). Wait for SSL provisioning.

- [ ] **Step 5: Final live verification**

Once `https://alvinreyes.ai` resolves:
- Open in a fresh browser. Page loads, fonts render, sections present.
- Test the Calendly CTA — opens Calendly in a new tab
- View source — JSON-LD blocks present, canonical points at `https://alvinreyes.ai/`
- Fetch `https://alvinreyes.ai/sitemap-index.xml` — returns valid XML
- Fetch `https://alvinreyes.ai/robots.txt` — returns the allow-all config

- [ ] **Step 6: Tag the release**

```bash
git tag -a v2026.05.0 -m "v1 launch — alvinreyes.ai live"
git push origin v2026.05.0
```

---

## Post-launch (not in this plan)

These items from the spec §8 are launch-blockers but not implementation tasks — they need real content from Alvin:

- Real OG image (1200×630 PNG) for `public/og-image.png`
- Real copy for Koneksi and MetaDev branches in Holdings
- Verified domains for all operating companies
- Confirmed Arc dates and cities
- Real case studies with verified metrics (or strip case-study cards down to just "Your engagement here")
- Confirm `alvin@ardata.tech` is the right contact email

Each is a one-line copy change once the answer is known.
