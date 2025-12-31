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

- ✅ **Automatic optimization** (WebP, AVIF)
- ✅ **Responsive images** (srcset)
- ✅ **Lazy loading** by default
- ✅ **Blur placeholder**
- ✅ **No layout shift**

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
import { Inter } from 'next/font/google';

const inter = Inter({ subsets: ['latin'], display: 'swap' });

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body className={inter.className}>{children}</body>
    </html>
  );
}
```

<v-clicks>

- **Self-hosted** - no external Google request
- **Zero layout shift** - font loads with page
- **Automatic subsetting** - only characters you need

</v-clicks>

---

# SEO: The React Problem

What Google bot sees when crawling a React SPA:

```html
<body>
  <div id="root"></div>  <!-- Empty! -->
  <script src="/bundle.js"></script>
</body>
```

<v-clicks>

**SEO Issues:**
- Empty HTML = nothing to index
- No meta tags for search/social
- Poor search rankings

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

<div class="bg-blue-500/20 px-4 py-2 text-white rounded mt-4">💡 Rich previews = more clicks = more traffic</div>

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
export async function generateMetadata({ params }): Promise<Metadata> {
  const { slug } = await params;
  const post = await fetchPost(slug);

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      images: [post.image],
    },
  };
}
```

<v-clicks>

- Dynamic metadata based on content
- OpenGraph for social previews
- Also supports `twitter`, `robots`, `icons`, etc.

</v-clicks>

---

# Dynamic Metadata

```tsx
// app/products/[id]/page.tsx
export async function generateMetadata({ params }): Promise<Metadata> {
  const { id } = await params;
  const product = await db.product.findUnique({ where: { id } });

  return {
    title: `${product.name} - Shop`,
    description: product.description,
    openGraph: { images: [{ url: product.image, width: 1200, height: 630 }] },
  };
}
```

---

# Structured Data (JSON-LD)

```tsx
// app/products/[id]/page.tsx
const jsonLd = {
  '@context': 'https://schema.org',
  '@type': 'Product',
  name: product.name,
  image: product.image,
  offers: { '@type': 'Offer', price: product.price, priceCurrency: 'USD' },
};

return (
  <>
    <script type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }} />
    <ProductDetails product={product} />
  </>
);
```

**Enables rich snippets in Google search results!**

---

# Sitemap Generation

```tsx
// app/sitemap.ts
export default async function sitemap(): MetadataRoute.Sitemap {
  const products = await db.product.findMany();

  return [
    { url: 'https://myshop.com', changeFrequency: 'daily', priority: 1 },
    ...products.map((p) => ({
      url: `https://myshop.com/products/${p.id}`,
      lastModified: p.updatedAt,
    })),
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

# Code Splitting (Automatic)

Next.js automatically code-splits by route:

```
app/
├── page.tsx           → Separate bundle
├── about/
│   └── page.tsx      → Separate bundle
└── dashboard/
    └── page.tsx      → Separate bundle
```

<v-clicks>

- Each page only loads its own JavaScript
- Users don't download code they don't need
- Faster initial page loads

</v-clicks>

---

# Code Splitting (Dynamic Imports)

```tsx
import dynamic from 'next/dynamic';

const HeavyComponent = dynamic(() => import('@/components/HeavyComponent'), {
  loading: () => <div>Loading...</div>,
  ssr: false, // Don't render on server
});

export default function Page() {
  return <HeavyComponent />;
}
```

<v-clicks>

- Load components only when needed
- `ssr: false` for client-only components (maps, charts)

</v-clicks>

---

# Next.js Caching Layers

<div class="grid grid-cols-2 gap-4 mt-6">
  <v-click>
    <div class="px-4 py-3 bg-blue-500 text-white rounded-lg shadow-lg">
      <div class="font-bold">Server Components</div>
      <div class="text-xs opacity-80">Cached at build time (SSG) or revalidated (ISR)</div>
    </div>
  </v-click>
  <v-click>
    <div class="px-4 py-3 bg-green-500 text-white rounded-lg shadow-lg">
      <div class="font-bold">fetch()</div>
      <div class="text-xs opacity-80">`next: { revalidate }` or `cache: 'force-cache'`</div>
    </div>
  </v-click>
  <v-click>
    <div class="px-4 py-3 bg-yellow-500 text-black rounded-lg shadow-lg">
      <div class="font-bold">API Routes</div>
      <div class="text-xs opacity-80">Set `Cache-Control` headers in Response</div>
    </div>
  </v-click>
  <v-click>
    <div class="px-4 py-3 bg-purple-500 text-white rounded-lg shadow-lg">
      <div class="font-bold">Router Cache</div>
      <div class="text-xs opacity-80">Client-side navigation cache (automatic)</div>
    </div>
  </v-click>
</div>

<v-click>
<div class="mt-8 text-center">
  <span class="px-4 py-2 bg-gray-800 text-green-400 rounded-full font-bold">Cache at every layer for maximum performance!</span>
</div>
</v-click>

---

# Caching Strategies

| Strategy | Use Case |
|----------|----------|
| `cache: 'force-cache'` | Static data (fastest) |
| `next: { revalidate: N }` | Update every N seconds |
| `next: { tags: [...] }` | On-demand invalidation |

---

# Caching Example

```tsx
// Cache product catalog, revalidate hourly
const products = await fetch('https://api.example.com/products', {
  next: { revalidate: 3600, tags: ['products'] }
});
```

```tsx
// Invalidate when admin updates products
import { revalidateTag } from 'next/cache';
revalidateTag('products');
```

<v-clicks>

- `revalidate: 3600` - refresh data every hour
- `tags: ['products']` - invalidate on demand

</v-clicks>

---

# Streaming SSR

```tsx
import { Suspense } from 'react';

export default function Page() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <SlowComponent /> {/* Streams in when ready */}
      </Suspense>
    </div>
  );
}
```

<v-clicks>

- Page shell renders immediately
- Slow components stream in when ready
- User sees content faster (better TTFB)

</v-clicks>

---

# React Server Components Benefits

```tsx
// Server Component - db library NOT sent to browser!
import { db } from '@/lib/db';

export default async function Posts() {
  const posts = await db.post.findMany();
  return posts.map(post => <PostCard key={post.id} post={post} />);
}
```

<v-clicks>

- Database/API libraries stay on server
- Only HTML sent to browser
- Smaller bundles, faster loads

</v-clicks>

<v-click>
<div class="bg-yellow-500/20 px-4 py-2 text-yellow-200 rounded mt-2">⚠️ Can read cookies, but cannot set them (use Server Actions)</div>
</v-click>
