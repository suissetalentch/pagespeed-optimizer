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

## Mobile-First Methodology

**CRITICAL:** Always test MOBILE FIRST. If mobile passes → Desktop will automatically pass.

### Why Mobile-First?

Mobile Lighthouse simulates much harsher conditions than real desktop browsing:

| Aspect | Desktop Lighthouse | Mobile Lighthouse |
|--------|-------------------|-------------------|
| Network | No throttling | 4G: 1.6 Mbps down, 150ms RTT |
| CPU | Normal | 4x slowdown |
| Viewport | 1350x940 | 412x823 |
| Impact of 500KB image | ~instant | ~2.5s download |

**Key insight:** Optimizing for mobile constraints guarantees desktop will perform even better.

### Mobile-First Workflow

```
┌─────────────────────────────────────────────────────────────┐
│  MOBILE-FIRST METHODOLOGY                                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. AUDIT MOBILE    2. FIX FOR MOBILE    3. VERIFY BOTH     │
│  ──────────────     ────────────────     ──────────────     │
│  • Test mobile      • Budget: 150KB      • Mobile 95+       │
│  • Identify LCP     • Images mobile-     • Desktop auto-    │
│  • Network impact     sized first          passes           │
│                     • Preload critical                      │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Run Mobile Audit First

```bash
# Step 1: Mobile audit (ALWAYS first)
CHROME_PATH=$(find $HOME/.cache/puppeteer -name chrome -type f 2>/dev/null | head -1) \
npx lighthouse URL --output=json --output-path=./mobile.json --chrome-flags="--headless --no-sandbox"

# Step 2: Desktop audit (only after mobile passes)
CHROME_PATH=$(find $HOME/.cache/puppeteer -name chrome -type f 2>/dev/null | head -1) \
npx lighthouse URL --output=json --output-path=./desktop.json --preset=desktop --chrome-flags="--headless --no-sandbox"
```

### Mobile Performance Budget

| Resource | Budget | Calculation |
|----------|--------|-------------|
| Total page | < 500KB | Ideal: < 300KB |
| **LCP image** | **< 100KB** | Critical for 2.5s target |
| Critical CSS | < 14KB | First TCP packet |
| JavaScript | < 150KB | Parsed size |

### Key Diagnostic Questions

Before fixing, always identify:

1. **What is the LCP element?** (image/text/video)
2. **What is its file size?** (must be < 100KB for mobile)
3. **Is it preloaded with `fetchpriority="high"`?**
4. **What's blocking FCP?** (fonts/CSS/JS)

```javascript
// Identify LCP element in DevTools Console
new PerformanceObserver((list) => {
  const entries = list.getEntries();
  const lastEntry = entries[entries.length - 1];
  console.log('LCP element:', lastEntry.element);
  console.log('LCP time:', lastEntry.startTime);
}).observe({type: 'largest-contentful-paint', buffered: true});
```

### DevTools Mobile Testing

For quick iteration before running Lighthouse:

1. Open DevTools (F12)
2. Toggle device toolbar (Ctrl+Shift+M)
3. Select "Mid-tier mobile" or custom profile:
   - Network: Slow 4G (1.5 Mbps down, 400ms RTT)
   - CPU: 4x slowdown
4. Hard refresh (Ctrl+Shift+R)
5. Check Network tab waterfall

See [docs/mobile-optimization.md](docs/mobile-optimization.md) for comprehensive mobile optimization strategies.

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
   - `composer.json` + `vite.config.js` → Laravel + Vite → use [docs/laravel-patterns.md](docs/laravel-patterns.md)
   - `package.json` with `react` + `vite` → React + Vite → use [docs/react-patterns.md](docs/react-patterns.md)
   - `next.config.js` or `next.config.ts` → Next.js → use [docs/nextjs-patterns.md](docs/nextjs-patterns.md)
   - `nuxt.config.ts` or `nuxt.config.js` → Nuxt 3 → use [docs/vue-nuxt-patterns.md](docs/vue-nuxt-patterns.md)
   - `wp-config.php` or `style.css` with Theme → WordPress → use [docs/wordpress-patterns.md](docs/wordpress-patterns.md) **+ WordPress Safe Mode**

4. **Categorize issues** by score impact:
   | Priority | Impact | Examples |
   |----------|--------|----------|
   | CRITICAL | >10 pts | LCP image, render-blocking resources |
   | HIGH | 5-10 pts | Unoptimized images, missing preconnect |
   | MEDIUM | 2-5 pts | Cache headers, resource hints |
   | LOW | <2 pts | Minor optimizations |

5. **If WordPress detected:** Follow WordPress Safe Mode (see below)

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

See [docs/troubleshooting.md](docs/troubleshooting.md) for rollback procedures.

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

## Execution Checklist (Mobile-First)

| # | Task | Category | Status |
|---|------|----------|--------|
| 1 | Run **Mobile** audit FIRST | Setup | [ ] |
| 2 | Identify LCP element + measure file size | Setup | [ ] |
| 3 | **Calculate mobile budget** (LCP < 100KB) | Setup | [ ] |
| 4 | Analyze report, identify stack | Setup | [ ] |
| 5 | Pre-fix verification (Safety Checklist) | Safety | [ ] |
| 6 | **LCP image < 100KB + preload + fetchpriority** | Performance | [ ] |
| 7 | Font loading (preconnect, preload, swap) | Performance | [ ] |
| 8 | Image dimensions (width/height) | Performance/CLS | [ ] |
| 9 | Code splitting, defer scripts | Performance | [ ] |
| 10 | Form autocomplete attributes | Accessibility | [ ] |
| 11 | Button/link aria-labels | Accessibility | [ ] |
| 12 | Color contrast fixes | Accessibility | [ ] |
| 13 | Touch targets (44x44px minimum) | Accessibility | [ ] |
| 14 | Security headers middleware | Best Practices | [ ] |
| 15 | Meta tags, canonical URL | SEO | [ ] |
| 16 | Post-fix verification (Safety Checklist) | Safety | [ ] |
| 17 | **Re-test Mobile** (if 95+ → Desktop auto-passes) | Verify | [ ] |
| 18 | Run Desktop audit (confirmation only) | Verify | [ ] |
| 19 | Fill Final Results (95+ = ✅ PASSED) | Complete | [ ] |

## Framework Patterns

Detailed implementation patterns for each framework:

| Framework | File | Key Focus |
|-----------|------|-----------|
| Laravel + Vite + Alpine | [docs/laravel-patterns.md](docs/laravel-patterns.md) | Blade, middleware, Vite config |
| React + Vite | [docs/react-patterns.md](docs/react-patterns.md) | Components, lazy loading, hooks |
| Next.js | [docs/nextjs-patterns.md](docs/nextjs-patterns.md) | App router, Image component, metadata |
| Vue 3 + Nuxt 3 | [docs/vue-nuxt-patterns.md](docs/vue-nuxt-patterns.md) | SSR, @nuxt/image, composables |
| WordPress | [docs/wordpress-patterns.md](docs/wordpress-patterns.md) | Safe Mode, plugins, .htaccess |

---

## WordPress Safe Mode

WordPress requires special handling due to live changes and lack of version control.

### WordPress Detection

If any of these are detected, activate WordPress Safe Mode:
- `wp-config.php` file present
- `/wp-admin` URL mentioned
- `style.css` with `Theme Name:` header
- User mentions WordPress, WooCommerce, or common WP plugins

### Step 1: Determine Access Level

**Ask the user:**

1. Do you have FTP/SFTP access to upload files?
2. Do you have SSH access (can run `wp` CLI commands)?
3. Do you have database access (phpMyAdmin)?
4. **Do you have a current backup of your site?**

### Step 2: Safety Mode Based on Access

```
┌─────────────────────────────────────────────────────────┐
│  WORDPRESS SAFE MODE                                     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  1. DETECT      2. ASSESS       3. RECOMMEND             │
│  ──────────     ──────────      ────────────             │
│  • wp-config?   • Backup?       • Plugins first          │
│  • Access level • FTP/SSH?      • .htaccess safe         │
│  • Audit only   • DB access?    • Manual instructions    │
│                                                          │
│  4. VERIFY (User applies changes)                        │
│  ─────────────────────────────────                       │
│  • User makes changes manually or approves               │
│  • Re-run PageSpeed                                      │
│  • Confirm no breakage                                   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

**WITHOUT confirmed backup:**
- ONLY provide audit results and recommendations
- NO file modifications at all
- Guide user to create backup first (see [wordpress-patterns.md](wordpress-patterns.md))

**WITH backup + Admin only (no FTP):**
- Plugin recommendations only
- Configuration instructions for plugins
- No code changes possible

**WITH backup + FTP only:**
- `.htaccess` modifications (show diff first, require approval)
- Plugin recommendations
- Manual copy-paste instructions for functions.php
- Child theme setup instructions

**WITH backup + SSH:**
- Can use `wp-cli` commands for safer operations
- Still prefer plugin solutions over code
- Code changes require explicit user approval

### Step 3: Plugin-First Approach

**Always recommend plugins before code changes:**
- Plugins can be disabled via Admin panel
- Users can manage without technical knowledge
- Less risk of conflicts with updates
- Easier rollback

See [docs/wordpress-patterns.md](docs/wordpress-patterns.md) for plugin recommendations.

### WordPress Safety Rules

#### NEVER Do (Even With User Approval)

| Action | Reason |
|--------|--------|
| Modify `wp-config.php` | Critical file - can break entire site |
| Modify parent theme files | Updates will overwrite changes |
| Modify plugin files | Updates will overwrite changes |
| Modify core WordPress files | Never touch core |
| Run database queries directly | High risk of data loss |
| Delete any files | No automatic recovery |

#### Safe to Do (With Backup Confirmation)

| Action | Requirement |
|--------|-------------|
| Read files for analysis | Always safe |
| Create new files in child theme | Backup confirmed |
| Modify `.htaccess` | Show diff preview + user approval |
| Add code to child theme's `functions.php` | Backup confirmed + user approval |
| Use `wp-cli` for non-destructive commands | SSH access + backup confirmed |

#### Always Ask First

Before any modification:

1. "Do you have a current backup of your WordPress site?"
2. "What level of access do you have? (Admin only / FTP / SSH)"
3. "Would you prefer I show you the changes to apply manually, or should I apply them directly?"

### WordPress Modification Workflow

When user requests a change that requires file modification:

```
1. Confirm backup exists
   └─ No backup → STOP, provide backup instructions first

2. Identify file to modify
   └─ Core/Plugin/Parent theme → REFUSE, explain why

3. Show proposed change
   └─ Display diff preview of changes

4. Get explicit approval
   └─ User must confirm: "Yes, apply this change"

5. Apply change
   └─ Make the modification

6. Verify
   └─ Ask user to test site immediately
```

### Example WordPress Interaction

```
User: My WordPress site has slow PageSpeed scores

Claude: I'll help optimize your WordPress site. First, I need to understand your setup:

1. Do you have FTP/SFTP access to your site?
2. Do you have SSH access (can you run wp-cli commands)?
3. Do you have a current backup of your site?

[User responds: FTP only, no backup]

Claude: Before we make any changes, you'll need a backup. Here's how:

Option A - Plugin (easiest):
1. Install UpdraftPlus from Plugins > Add New
2. Go to Settings > UpdraftPlus
3. Click "Backup Now"
4. Download the backup files

Option B - Manual via FTP:
1. Download your entire wp-content folder
2. Export database via phpMyAdmin

Let me know when your backup is complete.

[User confirms backup]

Claude: Now I can safely help. Based on your PageSpeed issues:

**Recommended plugins (safest approach):**
- WP Super Cache - for page caching
- ShortPixel - for image optimization
- Autoptimize - for CSS/JS minification

**If you want better cache headers**, I can show you .htaccess
changes to apply manually. Would you like me to generate the
code for you to copy-paste?
```

---

## Optimization Guides

Specialized optimization topics:

| Topic | File | Content |
|-------|------|---------|
| **Mobile Optimization** | [docs/mobile-optimization.md](docs/mobile-optimization.md) | Mobile-first methodology, budgets, LCP/FCP fixes |
| Image Optimization | [docs/images-optimization.md](docs/images-optimization.md) | WebP/AVIF, srcset, compression tools |
| Advanced Optimization | [docs/advanced-optimization.md](docs/advanced-optimization.md) | Database, PWA, local caching |

## Real-World Examples

See [docs/EXAMPLES.md](docs/EXAMPLES.md) for case studies with before/after scores.

## Troubleshooting

Common issues and solutions: [docs/troubleshooting.md](docs/troubleshooting.md)

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
