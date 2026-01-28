# Next.js Patterns

## Image Optimization (Built-in)

### Using next/image
```tsx
import Image from 'next/image';

// Hero image (LCP)
<Image
  src="/hero.jpg"
  alt="Hero description"
  width={1200}
  height={600}
  priority // Preloads for LCP
  quality={85}
/>

// Regular image (lazy loaded by default)
<Image
  src="/team/photo.jpg"
  alt="Team member"
  width={400}
  height={500}
  loading="lazy"
/>

// Fill container with aspect ratio
<div className="relative aspect-video">
  <Image
    src="/video-thumb.jpg"
    alt="Video thumbnail"
    fill
    className="object-cover"
  />
</div>
```

## Font Optimization

### Using next/font (Recommended)
```tsx
// app/layout.tsx or pages/_app.tsx
import { Inter, Lora } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
});

const lora = Lora({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-lora',
});

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={`${inter.variable} ${lora.variable}`}>
      <body className="font-sans">{children}</body>
    </html>
  );
}
```

### tailwind.config.ts Integration
```typescript
import type { Config } from 'tailwindcss';

const config: Config = {
  theme: {
    extend: {
      fontFamily: {
        sans: ['var(--font-inter)', 'system-ui', 'sans-serif'],
        serif: ['var(--font-lora)', 'Georgia', 'serif'],
      },
    },
  },
};

export default config;
```

## Script Optimization

### Using next/script
```tsx
import Script from 'next/script';

// Analytics (load after interactive)
<Script
  src="https://www.googletagmanager.com/gtag/js?id=GA_ID"
  strategy="afterInteractive"
/>

// Lazy load third-party widget
<Script
  src="https://widget.example.com/script.js"
  strategy="lazyOnload"
/>

// Critical script (blocks hydration - use sparingly)
<Script
  src="/critical.js"
  strategy="beforeInteractive"
/>
```

### Conditional Script Loading
```tsx
// app/contact/page.tsx
import Script from 'next/script';

export default function ContactPage() {
  return (
    <>
      {/* Only loads on contact page */}
      <Script
        src="https://challenges.cloudflare.com/turnstile/v0/api.js"
        strategy="lazyOnload"
      />
      <ContactForm />
    </>
  );
}
```

## Metadata & SEO

### App Router (app directory)
```tsx
// app/layout.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    default: 'Site Name',
    template: '%s | Site Name',
  },
  description: 'Default description',
  openGraph: {
    type: 'website',
    locale: 'en_US',
    url: 'https://example.com',
    siteName: 'Site Name',
  },
  robots: {
    index: true,
    follow: true,
  },
};

// app/about/page.tsx
export const metadata: Metadata = {
  title: 'About Us',
  description: 'Learn about our company',
};
```

### Dynamic Metadata
```tsx
// app/blog/[slug]/page.tsx
import type { Metadata } from 'next';

type Props = {
  params: { slug: string };
};

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const post = await getPost(params.slug);

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      images: [post.coverImage],
    },
  };
}
```

## Security Headers

### next.config.js
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'X-Frame-Options', value: 'SAMEORIGIN' },
          { key: 'X-XSS-Protection', value: '1; mode=block' },
          { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
          {
            key: 'Permissions-Policy',
            value: 'accelerometer=(), camera=(), geolocation=(), gyroscope=(), magnetometer=(), microphone=(), payment=(), usb=()',
          },
          {
            key: 'Content-Security-Policy',
            value: "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:;",
          },
        ],
      },
      {
        // Cache static assets
        source: '/_next/static/:path*',
        headers: [
          { key: 'Cache-Control', value: 'public, max-age=31536000, immutable' },
        ],
      },
      {
        // Cache images
        source: '/images/:path*',
        headers: [
          { key: 'Cache-Control', value: 'public, max-age=86400' },
        ],
      },
    ];
  },
};

module.exports = nextConfig;
```

## Dynamic Imports

### Component-level Code Splitting
```tsx
import dynamic from 'next/dynamic';

// Heavy chart component
const Chart = dynamic(() => import('@/components/Chart'), {
  loading: () => <ChartSkeleton />,
  ssr: false, // Disable SSR for client-only components
});

// Map component
const Map = dynamic(() => import('@/components/Map'), {
  loading: () => <div className="h-96 bg-gray-100 animate-pulse" />,
  ssr: false,
});
```

## Accessible Forms

### Form Component
```tsx
'use client';

import { useFormState } from 'react-dom';
import { submitContact } from '@/actions/contact';

export function ContactForm() {
  const [state, formAction] = useFormState(submitContact, { message: '' });

  return (
    <form action={formAction}>
      <div>
        <label htmlFor="name">
          Full Name <span aria-hidden="true">*</span>
        </label>
        <input
          id="name"
          name="name"
          type="text"
          required
          aria-required="true"
          autoComplete="name"
        />
      </div>

      <div>
        <label htmlFor="email">
          Email <span aria-hidden="true">*</span>
        </label>
        <input
          id="email"
          name="email"
          type="email"
          required
          aria-required="true"
          autoComplete="email"
        />
      </div>

      <div>
        <label htmlFor="phone">Phone</label>
        <input
          id="phone"
          name="phone"
          type="tel"
          autoComplete="tel"
        />
      </div>

      <button type="submit">Send</button>

      {state.message && (
        <p role="status" aria-live="polite">{state.message}</p>
      )}
    </form>
  );
}
```

## Structured Data (JSON-LD)

### Schema Component
```tsx
// components/JsonLd.tsx
export function JsonLd({ data }: { data: Record<string, unknown> }) {
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(data) }}
    />
  );
}

// Usage in page
import { JsonLd } from '@/components/JsonLd';

export default function BusinessPage() {
  const businessSchema = {
    '@context': 'https://schema.org',
    '@type': 'LocalBusiness',
    name: 'Business Name',
    address: {
      '@type': 'PostalAddress',
      streetAddress: '123 Main St',
      addressLocality: 'City',
      postalCode: '12345',
    },
  };

  return (
    <>
      <JsonLd data={businessSchema} />
      <main>...</main>
    </>
  );
}
```

## Bundle Analysis

### Analyze Bundle Size
```bash
# Install analyzer
npm install @next/bundle-analyzer

# next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer(nextConfig);

# Run analysis
ANALYZE=true npm run build
```

## Common Optimizations Checklist

- [ ] Use `next/image` for all images
- [ ] Use `next/font` for fonts (self-hosted, optimized)
- [ ] Use `next/script` with appropriate strategy
- [ ] Add `priority` to LCP image
- [ ] Set metadata in layout and pages
- [ ] Configure security headers in next.config.js
- [ ] Use dynamic imports for heavy components
- [ ] Add proper form accessibility attributes
- [ ] Include structured data (JSON-LD)
- [ ] Run bundle analyzer to check sizes
