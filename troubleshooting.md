# PageSpeed Troubleshooting Guide

## Performance Issues

### LCP (Largest Contentful Paint) > 2.5s

**Symptoms:** Hero image or main content takes too long to appear

**Common Causes & Fixes:**

| Cause | Fix |
|-------|-----|
| Large hero image | Compress, use WebP, resize to actual display size |
| Font blocking render | Use font-display: swap, preload fonts |
| Render-blocking CSS | Inline critical CSS, defer non-critical |
| Render-blocking JS | Add `defer` or `async`, load conditionally |
| No preconnect | Add `<link rel="preconnect">` for critical origins |
| TTFB too high | Server optimization, CDN, caching |

**Quick Check:**
```bash
# Check image sizes
find public/images -type f -exec ls -lh {} \; | sort -k5 -h

# Check bundle sizes
cat public/build/manifest.json | jq '.'
```

### FCP (First Contentful Paint) > 1.8s

**Common Causes:**
- @import in CSS blocking fonts
- Large initial JavaScript bundle
- No resource hints

**Fixes:**
```html
<!-- Move fonts from CSS to HTML with non-blocking pattern -->
<link rel="preload" as="style" href="FONT_URL">
<link rel="stylesheet" href="FONT_URL" media="print" onload="this.media='all'">
```

### CLS (Cumulative Layout Shift) > 0.1

**Symptoms:** Content jumping around during load

**Common Causes & Fixes:**

| Cause | Fix |
|-------|-----|
| Images without dimensions | Add `width` and `height` attributes |
| Fonts causing FOUT | Use `font-display: optional` or size-adjust |
| Dynamic content above fold | Reserve space with min-height |
| Ads/embeds without size | Set explicit container dimensions |
| CSS animations on load | Use `transform` instead of `top/left/width/height` |

**Quick Fix Pattern:**
```html
<!-- Always specify dimensions -->
<img src="photo.jpg" width="800" height="600" alt="...">

<!-- Or use aspect-ratio -->
<div style="aspect-ratio: 16/9;">
  <img src="photo.jpg" alt="..." class="w-full h-full object-cover">
</div>
```

### TBT (Total Blocking Time) > 200ms

**Symptoms:** Page feels unresponsive

**Common Causes:**
- Large JavaScript bundles
- No code splitting
- Parsing large JSON inline

**Fixes:**
```javascript
// vite.config.js - Split large libraries
build: {
  rollupOptions: {
    output: {
      manualChunks: {
        vendor: ['react', 'react-dom'],
        charts: ['chart.js'],
      }
    }
  }
}
```

---

## Accessibility Issues

### "Buttons do not have an accessible name"

**Problem:** Icon-only buttons without text

**Fix:**
```html
<!-- Before -->
<button><svg>...</svg></button>

<!-- After -->
<button aria-label="Close menu">
  <svg aria-hidden="true">...</svg>
</button>
```

### "Form elements do not have associated labels"

**Problem:** Inputs without proper label association

**Fix:**
```html
<!-- Before -->
<label>Name</label>
<input name="name">

<!-- After -->
<label for="name">Name</label>
<input id="name" name="name">

<!-- Alternative: wrap input in label -->
<label>
  Name
  <input name="name">
</label>
```

### "Background and foreground colors do not have sufficient contrast"

**Problem:** Text color too similar to background

**Diagnosis:**
```bash
# Check current colors in Tailwind config
grep -r "primary\|secondary" tailwind.config.js
```

**Common Fixes:**
| Original | Fix |
|----------|-----|
| `text-primary-600` on white | Use `text-primary-700` or `text-primary-800` |
| `text-gray-400` | Use `text-gray-600` |
| `bg-primary-100 text-primary-600` | Darken text or lighten background |

**Tool:** Use Chrome DevTools > Elements > Inspect > Check "Contrast" in Styles panel

### "Links do not have a discernible name"

**Problem:** Links with only images/icons

**Fix:**
```html
<!-- Before -->
<a href="/"><img src="logo.png"></a>

<!-- After -->
<a href="/" aria-label="Home - Company Name">
  <img src="logo.png" alt="Company Logo">
</a>
```

### "[user-scalable="no"] in viewport meta"

**Problem:** Zoom disabled, bad for accessibility

**Fix:**
```html
<!-- Before -->
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">

<!-- After -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

---

## Best Practices Issues

### "Missing Content Security Policy"

**Fix (Laravel middleware):**
```php
$response->headers->set('Content-Security-Policy', implode('; ', [
    "default-src 'self'",
    "script-src 'self' 'unsafe-inline'",
    "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
    "img-src 'self' data: https:",
    "font-src 'self' https://fonts.gstatic.com",
]));
```

### "Errors logged to console"

**Common Causes:**
- Failed API requests
- Missing resources (404)
- JavaScript errors

**Diagnosis:**
```bash
# Check for 404s in logs
grep "404" storage/logs/laravel.log | tail -20
```

### "Uses deprecated APIs"

**Common Deprecated APIs:**
- `document.write()` - Remove or use insertAdjacentHTML
- Synchronous XMLHttpRequest - Use fetch or async XHR
- `unload` event - Use `pagehide` instead

---

## SEO Issues

### "robots.txt is not valid"

**Problem:** Server returns HTML instead of robots.txt

**Laravel Fix:**
```php
// routes/web.php
Route::get('/robots.txt', function () {
    $content = "User-agent: *\nAllow: /\nSitemap: " . url('/sitemap.xml');
    return response($content, 200)
        ->header('Content-Type', 'text/plain');
});
```

### "Document does not have a meta description"

**Fix:**
```html
<meta name="description" content="150-160 characters describing the page content">
```

### "Links are not crawlable"

**Problem:** JavaScript-only navigation

**Fix:**
```html
<!-- Before -->
<a @click="navigate('/page')">Link</a>

<!-- After -->
<a href="/page" @click.prevent="navigate('/page')">Link</a>
```

---

## Framework-Specific Issues

### Vite: "terser not found"

**Problem:** Using `minify: 'terser'` without installing terser

**Fix:** Use esbuild instead (built-in):
```javascript
build: {
  minify: 'esbuild',
  esbuild: {
    drop: ['console', 'debugger'],
  }
}
```

### Laravel: Route caching error

**Problem:** "Unable to prepare route for serialization"

**Cause:** Closure routes can't be cached

**Fix:** Convert closures to controller methods

### Mixed Content on HTTPS

**Problem:** Assets loading over HTTP on HTTPS site

**Fix (.env):**
```env
APP_URL=https://example.com
ASSET_URL=https://example.com
```

---

## Quick Diagnostic Commands

```bash
# Check bundle sizes
ls -lhS public/build/assets/

# Check image sizes
find public/images -type f -name "*.jpg" -o -name "*.png" | xargs ls -lhS | head -20

# Check for console.log in production
grep -r "console.log" resources/js/

# Verify build output
cat public/build/manifest.json

# Test security headers
curl -I https://example.com 2>/dev/null | grep -i "x-\|content-security\|strict-transport"

# Check robots.txt
curl https://example.com/robots.txt
```

---

## Before Deploying Checklist

- [ ] `npm run build` completed successfully
- [ ] No console.log in production code
- [ ] All images optimized (WebP, proper sizes)
- [ ] Security headers configured
- [ ] Cache headers set
- [ ] HTTPS assets verified
- [ ] robots.txt accessible
- [ ] sitemap.xml generated
- [ ] Run `php artisan optimize` (Laravel)
