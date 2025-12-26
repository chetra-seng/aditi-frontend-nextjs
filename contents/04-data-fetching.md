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
  const users = await db.user.findMany({ take: 10 });

  return <UserList users={users} />;
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

---

# Client-Side Data Fetching

```tsx
'use client';

import { useState, useEffect } from 'react';

export default function ClientProducts() {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    fetch('/api/products')
      .then(res => res.json())
      .then(setProducts);
  }, []);

  return <ProductList products={products} />;
}
```

Use when you need interactivity or real-time updates.

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

# Why React Query?

<div class="grid grid-cols-2 gap-4">

<div>

## Without React Query

- Manual loading/error states
- No caching strategy
- Duplicate requests
- Stale data problems
- Complex refetch logic

</div>

<div>

## With React Query

- Automatic state management
- Smart caching out of the box
- Request deduplication
- Background refetching
- Declarative and simple

</div>

</div>

---

# React Query Setup: Provider

```tsx
// app/providers.tsx
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useState } from 'react';

export default function Providers({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(() => new QueryClient());

  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}
```

<v-clicks>

- Must be a Client Component (`'use client'`)
- Create QueryClient inside `useState` to avoid sharing between requests
- Wrap your app with `QueryClientProvider`

</v-clicks>

---

# React Query Setup: Layout

```tsx
// app/layout.tsx
import Providers from './providers';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

Now all components can use `useQuery` and `useMutation`!

---

# Query Keys

```tsx
// Simple key
useQuery({ queryKey: ['todos'], queryFn: fetchTodos });

// Key with variables - React Query auto-refetches when key changes!
useQuery({
  queryKey: ['todo', todoId],
  queryFn: () => fetchTodo(todoId)
});

// Complex keys for filtering/pagination
useQuery({
  queryKey: ['todos', { status: 'done', page: 1 }],
  queryFn: () => fetchTodos({ status: 'done', page: 1 })
});
```

<v-clicks>

- Keys are compared **by value** - no need to memoize or keep stable references
- Keys are **hierarchical** - invalidating `['todos']` invalidates all todo queries
- Include **all variables** your query depends on

</v-clicks>

---

# useMutation: Creating Data

```tsx
'use client';

import { useMutation, useQueryClient } from '@tanstack/react-query';

export default function CreatePost() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (newPost: { title: string }) => {
      return fetch('/api/posts', {
        method: 'POST',
        body: JSON.stringify(newPost),
      }).then(res => res.json());
    },
    onSuccess: () => {
      // Invalidate and refetch posts
      queryClient.invalidateQueries({ queryKey: ['posts'] });
    },
  });

  return (
    <button onClick={() => mutation.mutate({ title: 'New Post' })}>
      {mutation.isPending ? 'Creating...' : 'Create Post'}
    </button>
  );
}
```

---

# Mutation States

```tsx
const mutation = useMutation({ mutationFn: createPost });

// Available states
mutation.isPending   // Request in progress
mutation.isSuccess   // Completed successfully
mutation.isError     // Failed
mutation.error       // Error object if failed
mutation.data        // Response data if successful

// Trigger the mutation
mutation.mutate(data);

// Or with async/await
const result = await mutation.mutateAsync(data);
```

---

# Invalidation Strategies

```tsx
const queryClient = useQueryClient();

// Invalidate a single query
queryClient.invalidateQueries({ queryKey: ['posts'] });

// Invalidate all queries starting with 'posts'
queryClient.invalidateQueries({ queryKey: ['posts'], exact: false });

// Invalidate specific post
queryClient.invalidateQueries({ queryKey: ['posts', postId] });

// Invalidate multiple queries
queryClient.invalidateQueries({
  predicate: (query) => query.queryKey[0] === 'posts'
});

// Remove from cache entirely
queryClient.removeQueries({ queryKey: ['posts', postId] });
```

---

# Caching Configuration

```tsx
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,  // 5 minutes until data is "stale"
      gcTime: 1000 * 60 * 30,    // 30 minutes in cache after unmount
    },
  },
});

// Or per-query
useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  staleTime: 1000 * 60,     // This specific query stays fresh for 1 min
  refetchOnWindowFocus: false,  // Don't refetch when tab gains focus
});
```

<v-clicks>

- **staleTime**: How long until data is considered stale (default: 0)
- **gcTime**: How long inactive data stays in cache (default: 5 min)
- Stale queries refetch automatically on mount/focus/reconnect

</v-clicks>

---

# React Query vs SWR

| Feature | React Query | SWR |
|---------|-------------|-----|
| Mutations | Built-in `useMutation` | Manual handling |
| Devtools | Yes | Yes |
| Cache invalidation | Powerful API | Basic |
| Pagination | Built-in | Plugin |
| Infinite queries | Built-in | Built-in |
| Bundle size | ~13kb | ~4kb |
| Learning curve | Moderate | Easy |

**Choose SWR** for simple fetching needs

**Choose React Query** for complex data management
