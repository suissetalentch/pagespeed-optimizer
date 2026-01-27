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

## Scores (à remplir)

| Catégorie | Avant | Après | Δ |
|-----------|-------|-------|---|
| Performance | | | |
| Accessibility | | | |
| Best Practices | | | |
| SEO | | | |

## Obtenir un rapport PageSpeed

**Méthode automatique (Lighthouse CLI):**

Si Chrome n'est pas installé, installer d'abord:
```bash
npx @puppeteer/browsers install chrome@stable
```

Puis lancer l'audit:
```bash
CHROME_PATH=$(find $HOME/.cache/puppeteer -name chrome -type f 2>/dev/null | head -1) \
npx lighthouse https://example.com --output=json --output-path=./report.json --chrome-flags="--headless --no-sandbox"
```

**Méthode manuelle (si rate-limité):**

Demander à l'utilisateur de fournir les scores depuis https://pagespeed.web.dev :
```
Quels sont vos scores PageSpeed actuels ?
Format: Performance: XX, Accessibility: XX, Best Practices: XX, SEO: XX
```

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

1. **Obtenir les scores initiaux** (voir section "Obtenir un rapport" ci-dessus)
   - Remplir le tableau "Scores - Avant"

2. **Si fichier JSON fourni:** Lire et parser le rapport Lighthouse

3. **Detect framework** from project files:
   - `composer.json` + `vite.config.js` → Laravel + Vite → use [laravel-patterns.md](laravel-patterns.md)
   - `package.json` with `react` + `vite` → React + Vite → use [react-patterns.md](react-patterns.md)
   - `next.config.js` → Next.js → use [nextjs-patterns.md](nextjs-patterns.md)

4. **Categorize issues** by score impact:
   | Priority | Impact | Examples |
   |----------|--------|----------|
   | CRITICAL | >10 pts | LCP image, render-blocking resources |
   | HIGH | 5-10 pts | Unoptimized images, missing preconnect |
   | MEDIUM | 2-5 pts | Cache headers, resource hints |
   | LOW | <2 pts | Minor optimizations |

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
2. Test locally or deploy
3. Re-run PageSpeed Insights
4. Confirm all 4 categories at 100%

## Execution Checklist

| # | Task | Category | Status |
|---|------|----------|--------|
| 1 | Analyze report, identify stack | Setup | [ ] |
| 2 | Font loading (preconnect, preload, swap) | Performance | [ ] |
| 3 | LCP image optimization | Performance | [ ] |
| 4 | Image dimensions (width/height) | Performance/CLS | [ ] |
| 5 | Code splitting, defer scripts | Performance | [ ] |
| 6 | Form autocomplete attributes | Accessibility | [ ] |
| 7 | Button/link aria-labels | Accessibility | [ ] |
| 8 | Color contrast fixes | Accessibility | [ ] |
| 9 | Security headers middleware | Best Practices | [ ] |
| 10 | Meta tags, canonical URL | SEO | [ ] |
| 11 | Production build + test | Verify | [ ] |

## Framework Patterns

Detailed implementation patterns for each framework:

| Framework | File | Key Focus |
|-----------|------|-----------|
| Laravel + Vite + Alpine | [laravel-patterns.md](laravel-patterns.md) | Blade, middleware, Vite config |
| React + Vite | [react-patterns.md](react-patterns.md) | Components, lazy loading, hooks |
| Next.js | [nextjs-patterns.md](nextjs-patterns.md) | App router, Image component, metadata |

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
