# Next.js 16 - Complete Introduction

## What is Next.js?

Next.js is a powerful React framework built by Vercel that enables you to build full-stack web applications with modern features like server-side rendering, static site generation, and API routes - all with minimal configuration.

## Why Choose Next.js?

### 1. **Hybrid Rendering Strategies**

Next.js offers multiple rendering options to optimize performance:

- **Server-Side Rendering (SSR)**: Generate HTML on each request for dynamic content
- **Static Site Generation (SSG)**: Pre-render pages at build time for blazing-fast performance
- **Incremental Static Regeneration (ISR)**: Update static content without rebuilding the entire site
- **Client-Side Rendering (CSR)**: Traditional React rendering when appropriate

### 2. **App Router (Next.js 13+)**

The modern App Router brings powerful features:

- File-based routing with intuitive folder structure
- Nested layouts and templates
- React Server Components by default
- Streaming and Suspense support
- Parallel and intercepting routes
- Route groups for organization

### 3. **Server Components & Actions**

- **React Server Components**: Render components on the server, reducing client-side JavaScript
- **Server Actions**: Write server-side logic directly in your components without separate API routes
- Improved performance and reduced bundle size

### 4. **Built-in Optimizations**

- **Image Optimization**: Automatic image resizing, lazy loading, and modern format conversion
- **Font Optimization**: Automatic font loading with zero layout shift
- **Script Optimization**: Efficient third-party script loading
- **Code Splitting**: Automatic code splitting for faster page loads

### 5. **API Routes & Backend Integration**

- Create serverless API endpoints alongside your frontend code
- Middleware for authentication, logging, and more
- Easy database and external API integration

### 6. **Developer Experience**

- Fast Refresh: Instant feedback on code changes
- TypeScript support out of the box
- Built-in ESLint configuration
- Comprehensive error messages
- Zero-config setup

### 7. **SEO & Performance**

- Excellent SEO capabilities with server-side rendering
- Automatic static optimization
- Built-in support for metadata and Open Graph tags
- Lighthouse score optimizations

### 8. **Data Fetching**

- Flexible data fetching patterns
- `fetch()` with automatic caching and revalidation
- Streaming with Suspense boundaries
- Parallel and sequential data fetching strategies

## Key Concepts

### File-System Based Routing

```
app/
├── page.tsx                 # Home page (/)
├── about/
│   └── page.tsx            # About page (/about)
├── blog/
│   ├── page.tsx            # Blog list (/blog)
│   └── [slug]/
│       └── page.tsx        # Dynamic blog post (/blog/post-1)
└── dashboard/
    ├── layout.tsx          # Shared layout
    └── settings/
        └── page.tsx        # Settings page (/dashboard/settings)
```

### Server vs Client Components

- **Server Components** (default): Fetch data, access backend resources, keep sensitive data secure
- **Client Components** (`'use client'`): Handle interactivity, use React hooks, access browser APIs

### Metadata API

```tsx
export const metadata = {
  title: "My App",
  description: "Welcome to my Next.js app",
};
```

### Route Handlers

Create API endpoints using `route.ts` files:

```tsx
export async function GET(request: Request) {
  return Response.json({ message: "Hello" });
}
```

## Getting Started

### Installation

```bash
npx create-next-app@latest my-app
cd my-app
npm run dev
```

### Project Structure

```
my-app/
├── app/                    # App router directory
│   ├── layout.tsx         # Root layout
│   ├── page.tsx           # Home page
│   └── globals.css        # Global styles
├── public/                # Static assets
├── next.config.js         # Next.js configuration
├── package.json
└── tsconfig.json          # TypeScript config
```

## What's New in Next.js 16?

### Major Features

1. **React 19 Support**: Built on the latest React with improved Server Components
2. **Turbopack Stable**: Faster bundler for development (up to 76.7% faster)
3. **Async Request APIs**: Breaking change - `params`, `searchParams`, and other APIs are now async
4. **Enhanced Caching**: More granular control over caching strategies
5. **Partial Prerendering (Stable)**: Combines static and dynamic rendering in a single route
6. **Form Actions**: Improved server actions with better TypeScript support

### Breaking Changes

- Request APIs like `cookies()`, `headers()`, and `params` are now asynchronous
- Minimum React version: 19
- Minimum Node.js version: 18.18

## When to Use Next.js?

✅ **Perfect for:**

- Marketing websites and landing pages
- E-commerce platforms
- Content-heavy sites (blogs, documentation)
- Dashboards and admin panels
- Applications requiring SEO
- Full-stack applications

❌ **Consider alternatives for:**

- Simple static sites (might be overkill)
- Apps requiring 100% client-side rendering
- Projects with very specific bundler requirements

## App Router vs Pages Router

### Overview

Next.js 13 introduced the **App Router** as a major paradigm shift, while maintaining backward compatibility with the **Pages Router** (used in Next.js 12 and earlier).

### Pages Router (Next.js 12 & Earlier)

The traditional routing system that uses the `pages/` directory:

```
pages/
├── index.tsx              # Route: /
├── about.tsx              # Route: /about
├── blog/
│   ├── index.tsx         # Route: /blog
│   └── [slug].tsx        # Route: /blog/:slug
├── _app.tsx              # Custom App component
├── _document.tsx         # Custom Document
└── api/
    └── hello.ts          # API Route: /api/hello
```

**Key Characteristics:**

- File represents a route (e.g., `pages/about.tsx` → `/about`)
- Uses `getServerSideProps`, `getStaticProps`, `getStaticPaths` for data fetching
- Single `_app.tsx` for global layout
- Client-side rendering by default
- Simpler mental model for beginners

**Data Fetching Example:**

```tsx
// pages/blog/[slug].tsx
export async function getServerSideProps(context) {
  const { slug } = context.params;
  const post = await fetchPost(slug);

  return {
    props: { post },
  };
}

export default function BlogPost({ post }) {
  return <div>{post.title}</div>;
}
```

### App Router (Next.js 13+)

The modern routing system using the `app/` directory:

```
app/
├── page.tsx              # Route: /
├── layout.tsx            # Root layout
├── loading.tsx           # Loading UI
├── error.tsx             # Error UI
├── about/
│   └── page.tsx         # Route: /about
├── blog/
│   ├── page.tsx         # Route: /blog
│   ├── layout.tsx       # Nested layout for /blog
│   └── [slug]/
│       ├── page.tsx     # Route: /blog/:slug
│       └── loading.tsx  # Loading state for this route
└── api/
    └── hello/
        └── route.ts     # API Route: /api/hello
```

**Key Characteristics:**

- `page.tsx` files define routes
- Nested layouts with `layout.tsx`
- Server Components by default (better performance)
- Built-in support for loading and error states
- Streaming and Suspense out of the box
- Colocation of components and routes

**Data Fetching Example:**

```tsx
// app/blog/[slug]/page.tsx
async function getPost(slug: string) {
  const res = await fetch(`https://api.example.com/posts/${slug}`, {
    next: { revalidate: 3600 }, // ISR with 1 hour revalidation
  });
  return res.json();
}

export default async function BlogPost({ params }) {
  const post = await getPost(params.slug);

  return <div>{post.title}</div>;
}
```

## Next.js 12 vs Next.js 13+: Major Differences

### 1. **Routing Architecture**

| Feature          | Next.js 12 (Pages Router) | Next.js 13+ (App Router)       |
| ---------------- | ------------------------- | ------------------------------ |
| Directory        | `pages/`                  | `app/`                         |
| Route definition | File = Route              | `page.tsx` = Route             |
| Layouts          | Single `_app.tsx`         | Nested `layout.tsx` files      |
| Loading states   | Manual implementation     | Built-in `loading.tsx`         |
| Error handling   | `_error.tsx`              | `error.tsx` per route          |
| Templates        | Not available             | `template.tsx` for re-mounting |

### 2. **Component Architecture**

| Feature              | Next.js 12        | Next.js 13+                |
| -------------------- | ----------------- | -------------------------- |
| Default rendering    | Client-side       | Server Components          |
| Client components    | All components    | Opt-in with `'use client'` |
| Server components    | Not available     | Default                    |
| Data fetching        | Special functions | Async components           |
| Component colocation | Separate folders  | Same folder as routes      |

### 3. **Data Fetching**

**Next.js 12:**

```tsx
// Three different functions for different strategies
export async function getServerSideProps() {} // SSR
export async function getStaticProps() {} // SSG
export async function getStaticPaths() {} // Dynamic SSG
```

**Next.js 13+:**

```tsx
// Unified approach with fetch API
async function getData() {
  const res = await fetch("https://api.example.com/data", {
    cache: "force-cache", // SSG (default)
    // cache: 'no-store',     // SSR
    // next: { revalidate: 10 } // ISR
  });
  return res.json();
}
```

### 4. **Metadata & SEO**

**Next.js 12:**

```tsx
import Head from "next/head";

export default function Page() {
  return (
    <>
      <Head>
        <title>My Page</title>
        <meta name="description" content="Page description" />
      </Head>
      <div>Content</div>
    </>
  );
}
```

**Next.js 13+:**

```tsx
export const metadata = {
  title: "My Page",
  description: "Page description",
  openGraph: {
    title: "My Page",
    description: "Page description",
  },
};

export default function Page() {
  return <div>Content</div>;
}
```

### 5. **API Routes**

**Next.js 12:**

```tsx
// pages/api/users.ts
export default function handler(req, res) {
  if (req.method === "GET") {
    res.status(200).json({ users: [] });
  }
}
```

**Next.js 13+:**

```tsx
// app/api/users/route.ts
export async function GET(request: Request) {
  return Response.json({ users: [] });
}

export async function POST(request: Request) {
  const body = await request.json();
  return Response.json({ success: true });
}
```

### 6. **Streaming & Suspense**

**Next.js 12:**

- Limited support for Suspense
- No native streaming
- Requires manual implementation

**Next.js 13+:**

```tsx
// app/dashboard/page.tsx
import { Suspense } from "react";

export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<p>Loading feed...</p>}>
        <Feed />
      </Suspense>
      <Suspense fallback={<p>Loading stats...</p>}>
        <Stats />
      </Suspense>
    </div>
  );
}
```

### 7. **Layouts & Nesting**

**Next.js 12:**

```tsx
// Single layout for entire app
// pages/_app.tsx
export default function App({ Component, pageProps }) {
  return (
    <Layout>
      <Component {...pageProps} />
    </Layout>
  );
}
```

**Next.js 13+:**

```tsx
// Nested layouts
// app/layout.tsx (Root)
export default function RootLayout({ children }) {
  return (
    <html>
      <body>{children}</body>
    </html>
  );
}

// app/dashboard/layout.tsx (Nested)
export default function DashboardLayout({ children }) {
  return (
    <div>
      <Sidebar />
      <main>{children}</main>
    </div>
  );
}
```

### 8. **Performance Improvements**

| Aspect               | Next.js 12         | Next.js 13+                 |
| -------------------- | ------------------ | --------------------------- |
| JavaScript bundle    | Larger             | Smaller (Server Components) |
| Data fetching        | Waterfall pattern  | Parallel fetching           |
| Code splitting       | Automatic by route | More granular               |
| Streaming            | Limited            | Built-in support            |
| Concurrent rendering | Not available      | Supported                   |

### 9. **Special Files**

**Pages Router:**

- `_app.tsx` - Custom App
- `_document.tsx` - Custom Document
- `_error.tsx` - Error page
- `404.tsx` - 404 page

**App Router:**

- `layout.tsx` - Shared UI for segments
- `page.tsx` - Route UI
- `loading.tsx` - Loading UI
- `error.tsx` - Error UI
- `not-found.tsx` - 404 UI
- `template.tsx` - Re-rendered layout
- `route.ts` - API endpoint

## Migration Considerations

### Should You Migrate?

**Stick with Pages Router if:**

- You have a stable, working application
- Team isn't familiar with Server Components
- You need features not yet available in App Router
- Migration effort outweighs benefits

**Migrate to App Router if:**

- Starting a new project
- You want better performance and smaller bundles
- You need advanced features (streaming, parallel routes)
- You want to leverage React Server Components
- You're comfortable with the learning curve

### Incremental Adoption

Good news: You can use both routers simultaneously!

```
my-app/
├── app/           # New routes with App Router
│   └── dashboard/
│       └── page.tsx
└── pages/         # Existing routes with Pages Router
    ├── index.tsx
    └── about.tsx
```

The App Router takes precedence for matching routes.

## Comparison: Next.js vs Plain React

| Feature            | Plain React             | Next.js                     |
| ------------------ | ----------------------- | --------------------------- |
| Routing            | Requires React Router   | Built-in file-based routing |
| SSR                | Manual setup required   | Built-in, configurable      |
| SEO                | More challenging        | Optimized by default        |
| Image Optimization | Manual                  | Automatic                   |
| API Routes         | Separate backend needed | Integrated                  |
| Code Splitting     | Manual optimization     | Automatic                   |
| Configuration      | Extensive setup         | Minimal config              |
| Performance        | Depends on setup        | Optimized out of the box    |

## Learning Resources

- **Official Documentation**: [nextjs.org/docs](https://nextjs.org/docs)
- **Next.js Learn**: Interactive tutorial at [nextjs.org/learn](https://nextjs.org/learn)
- **Examples**: [github.com/vercel/next.js/tree/canary/examples](https://github.com/vercel/next.js/tree/canary/examples)
- **Community**: [GitHub Discussions](https://github.com/vercel/next.js/discussions)

## Next Steps

1. Follow the [official tutorial](https://nextjs.org/learn)
2. Explore the [examples repository](https://github.com/vercel/next.js/tree/canary/examples)
3. Build a small project (blog, portfolio, or todo app)
4. Learn about deployment with Vercel
5. Dive into advanced topics (middleware, internationalization, authentication)

---

## Related Documentation

- [Next.js 16 App Router Routing Guide](file:///c:/Users/mmt20/Desktop/beetleware-frontend-intern/nexjs/app-router-routing-guide.md) - Comprehensive routing guide
- [Next.js 16 Folder Structure](file:///c:/Users/mmt20/Desktop/beetleware-frontend-intern/nexjs/next16-folder-structure.md) - Project organization patterns
- [Rendering & Data Fetching](file:///c:/Users/mmt20/Desktop/beetleware-frontend-intern/nexjs/rendering-and-data-fetching.md) - SSR, SSG, ISR strategies
- [Optimization Guide](file:///c:/Users/mmt20/Desktop/beetleware-frontend-intern/nexjs/optimization-guide.md) - Performance optimization

---

**Pro Tip**: Start with the App Router (not the older Pages Router) for new projects to leverage the latest features and best practices in Next.js 16.

