# React Deployment: The Setup

Deploying a React app requires extra configuration:

```bash
# React build output
npm run build

# Output:
build/
├── static/
│   ├── js/
│   │   └── main.abc123.js   # All your code
│   └── css/
│       └── main.def456.css
└── index.html               # Single HTML file
```

**You still need to:**
- Configure a web server (nginx, Apache)
- Set up client-side routing fallbacks
- Handle 404s for SPAs
- Set up a separate API server

---

# React vs Next.js Deployment

<div class="grid grid-cols-2 gap-4">

<div>

## React (Create React App)

- Static files only
- Need separate backend
- Manual server config
- SPA routing issues
- No SSR/SSG built-in

</div>

<div>

## Next.js

- Full-stack deployment
- API routes included
- Zero-config on Vercel
- SSR/SSG/ISR support
- Automatic optimizations

</div>

</div>

---

# Build for Production

```bash
# Production build
npm run build

# Output:
# Route (app)              Size     First Load JS
# ┌ ○ /                    142 B          87.4 kB
# ├ ○ /about               142 B          87.4 kB
# ├ ƒ /api/posts           0 B                0 B
# ├ ○ /blog                142 B          87.4 kB
# └ ● /blog/[slug]         142 B          87.4 kB

# Legend:
# ○ Static  - pre-rendered as static content
# ● SSG     - pre-rendered as static HTML (uses getStaticProps)
# ƒ Dynamic - server-rendered on demand
```

---

# Deploy to Vercel

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel

# Follow prompts:
# - Link to existing project or create new
# - Select Next.js framework (auto-detected)
# - Deploy!

# Production deployment
vercel --prod
```

<v-clicks>

✅ **Zero configuration**
✅ **Automatic HTTPS**
✅ **Global CDN**
✅ **Serverless functions**
✅ **Preview deployments** for every PR

</v-clicks>

---

# Environment Variables

```bash
# .env.local (for local development)
DATABASE_URL=postgresql://localhost:5432/mydb
API_KEY=secret_key_here
NEXT_PUBLIC_API_URL=https://api.example.com
```

```tsx
// Server-side (API routes, Server Components)
const apiKey = process.env.API_KEY; // ✅ Secret, not exposed

// Client-side (must use NEXT_PUBLIC_ prefix)
const apiUrl = process.env.NEXT_PUBLIC_API_URL; // ✅ Public, available in browser
```

<v-clicks>

**Important:**
- `NEXT_PUBLIC_*` = Available in browser (public)
- Without prefix = Server-only (secure)

</v-clicks>

---

# Production Environment Variables

On Vercel:

1. **Dashboard → Project → Settings → Environment Variables**

2. Add variables:
   ```
   DATABASE_URL=production_db_url
   API_KEY=production_api_key
   NEXT_PUBLIC_API_URL=https://api.production.com
   ```

3. Choose environment:
   - Production
   - Preview
   - Development

---

# Docker Deployment

```dockerfile
# Dockerfile
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Production image
FROM base AS runner
WORKDIR /app
ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs
EXPOSE 3000
ENV PORT 3000

CMD ["node", "server.js"]
```

---

# Self-Hosting with Node.js

```js
// next.config.js
module.exports = {
  output: 'standalone',
};
```

```bash
# Build
npm run build

# Outputs to .next/standalone/
# Contains everything needed to run

# Run in production
node .next/standalone/server.js
```

---

# Static Export

```js
// next.config.js
module.exports = {
  output: 'export',
};
```

```bash
npm run build

# Outputs pure static files to 'out/'
# Deploy to:
# - GitHub Pages
# - Netlify
# - Cloudflare Pages
# - Any static host
```

<v-clicks>

**Limitations:**
- No API routes
- No SSR
- No ISR
- No dynamic routes without generateStaticParams

</v-clicks>

---

# Performance Monitoring

```tsx
// app/layout.tsx
import { Analytics } from '@vercel/analytics/react';
import { SpeedInsights } from '@vercel/speed-insights/next';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
        <SpeedInsights />
      </body>
    </html>
  );
}
```

Track:
- Page views
- Web Vitals (LCP, FID, CLS)
- Real user metrics

---

# Best Practices Summary

<div class="grid grid-cols-2 gap-4">

<div>

## Performance
- Use Server Components by default
- Lazy load heavy components
- Optimize images with `next/image`
- Enable ISR where possible
- Stream with Suspense

## SEO
- Generate metadata dynamically
- Use semantic HTML
- Implement structured data
- Create XML sitemap
- Add robots.txt

</div>

<div>

## Security
- Keep secrets server-side
- Validate all inputs
- Use HTTPS
- Implement CSRF protection
- Rate limit API routes

## Developer Experience
- Use TypeScript
- Follow ESLint rules
- Write tests
- Use Git hooks (Husky)
- Document your code

</div>

</div>

---

# Advanced Topics

<v-clicks>

## Topics We Didn't Cover (But You Should Explore)

- **Internationalization (i18n)** - Multi-language support
- **Testing** - Jest, React Testing Library, Playwright
- **Database Integration** - Prisma, Drizzle ORM
- **Authentication** - NextAuth.js, Clerk, Auth0
- **Forms** - React Hook Form, Zod validation
- **State Management** - Zustand, Redux Toolkit
- **Real-time** - WebSockets, Pusher, Ably
- **Payments** - Stripe integration
- **File Uploads** - UploadThing, Cloudinary
- **Edge Runtime** - Middleware at the edge

</v-clicks>
