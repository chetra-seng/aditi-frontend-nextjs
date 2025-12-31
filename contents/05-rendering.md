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

// Generate all paths at build time
export async function generateStaticParams() {
  const posts = await fetchAllPosts();
  return posts.map(post => ({ slug: post.slug }));
}

// Pre-rendered to HTML at build time
export default async function BlogPost({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;
  const post = await fetchPost(slug);
  return <article><h1>{post.title}</h1><div>{post.content}</div></article>;
}
```

<v-clicks>

- ✅ **Fastest** - serves pre-built HTML files
- ✅ **Best for:** blogs, docs, marketing pages
- ❌ **Avoid when:** data changes frequently, user-specific content

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

- ✅ **Always fresh** data
- ✅ **Personalized** content
- ✅ **SEO-friendly**
- ❌ Slower than SSG (renders on each request)

</v-clicks>

---

# Incremental Static Regeneration (ISR)

<div class="grid grid-cols-2 gap-4">

<div>

## Option 1: Page-level

```tsx
// Entire page revalidates every 60s
export const revalidate = 60;

export default async function Page() {
  const data = await fetchData();
  return <div>{data}</div>;
}
```

Applies to **all data** on the page

</div>

<div>

## Option 2: Fetch-level

```tsx
export default async function Page() {
  const data = await fetch(url, {
    next: { revalidate: 60 }
  });
  return <div>{data}</div>;
}
```

**Per-request** control

</div>

</div>

<v-clicks>

- ✅ **Page-level** - simple, one setting for everything
- ✅ **Fetch-level** - granular (e.g., products: 60s, categories: 3600s)
- ✅ **Static speed** - serves cached HTML instantly
- ✅ **Fresh data** - regenerates in background without blocking users
- ✅ **Best for:** e-commerce, news, content that updates periodically

</v-clicks>

---

# How ISR Works

<div class="flex items-center justify-center gap-3 mt-8">
  <v-click>
    <div class="px-4 py-3 bg-blue-500 text-white rounded-lg shadow-lg">
      <div class="font-bold">1. User Request</div>
      <div class="text-xs opacity-80">Browser requests page</div>
    </div>
  </v-click>
  <v-click><div class="text-2xl">→</div></v-click>
  <v-click>
    <div class="px-4 py-3 bg-green-500 text-white rounded-lg shadow-lg">
      <div class="font-bold">2. Serve Cache</div>
      <div class="text-xs opacity-80">Instant! No waiting</div>
    </div>
  </v-click>
  <v-click><div class="text-2xl">→</div></v-click>
  <v-click>
    <div class="px-4 py-3 bg-yellow-500 text-black rounded-lg shadow-lg">
      <div class="font-bold">3. Check Age</div>
      <div class="text-xs opacity-80">Is cache older than revalidate?</div>
    </div>
  </v-click>
</div>

<div class="flex justify-center mt-6">
  <v-click><div class="text-xl font-bold text-yellow-400">↓ YES, cache is stale</div></v-click>
</div>

<div class="flex items-center justify-center gap-3 mt-6">
  <v-click>
    <div class="px-4 py-3 bg-purple-500 text-white rounded-lg shadow-lg">
      <div class="font-bold">4. Background Regeneration</div>
      <div class="text-xs opacity-80">Server rebuilds page async</div>
    </div>
  </v-click>
  <v-click><div class="text-2xl">→</div></v-click>
  <v-click>
    <div class="px-4 py-3 bg-green-600 text-white rounded-lg shadow-lg">
      <div class="font-bold">5. Cache Updated</div>
      <div class="text-xs opacity-80">Next visitor gets fresh page</div>
    </div>
  </v-click>
</div>

<v-click>
<div class="mt-8 text-center">
  <span class="px-4 py-2 bg-gray-800 text-green-400 rounded-full font-bold">✨ User is NEVER blocked - always instant response!</span>
</div>
</v-click>

---

# CSR with React Query

```tsx
'use client';

import { useQuery } from '@tanstack/react-query';

export default function Dashboard() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['user-stats'],
    queryFn: () => fetch('/api/stats').then(res => res.json()),
    refetchInterval: 30000, // Auto-refresh every 30s
  });

  if (isLoading) return <Skeleton />;
  if (error) return <Error message={error.message} />;

  return <StatsDisplay data={data} />;
}
```

<v-clicks>

- ✅ **Caching** - no duplicate requests
- ✅ **Auto-refresh** - keeps data fresh
- ✅ **Loading/error states** - built-in handling

</v-clicks>

---

# Why Client-Side Rendering (CSR)?

<v-clicks>

- ✅ **Real-time updates** - live data, notifications, chat
- ✅ **Browser APIs** - geolocation, camera, localStorage
- ✅ **User interactions** - forms, filters, infinite scroll
- ✅ **Personalized content** - dashboards after login
- ❌ **SEO not needed** - admin panels, authenticated pages

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

# Hybrid Rendering: ISR + CSR

```tsx
// app/product/[id]/page.tsx

export const revalidate = 3600; // Revalidate product data every hour

export default async function ProductPage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const product = await fetchProduct(id); // Cached + revalidated

  return (
    <div>
      {/* ISR: Product info refreshes every hour */}
      <h1>{product.name}</h1>
      <p>${product.price}</p>

      {/* CSR: Real-time stock updates */}
      <StockStatus productId={id} />

      {/* CSR: Live reviews with React Query */}
      <ClientReviews productId={id} />
    </div>
  );
}
```

**ISR for product data + CSR for real-time features!**

---

# CSR: StockStatus Component

```tsx
// components/StockStatus.tsx
'use client';

import { useQuery } from '@tanstack/react-query';

export function StockStatus({ productId }: { productId: string }) {
  const { data, isLoading } = useQuery({
    queryKey: ['stock', productId],
    queryFn: () => fetch(`/api/stock/${productId}`).then(r => r.json()),
    refetchInterval: 5000, // Poll every 5 seconds
  });

  if (isLoading) return <span>Checking stock...</span>;

  return <span>{data?.inStock ? '✅ In Stock' : '❌ Out of Stock'}</span>;
}
```

<v-clicks>

- ✅ **Real-time polling** - updates every 5 seconds
- ✅ **Never stale** - always shows current stock
- ✅ **User sees live changes** - no page refresh needed

</v-clicks>

---

# CSR: ClientReviews Component

```tsx
// components/ClientReviews.tsx
'use client';

import { useQuery } from '@tanstack/react-query';

export function ClientReviews({ productId }: { productId: string }) {
  const { data: reviews, isLoading } = useQuery({
    queryKey: ['reviews', productId],
    queryFn: () => fetch(`/api/reviews/${productId}`).then(r => r.json()),
  });

  if (isLoading) return <ReviewsSkeleton />;

  return (
    <ul>
      {reviews?.map(r => <li key={r.id}>⭐ {r.text}</li>)}
    </ul>
  );
}
```

<v-clicks>

- ✅ **Loaded after page renders** - doesn't block ISR
- ✅ **Cached by React Query** - instant on revisit
- ✅ **Can add mutations** - submit new reviews without page reload

</v-clicks>

---

# Choosing the Right Strategy

| Strategy | When to Use | Example |
|----------|-------------|---------|
| **SSG** | Content rarely changes | Blog, docs, marketing |
| **SSR** | Always need fresh data | User dashboard, admin |
| **ISR** | Content updates periodically | E-commerce, news |
| **CSR** | Browser-only features | Maps, real-time charts |
