# React + Vite Patterns

## Vite Configuration

### vite.config.ts with Full Optimization
```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // Core React
          'vendor-react': ['react', 'react-dom'],
          // Router (if used)
          'vendor-router': ['react-router-dom'],
          // UI library (if used)
          'vendor-ui': ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu'],
          // Charts (if used)
          'vendor-charts': ['recharts', 'chart.js'],
        },
      },
    },
    cssCodeSplit: true,
    minify: 'esbuild',
    esbuild: {
      drop: ['console', 'debugger'],
    },
    // Generate source maps for debugging (optional)
    sourcemap: false,
  },
});
```

## Font Loading

### index.html Pattern
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />

  <!-- DNS Prefetch -->
  <link rel="dns-prefetch" href="https://fonts.googleapis.com">
  <link rel="dns-prefetch" href="https://fonts.gstatic.com">

  <!-- Preconnect -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

  <!-- Non-blocking fonts -->
  <link
    rel="preload"
    as="style"
    href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap"
  >
  <link
    rel="stylesheet"
    href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap"
    media="print"
    onload="this.media='all'"
  >

  <title>App</title>
</head>
<body>
  <div id="root"></div>
  <script type="module" src="/src/main.tsx"></script>
</body>
</html>
```

## Lazy Loading Components

### Route-based Code Splitting
```tsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

// Lazy load pages
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Contact = lazy(() => import('./pages/Contact'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
      </Routes>
    </Suspense>
  );
}
```

### Component-level Lazy Loading
```tsx
import { lazy, Suspense } from 'react';

// Heavy component (charts, maps, etc.)
const HeavyChart = lazy(() => import('./components/HeavyChart'));

function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<div>Loading chart...</div>}>
        <HeavyChart />
      </Suspense>
    </div>
  );
}
```

## Image Optimization

### Optimized Image Component
```tsx
interface OptimizedImageProps {
  src: string;
  alt: string;
  width: number;
  height: number;
  isLCP?: boolean;
  className?: string;
}

export function OptimizedImage({
  src,
  alt,
  width,
  height,
  isLCP = false,
  className = '',
}: OptimizedImageProps) {
  return (
    <img
      src={src}
      alt={alt}
      width={width}
      height={height}
      loading={isLCP ? 'eager' : 'lazy'}
      fetchPriority={isLCP ? 'high' : 'auto'}
      decoding="async"
      className={className}
    />
  );
}
```

### Usage
```tsx
// Hero image (LCP element)
<OptimizedImage
  src="/hero.webp"
  alt="Hero description"
  width={1200}
  height={600}
  isLCP
/>

// Regular image
<OptimizedImage
  src="/team/photo.webp"
  alt="Team member"
  width={400}
  height={500}
/>
```

## Accessible Form Components

### Form Input with Accessibility
```tsx
interface FormInputProps {
  id: string;
  label: string;
  type?: string;
  required?: boolean;
  autoComplete?: string;
  error?: string;
  value: string;
  onChange: (value: string) => void;
}

export function FormInput({
  id,
  label,
  type = 'text',
  required = false,
  autoComplete,
  error,
  value,
  onChange,
}: FormInputProps) {
  return (
    <div>
      <label htmlFor={id} className="block text-sm font-medium">
        {label}
        {required && <span aria-hidden="true" className="text-red-500"> *</span>}
      </label>
      <input
        id={id}
        name={id}
        type={type}
        required={required}
        aria-required={required}
        autoComplete={autoComplete}
        aria-invalid={!!error}
        aria-describedby={error ? `${id}-error` : undefined}
        value={value}
        onChange={(e) => onChange(e.target.value)}
        className="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
      />
      {error && (
        <p id={`${id}-error`} className="mt-1 text-sm text-red-600" role="alert">
          {error}
        </p>
      )}
    </div>
  );
}
```

### Icon Button with Accessibility
```tsx
interface IconButtonProps {
  label: string;
  onClick: () => void;
  children: React.ReactNode;
  className?: string;
}

export function IconButton({ label, onClick, children, className = '' }: IconButtonProps) {
  return (
    <button
      type="button"
      onClick={onClick}
      aria-label={label}
      className={`p-2 rounded-lg hover:bg-gray-100 transition ${className}`}
    >
      {children}
    </button>
  );
}

// Usage
<IconButton label="Close menu" onClick={() => setOpen(false)}>
  <XIcon aria-hidden="true" className="w-5 h-5" />
</IconButton>
```

## Conditional Script Loading

### Load Third-party Scripts Only When Needed
```tsx
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';

function useConditionalScript(src: string, condition: boolean) {
  useEffect(() => {
    if (!condition) return;

    const script = document.createElement('script');
    script.src = src;
    script.async = true;
    document.head.appendChild(script);

    return () => {
      document.head.removeChild(script);
    };
  }, [src, condition]);
}

// Usage in layout
function Layout({ children }: { children: React.ReactNode }) {
  const location = useLocation();
  const isContactPage = location.pathname === '/contact';

  // Only load Turnstile on contact page
  useConditionalScript(
    'https://challenges.cloudflare.com/turnstile/v0/api.js',
    isContactPage
  );

  return <main>{children}</main>;
}
```

## CSS with Focus States

### Tailwind CSS Layer
```css
/* src/index.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  *:focus-visible {
    @apply outline-2 outline-offset-2 outline-primary-500 rounded-sm;
  }

  button:focus-visible,
  a:focus-visible {
    @apply ring-2 ring-primary-500 ring-offset-2 outline-none;
  }

  input:focus-visible,
  textarea:focus-visible,
  select:focus-visible {
    @apply ring-2 ring-primary-400 ring-offset-1 outline-none;
  }
}
```

## Performance Hooks

### useIntersectionObserver for Lazy Loading
```tsx
import { useEffect, useRef, useState } from 'react';

export function useIntersectionObserver(options?: IntersectionObserverInit) {
  const ref = useRef<HTMLElement>(null);
  const [isVisible, setIsVisible] = useState(false);

  useEffect(() => {
    const element = ref.current;
    if (!element) return;

    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) {
        setIsVisible(true);
        observer.disconnect();
      }
    }, options);

    observer.observe(element);

    return () => observer.disconnect();
  }, [options]);

  return { ref, isVisible };
}

// Usage
function HeavyComponent() {
  const { ref, isVisible } = useIntersectionObserver({ rootMargin: '100px' });

  return (
    <div ref={ref}>
      {isVisible ? <ActualHeavyContent /> : <Placeholder />}
    </div>
  );
}
```

## Environment Configuration

### .env for Production
```env
VITE_APP_URL=https://example.com
VITE_API_URL=https://api.example.com
```

### vite.config.ts for HTTPS Assets
```typescript
export default defineConfig({
  base: process.env.NODE_ENV === 'production' ? 'https://example.com/' : '/',
});
```
