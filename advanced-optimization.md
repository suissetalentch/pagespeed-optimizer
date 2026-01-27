# Advanced Optimization Guide

Advanced optimization techniques focusing on local development and application-level improvements.

## Database Optimization

### N+1 Query Detection

#### Laravel Debugbar

```bash
composer require barryvdh/laravel-debugbar --dev
```

```php
// config/debugbar.php
'collectors' => [
    'queries' => true,
    'models'  => true,
],
```

#### Clockwork (Framework-agnostic)

```bash
composer require itsgoingd/clockwork --dev
```

Check the browser extension or `/__clockwork` endpoint for query analysis.

### Eager Loading Patterns

```php
// BAD: N+1 queries
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->author->name; // Query per post!
}

// GOOD: Eager loading
$posts = Post::with('author')->get();
foreach ($posts as $post) {
    echo $post->author->name; // No additional queries
}

// Nested eager loading
$posts = Post::with(['author', 'comments.user'])->get();

// Conditional eager loading
$posts = Post::with(['comments' => function ($query) {
    $query->where('approved', true);
}])->get();
```

### Query Caching

#### Laravel Cache

```php
// Cache expensive queries
$popularPosts = Cache::remember('popular_posts', 3600, function () {
    return Post::withCount('views')
        ->orderByDesc('views_count')
        ->limit(10)
        ->get();
});

// Invalidate on update
Post::saved(function () {
    Cache::forget('popular_posts');
});
```

#### Redis Configuration

```php
// config/database.php
'redis' => [
    'client' => 'phpredis',
    'default' => [
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'port' => env('REDIS_PORT', 6379),
        'database' => env('REDIS_DB', 0),
    ],
    'cache' => [
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'port' => env('REDIS_PORT', 6379),
        'database' => env('REDIS_CACHE_DB', 1),
    ],
],
```

### Index Optimization

```sql
-- Identify slow queries
EXPLAIN ANALYZE SELECT * FROM posts WHERE user_id = 123;

-- Add missing indexes
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_created_at ON posts(created_at);

-- Composite index for common queries
CREATE INDEX idx_posts_user_status ON posts(user_id, status);

-- Check existing indexes
SHOW INDEX FROM posts;
```

## PWA & Offline Support

### manifest.json Template

```json
{
  "name": "My Application",
  "short_name": "MyApp",
  "description": "A fast, reliable progressive web app",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#3b82f6",
  "icons": [
    {
      "src": "/icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512-maskable.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable"
    }
  ]
}
```

```html
<!-- Link in HTML head -->
<link rel="manifest" href="/manifest.json">
<meta name="theme-color" content="#3b82f6">
<link rel="apple-touch-icon" href="/icons/icon-192.png">
```

### Basic Service Worker

```javascript
// sw.js
const CACHE_NAME = 'v1';
const STATIC_ASSETS = [
  '/',
  '/offline.html',
  '/css/app.css',
  '/js/app.js',
];

// Install: cache static assets
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.addAll(STATIC_ASSETS);
    })
  );
});

// Activate: clean old caches
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then((keys) => {
      return Promise.all(
        keys.filter((key) => key !== CACHE_NAME)
            .map((key) => caches.delete(key))
      );
    })
  );
});

// Fetch: network first, fallback to cache
self.addEventListener('fetch', (event) => {
  event.respondWith(
    fetch(event.request)
      .catch(() => caches.match(event.request))
      .then((response) => response || caches.match('/offline.html'))
  );
});
```

```javascript
// Register in main.js
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js');
}
```

### Workbox Configuration

```javascript
// workbox-config.js
module.exports = {
  globDirectory: 'dist/',
  globPatterns: [
    '**/*.{html,js,css,png,jpg,webp,woff2}'
  ],
  swDest: 'dist/sw.js',
  runtimeCaching: [
    {
      urlPattern: /\.(?:png|jpg|jpeg|svg|webp)$/,
      handler: 'CacheFirst',
      options: {
        cacheName: 'images',
        expiration: {
          maxEntries: 50,
          maxAgeSeconds: 30 * 24 * 60 * 60, // 30 days
        },
      },
    },
    {
      urlPattern: /^https:\/\/api\./,
      handler: 'NetworkFirst',
      options: {
        cacheName: 'api',
        expiration: {
          maxEntries: 100,
          maxAgeSeconds: 24 * 60 * 60, // 24 hours
        },
      },
    },
  ],
};
```

```bash
# Generate service worker
npx workbox generateSW workbox-config.js
```

## Local Caching Strategies

### Browser Cache Headers

```php
// Laravel middleware
public function handle($request, Closure $next)
{
    $response = $next($request);
    $path = $request->path();

    // Static assets: 1 year cache
    if (preg_match('/\.(js|css|png|jpg|webp|woff2)$/', $path)) {
        $response->header('Cache-Control', 'public, max-age=31536000, immutable');
    }

    // HTML: no cache, revalidate
    if (preg_match('/\.(html)$/', $path) || !str_contains($path, '.')) {
        $response->header('Cache-Control', 'no-cache, must-revalidate');
    }

    return $response;
}
```

```nginx
# Nginx configuration
location ~* \.(js|css|png|jpg|jpeg|gif|webp|svg|woff2)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}

location ~* \.html$ {
    expires -1;
    add_header Cache-Control "no-cache, must-revalidate";
}
```

### Application-Level Caching

#### View Caching (Laravel)

```bash
# Cache views for production
php artisan view:cache

# Clear when needed
php artisan view:clear
```

#### Config Caching

```bash
# Cache configuration
php artisan config:cache

# Cache routes
php artisan route:cache

# Single command for production
php artisan optimize
```

#### Data Caching Patterns

```php
// Cache computed data
class ProductService
{
    public function getFeaturedProducts(): Collection
    {
        return Cache::remember('featured_products', 3600, function () {
            return Product::where('featured', true)
                ->with(['images', 'category'])
                ->get();
        });
    }

    public function invalidate(): void
    {
        Cache::forget('featured_products');
    }
}
```

### Static File Optimization

#### Hashed Asset Names (Vite)

```javascript
// vite.config.js
export default {
  build: {
    rollupOptions: {
      output: {
        entryFileNames: 'js/[name].[hash].js',
        chunkFileNames: 'js/[name].[hash].js',
        assetFileNames: 'assets/[name].[hash].[ext]',
      },
    },
  },
};
```

#### Compression

```bash
# Gzip static files
gzip -k dist/**/*.{js,css,html,svg}

# Brotli (better compression)
brotli dist/**/*.{js,css,html,svg}
```

```nginx
# Nginx: serve pre-compressed files
gzip_static on;
brotli_static on;
```

## Performance Monitoring

### Local Lighthouse Automation

```bash
# Install Lighthouse CI
npm install -g @lhci/cli

# Run audit
lhci autorun --collect.url=http://localhost:3000

# With configuration
# lighthouserc.js
module.exports = {
  ci: {
    collect: {
      url: ['http://localhost:3000/', 'http://localhost:3000/products'],
      numberOfRuns: 3,
    },
    assert: {
      assertions: {
        'categories:performance': ['error', { minScore: 0.9 }],
        'categories:accessibility': ['error', { minScore: 0.9 }],
      },
    },
  },
};
```

### Web Vitals Tracking

```javascript
// Track Core Web Vitals locally
import { onCLS, onFCP, onLCP, onTTFB } from 'web-vitals';

function logMetric({ name, value, rating }) {
  console.log(`${name}: ${value} (${rating})`);
}

onCLS(logMetric);
onFCP(logMetric);
onLCP(logMetric);
onTTFB(logMetric);
```

## Optimization Checklist

| Category | Task | Priority | Done |
|----------|------|----------|------|
| Database | Fix N+1 queries | HIGH | [ ] |
| Database | Add eager loading | HIGH | [ ] |
| Database | Add missing indexes | MEDIUM | [ ] |
| Database | Implement query caching | MEDIUM | [ ] |
| PWA | Add manifest.json | LOW | [ ] |
| PWA | Register service worker | LOW | [ ] |
| Caching | Configure browser cache headers | HIGH | [ ] |
| Caching | Enable view/config caching | HIGH | [ ] |
| Caching | Hash asset filenames | HIGH | [ ] |
| Monitoring | Set up Lighthouse CI | MEDIUM | [ ] |
