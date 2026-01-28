# WordPress Performance Patterns

## WordPress Safety Notice

WordPress requires extra caution compared to modern frameworks:

| Aspect | Modern Framework | WordPress |
|--------|------------------|-----------|
| Version control | Git standard | Often none |
| Build process | npm run build | None - changes are live |
| Access | SSH + full code | Sometimes FTP only |
| Database | Local / direct access | External hosting |
| Codebase | Known | Unknown plugins |
| Rollback | git checkout | Manual backup |
| Who manages | Developer | Often non-developer |

**Golden Rule:** BACKUP FIRST, change second.

### Files That Can Be Modified (With Approval)

| File | Safety | Requirements |
|------|--------|--------------|
| `.htaccess` | Medium | Backup confirmed + diff preview |
| Child theme `functions.php` | Medium | Backup confirmed + user approval |
| Child theme `style.css` | Low | Backup confirmed |

### Files NEVER Modified

| File | Reason |
|------|--------|
| `wp-config.php` | Critical - can break entire site |
| Parent theme files | Updates will overwrite changes |
| Plugin files | Updates will overwrite changes |
| Core WordPress files | Never modify core |

---

## Access Level Detection

Before optimizing, determine your access level:

### Check Your Access

| Access Type | How to Check | What You Can Do |
|-------------|--------------|-----------------|
| **Admin Panel** | Can you login to `/wp-admin`? | Install plugins, basic settings |
| **FTP/SFTP** | Can you upload files? | Edit .htaccess, child theme files |
| **SSH** | Can you run `wp` CLI commands? | Full CLI access, safer operations |
| **Database** | Can you access phpMyAdmin? | Query optimization, cleanup |

### Recommendations by Access Level

| Access Level | Detection | Safe Actions |
|--------------|-----------|--------------|
| **Full** | wp-config.php + SSH + DB | Code changes (with backup) |
| **FTP Only** | wp-config.php, no SSH | Instructions + .htaccess |
| **Admin Only** | No code access | Plugin recommendations only |
| **None** | URL only | Audit + general recommendations |

---

## Backup Procedures (MANDATORY)

### Before ANY Change

**Option A: Plugin (if Admin access)**

| Plugin | Free? | Best For |
|--------|-------|----------|
| UpdraftPlus | Yes | Full site + database |
| All-in-One WP Migration | Yes | Site migration |
| Duplicator | Yes | Complete packages |

**Option B: FTP Manual Backup**

1. Download entire `wp-content` folder via FTP
2. Export database via phpMyAdmin:
   - Go to phpMyAdmin
   - Select your database
   - Click "Export"
   - Choose "Quick" or "Custom" (SQL format)
   - Download the `.sql` file

**Option C: Hosting Backup**

Most hosts offer snapshot features:
- cPanel: Backup Wizard
- Plesk: Backup Manager
- Managed WordPress: Often automatic daily

### Verify Your Backup

Before proceeding:
- [ ] Files backed up (wp-content at minimum)
- [ ] Database exported
- [ ] Backup tested (can you restore?)
- [ ] Backup stored off-server (local or cloud)

---

## Safe Optimizations (No Code Changes)

These optimizations use plugins and require only Admin access.

### Plugin-Based Solutions

| Problem | Recommended Plugin | Alternative | Notes |
|---------|-------------------|-------------|-------|
| **Caching** | WP Super Cache | LiteSpeed Cache | LiteSpeed for LiteSpeed servers |
| **Images** | ShortPixel | Imagify | Automatic compression |
| **WebP** | WebP Express | ShortPixel | WebP conversion |
| **Lazy Load** | Native WP 5.5+ | a3 Lazy Load | Built-in since WP 5.5 |
| **Security Headers** | HTTP Headers | Headers Security | Or use .htaccess |
| **Minification** | Autoptimize | LiteSpeed Cache | JS/CSS minification |
| **Database** | WP-Optimize | Advanced DB Cleaner | Cleanup and optimize |

### Plugin Configuration Guide

#### WP Super Cache

```
Settings > WP Super Cache:
1. Enable Caching: ON
2. Caching > Simple (recommended for most)
3. Advanced:
   - [x] Cache hits
   - [x] Compress pages
   - [x] Don't cache for logged-in users
4. Preload > Enable preload mode
```

#### Autoptimize

```
Settings > Autoptimize:
1. JavaScript:
   - [x] Optimize JavaScript
   - [x] Aggregate JS files
   - [ ] Also aggregate inline JS (careful - can break)
2. CSS:
   - [x] Optimize CSS
   - [x] Aggregate CSS files
   - [x] Generate data: URIs for images
3. HTML:
   - [x] Optimize HTML
4. Images:
   - [x] Lazy-load images
```

#### ShortPixel

```
Settings > ShortPixel:
1. Compression type: Lossy (best results)
2. Also include thumbnails: YES
3. Create WebP: YES
4. Backup originals: YES
5. Bulk optimize existing images
```

---

## .htaccess Optimizations (FTP Safe)

These changes require FTP access. Always backup `.htaccess` first.

### Before Editing

1. Download current `.htaccess` via FTP
2. Save a copy as `.htaccess.backup`
3. Test site works before changes
4. Make ONE change at a time
5. Test site after each change

### Browser Caching

Add BEFORE `# BEGIN WordPress`:

```apache
# BEGIN Browser Caching
<IfModule mod_expires.c>
    ExpiresActive On

    # Images
    ExpiresByType image/jpeg "access plus 1 year"
    ExpiresByType image/png "access plus 1 year"
    ExpiresByType image/gif "access plus 1 year"
    ExpiresByType image/webp "access plus 1 year"
    ExpiresByType image/svg+xml "access plus 1 year"
    ExpiresByType image/x-icon "access plus 1 year"

    # Fonts
    ExpiresByType font/woff2 "access plus 1 year"
    ExpiresByType font/woff "access plus 1 year"
    ExpiresByType application/font-woff2 "access plus 1 year"
    ExpiresByType application/font-woff "access plus 1 year"

    # CSS and JavaScript
    ExpiresByType text/css "access plus 1 month"
    ExpiresByType application/javascript "access plus 1 month"
    ExpiresByType text/javascript "access plus 1 month"

    # Others
    ExpiresByType application/pdf "access plus 1 month"
    ExpiresByType text/html "access plus 0 seconds"
</IfModule>

<IfModule mod_headers.c>
    <FilesMatch "\.(ico|jpe?g|png|gif|webp|svg|woff2?|css|js)$">
        Header set Cache-Control "public, max-age=31536000, immutable"
    </FilesMatch>
    <FilesMatch "\.(html|htm|php)$">
        Header set Cache-Control "no-cache, no-store, must-revalidate"
    </FilesMatch>
</IfModule>
# END Browser Caching
```

**To verify:** Check headers with browser DevTools > Network > select file > Headers tab.

**To rollback:** Remove the section between `# BEGIN Browser Caching` and `# END Browser Caching`.

### Gzip Compression

Add BEFORE `# BEGIN WordPress`:

```apache
# BEGIN Gzip Compression
<IfModule mod_deflate.c>
    # Compress HTML, CSS, JavaScript, Text, XML and fonts
    AddOutputFilterByType DEFLATE application/javascript
    AddOutputFilterByType DEFLATE application/rss+xml
    AddOutputFilterByType DEFLATE application/vnd.ms-fontobject
    AddOutputFilterByType DEFLATE application/x-font
    AddOutputFilterByType DEFLATE application/x-font-opentype
    AddOutputFilterByType DEFLATE application/x-font-otf
    AddOutputFilterByType DEFLATE application/x-font-truetype
    AddOutputFilterByType DEFLATE application/x-font-ttf
    AddOutputFilterByType DEFLATE application/x-javascript
    AddOutputFilterByType DEFLATE application/xhtml+xml
    AddOutputFilterByType DEFLATE application/xml
    AddOutputFilterByType DEFLATE font/opentype
    AddOutputFilterByType DEFLATE font/otf
    AddOutputFilterByType DEFLATE font/ttf
    AddOutputFilterByType DEFLATE image/svg+xml
    AddOutputFilterByType DEFLATE image/x-icon
    AddOutputFilterByType DEFLATE text/css
    AddOutputFilterByType DEFLATE text/html
    AddOutputFilterByType DEFLATE text/javascript
    AddOutputFilterByType DEFLATE text/plain
    AddOutputFilterByType DEFLATE text/xml

    # Remove browser bugs (only needed for old browsers)
    BrowserMatch ^Mozilla/4 gzip-only-text/html
    BrowserMatch ^Mozilla/4\.0[678] no-gzip
    BrowserMatch \bMSIE !no-gzip !gzip-only-text/html
    Header append Vary User-Agent
</IfModule>
# END Gzip Compression
```

**To verify:** Use online Gzip tester or check `Content-Encoding: gzip` header.

**To rollback:** Remove the section between markers.

### Security Headers

Add BEFORE `# BEGIN WordPress`:

```apache
# BEGIN Security Headers
<IfModule mod_headers.c>
    # Prevent MIME type sniffing
    Header set X-Content-Type-Options "nosniff"

    # Prevent clickjacking
    Header set X-Frame-Options "SAMEORIGIN"

    # XSS Protection (legacy browsers)
    Header set X-XSS-Protection "1; mode=block"

    # Referrer Policy
    Header set Referrer-Policy "strict-origin-when-cross-origin"

    # Permissions Policy (limit browser features)
    Header set Permissions-Policy "accelerometer=(), camera=(), geolocation=(), gyroscope=(), magnetometer=(), microphone=(), payment=(), usb=()"

    # HSTS (ONLY if site is HTTPS - uncomment next line)
    # Header set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
</IfModule>
# END Security Headers
```

**Warning:** Only enable HSTS if your site is fully HTTPS. Once enabled, browsers will refuse HTTP connections.

**To verify:** Check headers in browser DevTools or use securityheaders.com.

**To rollback:** Remove the section between markers.

### WebP Serving (If Not Using Plugin)

Add BEFORE `# BEGIN WordPress`:

```apache
# BEGIN WebP Serving
<IfModule mod_rewrite.c>
    RewriteEngine On

    # Check if browser supports WebP
    RewriteCond %{HTTP_ACCEPT} image/webp

    # Check if WebP version exists
    RewriteCond %{REQUEST_FILENAME}.webp -f

    # Serve WebP instead of jpg/png
    RewriteRule (.+)\.(jpe?g|png)$ $1.$2.webp [T=image/webp,E=REQUEST_image]
</IfModule>

<IfModule mod_headers.c>
    Header append Vary Accept env=REQUEST_image
</IfModule>
# END WebP Serving
```

**Note:** This only works if WebP versions exist alongside originals. Use ShortPixel or similar to generate them.

---

## Code Changes (Full Access Only)

Only proceed if:
- [ ] You have FTP or SSH access
- [ ] You have a confirmed backup
- [ ] Child theme is set up
- [ ] User explicitly approved changes

### Child Theme Setup (REQUIRED)

**Never modify parent theme files.** Always use a child theme.

#### Create Child Theme

1. Create folder: `wp-content/themes/YOUR-THEME-child/`

2. Create `style.css`:
```css
/*
 Theme Name:   Your Theme Child
 Template:     your-theme
 Description:  Child theme for Your Theme
 Version:      1.0.0
*/

/* Add custom CSS below this line */
```

3. Create `functions.php`:
```php
<?php
/**
 * Your Theme Child functions
 */

// Enqueue parent theme styles
add_action('wp_enqueue_scripts', function() {
    wp_enqueue_style('parent-style', get_template_directory_uri() . '/style.css');
    wp_enqueue_style('child-style', get_stylesheet_uri(), array('parent-style'));
});

// Add your customizations below this line
```

4. Activate child theme in Admin > Appearance > Themes

### functions.php Performance Patterns

Add to child theme's `functions.php`:

#### Remove Unnecessary Resources

```php
/**
 * Remove unnecessary WordPress features for performance
 */
add_action('init', function() {
    // Remove emojis (saves HTTP request)
    remove_action('wp_head', 'print_emoji_detection_script', 7);
    remove_action('wp_print_styles', 'print_emoji_styles');
    remove_action('admin_print_scripts', 'print_emoji_detection_script');
    remove_action('admin_print_styles', 'print_emoji_styles');

    // Remove WordPress version (security)
    remove_action('wp_head', 'wp_generator');

    // Remove wlwmanifest link
    remove_action('wp_head', 'wlwmanifest_link');

    // Remove RSD link
    remove_action('wp_head', 'rsd_link');

    // Remove shortlink
    remove_action('wp_head', 'wp_shortlink_wp_head');
});
```

#### Optimize Script Loading

```php
/**
 * Defer non-critical JavaScript
 */
add_filter('script_loader_tag', function($tag, $handle, $src) {
    // Don't defer these scripts
    $no_defer = array('jquery', 'jquery-core', 'jquery-migrate');

    if (in_array($handle, $no_defer)) {
        return $tag;
    }

    // Don't defer admin scripts
    if (is_admin()) {
        return $tag;
    }

    // Add defer attribute
    return str_replace(' src', ' defer src', $tag);
}, 10, 3);
```

#### Preload Critical Resources

```php
/**
 * Preload LCP image and critical fonts
 */
add_action('wp_head', function() {
    // Preconnect to Google Fonts (if used)
    echo '<link rel="preconnect" href="https://fonts.googleapis.com">' . "\n";
    echo '<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>' . "\n";

    // Preload hero image on homepage (adjust path as needed)
    if (is_front_page()) {
        echo '<link rel="preload" as="image" href="' . get_stylesheet_directory_uri() . '/images/hero.webp" fetchpriority="high">' . "\n";
    }
}, 1);
```

#### Add Missing Image Dimensions

```php
/**
 * Add width and height attributes to images (prevents CLS)
 */
add_filter('wp_get_attachment_image_attributes', function($attr, $attachment, $size) {
    if (!isset($attr['width']) || !isset($attr['height'])) {
        $image = wp_get_attachment_image_src($attachment->ID, $size);
        if ($image) {
            $attr['width'] = $image[1];
            $attr['height'] = $image[2];
        }
    }
    return $attr;
}, 10, 3);
```

#### Disable Unnecessary Features

```php
/**
 * Disable features not needed for most sites
 */

// Disable XML-RPC (security + performance)
add_filter('xmlrpc_enabled', '__return_false');

// Disable pingbacks
add_filter('pings_open', '__return_false');

// Limit post revisions (add to wp-config.php instead if possible)
// define('WP_POST_REVISIONS', 5);

// Disable self-pingbacks
add_action('pre_ping', function(&$links) {
    $home = get_option('home');
    foreach ($links as $l => $link) {
        if (strpos($link, $home) === 0) {
            unset($links[$l]);
        }
    }
});
```

---

## WooCommerce Specific

WooCommerce sites have additional optimization needs.

### Cart Fragments Issue

WooCommerce loads cart fragments on every page, causing slow load times.

#### Disable Cart Fragments (If Not Needed)

```php
/**
 * Disable WooCommerce cart fragments on non-cart pages
 */
add_action('wp_enqueue_scripts', function() {
    // Only disable on non-WooCommerce pages
    if (!is_woocommerce() && !is_cart() && !is_checkout()) {
        wp_dequeue_script('wc-cart-fragments');
    }
}, 99);
```

**Warning:** This breaks the cart count in header. Only use if your theme doesn't show cart count or you have a custom solution.

#### Alternative: Lazy Load Cart

```php
/**
 * Load cart fragments only after user interaction
 */
add_action('wp_footer', function() {
    if (!is_cart() && !is_checkout()) {
        ?>
        <script>
        (function() {
            var loaded = false;
            function loadCartFragments() {
                if (loaded) return;
                loaded = true;
                jQuery(document.body).trigger('wc_fragment_refresh');
            }
            // Load on first interaction
            ['mousemove', 'scroll', 'keydown', 'touchstart'].forEach(function(event) {
                document.addEventListener(event, loadCartFragments, {once: true, passive: true});
            });
        })();
        </script>
        <?php
    }
}, 99);
```

### Product Image Optimization

```php
/**
 * Add fetchpriority to main product image
 */
add_filter('woocommerce_single_product_image_thumbnail_html', function($html, $post_thumbnail_id) {
    // Add fetchpriority="high" to first product image
    static $first = true;
    if ($first && strpos($html, 'fetchpriority') === false) {
        $html = str_replace('<img ', '<img fetchpriority="high" ', $html);
        $first = false;
    }
    return $html;
}, 10, 2);
```

---

## Troubleshooting

### If Something Breaks

**Step 1: Don't Panic**

Most WordPress issues can be fixed by restoring files.

**Step 2: Disable Plugins via FTP**

1. Connect via FTP
2. Navigate to `wp-content/`
3. Rename `plugins` folder to `plugins.broken`
4. Site should load (without plugins)
5. Create new `plugins` folder
6. Move plugins back one by one to find culprit

**Step 3: Reset .htaccess**

1. Connect via FTP
2. Rename `.htaccess` to `.htaccess.broken`
3. Create new `.htaccess` with just:
```apache
# BEGIN WordPress
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
</IfModule>
# END WordPress
```

**Step 4: Switch to Default Theme**

1. Connect via FTP
2. Rename your theme folder in `wp-content/themes/`
3. WordPress will fall back to default theme
4. Or via database: Change `template` and `stylesheet` options in `wp_options` table

**Step 5: Restore from Backup**

If all else fails, restore your backup:
1. Upload backed up files
2. Import database via phpMyAdmin
3. Clear any caching

### Common Issues

| Issue | Likely Cause | Fix |
|-------|--------------|-----|
| White screen | PHP error | Enable WP_DEBUG, check error log |
| 500 error | .htaccess or PHP | Reset .htaccess, check PHP version |
| Styles broken | Cache issue | Clear cache, hard refresh |
| Plugin conflict | Two plugins conflicting | Disable plugins one by one |
| Slow backend | Database bloat | Run WP-Optimize |

### Enable Debug Mode

Add to `wp-config.php` temporarily:

```php
define('WP_DEBUG', true);
define('WP_DEBUG_LOG', true);
define('WP_DEBUG_DISPLAY', false);
```

Check `wp-content/debug.log` for errors.

**Remove these lines after debugging!**

---

## Plugin Recommendations Matrix

### Performance Plugins

| Issue | Free Plugin | Premium | Notes |
|-------|-------------|---------|-------|
| Page Cache | WP Super Cache | WP Rocket | WP Rocket is easiest |
| Object Cache | Redis Object Cache | - | Requires Redis server |
| Image Compress | ShortPixel (100/mo) | ShortPixel Pro | Best quality |
| Lazy Load | Native WP 5.5+ | - | Built-in |
| Minify | Autoptimize | WP Rocket | Test carefully |
| CDN | Cloudflare | Cloudflare Pro | Free tier excellent |

### Security Plugins

| Purpose | Free Plugin | Notes |
|---------|-------------|-------|
| Headers | HTTP Headers | Simple setup |
| Firewall | Wordfence | Free tier good |
| SSL | Really Simple SSL | For SSL transition |
| Login Security | Limit Login Attempts | Brute force protection |

### SEO Plugins

| Purpose | Free Plugin | Premium |
|---------|-------------|---------|
| Full SEO | Yoast SEO | Yoast Premium |
| Alternative | Rank Math | Rank Math Pro |
| Sitemap | Yoast SEO | - |
| Schema | Yoast SEO | Schema Pro |

---

## Verification Checklist

After applying optimizations:

### Functionality Check

- [ ] Homepage loads correctly
- [ ] Navigation works
- [ ] Forms submit successfully
- [ ] Cart/checkout works (if WooCommerce)
- [ ] Login works
- [ ] Mobile view works
- [ ] No console errors

### Performance Check

- [ ] Run PageSpeed Insights again
- [ ] Check mobile AND desktop
- [ ] All scores improved or maintained
- [ ] No new issues introduced

### Security Check

- [ ] Site still uses HTTPS
- [ ] Admin login works
- [ ] Security headers present (check securityheaders.com)
- [ ] No sensitive files exposed

---

## Quick Reference

### PageSpeed Issue -> WordPress Fix

| PageSpeed Issue | WordPress Solution |
|-----------------|-------------------|
| Serve images in next-gen formats | ShortPixel + WebP Express |
| Efficiently encode images | ShortPixel compression |
| Properly size images | Regenerate Thumbnails plugin |
| Defer offscreen images | Native lazy loading (WP 5.5+) |
| Eliminate render-blocking resources | Autoptimize |
| Minify CSS/JS | Autoptimize |
| Enable text compression | .htaccess Gzip or plugin |
| Serve static assets with efficient cache policy | .htaccess cache headers |
| Reduce unused CSS | PurifyCSS (manual) or Asset CleanUp |
| Reduce unused JavaScript | Asset CleanUp plugin |
| Preconnect to required origins | Child theme functions.php |
| Avoid document.write() | Usually plugin issue - identify & replace |
| Uses deprecated APIs | Update WordPress & plugins |
| Missing security headers | .htaccess or HTTP Headers plugin |
