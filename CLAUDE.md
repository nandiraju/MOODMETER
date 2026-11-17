# Mood Meter Website (`CLAUDE.md`)

Marketing and legal website for the **Mood Meter** mobile application (iOS and Android). Deployed on GitHub Pages (`gh-pages` branch) at `https://moodmeter.cend.in/` via `CNAME`.

## Stack, Build, and Structure
- **Pages:** `index.html` (landing page), `privacy_policy.html`, `terms_of_service.html`, `contact.html`, and `404.html` (`noindex, follow`, root-absolute URLs for GitHub Pages fallback). Each page is static HTML with its own inline JSON-LD (on indexed pages), theme-bootstrap script, and page-specific stylesheet.
- **Precompiled Tailwind CSS v4:** Source stylesheets live in `src/css/{index,privacy_policy,terms_of_service,contact,404}.css` (`@import "tailwindcss" source(none);` + `@source "../../<page>.html";`). Compiled and minified output lives in `assets/css/*.css` and **must be committed to git** because GitHub Pages serves static files directly.
  - Rebuild all stylesheets after changing any HTML classes or CSS rules: `npm run build:css`
  - Watch during local edits: `npm run watch:css` (does **not** cache-bust; run `build:css` before committing).
  - `build:css` runs `scripts/build-css.mjs`: compiles each page, then stamps a content hash onto that page's stylesheet link (`assets/css/<page>.css?v=<hash>`) so browsers and the CDN never serve stale CSS. Adding a page means adding it to `PAGES` in that script.
  - Do **not** edit `assets/css/*.css` by hand, and do **not** reintroduce the runtime `@tailwindcss/browser` script tag.
- **Icons:** Ionicons 7.1.0 (`<ion-icon>`) loaded at the end of `<body>` (`module` + `nomodule`) with `<link rel="preconnect" href="https://unpkg.com" crossorigin>`. **Strict rule: never use emojis or Unicode dingbats anywhere in UI markup, CSS, JS, or docs.**
- **Typography:** Google Fonts `Outfit` (600–900, headings) and `Inter` (400–700, body) loaded with `display=swap` and `preconnect` hints.
- **Discovery & SEO Files:** `robots.txt` (welcomes search and AI crawlers), `sitemap.xml` (canonical URLs + Google Image Sitemap entries), `llms.txt` (structured AI/LLM fact sheet), `site.webmanifest` (PWA & site icon metadata).

## Assets
- `app_screens/01-mood-picker.webp` through `05-custom-moods.webp`: Official 1080x1920 WebP app screenshots (`screen_1.webp`–`screen_4.webp` are legacy duplicates).
- `app_screens/apple_store.svg`, `app_screens/android_store.svg`: Store download badges.
- `assets/moods/*.svg`: 20 expressive mood SVGs (~52 KB total) powering the `#vibes` horizontal ticker. Footer attribution to *Emoticon Set Pack by Smashicons on Flaticon* is required on every page.
- `assets/images/`:
  - `app-icon.webp` (256x256): Nav/CTA/footer app icon.
  - `favicon-48x48.png` (48x48), `icon-192x192.png` (192x192), `icon-512x512.png` (512x512): Multiples of 48px required for Google Search favicon indexing, Schema.org `Organization.logo`, and `site.webmanifest`.
  - `favicon-32x32.png` (32x32), `icon-128x128-1.png` (128x128), `apple-touch-icon.png` (180x180).
  - `og-image.png` / `og-image.webp` (1200x630): Open Graph and Twitter/X social card banner.

## Design System & Layout Conventions
- **Dark Theme by Default:** Every page ships `<html lang="en" class="dark">` (so no-JS visitors and crawlers get dark too). A synchronous `<head>` script removes `dark` only when `localStorage['mm-theme'] === 'light'` (the visitor chose light via the toggle), and sets the single `<meta name="theme-color">` to `#0f172a` (dark) or `#f8fbff` (light); the toggle updates both and saves `mm-theme`. `localStorage` access is wrapped in `try/catch`. `site.webmanifest` `background_color` is `#0f172a`.
- **Light Theme (opt-in):** Crisp ice-blue canvas (`#f8fbff`) with pure white cards (`#ffffff`) and subtle secondary tiles (`#f1f6fd`).
- **Tailwind v4 Class-Based Dark Variant:** Every source CSS file declares `@custom-variant dark (&:where(.dark, .dark *));`.
- **Dark Theme ("Liquid Glass"):** In `.dark`, `.ambient` renders three fixed blurred color orbs (`#4f46e5`, `#0ea5e9`, `#db2777` at `z-index: -1`) behind translucent gradient surfaces:
  - `.dark .app-card`: 145deg white-alpha gradient, `backdrop-filter: blur(26px) saturate(170%)`, 1px translucent border, and inset specular highlights.
  - `.dark .subtle-tile`: `backdrop-filter: blur(14px) saturate(150%)` with subtle top highlight.
  - `.dark .nav-glass`: `rgba(15, 23, 42, 0.45)` with `backdrop-filter: blur(24px) saturate(170%)`.
  - Dark body text uses `#aab6cb` (`--text-main`) and `#ffffff` (`--text-heading`) for comfortable contrast.
- **Footer Theme Toggle:** The light/dark theme toggle (`#theme-toggle` with `<ion-icon id="theme-toggle-icon">`) lives as a circular icon button at the end of the footer navigation links on all four pages—never in the top header. Keep `ion-icon:not(.hidden) { display: inline-block; }` intact.
- **Plumb-Line Symmetry & Card Geometry:**
  - All sections, headers, and footers align to a single `max-w-6xl mx-auto px-6` container.
  - Multi-column card grids use `gap-6 items-stretch` with `h-full` on `.app-card` (`rounded-3xl p-8`) so sibling cards always match height.
  - Every button, badge, and interactive pill uses `rounded-full`.
- **Container-Scaled iPhone 16 Pro Mockups:**
  - Pure CSS `.iphone`, `.iphone-controls`, `.iphone-screen`, `.iphone-status`, and `.iphone-island` use `container-type: inline-size` and `cqw` units (`aspect-ratio: 71.5 / 149.6`) so frame bezels, hardware buttons, and Dynamic Island scale proportionally at any width (`max-w-[292px]` in hero, `max-w-[300px]` in 3-col feature row, `max-w-[210px]` in 2-col wide cards).
  - `.iphone-screen img` uses `clip-path: inset(5% 0 2.5%)` to trim the source screenshot status bar and home indicator cleanly.
  - Do **not** place an extra wrapper stage card behind `.iphone` mockups; let the phone sit directly inside `.app-card` to maximize screen legibility.
- **Shiny Animated Borders (`@property --shine-angle`, `shine-spin` keyframes):**
  - Selected hero pill (`.screen-tab[aria-selected="true"]`, plus a `.dark` variant so it outranks `.dark .app-card`): indigo `padding-box` fill over a conic-gradient `border-box` layer, spun every 2.8 s, plus a one-shot `pill-select-glow` ring on selection.
  - Store badges (`a.store-badge`, hero + final download CTA): each link holds two `aria-hidden` spans after the `<img>`. `.badge-ring` is a 2.5px border-only ring (`mask … exclude`, `inset:-1px`) laid over the SVG's own 1px border; `.badge-glow` is an `8px`-blurred halo behind the badge. Each clips an oversized (130%, square) conic-gradient `::before` spun with **`transform: rotate()`** (`badge-shine-rotate`, 12 s). **Don't animate `--shine-angle` on pseudo-elements:** some Safari versions never run custom-property animations on `::before`/`::after` (the pill is fine because it animates the property on the element itself). Badge `<img>`s are `display:block`, `width="168" height="56"` (both SVGs are 3:1). Under `prefers-reduced-motion` both freeze.
- **Hero Autoplay:** pills auto-advance every 3.8 s via the same `activateScreenTab()` a click uses. Pauses on hover, keyboard focus, when the phone is <35% visible, and when the tab is hidden; a trusted click/keypress holds the screen for 10 s (`USER_IDLE`). Disabled under `prefers-reduced-motion`. Remaining screenshots preload when the phone first scrolls into view.
- **Anti-Scraper Email & Bot Protection (Mandatory):**
  - **Never** place any raw email address or literal `m-a-i-l-t-o` URI scheme in HTML, JSON-LD, CSS, JS string literals, `llms.txt`, or markdown files served from the repo.
  - Support email reveal/launch uses shifted character-code arrays decoded only inside `e.isTrusted` click/submit handlers, rendered visually via CSS `::before { content: attr(data-u) "\0040" attr(data-d); }` (`.obf-addr`), paired with the offscreen `#website-url` honeypot field on `contact.html`.
- **Factual Integrity:** Never invent user counts, star ratings, `aggregateRating` schema, or fake testimonials. Use verified product terms from the companion app (`Capture Vibe`, `Custom Vibes` / `Custom Vibemojis`, `Current Vibe Streak`, `Vibe Power Gauge`, `Intensity Trend`, `Hourly Activity`).

## Performance Optimizations
- **Zero Runtime CSS Compilation:** Replacing `@tailwindcss/browser` (~100+ KB render-blocking JS) with precompiled, minified per-page stylesheets (`28–43 KB` raw, ~6–8 KB brotli) eliminates FOUC, main-thread CSS compilation, and third-party CSS CDN dependency.
- **LCP Prioritization:** `index.html` preloads `app_screens/01-mood-picker.webp` (`<link rel="preload" as="image" type="image/webp" fetchpriority="high">`) and sets `fetchpriority="high" decoding="sync"` without `loading="lazy"` on `#hero-screen-img`.
- **Zero CLS & Lazy Media:** Every `<img>` declares explicit `width` and `height`. Below-the-fold screenshots and badges use `loading="lazy" decoding="async"`.
- **Hero Screen Cross-Fade (WAAPI):** Switching tabs runs a compositor-friendly Web Animations API fade (`opacity`), awaits `heroImg.decode()` before fading back in, guards rapid clicks with a monotonic `swapToken`, supports WAI-ARIA `tablist` Left/Right/Home/End keys, and swaps instantaneously under `prefers-reduced-motion: reduce`.
- **60fps Auto-Scrolling Mood Ticker (`#vibes`):**
  - Renders a curated set of 20 lightweight SVGs (`assets/moods/*.svg`) once in HTML (`alt=""` next to visible mood labels to avoid duplicate screen-reader speech).
  - Runtime JS clones `.ticker-set` (`aria-hidden="true" data-clone`) only `Math.ceil(viewportWidth / setWidth) + 1` times and animates `.ticker-track` via a single GPU-composited `translate3d` WAAPI timeline (`55px/s`).
  - **No `backdrop-filter` on moving elements:** `.ticker .ticker-tile` explicitly disables `backdrop-filter` (`none`) and uses a flat translucent gradient in dark mode so the browser never re-blurs moving layers each frame.
  - Uses `contain: layout paint style` and CSS `mask-image` edge fades.
  - Automatically pauses via `IntersectionObserver` when scrolled offscreen, on `document.visibilitychange`, and on pointer hover or keyboard focus; preserves playback progress across debounced (`150ms`) window resizes; degrades to a static scrollable strip (`[data-static]`) under `prefers-reduced-motion: reduce`.
- **Non-JS & Reduced-Motion Resilience:** `<noscript>` forces `.reveal { opacity: 1 !important; transform: none !important; }` so crawlers and non-JS browsers see all content immediately, and `@media (prefers-reduced-motion: reduce)` collapses transitions/animations site-wide.

## SEO & AI-Search (GEO / AEO): Rules for This Site
- **Canonical URL Hygiene:** Always link to home with `href="/"` or `href="/#section"`, never `index.html`, so crawlers only ever index `https://moodmeter.cend.in/`. GitHub Pages also answers `/index.html`, `/index`, `/contact`; the self-referencing canonicals handle those duplicates.
- **Head Parity Across All Indexable Pages:**
  - Single `<h1>` that carries the page's main keyword, and strict `h1 -> h2 -> h3` hierarchy; single `<header>`, `<nav aria-label="Primary">`, `<main id="main-content">` (with skip link), and `<footer>` landmarks.
  - `<title>` <= 60 chars and `<meta name="description">` ~140–155 chars, unique per page.
  - Self-referencing `<link rel="canonical">`, `hreflang="en"`, and `hreflang="x-default"`.
  - `<meta name="robots" content="index, follow, max-image-preview:large, max-snippet:-1, max-video-preview:-1">` and `<meta name="apple-itunes-app" content="app-id=391278676">`.
  - Complete Open Graph (`og:type`, `og:site_name`, `og:locale`, `og:url`, `og:title`, `og:description`, `og:image`, `og:image:type`, `og:image:width`, `og:image:height`, `og:image:alt`) and Twitter/X `summary_large_image` tags (`twitter:url`, `twitter:title`, `twitter:description`, `twitter:image`, `twitter:image:alt`).
  - `<link rel="manifest" href="site.webmanifest">`, 32px and 48px PNG favicons, 180px apple-touch-icon.
  - `404.html` is `noindex, follow`, uses root-absolute URLs, and is excluded from the sitemap.
- **Structured Data (Schema.org JSON-LD):**
  - `index.html` publishes the canonical `@graph`: `Organization` (`#org` with 512x512 logo and App Store / Google Play `sameAs` links), `WebSite` (`#website`), `WebPage` (`#webpage`), `MobileApplication` (`#app`), and `FAQPage` (`#faq-schema`).
  - **Keep `FAQPage` JSON-LD answers synced word-for-word with the visible `#faq` cards.** (FAQ rich results no longer show for normal sites; keep the block for AI/answer engines.)
  - No `softwareVersion` (iOS/Android versions differ and go stale). No `aggregateRating`/`review` unless taken from real store data **and** shown on the page (US App Store had 0 ratings as of 2026-11-17), so no software rich result for now; that is accepted.
  - Subpages reference `#website`, `#org`, and `#app` by `@id` and include a `BreadcrumbList`.
- **Crawl & Discovery Files:** `robots.txt` = `User-agent: * Allow: /` + named groups for AI crawlers (GPTBot, OAI-SearchBot, ChatGPT-User, ClaudeBot, Claude-SearchBot, Claude-User, PerplexityBot, Google-Extended, Applebot-Extended, Amazonbot, DuckAssistBot, Meta-ExternalAgent) + `Sitemap:` line. Named groups do **not** inherit rules from `*` (RFC 9309), so any future `Disallow` must be repeated in each group. `sitemap.xml` lists the 4 indexable pages with image entries (screenshots, OG image, 512 icon). `llms.txt` is a fact sheet; Google ignores it, it costs nothing.
- **Freshness:** Update a page's `<lastmod>` in `sitemap.xml` only when that page changes, plus `<time datetime="...">` / `dateModified` on edited legal pages, and `llms.txt` whenever product facts or URLs change.
- **Analytics (GA4 `G-HBMC3XJK4X`, dedicated MoodMeter property; never the Cend.in stream `G-84VRLCP0H7`):** every page `<head>` loads `assets/js/analytics.js` **synchronously** and then `<script async src="https://www.googletagmanager.com/gtag/js?id=G-HBMC3XJK4X">` (keep that order). Consent Mode v2 in `analytics.js`: region default denies everything for the EEA (EU 27 + IS, LI, NO), GB and CH; the global default grants `analytics_storage`; `ad_storage`, `ad_user_data`, `ad_personalization` are always denied. A stored choice (`localStorage["mm-consent"]`) is replayed as `consent update`. The accept/decline banner shows only when no choice is stored **and** the browser time zone is an EEA/UK/CH zone (GitHub Pages cannot geolocate; mismatches fail safe because Google applies the region default by IP). `window.mmConsent.open()` reopens it (privacy policy "Cookie settings" button). `store_click` (`store: app_store | google_play`) fires on any App Store / Google Play link (key event in GA4). Visible Google Play buttons carry `&referrer=utm_source%3Dmoodmeter_site%26utm_medium%3Dwebsite%26utm_campaign%3Dlanding`; App Store links stay plain until an App Store Connect campaign link exists. JSON-LD and `llms.txt` keep the clean store URLs. Privacy policy section `#website-analytics` states the website uses Google Analytics with cookies and the **app** collects no data; all "no third-party analytics/trackers" claims are scoped to the app. Change consent defaults and that wording in the same release. When testing locally, block `googletagmanager.com`/`google-analytics.com` so localhost hits never reach the property.

## SEO Status Log (moodmeter.cend.in)
- **2026-11-17 done:** shorter titles/descriptions; keyword line in the hero H1; `/` home links; Organization `sameAs`; subpage `BreadcrumbList` + `@id` links; OG/Twitter image alts; image sitemap; web manifest + 48/192/512 icons; 404 page; removed stale `softwareVersion`; precompiled Tailwind with hashed CSS URLs; dark theme default (`class="dark"` in HTML).
- **2026-11-17 Analytics:** GA4 `G-HBMC3XJK4X` live on all pages with Consent Mode v2 (analytics granted globally, denied by default in EEA/UK/CH with a banner; ads always denied), `store_click` key event, Play referrer UTM, privacy policy updated (website uses GA cookies; app collects nothing).
- **2026-11-17 Search Console:** `cend.in` verified as a **domain property** (covers this subdomain); `sitemap.xml` submitted and accepted (4 pages discovered); indexing requested for the homepage.
- **2026-11-17 audit (claude-seo method), health 78/100:** Technical 91, Content 62, On-page 78, Schema 86, Performance 76, AI search 78, Images 84. No critical issues.
- **Open items (priority order):**
  1. Name the developer/publisher (matching the store "Seller" field) in the footer, privacy policy, `Organization.legalName`, and `llms.txt`; add a plain-text support role address in `contactPoint`. Disambiguate from Yale RULER's "Mood Meter" (wording to be confirmed by the owner).
  2. FAQ additions that answer real objections, mirrored in JSON-LD: works offline, no account needed, not a medical app, what happens when changing phones/backups (verify app behaviour first).
  3. Keyword-led H2s (keep slogans as eyebrow text); contact H1 "Mood Meter Support & Contact".
  4. Self-host fonts (2 variable woff2 instead of 8 Google files) and replace Ionicons with a local SVG sprite (removes two third-party origins; makes "no IP addresses shared" true for the website).
  5. Reduce dark-mode `backdrop-filter` cost on touch devices (about 70 blurred cards); keep glass on desktop.
  6. Responsive screenshot variants (`srcset`, 540w/810w) incl. the hero preload; compress `og-image.png` (236 KB) to under 100 KB; delete unused `app_screens/screen_1-4.webp` and `og-image.webp`; maskable 512 manifest icon; `favicon.ico`.
  7. Schema polish: screenshots as `ImageObject`s, `applicationSubCategory`, `offers.availability`, ContactPage `mainEntity` as an `@id` reference.
  8. Off-site: set this URL as the website on both store listings; earn genuine store ratings; list on AlternativeTo / Product Hunt / privacy-app directories with the same name and developer everywhere.
  9. Content investments: data-protection explainer page, factual comparison page (vs Daylio / How We Feel / Bearable), small guide/blog cluster with Article schema and a named author.

## New-Site SEO Playbook (reuse for any new static site)
Work through these in order. Verify each step with a command or a real browser, not by assumption.

1. **Decide the basics before writing pages.**
   - Pick one canonical origin (`https://`, with or without `www`) and stick to it in every URL, canonical, sitemap and schema.
   - Write down the 3–5 queries each page must win and the one fact-based sentence that defines the product ("X is a free, ... for ..."). That sentence goes in the first paragraph, the meta description, schema `description` and `llms.txt`.
   - Agree the brand spelling (and any alternate spelling) and the real legal/developer name. Check whether another well-known product shares the name.
2. **Build fast by default.**
   - Static HTML with all content in the raw HTML (no client-side rendering for text).
   - Precompile CSS (e.g. Tailwind CLI) instead of runtime CDN builds; stamp a content hash on asset URLs (`?v=<hash>`) via a build script so caches never serve stale files.
   - Self-host fonts (variable woff2, `font-display: swap`, preload the heading font) and icons (inline SVG sprite). Every third-party origin costs speed and leaks visitor IPs.
   - Identify the LCP element: preload it, `fetchpriority="high"`, never lazy. Everything below the fold: `loading="lazy" decoding="async"`.
   - Every `<img>` gets `width`/`height` matching its real aspect ratio; serve WebP/AVIF with `srcset`/`sizes` sized to the rendered width.
   - Animate only `transform`/`opacity`; never put `backdrop-filter` on moving or numerous elements; pause infinite animations offscreen; honour `prefers-reduced-motion`.
   - Put the stylesheet link before large JSON-LD blocks in `<head>`.
3. **Head tags on every indexable page:** unique `<title>` (<= 60 chars, keyword first, brand last), unique meta description (140–155 chars), self-referencing absolute canonical, `robots` (`index, follow, max-image-preview:large`), `hreflang` if multilingual, theme-color, favicons (48px multiple), apple-touch-icon, manifest, full Open Graph + Twitter card with a 1200x630 image (< 100 KB) and alt text. Skip `meta keywords` (ignored).
4. **Page structure:** one H1 containing the main keyword; H2s that say what the section is about (slogans go in eyebrow text, not headings); question-shaped headings where people ask questions; landmarks (`header`, `nav aria-label`, `main id` + skip link, `footer`); descriptive link text; `alt=""` for decorative images and for logos next to the same visible text.
5. **Structured data:** one `@graph` on the home page with stable `@id`s (`#org`, `#website`, `#webpage`, plus the product type: `MobileApplication`/`SoftwareApplication`, `Product`, `LocalBusiness`, etc.). Subpages reference those `@id`s and add `BreadcrumbList`. Only mark up what is visible on the page. Never invent ratings, reviews, user counts or versions. Validate with Google's Rich Results Test and a JSON parser.
6. **Trust (E-E-A-T):** name who makes the product (legal name, country), give a real contact route, date legal pages with `<time>`, and add a disclaimer for health/finance-adjacent topics. Make privacy claims precise and true for both the product **and** the website (fonts, CDNs and analytics count).
7. **Crawl files:**
   - `robots.txt`: allow everything you want indexed, add named groups for AI crawlers you welcome, include the `Sitemap:` line, and remember named groups don't inherit `*` rules.
   - `sitemap.xml`: canonical URLs only (no 404/noindex pages), real per-page `<lastmod>`, image entries for key images. `changefreq`/`priority` are ignored by Google.
   - `llms.txt`: short fact sheet (H1, blockquote summary, key facts, links). Optional, cheap.
   - A real `404.html` (returns 404, `noindex, follow`, root-absolute links).
   - On GitHub Pages, keep build/source files out of the public site (Jekyll `_config.yml` `exclude:` list, or deploy only the built folder).
8. **Analytics and privacy:** create a dedicated GA4 property per site (free; up to 2,000 per account); set retention to 14 months; load with Consent Mode v2 default-denied unless you add a consent banner; track the main conversion (e.g. store clicks) as a key event; update the privacy policy in the same release.
9. **Launch checks (run every time):**
   - Live status codes: pages 200, unknown path 404, `http` -> `https` single 301.
   - Deployed files match the repo (hash compare), CSS loads, no console errors or failed requests, no horizontal scroll at 390px and 1280px, both themes.
   - Titles/descriptions lengths, one H1 per page, canonical present, JSON-LD parses, FAQ schema matches visible text, no broken local `src`/`href`, sitemap valid XML.
10. **Get indexed:** verify in Google Search Console (a **domain** property via DNS covers all subdomains), submit the sitemap, request indexing for the homepage; do the same in Bing Webmaster Tools (can import from Search Console); link Search Console to GA4.
11. **Off-site:** point app-store/marketplace listings and social profiles at the site, use the identical name and developer everywhere, earn genuine reviews, and list in relevant directories.
12. **Keep it fresh:** re-run the launch checks after every deploy, re-audit quarterly, update `lastmod`/dates/`llms.txt` only when content really changes, and add focused pages (how it works, comparisons, guides) to reach more queries.
