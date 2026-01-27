---
name: pagespeed-optimizer
description: Optimize website performance, accessibility, best practices and SEO to achieve 100% PageSpeed scores. Use when user shares a PageSpeed report, asks to improve Core Web Vitals, fix LCP/FCP/CLS issues, or improve website performance.
argument-hint: [report-file] | [url] | --audit | --fix-performance | --fix-accessibility
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, WebFetch
---

# PageSpeed Optimizer

Optimize website for Google PageSpeed Insights: **$ARGUMENTS**

## Overview

This skill implements a systematic methodology to achieve 100% scores across all four PageSpeed categories:
- **Performance** (Core Web Vitals: LCP, FCP, CLS, TBT, SI)
- **Accessibility** (WCAG compliance)
- **Best Practices** (Security headers, modern APIs)
- **SEO** (Structured data, meta tags, crawlability)

## Methodology

### Phase 1: Audit & Analysis

1. **Read the PageSpeed report** (if provided)
2. **Identify framework/stack** from project files
3. **Categorize issues** by impact and effort
4. **Create prioritized task list**

### Phase 2: Performance Optimizations (Highest Impact First)

#### 2.1 Critical Rendering Path
```
Priority: CRITICAL - Impacts LCP/FCP directly
```

**Font Loading Optimization:**
- Remove `@import` for fonts from CSS (blocks render)
- Add `<link rel="preconnect">` for font origins
- Use `<link rel="preload" as="style">` with media="print" trick
- Ensure `font-display: swap` or `font-display: optional`

```html
<!-- Pattern: Non-blocking font loading -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="preload" as="style" href="FONT_URL">
<link rel="stylesheet" href="FONT_URL" media="print" onload="this.media='all'">
<noscript><link rel="stylesheet" href="FONT_URL"></noscript>
```

**CSS Optimization:**
- Inline critical CSS (above-the-fold)
- Defer non-critical CSS
- Remove unused CSS rules
- Use CSS containment (`contain: layout`)

**JavaScript Optimization:**
- Defer non-critical scripts (`defer` or `async`)
- Conditionally load scripts only on pages that need them
- Code split large bundles (separate vendor chunks)
- Remove console.log in production builds

#### 2.2 Image Optimization
```
Priority: HIGH - Often biggest payload
```

**Format & Compression:**
- Convert to WebP/AVIF with JPEG fallback
- Use appropriate quality (80-85% for photos)
- Compress SVGs (SVGO)

**Responsive Images:**
- Use `srcset` for different viewport sizes
- Set explicit `width` and `height` attributes (prevents CLS)
- Use `loading="lazy"` for below-fold images
- Use `loading="eager"` + `fetchpriority="high"` for LCP image
- Add `decoding="async"` for non-critical images

```html
<!-- Pattern: Optimized image -->
<img
  src="image.webp"
  alt="Descriptive alt text"
  width="800" height="600"
  loading="lazy"
  decoding="async"
>

<!-- Pattern: LCP image (hero) -->
<img
  src="hero.webp"
  alt="Hero description"
  width="1200" height="600"
  loading="eager"
  fetchpriority="high"
>
```

**Background Images:**
- Convert `background-image` to `<img>` when possible
- Use `<picture>` element for art direction

#### 2.3 Resource Hints
```
Priority: MEDIUM - Improves perceived performance
```

```html
<!-- DNS Prefetch (external domains you'll use) -->
<link rel="dns-prefetch" href="https://fonts.googleapis.com">
<link rel="dns-prefetch" href="https://www.google-analytics.com">

<!-- Preconnect (critical external origins) -->
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

<!-- Preload (critical assets for current page) -->
<link rel="preload" as="image" href="/logo.png">
<link rel="preload" as="font" href="/font.woff2" type="font/woff2" crossorigin>
```

#### 2.4 Caching Strategy
```
Priority: MEDIUM - Improves repeat visits
```

**Cache Headers by Asset Type:**
| Asset Type | Cache-Control | Max-Age |
|------------|---------------|---------|
| Hashed assets (Vite/Webpack) | `public, immutable` | 1 year |
| Images | `public` | 1 day - 1 week |
| Fonts | `public` | 1 year |
| HTML | `no-cache, must-revalidate` | - |

#### 2.5 Build Tool Configuration

**Vite (Recommended):**
```javascript
// vite.config.js
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // Separate large libraries
          vendor: ['react', 'react-dom'],
          // Or Alpine.js
          alpine: ['alpinejs', '@alpinejs/collapse'],
        },
      },
    },
    cssCodeSplit: true,
    minify: 'esbuild',
    esbuild: {
      drop: ['console', 'debugger'],
    },
  },
});
```

**Webpack:**
```javascript
// webpack.config.js
optimization: {
  splitChunks: {
    chunks: 'all',
    cacheGroups: {
      vendor: {
        test: /[\\/]node_modules[\\/]/,
        name: 'vendors',
        chunks: 'all',
      },
    },
  },
},
```

### Phase 3: Accessibility Fixes

#### 3.1 Form Accessibility
```
Priority: HIGH - Common PageSpeed flag
```

**Required Attributes:**
```html
<!-- Name field -->
<label for="name">Full Name *</label>
<input
  type="text"
  id="name"
  name="name"
  required
  aria-required="true"
  autocomplete="name"
>

<!-- Email field -->
<input type="email" autocomplete="email" aria-required="true">

<!-- Phone field -->
<input type="tel" autocomplete="tel">
```

**Autocomplete Values Reference:**
| Field Type | autocomplete Value |
|------------|-------------------|
| Full name | `name` |
| Email | `email` |
| Phone | `tel` |
| Address | `street-address` |
| City | `address-level2` |
| Postal code | `postal-code` |
| Country | `country` |

#### 3.2 Button & Link Accessibility
```
Priority: HIGH - Screen reader usability
```

```html
<!-- Icon-only buttons need aria-label -->
<button type="button" aria-label="Previous slide">
  <svg aria-hidden="true">...</svg>
</button>

<!-- Decorative elements -->
<span aria-hidden="true">*</span>
<svg aria-hidden="true">...</svg>
```

#### 3.3 Color Contrast
```
Priority: MEDIUM - WCAG compliance
```

**Minimum Contrast Ratios:**
- Normal text: 4.5:1
- Large text (18px+ or 14px+ bold): 3:1
- UI components: 3:1

**Common Fixes:**
- Darken light colors (e.g., `text-primary-600` → `text-primary-700`)
- Lighten dark backgrounds
- Add text shadows for text on images

#### 3.4 Focus States
```
Priority: MEDIUM - Keyboard navigation
```

```css
/* Enhanced focus-visible */
*:focus-visible {
  outline: 2px solid var(--primary-500);
  outline-offset: 2px;
  border-radius: 2px;
}

/* Interactive elements */
a:focus-visible,
button:focus-visible {
  ring: 2px;
  ring-color: var(--primary-500);
  ring-offset: 2px;
}
```

### Phase 4: Best Practices

#### 4.1 Security Headers
```
Priority: HIGH - Security & PageSpeed flag
```

**Essential Headers:**
```
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: accelerometer=(), camera=(), geolocation=()
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: [see CSP section]
```

#### 4.2 HTTPS & Mixed Content
- Ensure `ASSET_URL` is set to HTTPS in production
- Check for HTTP resources in HTML/CSS
- Use protocol-relative URLs or HTTPS only

### Phase 5: SEO Optimization

#### 5.1 Meta Tags
```html
<title>Page Title - Site Name</title>
<meta name="description" content="150-160 character description">
<link rel="canonical" href="https://example.com/page">
```

#### 5.2 Structured Data
- LocalBusiness schema for businesses
- Article schema for blog posts
- BreadcrumbList for navigation
- FAQ schema for FAQ pages

#### 5.3 Crawlability
- Valid robots.txt
- XML sitemap
- Proper hreflang tags for multilingual sites

## Execution Checklist

When optimizing a site, follow this order:

1. [ ] **Analyze** - Read PageSpeed report, identify stack
2. [ ] **Fonts** - Non-blocking loading, preconnect
3. [ ] **Scripts** - Conditional loading, code splitting
4. [ ] **Images** - Format, lazy loading, dimensions
5. [ ] **Resource hints** - Preconnect, prefetch, preload
6. [ ] **Forms** - Autocomplete, aria-required, labels
7. [ ] **Buttons** - aria-label for icon buttons
8. [ ] **Contrast** - Check and fix color ratios
9. [ ] **Cache** - Set appropriate headers
10. [ ] **Build** - Run production build
11. [ ] **Test** - Re-run PageSpeed, verify fixes

## Framework-Specific Patterns

### Laravel + Vite + Alpine.js
See [laravel-patterns.md](laravel-patterns.md)

### React + Vite
See [react-patterns.md](react-patterns.md)

### Next.js
See [nextjs-patterns.md](nextjs-patterns.md)

## Common Issues & Solutions

See [troubleshooting.md](troubleshooting.md)
