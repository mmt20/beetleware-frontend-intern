# Next.js 16 App Router - Complete Routing Guide

## Table of Contents

1. [Introduction](#introduction)
2. [Static Routes](#static-routes)
3. [Nested Routes](#nested-routes)
4. [Dynamic Routes](#dynamic-routes)
5. [Route Groups](#route-groups)
6. [Layouts](#layouts)
7. [Navigation](#navigation)
8. [Loading.js File](#loadingjs-file)
9. [Error.js File](#errorjs-file)
10. [Parallel Routes](#parallel-routes)
11. [Intercepting Routes](#intercepting-routes)
12. [Route Handlers](#route-handlers)
13. [Best Practices](#best-practices)

---

## Introduction

Next.js 16 App Router introduces a file-system based routing mechanism built on React Server Components. The `app` directory provides a powerful and intuitive way to structure your application.

### Key Concepts

- **File-based routing** - Folders define routes
- **Server Components by default** - Better performance
- **Layouts** - Shared UI between routes
- **Loading & Error states** - Built-in UI states
- **Parallel & Intercepting Routes** - Advanced patterns

### App Directory Structure

```
app/
├── layout.tsx          # Root layout
├── page.tsx            # Home page (/)
├── loading.tsx         # Loading UI
├── error.tsx           # Error UI
├── not-found.tsx       # 404 UI
├── about/
│   └── page.tsx        # /about
├── blog/
│   ├── layout.tsx      # Blog layout
│   ├── page.tsx        # /blog
│   └── [slug]/
│       └── page.tsx    # /blog/[slug]
└── api/
    └── users/
        └── route.ts    # API route
```

---

## Static Routes

Static routes are defined by creating a `page.tsx` file inside a folder.

### Basic Static Route

```tsx
// app/page.tsx
export default function HomePage() {
  return (
    <div>
      <h1>Welcome to Home Page</h1>
      <p>This is the root route (/)</p>
    </div>
  );
}
```

### About Page

```tsx
// app/about/page.tsx
export default function AboutPage() {
  return (
    <div>
      <h1>About Us</h1>
      <p>Learn more about our company</p>
    </div>
  );
}
```

**URL:** `/about`

### Contact Page

```tsx
// app/contact/page.tsx
export default function ContactPage() {
  return (
    <div>
      <h1>Contact Us</h1>
      <form>
        <input type="email" placeholder="Your email" />
        <textarea placeholder="Your message" />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

**URL:** `/contact`

### Multiple Static Routes

```
app/
├── page.tsx           # /
├── about/
│   └── page.tsx       # /about
├── services/
│   └── page.tsx       # /services
├── pricing/
│   └── page.tsx       # /pricing
└── contact/
    └── page.tsx       # /contact
```

---

## Nested Routes

Nested routes are created by nesting folders within folders.

### Basic Nested Routes

```tsx
// app/blog/page.tsx
export default function BlogPage() {
  return (
    <div>
      <h1>Blog</h1>
      <p>Welcome to our blog</p>
    </div>
  );
}
```

**URL:** `/blog`

```tsx
// app/blog/first-post/page.tsx
export default function FirstPostPage() {
  return (
    <div>
      <h1>First Post</h1>
      <p>This is our first blog post</p>
    </div>
  );
}
```

**URL:** `/blog/first-post`

### Deep Nesting

```
app/
└── dashboard/
    ├── page.tsx              # /dashboard
    ├── settings/
    │   ├── page.tsx          # /dashboard/settings
    │   ├── profile/
    │   │   └── page.tsx      # /dashboard/settings/profile
    │   └── security/
    │       └── page.tsx      # /dashboard/settings/security
    └── analytics/
        └── page.tsx          # /dashboard/analytics
```

```tsx
// app/dashboard/settings/profile/page.tsx
export default function ProfileSettingsPage() {
  return (
    <div>
      <h1>Profile Settings</h1>
      <p>Update your profile information</p>
    </div>
  );
}
```

**URL:** `/dashboard/settings/profile`

### Nested Routes with Shared Layout

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="dashboard">
      <aside className="sidebar">
        <nav>
          <a href="/dashboard">Overview</a>
          <a href="/dashboard/settings">Settings</a>
          <a href="/dashboard/analytics">Analytics</a>
        </nav>
      </aside>
      <main className="content">{children}</main>
    </div>
  );
}
```

---

## Dynamic Routes

Dynamic routes use square brackets `[param]` to create route segments with dynamic parameters.

### Basic Dynamic Route

```tsx
// app/blog/[slug]/page.tsx
export default function BlogPost({ params }: { params: { slug: string } }) {
  return (
    <div>
      <h1>Blog Post: {params.slug}</h1>
      <p>You are viewing: {params.slug}</p>
    </div>
  );
}
```

**URLs:**

- `/blog/hello-world` → `{ slug: 'hello-world' }`
- `/blog/nextjs-guide` → `{ slug: 'nextjs-guide' }`

### Async Server Component with Dynamic Route

```tsx
// app/blog/[slug]/page.tsx
interface Post {
  id: string;
  title: string;
  content: string;
  author: string;
}

async function getPost(slug: string): Promise<Post> {
  const res = await fetch(`https://api.example.com/posts/${slug}`, {
    next: { revalidate: 3600 }, // Cache for 1 hour
  });

  if (!res.ok) {
    throw new Error("Failed to fetch post");
  }

  return res.json();
}

export default async function BlogPost({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug);

  return (
    <article>
      <h1>{post.title}</h1>
      <p>By {post.author}</p>
      <div>{post.content}</div>
    </article>
  );
}
```

### Multiple Dynamic Segments

```tsx
// app/shop/[category]/[productId]/page.tsx
export default function ProductPage({ params }: { params: { category: string; productId: string } }) {
  return (
    <div>
      <h1>Product Details</h1>
      <p>Category: {params.category}</p>
      <p>Product ID: {params.productId}</p>
    </div>
  );
}
```

**URLs:**

- `/shop/electronics/laptop-123` → `{ category: 'electronics', productId: 'laptop-123' }`
- `/shop/clothing/shirt-456` → `{ category: 'clothing', productId: 'shirt-456' }`

### Catch-all Routes

Use `[...param]` to catch all route segments.

```tsx
// app/docs/[...slug]/page.tsx
export default function DocsPage({ params }: { params: { slug: string[] } }) {
  return (
    <div>
      <h1>Documentation</h1>
      <p>Path segments: {params.slug.join(" / ")}</p>
    </div>
  );
}
```

**URLs:**

- `/docs/getting-started` → `{ slug: ['getting-started'] }`
- `/docs/api/authentication` → `{ slug: ['api', 'authentication'] }`
- `/docs/guides/deployment/vercel` → `{ slug: ['guides', 'deployment', 'vercel'] }`

### Optional Catch-all Routes

Use `[[...param]]` to make catch-all routes optional.

```tsx
// app/shop/[[...slug]]/page.tsx
export default function ShopPage({ params }: { params: { slug?: string[] } }) {
  if (!params.slug) {
    return <div>Welcome to Shop Home</div>;
  }

  return (
    <div>
      <h1>Shop</h1>
      <p>Browsing: {params.slug.join(" / ")}</p>
    </div>
  );
}
```

**URLs:**

- `/shop` → `{ slug: undefined }`
- `/shop/electronics` → `{ slug: ['electronics'] }`
- `/shop/electronics/laptops` → `{ slug: ['electronics', 'laptops'] }`

### generateStaticParams for Static Generation

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await fetch("https://api.example.com/posts").then((res) => res.json());

  return posts.map((post: { slug: string }) => ({
    slug: post.slug,
  }));
}

export default async function BlogPost({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug);
  return <article>{/* Post content */}</article>;
}
```

### Dynamic Metadata

```tsx
// app/blog/[slug]/page.tsx
import { Metadata } from "next";

export async function generateMetadata({ params }: { params: { slug: string } }): Promise<Metadata> {
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

export default async function BlogPost({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug);
  return <article>{/* Post content */}</article>;
}
```

---

## Route Groups

Route groups allow you to organize routes without affecting the URL structure. Create a route group by wrapping a folder name in parentheses `(name)`.

### Basic Route Group

```
app/
├── (marketing)/
│   ├── about/
│   │   └── page.tsx       # /about
│   ├── contact/
│   │   └── page.tsx       # /contact
│   └── layout.tsx         # Marketing layout
└── (shop)/
    ├── products/
    │   └── page.tsx       # /products
    ├── cart/
    │   └── page.tsx       # /cart
    └── layout.tsx         # Shop layout
```

### Marketing Layout

```tsx
// app/(marketing)/layout.tsx
export default function MarketingLayout({ children }: { children: React.ReactNode }) {
  return (
    <div>
      <header className="marketing-header">
        <nav>
          <a href="/">Home</a>
          <a href="/about">About</a>
          <a href="/contact">Contact</a>
        </nav>
      </header>
      <main>{children}</main>
      <footer className="marketing-footer">
        <p>&copy; 2024 Company Name</p>
      </footer>
    </div>
  );
}
```

### Shop Layout

```tsx
// app/(shop)/layout.tsx
export default function ShopLayout({ children }: { children: React.ReactNode }) {
  return (
    <div>
      <header className="shop-header">
        <nav>
          <a href="/products">Products</a>
          <a href="/cart">Cart</a>
          <a href="/checkout">Checkout</a>
        </nav>
      </header>
      <main>{children}</main>
    </div>
  );
}
```

### Authentication Route Groups

```
app/
├── (auth)/
│   ├── login/
│   │   └── page.tsx       # /login
│   ├── register/
│   │   └── page.tsx       # /register
│   └── layout.tsx         # Auth layout (centered form)
└── (dashboard)/
    ├── overview/
    │   └── page.tsx       # /overview
    ├── settings/
    │   └── page.tsx       # /settings
    └── layout.tsx         # Dashboard layout (sidebar)
```

```tsx
// app/(auth)/layout.tsx
export default function AuthLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="auth-container">
      <div className="auth-card">{children}</div>
    </div>
  );
}
```

### Multiple Root Layouts

```
app/
├── (public)/
│   ├── layout.tsx         # Public root layout
│   ├── page.tsx           # /
│   └── about/
│       └── page.tsx       # /about
└── (admin)/
    ├── layout.tsx         # Admin root layout
    ├── dashboard/
    │   └── page.tsx       # /dashboard
    └── users/
        └── page.tsx       # /users
```

```tsx
// app/(public)/layout.tsx
export default function PublicLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body className="public-theme">
        <nav>Public Navigation</nav>
        {children}
      </body>
    </html>
  );
}
```

```tsx
// app/(admin)/layout.tsx
export default function AdminLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body className="admin-theme">
        <nav>Admin Navigation</nav>
        {children}
      </body>
    </html>
  );
}
```

---

## Layouts

Layouts are UI that is shared between multiple routes. They preserve state, remain interactive, and don't re-render.

### Root Layout (Required)

```tsx
// app/layout.tsx
import "./globals.css";
import { Inter } from "next/font/google";

const inter = Inter({ subsets: ["latin"] });

export const metadata = {
  title: "My App",
  description: "Created with Next.js 16",
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <header>
          <nav>
            <a href="/">Home</a>
            <a href="/about">About</a>
            <a href="/blog">Blog</a>
          </nav>
        </header>
        {children}
        <footer>
          <p>&copy; 2024 My App</p>
        </footer>
      </body>
    </html>
  );
}
```

### Nested Layout

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="dashboard-container">
      <aside className="sidebar">
        <h2>Dashboard</h2>
        <ul>
          <li>
            <a href="/dashboard">Overview</a>
          </li>
          <li>
            <a href="/dashboard/analytics">Analytics</a>
          </li>
          <li>
            <a href="/dashboard/settings">Settings</a>
          </li>
        </ul>
      </aside>
      <main className="dashboard-content">{children}</main>
    </div>
  );
}
```

### Layout Composition

Layouts nest automatically. A page uses all layouts from its route hierarchy.

```
app/
├── layout.tsx                    # Root Layout
└── dashboard/
    ├── layout.tsx                # Dashboard Layout
    └── settings/
        ├── layout.tsx            # Settings Layout
        └── profile/
            └── page.tsx          # Uses all 3 layouts
```

```tsx
// app/dashboard/settings/layout.tsx
export default function SettingsLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="settings-container">
      <nav className="settings-nav">
        <a href="/dashboard/settings/profile">Profile</a>
        <a href="/dashboard/settings/account">Account</a>
        <a href="/dashboard/settings/security">Security</a>
      </nav>
      <div className="settings-content">{children}</div>
    </div>
  );
}
```

### Layout with Data Fetching

```tsx
// app/dashboard/layout.tsx
async function getUser() {
  const res = await fetch("https://api.example.com/user", {
    cache: "no-store", // Always fetch fresh data
  });
  return res.json();
}

export default async function DashboardLayout({ children }: { children: React.ReactNode }) {
  const user = await getUser();

  return (
    <div>
      <header>
        <h1>Welcome, {user.name}</h1>
      </header>
      {children}
    </div>
  );
}
```

### Template (Re-renders on Navigation)

Unlike layouts, templates create a new instance on navigation.

```tsx
// app/template.tsx
"use client";

import { useEffect } from "react";

export default function Template({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    console.log("Template mounted - runs on every navigation");
  }, []);

  return <div className="template-wrapper">{children}</div>;
}
```

**Differences:**

- **Layout:** Persists across navigations, maintains state
- **Template:** Creates new instance on each navigation, useful for animations

---

## Navigation

Next.js provides several ways to navigate between routes.

### Link Component

The primary way to navigate between routes.

```tsx
// app/page.tsx
import Link from "next/link";

export default function HomePage() {
  return (
    <div>
      <h1>Home Page</h1>

      {/* Basic link */}
      <Link href="/about">About Us</Link>

      {/* Link with custom styling */}
      <Link href="/blog" className="btn btn-primary">
        Read Blog
      </Link>

      {/* Dynamic link */}
      <Link href="/blog/my-first-post">First Post</Link>

      {/* Link with replace (replaces history instead of push) */}
      <Link href="/login" replace>
        Login
      </Link>

      {/* Prefetch disabled (default is true) */}
      <Link href="/heavy-page" prefetch={false}>
        Heavy Page
      </Link>
    </div>
  );
}
```

### Active Link

```tsx
"use client";

import Link from "next/link";
import { usePathname } from "next/navigation";

export default function Navigation() {
  const pathname = usePathname();

  return (
    <nav>
      <Link href="/" className={pathname === "/" ? "active" : ""}>
        Home
      </Link>
      <Link href="/about" className={pathname === "/about" ? "active" : ""}>
        About
      </Link>
      <Link href="/blog" className={pathname.startsWith("/blog") ? "active" : ""}>
        Blog
      </Link>
    </nav>
  );
}
```

### useRouter Hook

For programmatic navigation in Client Components.

```tsx
"use client";

import { useRouter } from "next/navigation";

export default function LoginForm() {
  const router = useRouter();

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();

    // Perform login
    const success = await login();

    if (success) {
      // Navigate to dashboard
      router.push("/dashboard");
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input type="email" />
      <input type="password" />
      <button type="submit">Login</button>
    </form>
  );
}
```

### Router Methods

```tsx
"use client";

import { useRouter } from "next/navigation";

export default function NavigationExamples() {
  const router = useRouter();

  return (
    <div>
      {/* Push new entry to history */}
      <button onClick={() => router.push("/dashboard")}>Go to Dashboard</button>

      {/* Replace current history entry */}
      <button onClick={() => router.replace("/login")}>Replace with Login</button>

      {/* Go back */}
      <button onClick={() => router.back()}>Go Back</button>

      {/* Go forward */}
      <button onClick={() => router.forward()}>Go Forward</button>

      {/* Refresh current route */}
      <button onClick={() => router.refresh()}>Refresh</button>

      {/* Prefetch route */}
      <button onClick={() => router.prefetch("/products")}>Prefetch Products</button>
    </div>
  );
}
```

### usePathname Hook

```tsx
"use client";

import { usePathname } from "next/navigation";

export default function CurrentPath() {
  const pathname = usePathname();

  return (
    <div>
      <p>Current path: {pathname}</p>
      {pathname === "/blog" && <p>You are on the blog</p>}
    </div>
  );
}
```

### useSearchParams Hook

```tsx
"use client";

import { useSearchParams } from "next/navigation";

export default function SearchResults() {
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

**URL:** `/search?q=nextjs&page=2`

### redirect Function

Server-side redirect in Server Components.

```tsx
// app/profile/page.tsx
import { redirect } from "next/navigation";
import { getUser } from "@/lib/auth";

export default async function ProfilePage() {
  const user = await getUser();

  if (!user) {
    redirect("/login");
  }

  return (
    <div>
      <h1>Profile: {user.name}</h1>
    </div>
  );
}
```

### permanentRedirect Function

```tsx
// app/old-page/page.tsx
import { permanentRedirect } from "next/navigation";

export default function OldPage() {
  permanentRedirect("/new-page");
}
```

### Navigation Events

```tsx
"use client";

import { usePathname, useSearchParams } from "next/navigation";
import { useEffect } from "react";

export default function NavigationEvents() {
  const pathname = usePathname();
  const searchParams = useSearchParams();

  useEffect(() => {
    const url = `${pathname}?${searchParams}`;
    console.log("Navigation to:", url);

    // Track page view
    gtag("event", "page_view", { page_path: url });
  }, [pathname, searchParams]);

  return null;
}
```

---

## Loading.js File

The `loading.js` file creates loading UI that is shown while a route segment is loading.

### Basic Loading UI

```tsx
// app/loading.tsx
export default function Loading() {
  return (
    <div className="loading-container">
      <div className="spinner" />
      <p>Loading...</p>
    </div>
  );
}
```

### Loading for Specific Route

```tsx
// app/dashboard/loading.tsx
export default function DashboardLoading() {
  return (
    <div className="dashboard-skeleton">
      <div className="skeleton-header" />
      <div className="skeleton-content" />
      <div className="skeleton-sidebar" />
    </div>
  );
}
```

### Loading with Skeleton

```tsx
// app/blog/loading.tsx
export default function BlogLoading() {
  return (
    <div className="blog-loading">
      {[1, 2, 3, 4].map((i) => (
        <div key={i} className="skeleton-card">
          <div className="skeleton-image" />
          <div className="skeleton-title" />
          <div className="skeleton-text" />
          <div className="skeleton-text" />
        </div>
      ))}
    </div>
  );
}
```

### Instant Loading States with Suspense

```tsx
// app/blog/page.tsx
import { Suspense } from "react";

function PostList() {
  // This component fetches data
  const posts = await getPosts();
  return (
    <div>
      {posts.map((post) => (
        <article key={post.id}>{post.title}</article>
      ))}
    </div>
  );
}

function PostListSkeleton() {
  return (
    <div className="skeleton-list">
      {[1, 2, 3].map((i) => (
        <div key={i} className="skeleton-item" />
      ))}
    </div>
  );
}

export default function BlogPage() {
  return (
    <div>
      <h1>Blog</h1>
      <Suspense fallback={<PostListSkeleton />}>
        <PostList />
      </Suspense>
    </div>
  );
}
```

### Multiple Loading States

```tsx
// app/dashboard/page.tsx
import { Suspense } from "react";

async function RecentSales() {
  const sales = await getSales();
  return <div>{/* Sales content */}</div>;
}

async function Analytics() {
  const analytics = await getAnalytics();
  return <div>{/* Analytics content */}</div>;
}

function LoadingCard() {
  return <div className="skeleton-card" />;
}

export default function DashboardPage() {
  return (
    <div className="dashboard-grid">
      <Suspense fallback={<LoadingCard />}>
        <RecentSales />
      </Suspense>

      <Suspense fallback={<LoadingCard />}>
        <Analytics />
      </Suspense>
    </div>
  );
}
```

### Loading with Streaming

```tsx
// app/page.tsx
import { Suspense } from "react";

async function SlowComponent() {
  await new Promise((resolve) => setTimeout(resolve, 3000));
  return <div>Slow content loaded!</div>;
}

export default function Page() {
  return (
    <div>
      <h1>Fast content renders immediately</h1>

      <Suspense fallback={<div>Loading slow content...</div>}>
        <SlowComponent />
      </Suspense>
    </div>
  );
}
```

### Custom Loading Component

```tsx
// components/LoadingSpinner.tsx
export default function LoadingSpinner({ text = "Loading..." }) {
  return (
    <div className="flex items-center justify-center min-h-screen">
      <div className="text-center">
        <div className="animate-spin rounded-full h-32 w-32 border-b-2 border-gray-900" />
        <p className="mt-4 text-gray-600">{text}</p>
      </div>
    </div>
  );
}
```

```tsx
// app/products/loading.tsx
import LoadingSpinner from "@/components/LoadingSpinner";

export default function Loading() {
  return <LoadingSpinner text="Loading products..." />;
}
```

---

## Error.js File

The `error.js` file creates error UI for route segments and automatically wraps components in a React Error Boundary.

### Basic Error UI

```tsx
// app/error.tsx
"use client";

export default function Error({ error, reset }: { error: Error & { digest?: string }; reset: () => void }) {
  return (
    <div className="error-container">
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

### Error with Logging

```tsx
// app/error.tsx
"use client";

import { useEffect } from "react";

export default function Error({ error, reset }: { error: Error & { digest?: string }; reset: () => void }) {
  useEffect(() => {
    // Log error to error reporting service
    console.error("Error:", error);

    // Send to analytics
    if (typeof window !== "undefined") {
      window.gtag?.("event", "exception", {
        description: error.message,
        fatal: false,
      });
    }
  }, [error]);

  return (
    <div className="min-h-screen flex items-center justify-center">
      <div className="text-center">
        <h1 className="text-4xl font-bold text-red-600">Oops!</h1>
        <h2 className="text-2xl mt-4">Something went wrong</h2>
        <p className="mt-2 text-gray-600">{error.message}</p>
        <button onClick={reset} className="mt-6 px-4 py-2 bg-blue-500 text-white rounded">
          Try again
        </button>
      </div>
    </div>
  );
}
```

### Nested Error Boundaries

```tsx
// app/dashboard/error.tsx
"use client";

export default function DashboardError({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div className="dashboard-error">
      <h2>Dashboard Error</h2>
      <p>Failed to load dashboard: {error.message}</p>
      <button onClick={reset}>Reload Dashboard</button>
    </div>
  );
}
```

```tsx
// app/dashboard/settings/error.tsx
"use client";

export default function SettingsError({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div className="settings-error">
      <h2>Settings Error</h2>
      <p>Failed to load settings: {error.message}</p>
      <button onClick={reset}>Reload Settings</button>
    </div>
  );
}
```

### Global Error (app/global-error.tsx)

Handles errors in the root layout.

```tsx
// app/global-error.tsx
"use client";

export default function GlobalError({ error, reset }: { error: Error & { digest?: string }; reset: () => void }) {
  return (
    <html>
      <body>
        <h2>Global Error!</h2>
        <p>{error.message}</p>
        <button onClick={() => reset()}>Try again</button>
      </body>
    </html>
  );
}
```

### Not Found Error

```tsx
// app/not-found.tsx
import Link from "next/link";

export default function NotFound() {
  return (
    <div className="min-h-screen flex items-center justify-center">
      <div className="text-center">
        <h1 className="text-6xl font-bold">404</h1>
        <h2 className="text-2xl mt-4">Page Not Found</h2>
        <p className="mt-2 text-gray-600">The page you're looking for doesn't exist.</p>
        <Link href="/" className="mt-6 inline-block px-4 py-2 bg-blue-500 text-white rounded">
          Go Home
        </Link>
      </div>
    </div>
  );
}
```

### Trigger Not Found

```tsx
// app/blog/[slug]/page.tsx
import { notFound } from "next/navigation";

async function getPost(slug: string) {
  const res = await fetch(`https://api.example.com/posts/${slug}`);

  if (!res.ok) {
    return null;
  }

  return res.json();
}

export default async function BlogPost({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug);

  if (!post) {
    notFound(); // Triggers not-found.tsx
  }

  return <article>{post.title}</article>;
}
```

### Custom Error Types

```tsx
// app/api/error.tsx
"use client";

export default function APIError({ error, reset }: { error: Error & { statusCode?: number }; reset: () => void }) {
  const errorMessage =
    error.statusCode === 404
      ? "API endpoint not found"
      : error.statusCode === 500
      ? "Internal server error"
      : "API request failed";

  return (
    <div>
      <h2>API Error ({error.statusCode})</h2>
      <p>{errorMessage}</p>
      <button onClick={reset}>Retry</button>
    </div>
  );
}
```

---

## Parallel Routes

Parallel Routes allow you to simultaneously render multiple pages in the same layout using named slots.

### Basic Parallel Routes

```
app/
└── dashboard/
    ├── layout.tsx
    ├── page.tsx
    ├── @analytics/
    │   └── page.tsx
    └── @team/
        └── page.tsx
```

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
  analytics,
  team,
}: {
  children: React.ReactNode;
  analytics: React.ReactNode;
  team: React.ReactNode;
}) {
  return (
    <div className="dashboard">
      <div className="main">{children}</div>
      <div className="analytics">{analytics}</div>
      <div className="team">{team}</div>
    </div>
  );
}
```

```tsx
// app/dashboard/@analytics/page.tsx
export default function Analytics() {
  return (
    <div>
      <h2>Analytics</h2>
      <p>User analytics data</p>
    </div>
  );
}
```

```tsx
// app/dashboard/@team/page.tsx
export default function Team() {
  return (
    <div>
      <h2>Team</h2>
      <p>Team member list</p>
    </div>
  );
}
```

### Conditional Rendering with Parallel Routes

```tsx
// app/dashboard/layout.tsx
import { getUser } from "@/lib/auth";

export default async function DashboardLayout({
  children,
  analytics,
  team,
  admin,
}: {
  children: React.ReactNode;
  analytics: React.ReactNode;
  team: React.ReactNode;
  admin: React.ReactNode;
}) {
  const user = await getUser();

  return (
    <div>
      <main>{children}</main>
      <aside>
        {analytics}
        {team}
        {user.role === "admin" && admin}
      </aside>
    </div>
  );
}
```

### Parallel Routes with Loading States

```
app/
└── dashboard/
    ├── @analytics/
    │   ├── page.tsx
    │   └── loading.tsx
    └── @team/
        ├── page.tsx
        └── loading.tsx
```

```tsx
// app/dashboard/@analytics/loading.tsx
export default function AnalyticsLoading() {
  return <div>Loading analytics...</div>;
}
```

### Default Parallel Route

```tsx
// app/dashboard/@analytics/default.tsx
export default function Default() {
  return null; // Or return a default component
}
```

### Tab-Based Navigation with Parallel Routes

```
app/
└── settings/
    ├── layout.tsx
    ├── @profile/
    │   └── page.tsx
    ├── @security/
    │   └── page.tsx
    └── @notifications/
        └── page.tsx
```

```tsx
// app/settings/layout.tsx
"use client";

import { useState } from "react";

export default function SettingsLayout({
  profile,
  security,
  notifications,
}: {
  profile: React.ReactNode;
  security: React.ReactNode;
  notifications: React.ReactNode;
}) {
  const [activeTab, setActiveTab] = useState("profile");

  return (
    <div>
      <nav>
        <button onClick={() => setActiveTab("profile")}>Profile</button>
        <button onClick={() => setActiveTab("security")}>Security</button>
        <button onClick={() => setActiveTab("notifications")}>Notifications</button>
      </nav>

      <div>
        {activeTab === "profile" && profile}
        {activeTab === "security" && security}
        {activeTab === "notifications" && notifications}
      </div>
    </div>
  );
}
```

### Parallel Routes with Different Data

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
  revenue,
  users,
}: {
  children: React.ReactNode;
  revenue: React.ReactNode;
  users: React.ReactNode;
}) {
  return (
    <div className="dashboard-grid">
      <main className="col-span-2">{children}</main>
      <aside className="space-y-4">
        {revenue}
        {users}
      </aside>
    </div>
  );
}
```

```tsx
// app/dashboard/@revenue/page.tsx
async function getRevenue() {
  const res = await fetch("https://api.example.com/revenue");
  return res.json();
}

export default async function Revenue() {
  const revenue = await getRevenue();
  return (
    <div className="card">
      <h2>Revenue</h2>
      <p>${revenue.total}</p>
    </div>
  );
}
```

---

## Intercepting Routes

Intercepting Routes allow you to load a route from another part of your application within the current layout.

### Basic Intercepting Route

Use `(..)` notation to intercept routes:

- `(.)` - match same level
- `(..)` - match one level above
- `(..)(..)` - match two levels above
- `(...)` - match from root

```
app/
├── feed/
│   └── page.tsx
└── photo/
    ├── [id]/
    │   └── page.tsx
    └── (..)photo/
        └── [id]/
            └── page.tsx
```

### Modal with Intercepting Route

```tsx
// app/photo/(..)photo/[id]/page.tsx
import Modal from "@/components/Modal";
import Image from "next/image";

export default function PhotoModal({ params }: { params: { id: string } }) {
  return (
    <Modal>
      <Image src={`/photos/${params.id}.jpg`} alt="Photo" width={800} height={600} />
    </Modal>
  );
}
```

```tsx
// components/Modal.tsx
"use client";

import { useRouter } from "next/navigation";
import { useEffect, useRef } from "react";

export default function Modal({ children }: { children: React.ReactNode }) {
  const router = useRouter();
  const dialogRef = useRef<HTMLDialogElement>(null);

  useEffect(() => {
    dialogRef.current?.showModal();
  }, []);

  function onDismiss() {
    router.back();
  }

  return (
    <dialog ref={dialogRef} className="modal" onClose={onDismiss}>
      <button onClick={onDismiss}>Close</button>
      {children}
    </dialog>
  );
}
```

### Gallery with Intercepting Routes

```
app/
├── gallery/
│   ├── page.tsx
│   └── (..)photo/
│       └── [id]/
│           └── page.tsx
└── photo/
    └── [id]/
        └── page.tsx
```

```tsx
// app/gallery/page.tsx
import Link from "next/link";

export default function Gallery() {
  const photos = [1, 2, 3, 4, 5, 6];

  return (
    <div className="gallery-grid">
      {photos.map((id) => (
        <Link key={id} href={`/photo/${id}`}>
          <img src={`/photos/${id}.jpg`} alt={`Photo ${id}`} />
        </Link>
      ))}
    </div>
  );
}
```

```tsx
// app/photo/[id]/page.tsx - Full page view
export default function PhotoPage({ params }: { params: { id: string } }) {
  return (
    <div className="full-page">
      <h1>Photo {params.id}</h1>
      <img src={`/photos/${params.id}.jpg`} alt={`Photo ${params.id}`} />
    </div>
  );
}
```

```tsx
// app/gallery/(..)photo/[id]/page.tsx - Modal view
import Modal from "@/components/Modal";

export default function PhotoModal({ params }: { params: { id: string } }) {
  return (
    <Modal>
      <img src={`/photos/${params.id}.jpg`} alt={`Photo ${params.id}`} />
    </Modal>
  );
}
```

### Login Modal with Intercepting Route

```
app/
├── page.tsx
├── login/
│   └── page.tsx
└── (.)login/
    └── page.tsx
```

```tsx
// app/(.)login/page.tsx
import Modal from "@/components/Modal";
import LoginForm from "@/components/LoginForm";

export default function LoginModal() {
  return (
    <Modal>
      <h2>Login</h2>
      <LoginForm />
    </Modal>
  );
}
```

```tsx
// app/login/page.tsx - Full page login
import LoginForm from "@/components/LoginForm";

export default function LoginPage() {
  return (
    <div className="login-page">
      <h1>Login to Your Account</h1>
      <LoginForm />
    </div>
  );
}
```

### Combining Parallel and Intercepting Routes

```
app/
├── @modal/
│   ├── (.)photo/
│   │   └── [id]/
│   │       └── page.tsx
│   └── default.tsx
├── layout.tsx
└── photo/
    └── [id]/
        └── page.tsx
```

```tsx
// app/layout.tsx
export default function RootLayout({ children, modal }: { children: React.ReactNode; modal: React.ReactNode }) {
  return (
    <html>
      <body>
        {children}
        {modal}
      </body>
    </html>
  );
}
```

```tsx
// app/@modal/default.tsx
export default function Default() {
  return null;
}
```

---

## Route Handlers

Route Handlers allow you to create custom request handlers for a given route using Web APIs.

### Basic Route Handler

```tsx
// app/api/hello/route.ts
export async function GET() {
  return Response.json({ message: "Hello, World!" });
}
```

**URL:** `/api/hello`

### HTTP Methods

```tsx
// app/api/posts/route.ts
import { NextRequest } from "next/server";

// GET - Fetch all posts
export async function GET(request: NextRequest) {
  const posts = await db.post.findMany();
  return Response.json(posts);
}

// POST - Create new post
export async function POST(request: NextRequest) {
  const body = await request.json();
  const post = await db.post.create({ data: body });
  return Response.json(post, { status: 201 });
}

// PUT - Update post
export async function PUT(request: NextRequest) {
  const body = await request.json();
  const post = await db.post.update({
    where: { id: body.id },
    data: body,
  });
  return Response.json(post);
}

// DELETE - Delete post
export async function DELETE(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const id = searchParams.get("id");

  await db.post.delete({ where: { id } });
  return Response.json({ success: true });
}

// PATCH - Partial update
export async function PATCH(request: NextRequest) {
  const body = await request.json();
  const post = await db.post.update({
    where: { id: body.id },
    data: body,
  });
  return Response.json(post);
}
```

### Dynamic Route Handlers

```tsx
// app/api/posts/[id]/route.ts
import { NextRequest } from "next/server";

export async function GET(request: NextRequest, { params }: { params: { id: string } }) {
  const post = await db.post.findUnique({
    where: { id: params.id },
  });

  if (!post) {
    return Response.json({ error: "Post not found" }, { status: 404 });
  }

  return Response.json(post);
}

export async function DELETE(request: NextRequest, { params }: { params: { id: string } }) {
  await db.post.delete({
    where: { id: params.id },
  });

  return Response.json({ success: true });
}
```

### Request Object

```tsx
// app/api/search/route.ts
import { NextRequest } from "next/server";

export async function GET(request: NextRequest) {
  // URL and search params
  const { searchParams } = new URL(request.url);
  const query = searchParams.get("q");
  const page = searchParams.get("page") || "1";

  // Headers
  const authorization = request.headers.get("authorization");
  const userAgent = request.headers.get("user-agent");

  // Cookies
  const token = request.cookies.get("token");

  // Perform search
  const results = await search(query, parseInt(page));

  return Response.json(results);
}
```

### Response Object

```tsx
// app/api/users/route.ts
export async function GET() {
  const users = await getUsers();

  return Response.json(
    { users },
    {
      status: 200,
      headers: {
        "Content-Type": "application/json",
        "Cache-Control": "max-age=3600",
      },
    }
  );
}
```

### NextResponse Helper

```tsx
// app/api/data/route.ts
import { NextResponse } from "next/server";

export async function GET() {
  const data = await fetchData();

  // JSON response with headers
  return NextResponse.json(data, {
    status: 200,
    headers: {
      "Cache-Control": "max-age=3600",
    },
  });
}

// Redirect
export async function POST(request: Request) {
  const data = await request.json();
  await createItem(data);

  return NextResponse.redirect(new URL("/success", request.url));
}
```

### Cookies

```tsx
// app/api/auth/route.ts
import { NextRequest, NextResponse } from "next/server";

export async function POST(request: NextRequest) {
  const { username, password } = await request.json();

  // Authenticate user
  const user = await authenticate(username, password);

  if (!user) {
    return NextResponse.json({ error: "Invalid credentials" }, { status: 401 });
  }

  // Create response with cookie
  const response = NextResponse.json({ success: true });

  response.cookies.set("token", user.token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    sameSite: "lax",
    maxAge: 60 * 60 * 24 * 7, // 1 week
  });

  return response;
}

export async function DELETE(request: NextRequest) {
  const response = NextResponse.json({ success: true });

  // Delete cookie
  response.cookies.delete("token");

  return response;
}
```

### Error Handling

```tsx
// app/api/users/[id]/route.ts
import { NextRequest, NextResponse } from "next/server";

export async function GET(request: NextRequest, { params }: { params: { id: string } }) {
  try {
    const user = await db.user.findUnique({
      where: { id: params.id },
    });

    if (!user) {
      return NextResponse.json({ error: "User not found" }, { status: 404 });
    }

    return NextResponse.json(user);
  } catch (error) {
    console.error("Database error:", error);
    return NextResponse.json({ error: "Internal server error" }, { status: 500 });
  }
}
```

### Middleware in Route Handlers

```tsx
// app/api/admin/route.ts
import { NextRequest, NextResponse } from "next/server";
import { verifyAuth } from "@/lib/auth";

export async function GET(request: NextRequest) {
  // Verify authentication
  const token = request.headers.get("authorization")?.split(" ")[1];

  if (!token) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  const user = await verifyAuth(token);

  if (!user || user.role !== "admin") {
    return NextResponse.json({ error: "Forbidden" }, { status: 403 });
  }

  // Admin logic
  const data = await getAdminData();
  return NextResponse.json(data);
}
```

### CORS

```tsx
// app/api/public/route.ts
import { NextResponse } from "next/server";

export async function GET(request: Request) {
  const data = await fetchPublicData();

  return NextResponse.json(data, {
    headers: {
      "Access-Control-Allow-Origin": "*",
      "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, OPTIONS",
      "Access-Control-Allow-Headers": "Content-Type, Authorization",
    },
  });
}

export async function OPTIONS() {
  return NextResponse.json(
    {},
    {
      headers: {
        "Access-Control-Allow-Origin": "*",
        "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, OPTIONS",
        "Access-Control-Allow-Headers": "Content-Type, Authorization",
      },
    }
  );
}
```

### Streaming Response

```tsx
// app/api/stream/route.ts
export async function GET() {
  const encoder = new TextEncoder();

  const stream = new ReadableStream({
    async start(controller) {
      for (let i = 0; i < 10; i++) {
        const message = `data: ${JSON.stringify({ count: i })}\n\n`;
        controller.enqueue(encoder.encode(message));
        await new Promise((resolve) => setTimeout(resolve, 1000));
      }
      controller.close();
    },
  });

  return new Response(stream, {
    headers: {
      "Content-Type": "text/event-stream",
      "Cache-Control": "no-cache",
      Connection: "keep-alive",
    },
  });
}
```

### File Upload

```tsx
// app/api/upload/route.ts
import { NextRequest, NextResponse } from "next/server";
import { writeFile } from "fs/promises";
import path from "path";

export async function POST(request: NextRequest) {
  const formData = await request.formData();
  const file = formData.get("file") as File;

  if (!file) {
    return NextResponse.json({ error: "No file uploaded" }, { status: 400 });
  }

  const bytes = await file.arrayBuffer();
  const buffer = Buffer.from(bytes);

  const filePath = path.join(process.cwd(), "public", "uploads", file.name);
  await writeFile(filePath, buffer);

  return NextResponse.json({
    success: true,
    filename: file.name,
    size: file.size,
  });
}
```

---

## Best Practices

### 1. **Use Server Components by Default**

```tsx
// ✅ Good - Server Component (default)
export default async function Page() {
  const data = await fetchData();
  return <div>{data}</div>;
}

// ❌ Only use 'use client' when needed
("use client");
export default function Page() {
  // Client Component
}
```

### 2. **Organize Routes with Groups**

```
app/
├── (marketing)/      # Marketing pages
├── (shop)/          # E-commerce pages
└── (auth)/          # Authentication pages
```

### 3. **Use Loading and Error Boundaries**

```tsx
// ✅ Good - Provide loading and error states
app/
├── loading.tsx
├── error.tsx
└── page.tsx
```

### 4. **Implement Proper Metadata**

```tsx
// ✅ Good - SEO-friendly metadata
export const metadata = {
  title: "Page Title",
  description: "Page description",
};
```

### 5. **Use Dynamic Imports for Heavy Components**

```tsx
// ✅ Good - Lazy load heavy components
const Chart = dynamic(() => import("./Chart"), {
  loading: () => <Skeleton />,
  ssr: false,
});
```

### 6. **Implement Authentication Guards**

```tsx
// ✅ Good - Check auth in Server Components
export default async function ProtectedPage() {
  const user = await getUser();

  if (!user) {
    redirect("/login");
  }

  return <div>Protected content</div>;
}
```

### 7. **Use Route Handlers for APIs**

```tsx
// ✅ Good - RESTful API routes
app/api/
├── posts/
│   ├── route.ts          # GET /api/posts
│   └── [id]/
│       └── route.ts      # GET /api/posts/:id
```

### 8. **Optimize Data Fetching**

```tsx
// ✅ Good - Parallel data fetching
export default async function Page() {
  const [user, posts] = await Promise.all([getUser(), getPosts()]);

  return (
    <div>
      <User data={user} />
      <Posts data={posts} />
    </div>
  );
}
```

### 9. **Use Suspense for Streaming**

```tsx
// ✅ Good - Stream content progressively
export default function Page() {
  return (
    <div>
      <h1>Fast content</h1>
      <Suspense fallback={<Loading />}>
        <SlowComponent />
      </Suspense>
    </div>
  );
}
```

### 10. **Implement Proper Error Handling**

```tsx
// ✅ Good - Handle errors gracefully
export default async function Page() {
  try {
    const data = await fetchData();
    return <div>{data}</div>;
  } catch (error) {
    console.error(error);
    throw error; // Caught by error.tsx
  }
}
```

---

## Summary

### Key Routing Concepts

✅ **File-System Based Routing**

- `page.tsx` - Creates a route
- `layout.tsx` - Shared UI
- `loading.tsx` - Loading state
- `error.tsx` - Error boundary
- `not-found.tsx` - 404 page

✅ **Dynamic Routes**

- `[param]` - Single dynamic segment
- `[...param]` - Catch-all segments
- `[[...param]]` - Optional catch-all

✅ **Advanced Patterns**

- Route Groups - `(name)`
- Parallel Routes - `@name`
- Intercepting Routes - `(..)name`

✅ **Navigation**

- `<Link>` component
- `useRouter()` hook
- `redirect()` function

### Route File Convention

| File            | Purpose                 |
| --------------- | ----------------------- |
| `page.tsx`      | Page UI                 |
| `layout.tsx`    | Shared layout           |
| `loading.tsx`   | Loading UI              |
| `error.tsx`     | Error boundary          |
| `not-found.tsx` | 404 UI                  |
| `route.ts`      | API endpoint            |
| `template.tsx`  | Re-rendered layout      |
| `default.tsx`   | Parallel route fallback |

### Quick Reference

```
app/
├── page.tsx                      # /
├── about/page.tsx                # /about
├── blog/[slug]/page.tsx          # /blog/:slug
├── shop/[...slug]/page.tsx       # /shop/*
├── (auth)/login/page.tsx         # /login (grouped)
├── dashboard/@team/page.tsx      # Parallel route
├── photo/(..)photo/page.tsx      # Intercepting route
└── api/users/route.ts            # /api/users
```

---

---

## Pages Router (Legacy)

> [!NOTE]
> The Pages Router is the traditional routing system from Next.js 12 and earlier. While still supported in Next.js 16, the App Router is recommended for new projects.

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

### Navigation in Pages Router

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

### Key Differences: App Router vs Pages Router

| Feature          | Pages Router                  | App Router                     |
| ---------------- | ----------------------------- | ------------------------------ |
| Directory        | `pages/`                      | `app/`                         |
| Route definition | File = Route                  | `page.tsx` = Route             |
| Layouts          | Single `_app.tsx`             | Nested `layout.tsx` files      |
| Loading states   | Manual implementation         | Built-in `loading.tsx`         |
| Error handling   | `_error.tsx`                  | `error.tsx` per route          |
| Data fetching    | `getServerSideProps`, `getStaticProps` | Async Server Components |
| Default rendering| Client-side                   | Server Components              |

---

## Best Practices

### 1. Use App Router for New Projects

Start with the App Router to leverage Server Components and modern features.

### 2. Organize with Route Groups

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

### 3. Colocate Components

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

### 4. Loading States

Always provide loading states for better UX:

```tsx
// app/products/loading.tsx
export default function Loading() {
  return <ProductsSkeleton />;
}
```

### 5. Error Boundaries

Handle errors gracefully at appropriate levels:

```
app/
├── error.tsx              # Catches all errors
└── dashboard/
    ├── error.tsx         # Catches dashboard errors
    └── settings/
        └── error.tsx     # Catches settings errors
```

### 6. Metadata for SEO

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

### 7. Prefetch Important Routes

```tsx
<Link href="/dashboard" prefetch={true}>
  Dashboard
</Link>
```

### 8. Use Proper Status Codes

```tsx
// 404
notFound();

// 301 Permanent Redirect
permanentRedirect("/new-url");

// 302 Temporary Redirect
redirect("/temporary-url");
```

### 9. Validate Dynamic Parameters

```tsx
export default async function Page({ params }: { params: { id: string } }) {
  const id = parseInt(params.id);

  if (isNaN(id)) {
    notFound();
  }

  // Continue with valid id
}
```

### 10. Private Folders

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

## Related Documentation

- [Next.js 16 Folder Structure Guide](file:///c:/Users/mmt20/Desktop/beetleware-frontend-intern/nexjs/next16-folder-structure.md) - Comprehensive folder organization patterns
- [Rendering & Data Fetching Guide](file:///c:/Users/mmt20/Desktop/beetleware-frontend-intern/nexjs/rendering-and-data-fetching.md) - SSR, SSG, ISR strategies
- [Optimization Guide](file:///c:/Users/mmt20/Desktop/beetleware-frontend-intern/nexjs/optimization-guide.md) - Performance optimization techniques
- [Next.js Basics](file:///c:/Users/mmt20/Desktop/beetleware-frontend-intern/nexjs/nextjs-basics.md) - Getting started with Next.js

---

## Resources

- **Official Docs:** [nextjs.org/docs/app](https://nextjs.org/docs/app)
- **Routing Guide:** [nextjs.org/docs/app/building-your-application/routing](https://nextjs.org/docs/app/building-your-application/routing)
- **Examples:** [github.com/vercel/next.js/tree/canary/examples](https://github.com/vercel/next.js/tree/canary/examples)
- **App Router Playground:** [app-router.vercel.app](https://app-router.vercel.app)

---

**Next.js 16 App Router** - The modern way to build Next.js applications with powerful routing capabilities, Server Components, and excellent developer experience.s/app/building-your-application/rendering/server-components](https://nextjs.org/docs/app/building-your-application/rendering/server-components)
