# Next.js Routing - Complete Guide

## Table of Contents

1. [Introduction](#introduction)
2. [App Router (Next.js 13+)](#app-router-nextjs-13)
3. [File Conventions](#file-conventions)
4. [Dynamic Routes](#dynamic-routes)
5. [Route Groups](#route-groups)
6. [Parallel Routes](#parallel-routes)
7. [Intercepting Routes](#intercepting-routes)
8. [Navigation](#navigation)
9. [Loading UI & Streaming](#loading-ui--streaming)
10. [Error Handling](#error-handling)
11. [Pages Router (Legacy)](#pages-router-legacy)
12. [Best Practices](#best-practices)

---

## Introduction

Next.js provides a **file-system based router** where folders define routes. The structure of your files and folders directly maps to URL paths.

### Key Concepts

- **Folders** define route segments that map to URL segments
- **Files** define the UI shown for a route segment
- Routes are automatically accessible based on their file structure

---

## App Router (Next.js 13+)

The modern routing system built on React Server Components.

### Basic Route Structure

```
app/
├── page.tsx              # /
├── about/
│   └── page.tsx         # /about
├── blog/
│   ├── page.tsx         # /blog
│   └── [slug]/
│       └── page.tsx     # /blog/my-post
└── dashboard/
    ├── page.tsx         # /dashboard
    └── settings/
        └── page.tsx     # /dashboard/settings
```

### Creating Routes

**Root Route (`/`)**

```tsx
// app/page.tsx
export default function HomePage() {
  return <h1>Welcome to Home Page</h1>;
}
```

**Nested Route (`/about`)**

```tsx
// app/about/page.tsx
export default function AboutPage() {
  return <h1>About Us</h1>;
}
```

**Deeply Nested Route (`/blog/categories/tech`)**

```tsx
// app/blog/categories/tech/page.tsx
export default function TechCategoryPage() {
  return <h1>Tech Articles</h1>;
}
```

---

## File Conventions

Next.js uses special file names to create UI with specific behavior:

| File            | Purpose                                   | Example                  |
| --------------- | ----------------------------------------- | ------------------------ |
| `page.tsx`      | Route UI (makes path publicly accessible) | `app/blog/page.tsx`      |
| `layout.tsx`    | Shared UI for segment and children        | `app/blog/layout.tsx`    |
| `template.tsx`  | Re-rendered layout                        | `app/template.tsx`       |
| `loading.tsx`   | Loading UI for segment and children       | `app/blog/loading.tsx`   |
| `error.tsx`     | Error UI for segment and children         | `app/blog/error.tsx`     |
| `not-found.tsx` | Not found UI                              | `app/not-found.tsx`      |
| `route.ts`      | API endpoint                              | `app/api/users/route.ts` |
| `default.tsx`   | Fallback UI for parallel routes           | `app/@modal/default.tsx` |

### Layouts

Layouts wrap multiple pages and persist across route changes.

**Root Layout (Required)**

```tsx
// app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <header>
          <nav>Navigation</nav>
        </header>
        <main>{children}</main>
        <footer>Footer</footer>
      </body>
    </html>
  );
}
```

**Nested Layout**

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="dashboard">
      <aside>
        <nav>Dashboard Menu</nav>
      </aside>
      <section>{children}</section>
    </div>
  );
}
```

**Layout Features:**

- Layouts **don't re-render** on route changes (performance benefit)
- Layouts can fetch data
- Layouts can be nested
- Parent layouts wrap child layouts

### Templates

Templates are similar to layouts but create a new instance on navigation.

```tsx
// app/template.tsx
export default function Template({ children }: { children: React.ReactNode }) {
  return <div className="template-wrapper">{children}</div>;
}
```

**When to use Templates:**

- When you need to re-mount on navigation
- For animations with `framer-motion`
- For resetting component state
- For `useEffect` triggers on navigation

---

## Dynamic Routes

Dynamic segments are created by wrapping folder names in square brackets.

### Single Dynamic Segment

```
app/
└── blog/
    └── [slug]/
        └── page.tsx     # Matches /blog/hello, /blog/world, etc.
```

```tsx
// app/blog/[slug]/page.tsx
export default function BlogPost({ params }: { params: { slug: string } }) {
  return <h1>Post: {params.slug}</h1>;
}

// Generate static params at build time
export async function generateStaticParams() {
  const posts = await getPosts();

  return posts.map((post) => ({
    slug: post.slug,
  }));
}
```

### Multiple Dynamic Segments

```
app/
└── shop/
    └── [category]/
        └── [product]/
            └── page.tsx     # /shop/electronics/laptop
```

```tsx
// app/shop/[category]/[product]/page.tsx
export default function ProductPage({ params }: { params: { category: string; product: string } }) {
  return (
    <div>
      <h1>Category: {params.category}</h1>
      <h2>Product: {params.product}</h2>
    </div>
  );
}
```

### Catch-all Segments

Catch all subsequent segments with `[...folder]`:

```
app/
└── docs/
    └── [...slug]/
        └── page.tsx
```

```tsx
// app/docs/[...slug]/page.tsx
// Matches: /docs/a, /docs/a/b, /docs/a/b/c
export default function DocsPage({ params }: { params: { slug: string[] } }) {
  return (
    <div>
      <h1>Documentation</h1>
      <p>Path: {params.slug.join("/")}</p>
    </div>
  );
}
```

### Optional Catch-all Segments

Make catch-all optional with `[[...folder]]`:

```
app/
└── shop/
    └── [[...category]]/
        └── page.tsx
```

```tsx
// app/shop/[[...category]]/page.tsx
// Matches: /shop, /shop/clothes, /shop/clothes/shirts
export default function ShopPage({ params }: { params: { category?: string[] } }) {
  if (!params.category) {
    return <h1>All Products</h1>;
  }

  return <h1>Category: {params.category.join(" > ")}</h1>;
}
```

---

## Route Groups

Organize routes without affecting the URL structure using parentheses `(folder)`.

```
app/
├── (marketing)/
│   ├── about/
│   │   └── page.tsx        # /about (not /marketing/about)
│   └── contact/
│       └── page.tsx        # /contact
├── (shop)/
│   ├── products/
│   │   └── page.tsx        # /products
│   └── cart/
│       └── page.tsx        # /cart
└── page.tsx                # /
```

### Multiple Root Layouts

Use route groups to create multiple root layouts:

```
app/
├── (dashboard)/
│   ├── layout.tsx          # Dashboard layout
│   └── analytics/
│       └── page.tsx
└── (marketing)/
    ├── layout.tsx          # Marketing layout
    └── home/
        └── page.tsx
```

```tsx
// app/(dashboard)/layout.tsx
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <div className="dashboard-layout">{children}</div>
      </body>
    </html>
  );
}
```

### Use Cases for Route Groups

- **Organization:** Group related routes logically
- **Layouts:** Apply different layouts to different route groups
- **Code splitting:** Separate code by feature
- **Team structure:** Organize by team ownership

---

## Parallel Routes

Load multiple pages in the same layout simultaneously using named slots.

```
app/
├── @team/
│   └── page.tsx
├── @analytics/
│   └── page.tsx
├── layout.tsx
└── page.tsx
```

```tsx
// app/layout.tsx
export default function Layout({
  children,
  team,
  analytics,
}: {
  children: React.ReactNode;
  team: React.ReactNode;
  analytics: React.ReactNode;
}) {
  return (
    <div>
      <div>{children}</div>
      <div className="grid">
        <div>{team}</div>
        <div>{analytics}</div>
      </div>
    </div>
  );
}
```

### Default.tsx for Parallel Routes

Provide fallbacks when slots don't match:

```tsx
// app/@analytics/default.tsx
export default function Default() {
  return <div>Analytics not available</div>;
}
```

### Use Cases

- **Dashboards:** Multiple independent sections
- **Modals:** Show modal alongside page content
- **Split views:** Side-by-side content
- **Tabs:** Different tab contents

---

## Intercepting Routes

Intercept routes to show content in a modal while keeping URL updated.

```
app/
├── photos/
│   ├── [id]/
│   │   └── page.tsx        # /photos/1 (full page)
│   └── page.tsx            # /photos (list)
├── @modal/
│   └── (.)photos/
│       └── [id]/
│           └── page.tsx    # Intercepts /photos/1 in modal
└── layout.tsx
```

### Intercepting Conventions

- `(.)` - match same level
- `(..)` - match one level above
- `(..)(..)` - match two levels above
- `(...)` - match from root

### Example: Photo Gallery with Modal

```tsx
// app/layout.tsx
export default function Layout({ children, modal }: { children: React.ReactNode; modal: React.ReactNode }) {
  return (
    <>
      {children}
      {modal}
    </>
  );
}
```

```tsx
// app/@modal/(.)photos/[id]/page.tsx
import Modal from "@/components/Modal";

export default function PhotoModal({ params }: { params: { id: string } }) {
  return (
    <Modal>
      <img src={`/photos/${params.id}.jpg`} alt="Photo" />
    </Modal>
  );
}
```

```tsx
// app/photos/[id]/page.tsx
export default function PhotoPage({ params }: { params: { id: string } }) {
  return (
    <div>
      <img src={`/photos/${params.id}.jpg`} alt="Photo" />
    </div>
  );
}
```

**Result:**

- Clicking from `/photos` → Opens modal at `/photos/1`
- Refreshing `/photos/1` → Shows full page
- Sharing link → Works correctly

---

## Navigation

### Link Component

The primary way to navigate between routes:

```tsx
import Link from "next/link";

export default function Nav() {
  return (
    <nav>
      <Link href="/">Home</Link>
      <Link href="/about">About</Link>
      <Link href="/blog/hello-world">Blog Post</Link>

      {/* Dynamic href */}
      <Link href={`/blog/${post.slug}`}>Read More</Link>

      {/* With query params */}
      <Link href={{ pathname: "/search", query: { q: "nextjs" } }}>Search</Link>

      {/* Replace instead of push */}
      <Link href="/login" replace>
        Login
      </Link>

      {/* Prefetch disabled */}
      <Link href="/heavy-page" prefetch={false}>
        Heavy Page
      </Link>
    </nav>
  );
}
```

### useRouter Hook

Programmatic navigation:

```tsx
"use client";

import { useRouter } from "next/navigation";

export default function LoginButton() {
  const router = useRouter();

  const handleLogin = async () => {
    await login();
    router.push("/dashboard"); // Navigate to dashboard
  };

  return <button onClick={handleLogin}>Login</button>;
}
```

**Router Methods:**

```tsx
router.push("/dashboard"); // Add to history
router.replace("/dashboard"); // Replace current entry
router.refresh(); // Refresh current route
router.back(); // Go back
router.forward(); // Go forward
router.prefetch("/dashboard"); // Prefetch route
```

### usePathname Hook

Get current pathname:

```tsx
"use client";

import { usePathname } from "next/navigation";
import Link from "next/link";

export default function Nav() {
  const pathname = usePathname();

  return (
    <nav>
      <Link href="/" className={pathname === "/" ? "active" : ""}>
        Home
      </Link>
      <Link href="/about" className={pathname === "/about" ? "active" : ""}>
        About
      </Link>
    </nav>
  );
}
```

### useSearchParams Hook

Access URL search parameters:

```tsx
"use client";

import { useSearchParams } from "next/navigation";

export default function SearchPage() {
  const searchParams = useSearchParams();

  const query = searchParams.get("q");
  const page = searchParams.get("page") || "1";

  return (
    <div>
      <h1>Search Results for: {query}</h1>
      <p>Page: {page}</p>
    </div>
  );
}
```

### redirect Function

Server-side redirects:

```tsx
import { redirect } from "next/navigation";

export default async function ProfilePage() {
  const user = await getUser();

  if (!user) {
    redirect("/login");
  }

  return <div>Welcome {user.name}</div>;
}
```

### permanentRedirect Function

Permanent redirects (308 status):

```tsx
import { permanentRedirect } from "next/navigation";

export default function OldPage() {
  permanentRedirect("/new-page");
}
```

---

## Loading UI & Streaming

### Loading.tsx

Instant loading states with React Suspense:

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <div>Loading dashboard...</div>;
}
```

**How it works:**

1. User navigates to `/dashboard`
2. `loading.tsx` shows immediately
3. Page content streams in when ready
4. Loading UI automatically replaced

### Streaming with Suspense

Granular streaming for specific components:

```tsx
// app/dashboard/page.tsx
import { Suspense } from "react";

export default function DashboardPage() {
  return (
    <div>
      <h1>Dashboard</h1>

      {/* This loads immediately */}
      <UserGreeting />

      {/* These stream independently */}
      <Suspense fallback={<div>Loading revenue...</div>}>
        <RevenueChart />
      </Suspense>

      <Suspense fallback={<div>Loading activity...</div>}>
        <RecentActivity />
      </Suspense>
    </div>
  );
}

async function RevenueChart() {
  const data = await fetchRevenueData(); // Server-side fetch
  return <Chart data={data} />;
}
```

### Loading Skeleton

```tsx
// app/dashboard/loading.tsx
export default function DashboardSkeleton() {
  return (
    <div className="skeleton">
      <div className="skeleton-header" />
      <div className="skeleton-content">
        <div className="skeleton-card" />
        <div className="skeleton-card" />
        <div className="skeleton-card" />
      </div>
    </div>
  );
}
```

---

## Error Handling

### Error.tsx

Automatic error boundaries for route segments:

```tsx
// app/dashboard/error.tsx
"use client"; // Error components must be Client Components

export default function Error({ error, reset }: { error: Error & { digest?: string }; reset: () => void }) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

### Global Error Handler

```tsx
// app/global-error.tsx
"use client";

export default function GlobalError({ error, reset }: { error: Error & { digest?: string }; reset: () => void }) {
  return (
    <html>
      <body>
        <h2>Application Error</h2>
        <button onClick={() => reset()}>Try again</button>
      </body>
    </html>
  );
}
```

### Not Found Pages

```tsx
// app/blog/[slug]/page.tsx
import { notFound } from "next/navigation";

export default async function BlogPost({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug);

  if (!post) {
    notFound();
  }

  return <article>{post.content}</article>;
}
```

```tsx
// app/blog/[slug]/not-found.tsx
export default function NotFound() {
  return (
    <div>
      <h2>Post Not Found</h2>
      <p>Could not find the requested blog post.</p>
    </div>
  );
}
```

---

## Pages Router (Legacy)

The traditional routing system (Next.js 12 and earlier).

### Basic Structure

```
pages/
├── index.tsx              # /
├── about.tsx              # /about
├── blog/
│   ├── index.tsx         # /blog
│   └── [slug].tsx        # /blog/:slug
└── api/
    └── hello.ts          # /api/hello
```

### Creating Pages

```tsx
// pages/index.tsx
export default function Home() {
  return <h1>Home Page</h1>;
}
```

### Dynamic Routes

```tsx
// pages/blog/[slug].tsx
import { useRouter } from "next/router";

export default function BlogPost() {
  const router = useRouter();
  const { slug } = router.query;

  return <h1>Post: {slug}</h1>;
}
```

### Catch-all Routes

```tsx
// pages/docs/[...slug].tsx
import { useRouter } from "next/router";

export default function Docs() {
  const router = useRouter();
  const { slug } = router.query; // ['getting-started', 'installation']

  return <div>Docs: {slug?.join("/")}</div>;
}
```

### Navigation

```tsx
import Link from "next/link";
import { useRouter } from "next/router";

export default function Navigation() {
  const router = useRouter();

  return (
    <div>
      <Link href="/">Home</Link>
      <Link href="/about">About</Link>

      <button onClick={() => router.push("/dashboard")}>Go to Dashboard</button>
    </div>
  );
}
```

---

## Best Practices

### 1. **Use App Router for New Projects**

Start with the App Router to leverage Server Components and modern features.

### 2. **Organize with Route Groups**

```
app/
├── (marketing)/
│   ├── home/
│   └── about/
├── (app)/
│   ├── dashboard/
│   └── profile/
└── (auth)/
    ├── login/
    └── register/
```

### 3. **Colocate Components**

Keep related components close to routes:

```
app/
└── dashboard/
    ├── page.tsx
    ├── layout.tsx
    ├── components/
    │   ├── Header.tsx
    │   └── Sidebar.tsx
    └── utils/
        └── helpers.ts
```

### 4. **Loading States**

Always provide loading states for better UX:

```tsx
// app/products/loading.tsx
export default function Loading() {
  return <ProductsSkeleton />;
}
```

### 5. **Error Boundaries**

Handle errors gracefully at appropriate levels:

```
app/
├── error.tsx              # Catches all errors
└── dashboard/
    ├── error.tsx         # Catches dashboard errors
    └── settings/
        └── error.tsx     # Catches settings errors
```

### 6. **Metadata for SEO**

```tsx
// app/blog/[slug]/page.tsx
export async function generateMetadata({ params }) {
  const post = await getPost(params.slug);

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [post.coverImage],
    },
  };
}
```

### 7. **Prefetch Important Routes**

```tsx
<Link href="/dashboard" prefetch={true}>
  Dashboard
</Link>
```

### 8. **Use Proper Status Codes**

```tsx
// 404
notFound();

// 301 Permanent Redirect
permanentRedirect("/new-url");

// 302 Temporary Redirect
redirect("/temporary-url");
```

### 9. **Validate Dynamic Parameters**

```tsx
export default async function Page({ params }: { params: { id: string } }) {
  const id = parseInt(params.id);

  if (isNaN(id)) {
    notFound();
  }

  // Continue with valid id
}
```

### 10. **Private Folders**

Prefix folders with `_` to exclude them from routing:

```
app/
├── _components/       # Not a route
│   └── Button.tsx
├── _lib/             # Not a route
│   └── utils.ts
└── page.tsx          # /
```

---

## Route Matching Priority

When multiple routes could match, Next.js follows this priority:

1. **Static routes** - `/about`
2. **Dynamic routes** - `/blog/[slug]`
3. **Catch-all routes** - `/docs/[...slug]`
4. **Optional catch-all** - `/shop/[[...category]]`

**Example:**

```
app/
├── products/
│   ├── new/
│   │   └── page.tsx        # Priority 1: /products/new
│   ├── [id]/
│   │   └── page.tsx        # Priority 2: /products/123
│   └── [...slug]/
│       └── page.tsx        # Priority 3: /products/a/b/c
```

---

## Advanced Patterns

### Protected Routes

```tsx
// app/dashboard/layout.tsx
import { redirect } from "next/navigation";
import { getSession } from "@/lib/auth";

export default async function DashboardLayout({ children }: { children: React.ReactNode }) {
  const session = await getSession();

  if (!session) {
    redirect("/login");
  }

  return <div className="dashboard">{children}</div>;
}
```

### Middleware for Routing

```tsx
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  // Redirect /old-blog to /blog
  if (request.nextUrl.pathname.startsWith("/old-blog")) {
    return NextResponse.redirect(new URL("/blog", request.url));
  }

  // Authentication check
  const token = request.cookies.get("token");
  if (!token && request.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ["/old-blog/:path*", "/dashboard/:path*"],
};
```

### Internationalization (i18n)

```
app/
├── [lang]/
│   ├── page.tsx          # /:lang
│   ├── about/
│   │   └── page.tsx      # /:lang/about
│   └── blog/
│       └── page.tsx      # /:lang/blog
```

```tsx
// app/[lang]/page.tsx
export default function Page({ params }: { params: { lang: string } }) {
  return <h1>{params.lang === "en" ? "Hello" : "Hola"}</h1>;
}

export async function generateStaticParams() {
  return [{ lang: "en" }, { lang: "es" }, { lang: "fr" }];
}
```

---

## Summary

### Key Takeaways

✅ **File-system based routing** - Folders and files define routes
✅ **Multiple rendering options** - SSR, SSG, ISR, Client-side
✅ **Built-in optimizations** - Prefetching, code splitting, streaming
✅ **Flexible patterns** - Dynamic routes, parallel routes, intercepting routes
✅ **Great DX** - Loading states, error handling, TypeScript support

### Quick Reference

| Need              | Solution                                  |
| ----------------- | ----------------------------------------- |
| Create a route    | Add `page.tsx` in a folder                |
| Dynamic parameter | Use `[param]` folder                      |
| Shared layout     | Add `layout.tsx`                          |
| Loading state     | Add `loading.tsx`                         |
| Error handling    | Add `error.tsx`                           |
| 404 page          | Add `not-found.tsx`                       |
| Navigate          | Use `<Link>` or `useRouter()`             |
| Redirect          | Use `redirect()` or `permanentRedirect()` |
| Organize routes   | Use route groups `(folder)`               |
| API endpoint      | Add `route.ts`                            |

---

## Resources

- **Official Docs:** [nextjs.org/docs/app/building-your-application/routing](https://nextjs.org/docs/app/building-your-application/routing)
- **Routing Examples:** [github.com/vercel/next.js/tree/canary/examples](https://github.com/vercel/next.js/tree/canary/examples)
- **App Router Playground:** [app-router.vercel.app](https://app-router.vercel.app)
