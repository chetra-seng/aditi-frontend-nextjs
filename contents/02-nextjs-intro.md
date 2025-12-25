# Enter Next.js

**The React Framework for Production**

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

<div class="grid grid-cols-4 gap-4 text-center">

<div>

**Netflix**
Streaming platform

</div>

<div>

**TikTok**
Web experience

</div>

<div>

**Notion**
Productivity app

</div>

<div>

**Hulu**
Media streaming

</div>

</div>

<div class="grid grid-cols-4 gap-4 text-center mt-4">

<div>

**Nike**
E-commerce

</div>

<div>

**Twitch**
Gaming platform

</div>

<div>

**Target**
Retail giant

</div>

<div>

**Washington Post**
News media

</div>

</div>

---

# Next.js by the Numbers

<div class="grid grid-cols-3 gap-8 text-center">

<div>

## 120K+
GitHub Stars

</div>

<div>

## 6M+
Weekly npm downloads

</div>

<div>

## 500K+
Websites in production

</div>

</div>

<v-clicks>

**Why developers choose Next.js:**
- Backed by Vercel (well-funded, active development)
- React team collaboration (React features land here first)
- Massive ecosystem and community
- Excellent documentation
- Production-proven at scale

</v-clicks>

---

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

# How Next.js Solves React Problems

<div class="grid grid-cols-2 gap-4">

<div>

## React Problem → Next.js Solution

**SEO Issues**
✓ Server-Side Rendering (SSR)

**Performance**
✓ Automatic Code Splitting

**Routing**
✓ File-based Routing

**Data Fetching**
✓ Server Components

</div>

<div>

**API Security**
✓ API Routes & Server Actions

**Image Optimization**
✓ Built-in Image Component

**Build Output**
✓ Static & Dynamic Rendering

**Code Organization**
✓ App Router Structure

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

# Understanding app/page.tsx

```tsx
// app/page.tsx - Your home page
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
- **Server Component** by default
- **TypeScript** support out of the box

</v-clicks>

---

# Understanding app/layout.tsx

```tsx
// app/layout.tsx - Root layout wraps all pages
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';
import './globals.css';

const inter = Inter({ subsets: ['latin'] });

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
      <body className={inter.className}>{children}</body>
    </html>
  );
}
```

---

# Server Components by Default

```tsx
// app/page.tsx - This is a Server Component!
export default async function HomePage() {
  // ✅ This runs on the SERVER
  const data = await fetch('https://api.example.com/data');
  const products = await data.json();

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

<v-clicks>

- **Runs on the server**
- Can directly access databases
- Smaller JavaScript bundles
- Better performance
- SEO-friendly

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
