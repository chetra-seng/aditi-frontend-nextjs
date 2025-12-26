
# What is Next.js?

<v-clicks>

- **A React framework** built by Vercel
- **Production-ready** out of the box
- **Solves all the limitations** we just discussed
- **Full-stack** framework (frontend + backend)
- **Highly optimized** by default
- **Great developer experience**

</v-clicks>

---

# Next.js: The Background

<div class="grid grid-cols-2 gap-4">

<div>

## Timeline

- **2016** - Created by Vercel (formerly Zeit)
- **2020** - Next.js 10 with Image component
- **2022** - Next.js 13 with App Router
- **2023** - Server Actions, Partial Prerendering
- **2024** - Next.js 15 with React 19
- **2025** - Next.js 16 with Turbopack, React 19.2 (current)

</div>

<div>

## Why It Was Created

Guillermo Rauch (Vercel CEO) saw that React alone wasn't enough for production:

- No built-in routing
- No server-side rendering
- Complex webpack configuration
- Poor SEO out of the box

**Goal:** Zero-config React for production

</div>

</div>

---

# Who Uses Next.js?

<div class="grid grid-cols-4 gap-8 text-center mt-4">

<div class="flex flex-col items-center">
<img src="https://cdn.simpleicons.org/netflix" class="w-12 h-12 mb-2" />
<strong>Netflix</strong>
</div>

<div class="flex flex-col items-center">
<img src="https://cdn.simpleicons.org/tiktok" class="w-12 h-12 mb-2" />
<strong>TikTok</strong>
</div>

<div class="flex flex-col items-center">
<img src="https://cdn.simpleicons.org/notion/white" class="w-12 h-12 mb-2" />
<strong>Notion</strong>
</div>

<div class="flex flex-col items-center">
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/ef/ChatGPT-Logo.svg/960px-ChatGPT-Logo.svg.png" class="w-12 h-12 mb-2 rounded" />
<strong>ChatGPT</strong>
</div>

<div class="flex flex-col items-center">
<img src="https://cdn.simpleicons.org/nike" class="w-12 h-12 mb-2" />
<strong>Nike</strong>
</div>

<div class="flex flex-col items-center">
<img src="https://cdn.simpleicons.org/twitch" class="w-12 h-12 mb-2" />
<strong>Twitch</strong>
</div>

<div class="flex flex-col items-center">
<img src="https://cdn.simpleicons.org/target" class="w-12 h-12 mb-2" />
<strong>Target</strong>
</div>

<div class="flex flex-col items-center">
<img src="https://cdn.simpleicons.org/thewashingtonpost" class="w-12 h-12 mb-2" />
<strong>Washington Post</strong>
</div>

</div>

---

# How Next.js Solves React Problems

<div class="grid grid-cols-2 gap-4">

<div>


**SEO Issues** <br />
✅ Server-Side Rendering (SSR)

**Performance** <br />

✅ Automatic Code Splitting

**Routing** <br />

✅ File-based Routing

**Data Fetching** <br />

✅ Server Components

</div>

<div>

**API Security** <br />

✅ API Routes & Server Actions

**Image Optimization** <br />

✅ Built-in Image Component

**Build Output** <br />

✅ Static & Dynamic Rendering

**Code Organization** <br />

✅ App Router Structure

</div>

</div>

---

# Next.js Key Features

```tsx
// Next.js solves this elegantly
import Image from 'next/image';

export default async function ProductPage({ params }) {
  // ✅ Fetch on the server - secure, fast
  const product = await db.products.findById(params.id);

  return (
    <div>
      {/* ✅ Automatic image optimization */}
      <Image
        src={product.image}
        alt={product.name}
        width={500}
        height={500}
      />

      <h1>{product.name}</h1>
      <p>{product.description}</p>
    </div>
  );
}
```

---

# Creating a Next.js App

```bash
# Create a new Next.js app
npx create-next-app@latest my-next-app

# You'll be prompted:
✔ Would you like to use TypeScript? Yes
✔ Would you like to use ESLint? Yes
✔ Would you like to use Tailwind CSS? Yes
✔ Would you like to use `src/` directory? No
✔ Would you like to use App Router? Yes
✔ Would you like to customize the default import alias? No

# Navigate and start
cd my-next-app
npm run dev
```

App running at http://localhost:3000

<v-clicks>
<div class="p-4 bg-blue-500/20 rounded-lg">
  💡 What is Tailwind CSS?
</div>
</v-clicks>
---
src: ./02a-tailwind-intro.md
---

# Project Structure

```
my-next-app/
├── app/                    # App Router (Next.js 13+)
│   ├── layout.tsx         # Root layout
│   ├── page.tsx           # Home page (/)
│   └── globals.css        # Global styles
├── public/                # Static assets
│   └── images/
├── node_modules/
├── package.json
├── next.config.js         # Next.js configuration
├── tsconfig.json          # TypeScript config
└── tailwind.config.ts     # Tailwind config
```

---

# App Router vs Pages Router

<div class="grid grid-cols-2 gap-4">

<div>

## Pages Router (Legacy)
```
pages/
  index.tsx         → /
  about.tsx         → /about
  blog/
    [slug].tsx      → /blog/:slug
  api/
    hello.ts        → /api/hello
```

- Older approach
- Client Components by default
- `getServerSideProps`
- `getStaticProps`

</div>

<div>

## App Router (Modern) ✅
```
app/
  page.tsx          → /
  about/
    page.tsx        → /about
  blog/
    [slug]/
      page.tsx      → /blog/:slug
  api/
    hello/
      route.ts      → /api/hello
```

- **Recommended** for new apps
- Server Components by default
- Better performance
- More features

</div>

</div>

**We'll focus on App Router!**

---

# Data Fetching: Pages vs App Router

````md magic-move
```tsx
// Pages Router - pages/products.tsx
import { GetServerSideProps } from 'next';

interface Props {
  products: Product[];
}

export const getServerSideProps: GetServerSideProps<Props> = async () => {
  const res = await fetch('https://api.example.com/products');
  const products = await res.json();

  return {
    props: { products }
  };
};

export default function ProductsPage({ products }: Props) {
  return (
    <div>
      {products.map(p => <div key={p.id}>{p.name}</div>)}
    </div>
  );
}
```

```tsx
// App Router - app/products/page.tsx



async function getProducts() {
  const res = await fetch('https://api.example.com/products');
  return res.json();
}



export default async function ProductsPage() {
  const products = await getProducts();

  return (
    <div>
      {products.map(p => <div key={p.id}>{p.name}</div>)}
    </div>
  );
}
```

```tsx
// App Router - Even simpler! 🎉



export default async function ProductsPage() {
  const res = await fetch('https://api.example.com/products');
  const products = await res.json();

  return (
    <div>
      {products.map(p => <div key={p.id}>{p.name}</div>)}
    </div>
  );
}
```
````

---
layout: center
---

# Server vs Client Components

Understanding the rendering model 🖥️

---

# How Next.js Renders a Page

```
Browser Request: /products
        │
        ▼
┌─────────────────────────────────────────────────┐
│                    SERVER                        │
│                                                  │
│  1. layout.tsx    ─── Renders shell (HTML/body) │
│        │                                         │
│        ▼                                         │
│  2. page.tsx      ─── Fetches data, renders UI  │
│        │                                         │
│        ▼                                         │
│  3. Components    ─── Server components render  │
│                                                  │
└─────────────────────────────────────────────────┘
        │
        ▼ HTML + minimal JS sent to browser
┌─────────────────────────────────────────────────┐
│                   BROWSER                        │
│                                                  │
│  4. Client Components hydrate (become interactive)│
│                                                  │
└─────────────────────────────────────────────────┘
```

---

# Server vs Client Components

<div class="grid grid-cols-2 gap-8">

<div>

## Server Components (Default)

- Run **only on the server**
- Can fetch data directly
- Can access backend resources
- **Zero JavaScript** sent to browser
- Cannot use hooks or browser APIs

```tsx
// No directive needed - default!
async function ProductList() {
  const products = await db.query();
  return <ul>...</ul>;
}
```

</div>

<div>

## Client Components

- Run on **server + browser**
- Enable interactivity
- Can use React hooks
- Add JavaScript to bundle

```tsx
'use client'; // ← Required!

function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={...}>
}
```

</div>

</div>

---

# The Component Tree

```
         layout.tsx (Server)
              │
              ▼
         page.tsx (Server)
         ┌────┴────┐
         │         │
    ProductList  Sidebar
     (Server)    (Server)
         │
    ┌────┴────┐
    │         │
ProductCard  AddToCart
 (Server)    (Client) ← 'use client'
                │
           useState, onClick
```

**Rule:** Client components can only import other client components

**Tip:** Keep client components as leaves in your tree

---

# Server Components by Default

```tsx
// app/page.tsx - This is a Server Component!
export default async function HomePage() {
  // ✅ Runs on the SERVER - never exposed to browser
  const products = await db.products.findMany();

  return (
    <div>
      <h1>Products</h1>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

<v-clicks>

- **No loading spinners** - data ready on first render
- **No API endpoints** needed for internal data
- **Secrets stay secret** - server code never reaches browser
- **SEO-friendly** - full HTML sent to crawlers

</v-clicks>

---

# When to Use Client Components

```tsx
'use client'; // ← Add this directive

import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

<v-clicks>

**Use 'use client' when you need:**
- `useState`, `useEffect`, hooks
- Event listeners (`onClick`, etc.)
- Browser APIs (`localStorage`, etc.)
- Interactive UI

</v-clicks>

---

# Understanding app/page.tsx

```tsx
// app/page.tsx - Your home page (Server Component!)
export default function Home() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-center">
      <h1 className="text-4xl font-bold">Welcome to Next.js!</h1>
      <p className="mt-4 text-lg">
        The React Framework for Production
      </p>
    </main>
  );
}
```

<v-clicks>

- **File name `page.tsx`** makes it a route
- **Default export** is the component
- **Server Component** by default - can be `async`!
- **TypeScript** support out of the box

</v-clicks>

---

# Understanding app/layout.tsx

```tsx
// app/layout.tsx - Root layout wraps all pages
import type { Metadata } from 'next';
import './globals.css';

export const metadata: Metadata = {
  title: 'My Next.js App',
  description: 'Built with Next.js',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

Layouts are **Server Components** - they wrap pages and persist across navigation.

---

# File Conventions in App Router

| File | Purpose |
|------|---------|
| `page.tsx` | Page UI (makes route publicly accessible) |
| `layout.tsx` | Shared UI wrapper for segment and children |
| `loading.tsx` | Loading UI (automatic Suspense boundary) |
| `error.tsx` | Error UI (automatic Error boundary) |
| `not-found.tsx` | 404 UI |
| `route.ts` | API endpoint |
| `template.tsx` | Re-rendered layout |
| `default.tsx` | Parallel route fallback |

---

# Creating Your First Page

Let's create an "About" page:

```bash
mkdir app/about
touch app/about/page.tsx
```

```tsx
// app/about/page.tsx
export default function AboutPage() {
  return (
    <main className="p-8">
      <h1 className="text-3xl font-bold mb-4">About Us</h1>
      <p className="text-lg">
        We're building amazing apps with Next.js!
      </p>
    </main>
  );
}
```

Now visit: http://localhost:3000/about

✅ **No routing configuration needed!**
