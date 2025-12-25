# React Rendering: Client-Side Only

In traditional React, there's only ONE rendering strategy:

```tsx
// index.html - What the server sends
<!DOCTYPE html>
<html>
  <body>
    <div id="root"></div>  <!-- Empty! -->
    <script src="/bundle.js"></script>
  </body>
</html>

// App.tsx - Renders in the browser
function App() {
  return <h1>Hello World</h1>;
}

ReactDOM.render(<App />, document.getElementById('root'));
```

**Everything happens in the browser!**

---

# React CSR: The Problem

```
Timeline of a React App Load:

1. Browser requests page     ─────┐
2. Server sends empty HTML        │  User sees
3. Browser downloads JS bundle    │  BLANK SCREEN
4. Browser parses/executes JS     │
5. React hydrates                 │
6. useEffect runs                 │
7. API calls made                 │
8. Data received                 ─┘
9. Content finally appears!
```

<v-clicks>

**Issues:**
- Long Time to First Contentful Paint (FCP)
- Search engines see empty page
- Poor Core Web Vitals
- Bad experience on slow connections

</v-clicks>

---

# Why Multiple Rendering Strategies?

| Content Type | Best Strategy |
|--------------|---------------|
| Blog post (rarely changes) | Pre-build the HTML once |
| User dashboard (personalized) | Render on each request |
| Product page (changes daily) | Rebuild periodically |
| Live chat widget | Render in browser |

**One size does NOT fit all!**

---

# How Next.js Solves This

<div class="grid grid-cols-2 gap-4">

<div>

## React (CSR Only)

```tsx
// Only option: client-side
function Page() {
  const [data, setData] = useState(null);
  useEffect(() => {
    fetchData().then(setData);
  }, []);
  return <div>{data}</div>;
}
```

- Always renders in browser
- Always fetches on client
- No SEO
- Same approach for everything

</div>

<div>

## Next.js (Choose Per Page)

```tsx
// SSG - Pre-built at deploy
export const dynamic = 'force-static';

// SSR - Fresh on every request
export const dynamic = 'force-dynamic';

// ISR - Rebuild every hour
export const revalidate = 3600;

// CSR - When you need it
'use client';
```

Right tool for each job!

</div>

</div>

---

# Rendering Strategies Overview

<div class="grid grid-cols-2 gap-4">

<div>

## Server-Side
**SSR** - Server-Side Rendering
**SSG** - Static Site Generation
**ISR** - Incremental Static Regeneration

</div>

<div>

## Client-Side
**CSR** - Client-Side Rendering
**Hybrid** - Mix and match!

</div>

</div>

Next.js lets you choose **per page**!

---

# Static Site Generation (SSG)

```tsx
// app/blog/[slug]/page.tsx

// Generate static paths at build time
export async function generateStaticParams() {
  const posts = await fetchAllPosts();

  return posts.map(post => ({
    slug: post.slug,
  }));
}

// This page is pre-rendered at build time
export default async function BlogPost({
  params
}: {
  params: { slug: string }
}) {
  const post = await fetchPost(params.slug);

  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  );
}
```

✅ **Fastest** - pre-built HTML files
✅ **Best for** blogs, docs, marketing pages

---

# When SSG Runs

```bash
# Build time
npm run build

# Next.js generates:
.next/
  server/
    pages/
      blog/
        post-1.html  # ✅ Pre-rendered!
        post-2.html  # ✅ Pre-rendered!
        post-3.html  # ✅ Pre-rendered!
```

<v-clicks>

**Perfect for:**
- Blog posts
- Documentation
- Marketing pages
- Product catalogs (updated infrequently)

**Avoid when:**
- Data changes frequently
- User-specific content
- Millions of pages

</v-clicks>

---

# Server-Side Rendering (SSR)

```tsx
// app/dashboard/page.tsx

// Force dynamic rendering
export const dynamic = 'force-dynamic';

export default async function Dashboard() {
  // This runs on EVERY request
  const user = await getCurrentUser();
  const stats = await fetchUserStats(user.id);

  return (
    <div>
      <h1>Welcome, {user.name}!</h1>
      <StatsDisplay stats={stats} />
    </div>
  );
}
```

<v-clicks>

✅ **Always fresh** data
✅ **Personalized** content
✅ **SEO-friendly**
❌ Slower than SSG (renders on each request)

</v-clicks>

---

# Incremental Static Regeneration (ISR)

```tsx
// app/products/[id]/page.tsx

export default async function ProductPage({
  params
}: {
  params: { id: string }
}) {
  // Fetch data with revalidation
  const product = await fetch(
    `https://api.example.com/products/${params.id}`,
    { next: { revalidate: 60 } } // Revalidate every 60 seconds
  );

  return (
    <div>
      <h1>{product.name}</h1>
      <p>${product.price}</p>
    </div>
  );
}
```

<v-clicks>

✅ **Static + Dynamic** - best of both worlds
✅ **Stale-while-revalidate** strategy
✅ Regenerates pages in the background

</v-clicks>

---

# How ISR Works

```
User Request → Static Page Served (instant!)
    ↓
Is page older than revalidate time?
    ↓
YES → Regenerate in background
    ↓
Next user gets fresh page
```

<v-clicks>

**Example with revalidate: 60**
- 0:00 - User A visits → Gets static page (built at deploy)
- 0:30 - User B visits → Gets same static page (still fresh)
- 1:10 - User C visits → Gets static page + triggers regeneration
- 1:15 - User D visits → Gets newly regenerated page!

</v-clicks>

---

# Client-Side Rendering (CSR)

```tsx
'use client';

import { useState, useEffect } from 'react';

export default function ClientOnlyComponent() {
  const [data, setData] = useState(null);

  useEffect(() => {
    // Only runs in the browser
    fetch('/api/data')
      .then(res => res.json())
      .then(setData);
  }, []);

  if (!data) return <div>Loading...</div>;

  return <div>{data.message}</div>;
}
```

<v-clicks>

Use when:
- Need browser APIs
- Highly interactive
- SEO not important
- User-specific dashboards

</v-clicks>

---

# Dynamic Rendering Configurations

```tsx
// Force static generation
export const dynamic = 'force-static';

// Force dynamic rendering (SSR)
export const dynamic = 'force-dynamic';

// Error if page can't be static
export const dynamic = 'error';

// Auto (default) - Next.js decides
export const dynamic = 'auto';
```

```tsx
// Control revalidation
export const revalidate = 3600; // 1 hour

// Never revalidate
export const revalidate = false;

// Revalidate on every request
export const revalidate = 0;
```

---

# Hybrid Rendering Example

```tsx
// app/product/[id]/page.tsx

// Page is static
export const dynamic = 'force-static';

export default async function ProductPage({ params }: { params: { id: string } }) {
  // Static product data
  const product = await fetchProduct(params.id);

  return (
    <div>
      {/* Static content */}
      <h1>{product.name}</h1>
      <p>{product.description}</p>

      {/* Client-side interactive reviews */}
      <ClientReviews productId={params.id} />

      {/* Client-side "Add to Cart" */}
      <AddToCartButton product={product} />
    </div>
  );
}
```

**Static shell + Dynamic interactivity!**

---

# Choosing the Right Strategy

| Strategy | When to Use | Example |
|----------|-------------|---------|
| **SSG** | Content rarely changes | Blog, docs, marketing |
| **SSR** | Always need fresh data | User dashboard, admin |
| **ISR** | Content updates periodically | E-commerce, news |
| **CSR** | Browser-only features | Maps, real-time charts |
