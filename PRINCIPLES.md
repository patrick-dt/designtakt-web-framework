# Web Principles & Checklist

Single source of truth. Shared build rules + pre-launch checklist.

1. Follow **Dev Principles** while building.
2. Open your **stack addendum** for framework- and CMS-specific rules.
3. Before launch, walk **All projects**, then only the checklist sections that match your stack.

---

## Stack addenda

| Stack | Addendum | Checklists to walk |
|-------|----------|-------------------|
| Astro only | [stacks/astro.md](stacks/astro.md) | All projects → [Astro](stacks/astro.md) |
| Astro + Sanity | [stacks/astro-sanity.md](stacks/astro-sanity.md) | All projects → [CMS](stacks/cms.md) → [Sanity](stacks/sanity.md) → [Astro](stacks/astro.md) |
| Astro + Supabase | [stacks/astro-supabase.md](stacks/astro-supabase.md) | All projects → [Supabase](stacks/supabase.md) → [Astro](stacks/astro.md) |
| Next.js + Sanity | [stacks/next-sanity.md](stacks/next-sanity.md) | All projects → [CMS](stacks/cms.md) → [Sanity](stacks/sanity.md) → [Next.js](stacks/next.md) |
| Next.js + Supabase | [stacks/next-supabase.md](stacks/next-supabase.md) | All projects → [Supabase](stacks/supabase.md) → [Next.js](stacks/next.md) |

---

## Dev Principles

- Fluid sizing. Prefer relative units over fixed.
- Size in `rem`. Never absolute `px` (borders/outlines ok).
- A11y: semantic HTML, contrast, [`focus-visible`](https://developer.mozilla.org/en-US/docs/Web/CSS/:focus-visible).
- One `h1` per page; heading order without skips.
- Interactive elements: keyboard reachable + visible focus.
- Optimize images (size, format, lazy-load below fold).
- No ugly / auto-generated class names (e.g. `DivBlock234`).
- Prefer reuse over one-off markup.
- No em dashes in copy. Use commas, colons, or separate sentences instead.

### Local preview

- Extra `dev` servers are not a framework need. They appear because Cursor and agents treat `npm run dev` as a default step without checking whether this project is already served.
- HMR is enough. A running process at the default URL picks up file changes on its own. A second server does nothing useful except occupy the next ports (`4322`, `4323`, … once Astro's `4321` is taken).
- Before starting anything, check existing terminals and open the URL already serving this project.
- Defaults (unless the project overrides them): Astro [`http://localhost:4321/`](http://localhost:4321/), Next.js [`http://localhost:3000/`](http://localhost:3000/), Sanity Studio [`http://localhost:3333/`](http://localhost:3333/).
- Same port can be two processes: `localhost` often binds IPv6 (`::1`), `--host 127.0.0.1` binds IPv4. They are not the same server. Don't treat two `:4321` listeners as duplicates; don't kill one to "clean up."
- Open the host the process bound to. Printed URL or `--host 127.0.0.1` → `http://127.0.0.1:<port>/`; otherwise `http://localhost:<port>/`. A response on `localhost:<port>` is not proof *this* project is already served — match cwd/command, then reuse that URL.
- If the default port belongs to a *different* project, do not kill that server. Use the next free port, or the URL this project's already-running server printed.
- Start `dev` only when nothing is serving this project.

### Markup / layout

- Landmark outside, container inside: `<section>` (or `header` / `nav` / `main` / `article` / `aside` / `footer`) wraps a site-wide container, not the reverse.
- Landmarks for meaning: `<section>` only for a thematic block with its own heading. Generic wrappers stay `div`.
- One `<main>` per page. Don't wrap `header` / `footer` / `nav` in extra `<section>`s.
- Container is site-wide: one component or utility (`max-width`, horizontal padding, centering). Don't re-invent per block.
- Full-bleed bands (hero, background) stay full width; put the container only around the inner content.

### Content / data files

- Central `data.ts` (or `content.ts` / similar) is for site-wide, rarely-edited facts only: business address, social links, org name, phone.
- Page copy stays in the page or component: headlines, body, FAQs, testimonials, nav labels, section text.
- Content that editors will change often belongs in the CMS, not a TypeScript file.

### Interactions

- Links are links: use `<a>` (or framework `<Link>`) for navigation, not `<button>` or `<div>`.
- Hit targets: visual target < 24px → expand to ≥ 24px; on mobile ≥ 44px.
- Never disable browser zoom (no `maximum-scale=1` viewport hacks).
- Modals, menus, dialogs: trap focus while open; return focus on close ([WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)).
- Semantics before ARIA: prefer native elements (`button`, `label`, `table`) over `aria-*`.
- Icon-only controls need an accessible name (`aria-label` or visible text).
- Status cues beyond color: errors, success, and warnings include text, not color alone.
- Sticky headers, overlays, and fixed UI must not obscure the focused element.

### CSS / Tailwind

- Utility-first ([Tailwind](https://tailwindcss.com/docs)). Custom CSS only when utilities can't.
- Colors via design tokens / CSS vars (e.g. `--color-*`). No raw hex in components.
- Spacing via scale / tokens; no magic numbers.
- Typography via project `text-*` / type scale. No ad-hoc font sizes.
- Fonts: load once (e.g. `@font-face`); use designated font utilities only.
- Breakpoints: `md:` (768px), `lg:` (1024px) unless project defines otherwise.
- Pair `:hover` with `:focus-visible` on interactive elements.
- Avoid `!important` unless overriding third-party.
- Keep class lists readable; extract repeated patterns into components.
- [`scroll-margin-top`](https://developer.mozilla.org/en-US/docs/Web/CSS/scroll-margin-top) on anchored headings (in-page nav / TOC links).
- `touch-action: manipulation` on tap controls (reduces double-tap zoom delay).

### Motion

Recipes: [MOTION.md](MOTION.md) (one file per pattern in [`motion/`](motion/)).

- CSS first for hover, focus, simple transitions, and sticky/stacking scroll effects (`transition`, `@keyframes`, `position: sticky`).
- No JS for motion unless CSS can't do it (timeline, scroll-driven, sequenced, interruptible).
- When JS is needed: [GSAP](https://gsap.com/docs/v3/). Use clear easing (e.g. `power2.out`, `power3.inOut`); avoid linear unless intentional.
- Prefer `autoAlpha` over `opacity` (also toggles `visibility`).
- Prefer transforms + `autoAlpha` over layout props (`top`, `height`, etc.).
- Never `transition: all`: animate only intended properties (`opacity`, `transform`, etc.).
- Use `gsap.matchMedia()` for breakpoint-specific motion.
- Kill / clean up GSAP on unmount or page leave (no orphaned tweens / ScrollTriggers).
- Respect [`prefers-reduced-motion`](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion): reduce or disable non-essential motion.

---

## Pre-Launch Checklist

Walk **All projects** on every launch. Then walk the technology sections that apply to your stack (see table above).

### All projects

#### Access
- [ ] DNS access confirmed, or contact person informed / reachable

#### Legal & Utility
- [ ] Cookies / consent banner
- [ ] Imprint
  - [ ] Link with UTM, e.g. `?utm_source=ClientName&utm_medium=referral&utm_campaign=imprint`
- [ ] Privacy policy
- [ ] 404 page

#### Content & Assets
- [ ] No placeholder / lorem / dummy links
- [ ] No em dashes in user-facing copy
- [ ] Final logos, images, copy, contact details
- [ ] Favicon (+ apple-touch if needed)
- [ ] Empty, error, and sparse states designed (not just happy path)
- [ ] Page `<title>` matches current page context
- [ ] [`theme-color`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meta/name/theme-color) meta matches page background (mobile browser chrome)

#### Forms
- [ ] All fields work (required, validation, error states)
- [ ] Success / error messaging
- [ ] Submit delivers to correct destination
- [ ] Spam protection (honeypot / captcha if needed)
- [ ] Every control has a visible `<label>` (or equivalent accessible name)
- [ ] Errors next to fields; first error focused on submit
- [ ] Mobile inputs ≥ 16px font size (avoids iOS auto-zoom)
- [ ] Paste works in all inputs (incl. OTP / codes)
- [ ] [`autocomplete`](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/autocomplete) + meaningful `name` where autofill applies
- [ ] Submit: button disabled in-flight; original label still visible
- [ ] Labels + a11y (focus, keyboard)
- [ ] Tested on mobile

#### SEO
- [ ] Image alt tags
- [ ] Unique title + description per indexable page
- [ ] [Open Graph](https://ogp.me/) image & settings
- [ ] Social share preview checked
- [ ] [Schema.org](https://schema.org/)
- [ ] Semantic tags (nav, main, section, heading, footer)
- [ ] Landmark wraps the shared site-wide container (full-bleed only when intended)
- [ ] Heading hierarchy matches outline
- [ ] Canonical URLs
- [ ] `robots.txt` + sitemap
- [ ] 301 redirects (Excel)
  - [ ] UTMs not passed through redirects
  - [ ] Google Ads redirects checked
  - [ ] No redirect chains

#### Analytics
- [ ] Tracking installed ([GA4](https://developers.google.com/analytics/devguides/collection/ga4) / [GTM](https://developers.google.com/tag-platform/tag-manager) / agreed tool)
- [ ] Consent mode / cookie gate respects choice
- [ ] Key events fire (form submit, CTA clicks, etc.)
- [ ] No double-counting
- [ ] Test in debug / preview before go-live

#### Google Search Console
- [ ] Property verified ([Google Search Console](https://search.google.com/search-console/about))
- [ ] Sitemap submitted
- [ ] No critical coverage / indexing errors
- [ ] Inspect key URLs (homepage + main landing pages)

#### Performance
- [ ] [LCP, CLS, INP](https://web.dev/vitals/) in acceptable range (mobile + desktop)
- [ ] Images sized / compressed; lazy-load below fold
- [ ] Fonts not blocking render (subset / `font-display`)
- [ ] No unused heavy scripts
- [ ] Caching / CDN as agreed

#### Links
- [ ] Footer links
- [ ] Menu links
- [ ] Deadlink check passed

#### Go-live
- [ ] HTTPS live
- [ ] Correct production domain (www vs non-www)
- [ ] No staging URLs; no leftover `noindex`
- [ ] Forms hit production endpoint (not test)

#### A11y
- [ ] Heading hierarchy
- [ ] Skip link or equivalent landmark nav
- [ ] Contrast check on key text / CTAs
- [ ] Keyboard-only walkthrough of main flows (nav, forms, modals)
- [ ] Icon-only buttons have accessible names

#### Testing
- [ ] Mobile
- [ ] Tablet / mid width
- [ ] Desktop
  - [ ] Chrome
  - [ ] Edge
  - [ ] Firefox
  - [ ] Safari
- [ ] Hover / focus states present
- [ ] Reduced-motion path verified
- [ ] No console errors on key pages

#### Handoff
- [ ] Webflow only: code fully embedded, or sandbox handed over
- [ ] Client credentials / access documented

Stack-specific checklists live in the stack addendum files (see table above).
