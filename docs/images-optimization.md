# Image Optimization Guide

Comprehensive guide for optimizing images to improve LCP and overall performance.

## Modern Formats

### Format Comparison

| Format | Compression | Browser Support | Best For |
|--------|-------------|-----------------|----------|
| **WebP** | 25-35% smaller than JPEG | 97%+ | Photos, graphics |
| **AVIF** | 50% smaller than JPEG | 92%+ | Photos (limited support) |
| **PNG** | Lossless | 100% | Graphics with transparency |
| **JPEG** | Lossy | 100% | Fallback for photos |
| **SVG** | Vector | 100% | Icons, logos, illustrations |

### Recommended Strategy

```html
<!-- Modern format with fallbacks -->
<picture>
  <source srcset="image.avif" type="image/avif">
  <source srcset="image.webp" type="image/webp">
  <img src="image.jpg" alt="Description" width="800" height="600">
</picture>
```

## Command Line Tools

### Sharp (Node.js) - Recommended

```bash
# Install
npm install sharp

# Convert to WebP
npx sharp -i input.jpg -o output.webp --format webp --quality 80

# Resize and convert
npx sharp -i input.jpg -o output.webp --width 800 --format webp --quality 80
```

```javascript
// sharp-convert.js
const sharp = require('sharp');

async function optimizeImage(input, output) {
  await sharp(input)
    .resize(1200, 800, { fit: 'inside', withoutEnlargement: true })
    .webp({ quality: 80 })
    .toFile(output);
}

// Batch process
const glob = require('glob');
glob.sync('images/*.{jpg,png}').forEach(file => {
  const output = file.replace(/\.(jpg|png)$/, '.webp');
  optimizeImage(file, output);
});
```

### Squoosh CLI

```bash
# Install
npm install @squoosh/cli

# Convert to WebP
npx squoosh-cli --webp '{"quality":80}' -d output/ input.jpg

# Convert to AVIF
npx squoosh-cli --avif '{"quality":60}' -d output/ input.jpg

# Resize and convert
npx squoosh-cli --resize '{"width":800}' --webp '{"quality":80}' input.jpg
```

### ImageMagick

```bash
# Convert to WebP
convert input.jpg -quality 80 output.webp

# Batch convert
for f in *.jpg; do convert "$f" -quality 80 "${f%.jpg}.webp"; done

# Resize
convert input.jpg -resize 800x600 -quality 80 output.webp
```

## Responsive Images

### Basic srcset

```html
<img
  src="photo-800.jpg"
  srcset="
    photo-400.jpg 400w,
    photo-800.jpg 800w,
    photo-1200.jpg 1200w,
    photo-1600.jpg 1600w
  "
  sizes="(max-width: 640px) 100vw,
         (max-width: 1024px) 50vw,
         800px"
  alt="Description"
  width="800"
  height="600"
  loading="lazy"
>
```

### Art Direction with Picture

```html
<picture>
  <!-- Mobile: cropped portrait -->
  <source
    media="(max-width: 640px)"
    srcset="hero-mobile.webp"
    type="image/webp"
  >
  <!-- Tablet: cropped landscape -->
  <source
    media="(max-width: 1024px)"
    srcset="hero-tablet.webp"
    type="image/webp"
  >
  <!-- Desktop: full image -->
  <source
    srcset="hero-desktop.webp"
    type="image/webp"
  >
  <img
    src="hero-desktop.jpg"
    alt="Hero image"
    width="1600"
    height="900"
    fetchpriority="high"
  >
</picture>
```

### Sizes Attribute Cheat Sheet

```html
<!-- Full width on mobile, half on tablet, fixed on desktop -->
sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 500px"

<!-- Always full width -->
sizes="100vw"

<!-- Grid layouts -->
sizes="(max-width: 640px) 100vw, (max-width: 1024px) 33vw, 300px"
```

## LCP Image Optimization

### Critical Image Markup

```html
<!-- Preload the LCP image -->
<link
  rel="preload"
  as="image"
  href="/hero.webp"
  fetchpriority="high"
  type="image/webp"
>

<!-- LCP image with all optimizations -->
<img
  src="/hero.webp"
  alt="Hero description"
  width="1200"
  height="600"
  loading="eager"
  fetchpriority="high"
  decoding="sync"
>
```

### What NOT to Do

```html
<!-- BAD: Missing dimensions causes CLS -->
<img src="photo.jpg" alt="Photo">

<!-- BAD: Lazy loading LCP image -->
<img src="hero.jpg" alt="Hero" loading="lazy">

<!-- BAD: Using CSS background for LCP -->
<div style="background-image: url('hero.jpg')"></div>
```

## Build Tool Integration

### Vite Plugin

```javascript
// vite.config.js
import { imagetools } from 'vite-imagetools';

export default {
  plugins: [
    imagetools({
      defaultDirectives: new URLSearchParams({
        format: 'webp',
        quality: '80',
      }),
    }),
  ],
};
```

```javascript
// Usage in code
import heroUrl from './hero.jpg?w=800&format=webp';
```

### Next.js Image

```jsx
import Image from 'next/image';

// Automatic optimization
<Image
  src="/hero.jpg"
  alt="Hero"
  width={1200}
  height={600}
  priority // For LCP images
  quality={80}
/>
```

### Nuxt Image

```vue
<NuxtImg
  src="/hero.jpg"
  alt="Hero"
  width="1200"
  height="600"
  format="webp"
  quality="80"
  loading="eager"
  fetchpriority="high"
/>
```

## Compression Quality Guidelines

| Image Type | WebP Quality | AVIF Quality | Notes |
|------------|--------------|--------------|-------|
| Hero/LCP | 80-85 | 65-70 | Balance quality and size |
| Product photos | 75-80 | 60-65 | Need good detail |
| Thumbnails | 70-75 | 55-60 | Smaller = more compression |
| Screenshots | 85-90 | 70-75 | Text needs clarity |
| Decorative | 60-70 | 50-55 | Less critical |

## Mobile Image Budget

### Why Mobile Budget Matters

Mobile Lighthouse simulates 4G: **~1.6 Mbps = 200KB/second**

This means a 500KB image takes ~2.5 seconds to download on mobile, which alone can fail LCP.

### Calculate Your LCP Image Budget

```
Target LCP: 2.5 seconds

Time breakdown:
├── DNS + TCP + TLS:  ~600ms
├── Server response:  ~200ms
├── HTML parse:       ~100ms
└── Available:        ~1.6s for LCP image

Max LCP image: 1.6s × 200KB/s = 320KB
Safe target:   100KB (accounts for network variance)
```

### Mobile Image Size Guidelines

| Image Type | Max Size | Rationale |
|------------|----------|-----------|
| **LCP image** | **100KB** | Must load in < 1.6s |
| Hero background | 80KB | Critical path |
| Product image | 50KB | Multiple on page |
| Thumbnail | 15KB | Many on page |
| Icon/Logo | 5KB | Should be SVG |

### Mobile-First srcset

Always list mobile sizes first (smallest to largest):

```html
<img
  src="hero-400.webp"
  srcset="
    hero-400.webp 400w,
    hero-800.webp 800w,
    hero-1200.webp 1200w
  "
  sizes="(max-width: 640px) 100vw, 50vw"
  alt="Hero image"
  width="400"
  height="300"
  loading="eager"
  fetchpriority="high"
>
```

**Key points:**
- `src` defaults to mobile size (400w)
- Mobile breakpoint comes first in `sizes`
- Smallest image first in `srcset`

### Mobile Image Sizing Reference

| Mobile Viewport | Full Width | Half Width | Third Width |
|-----------------|------------|------------|-------------|
| 412px (Lighthouse) | 412px | 206px | 137px |
| 375px (iPhone) | 375px | 187px | 125px |
| 360px (Android) | 360px | 180px | 120px |

### Quick Size Check

```bash
# Find images over mobile budget
find public -type f \( -name "*.jpg" -o -name "*.png" -o -name "*.webp" \) \
  -size +100k -exec ls -lh {} \;

# Get exact file sizes
ls -lhS public/images/*.{jpg,png,webp} 2>/dev/null | head -10
```

## Performance Checklist

| Task | Impact | Done |
|------|--------|------|
| LCP image < 100KB | CRITICAL | [ ] |
| Convert images to WebP | HIGH | [ ] |
| Add width/height to all images | HIGH | [ ] |
| Preload LCP image | HIGH | [ ] |
| Add fetchpriority="high" to LCP | HIGH | [ ] |
| Implement srcset for responsive | MEDIUM | [ ] |
| Use lazy loading for below-fold | MEDIUM | [ ] |
| Remove unused images | MEDIUM | [ ] |
| Compress to target quality | MEDIUM | [ ] |

## Quick Diagnostic

```bash
# Find large images (>100KB)
find public -type f \( -name "*.jpg" -o -name "*.png" \) -size +100k -exec ls -lh {} \;

# Find images without WebP alternative
for f in public/**/*.jpg; do [ ! -f "${f%.jpg}.webp" ] && echo "Missing: $f"; done

# Check image dimensions
identify public/images/*.jpg  # requires ImageMagick
```
