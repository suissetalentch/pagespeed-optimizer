# Vue 3 + Nuxt 3 Performance Patterns

Optimization patterns for Vue 3 and Nuxt 3 applications to achieve 100% PageSpeed scores.

## Detection

This file applies when your project contains:
- `nuxt.config.ts` or `nuxt.config.js`
- `package.json` with `nuxt` dependency

## Nuxt Configuration

### nuxt.config.ts - Optimized

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  // Enable SSR for better LCP
  ssr: true,

  // Nitro server optimization
  nitro: {
    compressPublicAssets: true,
    prerender: {
      crawlLinks: true,
      routes: ['/'],
    },
  },

  // Image optimization
  image: {
    provider: 'ipx',
    quality: 80,
    format: ['webp', 'avif', 'png', 'jpg'],
    screens: {
      xs: 320,
      sm: 640,
      md: 768,
      lg: 1024,
      xl: 1280,
      xxl: 1536,
    },
  },

  // Font optimization with fontaine
  fontMetrics: {
    fonts: ['Inter', 'Roboto'],
  },

  // Build optimization
  vite: {
    build: {
      cssMinify: true,
      minify: 'esbuild',
      rollupOptions: {
        output: {
          manualChunks: {
            vue: ['vue', 'vue-router'],
          },
        },
      },
    },
    esbuild: {
      drop: ['console', 'debugger'],
    },
  },

  // Experimental features for better performance
  experimental: {
    payloadExtraction: true,
    componentIslands: true,
  },

  // App head configuration
  app: {
    head: {
      htmlAttrs: { lang: 'en' },
      meta: [
        { charset: 'utf-8' },
        { name: 'viewport', content: 'width=device-width, initial-scale=1' },
      ],
      link: [
        { rel: 'preconnect', href: 'https://fonts.googleapis.com' },
        { rel: 'preconnect', href: 'https://fonts.gstatic.com', crossorigin: '' },
      ],
    },
  },
})
```

### Required Modules

```bash
# Install optimization modules
npx nuxi module add @nuxt/image
npx nuxi module add @nuxtjs/fontaine
```

## Components

### LazyImage Component

```vue
<!-- components/LazyImage.vue -->
<template>
  <div class="lazy-image-container" :style="containerStyle">
    <NuxtImg
      v-if="loaded || eager"
      :src="src"
      :alt="alt"
      :width="width"
      :height="height"
      :loading="eager ? 'eager' : 'lazy'"
      :fetchpriority="eager ? 'high' : 'auto'"
      format="webp"
      quality="80"
      :sizes="sizes"
      @load="onLoad"
    />
    <div v-else class="placeholder" :class="{ 'animate-pulse': !loaded }" />
  </div>
</template>

<script setup lang="ts">
interface Props {
  src: string
  alt: string
  width: number
  height: number
  eager?: boolean
  sizes?: string
}

const props = withDefaults(defineProps<Props>(), {
  eager: false,
  sizes: '100vw',
})

const loaded = ref(false)

const containerStyle = computed(() => ({
  aspectRatio: `${props.width} / ${props.height}`,
}))

const onLoad = () => {
  loaded.value = true
}
</script>

<style scoped>
.lazy-image-container {
  position: relative;
  overflow: hidden;
}

.placeholder {
  position: absolute;
  inset: 0;
  background: #e5e7eb;
}

.animate-pulse {
  animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}
</style>
```

### Usage

```vue
<!-- LCP image - eager loading -->
<LazyImage
  src="/hero.jpg"
  alt="Hero image"
  :width="1200"
  :height="600"
  eager
  sizes="100vw"
/>

<!-- Below fold - lazy loading -->
<LazyImage
  src="/product.jpg"
  alt="Product image"
  :width="400"
  :height="300"
  sizes="(max-width: 768px) 100vw, 400px"
/>
```

### Accessible Form Component

```vue
<!-- components/FormInput.vue -->
<template>
  <div class="form-group">
    <label :for="id" class="label">
      {{ label }}
      <span v-if="required" aria-hidden="true">*</span>
    </label>
    <input
      :id="id"
      v-model="model"
      :type="type"
      :name="name"
      :autocomplete="autocomplete"
      :aria-required="required"
      :aria-describedby="error ? `${id}-error` : undefined"
      :aria-invalid="error ? 'true' : undefined"
      class="input"
      :class="{ 'input-error': error }"
    />
    <p v-if="error" :id="`${id}-error`" class="error" role="alert">
      {{ error }}
    </p>
  </div>
</template>

<script setup lang="ts">
interface Props {
  id: string
  label: string
  name: string
  type?: string
  autocomplete?: string
  required?: boolean
  error?: string
}

const props = withDefaults(defineProps<Props>(), {
  type: 'text',
  autocomplete: 'off',
  required: false,
})

const model = defineModel<string>()
</script>
```

## Server Middleware

### Security Headers

```typescript
// server/middleware/security.ts
export default defineEventHandler((event) => {
  const headers = {
    'X-Content-Type-Options': 'nosniff',
    'X-Frame-Options': 'SAMEORIGIN',
    'X-XSS-Protection': '1; mode=block',
    'Referrer-Policy': 'strict-origin-when-cross-origin',
    'Permissions-Policy': 'accelerometer=(), camera=(), geolocation=(), microphone=()',
    'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',
  }

  Object.entries(headers).forEach(([key, value]) => {
    setHeader(event, key, value)
  })
})
```

### Cache Headers for Static Assets

```typescript
// server/middleware/cache.ts
export default defineEventHandler((event) => {
  const url = getRequestURL(event)

  // Static assets - long cache
  if (url.pathname.match(/\.(js|css|woff2?|png|jpg|webp|avif|svg)$/)) {
    setHeader(event, 'Cache-Control', 'public, max-age=31536000, immutable')
  }

  // HTML - short cache with revalidation
  if (url.pathname.endsWith('.html') || !url.pathname.includes('.')) {
    setHeader(event, 'Cache-Control', 'public, max-age=0, must-revalidate')
  }
})
```

## SEO & Meta

### useSeoMeta Composable

```vue
<script setup lang="ts">
useSeoMeta({
  title: 'Page Title - Site Name',
  description: 'A 150-160 character description of the page content.',
  ogTitle: 'Page Title - Site Name',
  ogDescription: 'A 150-160 character description of the page content.',
  ogImage: '/og-image.jpg',
  ogType: 'website',
  twitterCard: 'summary_large_image',
})

useHead({
  link: [
    { rel: 'canonical', href: 'https://example.com/page' },
  ],
})
</script>
```

## Performance Checklist

| Task | Priority | Done |
|------|----------|------|
| Enable SSR in nuxt.config.ts | CRITICAL | [ ] |
| Install @nuxt/image module | CRITICAL | [ ] |
| Add fetchpriority="high" to LCP image | CRITICAL | [ ] |
| Configure security headers middleware | HIGH | [ ] |
| Add font preconnect links | HIGH | [ ] |
| Enable payload extraction | MEDIUM | [ ] |
| Configure cache headers | MEDIUM | [ ] |
| Add useSeoMeta to pages | HIGH | [ ] |
| Remove console.log in production | MEDIUM | [ ] |

## Common Issues

### Hydration Mismatch

```vue
<!-- Wrap client-only content -->
<ClientOnly>
  <DynamicComponent />
  <template #fallback>
    <div class="skeleton" />
  </template>
</ClientOnly>
```

### Large Bundle Size

```typescript
// Use dynamic imports for heavy components
const HeavyChart = defineAsyncComponent(() => import('./HeavyChart.vue'))
```

### Image Not Optimized

```vue
<!-- Use NuxtImg instead of img -->
<NuxtImg
  src="/photo.jpg"
  alt="Description"
  width="800"
  height="600"
  format="webp"
  quality="80"
  loading="lazy"
/>
```
