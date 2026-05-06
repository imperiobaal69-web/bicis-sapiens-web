# Bicis Sapiens — Handoff

> **For:** the next agent (or future-me) picking up this work.
> Read this once and you should be productive in ~10 minutes.
>
> Notes in **English** for technical clarity, **Spanish/Portuguese** sprinkled
> in where it helps to understand the client.

---

## 1. Current state

### What this is

**Bicis Sapiens** is a civic-mobility movement and digital platform for Porto,
Portugal. Thesis: *cities can be 100% cyclable without new infrastructure —
through respect, smart signage, and cultural change.* The site is
simultaneously (a) a public-facing manifesto, (b) a pitch deck for city halls,
and (c) a silent investor pitch for the founder's bike-routing app.

> *"O espaço público é de todos. A cidade também."* — canonical tagline.

### Who's involved

| | |
|---|---|
| **Client** | Ricardo Villalobos — Mexican living in Porto, founder of the movement. Bio: *"Live learner about human mobility."* |
| **Builder (founder)** | Paulo (the user you're talking to). Owns FOAAP separately; this is a side gig. **Do NOT mix this repo with FOAAP at `/Users/baphomet/Downloads/foaap-new` or `/foaap-core`.** |
| **End audience** | (1) Porto families · (2) Schools running Bike Bus programs · (3) City halls / municípios · (4) Civic donors / app investors |

### Stack

- **Vite + React 18** (NOT Next.js — Paulo asked for the doc to mention Next, but the actual stack is Vite.)
- Tailwind CSS with shadcn-style HSL CSS-variable token system
- React Leaflet (4.x) for the choropleth map
- Framer Motion for scroll reveals
- Lucide icons
- Base44 SDK (`@base44/sdk`) — comes from how the project was originally bootstrapped on Base44; can be left alone or stripped later
- Custom React-context i18n (no library)

### URLs

| | |
|---|---|
| **Repo** | <https://github.com/imperiobaal69-web/bicis-sapiens-web> |
| **Production / preview** (currently the same) | <https://bicis-sapiens-web.vercel.app> |
| **Vercel project** | `bicis-sapiens-web` under team `imperiobaal69-5594s-projects` |
| **Final domain (pending DNS)** | <https://bicisapiens.org> — registered at **GoDaddy**, nameservers `ns45/ns46.domaincontrol.com`. Domain is added to the Vercel project; DNS records are pending Ricardo's action (see §5). |
| **Local clone** | `/Users/baphomet/Downloads/bicis-sapiens-web` |

### How auth is set up locally

- Git push uses **macOS Keychain** with the `imperiobaal69-web` GitHub account (HTTPS, no SSH key, no `gh` CLI).
- `git config --global credential.helper osxkeychain`
- Vercel CLI is **not globally installed**; we always invoke it as `npx -y vercel@latest`. Paulo is logged in as `imperiobaal69-5594` (verified with `vercel whoami`).
- Brew is **not** installed on this machine. Anything that wants `brew install` needs to be replaced with an `npx` package or a direct binary download.

---

## 2. Code architecture

### Folder layout

```
bicis-sapiens-web/
├── index.html                          # Root HTML. Google Fonts <link> with
│                                       #   preconnect, hero LCP image preload,
│                                       #   theme-color #003399, og:url
│                                       #   bicisapiens.org.
├── public/
│   ├── favicon.svg                     # Simplified bike, no border (≤32px use)
│   ├── logo-solid.svg                  # 100×100 — square tile, EU-blue frame,
│   │                                   #   bone bg, yellow rear-hub detail
│   ├── logo-anamorphic.svg             # 200×200 — same as solid + azulejo
│   │                                   #   floral motif in 4 corners
│   ├── wordmark-horizontal.svg         # 600×200 — logo + "Bicis Sapiens"
│   │                                   #   (italic Sapiens in EU blue)
│   ├── images/
│   │   ├── hero-bridge.webp            # 1264×832, 210KB — Ponte Dom Luís
│   │   ├── hero-bridge-mobile.webp     # 1200×790, 151KB
│   │   └── manifesto/
│   │       ├── manifesto-01-crianca-bicicleta.webp        # 1264×848, 215KB
│   │       ├── manifesto-02-ciclista-rua-partilhada.webp  # 1264×848, 193KB
│   │       └── manifesto-03-cargo-bike-familia.webp       # 1264×848, 235KB
│   └── data/
│       └── freguesias-porto.geojson    # 68 freguesias, 7 municípios, ~840KB
│                                       #   Geometry: real CAOP via GeoAPI.pt
│                                       #   Properties: real where known
│                                       #   (Porto + parts of Espinho/Matosinhos
│                                       #   populacao from INE 2021), estimated
│                                       #   elsewhere by municipal density.
│                                       #   Each feature has a _todo array
│                                       #   listing what's placeholder.
├── src/
│   ├── main.jsx                        # ReactDOM entry, mounts <App /> with
│   │                                   #   I18nProvider wrapper.
│   ├── App.jsx                         # Router + AuthProvider +
│   │                                   #   QueryClientProvider. Single route
│   │                                   #   "/" → <Landing />.
│   ├── index.css                       # 600+ lines:
│   │                                   #   • @tailwind base/components/utilities
│   │                                   #   • CSS-var palette (HSL)
│   │                                   #   • global h1-h4 typography
│   │                                   #   • bs-hero-* (hero image+text panel)
│   │                                   #   • bs-principle-* (manifesto poster)
│   │                                   #   • bs-btn-primary / bs-btn-secondary
│   │                                   #   • bs-ticker / bs-manifesto-colophon
│   │                                   #   • Leaflet overrides (zoom, tooltip)
│   │                                   #   • polygon stagger fade-in
│   ├── pages/
│   │   └── Landing.jsx                 # Composes all sections in order:
│   │                                   #   Navbar / Hero / Manifesto /
│   │                                   #   DataDashboard / Solution3Cs /
│   │                                   #   InteractiveMap / BikeBus / AppCTA /
│   │                                   #   CommunityHub / Municipalities /
│   │                                   #   Resources / AboutTeam / Donate /
│   │                                   #   Footer + JoinModal.
│   ├── lib/
│   │   ├── i18n.jsx                    # 4-language object (pt/en/es/fr) +
│   │   │                               #   I18nProvider context + useI18n()
│   │   │                               #   hook + t(path) lookup. ~530 lines.
│   │   ├── AuthContext.jsx             # Base44 auth state (not used in
│   │   │                               #   public landing; safe to ignore).
│   │   ├── app-params.js               # Reads VITE_BASE44_* env vars.
│   │   ├── query-client.js             # @tanstack/react-query setup.
│   │   ├── useScrollReveal.js          # IntersectionObserver hook used by
│   │   │                               #   most landing sections.
│   │   ├── i18n.jsx                    # (same file, listed once above)
│   │   ├── PageNotFound.jsx            # 404 page.
│   │   └── utils.js                    # cn() helper for shadcn classes.
│   ├── api/
│   │   └── base44Client.js             # createClient() from @base44/sdk.
│   │                                   #   appId resolved from URL params or
│   │                                   #   VITE_BASE44_APP_ID env.
│   ├── components/
│   │   ├── landing/                    # All section components live here:
│   │   │   ├── Navbar.jsx              # Sticky nav, lang switcher, gradient
│   │   │   │                           #   over hero / blur when scrolled.
│   │   │   ├── Hero.jsx                # Full-bleed bridge img + counter +
│   │   │   │                           #   eyebrow + Playfair tagline + lead +
│   │   │   │                           #   2 CTAs + EU-blue ticker strip.
│   │   │   ├── Manifesto.jsx           # 3 full-screen poster panels (one per
│   │   │   │                           #   principle), photo bg + overlay +
│   │   │   │                           #   centered type.
│   │   │   ├── DataDashboard.jsx       # 8 cream stat cards on obsidian,
│   │   │   │                           #   "green m²/hab" featured.
│   │   │   ├── Solution3Cs.jsx         # 3 cream cards (info/consensus/
│   │   │   │                           #   proposal), Consenso featured.
│   │   │   ├── InteractiveMap.jsx      # Leaflet choropleth, dark CARTO tiles,
│   │   │   │                           #   68 freguesia polygons, side panel
│   │   │   │                           #   sheet, A|B compare, search, layers.
│   │   │   ├── BikeBus.jsx             # 6 city cards, Porto featured.
│   │   │   ├── AppCTA.jsx              # Split layout, EU-blue phone mockup,
│   │   │   │                           #   waitlist + backer CTAs.
│   │   │   ├── CommunityHub.jsx        # 4 topic cards (top one featured) +
│   │   │   │                           #   2 poll cards.
│   │   │   ├── Municipalities.jsx      # Stats + dossier (5-item list).
│   │   │   ├── Resources.jsx           # 4 guide cards + 6 inspiring cities,
│   │   │   │                           #   Bike Bus + Pontevedra featured.
│   │   │   ├── AboutTeam.jsx           # Ricardo card (featured) + 4 advisors
│   │   │   │                           #   + 6 ally placeholders.
│   │   │   ├── Donate.jsx              # 3 tier buttons + breakdown chart.
│   │   │   ├── Footer.jsx              # Newsletter + contact + social + legal
│   │   │   ├── JoinModal.jsx           # Modal triggered by "Junta-te ao
│   │   │   │                           #   movimento" CTA across the site.
│   │   │   └── AnimatedCounter.jsx     # IntersectionObserver-driven count-up.
│   │   ├── ui/                         # shadcn-style primitives (button,
│   │   │   │                           #   dialog, dropdown, etc.) — mostly
│   │   │   │                           #   untouched; fine to leave alone.
│   │   ├── ProtectedRoute.jsx          # (Base44 auth wrapper, unused.)
│   │   └── UserNotRegisteredError.jsx  # (Base44 error state, unused.)
├── tailwind.config.js                  # Custom borderRadius (all 0 except
│                                       #   `full`), letterSpacing tightest
│                                       #   (-0.03em) + widest (0.4em),
│                                       #   bs- font families, named colors
│                                       #   (obsidian / eu-blue / eu-yellow /
│                                       #   bone) + semantic HSL tokens.
├── package.json
├── package-lock.json
├── vite.config.js                      # Includes @base44/vite-plugin —
│                                       #   safe to keep; emits "[base44]
│                                       #   Proxy not enabled" warning at
│                                       #   build, ignorable.
├── README.md                           # 1-line description.
└── HANDOFF.md                          # ← this file.
```

### Where the design system lives

There is **no separate `tokens.css` or `BRAND.md`** in this repo. The design
system is split across two files:

1. **`src/index.css`** — top of file:
   - `:root { --background, --foreground, --primary, --accent, ... --radius: 0 }` (HSL channels for shadcn-style opacity modifiers).
   - Global `h1, h2, h3, h4` rules (Playfair 900, -0.03em).
   - Italic-keyword default rule (`h1 em → italic + EU-blue`) — overridden per-context.
2. **`tailwind.config.js`** — exposes those tokens to Tailwind classes (`bg-primary`, `text-foreground`, etc.) plus 4 named hex shortcuts: `obsidian`, `eu-blue`, `eu-yellow`, `bone`.

Custom non-utility CSS classes are all prefixed `bs-` and live in `index.css`
below the token block.

### Image pipeline

We **don't** have a build-time image optimization step. Workflow:

1. Paulo drops PNG/JPG into `~/Downloads/` (Runway-generated, often with weird filenames like `foo.webp.png`).
2. Agent finds them via `find` / `ls`, copies to `/tmp`.
3. Convert to WebP via `npx -y cwebp-bin@7 cwebp -q 80 input.png -o output.webp` (use `-resize 2400 0` only when source > 2400px wide; do not upscale).
4. Move to `/public/images/...` with a clean kebab-case name.
5. Reference from JSX as `/images/foo.webp` (Vite serves `public/` at root).
6. Eager-load with `<link rel=preload>` only for above-the-fold (hero); use `loading="lazy" decoding="async"` for everything else.

`sips` (macOS native) **can't** export WebP — use `cwebp-bin` via npx.

### i18n system

Custom and tiny (~30 lines of provider + 530 lines of strings). No library.

```js
// src/lib/i18n.jsx
const translations = {
  pt: { nav, hero, manifesto, data, solution, map, bikeBus, app, community,
        municipalities, resources, about, donate, footer, join },
  en: { ... },
  es: { ... },
  fr: { ... },
};

// React context + useState for `lang`. Switching `lang` re-renders all
// consumers. Path-based lookup with dot notation, including array indexes:
t('manifesto.principles.1.mega_html')  // → '<em>infraestrutura.</em>'-style
```

The provider lives at the top of `<App />`. The Navbar exposes PT / EN / ES /
FR buttons that call `setLang(...)`. Default lang is `pt`.

**Mega phrases that contain `<em>` markup are stored as `*_html` keys** and
rendered with `dangerouslySetInnerHTML`. This is how the keyword highlight
travels with the translation — each language picks its own outlined word
(*todos / everyone / todos / tous*).

---

## 3. Design decisions in force

### Palette — 80 / 15 / 4 / 1

| Role | Hex | Usage rule |
|---|---|---|
| Background **(80%)** | `#0A0A0A` *obsidian* | Dominant section bg site-wide. Almost every section is `bg-background`. |
| Structure / CTAs **(15%)** | `#003399` *EU blue* | Buttons, featured-card left border, italic keywords in titles, stat highlights, focus rings. |
| Accent / counters **(4%)** | `#FFD60A` *EU yellow* | **ONLY** in two places: (a) section counter eyebrows like "02 / 13 · O Manifesto", and (b) the keyword inside each manifesto principle's mega phrase ("crianças" / "infraestrutura" / "família"). Never in buttons, never as a background fill, never in supporting text. |
| Text **(1%)** | `#FAFAF7` *bone* | All foreground type. Variable opacity (100 / 70 / 55 / 40 %) for hierarchy. **NEVER use gray** for muted text — always bone with reduced opacity. |

> **Hard rule** (the one Paulo will yell about if broken): if you find yourself adding yellow somewhere new, ask if it's a counter or the manifesto keyword. If neither, make it blue, bone, or muted bone.

### Typography

| Use | Family | Weight | Letter-spacing |
|---|---|---|---|
| Display (h1, h2, h3, h4) | **Playfair Display** | 900 (Black) | `-0.03em` (tracking-tightest) |
| Display italic keyword inside a title | Playfair Display | 900 italic | inherits |
| Body / supporting | **Inter** | 400 (regular) / 500 | normal |
| Eyebrows / metadata / counters / CTAs | **JetBrains Mono** | 400 / 500 | `0.4em` uppercase (tracking-widest is overridden to `0.4em` in tailwind config) |

Loaded in `index.html` via Google Fonts `<link>` with `preconnect`. Weights
loaded: Playfair `400, 900, 1,900` (regular + black + italic black) — Inter
`400, 500` — JetBrains Mono `400, 500`. Don't add more weights without
checking LCP impact.

### Layout language

- **`border-radius: 0`** is the default for every box in the system. Tailwind config maps `rounded-sm/md/lg/xl/2xl/3xl` all to 0; only `rounded-full` remains for actual circles (avatars, dots). This is enforced — institutional rectangles, no soft edges.
- **Hairline grid** between adjacent cards: parent gets `gap-px bg-border`, children get `bg-bone` — produces 1px lines between cells (Bauhaus-style).
- **One featured card per section** with `border-left: 4px solid #003399` + extra `padding-left`. Featured chosen by content meaning, not aesthetics:
  - DataDashboard → `green m²/hab` (worst stat vs WHO target)
  - Solution3Cs → `Consenso`
  - BikeBus → Porto
  - CommunityHub → top hot topic
  - Resources → Bike Bus guide + Pontevedra
  - Donate → €25 tier
  - AboutTeam → Ricardo
- **Counters** use the format `0X / 13 · {section title}` in mono yellow uppercase (manifesto principles use `0X / 03` instead — they're sub-counters within a section).

### Patterns to keep

- Sections start with `<section className="reveal-section ...">` so `useScrollReveal` fades them in.
- Every section has at most one h2 (its main heading) plus h3s for subitems.
- Buttons use `bs-btn-primary` / `bs-btn-secondary` (custom CSS) when in the hero, or Tailwind utility classes elsewhere — both end up `bg-primary text-primary-foreground` with mono uppercase 0.4em.

### Patterns deliberately abandoned (do NOT bring back)

- ❌ **Glassmorphism / backdrop-blur cards over photos.** Tried this on hero + manifesto principles. Paulo's reaction (literal): *"bro que mierda haces???? coño las imagenes no se pueden tapar hdp regenra a como esta solo como se veian antes."* The frosted card hid the photo. Lesson: **photos breathe**. Use the existing dark gradient overlay on photos + a strong text-shadow on the type. That's enough.
- ❌ **Yellow keyword on a solid blue rectangle marker** in manifesto. Tried it; the rectangle clipped the line above. Now keyword is just yellow italic Playfair, no background.
- ❌ **Different alignments per principle (left / right / center)** in manifesto. Removed when manifesto became poster panels.
- ❌ **Giant chapter watermarks behind text** in manifesto. Removed when posters took over — the photo IS the visual anchor.
- ❌ **`text-muted-foreground` (HSL gray)** for any visible text. Always `text-foreground/55` (bone with opacity) instead.

### Philosophy in one line

> Editorial-institutional. Heavy on type, restrained on color, photo as the
> dramatic device. Closer to **A24 posters / Monocle / The New Yorker** than
> to Decathlon, Trek, or any startup template.

---

## 4. Section-by-section status

| # | Section | Component | Status | Notes |
|---|---|---|---|---|
| — | Navbar | `Navbar.jsx` | ✅ done | Gradient over hero, blur when scrolled. Lang switcher works. CTA opens JoinModal. |
| 01 | **Hero** | `Hero.jsx` | ✅ done | Bridge `<picture>` (mobile + desktop sources, eager + preload), staggered entry animation, EU-blue ticker strip at the bottom. `<em>todos</em>` is *blue text-stroke outline* (different from manifesto keyword). |
| 02 | **Manifesto** | `Manifesto.jsx` | ✅ done | 3 full-screen 100vh/100dvh poster panels, photo bg + dark vertical-gradient overlay + centered Playfair Black + Inter supporting + EU-yellow italic keyword. Colophon at end. |
| 03 | **Os Números (Data)** | `DataDashboard.jsx` | ✅ done — *minor refinement* | 8 cream cards on obsidian, "green m²/hab" featured. Compare toggle (Amsterdam/Paris/Copenhagen). Could use icons per stat for stronger visual rhythm — currently text-only. |
| 04 | **Solução · 3 Cs** | `Solution3Cs.jsx` | ✅ done | 3 cream cards, Consenso featured. Watermark numbers behind. Italic Playfair subtitle below the grid. |
| 05 | **Mapa Interativo** | `InteractiveMap.jsx` | ⚠️ **partially done** | **Choropleth works** — 68 freguesias from real CAOP via GeoAPI.pt, painted by heraldic color, side-panel sheet with stats + câmara contact, A\|B comparison, diacritic-insensitive search, ESC closes, double-click flies. **Layers toggle UI exists but does nothing yet** — no real escolas/parques/Bike-Bus data layer. Schools/parks/Bike-Bus data is the biggest data-gathering TODO on the project. |
| 06 | **Comboios / Bike Bus** | `BikeBus.jsx` | ✅ done | 6 city cards (PT/ES/NL), Porto featured with "Sede" tag. CTAs open JoinModal. Image URLs are Unsplash placeholders — replace with real photos when available. |
| 07 | **App Bicis Sapiens** | `AppCTA.jsx` | ✅ done | Split layout, EU-blue phone mockup with mini route preview, waitlist + backer CTAs (both POST to Base44 Subscriber entity if env vars are set). |
| 08 | **Comunidade** | `CommunityHub.jsx` | ✅ done | Mock topics + polls (4 + 2). All data is local mock — no backend yet. Top "hot" topic featured. CTA opens JoinModal. |
| 09 | **Para Municípios** | `Municipalities.jsx` | ⚠️ counter is "09" but conflicts with Resources also using 09 — needs fix | Stats grid + dossier panel listing 5 document types. No real PDF download yet — the button is a placeholder. Calendly integration is also pending. |
| 10 | **Recursos** | `Resources.jsx` | ✅ done | 4 guide cards + 6 inspiring cities, Bike Bus guide + Pontevedra featured. Guide images are Unsplash placeholders. Real guide content TBD. |
| 11 | **Sobre / Equipa** | `AboutTeam.jsx` | ⚠️ placeholders | Founder card (Ricardo, featured with EU-blue stripe + EU-blue placeholder portrait) + 4 advisor cards with fake names + 6 ally logo placeholders. **All names except Ricardo are fake** and clearly marked TODO in the brief. |
| 12 | **Doar / Transparência** | `Donate.jsx` | ⚠️ structurally done, no payment | 3 tier buttons (€5/€25/€100, €25 featured) + custom amount + "Doar €X" button (no Stripe wired) + breakdown chart (4 segments) + payment-method labels (Stripe / MB Way / Transferência). Real payment integration is **explicit phase-2 work**. |
| 13 | **Contacto / Footer** | `Footer.jsx` | ✅ done | Newsletter signup (POSTs to Base44 if configured), contact email, social links (placeholder #), legal links (placeholder #), copyright. |
| modal | JoinModal | `JoinModal.jsx` | ✅ done | Triggered by every "Junta-te" CTA across the site. POSTs to Base44 Subscriber entity if env vars present. |

### What every section's counter looks like

```
01 / 13 · O MANIFESTO       (in hero — points to next section)
02 / 13 · O Manifesto       (manifesto's own header — but rendered as colophon)
02 / 13 · Porto em Números  (DataDashboard)
03 / 13 · A Nossa Solução
05 / 13 · Mapa
06 / 13 · Comboios de Bicicletas
... etc.
```

The counter is yellow mono uppercase 0.4em. **It is the only place yellow lives in flat copy.**

---

## 5. What's left / TODO

### High priority — blocks shipping

1. **DNS for `bicisapiens.org`.** Domain is added to the Vercel project, but DNS still resolves to GoDaddy default. Ricardo needs to (in the GoDaddy DNS panel for `bicisapiens.org`):
   - Delete any existing A or CNAME records on `@` and `www`, plus any "Parked" / "Forwarding" entries.
   - Add `A @ → 76.76.21.21`
   - Add `CNAME www → cname.vercel-dns.com.`
   - Wait 5–60 min for propagation; Vercel auto-verifies and issues the SSL cert.
2. **Base44 env vars on Vercel.** Without `VITE_BASE44_APP_ID` and `VITE_BASE44_APP_BASE_URL`, every Subscriber-create call (newsletter, JoinModal, app waitlist) silently fails with a 404. Site renders fine, lead capture does not. Set them via `vercel env add` once Paulo has them.
3. **Connect GitHub to Vercel for auto-deploy.** Currently every deploy is manual `npx vercel --prod --yes` from the laptop. Vercel reports *"You need to add a Login Connection to your GitHub account first"* — Paulo needs to go to **vercel.com → Account Settings → Login Connections → Connect GitHub**. After that, run `vercel git connect` once and pushes auto-deploy.

### Medium priority — quality

4. **Real INE 2021 freguesia population** for the 7 municípios. Currently Porto + Espinho + parts of Matosinhos use the real census number; the rest are estimated from `area_km2 × municipal_density`. Each feature in `freguesias-porto.geojson` has a `_todo` array flagging which fields are estimated. Source: `https://recenseamento.ine.pt`.
5. **Map layers data.** `InteractiveMap.jsx` has a UI toggle for "Escolas / Parques / Bike Bus" but no underlying layer is rendered. Needs:
   - Schools dataset (DGEEC has one).
   - Parks polygons (each câmara publishes openly; harder to aggregate).
   - Bike Bus active routes (Bicis Sapiens internal registry — needs to be created).
6. **Heraldic shields per freguesia** (`escudo_url` field). All currently `null`; the side panel shows a colored placeholder with the freguesia's first 3 letters. Pull official SVGs from Wikimedia Commons (URL pattern: `https://commons.wikimedia.org/wiki/Special:FilePath/POR-{name}.svg`). Lazy-load on click.
7. **Verify `cor_principal` per freguesia** against the actual heraldic blazon. Currently a deterministic hash → 12-color palette. Some will be wrong vs the real shield.
8. **Performance pass.** Bundle is 736KB JS / 91KB CSS, no code splitting. Lighthouse Performance ~69. Top wins: dynamic-import Leaflet, three.js, recharts, framer-motion. Estimated 30–40% reduction on initial bundle if done right.
9. **Section-counter numbering bug**: Municipalities and Resources both render "09 / 13" in their counters (Resources should be 10). Sed fix.
10. **Mobile QA on Manifesto posters.** 100vh on iOS Safari with the URL bar visible can clip. CSS uses `min-height: 100vh; min-height: 100dvh;` to handle this — but worth testing on a real device.

### Lower priority — content

11. **Real testimonials, advisor list, ally logos.** All placeholders (clearly marked).
12. **Real photos for BikeBus and Resources** — currently Unsplash URLs.
13. **Manifesto.jsx mega phrases were translated for PT/EN/ES/FR** but supporting copy in some other sections (Solution bullets, AppCTA bullets, etc.) is only PT — i18n audit needed when copy stabilizes.

### Known bugs / quirks

- `vite.config.js` includes `@base44/vite-plugin` which emits a warning at every build: `[base44] Proxy not enabled (VITE_BASE44_APP_BASE_URL not set)`. Harmless. Will resolve once env vars are set.
- Initial GitHub push failed once with a transient HTTP 500 — retry worked. If it happens again, just `git push` again.
- The `manifesto.jsx` mega phrases hardcode the period inside the `<em>` (e.g. `<em>crianças.</em>`) on purpose — keeps the period from orphaning to a new line when the keyword wraps tight.

---

## 6. How Paulo (the founder you'll be talking to) works

### Communication style

- **Mix of PT-PT, ES, EN.** Casual. Will switch mid-sentence.
- **Feedback comes as feeling**, not specs. Examples from this project:
  - *"se ve sucio"* (about the navbar over the hero)
  - *"vibe-coded"* (about a flat data section)
  - *"el amarillo está en demasiados lugares — perdió fuerza"*
  - *"no tiene drama, no tiene ritmo"*
  - *"sensación: poster de A24, no maqueta de revista"*
  - *"bro que mierda haces"* (when glass cards covered the photos — also: when you mess up, just fix it; do not over-explain)
- **Translate feeling → CSS yourself.** He's not going to give you `border-radius: 0` — he'll say *"que se sienta institucional, sin cantos redondeados"*.
- **Iteration cadence:** screenshot from his laptop → quick verbal feedback → fix → redeploy. Often 4-6 cycles per visual decision. Don't try to land it in one shot.
- **References he uses:** Apple keynote, Stripe, A24 posters, Monocle Magazine, The New Yorker, Steidl photo books, FOAAP dashboard (his own product). When he names a reference, look it up if you don't know it — that's the target feeling.

### Ground rules he's stated (or yelled)

1. **Do every step yourself; never hand him commands to run.** *"haz todo tu"* / *"Execute, don't hand back commands."*
2. **NEVER mix this side-project repo with FOAAP** at `/Users/baphomet/Downloads/foaap-new` or `/foaap-core`. They are separate.
3. When something needs his decision (a credential, a content choice, a yes/no on scope), **stop and ask** — don't assume.
4. **The image he wants you to use lands in `~/Downloads/`** with a slightly weird name (e.g. `foo.webp.png` because Runway exports oddly). Always `find` for it; he won't paste a path.
5. **Photos are not allowed to be obscured.** Glass cards, dark overlays > 70%, blur effects on imagery — all bad. Photos are the dramatic device, treat them with respect.
6. **Yellow is sacred.** Show up only in counters and (in manifesto) the one keyword per principle. Anywhere else is dilution.
7. **No gray for muted text.** Always bone with opacity.
8. **Don't ship vibe-coded.** He'll know. Add weight, hierarchy, real data, real images.

### Tools / agents available locally

- macOS, no Homebrew, no `gh`, no global Vercel.
- Use `npx -y` for everything: `vercel@latest`, `cwebp-bin@7`, `lighthouse@latest`.
- Git over HTTPS via Keychain. Account: `imperiobaal69-web`.

---

## 7. Recommended next steps (ordered)

### If you have one session

1. **Fix the "09" duplication** in `Resources.jsx` — change to `10 / 13`.
2. **Pull real INE freguesia population** for the remaining 50+ freguesias and re-run `python3 /tmp/build_geojson.py` (or write a fresh script). Drop the `populacao_fonte: 'estimated'` flags in `_todo`.
3. **Source code split** — `vite.config.js` `build.rollupOptions.output.manualChunks` to put `react-leaflet/leaflet` and `framer-motion` in separate chunks. Should drop the initial bundle by ~250KB.

### If you have a few sessions

4. **Help Paulo set up DNS + Vercel-GitHub auto-deploy.** Walk him through GoDaddy and Vercel UIs once; after that the loop is auto.
5. **Build the BikeBus active-route registry**. Right now city-route counts are hardcoded in `BikeBus.jsx`. Move to a JSON file in `/public/data/bike-bus.json` and start collecting real route data — this also feeds the map's "Bike Bus" layer.
6. **Replace Unsplash placeholders** with real Porto photography — Paulo will source from Runway again.

### Strategy work for Ricardo's side

7. **Wikimedia escudo lazy-load** for the map side panel — visible improvement, low effort, makes the map feel "alive".
8. **Calendly + dossier PDF** for the Municipalities section — institutional credibility.
9. **Real i18n translations** by a native speaker once copy stabilizes; my drafts were best-effort but a Portuguese editor should review.

---

## 8. Useful commands

### Dev / build

```bash
cd /Users/baphomet/Downloads/bicis-sapiens-web

npm install                # ~10s, 624 packages.
npm run dev                # Vite dev server, default :5173. Hot-reloads.
npm run build              # → dist/. Build emits "[base44] Proxy not enabled"
                           #   warning — ignore.
npm run lint               # ESLint, --quiet (errors only).

# Image to WebP (no Homebrew needed):
npx -y cwebp-bin@7 cwebp -q 85 input.png -o output.webp

# Lighthouse against deployed URL:
npx -y lighthouse@latest https://bicis-sapiens-web.vercel.app/ \
  --quiet --chrome-flags="--headless=new" \
  --output=json --output-path=/tmp/lh.json
```

### Git / deploy

```bash
# Standard cycle:
git add .
git commit -m "feat(scope): short description

Longer wrap-at-72 explanation if helpful.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
git push origin main

# Vercel: auto-deploy is NOT wired up yet (see TODO §5.3).
# Always manually:
cd /Users/baphomet/Downloads/bicis-sapiens-web
npx -y vercel@latest --prod --yes

# Smoke test live:
curl -sI https://bicis-sapiens-web.vercel.app/      # expect HTTP/2 200
curl -sL https://bicis-sapiens-web.vercel.app/ | head -25
```

### Vercel project ops

```bash
npx -y vercel@latest whoami                                 # imperiobaal69-5594
npx -y vercel@latest env add VITE_BASE44_APP_ID            # interactive
npx -y vercel@latest env ls
npx -y vercel@latest domains add www.bicisapiens.org
```

### File patterns

```bash
# Find images Paulo just dropped in Downloads:
ls -lt /Users/baphomet/Downloads/ | head -10

# Audit hardcoded color hex codes that should be tokens:
grep -rnE "#[0-9A-Fa-f]{6}" src/components/landing/ | grep -v "FFD60A\|003399\|FAFAF7\|0A0A0A"

# Audit yellow leaks (should ONLY be in counters):
grep -rn "text-accent\|bg-accent\|#FFD60A" src/components/ | grep -v "tracking-widest uppercase text-accent"
```

---

## 9. Client-facing context (Ricardo's vision)

### Thesis in one sentence

Cities can be **100% cyclable without new infrastructure** — through respect,
smart signage, and cultural change.

### Why no new infrastructure

Porto is a hard case — slopes up to 30°, narrow streets, *calçada portuguesa*
hostile to bikes and pedestrians, descents up to 55 km/h on slippery stone.
Building a Copenhagen-style ciclovia network is impractical and expensive.
Bicis Sapiens argues the better lever is **respect + organization + Bike Bus
programs** — and proves it with the kind of dense, replicable model that other
cities (Lisbon, Barcelona, Mexico City) can borrow.

### The 3 Cs (philosophy) vs 3 Pilares (action)

The brief gives both, and the implemented site uses the action set:

| Philosophy (the 3 Cs) | Action pillars (what's on the site) |
|---|---|
| **Conciencia** — awareness | **Informação** (info / open data) |
| **Información** — information | **Consenso** (consensus across groups) |
| **Consenso** — consensus | **Proposta** (concrete proposals to câmaras) |

`Solution3Cs.jsx` renders the action pillars. The "Conciencia" frame is more
movement-language than UI-language and lives in copy, not in a dedicated
section.

### Audiences and primary CTAs

| Audience | What the site offers | Action |
|---|---|---|
| Famílias | Manifesto + community + forum | Adherir, opinar, votar |
| Escolas | Bike Bus program | Solicitar adhesão |
| Câmaras municipais | Dossier + Calendly + directory | Descarregar dossier, agendar reunião |
| Cidadãos | Manifesto + community | Firmar, partilhar |
| Voluntários | "Como ajudar" | Registar-se |
| Investidores / doadores | Apoyar la app + transparência | Donar / waitlist |

The single primary visitor CTA across the site: **"Súmate a la comunidade e
da tua opinião."** That's what `JoinModal` captures.

### Voice rules (quick reference)

- ✅ espacio público · autonomia · comunidade · consenso · cidade humana · ciclável · mobilidade humana · evidência
- ❌ biker · ciclista hardcore · eco-friendly · sustentável (cliché) · revolução · batalla · guerra
- **PT-PT first**, then EN/ES/FR.
- Editorial, not panphlet. Short declarative sentences. Adult readers. Civic, **not anti-car**.
- No emojis in official copy. No centered body text (left-align is the editorial padrão). No auto-play carousels.

### Pending from the client (use elegant placeholders + TODO)

1. Legal structure (foundation, NIPC pending)
2. Project email (suggested: `hola@bicisapiens.org`)
3. Social handles (IG / LinkedIn / TikTok — to create)
4. Real Porto data sources (INE, Câmara do Porto)
5. Advisor list (urbanists, pediatricians, transport engineers)
6. Real testimonials
7. Ally logos (associações, ONGs, escolas adheridas)
8. Educational guide content (4 PDFs)
9. Founder photo (Ricardo will shoot)

When in doubt about a missing piece: **placeholder elegantly, mark as TODO in
code, and surface to Paulo** rather than fabricating content.

---

## End notes

If you want to verify the site is actually in the state described above:

```bash
git -C /Users/baphomet/Downloads/bicis-sapiens-web log --oneline -10
```

Recent commits at time of writing (latest first):
```
d312a56 revert(manifesto): drop glass cards, clean yellow italic keyword
224437b feat: clean header gradient + real i18n switching + glass panels
1c03f45 fix(manifesto): shrink keyword marker so it doesn't crash line above
4c34272 feat(manifesto): A24 poster panels (image bg, text on top)
8b72674 feat(manifesto): real spreads with 3 Runway images
0f81c7a style(manifesto): keyword as solid marker, supporting as Inter
b550acc fix(manifesto): keyword marker no longer crashes adjacent lines
d7f9532 feat(manifesto): cinematic editorial rewrite
9c69c39 feat(visual): cream cards on obsidian + yellow only in counters
3ebbf09 feat(hero): full-bleed bridge image background + ticker
```

Good luck. Move fast, treat the photos with respect, keep yellow scarce.

— *Claude (handing off · session of May 2026)*
