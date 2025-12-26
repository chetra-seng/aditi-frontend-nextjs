# React: Manual Optimization Required

In React, you must handle optimization yourself:

```tsx
// React - Manual image handling
<img
  src="/hero.jpg"
  alt="Hero"
  loading="lazy"  // You add this manually
/>

// No automatic resizing
// No format conversion (WebP, AVIF)
// No responsive srcset
// No blur placeholder
// CLS (layout shift) issues
```

**You need:**
- Manual lazy loading attributes
- Third-party libraries for optimization
- Build-time image processing tools
- Manual meta tag management

---

# React vs Next.js Image Optimization

<div class="grid grid-cols-2 gap-4">

<div>

## React (Manual)

```tsx
<img
  src="/large-image.jpg"
  alt="Product"
  width="800"
  height="600"
  loading="lazy"
/>
```

- Original image served (2MB)
- No format optimization
- Manual lazy loading
- Layout shift issues

</div>

<div>

## Next.js (Automatic)

```tsx
import Image from 'next/image';

<Image
  src="/large-image.jpg"
  alt="Product"
  width={800}
  height={600}
/>
```

- Auto-optimized (50KB WebP)
- Responsive srcset
- Built-in lazy loading
- No layout shift

</div>

</div>

---

# Image Optimization

```tsx
import Image from 'next/image';

export default function Hero() {
  return (
    <Image
      src="/hero.jpg"
      alt="Hero image"
      width={1200}
      height={600}
      priority // Load immediately (above fold)
      placeholder="blur"
      blurDataURL="data:image/jpeg;base64,..."
    />
  );
}
```

<v-clicks>

✅ **Automatic optimization** (WebP, AVIF)
✅ **Responsive images** (srcset)
✅ **Lazy loading** by default
✅ **Blur placeholder**
✅ **No layout shift**

</v-clicks>

---

# Image Formats & Sizes

```tsx
<Image
  src="/product.jpg"
  alt="Product"
  width={800}
  height={600}
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
  quality={85}
  format="webp"
/>
```

Next.js automatically:
- Generates multiple sizes
- Serves WebP/AVIF to supported browsers
- Falls back to original format
- Optimizes on-demand (not at build time!)

---

# Font Optimization

```tsx
// app/layout.tsx
import { Inter, Roboto_Mono } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
});

const robotoMono = Roboto_Mono({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-roboto-mono',
});

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={`${inter.variable} ${robotoMono.variable}`}>
      <body className={inter.className}>{children}</body>
    </html>
  );
}
```

✅ **Automatic font optimization**
✅ **Zero layout shift**
✅ **Self-hosted** (no Google request)

---

# SEO: The React Problem

What search engines see when crawling a React SPA:

```html
<!-- Google bot sees this: -->
<!DOCTYPE html>
<html>
  <head>
    <title>My App</title>
  </head>
  <body>
    <div id="root"></div>
    <script src="/bundle.js"></script>
  </body>
</html>

<!-- No content! No meta tags! No structure! -->
```

<v-clicks>

**SEO Issues:**
- Empty HTML = nothing to index
- No meta descriptions for search results
- No Open Graph tags for social sharing
- Poor search rankings
- No rich snippets

</v-clicks>

---

# SEO: What Users Share on Social Media

<div class="grid grid-cols-2 gap-4">

<div>

## React SPA (No SSR)

When shared on Twitter/LinkedIn:

```
my-react-app.com
─────────────────
[No preview image]
[No description]
Just a boring URL
```

</div>

<div>

## Next.js (SSR/SSG)

When shared on Twitter/LinkedIn:

```
my-nextjs-app.com
─────────────────
[Beautiful preview image]
"Check out our amazing product
 that solves your problems..."
```

</div>

</div>

**Rich previews = more clicks = more traffic**

---

# How Next.js Solves SEO

```html
<!-- What Google bot sees with Next.js: -->
<!DOCTYPE html>
<html>
  <head>
    <title>Best Running Shoes 2024 | SportShop</title>
    <meta name="description" content="Find the perfect running shoes...">
    <meta property="og:title" content="Best Running Shoes 2024">
    <meta property="og:image" content="/images/shoes-preview.jpg">
    <meta property="og:description" content="Find the perfect...">
    <link rel="canonical" href="https://sportshop.com/shoes">
  </head>
  <body>
    <header>...</header>
    <main>
      <h1>Best Running Shoes 2024</h1>
      <article>Full content here...</article>
    </main>
    <footer>...</footer>
  </body>
</html>
```

**Full content, meta tags, structured HTML - ready to index!**

---

# SEO Benefits of Next.js

| Feature | React SPA | Next.js |
|---------|-----------|---------|
| Initial HTML | Empty div | Full content |
| Meta tags | Client-side only | Server-rendered |
| Crawlability | Poor | Excellent |
| Social previews | None | Rich cards |
| Core Web Vitals | Often poor | Optimized |
| Time to content | Slow (JS parse) | Fast (HTML ready) |

---

# Metadata for SEO

```tsx
// app/blog/[slug]/page.tsx
import type { Metadata } from 'next';

export async function generateMetadata({
  params
}: {
  params: Promise<{ slug: string }>
}): Promise<Metadata> {
  const { slug } = await params;
  const post = await fetchPost(slug);

  return {
    title: post.title,
    description: post.excerpt,
    keywords: post.tags,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [post.image],
      type: 'article',
      publishedTime: post.publishedAt,
      authors: [post.author.name],
    },
    twitter: {
      card: 'summary_large_image',
      title: post.title,
      description: post.excerpt,
      images: [post.image],
    },
  };
}
```

---

# Dynamic Metadata

```tsx
// app/products/[id]/page.tsx
export async function generateMetadata({
  params
}: {
  params: Promise<{ id: string }>
}): Promise<Metadata> {
  const { id } = await params;
  const product = await db.product.findUnique({
    where: { id },
  });

  return {
    title: `${product.name} - Shop`,
    description: product.description,
    openGraph: {
      images: [
        {
          url: product.image,
          width: 1200,
          height: 630,
          alt: product.name,
        },
      ],
    },
  };
}
```

---

# Structured Data (JSON-LD)

```tsx
// app/products/[id]/page.tsx
export default async function ProductPage({ params }) {
  const product = await getProduct(params.id);

  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: product.name,
    description: product.description,
    image: product.image,
    offers: {
      '@type': 'Offer',
      price: product.price,
      priceCurrency: 'USD',
    },
  };

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      <ProductDetails product={product} />
    </>
  );
}
```

**Enables rich snippets in Google search results!**

---

# Sitemap Generation

```tsx
// app/sitemap.ts
import { MetadataRoute } from 'next';

export default async function sitemap(): MetadataRoute.Sitemap {
  const products = await db.product.findMany();

  const productUrls = products.map((product) => ({
    url: `https://myshop.com/products/${product.id}`,
    lastModified: product.updatedAt,
    changeFrequency: 'weekly' as const,
    priority: 0.8,
  }));

  return [
    {
      url: 'https://myshop.com',
      lastModified: new Date(),
      changeFrequency: 'daily',
      priority: 1,
    },
    ...productUrls,
  ];
}
```

Auto-generates `/sitemap.xml` for search engines!

---

# Robots.txt

```tsx
// app/robots.ts
import { MetadataRoute } from 'next';

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      {
        userAgent: '*',
        allow: '/',
        disallow: ['/admin/', '/api/', '/private/'],
      },
    ],
    sitemap: 'https://myshop.com/sitemap.xml',
  };
}
```

<v-clicks>

**Controls what search engines can crawl:**
- Allow public pages
- Block admin/API routes
- Point to sitemap location

</v-clicks>

---

# Script Optimization

```tsx
import Script from 'next/script';

export default function Page() {
  return (
    <>
      {/* Load after page is interactive */}
      <Script src="https://example.com/script.js" strategy="afterInteractive" />

      {/* Load before page is interactive (blocking) */}
      <Script src="https://example.com/critical.js" strategy="beforeInteractive" />

      {/* Load lazily when browser is idle */}
      <Script src="https://example.com/analytics.js" strategy="lazyOnload" />

      {/* Inline script */}
      <Script id="inline-script">
        {`console.log('Hello from inline script')`}
      </Script>
    </>
  );
}
```

---

# Code Splitting

Next.js automatically code-splits by:

```
app/
├── page.tsx           → Separate bundle
├── about/
│   └── page.tsx      → Separate bundle
└── dashboard/
    └── page.tsx      → Separate bundle
```

**Manual code splitting with dynamic imports:**

```tsx
import dynamic from 'next/dynamic';

const HeavyComponent = dynamic(() => import('@/components/HeavyComponent'), {
  loading: () => <div>Loading...</div>,
  ssr: false, // Don't render on server
});

export default function Page() {
  return (
    <div>
      <h1>My Page</h1>
      <HeavyComponent />
    </div>
  );
}
```

---

# Bundle Analyzer

```bash
npm install @next/bundle-analyzer
```

```js
// next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // your next config
});
```

```bash
# Analyze your bundle (with webpack)
ANALYZE=true npm run build --webpack

# Turbopack is now default in Next.js 16
# Use experimental Turbopack analyzer in next.config.js
```

Opens interactive treemap showing bundle composition!

**Note:** Turbopack is now the default bundler in Next.js 16 (2-5x faster builds).

---

# Caching Strategies

```tsx
// No caching by default (Next.js 16+)
const data = await fetch('https://api.example.com/data');

// Revalidate every hour (time-based caching)
const data = await fetch('https://api.example.com/data', {
  next: { revalidate: 3600 }
});

// Force caching (explicit opt-in)
const data = await fetch('https://api.example.com/static', {
  cache: 'force-cache'
});

// Revalidate on-demand with tags
const data = await fetch('https://api.example.com/products', {
  next: { tags: ['products'] }
});

// Revalidate or update cache
import { revalidateTag, updateTag } from 'next/cache';
revalidateTag('products');  // Background refresh
updateTag('products');      // Immediate refresh (Next.js 16+)
```

---

# Streaming SSR

```tsx
import { Suspense } from 'react';

async function SlowData() {
  const data = await fetchSlowData();
  return <div>{data}</div>;
}

async function FastData() {
  const data = await fetchFastData();
  return <div>{data}</div>;
}

export default function Page() {
  return (
    <div>
      {/* Shows immediately */}
      <Suspense fallback={<div>Loading fast data...</div>}>
        <FastData />
      </Suspense>

      {/* Streams in when ready */}
      <Suspense fallback={<div>Loading slow data...</div>}>
        <SlowData />
      </Suspense>
    </div>
  );
}
```

**TTFB is fast, content streams progressively!**

---

# React Server Components Benefits

```tsx
// Server Component
import { db } from '@/lib/db';

export default async function Posts() {
  const posts = await db.post.findMany();

  return (
    <div>
      {posts.map(post => (
        <PostCard key={post.id} post={post} />
      ))}
    </div>
  );
}
```

<v-clicks>

**Bundle size benefits:**
- Database libraries NOT sent to client
- Only HTML sent to browser
- Smaller JavaScript bundles
- Faster page loads

</v-clicks>
