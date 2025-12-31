# React: The Backend Problem

React is frontend-only. For a full-stack app, you need:

```
┌─────────────────────┐     ┌─────────────────────┐
│   React Frontend    │────▶│   Separate Backend  │
│   (localhost:3000)  │     │   (localhost:4000)  │
└─────────────────────┘     └─────────────────────┘
         │                           │
         │   CORS issues!            │
         │   Two deployments!        │
         │   Two codebases!          │
         └───────────────────────────┘
```

<v-clicks>

**Common Setups:**
- React + Express.js (Node)
- React + Django (Python)
- React + Rails (Ruby)
- Two repos, two deployments, CORS config...

</v-clicks>

---

# React Backend Challenges

```tsx
// React component - exposed API key!
const STRIPE_KEY = 'sk_live_xxx';  // Anyone can see this!

function CheckoutButton() {
  const handlePayment = async () => {
    // Direct API call from browser - INSECURE
    await fetch('https://api.stripe.com/v1/charges', {
      headers: { Authorization: `Bearer ${STRIPE_KEY}` }
    });
  };
}
```

**Problems:**
<v-clicks>

- API keys exposed in client code
- Need separate backend for sensitive operations
- <span v-mark="{at: 3, color: 'red'}">CORS configuration required</span>
- Two separate deployments to manage
- Environment variable handling complex

</v-clicks>

---

# What is CORS?

**Cross-Origin Resource Sharing** - a browser security feature.

```
Frontend: http://localhost:3000
Backend:  http://localhost:4000  ← Different origin!
```

<v-clicks>

When your frontend tries to call a different origin:

```tsx
// Frontend at localhost:3000
fetch('http://localhost:4000/api/users')  // ❌ Blocked by browser!
```

**Browser says:** "That's a different origin. Backend must explicitly allow this."

</v-clicks>

---

# CORS in Practice

```tsx
// Express backend - must add CORS headers
import cors from 'cors';

app.use(cors({
  origin: 'http://localhost:3000',  // Allow frontend origin
  credentials: true,                 // Allow cookies
}));
```

<v-clicks>

**Same origin = no CORS needed:**
- `localhost:3000/page` → `localhost:3000/api` ✅

**Different origin = CORS required:**
- `localhost:3000` → `localhost:4000` ❌
- `myapp.com` → `api.myapp.com` ❌

**Next.js API routes are same-origin = no CORS config!**

</v-clicks>

---

# How Next.js Solves This

<div class="grid grid-cols-2 gap-4">

<div>

## React + Express

```
project/
├── frontend/          # React
│   ├── src/
│   └── package.json
└── backend/           # Express
    ├── routes/
    └── package.json
```

Two codebases, two deployments

</div>

<div>

## Next.js (Full-Stack)

```
project/
├── app/
│   ├── page.tsx       # Frontend
│   └── api/
│       └── route.ts   # Backend
└── package.json
```

One codebase, one deployment!

</div>

</div>

---

# API Routes

```tsx
// app/api/hello/route.ts
export async function GET() {
  return Response.json({ message: 'Hello from Next.js!' });
}
```

```bash
# Request
curl http://localhost:3000/api/hello

# Response
{"message":"Hello from Next.js!"}
```

<v-clicks>

- **File-based API routes**
- **Full Node.js environment**
- **Serverless-ready**
- **No CORS issues**

</v-clicks>

---

# HTTP Methods

```tsx
// app/api/posts/route.ts
import { NextRequest } from 'next/server';

export async function GET(request: NextRequest) {
  const posts = await db.post.findMany();
  return Response.json(posts);
}

export async function POST(request: NextRequest) {
  const body = await request.json();
  const post = await db.post.create({ data: body });
  return Response.json(post, { status: 201 });
}

export async function DELETE(request: NextRequest) {
  await db.post.deleteMany();
  return new Response(null, { status: 204 });
}
```

---

# Dynamic API Routes

```tsx
// app/api/posts/[id]/route.ts
export async function GET(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  const post = await db.post.findUnique({ where: { id } });

  if (!post) {
    return Response.json({ error: 'Not found' }, { status: 404 });
  }
  return Response.json(post);
}
```

<v-clicks>

- Same `[param]` syntax as pages
- Access via `params` object (must `await` in Next.js 15)
- Works with all HTTP methods (GET, POST, PATCH, DELETE)

</v-clicks>

---

# Request & Response

```tsx
// app/api/users/route.ts
import { NextRequest } from 'next/server';

export async function GET(request: NextRequest) {
  // Query params: /api/users?limit=5
  const limit = request.nextUrl.searchParams.get('limit');

  // Headers & Cookies
  const auth = request.headers.get('authorization');
  const token = request.cookies.get('token');

  return Response.json(data, {
    headers: { 'Cache-Control': 'max-age=60' },
  });
}
```

---

# Server Actions

```tsx
// app/actions.ts
'use server';

import { revalidatePath } from 'next/cache';

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;

  await db.post.create({ data: { title, content } });
  revalidatePath('/posts');
}
```

<v-clicks>

- `'use server'` marks functions as server-only
- Receives `FormData` from forms
- Can revalidate cached data after mutations

</v-clicks>

---

# Using Server Actions in Forms

```tsx
// app/posts/new/page.tsx
import { createPost } from '@/app/actions';

export default function NewPostPage() {
  return (
    <form action={createPost}>
      <input type="text" name="title" placeholder="Title" required />
      <textarea name="content" placeholder="Content" required />
      <button type="submit">Create Post</button>
    </form>
  );
}
```

<v-clicks>

- No API route needed
- Progressive enhancement (works without JS!)
- Automatic revalidation

</v-clicks>

---

# Server Actions with useFormState

```tsx
'use client';
import { useFormState } from 'react-dom';
import { createPost } from '@/app/actions';

export default function PostForm() {
  const [state, formAction] = useFormState(createPost, null);

  return (
    <form action={formAction}>
      <input type="text" name="title" required />
      <textarea name="content" required />
      <button type="submit">Create Post</button>
      {state?.error && <p className="text-red-500">{state.error}</p>}
    </form>
  );
}
```

<v-clicks>

- `useFormState` gives access to action return value
- Handle errors/success states in the UI

</v-clicks>

---

# Authentication Example

```tsx
// app/api/auth/login/route.ts
import { cookies } from 'next/headers';

export async function POST(request: NextRequest) {
  const { email, password } = await request.json();
  const user = await verifyCredentials(email, password);

  if (!user) {
    return Response.json({ error: 'Invalid credentials' }, { status: 401 });
  }

  const token = await createJWT({ userId: user.id }); // your JWT logic
  cookies().set('token', token, { httpOnly: true, secure: true });

  return Response.json({ user });
}
```

<v-clicks>

- Access secrets via `process.env` (server-only)
- Set HTTP-only cookies for secure auth
- Use libraries like `jose` or `jsonwebtoken` for JWTs

</v-clicks>

---

# Proxy (Next.js 16+)

Code that runs **between** the request and your application.

<v-clicks>

**Note:** In Next.js 16, `middleware.ts` was renamed to `proxy.ts` to make the network boundary explicit.

**Common use cases:**
- Authentication checks
- Redirects (e.g., logged-in users → dashboard)
- Adding headers (e.g., security headers)
- Logging & analytics

</v-clicks>

---

# Proxy Flow

<div class="flex items-center justify-center gap-3 mt-8">
  <v-click>
    <div class="px-4 py-3 bg-blue-500 text-white rounded-lg shadow-lg">
      <div class="font-bold">1. Request</div>
      <div class="text-xs opacity-80">User visits /dashboard</div>
    </div>
  </v-click>
  <v-click><div class="text-2xl">→</div></v-click>
  <v-click>
    <div class="px-4 py-3 bg-yellow-500 text-black rounded-lg shadow-lg">
      <div class="font-bold">2. Proxy</div>
      <div class="text-xs opacity-80">Check auth, modify headers</div>
    </div>
  </v-click>
  <v-click><div class="text-2xl">→</div></v-click>
  <v-click>
    <div class="px-4 py-3 bg-green-500 text-white rounded-lg shadow-lg">
      <div class="font-bold">3. Page / API</div>
      <div class="text-xs opacity-80">Render the response</div>
    </div>
  </v-click>
</div>

<v-click>
<div class="mt-10 text-center">
  <span class="px-4 py-2 bg-gray-800 text-green-400 rounded-full font-bold">✨ Runs BEFORE your app on every matched request!</span>
</div>
</v-click>

---

# Proxy Example

```tsx
// proxy.ts (renamed from middleware.ts in Next.js 16)
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function proxy(request: NextRequest) {
  // Check authentication
  const token = request.cookies.get('token');

  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  // Add custom header
  const response = NextResponse.next();
  response.headers.set('x-custom-header', 'my-value');

  return response;
}

export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*'],
};
```

<v-clicks>

Runs **before** every request on **Node.js runtime**!
Perfect for auth, redirects, headers.

</v-clicks>
