# Laravel + Vite + Alpine.js Patterns

## Layout Optimization (app.blade.php)

### Font Loading Pattern

```blade
{{-- In <head> section, BEFORE @vite directive --}}

<!-- DNS Prefetch for external resources -->
<link rel="dns-prefetch" href="https://fonts.googleapis.com">
<link rel="dns-prefetch" href="https://fonts.gstatic.com">
<link rel="dns-prefetch" href="https://www.google-analytics.com">

<!-- Preconnect for Google Fonts (critical) -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

<!-- Google Fonts with non-blocking loading -->
<link rel="preload" as="style" href="https://fonts.googleapis.com/css2?family=...&display=swap">
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=...&display=swap" media="print" onload="this.media='all'">
<noscript>
    <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=...&display=swap">
</noscript>

<!-- Preload critical assets -->
<link rel="preload" as="image" href="{{ asset('images/logo.png') }}" type="image/png">

@vite(['resources/css/app.css', 'resources/js/app.js'])
```

### Conditional Script Loading

```blade
{{-- Load external scripts only where needed --}}
@if(request()->routeIs('contact') || request()->routeIs('contact.submit'))
<script src="https://challenges.cloudflare.com/turnstile/v0/api.js" async defer></script>
@endif

@if(request()->routeIs('videos.*'))
<script src="https://www.youtube.com/iframe_api" async defer></script>
@endif
```

### Form Accessibility Pattern

```blade
{{-- Contact form with full accessibility --}}
<form action="{{ route('contact.submit') }}" method="POST">
    @csrf

    {{-- Honeypot (spam protection) --}}
    <div class="hidden" aria-hidden="true" style="position: absolute; left: -9999px;">
        <input type="text" name="website" tabindex="-1" autocomplete="off">
    </div>

    {{-- Name field --}}
    <div>
        <label for="name" class="block text-sm font-medium text-gray-700">
            {{ __('Full Name') }}
            <span class="text-red-600" aria-hidden="true">*</span>
        </label>
        <input
            type="text"
            id="name"
            name="name"
            value="{{ old('name') }}"
            required
            aria-required="true"
            autocomplete="name"
            class="..."
        >
        @error('name')
        <p class="mt-1 text-sm text-red-600" role="alert">{{ $message }}</p>
        @enderror
    </div>

    {{-- Email field --}}
    <div>
        <label for="email">{{ __('Email') }} <span aria-hidden="true">*</span></label>
        <input
            type="email"
            id="email"
            name="email"
            required
            aria-required="true"
            autocomplete="email"
        >
    </div>

    {{-- Phone field --}}
    <div>
        <label for="phone">{{ __('Phone') }}</label>
        <input
            type="tel"
            id="phone"
            name="phone"
            autocomplete="tel"
        >
    </div>

    {{-- Message field --}}
    <div>
        <label for="message">{{ __('Message') }} <span aria-hidden="true">*</span></label>
        <textarea
            id="message"
            name="message"
            required
            aria-required="true"
            rows="4"
        >{{ old('message') }}</textarea>
    </div>

    <button type="submit">{{ __('Send') }}</button>
</form>
```

## CSS Optimization (app.css)

### Remove Font Imports
```css
/* REMOVE THIS - blocks rendering */
/* @import url('https://fonts.googleapis.com/...'); */

/* Keep only Tailwind directives */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### Enhanced Focus States
```css
@layer base {
    /* General focus-visible */
    *:focus-visible {
        @apply outline-2 outline-offset-2 outline-primary-500 rounded-sm;
    }

    /* Interactive elements */
    a:focus-visible,
    button:focus-visible,
    [role="button"]:focus-visible {
        @apply ring-2 ring-primary-500 ring-offset-2 outline-none;
    }

    /* Form inputs */
    input:focus-visible,
    textarea:focus-visible,
    select:focus-visible {
        @apply ring-2 ring-primary-400 ring-offset-1 outline-none border-primary-400;
    }
}
```

### Aspect Ratio Utilities
```css
@layer utilities {
    .aspect-hero {
        aspect-ratio: 16 / 9;
    }
    .aspect-team {
        aspect-ratio: 3 / 4;
    }
    .aspect-card {
        aspect-ratio: 4 / 3;
    }
}
```

## Vite Configuration

### vite.config.js with Code Splitting
```javascript
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
    build: {
        rollupOptions: {
            output: {
                manualChunks: {
                    // Separate Alpine.js into its own chunk
                    alpine: ['alpinejs', '@alpinejs/collapse'],
                },
            },
        },
        cssCodeSplit: true,
        // Use esbuild (built-in, faster than terser)
        minify: 'esbuild',
        esbuild: {
            drop: ['console', 'debugger'],
        },
    },
});
```

## Security Headers Middleware

### app/Http/Middleware/SecurityHeaders.php
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class SecurityHeaders
{
    protected array $headers = [
        'X-Content-Type-Options' => 'nosniff',
        'X-Frame-Options' => 'SAMEORIGIN',
        'X-XSS-Protection' => '1; mode=block',
        'Referrer-Policy' => 'strict-origin-when-cross-origin',
        'Permissions-Policy' => 'accelerometer=(), camera=(), geolocation=(), gyroscope=(), magnetometer=(), microphone=(), payment=(), usb=()',
    ];

    public function handle(Request $request, Closure $next): Response
    {
        $response = $next($request);

        // Security headers
        foreach ($this->headers as $header => $value) {
            $response->headers->set($header, $value);
        }

        // HSTS
        if ($request->secure() || config('app.env') === 'production') {
            $response->headers->set(
                'Strict-Transport-Security',
                'max-age=31536000; includeSubDomains; preload'
            );
        }

        // CSP
        $response->headers->set('Content-Security-Policy', $this->getCSP());

        // Cache headers
        $this->addCacheHeaders($request, $response);

        return $response;
    }

    protected function addCacheHeaders(Request $request, Response $response): void
    {
        $path = $request->path();

        // Vite assets (have hash in filename)
        if (str_starts_with($path, 'build/')) {
            $response->headers->set('Cache-Control', 'public, max-age=31536000, immutable');
            return;
        }

        // Images
        if (str_starts_with($path, 'images/')) {
            $response->headers->set('Cache-Control', 'public, max-age=86400');
            return;
        }

        // Static files
        if (preg_match('/\.(ico|png|jpg|jpeg|gif|webp|svg|woff2?|ttf|eot)$/i', $path)) {
            $response->headers->set('Cache-Control', 'public, max-age=604800');
            return;
        }

        // HTML
        if ($request->acceptsHtml()) {
            $response->headers->set('Cache-Control', 'no-cache, must-revalidate');
        }
    }

    protected function getCSP(): string
    {
        return implode('; ', [
            "default-src 'self'",
            "script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.googletagmanager.com https://challenges.cloudflare.com",
            "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
            "img-src 'self' data: https: blob:",
            "font-src 'self' https://fonts.gstatic.com data:",
            "connect-src 'self' https://www.google-analytics.com",
            "frame-src 'self' https://www.youtube.com https://challenges.cloudflare.com",
            "frame-ancestors 'self'",
            "form-action 'self'",
            "base-uri 'self'",
            "object-src 'none'",
            "upgrade-insecure-requests",
        ]);
    }
}
```

## Blade Components

### Hero Image Component
```blade
{{-- resources/views/components/hero-image.blade.php --}}
@props([
    'src',
    'alt',
    'isLCP' => false,
    'class' => ''
])

<img
    src="{{ $src }}"
    alt="{{ $alt }}"
    class="{{ $class }}"
    loading="{{ $isLCP ? 'eager' : 'lazy' }}"
    fetchpriority="{{ $isLCP ? 'high' : 'auto' }}"
    decoding="async"
    {{ $attributes }}
>
```

### Accessible Button Component
```blade
{{-- resources/views/components/icon-button.blade.php --}}
@props([
    'label',
    'type' => 'button'
])

<button
    type="{{ $type }}"
    aria-label="{{ $label }}"
    {{ $attributes->merge(['class' => 'p-2 rounded-lg hover:bg-gray-100 transition']) }}
>
    {{ $slot }}
</button>
```

## Environment Configuration

### .env for Production HTTPS
```env
APP_URL=https://example.com
ASSET_URL=https://example.com

# Forces HTTPS asset URLs (prevents mixed content)
```

## Common Laravel + Tailwind Contrast Fixes

### Problem: text-primary-600 on white background
```css
/* If primary-600 fails contrast, use primary-700 */
.text-primary-600 {
    @apply text-primary-700; /* Darker for better contrast */
}

/* Or adjust the color scale in tailwind.config.js */
primary: {
    600: '#059669', /* Adjusted for 4.5:1 contrast */
}
```
