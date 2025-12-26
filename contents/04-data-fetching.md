# React Data Fetching: The Traditional Way

Every component that needs data follows the same pattern:

```tsx
function ProductList() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch('/api/products')
      .then(res => {
        if (!res.ok) throw new Error('Failed to fetch');
        return res.json();
      })
      .then(data => setProducts(data))
      .catch(err => setError(err))
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;
  return <ProductGrid products={products} />;
}
```

**useState + useEffect + loading + error = every component!**

---

# React Data Fetching Pain Points

<div class="grid grid-cols-2 gap-4">

<div>

## Boilerplate Everywhere

```tsx
// Repeated in every component:
const [data, setData] = useState(null);
const [loading, setLoading] = useState(true);
const [error, setError] = useState(null);

useEffect(() => {
  // fetch logic...
}, []);
```

</div>

<div>

## Problems

<v-clicks>

- Same pattern repeated everywhere
- Loading states managed manually
- Error handling in each component
- No built-in caching
- Waterfalls (fetch after render)
- SEO issues (data fetched on client)
- API keys exposed in browser

</v-clicks>

</div>

</div>

---

# React: Request Waterfall Problem

```tsx
function Dashboard() {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState(null);

  useEffect(() => {
    // These run sequentially, not in parallel!
    fetch('/api/user').then(r => r.json()).then(setUser);
    fetch('/api/posts').then(r => r.json()).then(setPosts);
  }, []);

  // Even worse with nested components:
  return (
    <UserProfile user={user}>
      {/* PostList fetches its own data AFTER UserProfile renders */}
      <PostList userId={user?.id} />
    </UserProfile>
  );
}
```

**Each level waits for the parent to fetch first!**

---

# How Next.js Solves Data Fetching

<div class="grid grid-cols-2 gap-4">

<div>

## React (Client-Side)

```tsx
function Products() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/products')
      .then(r => r.json())
      .then(setProducts)
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <Spinner />;
  return <ProductList data={products} />;
}
```

</div>

<div>

## Next.js (Server Components)

```tsx
async function Products() {
  // Fetch on server - no loading state!
  const products = await db.products.findMany();

  return <ProductList data={products} />;
}
```

- No useState/useEffect
- No loading boilerplate
- SEO-friendly (HTML ready)
- Secure (secrets on server)

</div>

</div>

---

# Data Fetching in Next.js

<div class="grid grid-cols-2 gap-4">

<div>

## Server-Side
- **Server Components** (default)
- Direct database access
- No API needed
- Secure
- Fast

</div>

<div>

## Client-Side
- **Client Components**
- Browser APIs
- Interactive features
- Similar to React

</div>

</div>

---

# Server Component Data Fetching

```tsx
// app/products/page.tsx
// This is a Server Component by default!

async function getProducts() {
  const res = await fetch('https://api.example.com/products');

  if (!res.ok) {
    throw new Error('Failed to fetch products');
  }

  return res.json();
}

export default async function ProductsPage() {
  const products = await getProducts();

  return (
    <div>
      <h1>Products</h1>
      {products.map(product => (
        <div key={product.id}>{product.name}</div>
      ))}
    </div>
  );
}
```

✅ Runs on the server, SEO-friendly, no loading state needed!

---

# Direct Database Access

```tsx
// app/users/page.tsx
import { db } from '@/lib/database';

export default async function UsersPage() {
  // ✅ Direct database access in Server Component!
  const users = await db.user.findMany({
    orderBy: { createdAt: 'desc' },
    take: 10,
  });

  return (
    <div>
      <h1>Recent Users</h1>
      {users.map(user => (
        <div key={user.id}>
          {user.name} - {user.email}
        </div>
      ))}
    </div>
  );
}
```

<v-clicks>

- **No API route needed!**
- **Direct database access**
- **Secure** (credentials never sent to client)

</v-clicks>

---

# Fetch with Caching

```tsx
// No caching by default (Next.js 16+)
const data = await fetch('https://api.example.com/data');

// Revalidate every 60 seconds (time-based caching)
const data = await fetch('https://api.example.com/data', {
  next: { revalidate: 60 }
});

// Force caching (explicit opt-in)
const data = await fetch('https://api.example.com/data', {
  cache: 'force-cache'
});

// Revalidate with tag (for on-demand revalidation)
const data = await fetch('https://api.example.com/data', {
  next: { tags: ['products'] }
});
```

**Note:** Next.js 16 uses explicit opt-in caching with the `"use cache"` directive.

---

# Parallel Data Fetching

```tsx
export default async function Dashboard() {
  // ❌ Sequential - slow!
  const user = await fetchUser();
  const posts = await fetchPosts();
  const comments = await fetchComments();

  // ✅ Parallel - fast!
  const [user, posts, comments] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchComments(),
  ]);

  return (
    <div>
      <UserProfile user={user} />
      <PostsList posts={posts} />
      <CommentsList comments={comments} />
    </div>
  );
}
```

---

# Sequential Data Fetching

```tsx
export default async function BlogPost({
  params
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params;

  // Fetch post first
  const post = await fetchPost(slug);

  // Then fetch author (depends on post data)
  const author = await fetchAuthor(post.authorId);

  // Then fetch related posts (depends on post categories)
  const relatedPosts = await fetchRelatedPosts(post.categories);

  return (
    <article>
      <h1>{post.title}</h1>
      <AuthorCard author={author} />
      <div>{post.content}</div>
      <RelatedPosts posts={relatedPosts} />
    </article>
  );
}
```

Use when data depends on previous fetches.

---

# Streaming with Suspense

```tsx
import { Suspense } from 'react';

async function SlowComponent() {
  const data = await fetchSlowData(); // Takes 3 seconds
  return <div>{data}</div>;
}

export default function Page() {
  return (
    <div>
      <h1>Welcome</h1>

      {/* This shows immediately */}
      <p>Some fast content here</p>

      {/* This streams in when ready */}
      <Suspense fallback={<div>Loading...</div>}>
        <SlowComponent />
      </Suspense>
    </div>
  );
}
```

**Progressive rendering** - don't wait for slow data!

---

# Client-Side Data Fetching

```tsx
'use client';

import { useState, useEffect } from 'react';

export default function ClientProducts() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/products')
      .then(res => res.json())
      .then(data => {
        setProducts(data);
        setLoading(false);
      });
  }, []);

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      {products.map(product => (
        <div key={product.id}>{product.name}</div>
      ))}
    </div>
  );
}
```

Use when you need interactivity or client-side updates.

---

# SWR for Client Fetching

```tsx
'use client';

import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(res => res.json());

export default function Profile() {
  const { data, error, isLoading } = useSWR('/api/user', fetcher);

  if (error) return <div>Failed to load</div>;
  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      <h1>Hello {data.name}!</h1>
    </div>
  );
}
```

<v-clicks>

✅ Automatic caching
✅ Automatic revalidation
✅ Focus revalidation
✅ Built by Vercel team

</v-clicks>

---

# React Query (TanStack Query)

```tsx
'use client';

import { useQuery } from '@tanstack/react-query';

export default function Posts() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['posts'],
    queryFn: async () => {
      const res = await fetch('/api/posts');
      return res.json();
    },
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error loading posts</div>;

  return (
    <div>
      {data.map(post => (
        <div key={post.id}>{post.title}</div>
      ))}
    </div>
  );
}
```

---

# Revalidating Data

```tsx
// app/actions.ts
'use server';

import { revalidatePath, revalidateTag } from 'next/cache';

export async function createPost(data: FormData) {
  // Create post in database
  await db.post.create({
    data: {
      title: data.get('title'),
      content: data.get('content'),
    },
  });

  // Revalidate the posts page
  revalidatePath('/posts');

  // Or revalidate by tag
  revalidateTag('posts');
}
```
