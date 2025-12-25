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

<v-clicks>

**Problems:**
- API keys exposed in client code
- Need separate backend for sensitive operations
- CORS configuration required
- Two separate deployments to manage
- Environment variable handling complex

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
  { params }: { params: { id: string } }
) {
  const post = await db.post.findUnique({
    where: { id: params.id }
  });

  if (!post) {
    return Response.json(
      { error: 'Post not found' },
      { status: 404 }
    );
  }

  return Response.json(post);
}

export async function PATCH(
  request: Request,
  { params }: { params: { id: string } }
) {
  const body = await request.json();
  const post = await db.post.update({
    where: { id: params.id },
    data: body,
  });

  return Response.json(post);
}
```

---

# Request & Response

```tsx
// app/api/users/route.ts
import { NextRequest } from 'next/server';

export async function GET(request: NextRequest) {
  // Query parameters
  const searchParams = request.nextUrl.searchParams;
  const limit = searchParams.get('limit') || '10';

  // Headers
  const authHeader = request.headers.get('authorization');

  // Cookies
  const token = request.cookies.get('token');

  const users = await db.user.findMany({
    take: parseInt(limit),
  });

  return Response.json(users, {
    headers: {
      'Content-Type': 'application/json',
      'Cache-Control': 'max-age=60',
    },
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

  // Validate
  if (!title || !content) {
    return { error: 'Title and content required' };
  }

  // Create post
  const post = await db.post.create({
    data: { title, content },
  });

  // Revalidate posts page
  revalidatePath('/posts');

  return { success: true, post };
}
```

---

# Using Server Actions in Forms

```tsx
// app/posts/new/page.tsx
import { createPost } from '@/app/actions';

export default function NewPostPage() {
  return (
    <form action={createPost}>
      <input
        type="text"
        name="title"
        placeholder="Title"
        required
      />

      <textarea
        name="content"
        placeholder="Content"
        required
      />

      <button type="submit">
        Create Post
      </button>
    </form>
  );
}
```

<v-clicks>

✅ **No API route needed!**
✅ **Progressive enhancement** (works without JS!)
✅ **Automatic revalidation**

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

      {state?.error && (
        <p className="text-red-500">{state.error}</p>
      )}

      {state?.success && (
        <p className="text-green-500">Post created!</p>
      )}
    </form>
  );
}
```

---

# Authentication Example

```tsx
// app/api/auth/login/route.ts
import { NextRequest } from 'next/server';
import { SignJWT } from 'jose';
import { cookies } from 'next/headers';

export async function POST(request: NextRequest) {
  const { email, password } = await request.json();

  // Verify credentials
  const user = await verifyCredentials(email, password);

  if (!user) {
    return Response.json(
      { error: 'Invalid credentials' },
      { status: 401 }
    );
  }

  // Create JWT token
  const token = await new SignJWT({ userId: user.id })
    .setProtectedHeader({ alg: 'HS256' })
    .setExpirationTime('24h')
    .sign(new TextEncoder().encode(process.env.JWT_SECRET));

  // Set cookie
  cookies().set('token', token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    maxAge: 60 * 60 * 24, // 24 hours
  });

  return Response.json({ user });
}
```

---

# Middleware

```tsx
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
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

Runs **before** every request!
Perfect for auth, redirects, headers.

</v-clicks>
