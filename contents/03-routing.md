# React Routing: The Manual Way

How routing works in traditional React apps:

```tsx
// React requires manual route configuration
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <Link to="/blog">Blog</Link>
      </nav>

      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/blog" element={<BlogList />} />
        <Route path="/blog/:slug" element={<BlogPost />} />
        <Route path="/dashboard/*" element={<Dashboard />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}
```

---

# React Router Pain Points

<div class="grid grid-cols-2 gap-4">

<div>

## Manual Configuration

```tsx
// Every route needs explicit setup
<Route path="/" element={<Home />} />
<Route path="/products" element={<Products />} />
<Route path="/products/:id" element={<Product />} />
<Route path="/products/:id/reviews" element={<Reviews />} />
// ... repeat for every page
```

</div>

<div>

## Common Problems

<v-clicks>

- Routes defined separately from components
- No co-location of route + page code
- Dynamic routes need manual params parsing
- Nested routes are complex to set up
- Loading states require manual Suspense
- Error boundaries need manual configuration
- No automatic 404 handling per route

</v-clicks>

</div>

</div>

---

# React Router: Dynamic Routes

```tsx
import { useParams, useSearchParams } from 'react-router-dom';

function BlogPost() {
  // Must extract params manually
  const { slug } = useParams();
  const [searchParams] = useSearchParams();
  const page = searchParams.get('page');

  const [post, setPost] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(`/api/posts/${slug}`)
      .then(res => res.json())
      .then(setPost)
      .finally(() => setLoading(false));
  }, [slug]);

  if (loading) return <Spinner />;
  return <article>{post.content}</article>;
}
```

**Every dynamic route needs this boilerplate!**

---

# React Router: No Built-in Loading/Error States

```tsx
// You must manually add Suspense and ErrorBoundary
import { Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

function App() {
  return (
    <BrowserRouter>
      <ErrorBoundary fallback={<Error />}>
        <Suspense fallback={<Loading />}>
          <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/dashboard" element={
              <Suspense fallback={<DashboardLoading />}>
                <Dashboard />
              </Suspense>
            } />
          </Routes>
        </Suspense>
      </ErrorBoundary>
    </BrowserRouter>
  );
}
```

**Manual setup for every route that needs loading/error states!**

---

# How Next.js Solves Routing

<div class="grid grid-cols-2 gap-4">

<div>

## React Router (Manual)

```tsx
// routes.tsx
<Route path="/" element={<Home />} />
<Route path="/about" element={<About />} />
<Route path="/blog/:slug" element={<Blog />} />
```

- Separate route configuration
- Manual params extraction
- Manual loading/error handling
- No automatic prefetching

</div>

<div>

## Next.js (File-Based)

```
app/
├── page.tsx          → /
├── about/
│   └── page.tsx      → /about
└── blog/
    └── [slug]/
        └── page.tsx  → /blog/:slug
```

- **Folder = Route**
- Automatic params injection
- Built-in loading.tsx/error.tsx
- Automatic link prefetching

</div>

</div>

---

# File-Based Routing

```
app/
├── page.tsx                    → /
├── about/
│   └── page.tsx               → /about
├── blog/
│   ├── page.tsx               → /blog
│   └── [slug]/
│       └── page.tsx           → /blog/:slug
├── dashboard/
│   ├── page.tsx               → /dashboard
│   ├── settings/
│   │   └── page.tsx           → /dashboard/settings
│   └── analytics/
│       └── page.tsx           → /dashboard/analytics
```

**Folder structure = URL structure**

---

# Dynamic Routes

```tsx
// app/blog/[slug]/page.tsx
export default function BlogPost({
  params
}: {
  params: { slug: string }
}) {
  return (
    <article>
      <h1>Blog Post: {params.slug}</h1>
    </article>
  );
}
```

<v-clicks>

- `[slug]` folder = dynamic segment
- Access via `params.slug`
- `/blog/hello-world` → `params.slug = "hello-world"`
- `/blog/nextjs-tutorial` → `params.slug = "nextjs-tutorial"`

</v-clicks>

---

# Multiple Dynamic Segments

```
app/
└── shop/
    └── [category]/
        └── [productId]/
            └── page.tsx
```

```tsx
// app/shop/[category]/[productId]/page.tsx
export default function ProductPage({
  params
}: {
  params: { category: string; productId: string }
}) {
  return (
    <div>
      <h1>Category: {params.category}</h1>
      <h2>Product ID: {params.productId}</h2>
    </div>
  );
}
```

`/shop/electronics/laptop-123` → `{ category: "electronics", productId: "laptop-123" }`

---

# Catch-All Routes

```tsx
// app/docs/[...slug]/page.tsx
export default function DocsPage({
  params
}: {
  params: { slug: string[] }
}) {
  return (
    <div>
      <h1>Docs Path: {params.slug.join('/')}</h1>
    </div>
  );
}
```

<v-clicks>

- `[...slug]` = catch-all segment
- Returns an **array** of path segments
- `/docs/getting-started` → `slug = ["getting-started"]`
- `/docs/api/reference/auth` → `slug = ["api", "reference", "auth"]`

</v-clicks>

---

# Optional Catch-All Routes

```tsx
// app/docs/[[...slug]]/page.tsx
export default function DocsPage({
  params
}: {
  params: { slug?: string[] }
}) {
  const path = params.slug ? params.slug.join('/') : 'home';

  return <h1>Docs: {path}</h1>;
}
```

<v-clicks>

- `[[...slug]]` = optional catch-all
- Matches parent route too
- `/docs` → `slug = undefined`
- `/docs/getting-started` → `slug = ["getting-started"]`

</v-clicks>

---

# Route Groups

```
app/
├── (marketing)/
│   ├── about/
│   │   └── page.tsx          → /about
│   └── contact/
│       └── page.tsx          → /contact
├── (shop)/
│   ├── products/
│   │   └── page.tsx          → /products
│   └── cart/
│       └── page.tsx          → /cart
└── page.tsx                  → /
```

<v-clicks>

- `(folderName)` = route group
- **Not included** in URL path
- Useful for organization
- Can have separate layouts

</v-clicks>

---

# Route Groups with Layouts

```tsx
// app/(marketing)/layout.tsx
export default function MarketingLayout({
  children
}: {
  children: React.ReactNode
}) {
  return (
    <div>
      <nav>Marketing Nav</nav>
      {children}
      <footer>Marketing Footer</footer>
    </div>
  );
}
```

```tsx
// app/(shop)/layout.tsx
export default function ShopLayout({ children }: { children: React.ReactNode }) {
  return (
    <div>
      <nav>Shop Nav with Cart</nav>
      {children}
    </div>
  );
}
```

---

# React Router vs Next.js Navigation

<div class="grid grid-cols-2 gap-4">

<div>

## React Router Link

```tsx
import { Link } from 'react-router-dom';

function Nav() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>
    </nav>
  );
}
```

- `to` prop for destination
- No automatic prefetching
- Must be inside `<BrowserRouter>`

</div>

<div>

## Next.js Link

```tsx
import Link from 'next/link';

function Nav() {
  return (
    <nav>
      <Link href="/">Home</Link>
      <Link href="/about">About</Link>
    </nav>
  );
}
```

- `href` prop (like native anchor)
- **Automatic prefetching**
- Works anywhere in the app

</div>

</div>

---

# Navigation with Link Component

```tsx
import Link from 'next/link';

export default function Navigation() {
  return (
    <nav>
      <Link href="/">Home</Link>
      <Link href="/about">About</Link>
      <Link href="/blog">Blog</Link>
      <Link href="/contact">Contact</Link>
    </nav>
  );
}
```

<v-clicks>

- **Client-side navigation** (no page reload)
- **Automatic prefetching** (visible links are prefetched)
- **Better performance** than `<a>` tags
- **Programmatic navigation** also available

</v-clicks>

---

# Dynamic Navigation

```tsx
import Link from 'next/link';

export default function BlogList({ posts }: { posts: Post[] }) {
  return (
    <div>
      {posts.map(post => (
        <Link
          key={post.id}
          href={`/blog/${post.slug}`}
        >
          {post.title}
        </Link>
      ))}
    </div>
  );
}
```

---

# Programmatic Navigation

```tsx
'use client';

import { useRouter } from 'next/navigation';

export default function LoginForm() {
  const router = useRouter();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    // Perform login
    const success = await login();

    if (success) {
      router.push('/dashboard'); // Navigate programmatically
    }
  };

  return <form onSubmit={handleSubmit}>...</form>;
}
```

---

# useRouter Methods

```tsx
'use client';

import { useRouter } from 'next/navigation';

export default function NavigationDemo() {
  const router = useRouter();

  return (
    <div>
      <button onClick={() => router.push('/about')}>
        Go to About
      </button>

      <button onClick={() => router.replace('/login')}>
        Replace with Login (no history)
      </button>

      <button onClick={() => router.back()}>
        Go Back
      </button>

      <button onClick={() => router.refresh()}>
        Refresh Server Data
      </button>
    </div>
  );
}
```

---

# Link Component Props

```tsx
import Link from 'next/link';

export default function LinkExamples() {
  return (
    <>
      {/* Basic link */}
      <Link href="/about">About</Link>

      {/* Replace history instead of push */}
      <Link href="/login" replace>Login</Link>

      {/* Scroll to top on navigation */}
      <Link href="/docs" scroll={true}>Docs</Link>

      {/* Disable prefetching */}
      <Link href="/heavy-page" prefetch={false}>
        Heavy Page
      </Link>

      {/* Custom className */}
      <Link href="/contact" className="text-blue-500 hover:underline">
        Contact Us
      </Link>
    </>
  );
}
```

---

# Active Links

```tsx
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';

export default function Navigation() {
  const pathname = usePathname();

  const links = [
    { href: '/', label: 'Home' },
    { href: '/about', label: 'About' },
    { href: '/blog', label: 'Blog' },
  ];

  return (
    <nav>
      {links.map(link => {
        const isActive = pathname === link.href;

        return (
          <Link
            key={link.href}
            href={link.href}
            className={isActive ? 'font-bold text-blue-600' : 'text-gray-600'}
          >
            {link.label}
          </Link>
        );
      })}
    </nav>
  );
}
```

---

# Loading States

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return (
    <div className="flex items-center justify-center min-h-screen">
      <div className="animate-spin rounded-full h-32 w-32 border-b-2 border-gray-900" />
    </div>
  );
}
```

<v-clicks>

- Automatically wraps page in React Suspense
- Shows while page/layout loads
- Instant loading states
- Works with streaming SSR

</v-clicks>

---

# Error Handling

```tsx
// app/dashboard/error.tsx
'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h2 className="text-2xl font-bold mb-4">Something went wrong!</h2>
      <p className="mb-4">{error.message}</p>
      <button
        onClick={reset}
        className="px-4 py-2 bg-blue-500 text-white rounded"
      >
        Try again
      </button>
    </div>
  );
}
```

---

# Not Found Pages

```tsx
// app/blog/[slug]/not-found.tsx
import Link from 'next/link';

export default function NotFound() {
  return (
    <div className="text-center py-20">
      <h2 className="text-3xl font-bold mb-4">Blog Post Not Found</h2>
      <p className="mb-4">Could not find the requested blog post.</p>
      <Link href="/blog" className="text-blue-500 hover:underline">
        Return to Blog
      </Link>
    </div>
  );
}
```

```tsx
// Trigger it in page.tsx
import { notFound } from 'next/navigation';

export default async function BlogPost({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug);

  if (!post) {
    notFound(); // Triggers not-found.tsx
  }

  return <article>...</article>;
}
```

---

# Parallel Routes

**Problem in React:** Loading multiple independent sections in a layout requires manual state management for each section's loading/error states.

```
app/
└── dashboard/
    ├── @user/
    │   ├── page.tsx
    │   ├── loading.tsx    # Independent loading state
    │   └── error.tsx      # Independent error handling
    ├── @analytics/
    │   ├── page.tsx
    │   ├── loading.tsx    # Separate loading state
    │   └── error.tsx      # Separate error handling
    └── layout.tsx
```

---

# Parallel Routes in Action

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  user,
  analytics,
}: {
  user: React.ReactNode;
  analytics: React.ReactNode;
}) {
  return (
    <div className="grid grid-cols-2 gap-4">
      <div>{user}</div>
      <div>{analytics}</div>
    </div>
  );
}
```

<v-clicks>

- `@folder` = named slot (parallel route)
- Each slot loads independently
- Each slot can have its own loading.tsx/error.tsx
- Perfect for dashboards, split views

</v-clicks>

---

# Intercepting Routes

**Use Case:** Instagram-style photo modals - click a photo in the feed, it opens in a modal, but if you share/refresh the URL, it shows the full page.

```
app/
├── feed/
│   └── page.tsx              # Photo feed
├── photo/
│   └── [id]/
│       └── page.tsx          # Full photo page (direct URL access)
└── @modal/
    └── (.)photo/
        └── [id]/
            └── page.tsx      # Photo in modal (when navigating from feed)
```

---

# Intercepting Routes Conventions

| Pattern | Description |
|---------|-------------|
| (.) | Same level |
| (..) | One level above |
| (..)(..) | Two levels above |
| (...) | From root app |

<v-clicks>

**How it works:**
- Clicking link in feed shows modal (intercepted route)
- Direct URL access shows full page (original route)
- Share URL and recipient sees full page
- Back button closes modal, returns to feed

</v-clicks>

---

# Routing Summary

| React Router | Next.js App Router |
|--------------|-------------------|
| Manual route config | File-based routing |
| `useParams()` hook | `params` prop injected |
| Manual Suspense | `loading.tsx` automatic |
| Manual ErrorBoundary | `error.tsx` automatic |
| No prefetching | Auto prefetch visible links |
| Single route per view | Parallel routes for multi-view |
| No interception | Route interception for modals |

**Next.js routing = Less code, more features, better DX**
