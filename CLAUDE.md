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
- **Light Theme by Default:** Crisp ice-blue canvas (`#f8fbff`) with pure white cards (`#ffffff`) and subtle secondary tiles (`#f1f6fd`). Dark mode activates only when `localStorage.getItem('mm-theme') === 'dark'`, applied synchronously in `<head>` before paint to prevent theme flash.
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

## SEO & AI-Search (GEO / AEO) Checklist
- **Canonical URL Hygiene:** Always link to home with `href="/"` or `href="/#section"`—never `index.html`—so crawlers only ever index `https://moodmeter.cend.in/`.
- **Head Parity Across All 4 Pages:**
  - Single `<h1>` and strict `h1 -> h2 -> h3` hierarchy; single `<header>`, `<nav aria-label="Primary">`, `<main id="main-content">` (with skip link), and `<footer>` landmarks.
  - `<title>` <= 60 chars and `<meta name="description">` ~140–155 chars, unique per page.
  - Self-referencing `<link rel="canonical">`, `hreflang="en"`, and `hreflang="x-default"`.
  - `<meta name="robots" content="index, follow, max-image-preview:large, max-snippet:-1, max-video-preview:-1">` and `<meta name="apple-itunes-app" content="app-id=391278676">`.
  - Complete Open Graph (`og:type`, `og:site_name`, `og:locale`, `og:url`, `og:title`, `og:description`, `og:image`, `og:image:type`, `og:image:width`, `og:image:height`, `og:image:alt`) and Twitter/X `summary_large_image` tags (`twitter:url`, `twitter:title`, `twitter:description`, `twitter:image`, `twitter:image:alt`).
- **Structured Data (Schema.org JSON-LD):**
  - `index.html` publishes the canonical `@graph`: `Organization` (`#org` with 512x512 logo and App Store / Google Play `sameAs` links), `WebSite` (`#website`), `WebPage` (`#webpage`), `MobileApplication` (`#app`), and `FAQPage` (`#faq-schema`).
  - **Keep `FAQPage` JSON-LD answers synced word-for-word with the visible `#faq` cards.**
  - Subpages reference `#website`, `#org`, and `#app` by `@id` and include a `BreadcrumbList`.
- **Freshness & Crawl Files:** Update `<lastmod>` in `sitemap.xml`, `<time datetime="...">` / `dateModified` on edited legal pages, and `llms.txt` whenever product facts or URLs change.
