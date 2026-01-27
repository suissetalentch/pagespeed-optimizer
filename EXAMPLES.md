# Real-World Examples

Real optimization case studies demonstrating the PageSpeed Optimizer skill in action.

## Case Study: Physio Servette

A physiotherapy clinic website optimized to achieve perfect PageSpeed scores.

### Project Overview

| Property | Value |
|----------|-------|
| **URL** | https://physio.chevassut.ch |
| **Stack** | Laravel + Vite + Alpine.js + Tailwind CSS |
| **Type** | Business website with contact form |

### Initial Audit

Run Lighthouse audit:

```bash
npx lighthouse https://physio.chevassut.ch --output=json --output-path=./report.json
```

#### Before Optimization

| Category | Score |
|----------|-------|
| Performance | 98 |
| Accessibility | 94 |
| Best Practices | 100 |
| SEO | 100 |

### Issues Identified

#### Issue 1: Color Contrast (Accessibility)

**Problem:** Carousel navigation dots had insufficient color contrast against the background.

**PageSpeed Message:**
> Background and foreground colors do not have a sufficient contrast ratio.

**Location:** Carousel component pagination dots

**Before:**
```css
.swiper-pagination-bullet {
  background: rgba(255, 255, 255, 0.5);
}
```

**After:**
```css
.swiper-pagination-bullet {
  background: rgba(255, 255, 255, 0.8);
  border: 1px solid rgba(0, 0, 0, 0.3);
}
```

**Impact:** +3 points Accessibility

---

#### Issue 2: Touch Target Size (Accessibility)

**Problem:** Carousel navigation dots were too small for touch interaction.

**PageSpeed Message:**
> Touch targets do not have sufficient size or spacing.

**WCAG Requirement:** Minimum 44x44px touch target

**Before:**
```css
.swiper-pagination-bullet {
  width: 8px;
  height: 8px;
}
```

**After:**
```css
.swiper-pagination-bullet {
  width: 12px;
  height: 12px;
  padding: 16px; /* Extends touch area */
  background-clip: content-box;
}
```

**Impact:** +3 points Accessibility

---

### Fixes Applied

#### 1. Updated Carousel Styles

```css
/* resources/css/components/carousel.css */

/* Navigation dots with proper contrast and touch targets */
.swiper-pagination-bullet {
  width: 12px;
  height: 12px;
  margin: 0 6px;
  background: rgba(255, 255, 255, 0.8);
  border: 1px solid rgba(0, 0, 0, 0.3);
  opacity: 1;
  transition: all 0.2s ease;

  /* Expand touch target */
  padding: 16px;
  background-clip: content-box;
}

.swiper-pagination-bullet-active {
  background: #ffffff;
  border-color: rgba(0, 0, 0, 0.5);
}
```

#### 2. Verified Existing Optimizations

The site already had these optimizations in place:

```php
// Security headers middleware
$response->headers->set('X-Content-Type-Options', 'nosniff');
$response->headers->set('X-Frame-Options', 'SAMEORIGIN');
$response->headers->set('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
```

```html
<!-- LCP image optimization -->
<link rel="preload" as="image" href="/images/hero.webp" fetchpriority="high">

<!-- Font preconnect -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
```

```javascript
// vite.config.js - Build optimization
build: {
  rollupOptions: {
    output: {
      manualChunks: {
        vendor: ['alpinejs'],
      }
    }
  },
  esbuild: { drop: ['console', 'debugger'] }
}
```

---

### After Optimization

| Category | Score | Change |
|----------|-------|--------|
| Performance | 100 | +2 |
| Accessibility | 100 | +6 |
| Best Practices | 100 | - |
| SEO | 100 | - |

### Verification

```bash
# Re-run Lighthouse
npx lighthouse https://physio.chevassut.ch --output=json

# Or use PageSpeed Insights
# https://pagespeed.web.dev/report?url=https://physio.chevassut.ch
```

### Key Takeaways

1. **Small details matter:** Two CSS fixes improved Accessibility by 6 points
2. **Touch targets are often overlooked:** WCAG requires 44x44px minimum
3. **Color contrast tools help:** Use browser DevTools contrast checker
4. **Test on mobile:** Touch target issues only appear in mobile audit

---

## Quick Reference: Common Fixes

### Accessibility Quick Wins

| Issue | Points | Fix Time |
|-------|--------|----------|
| Color contrast | 2-5 | 5 min |
| Touch targets | 2-4 | 5 min |
| Missing alt text | 1-3 | 5 min |
| Form labels | 2-4 | 10 min |
| Button aria-labels | 1-3 | 5 min |

### Performance Quick Wins

| Issue | Points | Fix Time |
|-------|--------|----------|
| LCP image preload | 5-15 | 2 min |
| Image dimensions | 3-8 | 10 min |
| Font preconnect | 2-5 | 2 min |
| Defer scripts | 3-10 | 5 min |

---

## How to Run Your Own Audit

### Step 1: Install Chrome

```bash
npx @puppeteer/browsers install chrome@stable
```

### Step 2: Run Lighthouse

```bash
CHROME_PATH=$(find $HOME/.cache/puppeteer -name chrome -type f 2>/dev/null | head -1) \
npx lighthouse https://your-site.com \
  --output=json \
  --output-path=./report.json \
  --chrome-flags="--headless --no-sandbox"
```

### Step 3: Analyze with PageSpeed Optimizer

```
/pagespeed-optimizer ./report.json
```

### Step 4: Apply Fixes and Re-test

Build, deploy, and run the audit again to verify improvements.

---

## Contributing Examples

Have a success story? We'd love to include it!

1. Fork the repository
2. Add your case study following this format
3. Include before/after scores
4. Show the specific fixes applied
5. Submit a pull request
