# Bicis Sapiens — Design Handoff

> **For the next agent / future-me.** This is the canonical visual-
> system reference for the project, layered on top of the broader
> `/HANDOFF.md` at the repo root (which covers stack, deploy, infra).

---

## §00.5 / Reality check (added at save-time)

A few small corrections to the handoff text below so it matches what's
actually in the repo when you read this:

| Claimed in §00 | Actual repo state |
|---|---|
| "Next.js, deployed on Vercel" | **Vite + React** (not Next.js), deployed on Vercel. The i18n is a custom React Context in `src/lib/i18n.jsx` — not next-intl. |
| §02 — "STATUS: HTML reference written, prompt provided to user. User to verify implementation." | **Already implemented** — see `src/components/landing/DataDashboard.jsx` and commit `5332c26` (`feat(data): dark-mode reskin + double parallel bars`). The two-parallel-bars pattern, italic blue deltas, yellow kicker line, AMS/PAR/CPH short names, and `numbers.*` i18n namespace are live. Default `compareCity` is `'Amsterdam'` so the thesis renders on first paint. |
| §03 — "Design direction finalized. Prompt provided. User to implement next." | **Already implemented** — see `src/components/landing/Solution3Cs.jsx` and commit `d10fb7b` (`feat(solution): dark editorial reskin, no cards, no icons`). Roman numerals I./II./III. in yellow, role sub-labels per pillar, declarative period on titles, signature block centered at the bottom, no animations. |
| §04 — "HTML reference written and shared with user (Option B — graphic activism direction)." | **Diverged.** The current `Interstitial.jsx` was rebuilt earlier in the session into a different design — full-screen 100/120vh panel with starfield + nebula, inline-SVG STOP/bike signs, and a sculpted 10-layer text-shadow 3D word ("JUNTOS"). The "Option B graphic activism" CSS-built signs + outlined italic big-word direction documented below has **not** been applied. If you take this on, decide with the user whether to migrate the current build to Option B or keep what's there. The i18n keys live under `pausa.*` (kicker / signs.stop / headline.line1 / headline.line2 / quote.line1-3 / bigword). |
| `/docs/porto-numbers-dark.html` | Does not exist in the repo. The §02 reference HTML was pasted inline in the session prompt — not saved as a file. The implemented version in `DataDashboard.jsx` is the canonical record. |
| `/docs/uma-pausa-redesign.html` | Does not exist in the repo. Same situation — the spec lived in prompt text. |
| "Translation system not specified but inferred… URL-based routing." | Not URL-based. Language is held in React Context (`src/lib/i18n.jsx`) — `useState('pt')` default — and toggled by the PT/EN/ES/FR pills in the navbar via `setLang(...)`. No routing involvement. |

Other context that may help: the broader handoff at `/HANDOFF.md`
(commit `9136488`) covers folder layout, image pipeline (cwebp-bin via
npx, no brew, no global vercel CLI), GitHub auth via Keychain, the
manual `npx -y vercel@latest --prod --yes` deploy loop, and the rules
on yellow scarcity / no glass cards / no gray for muted text.

The remainder of this document is Paulo's design-system handoff, kept
verbatim as the canonical visual reference.

---

# BICIS SAPIENS — DESIGN HANDOFF
## §00 / Context

Project: bicis-sapiens-web (Next.js, deployed on Vercel)
URL: bicis-sapiens-web.vercel.app
Type: Civic movement website / manifesto
Languages: PT (primary), EN, ES, FR
Numbering convention: §01 / 13, §02 / 13, etc. (13 total sections)

The site has a strong dark editorial visual system already established
in some sections (the map section "A cidade vista por freguesia", the
"Uma Pausa" section). Other sections are inconsistent — they use light
backgrounds, SaaS-style cards, generic icons, or clip-art.

Goal of this redesign pass: bring every section in line with the dark
editorial system so the whole site reads as one coherent civic
publication, not a Frankenstein of templates.

---

## §01 / Visual System (canonical)

These are the design tokens. Apply them consistently across all sections.

### Colors

- Background:           #0a0a0a   (solid black, no gradients)
- Yellow accent:        #d4a017   (kickers, Roman numerals, "warning" stats)
- Blue accent:          #1d4ed8   (italic highlight words, primary bars,
                                   active selector pills)
- White text:           #ffffff
- Muted text:           rgba(255,255,255,0.65-0.7)
- Faint text:           rgba(255,255,255,0.4)
- Hairline:             rgba(255,255,255,0.15)
- Surface (rare):       rgba(255,255,255,0.04) with 0.5px border
                        rgba(255,255,255,0.1)

DO NOT introduce: red (#993C1D was rejected), pastels, gradients,
drop shadows, glows. The system is flat.

### Typography

- Serif (headlines, numbers, italic accents): Fraunces from Google Fonts
  if not already loaded. Otherwise use the existing serif (Tiempos,
  EB Garamond, or similar — check what "vista" in the map section uses).
- Sans (body, labels, UI): Inter or whatever's already loaded.
- Weights: 400 for serif headlines, 500 for kicker labels and stat values.
  Never 600 or 700 — heavy weights destroy the editorial feel.
- Sentence case everywhere except kickers and tracked uppercase labels.

### Patterns

- Kickers: small horizontal yellow line (32px × 1px) + uppercase tracked
  yellow text (11px, letter-spacing 0.3em). Used at the top of every
  section to anchor the section number/name.
- Italic blue accent words: serif italic in #1d4ed8, used inside white
  serif headlines for emphasis. Pattern from "vista" in §05.
- Hairlines: 0.5px solid rgba(255,255,255,0.15) for section dividers
  and internal structure.
- No cards with backgrounds in most sections. Whitespace and hairlines
  do the structural work. Exception: the hero stat in §02 uses a
  rgba(255,255,255,0.04) surface.

---

## §02 / Sections completed in this session

### §02 — "Os dados que faltam ao debate público" (Porto in Numbers)

STATUS: HTML reference written, prompt for Claude Code provided to user.
        User to verify implementation in their next session.

What changed:
- Background flipped from beige/white to dark
- Headline now has italic blue accent on the last word ("público.")
- Added comparison thesis with dynamic deltas in italic blue
- Hero stat card for green space (5.2 m²) with WHO 9m² annotation in
  yellow
- Mini bar charts: TWO PARALLEL BARS (Porto in blue, comparison city in
  muted white) instead of single-bar-with-tick. Cars metric uses yellow
  because Porto is worse on that one.
- City selector pills: active = blue background, inactive = transparent
  with white border
- Short city names for bar labels: AMS / PAR / CPH

Reference HTML: see /docs/porto-numbers-dark.html (if user saved it).

i18n keys namespaced under `numbers.*` — translations provided in PT/EN/ES/FR.

Helper functions needed:
- computeDelta(portoVal, compareVal, inverse) → returns "25%" or "5.3×"
- computeBarPair(portoVal, compareVal) → returns {portoPct, comparePct}
- computeWhoDelta(portoGreenM2) → returns int percentage below 9m²

DO NOT touch the data layer or the existing city-toggle state — only
the presentation layer.

### §03 — "A Nossa Solução" / "O que oferecemos" (3 pillars)

STATUS: Design direction finalized. Prompt for Claude Code provided
        in plain-English form (no HTML). User to implement next.

What changed:
- Removed pastel-blue square icons (book / people / document)
- Removed ghost numerals (1, 2, 3) behind cards
- Removed vertical blue divider between cards
- Removed square blue bullet points
- Removed card backgrounds and borders
- Removed status indicators (Live / Q2 2026 / Draft) — were over-promising
- Removed "Conhece a equipa →" link — was a vague CTA
- Repositioned the orphaned italic quote ("Sem construir nova
  infraestrutura. Com respeito e organização.") as the header tagline
  in the top-right, paired with the headline

What was added:
- Headline "O que oferecemos." with "oferecemos" in italic blue
- Each pillar gets a Roman numeral in yellow serif (I. II. III.)
  + a tiny uppercase role sub-label ("o ponto de partida" / "a ponte"
  / "a entrega")
- Pillar titles end with a period for declarative weight
  ("Informação." / "Consenso." / "Proposta.")
- Deliverables are unstyled lists (no bullets, no statuses)
- Signature block at the bottom: "Assinado · Porto, 2026" small caps
  + "A equipa Bicis Sapiens" in italic serif, centered, no avatars

i18n keys namespaced under `solution.*` — translations needed in PT/EN/ES/FR.

The full prompt for Claude Code was provided in plain English (no HTML).

---

## §03 / Sections still to redesign

These are the sections we identified during the session but haven't
finished yet:

### §04 — "Uma Pausa" (the manifesto pause section)

STATUS: HTML reference written and shared with user (Option B —
        graphic activism direction).

What's planned:
- Removes starry-night background, clip-art STOP sign, clip-art bike
  sign, 3D-shadowed "JUNTOS" big word
- Adds custom CSS-built signs: white "PARE/STOP/ALTO" with black border
  and blue offset shadow, slight -3deg rotation; blue bike sign with
  inline SVG bike icon, +2deg rotation
- Adds outlined italic serif big word ("juntos" / "together" /
  "ensemble") with text-stroke, no fill, no drop shadow
- Adds full-bleed crosswalk divider at the bottom
- i18n: PT "PARE", EN "STOP", ES "ALTO", FR "STOP"
- i18n: PT/ES "juntos", EN "together", FR "ensemble" — French is
  longer, may need per-language size override

Reference HTML: see /docs/uma-pausa-redesign.html (if user saved it).

### §05 — Map section "A cidade vista por freguesia"

STATUS: Already on-system. No redesign needed unless user requests it.
        This section is the canonical reference for the visual system.

### Other sections (§01 hero, §06+ community/donate/etc.)

STATUS: Not yet reviewed. User to share screenshots when ready.

---

## §04 / Decisions made (do not relitigate)

These were debated and resolved during the session:

1. NO red color in the palette. The original mockup used #993C1D
   terracotta — rejected because it doesn't appear elsewhere on the site.
   Yellow #d4a017 plays the "warning" role instead.

2. NO icons in §03 pillars. No Tabler, Lucide, Heroicons, or any SVG
   icons. Roman numerals + role labels do the structural work.

3. NO status indicators on deliverables. Was tested and rejected by user
   — too SaaS roadmap, also creates public commitments to dates.

4. NO team avatars yet. Signature is text-only ("A equipa Bicis Sapiens"
   in italic serif). Avatars can come later when there's a real team page.

5. KEEP the original three Cs naming: Informação / Consenso / Proposta.
   We considered renaming to Conhecimento / Consenso / Concretização —
   rejected. The user wanted to drop the "3 Cs" branding entirely and
   just present them as three pillars under "O que oferecemos."

6. KEEP the section numbering format: §02 / 13, §03 / 13, etc. with the
   section program name following ("§03 / 13 — A Nossa Solução"). This
   replaces "Issue 01" framing — the user prefers the numbered manifesto
   convention.

7. "Issue" / section labels: keep section program names translated,
   but the §XX / 13 numbering format is universal across languages.

8. Bar chart pattern in §02: TWO PARALLEL BARS, not single-bar-with-tick.
   The two-bar pattern is clearer at a glance and matches NYT/Bloomberg
   editorial conventions.

---

## §05 / Working principles for next session

When approaching a new section:

1. First identify what visual pattern the section currently uses and
   whether it conflicts with the canonical system in §01 above. Common
   conflicts: light backgrounds, pastel surfaces, generic icons, SaaS
   chrome (cards with hover, status pills, pricing tiers), clip-art.

2. Translate to the system: dark background, yellow kicker, italic blue
   accent in the headline, hairlines instead of borders, typography over
   ornament.

3. Strip elements rather than add. Most sections look "vibe-coded"
   because they have too much going on. The dark editorial system gets
   stronger as you remove things.

4. Preserve existing logic. Don't refactor data layers, state management,
   i18n setup, or routing. These redesigns are RESKINS, not rewrites.

5. Always provide i18n strings for PT, EN, ES, FR. The user reads
   Portuguese natively — they'll catch awkward translations there
   especially.

6. Section numbering: every new kicker uses §XX / 13 — Section Name
   format. Yellow line + yellow uppercase tracked text.

7. Don't add features the user didn't ask for: no scroll animations,
   no hover effects beyond accessibility minimums, no extra CTAs, no
   "Read more" links to pages that don't exist yet.

---

## §06 / Open questions for next session

- Are sections §01 (hero), §06 (community), §07 (donate) on-system or
  do they need redesign? User to share screenshots.
- Does a real team page exist or should it stay aspirational? Affects
  whether §03 should re-add a "Conhece a equipa →" link later.
- Confirm the exact serif font in production. Fraunces was assumed but
  user should verify what's loaded.

---

## §07 / File locations referenced

- /docs/porto-numbers-dark.html — §02 reference HTML
- /docs/uma-pausa-redesign.html — §04 reference HTML
- §03 reference is plain-English prompt only (no HTML file written)

User stack: Next.js with i18n already configured. Translation system
not specified but inferred from the site supporting PT/EN/ES/FR via
URL-based routing.
