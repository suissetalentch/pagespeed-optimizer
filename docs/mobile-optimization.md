# Mobile Performance Optimization

Dedicated guide for optimizing Core Web Vitals on mobile devices where constraints are significantly stricter than desktop.

## Why Mobile-First?

Mobile Lighthouse testing simulates conditions that are 4-10x harder than desktop. Optimizing for mobile guarantees desktop will pass.

### Mobile vs Desktop Constraints

| Aspect | Desktop Lighthouse | Mobile Lighthouse | Impact |
|--------|-------------------|-------------------|--------|
| Network | No throttling | 4G: 1.6 Mbps, 150ms RTT | 10x slower |
| CPU | Normal | 4x slowdown | JS 4x slower |
| Viewport | 1350x940 | 412x823 | Different LCP element |
| 500KB image | ~instant | ~2.5s download | LCP failure |
| 1MB JS bundle | ~instant | 5s+ parse/execute | TBT failure |

### The Mobile-First Rule

> **If mobile passes 95+ → Desktop will automatically pass**

Mobile constraints are always stricter. Never optimize for desktop first.

## Mobile Performance Budget

### Network Budget (4G Simulation)

4G speed: ~1.6 Mbps = **200KB/second**

| Resource | Budget | Rationale |
|----------|--------|-----------|
| Total page weight | < 500KB | Ideal: < 300KB |
| **LCP image** | **< 100KB** | Critical for 2.5s target |
| Critical CSS | < 14KB | First TCP packet (congestion window) |
| JavaScript (parsed) | < 150KB | Avoid TBT issues |
| Web fonts | < 50KB | One font family max |
| Above-fold content | < 200KB | For fast FCP |

### Time Budget

| Metric | Target | Mobile Reality |
|--------|--------|----------------|
| FCP | < 1.8s | First paint visible |
| LCP | < 2.5s | Main content loaded |
| TTI | < 3.8s | Page fully interactive |
| TBT | < 200ms | No long tasks blocking |
| CLS | < 0.1 | No layout shifts |

### LCP Image Budget Calculator

```
4G download: 1.6 Mbps = 200KB/second

Time breakdown for LCP < 2.5s:
├── DNS/TCP/TLS:     ~600ms
├── Server response: ~200ms
├── HTML parse:      ~100ms
└── Remaining:       ~1.6s for LCP resource

Max safe LCP image: 1.6s × 200KB/s = 320KB
Recommended:        100KB (leaves buffer for network variance)
```

## LCP Optimization for Mobile

### Identify the LCP Element

```javascript
// Run in DevTools Console
new PerformanceObserver((list) => {
  const entries = list.getEntries();
  const lastEntry = entries[entries.length - 1];
  console.log('LCP element:', lastEntry.element);
  console.log('LCP time:', lastEntry.startTime, 'ms');
  console.log('LCP size:', lastEntry.size);
}).observe({type: 'largest-contentful-paint', buffered: true});
```

### Image LCP (Most Common - 80% of cases)

**Requirements:**
1. Size for mobile viewport first (412px width)
2. Use srcset with mobile-first breakpoints
3. Preload with `fetchpriority="high"`
4. Format: WebP/AVIF (50-80% smaller)
5. **Total size < 100KB**

```html
<!-- Preload in <head> -->
<link
  rel="preload"
  as="image"
  href="/hero-mobile.webp"
  fetchpriority="high"
  media="(max-width: 768px)"
>
<link
  rel="preload"
  as="image"
  href="/hero-desktop.webp"
  fetchpriority="high"
  media="(min-width: 769px)"
>

<!-- Mobile-first image markup -->
<img
  src="/hero-mobile.webp"
  srcset="
    /hero-400.webp 400w,
    /hero-800.webp 800w,
    /hero-1200.webp 1200w
  "
  sizes="(max-width: 640px) 100vw, 50vw"
  alt="Hero description"
  width="400"
  height="300"
  loading="eager"
  fetchpriority="high"
  decoding="sync"
>
```

### Text LCP

When text is the LCP element:

```html
<!-- Preload primary font -->
<link
  rel="preload"
  as="font"
  href="/fonts/primary.woff2"
  type="font/woff2"
  crossorigin
>

<!-- Inline critical CSS (< 14KB) -->
<style>
  /* Only above-fold styles */
  h1 { font-family: 'Primary', system-ui; font-size: 2rem; }
</style>

<!-- Font CSS with display:swap -->
<style>
  @font-face {
    font-family: 'Primary';
    src: url('/fonts/primary.woff2') format('woff2');
    font-display: swap;
  }
</style>
```

### Video LCP

```html
<!-- Poster image as LCP fallback -->
<video
  poster="/video-poster.webp"
  preload="none"
  playsinline
  muted
>
  <source src="/video.webp" type="video/webp">
  <source src="/video.mp4" type="video/mp4">
</video>

<!-- Preload the poster -->
<link rel="preload" as="image" href="/video-poster.webp" fetchpriority="high">
```

## FCP Optimization for Mobile

### Render-Blocking Resources

The biggest FCP killers on mobile:

| Resource | Problem | Solution |
|----------|---------|----------|
| External CSS | Blocks render | Inline critical CSS |
| External fonts | FOIT/FOUT | Preload + font-display: swap |
| Synchronous JS | Blocks parsing | defer or async |
| Third-party scripts | Network latency | Delay or remove |

### Critical CSS Strategy

```html
<!-- Inline critical CSS (must be < 14KB) -->
<style>
  /* Above-fold styles only */
  :root { --primary: #007bff; }
  body { margin: 0; font-family: system-ui; }
  .header { /* ... */ }
  .hero { /* ... */ }
</style>

<!-- Load full CSS asynchronously -->
<link rel="preload" href="/styles.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
<noscript><link rel="stylesheet" href="/styles.css"></noscript>
```

### Font Loading Strategy

```html
<!-- 1. Preconnect to font origin -->
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

<!-- 2. Preload critical font file -->
<link
  rel="preload"
  as="font"
  href="https://fonts.gstatic.com/s/inter/v13/font.woff2"
  type="font/woff2"
  crossorigin
>

<!-- 3. Font CSS with swap -->
<style>
  @font-face {
    font-family: 'Inter';
    src: url('...') format('woff2');
    font-display: swap; /* Critical! */
  }
</style>
```

### JavaScript Optimization

```html
<!-- Defer non-critical JS -->
<script src="/app.js" defer></script>

<!-- Async for independent scripts -->
<script src="/analytics.js" async></script>

<!-- Module scripts are deferred by default -->
<script type="module" src="/app.js"></script>
```

## Mobile Testing Methodology

### Step 1: DevTools Quick Check

1. Open DevTools (F12)
2. Toggle device toolbar (Ctrl+Shift+M)
3. Configure throttling:
   - Network: Slow 4G (or custom: 1.5 Mbps, 400ms RTT)
   - CPU: 4x slowdown
4. Select mobile device (e.g., Moto G Power)
5. Hard refresh (Ctrl+Shift+R)

### Step 2: Lighthouse Mobile Audit

```bash
# Set Chrome path
CHROME_PATH=$(find $HOME/.cache/puppeteer -name chrome -type f 2>/dev/null | head -1)

# Run mobile audit
npx lighthouse https://example.com \
  --output=json \
  --output-path=./mobile.json \
  --chrome-flags="--headless --no-sandbox"

# Parse key metrics
cat mobile.json | jq '{
  FCP: .audits["first-contentful-paint"].numericValue,
  LCP: .audits["largest-contentful-paint"].numericValue,
  TBT: .audits["total-blocking-time"].numericValue,
  CLS: .audits["cumulative-layout-shift"].numericValue,
  Performance: .categories.performance.score
}'
```

### Step 3: Analyze Network Waterfall

Key things to look for:

1. **LCP image position** - Should start loading early (top 3 requests)
2. **Render-blocking resources** - Red bars before first paint
3. **Long chains** - Resources waiting on other resources
4. **Large files** - Anything > 100KB is suspicious on mobile

### Step 4: Real Device Validation

```bash
# Chrome Remote Debugging
# 1. Enable USB debugging on Android
# 2. Connect device via USB
# 3. Open chrome://inspect in desktop Chrome
# 4. Click "inspect" on your page

# Or use WebPageTest with real devices
# https://www.webpagetest.org (select Moto G4 - 4G)
```

## Mobile-Specific Debugging

### Find What's Blocking FCP

```javascript
// DevTools Console - Find render-blocking resources
const entries = performance.getEntriesByType('resource');
entries
  .filter(e => e.renderBlockingStatus === 'blocking')
  .forEach(e => console.log('Blocking:', e.name, e.duration + 'ms'));
```

### Network Waterfall Red Flags

| Pattern | Problem | Fix |
|---------|---------|-----|
| CSS at position 1-3 | Render blocking | Inline critical CSS |
| Font before CSS ends | Blocking chain | Preload font |
| LCP image at position 10+ | Late discovery | Preload image |
| JS bundle > 200KB | Long download | Code split |
| Multiple font files | Font bloat | Use 1 weight, system fallback |

### Measure Real LCP

```javascript
// More detailed LCP debugging
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log('LCP candidate:', {
      element: entry.element,
      tagName: entry.element?.tagName,
      time: entry.startTime.toFixed(0) + 'ms',
      size: entry.size,
      url: entry.url || 'N/A'
    });
  }
}).observe({type: 'largest-contentful-paint', buffered: true});
```

## Framework-Specific Mobile Patterns

### React / Vite

```jsx
// Mobile-first image component
function HeroImage({ src, alt }) {
  return (
    <img
      src={`${src}-400.webp`}
      srcSet={`
        ${src}-400.webp 400w,
        ${src}-800.webp 800w,
        ${src}-1200.webp 1200w
      `}
      sizes="(max-width: 640px) 100vw, 50vw"
      alt={alt}
      loading="eager"
      fetchPriority="high"
      decoding="sync"
    />
  );
}
```

### Next.js

```jsx
import Image from 'next/image';

// Mobile-optimized hero
<Image
  src="/hero.jpg"
  alt="Hero"
  width={400}  // Mobile width first
  height={300}
  sizes="(max-width: 640px) 100vw, 50vw"
  priority     // Preload for LCP
  quality={75} // Reduce file size
/>
```

### Nuxt 3

```vue
<template>
  <NuxtImg
    src="/hero.jpg"
    alt="Hero"
    width="400"
    height="300"
    sizes="sm:100vw md:50vw"
    format="webp"
    quality="75"
    loading="eager"
    fetchpriority="high"
  />
</template>
```

### Laravel Blade

```blade
{{-- Preload in layout head --}}
<link
  rel="preload"
  as="image"
  href="{{ Vite::asset('resources/images/hero-mobile.webp') }}"
  fetchpriority="high"
  media="(max-width: 768px)"
>

{{-- Mobile-first hero image --}}
<img
  src="{{ Vite::asset('resources/images/hero-400.webp') }}"
  srcset="
    {{ Vite::asset('resources/images/hero-400.webp') }} 400w,
    {{ Vite::asset('resources/images/hero-800.webp') }} 800w,
    {{ Vite::asset('resources/images/hero-1200.webp') }} 1200w
  "
  sizes="(max-width: 640px) 100vw, 50vw"
  alt="Hero"
  width="400"
  height="300"
  loading="eager"
  fetchpriority="high"
>
```

## Lazy Loading Strategy

### Above-the-Fold (Mobile Viewport: ~823px height)

```html
<!-- NEVER lazy load -->
<img src="hero.webp" loading="eager" fetchpriority="high">
<img src="logo.webp" loading="eager">
```

### Below-the-Fold

```html
<!-- ALWAYS lazy load -->
<img src="product1.webp" loading="lazy" decoding="async">
<img src="product2.webp" loading="lazy" decoding="async">
```

### Calculate Fold Position

```javascript
// Mobile viewport height
const mobileViewport = 823; // Lighthouse mobile

// Check if element is above fold
function isAboveFold(element) {
  const rect = element.getBoundingClientRect();
  return rect.top < mobileViewport;
}
```

## Mobile-First Checklist

### Before Deployment

- [ ] LCP image < 100KB
- [ ] Total page weight < 500KB
- [ ] Preload LCP image with `fetchpriority="high"`
- [ ] No render-blocking CSS (critical CSS inlined)
- [ ] No render-blocking JavaScript (all deferred)
- [ ] Fonts: `font-display: swap` + preload primary font
- [ ] All images have width/height attributes
- [ ] Below-fold images use `loading="lazy"`

### Testing Verification

- [ ] Tested with DevTools 4G throttling
- [ ] Lighthouse mobile score 95+
- [ ] FCP < 1.8s
- [ ] LCP < 2.5s
- [ ] No CLS issues on mobile viewport

### Common Mobile Failures

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| FCP > 2.5s | Render-blocking resources | Inline critical CSS, defer JS |
| LCP > 4s | Large LCP image | Compress to < 100KB, preload |
| LCP > 4s | Late image discovery | Add preload link |
| TBT > 500ms | Large JS bundle | Code split, defer non-critical |
| CLS > 0.1 | Missing image dimensions | Add width/height |
| CLS > 0.1 | Font swap | Use font-display: optional |

## Quick Reference

### Mobile-First Image Sizes

```
Mobile viewport: 412px wide
└── Full-width image: 412px (src default)
└── Half-width image: 206px
└── Grid 3-col image: 137px

Tablet viewport: 768px wide
└── Full-width: 768px
└── Half-width: 384px

Desktop viewport: 1200px wide
└── Full-width: 1200px
└── Half-width: 600px
```

### srcset Mobile-First Pattern

```html
<img
  src="image-400.webp"
  srcset="
    image-400.webp 400w,
    image-800.webp 800w,
    image-1200.webp 1200w,
    image-1600.webp 1600w
  "
  sizes="
    (max-width: 640px) 100vw,
    (max-width: 1024px) 50vw,
    800px
  "
>
```

### Resource Hints Priority

```html
<head>
  <!-- 1. Critical third-party origins -->
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

  <!-- 2. LCP image (highest priority) -->
  <link rel="preload" as="image" href="/hero.webp" fetchpriority="high">

  <!-- 3. Critical font -->
  <link rel="preload" as="font" href="/font.woff2" type="font/woff2" crossorigin>

  <!-- 4. Critical CSS (if not inlined) -->
  <link rel="preload" as="style" href="/critical.css">
</head>
```
