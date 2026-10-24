# MASTER PROMPT — Landing Page Rebuild & Checklist Compliance Mission

> Copy everything below this line into your AI coding agent (Cursor, Claude Code, Copilot Workspace, etc.).

---

## 1. ROLE & MISSION

You are a senior product engineer + motion designer. Your mission is to take the existing landing page in this repository, audit it against the CHECKLIST in Section 8, fix every failing item, and ship a production-grade landing page that matches the quality bar of the best design-inspiration sites on the internet (reference benchmark: vibrant.design).

You must NOT ask questions. You must NOT leave TODO comments. You must NOT skip verification. Every change you make must be justified by a checklist item ID (e.g. "fixes A2, G3") so the work is traceable.

Work in this order: AUDIT → TOKENS → LAYOUT → COPY → MOTION → ACCESSIBILITY → PERFORMANCE → VERIFY. Do not start a later phase until the earlier phase passes its inline checks.

---

## 2. BENCHMARK DATA — What "great" looks like (measured live from vibrant.design)

These are real, extracted production values from vibrant.design (fetched from its live HTML + compiled CSS). Use them as your calibration target, not as assets to copy.

### 2.1 Brand & color system
- Page background (light theme): `#f7f7f7` — used consistently as the page canvas; cards sit on `#ffffff`.
- Primary interactive blue: `#3074FE`; deep accent blue: `#1838f8`; component fill blue (e.g. animated check fill): `#2563eb`.
- Full neutral ramp is tokenized in OKLCH (shadcn-style), light theme: background `oklch(100% 0 0)`, foreground `oklch(14.5% 0 0)`, muted-foreground gray, card white, border `oklch(92.2% 0 0)`.
- Dark theme equivalents: background `oklch(14.5% 0 0)` (≈ #0a0a0a), card `oklch(20.5% 0 0)` (≈ #171717), border `oklch(100% 0 0 / .1)` (white at 10% alpha).
- Lesson: ONE saturated brand hue + a strict neutral ramp. No random accent soup. The blue appears ONLY on primary actions, links, and live/status indicators.

### 2.2 Typography
- Single family: **Inter** (`--font-inter`), fallback `ui-sans-serif, system-ui, sans-serif`. No second display font anywhere.
- Tight tracking on headlines (negative letter-spacing ≈ -0.02em to -0.04em at hero sizes), normal tracking below 20px.
- H1 ≤ 10 words. Measured example: "Design inspiration from the best of the internet" (8 words). Sub-headline: "Curated websites, sections, launch videos and UI shots." (9 words).
- Self-hosted, subset (latin only), `font-display: swap`, preload the weights actually used (typically 400/500/600/700).

### 2.3 Geometry
- Radius scale (shadcn-derived): `--radius: 0.625rem` (10px); `md = 8px`, `lg = 10px`, `xl = 14px`, `2xl = 18px` (1.8 × base). Cards use lg/xl; pills and badges use fully rounded.
- Blur scale: `xs 4px / sm 8px / md 12px / xl 24px` — used for frosted-glass nav and overlays (`backdrop-filter` with matching border: white/10 in dark, black/5 in light).
- Media cards are strictly 16/9 (`--aspect-video`) so masonry rows align without CLS.
- Spacing runs on the Tailwind 4px base grid; section vertical rhythm ≈ 96–128px desktop, 64px mobile.

### 2.4 Motion system (this is the crown jewel — replicate the *system*, not the animations)
Easing library (exact cubic-beziers extracted from production CSS):
| Token | Curve | Use |
|---|---|---|
| ease-out-quint (default) | `cubic-bezier(.22, 1, .36, 1)` | entrances, drawers, most UI motion |
| ease-out-quart | `cubic-bezier(.23, 1, .32, 1)` | panels, cards |
| ease-out-expo | `cubic-bezier(.16, 1, .3, 1)` | hero reveals, big surfaces |
| ease-in-out-quint | `cubic-bezier(.32, .72, 0, 1)` | iOS-like state changes, layout shifts |
| ease-in-out-cubic | `cubic-bezier(.66, 0, .34, 1)` / `(.33, 0, .67, 1)` | loops, position swaps |
| spring overshoot | `cubic-bezier(.34, 1.96, .94, 1)` | success pops, badge bounces |
| soft overshoot | `cubic-bezier(.34, 1.45, .64, 1)` / `(.22, 1.32, .36, 1)` | buttons pressed/released |

Duration + micro-interaction spec (from their animated "task check" component — a model micro-interaction):
- Arc draw: `0.3s` ease-out-quint with a 0.64 progress lead.
- Fill wipe: `0.22s` `cubic-bezier(.34, 1.35, .64, 1)`, fill `#2563eb`.
- Success pop: `0.22s` spring overshoot, scale to `1.09`.
- Revert: `0.2s` ease-out-quint.
- Ring: 22px hit area, ring width = size × 2.5/22, ring color `rgba(0,0,0,.10)` light / `rgba(255,255,255,.12)` dark, blur `0.5px`.

Keyframe patterns in production CSS:
- `shimmer`: `background-position: 200% 0 → -200% 0` (skeleton + text shimmer).
- `orb-wave`: opacity rest → 1 at 28%, scale `1 → 1.18 → 1` (breathing ambient orbs).
- `enter/exit` (tw-animate style): animate opacity + `translate3d` + `scale3d` + `rotate` from CSS vars — i.e. every entrance is a single transform/opacity animation.
- Ambient loops: `ping 1s`, `pulse 2s cubic-bezier(.4,0,.6,1)`, `spin 1s linear`.
- Content feedback loops: `text-shimmer`, `streaming-word-fade`, `digit-pop-in`, `scan-line` — all short (0.2–0.9s), all transform/opacity/background-position only.

Motion laws to enforce:
- Entrances: 300–600ms, distance ≤ 24px, stagger 40–80ms per item, capped at ~8 items then reveal rest instantly.
- Hovers: 150–250ms; scale ≤ 1.03; shadows/opacity shift, never layout properties.
- Ambient loops: ≥ 2s period, subtle (opacity/scale ≤ 1.18), always paused under `prefers-reduced-motion`.
- Only `transform`, `opacity`, `filter`, `background-position` animate. Animating `width/height/top/left` is a checklist failure.

### 2.5 Page architecture (measured section order)
1. Announcement strip: small pill row ("Daily inspiration" + rotating "Latest" tags).
2. Sticky nav: wordmark left, category links center, search + CTA right. Frosted on scroll (`blur-sm/md` + hairline border).
3. Hero: 8–10 word H1, 1-line subhead, primary CTA ("Surprise me") + secondary CTA ("Explore Moodboards"), immediately followed by proof (the product itself visible).
4. Live product grid: masonry of named media cards (16/9), each with a one-line label — the content IS the social proof.
5. Sponsored slot between grid rows (single, clearly labeled).
6. Category rails: Hero, Features, Bento, Pricing, CTA, Logo Cloud, Testimonials, FAQ, Changelog, Footer, How it works, Integrations, Stats, Careers, Blogs, Docs, Login/Signup, 404.
7. Footer: link columns (Newsletter, Contact, Sponsor, Privacy, Terms, Licence, Refunds) + oversized wordmark as a closing visual anchor.
- SEO baseline: `<title>` "Vibrant Design - Best of the Internet" (38 chars), meta description "Curated website, section, launch-video and UI design inspiration from the best of the internet." (97 chars).

---

## 3. PHASE 1 — AUDIT (do this first, print the results)

1. Run the dev server and build. Record: build success, bundle size, warnings.
2. Crawl every route reachable from the landing page. List broken links (fix: C9).
3. Grep the codebase for: `!important`, inline hex colors outside the token file, `transition: all`, `width`/`height` in `@keyframes`, fixed pixel font sizes outside the scale, `console.log`, unused exports.
4. Screenshot the page at 360, 768, 1024, 1440, 1920 widths. Note horizontal overflow, wrapping disasters, touch-target violations.
5. Score the current page against every checklist item in Section 8 and print a table: `ID | Item | Status (PASS/FAIL/PARTIAL) | Evidence | Fix plan`.

Print this audit table before changing any code. Then fix in the phase order given in Section 1, re-printing the table whenever you complete a phase.

---

## 4. PHASE 2 — DESIGN TOKENS (single source of truth)

Create/update `styles/tokens.css` (or equivalent) so that NOTHING hardcodes values anymore:

```css
:root {
  /* color — one brand hue + neutrals only */
  --bg-page: #f7f7f7;
  --bg-card: #ffffff;
  --bg-page-dark: oklch(14.5% 0 0);
  --bg-card-dark: oklch(20.5% 0 0);
  --brand: #3074FE;
  --brand-deep: #1838f8;
  --brand-fill: #2563eb;
  --fg: oklch(14.5% 0 0);
  --fg-muted: oklch(55.6% 0 0);
  --border: oklch(92.2% 0 0);
  --border-dark: oklch(100% 0 0 / .1);

  /* geometry */
  --radius: 10px;                    /* md 8 / lg 10 / xl 14 / 2xl 18 */
  --radius-md: calc(var(--radius) - 2px);
  --radius-lg: var(--radius);
  --radius-xl: calc(var(--radius) + 4px);
  --radius-2xl: calc(var(--radius) * 1.8);
  --blur-xs: 4px; --blur-sm: 8px; --blur-md: 12px; --blur-xl: 24px;

  /* type */
  --font-sans: 'Inter', ui-sans-serif, system-ui, sans-serif;

  /* motion — verbatim from benchmark */
  --ease-out-quint: cubic-bezier(.22, 1, .36, 1);
  --ease-out-quart: cubic-bezier(.23, 1, .32, 1);
  --ease-out-expo: cubic-bezier(.16, 1, .3, 1);
  --ease-inout-quint: cubic-bezier(.32, .72, 0, 1);
  --ease-inout-cubic: cubic-bezier(.66, 0, .34, 1);
  --ease-spring: cubic-bezier(.34, 1.96, .94, 1);
  --ease-spring-soft: cubic-bezier(.34, 1.45, .64, 1);
  --dur-hover: 180ms;
  --dur-entrance: 500ms;
  --dur-ambient: 2400ms;
  --stagger: 60ms;
}
```

Rules: delete every hex value found outside this file and replace with a var (fixes F2). Wire the font stack through the framework's font loader with subsetting (fixes A6). Keep both light and dark themes valid — components must never assume one theme.

---

## 5. PHASE 3 — LAYOUT & SECTION SPEC (fix against this exact blueprint)

Rebuild/repair the sections in this order, each meeting the stated spec. Mobile-first CSS; breakpoints 360 / 768 / 1024 / 1440.

1. **Announcement strip** — max height 40px, dismissible, rotating pill messages (crossfade 300ms ease-out-quint). No layout shift when it appears (reserve space or mount before hydration).
2. **Sticky nav** — 64px tall; transparent at top; on scroll >8px: `backdrop-filter: blur(8px)`, bg `color-mix(in oklab, var(--bg-page) 80%, transparent)`, hairline bottom border. Active link gets the brand color; all links keyboard reachable with visible focus (fixes B3).
3. **Hero** — H1 ≤ 10 words with negative tracking; subhead ≤ 20 words; primary CTA with brand fill + white text (contrast ≥ 4.5:1) and secondary ghost CTA; entrance animation: opacity 0→1 + translateY 16px→0, 500ms ease-out-expo, staggered headline→subhead→CTAs at 60ms. Below the CTAs, show the product immediately (grid/preview). No hero carousel. No autoplaying unmuted video (fixes A4, C1).
4. **Product/proof grid** — masonry or uniform grid, all media 16/9 with explicit `width`/`height` or `aspect-ratio` (fixes A2/CLS), `loading="lazy"` + `decoding="async"` for below-fold images, AVIF/WebP with fallback, hover: card scale 1.02 + shadow, 180ms ease-out-quart, label crossfades in. Stagger first-row reveal at 60ms/item.
5. **Social-proof rail** — logo cloud in grayscale `opacity .6`, hover to full color, marquee optional but must pause on hover and under reduced motion (fixes B6, G5).
6. **Feature section** — 2–3 column grid desktop, 1 column mobile; icon + title + 2–3 sentence body (never 1 sentence — content depth rule); optional bento variant with one 2×2 card.
7. **Testimonials** — real-looking quotes with name/role/avatar; 3 visible desktop; no fake 5-star walls.
8. **Pricing** — 3 tiers, middle tier visually elevated (ring in brand color), each feature row an honest checkmark (reuse the micro-check spec from 2.4: draw .3s, fill .22s, pop .22s scale 1.09 — it's a delightful detail that costs ~10 lines).
9. **FAQ** — `<details>`/accordion, one open at a time, chevron rotate 200ms ease-inout-quint, height animation via `grid-template-rows: 0fr→1fr` trick (no JS height measuring).
10. **Final CTA** — one sentence + one button; brand background with subtle `orb-wave` breathing orbs (opacity .4→1, scale 1→1.18, 2.4s loop) behind it.
11. **Footer** — 4 link columns + oversized wordmark (clamp font-size 96–200px, `opacity .9`, tight tracking). Newsletter input with inline validation and success state using the spring-pop micro-interaction.

Global layout laws: no horizontal scroll at any breakpoint (fixes D1); max content width 1200–1280px; section padding `clamp(64px, 10vw, 128px)` vertical; consistent 8pt spacing grid; every interactive element ≥ 44×44px touch target (fixes D3).

---

## 6. PHASE 4 — COPY REWRITE (do not keep vague filler)

Apply these measured formulas to all landing copy (fixes E1–E5):
- H1: benefit-led, concrete, ≤ 10 words. Pattern: "[What users get] from/without [the friction]". Bad: "Welcome to our platform". Good (benchmark): "Design inspiration from the best of the internet".
- Subhead: one line ≤ 20 words naming the 3–4 concrete things they get (benchmark names exactly four: websites, sections, launch videos, UI shots).
- CTAs: verb-first, outcome-flavored. Benchmark uses playful-but-clear: "Surprise me", "Explore Moodboards". Avoid "Submit", "Learn more", "Click here".
- Every claim gets a number, name, or logo near it (social proof: counts, customer names, or product previews). No "trusted by thousands" without proof.
- FAQ answers 2–4 sentences each, written to pre-empt objections (pricing, data, cancellation, support).
- Meta title ≤ 60 chars, meta description 90–155 chars — write both to the measured benchmark pattern (see 2.5).
- Kill every instance of: "cutting-edge", "seamless", "revolutionary", "empower", "unleash", "next-generation", "in today's fast-paced world".

---

## 7. PHASE 5 — MOTION IMPLEMENTATION

Implement a tiny motion utility (CSS classes + one IntersectionObserver hook), then apply everywhere:

```css
.reveal { opacity: 0; transform: translateY(16px); }
.reveal.is-in { opacity: 1; transform: none;
  transition: opacity var(--dur-entrance) var(--ease-out-expo),
              transform var(--dur-entrance) var(--ease-out-expo);
  transition-delay: calc(var(--i, 0) * var(--stagger)); }

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: .01ms !important; animation-iteration-count: 1 !important;
    transition-duration: .01ms !important; scroll-behavior: auto !important;
  }
  .reveal { opacity: 1; transform: none; }
}
```

Requirements (fixes G1–G6): observer threshold 0.15, unobserve after reveal; stagger via `--i` custom property; hover/focus transitions 150–250ms on the token durations; skeletons use the `shimmer` keyframe (background-position 200%→-200%, ≥1.5s loop) on `oklch` neutral gradients; success states use `--ease-spring`; ambient loops only in the final CTA and status dots, `animation-play-state: paused` under reduced motion; zero `transition: all` anywhere (grep to confirm).

---

## 8. THE CHECKLIST — all gates must PASS before you are done

**A. Performance**
- A1 Lighthouse mobile Performance ≥ 90 (CLI run, paste the number).
- A2 CLS < 0.05: every image/video/iframe/embed has explicit dimensions or `aspect-ratio`; fonts preloaded; no late-mounting banners.
- A3 LCP < 2.5s simulated: hero media preloaded (`fetchpriority="high"`), everything below fold lazy.
- A4 No autoplaying audio; hero video (if any) muted + `playsinline` + poster.
- A5 Images AVIF/WebP with sized `srcset`; total landing JS ≤ 200KB gzipped; no unused dependencies (run depcheck or equivalent).
- A6 Fonts: one family, subset, `font-display: swap`, ≤ 4 weights.

**B. Accessibility (WCAG 2.2 AA)**
- B1 Body text contrast ≥ 4.5:1, large text/UI ≥ 3:1 — verify with axe or contrast script, paste results.
- B2 Full keyboard operability: tab order logical, Enter/Space activate controls, no traps.
- B3 Visible focus on every interactive element (`:focus-visible` ring, 2px brand or 1.5px offset ring).
- B4 Semantic landmarks: `header/nav/main/section/footer`, one `h1`, logical `h2/h3`; images have meaningful alt (decorative = `alt=""`).
- B5 `prefers-reduced-motion` fully honored (Section 7 block present and tested).
- B6 Marquees/carousels pause on hover and focus; ARIA labels on icon-only buttons.

**C. SEO & meta**
- C1 Title ≤ 60 chars; description 90–155 chars; canonical; OG + Twitter card tags with a 1200×630 image.
- C2 `lang` attribute correct; JSON-LD `Organization`/`WebSite` (or `Product`) present.
- C3 Valid sitemap.xml + robots.txt; no `noindex` on the landing page.
- C4 All headings hierarchical; content readable in HTML before JS (SSR/SSG).

**D. Responsive & device**
- D1 Zero horizontal scroll at 360/768/1024/1440/1920.
- D2 Nav collapses to a functional mobile menu (focus-trapped, ESC closes, body scroll locked).
- D3 Tap targets ≥ 44×44px; form inputs ≥ 16px font (no iOS zoom).
- D4 Tested at device-pixel-ratio 3 for image sharpness (srcset covers it).

**E. Content quality**
- E1 H1 ≤ 10 words, benefit-led. E2 Subhead ≤ 20 words, concrete nouns. E3 One primary CTA above the fold.
- E4 Social proof visible without scrolling past 2 screens. E5 Zero banned filler words (list in Phase 4). E6 No lorem ipsum, no placeholder imagery with watermarks.

**F. Visual system**
- F1 Exactly one font family; F2 all colors via tokens (grep audit = 0 raw hex outside tokens.css); F3 radius from the scale only; F4 spacing on the 8pt grid; F5 max one brand hue + neutrals (status colors allowed for semantics only).

**G. Motion**
- G1 Entrances 300–600ms with 40–80ms stagger; G2 hovers 150–250ms, scale ≤ 1.03; G3 only transform/opacity/filter/background-position animated; G4 `transition: all` count = 0; G5 all loops pause under reduced motion; G6 nothing animates on first paint before LCP element.

**H. Conversion & trust**
- H1 Forms validate inline with helpful messages; success uses the spring micro-interaction. H2 Zero dead links (link checker run). H3 Pricing/testimonial claims consistent between sections. H4 Privacy/Terms reachable from footer.

**I. Engineering**
- I1 `build` and `typecheck` pass with zero errors/warnings. I2 No console errors/warnings in production mode. I3 No secrets in client bundle. I4 Components under ~200 lines, repeated UI extracted. I5 Grep confirms: 0 `console.log`, 0 `!important` (except reduced-motion block), 0 TODO.

---

## 9. VERIFICATION PROTOCOL (run everything, paste evidence)

Execute in order and include the outputs in your final report:
1. `build` + `typecheck` (paste exit codes).
2. Lighthouse CLI, mobile profile, landing route (paste all four category scores).
3. axe/automated a11y scan (paste violation count before → after; target: 0 serious/critical).
4. Link checker across the page (paste: 0 broken).
5. Screenshot script at 360/768/1024/1440/1920 (confirm no overflow via `document.documentElement.scrollWidth === clientWidth`).
6. Keyboard-only walkthrough: log the tab path and confirm focus visibility on every stop.
7. Bundle report: JS/CSS gzip sizes before → after.
8. Reduced-motion emulation pass: confirm zero animations run.
9. Re-print the full checklist table from Phase 1 with final PASS/FAIL per ID.

## 10. DEFINITION OF DONE

Done = every checklist ID in Section 8 is PASS with pasted evidence, the design-token file is the only source of hex/radius/duration values, the motion system matches the benchmark curves in 2.4, and a reviewer can open the final report (audit table before → after + verification outputs) and reproduce every claim. If any item cannot pass, state the blocker, the checklist ID, and your proposed workaround — never silently skip.

---

*Benchmark data source: vibrant.design live production HTML + compiled CSS (`globals-*.css`), extracted 2026-10. Token values, easing curves, keyframes, copy, and section order are measured facts, not guesses.*
