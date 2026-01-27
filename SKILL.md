---
name: pagespeed-optimizer
description: Optimize website performance, accessibility, best practices and SEO to achieve 100% PageSpeed scores. Use when user shares a PageSpeed report, asks to improve Core Web Vitals, fix LCP/FCP/CLS issues, or improve website performance.
argument-hint: [url] | [report.json] | --audit | --fix <category>
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, WebFetch
---

# PageSpeed Optimizer

**Target:** $ARGUMENTS

## Auto-detected Project Context

- package.json: !`cat package.json 2>/dev/null | head -30 || echo "Not found"`
- composer.json: !`cat composer.json 2>/dev/null | head -20 || echo "Not found"`
- Framework config: !`ls -la vite.config.* next.config.* nuxt.config.* webpack.config.* 2>/dev/null || echo "No config found"`

## Scores

| Category | Mobile | Desktop | Status |
|----------|--------|---------|--------|
| Performance | | | |
| Accessibility | | | |
| Best Practices | | | |
| SEO | | | |

## Score Interpretation

| Score | Status | Action |
|-------|--------|--------|
| 95-100 | ✅ Excellent | Goal achieved |
| 90-94 | 🟡 Good | Minor improvements possible |
| 50-89 | 🟠 Needs Work | Significant issues to fix |
| 0-49 | 🔴 Poor | Critical problems |

**Minimum Target:** 95+ on all 4 categories (Mobile AND Desktop)

## Get a PageSpeed Report

**Automatic method (Lighthouse CLI):**

If Chrome is not installed, install first:
```bash
npx @puppeteer/browsers install chrome@stable
```

Then run the audit:
```bash
CHROME_PATH=$(find $HOME/.cache/puppeteer -name chrome -type f 2>/dev/null | head -1) \
npx lighthouse https://example.com --output=json --output-path=./report.json --chrome-flags="--headless --no-sandbox"
```

**Manual method (if rate-limited):**

Ask the user to provide scores from https://pagespeed.web.dev:
```
What are your current PageSpeed scores?
Format: Performance: XX, Accessibility: XX, Best Practices: XX, SEO: XX
```

## Mobile & Desktop Testing

**IMPORTANT:** Always test BOTH mobile and desktop.

### Run Both Audits

```bash
# Mobile (default - more strict)
CHROME_PATH=$(find $HOME/.cache/puppeteer -name chrome -type f 2>/dev/null | head -1) \
npx lighthouse URL --output=json --output-path=./mobile.json --chrome-flags="--headless --no-sandbox"

# Desktop
CHROME_PATH=$(find $HOME/.cache/puppeteer -name chrome -type f 2>/dev/null | head -1) \
npx lighthouse URL --output=json --output-path=./desktop.json --preset=desktop --chrome-flags="--headless --no-sandbox"
```

**Note:** Mobile scores are typically lower due to:
- Simulated 4G throttling (slower network)
- Smaller viewport (320px width)
- Touch target requirements (minimum 44x44px)
- More aggressive CPU throttling

## Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│  1. ANALYZE        2. FIX              3. VERIFY               │
│  ───────────       ───────────         ───────────             │
│  • Identify stack  • Performance       • Re-run PageSpeed      │
│  • Read report     • Accessibility     • Check all 4 scores    │
│  • Prioritize      • Best Practices    • Validate fixes        │
│                    • SEO                                       │
└─────────────────────────────────────────────────────────────────┘
```

### Step 1: Analyze

1. **Get initial scores** (see "Get a PageSpeed Report" section above)
   - Fill in the Scores table with "Before" values for Mobile AND Desktop

2. **If JSON file provided:** Read and parse the Lighthouse report

3. **Detect framework** from project files:
   - `composer.json` + `vite.config.js` → Laravel + Vite → use [laravel-patterns.md](laravel-patterns.md)
   - `package.json` with `react` + `vite` → React + Vite → use [react-patterns.md](react-patterns.md)
   - `next.config.js` or `next.config.ts` → Next.js → use [nextjs-patterns.md](nextjs-patterns.md)
   - `nuxt.config.ts` or `nuxt.config.js` → Nuxt 3 → use [vue-nuxt-patterns.md](vue-nuxt-patterns.md)
   - `wp-config.php` or `style.css` with Theme → WordPress → use [wordpress-patterns.md](wordpress-patterns.md) *(Coming Soon)*

4. **Categorize issues** by score impact:
   | Priority | Impact | Examples |
   |----------|--------|----------|
   | CRITICAL | >10 pts | LCP image, render-blocking resources |
   | HIGH | 5-10 pts | Unoptimized images, missing preconnect |
   | MEDIUM | 2-5 pts | Cache headers, resource hints |
   | LOW | <2 pts | Minor optimizations |

## Safety Checklist (No Breaking Changes)

Before applying ANY fix, verify:

### Pre-Fix Verification
- [ ] Site loads correctly in browser
- [ ] All interactive elements work (forms, buttons, navigation)
- [ ] No console errors
- [ ] Take screenshots of key pages (or remember current state)

### Post-Fix Verification
- [ ] Site still loads correctly
- [ ] All interactive elements still work
- [ ] No new console errors
- [ ] Visual comparison with screenshots
- [ ] Run `npm run build` without errors

### High-Risk Changes (Extra Care)

| Change Type | Risk | Verification |
|-------------|------|--------------|
| CSS modifications | Layout breaks | Visual check all breakpoints |
| JS defer/async | Functionality breaks | Test all interactions |
| Font changes | FOUT/FOIT | Check text rendering |
| Image optimization | Quality loss | Visual comparison |
| Security headers | API/iframe breaks | Test all integrations |

See [troubleshooting.md](troubleshooting.md) for rollback procedures.

### Step 2: Fix (by category)

Execute fixes in this order for maximum impact:

#### Performance (Core Web Vitals)

**LCP (Largest Contentful Paint) < 2.5s:**
```html
<!-- Preload LCP image -->
<link rel="preload" as="image" href="/hero.webp" fetchpriority="high">

<!-- LCP image markup -->
<img src="/hero.webp" alt="..." width="1200" height="600"
     loading="eager" fetchpriority="high">
```

**FCP (First Contentful Paint) < 1.8s:**
```html
<!-- Non-blocking fonts -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="preload" as="style" href="FONT_URL">
<link rel="stylesheet" href="FONT_URL" media="print" onload="this.media='all'">
```

**CLS (Cumulative Layout Shift) < 0.1:**
```html
<!-- Always set dimensions -->
<img src="photo.webp" width="800" height="600" alt="...">

<!-- Or use aspect-ratio -->
<div style="aspect-ratio: 16/9">
  <img src="photo.webp" alt="..." class="w-full h-full object-cover">
</div>
```

**TBT (Total Blocking Time) < 200ms:**
```javascript
// vite.config.js - Code splitting
build: {
  rollupOptions: {
    output: {
      manualChunks: {
        vendor: ['react', 'react-dom'], // or ['alpinejs']
      }
    }
  },
  esbuild: { drop: ['console', 'debugger'] }
}
```

#### Accessibility (WCAG)

```html
<!-- Form fields: always use autocomplete + labels -->
<label for="email">Email *</label>
<input type="email" id="email" name="email"
       autocomplete="email" aria-required="true" required>

<!-- Icon buttons: always use aria-label -->
<button type="button" aria-label="Close menu">
  <svg aria-hidden="true">...</svg>
</button>

<!-- Viewport: never disable zoom -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

**Autocomplete values:** `name`, `email`, `tel`, `street-address`, `postal-code`, `country`

**Contrast ratios:** Normal text 4.5:1, Large text 3:1, UI components 3:1

#### Best Practices (Security)

```php
// Laravel middleware - Security headers
$response->headers->set('X-Content-Type-Options', 'nosniff');
$response->headers->set('X-Frame-Options', 'SAMEORIGIN');
$response->headers->set('Referrer-Policy', 'strict-origin-when-cross-origin');
$response->headers->set('Permissions-Policy', 'accelerometer=(), camera=(), geolocation=()');
$response->headers->set('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
```

#### SEO

```html
<title>Page Title - Site Name</title>
<meta name="description" content="150-160 character description">
<link rel="canonical" href="https://example.com/page">
```

### Step 3: Verify

After applying fixes:
1. Run production build: `npm run build`
2. Verify no breaking changes (see Safety Checklist)
3. Test locally or deploy
4. Re-run PageSpeed for BOTH Mobile AND Desktop
5. Confirm all 4 categories at 95+ minimum (target: 100)
6. Fill in "Final Results" section

## Execution Checklist

| # | Task | Category | Status |
|---|------|----------|--------|
| 1 | Run Mobile + Desktop audits | Setup | [ ] |
| 2 | Analyze reports, identify stack | Setup | [ ] |
| 3 | Pre-fix verification (Safety Checklist) | Safety | [ ] |
| 4 | Font loading (preconnect, preload, swap) | Performance | [ ] |
| 5 | LCP image optimization | Performance | [ ] |
| 6 | Image dimensions (width/height) | Performance/CLS | [ ] |
| 7 | Code splitting, defer scripts | Performance | [ ] |
| 8 | Form autocomplete attributes | Accessibility | [ ] |
| 9 | Button/link aria-labels | Accessibility | [ ] |
| 10 | Color contrast fixes | Accessibility | [ ] |
| 11 | Touch targets (44x44px minimum) | Accessibility | [ ] |
| 12 | Security headers middleware | Best Practices | [ ] |
| 13 | Meta tags, canonical URL | SEO | [ ] |
| 14 | Post-fix verification (Safety Checklist) | Safety | [ ] |
| 15 | Production build + re-test Mobile/Desktop | Verify | [ ] |
| 16 | Fill Final Results (95+ = ✅ PASSED) | Complete | [ ] |

## Framework Patterns

Detailed implementation patterns for each framework:

| Framework | File | Key Focus |
|-----------|------|-----------|
| Laravel + Vite + Alpine | [laravel-patterns.md](laravel-patterns.md) | Blade, middleware, Vite config |
| React + Vite | [react-patterns.md](react-patterns.md) | Components, lazy loading, hooks |
| Next.js | [nextjs-patterns.md](nextjs-patterns.md) | App router, Image component, metadata |
| Vue 3 + Nuxt 3 | [vue-nuxt-patterns.md](vue-nuxt-patterns.md) | SSR, @nuxt/image, composables |
| WordPress | [wordpress-patterns.md](wordpress-patterns.md) | *Coming Soon* |

## Optimization Guides

Specialized optimization topics:

| Topic | File | Content |
|-------|------|---------|
| Image Optimization | [images-optimization.md](images-optimization.md) | WebP/AVIF, srcset, compression tools |
| Advanced Optimization | [advanced-optimization.md](advanced-optimization.md) | Database, PWA, local caching |

## Real-World Examples

See [EXAMPLES.md](EXAMPLES.md) for case studies with before/after scores.

## Troubleshooting

Common issues and solutions: [troubleshooting.md](troubleshooting.md)

**Quick diagnostics:**
```bash
# Bundle sizes
ls -lhS public/build/assets/ 2>/dev/null || ls -lhS .next/static/chunks/ 2>/dev/null

# Large images
find public -type f \( -name "*.jpg" -o -name "*.png" \) -size +100k

# Console.log in production
grep -r "console.log" resources/js/ src/ 2>/dev/null

# Security headers test
curl -I https://example.com 2>/dev/null | grep -i "x-\|content-security\|strict"
```

## Final Results

After all optimizations, present results clearly:

### Results Summary

| Category | Mobile | Desktop | Status |
|----------|--------|---------|--------|
| Performance | XX | XX | ✅/🟡/🔴 |
| Accessibility | XX | XX | ✅/🟡/🔴 |
| Best Practices | XX | XX | ✅/🟡/🔴 |
| SEO | XX | XX | ✅/🟡/🔴 |

### Overall Status

| Status | Criteria |
|--------|----------|
| ✅ **PASSED** | All scores 95+ on both Mobile and Desktop |
| 🟡 **ACCEPTABLE** | All scores 90+ but some below 95 |
| 🔴 **NEEDS MORE WORK** | Any score below 90 |

### Changes Applied

List each fix with the file modified:
1. [Fix description] - `path/to/file`
2. ...

### Remaining Issues (if any)

Document issues that couldn't be fixed and explain why:
- [Issue] - [Reason / Limitation]
