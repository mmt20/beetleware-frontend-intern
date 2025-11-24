# Rendering & Data Fetching in Next.js

## Table of Contents

1. [Introduction to Rendering](#1-introduction-to-rendering)
2. [Static Site Generation (SSG)](#2-static-site-generation-ssg)
3. [SSG with getStaticProps](#3-ssg-with-getstaticprops)
4. [Incremental Static Regeneration (ISR)](#4-incremental-static-regeneration-isr)
5. [SSG with Dynamic Parameters](#5-ssg-with-dynamic-parameters)
6. [getStaticPaths](#6-getstaticpaths)
7. [getStaticPaths Fallback](#7-getstaticpaths-fallback)
8. [Server-Side Rendering (SSR) Introduction](#8-server-side-rendering-ssr-introduction)
9. [SSR with Dynamic Routes](#9-ssr-with-dynamic-routes)
10. [Client-Side Rendering (CSR)](#10-client-side-rendering-csr)
11. [Modern App Router Data Fetching](#11-modern-app-router-data-fetching)
12. [Comparison & Best Practices](#12-comparison--best-practices)

---

## 1. Introduction to Rendering

Next.js provides **multiple rendering strategies** to optimize performance and user experience. Each method has specific use cases.

### Rendering Methods Overview

| Method                       | When Rendered                 | Use Case                      | Data Freshness      |
| ---------------------------- | ----------------------------- | ----------------------------- | ------------------- |
| **SSG** (Static)             | Build time                    | Content rarely changes        | Stale until rebuild |
| **ISR** (Incremental Static) | Build time + periodic updates | Content changes occasionally  | Periodic updates    |
| **SSR** (Server-side)        | Request time                  | Content changes frequently    | Always fresh        |
| **CSR** (Client-side)        | Browser runtime               | Personalized/interactive data | Fresh on client     |

### Why Multiple Rendering Strategies?

```tsx
// Different pages, different needs:
- Homepage → SSG (fast, cached)
- Blog posts → ISR (updated periodically)
- User dashboard → SSR (personalized)
- Shopping cart → CSR (real-time updates)
```

### Pre-rendering Benefits

✅ **Better SEO** - Content visible to search engines  
✅ **Faster initial load** - HTML ready immediately  
✅ **Social sharing** - Open Graph tags work correctly  
✅ **Performance** - Less JavaScript to download

---

## 2. Static Site Generation (SSG)

SSG generates HTML at **build time**. The same HTML is served to all users.

### How SSG Works

```
1. Build Time (next build)
   ↓
2. Generate static HTML for each page
   ↓
3. Deploy static files
   ↓
4. User Request → Serve pre-built HTML (instant!)
```

### Basic Example

```tsx
// pages/about.tsx
export default function About() {
  return (
    <div>
      <h1>About Us</h1>
      <p>This page was generated at build time.</p>
      <p>Build time: {new Date().toISOString()}</p>
    </div>
  );
}
```

When you run `npm run build`, this page becomes a static HTML file.

### When to Use SSG

✅ **Perfect for:**

- Marketing pages (homepage, about, contact)
- Blog posts and articles
- Documentation
- Product listings (that don't change often)
- Landing pages
- Help/FAQ pages

❌ **Not suitable for:**

- User-specific content (dashboards)
- Real-time data (stock prices)
- Frequently changing content (every second)

### Benefits of SSG

```tsx
// ✅ Advantages
- Lightning fast (served from CDN)
- No server required for each request
- Best SEO performance
- Can handle massive traffic
- Low hosting costs

// ❌ Limitations
- Data can become stale
- Need to rebuild to update content
- Not suitable for personalized content
```

---

## 3. SSG with getStaticProps

`getStaticProps` fetches data at build time for static generation.

### Basic Usage

```tsx
// pages/blog.tsx
interface Post {
  id: number;
  title: string;
  excerpt: string;
}

interface BlogProps {
  posts: Post[];
  buildTime: string;
}

export default function Blog({ posts, buildTime }: BlogProps) {
  return (
    <div>
      <h1>Blog Posts</h1>
      <p>Generated at: {buildTime}</p>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>
            <h2>{post.title}</h2>
            <p>{post.excerpt}</p>
          </li>
        ))}
      </ul>
    </div>
  );
}

// This function runs at BUILD TIME
export async function getStaticProps() {
  // Fetch data from external API
  const res = await fetch("https://api.example.com/posts");
  const posts = await res.json();

  return {
    props: {
      posts,
      buildTime: new Date().toISOString(),
    },
  };
}
```

### How getStaticProps Works

```
Build Time:
1. Next.js calls getStaticProps()
2. Fetch data from API/database
3. Return data as props
4. Generate HTML with the data
5. Save as static file

Request Time:
1. User visits page
2. Serve pre-generated HTML
3. No API calls needed!
```

### Advanced getStaticProps

```tsx
// pages/products.tsx
export async function getStaticProps(context) {
  // Context provides useful info
  console.log(context.params); // Route parameters
  console.log(context.preview); // Preview mode
  console.log(context.previewData); // Preview mode data
  console.log(context.locale); // Current locale (i18n)

  // Fetch from multiple sources
  const [products, categories] = await Promise.all([
    fetch("https://api.example.com/products").then((r) => r.json()),
    fetch("https://api.example.com/categories").then((r) => r.json()),
  ]);

  // Handle errors
  if (!products) {
    return {
      notFound: true, // Shows 404 page
    };
  }

  return {
    props: {
      products,
      categories,
    },
  };
}
```

### Redirects and notFound

```tsx
export async function getStaticProps() {
  const data = await fetchData();

  // Return 404 page
  if (!data) {
    return {
      notFound: true,
    };
  }

  // Redirect to another page
  if (data.moved) {
    return {
      redirect: {
        destination: "/new-page",
        permanent: false, // 307 temporary redirect
      },
    };
  }

  return {
    props: { data },
  };
}
```

### TypeScript Support

```tsx
import { GetStaticProps } from "next";

interface PageProps {
  posts: Post[];
}

export const getStaticProps: GetStaticProps<PageProps> = async () => {
  const posts = await fetchPosts();

  return {
    props: {
      posts,
    },
  };
};
```

---

## 4. Incremental Static Regeneration (ISR)

ISR allows you to **update static pages after build** without rebuilding the entire site.

### The Problem ISR Solves

```tsx
// Pure SSG Problem:
- 10,000 blog posts
- Add 1 new post
- Must rebuild ALL 10,000 pages 😢
- Takes 30 minutes

// ISR Solution:
- Add 1 new post
- Rebuild only that page
- Other pages rebuild on-demand ✨
```

### Basic ISR Usage

```tsx
// pages/posts.tsx
export default function Posts({ posts, lastUpdate }: PostsProps) {
  return (
    <div>
      <h1>Latest Posts</h1>
      <p>Last updated: {lastUpdate}</p>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}

export async function getStaticProps() {
  const res = await fetch("https://api.example.com/posts");
  const posts = await res.json();

  return {
    props: {
      posts,
      lastUpdate: new Date().toISOString(),
    },
    revalidate: 60, // Revalidate every 60 seconds
  };
}
```

### How ISR Works

```
1. First Request (after build):
   - Serve static page (fast!)

2. Request after 60 seconds:
   - Still serve cached page (fast!)
   - Trigger regeneration in background

3. Next Request:
   - Serve newly generated page
   - Start new 60-second timer

Timeline:
0s    → User A visits → Gets cached version
65s   → User B visits → Gets cached version + triggers rebuild
70s   → Rebuild completes
75s   → User C visits → Gets NEW version
```

### ISR with Dynamic Routes

```tsx
// pages/posts/[id].tsx
export default function Post({ post }: PostProps) {
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
      <p>Last updated: {post.updatedAt}</p>
    </article>
  );
}

export async function getStaticPaths() {
  // Generate most popular posts at build time
  const res = await fetch("https://api.example.com/posts?popular=true");
  const posts = await res.json();

  const paths = posts.map((post) => ({
    params: { id: post.id.toString() },
  }));

  return {
    paths,
    fallback: "blocking", // Generate other pages on-demand
  };
}

export async function getStaticProps({ params }) {
  const res = await fetch(`https://api.example.com/posts/${params.id}`);
  const post = await res.json();

  return {
    props: { post },
    revalidate: 300, // Revalidate every 5 minutes
  };
}
```

### On-Demand Revalidation

Trigger revalidation manually via API route:

```tsx
// pages/api/revalidate.ts
import { NextApiRequest, NextApiResponse } from "next";

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  // Check for secret to confirm this is a valid request
  if (req.query.secret !== process.env.REVALIDATE_TOKEN) {
    return res.status(401).json({ message: "Invalid token" });
  }

  try {
    // Revalidate specific path
    await res.revalidate("/posts");
    await res.revalidate(`/posts/${req.query.id}`);

    return res.json({ revalidated: true });
  } catch (err) {
    return res.status(500).send("Error revalidating");
  }
}

// Usage:
// POST /api/revalidate?secret=TOKEN&id=123
```

### ISR Best Practices

```tsx
// ✅ Good revalidate values:
- News articles: 60 seconds (1 minute)
- Blog posts: 3600 seconds (1 hour)
- Product pages: 300 seconds (5 minutes)
- Marketing pages: 86400 seconds (1 day)

// ❌ Bad revalidate values:
- 1 second (use SSR instead)
- 0 (disables ISR)
- Too long for frequently updated content
```

### When to Use ISR

✅ **Perfect for:**

- Blog with frequent updates
- E-commerce product pages
- News websites
- Content that changes throughout the day
- Large sites with thousands of pages

❌ **Not suitable for:**

- Real-time data (use SSR)
- User-specific content (use CSR)
- Content that must be always up-to-date

---

## 5. SSG with Dynamic Parameters

Generate static pages with dynamic routes using route parameters.

### Dynamic Route Example

```tsx
// pages/posts/[slug].tsx
interface PostProps {
  post: {
    slug: string;
    title: string;
    content: string;
    publishedAt: string;
  };
}

export default function Post({ post }: PostProps) {
  return (
    <article>
      <h1>{post.title}</h1>
      <time>{post.publishedAt}</time>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}

// Tell Next.js which dynamic routes to generate
export async function getStaticPaths() {
  // Fetch all posts
  const res = await fetch("https://api.example.com/posts");
  const posts = await res.json();

  // Generate paths for each post
  const paths = posts.map((post) => ({
    params: { slug: post.slug },
  }));

  // paths = [
  //   { params: { slug: 'hello-world' } },
  //   { params: { slug: 'next-js-guide' } },
  // ]

  return {
    paths,
    fallback: false, // 404 for non-existent paths
  };
}

// Fetch data for each post
export async function getStaticProps({ params }) {
  const res = await fetch(`https://api.example.com/posts/${params.slug}`);
  const post = await res.json();

  return {
    props: { post },
  };
}
```

### Multiple Dynamic Parameters

```tsx
// pages/blog/[category]/[slug].tsx
// Example: /blog/javascript/react-hooks

export default function BlogPost({ post, category }) {
  return (
    <article>
      <span>Category: {category}</span>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}

export async function getStaticPaths() {
  const categories = await fetchCategories();
  const paths = [];

  // Generate paths for all category/slug combinations
  for (const category of categories) {
    const posts = await fetchPostsByCategory(category.slug);

    posts.forEach((post) => {
      paths.push({
        params: {
          category: category.slug,
          slug: post.slug,
        },
      });
    });
  }

  // paths = [
  //   { params: { category: 'javascript', slug: 'react-hooks' } },
  //   { params: { category: 'javascript', slug: 'async-await' } },
  //   { params: { category: 'css', slug: 'flexbox-guide' } },
  // ]

  return {
    paths,
    fallback: false,
  };
}

export async function getStaticProps({ params }) {
  const post = await fetchPost(params.category, params.slug);

  return {
    props: {
      post,
      category: params.category,
    },
  };
}
```

### Catch-All Routes

```tsx
// pages/docs/[...slug].tsx
// Matches: /docs/a, /docs/a/b, /docs/a/b/c

export default function Docs({ path, content }) {
  return (
    <div>
      <h1>Documentation: {path.join(" / ")}</h1>
      <div>{content}</div>
    </div>
  );
}

export async function getStaticPaths() {
  const paths = [
    { params: { slug: ["getting-started"] } },
    { params: { slug: ["getting-started", "installation"] } },
    { params: { slug: ["api", "components", "button"] } },
  ];

  return {
    paths,
    fallback: false,
  };
}

export async function getStaticProps({ params }) {
  const slug = params.slug.join("/");
  const content = await fetchDocContent(slug);

  return {
    props: {
      path: params.slug,
      content,
    },
  };
}
```

---

## 6. getStaticPaths

`getStaticPaths` defines which dynamic routes to pre-render at build time.

### Basic Structure

```tsx
export async function getStaticPaths() {
  return {
    paths: [
      // Array of routes to pre-render
    ],
    fallback: false, // or true, 'blocking'
  };
}
```

### Detailed Example

```tsx
// pages/products/[id].tsx
export default function Product({ product }) {
  return (
    <div>
      <h1>{product.name}</h1>
      <p>${product.price}</p>
    </div>
  );
}

export async function getStaticPaths() {
  // Fetch all products
  const res = await fetch("https://api.example.com/products");
  const products = await res.json();

  // Map to Next.js paths format
  const paths = products.map((product) => ({
    params: { id: product.id.toString() }, // Must be string
  }));

  console.log("Generating static pages for:", paths);

  return {
    paths,
    fallback: false,
  };
}

export async function getStaticProps({ params }) {
  const res = await fetch(`https://api.example.com/products/${params.id}`);
  const product = await res.json();

  if (!product) {
    return {
      notFound: true,
    };
  }

  return {
    props: { product },
    revalidate: 3600, // Can combine with ISR
  };
}
```

### Partial Static Generation

For large sites, generate only popular pages at build time:

```tsx
export async function getStaticPaths() {
  // Only generate top 100 most popular products
  const res = await fetch("https://api.example.com/products?popular=100");
  const popularProducts = await res.json();

  const paths = popularProducts.map((product) => ({
    params: { id: product.id.toString() },
  }));

  return {
    paths,
    fallback: "blocking", // Generate others on-demand
  };
}
```

### Best Practices

```tsx
// ✅ Good: Generate critical pages
export async function getStaticPaths() {
  // Homepage categories, top 20 products
  const criticalPaths = await fetchCriticalPages();

  return {
    paths: criticalPaths,
    fallback: "blocking",
  };
}

// ❌ Bad: Generate all pages (slow builds)
export async function getStaticPaths() {
  // 10,000 products = 10,000 pages = slow build
  const allProducts = await fetchAllProducts(); // DON'T DO THIS

  return {
    paths: allProducts.map((p) => ({ params: { id: p.id } })),
    fallback: false,
  };
}
```

---

## 7. getStaticPaths Fallback

The `fallback` option controls behavior for paths not generated at build time.

### Fallback Options

```tsx
fallback: false; // 404 for non-generated paths
fallback: true; // Generate on-demand, show fallback UI
fallback: "blocking"; // Generate on-demand, wait for page
```

### Fallback: false

```tsx
export async function getStaticPaths() {
  const paths = [{ params: { id: "1" } }, { params: { id: "2" } }, { params: { id: "3" } }];

  return {
    paths,
    fallback: false, // Only these 3 pages exist
  };
}

// Result:
// /products/1 → ✅ Works (pre-generated)
// /products/2 → ✅ Works (pre-generated)
// /products/99 → ❌ 404 Page Not Found
```

**Use when:**

- Small, fixed number of pages
- All pages known at build time
- 404 for unknown paths is acceptable

### Fallback: true

```tsx
// pages/products/[id].tsx
import { useRouter } from "next/router";

export default function Product({ product }) {
  const router = useRouter();

  // Show loading state while page generates
  if (router.isFallback) {
    return (
      <div>
        <h1>Loading...</h1>
        <p>Generating page...</p>
      </div>
    );
  }

  return (
    <div>
      <h1>{product.name}</h1>
      <p>${product.price}</p>
    </div>
  );
}

export async function getStaticPaths() {
  // Only generate top 10 products
  const popularProducts = await fetchPopularProducts(10);

  const paths = popularProducts.map((p) => ({
    params: { id: p.id.toString() },
  }));

  return {
    paths,
    fallback: true, // Generate other products on-demand
  };
}

export async function getStaticProps({ params }) {
  const product = await fetchProduct(params.id);

  if (!product) {
    return {
      notFound: true,
    };
  }

  return {
    props: { product },
    revalidate: 3600,
  };
}
```

**How it works:**

```
1. User visits /products/99 (not pre-generated)
2. Show fallback UI (loading state)
3. Generate page in background
4. Replace fallback with real content
5. Cache generated page for future requests
```

**Use when:**

- Large number of pages
- Want to show loading UI
- Need fast builds

### Fallback: 'blocking'

```tsx
export async function getStaticPaths() {
  const popularProducts = await fetchPopularProducts(10);

  const paths = popularProducts.map((p) => ({
    params: { id: p.id.toString() },
  }));

  return {
    paths,
    fallback: "blocking", // Wait for page to generate
  };
}

// No need to check router.isFallback!
export default function Product({ product }) {
  return (
    <div>
      <h1>{product.name}</h1>
      <p>${product.price}</p>
    </div>
  );
}
```

**How it works:**

```
1. User visits /products/99
2. Server generates page (user waits)
3. Server sends complete HTML
4. Cache generated page
5. Future requests are instant
```

**Use when:**

- SEO is critical (no fallback UI)
- Don't want to show loading state
- Acceptable for user to wait a bit

### Comparison Table

| Aspect          | `fallback: false` | `fallback: true`     | `fallback: 'blocking'`    |
| --------------- | ----------------- | -------------------- | ------------------------- |
| Build time      | Slow (all pages)  | Fast (some pages)    | Fast (some pages)         |
| First visit     | Instant           | Shows loading        | Waits for generation      |
| SEO             | ✅ Perfect        | ⚠️ Loading state     | ✅ Perfect                |
| User experience | Best              | Good                 | Good                      |
| 404 handling    | Shows 404         | Shows 404            | Shows 404                 |
| Use case        | Small sites       | Large sites, good UX | Large sites, SEO critical |

### Real-World Example

```tsx
// E-commerce with 10,000 products
export async function getStaticPaths() {
  // Pre-generate only:
  // - Featured products
  // - Best sellers
  // - New arrivals
  const criticalProducts = await Promise.all([fetchFeaturedProducts(), fetchBestSellers(), fetchNewArrivals()]);

  const paths = criticalProducts.flat().map((p) => ({ params: { id: p.id.toString() } }));

  return {
    paths, // ~100 products
    fallback: "blocking", // Generate other 9,900 on-demand
  };
}

// Result:
// - Fast builds (only 100 pages)
// - Popular pages are instant
// - Rare pages generate once, then cached
// - Perfect SEO for all pages
```

---

## 8. Server-Side Rendering (SSR) Introduction

SSR generates HTML on **every request**. Fresh data every time!

### How SSR Works

```
1. User requests /dashboard
   ↓
2. Server receives request
   ↓
3. Fetch fresh data from database/API
   ↓
4. Generate HTML with data
   ↓
5. Send HTML to user
   ↓
6. User sees page (with fresh data)

Every request = Fresh data
```

### Basic SSR Example

```tsx
// pages/dashboard.tsx
interface DashboardProps {
  user: {
    name: string;
    email: string;
  };
  stats: {
    views: number;
    sales: number;
  };
  requestTime: string;
}

export default function Dashboard({ user, stats, requestTime }: DashboardProps) {
  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <p>Email: {user.email}</p>

      <div>
        <h2>Your Stats</h2>
        <p>Views: {stats.views}</p>
        <p>Sales: {stats.sales}</p>
      </div>

      <p>Page generated at: {requestTime}</p>
    </div>
  );
}

// This runs on EVERY REQUEST
export async function getServerSideProps(context) {
  // Fetch fresh data
  const user = await fetchUser(context.req.cookies.userId);
  const stats = await fetchStats(user.id);

  return {
    props: {
      user,
      stats,
      requestTime: new Date().toISOString(),
    },
  };
}
```

### SSR vs SSG

```tsx
// SSG (getStaticProps) - Runs at build time
export async function getStaticProps() {
  const data = await fetchData();
  // This runs ONCE during build
  return { props: { data } };
}

// SSR (getServerSideProps) - Runs on every request
export async function getServerSideProps() {
  const data = await fetchData();
  // This runs on EVERY page request
  return { props: { data } };
}
```

### When to Use SSR

✅ **Perfect for:**

- User dashboards (personalized)
- Real-time data (stock prices, sports scores)
- Authentication-required pages
- Data that changes constantly
- Personalized recommendations
- Shopping carts

❌ **Not suitable for:**

- Public, static content (use SSG)
- Slow APIs (will slow down page)
- High-traffic pages (expensive)

### SSR with Authentication

```tsx
// pages/profile.tsx
import { getSession } from "next-auth/react";

export async function getServerSideProps(context) {
  // Check authentication
  const session = await getSession(context);

  if (!session) {
    return {
      redirect: {
        destination: "/login",
        permanent: false,
      },
    };
  }

  // Fetch user data
  const userData = await fetchUserData(session.user.id);

  return {
    props: {
      session,
      userData,
    },
  };
}

export default function Profile({ userData }) {
  return (
    <div>
      <h1>{userData.name}</h1>
      <p>{userData.bio}</p>
    </div>
  );
}
```

### SSR Context Object

```tsx
export async function getServerSideProps(context) {
  console.log(context.params); // Route parameters: { id: '123' }
  console.log(context.query); // Query string: { search: 'next' }
  console.log(context.req); // HTTP request object
  console.log(context.res); // HTTP response object
  console.log(context.req.cookies); // Cookies
  console.log(context.req.headers); // Request headers
  console.log(context.resolvedUrl); // Full URL path

  return {
    props: {},
  };
}
```

---

## 9. SSR with Dynamic Routes

Combine SSR with dynamic routes for personalized, real-time pages.

### User Profile Example

```tsx
// pages/users/[username].tsx
interface UserProfileProps {
  user: {
    username: string;
    name: string;
    bio: string;
    followers: number;
    following: number;
  };
  posts: Array<{
    id: number;
    title: string;
    likes: number;
  }>;
  viewedAt: string;
}

export default function UserProfile({ user, posts, viewedAt }: UserProfileProps) {
  return (
    <div>
      <header>
        <h1>
          {user.name} (@{user.username})
        </h1>
        <p>{user.bio}</p>
        <div>
          <span>{user.followers} followers</span>
          <span>{user.following} following</span>
        </div>
      </header>

      <section>
        <h2>Recent Posts</h2>
        {posts.map((post) => (
          <article key={post.id}>
            <h3>{post.title}</h3>
            <p>{post.likes} likes</p>
          </article>
        ))}
      </section>

      <footer>
        <small>Viewed at: {viewedAt}</small>
      </footer>
    </div>
  );
}

export async function getServerSideProps(context) {
  const { username } = context.params;

  // Fetch user data
  const userRes = await fetch(`https://api.example.com/users/${username}`);

  if (!userRes.ok) {
    return {
      notFound: true, // Show 404 page
    };
  }

  const user = await userRes.json();

  // Fetch user's posts
  const postsRes = await fetch(`https://api.example.com/users/${username}/posts`);
  const posts = await postsRes.json();

  return {
    props: {
      user,
      posts,
      viewedAt: new Date().toISOString(),
    },
  };
}
```

### Product Page with Real-Time Inventory

```tsx
// pages/products/[id].tsx
export default function ProductPage({ product, inventory, relatedProducts }) {
  return (
    <div>
      <h1>{product.name}</h1>
      <p>${product.price}</p>

      {/* Real-time inventory */}
      <div>
        {inventory.inStock ? <span>✅ In Stock ({inventory.quantity} available)</span> : <span>❌ Out of Stock</span>}
      </div>

      <div>
        <h2>Related Products</h2>
        {relatedProducts.map((p) => (
          <div key={p.id}>{p.name}</div>
        ))}
      </div>
    </div>
  );
}

export async function getServerSideProps(context) {
  const { id } = context.params;

  // Fetch data in parallel
  const [product, inventory, relatedProducts] = await Promise.all([
    fetch(`https://api.example.com/products/${id}`).then((r) => r.json()),
    fetch(`https://api.example.com/inventory/${id}`).then((r) => r.json()),
    fetch(`https://api.example.com/products/${id}/related`).then((r) => r.json()),
  ]);

  if (!product) {
    return {
      notFound: true,
    };
  }

  return {
    props: {
      product,
      inventory,
      relatedProducts,
    },
  };
}
```

### Search Results Page

```tsx
// pages/search.tsx
export default function SearchPage({ results, query, totalResults, page }) {
  return (
    <div>
      <h1>Search Results for "{query}"</h1>
      <p>{totalResults} results found</p>

      <div>
        {results.map((result) => (
          <div key={result.id}>
            <h2>{result.title}</h2>
            <p>{result.description}</p>
          </div>
        ))}
      </div>

      <div>
        <p>Page {page}</p>
      </div>
    </div>
  );
}

export async function getServerSideProps(context) {
  const { query, page = "1" } = context.query;

  if (!query) {
    return {
      redirect: {
        destination: "/",
        permanent: false,
      },
    };
  }

  // Fetch search results
  const res = await fetch(`https://api.example.com/search?q=${query}&page=${page}`);
  const data = await res.json();

  return {
    props: {
      results: data.results,
      query,
      totalResults: data.total,
      page: parseInt(page),
    },
  };
}
```

### Error Handling in SSR

```tsx
export async function getServerSideProps(context) {
  try {
    const data = await fetchData(context.params.id);

    if (!data) {
      return {
        notFound: true, // Shows 404 page
      };
    }

    return {
      props: { data },
    };
  } catch (error) {
    console.error("Error fetching data:", error);

    // Redirect to error page
    return {
      redirect: {
        destination: "/error",
        permanent: false,
      },
    };

    // Or return error props
    return {
      props: {
        error: error.message,
      },
    };
  }
}
```

---

## 10. Client-Side Rendering (CSR)

Fetch data in the browser after the page loads.

### Why Client-Side Rendering?

```tsx
// Use CSR when:
✅ Data doesn't need to be in HTML (no SEO)
✅ User-specific data (cart, notifications)
✅ Data updates frequently (every second)
✅ Interactive features (live search, filters)
✅ Personalized content after login

// Don't use CSR when:
❌ SEO is important
❌ First paint performance is critical
❌ Content should be in initial HTML
```

### Basic CSR with useEffect

```tsx
// pages/dashboard.tsx
"use client";

import { useState, useEffect } from "react";

interface Stats {
  views: number;
  sales: number;
  revenue: number;
}

export default function Dashboard() {
  const [stats, setStats] = useState<Stats | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    async function fetchStats() {
      try {
        const res = await fetch("/api/stats");
        if (!res.ok) throw new Error("Failed to fetch");

        const data = await res.json();
        setStats(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }

    fetchStats();
  }, []);

  if (loading) return <div>Loading stats...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!stats) return <div>No data available</div>;

  return (
    <div>
      <h1>Dashboard</h1>
      <div>
        <p>Views: {stats.views}</p>
        <p>Sales: {stats.sales}</p>
        <p>Revenue: ${stats.revenue}</p>
      </div>
    </div>
  );
}
```

### CSR with React Query (Recommended)

React Query (TanStack Query) is a powerful data-fetching and state management library.

```bash
npm install @tanstack/react-query
```

```tsx
// pages/_app.tsx - Setup QueryClient
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import type { AppProps } from 'next/app';

const queryClient = new QueryClient();

export default function App({ Component, pageProps }: AppProps) {
  return (
    <QueryClientProvider client={queryClient}>
      <Component {...pageProps} />
    </QueryClientProvider>
  );
}
```

```tsx
// pages/posts.tsx
import { useQuery } from '@tanstack/react-query';

const fetchPosts = async () => {
  const res = await fetch('/api/posts');
  if (!res.ok) throw new Error('Failed to fetch posts');
  return res.json();
};

export default function Posts() {
  const { data, error, isLoading } = useQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
  });

  if (error) return <div>Failed to load</div>;
  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      <h1>Blog Posts</h1>
      {data.posts.map((post) => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  );
}
```

### React Query Features

```tsx
import { useQuery, useQueryClient } from '@tanstack/react-query';

const fetchUser = async () => {
  const res = await fetch('/api/user');
  if (!res.ok) throw new Error('Failed to fetch user');
  return res.json();
};

export default function Profile() {
  const queryClient = useQueryClient();
  
  const { data, error, isLoading } = useQuery({
    queryKey: ['user'],
    queryFn: fetchUser,
    refetchInterval: 3000, // Refetch every 3 seconds
    refetchOnWindowFocus: true, // Refetch when window gains focus
    refetchOnReconnect: true, // Refetch when reconnecting
    staleTime: 2000, // Data fresh for 2 seconds
  });

  // Manual refetch
  const refresh = () => {
    queryClient.invalidateQueries({ queryKey: ['user'] });
  };

  if (error) return <div>Error loading profile</div>;
  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      <h1>{data?.name}</h1>
      <button onClick={refresh}>Refresh</button>
    </div>
  );
}
```

### Pagination with CSR

```tsx
"use client";

import { useState } from "react";
import { useQuery } from '@tanstack/react-query';

const fetchProducts = async (page: number) => {
  const res = await fetch(`/api/products?page=${page}`);
  if (!res.ok) throw new Error('Failed to fetch products');
  return res.json();
};

export default function Products() {
  const [page, setPage] = useState(1);

  const { data, error, isLoading } = useQuery({
    queryKey: ['products', page],
    queryFn: () => fetchProducts(page),
    keepPreviousData: true, // Keep previous data while fetching new page
  });

  if (error) return <div>Error loading products</div>;
  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      <h1>Products</h1>

      <div>
        {data.products.map((product) => (
          <div key={product.id}>
            <h2>{product.name}</h2>
            <p>${product.price}</p>
          </div>
        ))}
      </div>

      <div>
        <button onClick={() => setPage(page - 1)} disabled={page === 1}>
          Previous
        </button>
        <span>Page {page}</span>
        <button onClick={() => setPage(page + 1)} disabled={page >= data.totalPages}>
          Next
        </button>
      </div>
    </div>
  );
}
```

### Real-Time Updates

```tsx
"use client";

import { useState, useEffect } from "react";

export default function LiveScores() {
  const [scores, setScores] = useState([]);

  useEffect(() => {
    // Fetch initial data
    fetchScores();

    // Update every 10 seconds
    const interval = setInterval(fetchScores, 10000);

    return () => clearInterval(interval);
  }, []);

  async function fetchScores() {
    const res = await fetch("/api/scores");
    const data = await res.json();
    setScores(data);
  }

  return (
    <div>
      <h1>Live Scores</h1>
      {scores.map((game) => (
        <div key={game.id}>
          <span>
            {game.homeTeam} {game.homeScore}
          </span>
          <span> - </span>
          <span>
            {game.awayScore} {game.awayTeam}
          </span>
        </div>
      ))}
    </div>
  );
}
```

### Hybrid Approach: SSR + CSR

```tsx
// pages/product/[id].tsx
import { useState, useEffect } from "react";

export default function Product({ product, initialReviews }) {
  const [reviews, setReviews] = useState(initialReviews);

  useEffect(() => {
    // Update reviews in real-time (CSR)
    const interval = setInterval(async () => {
      const res = await fetch(`/api/products/${product.id}/reviews`);
      const newReviews = await res.json();
      setReviews(newReviews);
    }, 30000); // Every 30 seconds

    return () => clearInterval(interval);
  }, [product.id]);

  return (
    <div>
      {/* SSR: SEO-friendly product info */}
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p>${product.price}</p>

      {/* CSR: Real-time reviews */}
      <div>
        <h2>Reviews ({reviews.length})</h2>
        {reviews.map((review) => (
          <div key={review.id}>
            <p>{review.rating} ⭐</p>
            <p>{review.comment}</p>
          </div>
        ))}
      </div>
    </div>
  );
}

// SSR for product data
export async function getServerSideProps({ params }) {
  const [product, reviews] = await Promise.all([
    fetch(`https://api.example.com/products/${params.id}`).then((r) => r.json()),
    fetch(`https://api.example.com/products/${params.id}/reviews`).then((r) => r.json()),
  ]);

  return {
    props: {
      product,
      initialReviews: reviews,
    },
  };
}
```

---

## 11. Modern App Router Data Fetching

Next.js 13+ App Router uses a completely different approach.

### Server Components (Default)

```tsx
// app/posts/page.tsx
// This is a Server Component by default!

async function getPosts() {
  const res = await fetch("https://api.example.com/posts", {
    cache: "force-cache", // SSG (default)
  });

  return res.json();
}

export default async function PostsPage() {
  const posts = await getPosts();

  return (
    <div>
      <h1>Blog Posts</h1>
      {posts.map((post) => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  );
}
```

### Caching Strategies

```tsx
// SSG (Static Site Generation)
fetch(url, {
  cache: "force-cache", // Default - cached indefinitely
});

// SSR (Server-Side Rendering)
fetch(url, {
  cache: "no-store", // Fresh data on every request
});

// ISR (Incremental Static Regeneration)
fetch(url, {
  next: { revalidate: 60 }, // Revalidate every 60 seconds
});
```

### Complete Example

```tsx
// app/products/page.tsx

// SSG with revalidation (ISR)
async function getProducts() {
  const res = await fetch("https://api.example.com/products", {
    next: { revalidate: 3600 }, // Revalidate every hour
  });

  if (!res.ok) {
    throw new Error("Failed to fetch products");
  }

  return res.json();
}

// SSR - Always fresh
async function getFeaturedProduct() {
  const res = await fetch("https://api.example.com/featured", {
    cache: "no-store", // Always fresh
  });

  return res.json();
}

export default async function ProductsPage() {
  // Fetch in parallel
  const [products, featured] = await Promise.all([getProducts(), getFeaturedProduct()]);

  return (
    <div>
      {/* Featured product - always fresh */}
      <section>
        <h2>Featured Product</h2>
        <div>{featured.name}</div>
      </section>

      {/* All products - cached with revalidation */}
      <section>
        <h2>All Products</h2>
        {products.map((product) => (
          <div key={product.id}>{product.name}</div>
        ))}
      </section>
    </div>
  );
}
```

### Dynamic Routes in App Router

```tsx
// app/posts/[slug]/page.tsx

async function getPost(slug: string) {
  const res = await fetch(`https://api.example.com/posts/${slug}`, {
    next: { revalidate: 300 }, // Revalidate every 5 minutes
  });

  if (!res.ok) {
    throw new Error("Failed to fetch post");
  }

  return res.json();
}

export default async function PostPage({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug);

  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  );
}

// Generate static params (like getStaticPaths)
export async function generateStaticParams() {
  const posts = await fetch("https://api.example.com/posts").then((r) => r.json());

  return posts.map((post) => ({
    slug: post.slug,
  }));
}
```

### Client Components for Interactivity

```tsx
// app/products/page.tsx
import ProductList from "./ProductList";

async function getProducts() {
  const res = await fetch("https://api.example.com/products");
  return res.json();
}

export default async function ProductsPage() {
  const products = await getProducts();

  // Pass data to Client Component
  return <ProductList initialProducts={products} />;
}
```

```tsx
// app/products/ProductList.tsx
"use client";

import { useState } from "react";

export default function ProductList({ initialProducts }) {
  const [products, setProducts] = useState(initialProducts);
  const [filter, setFilter] = useState("");

  const filtered = products.filter((p) => p.name.toLowerCase().includes(filter.toLowerCase()));

  return (
    <div>
      <input type="text" value={filter} onChange={(e) => setFilter(e.target.value)} placeholder="Search products..." />

      {filtered.map((product) => (
        <div key={product.id}>{product.name}</div>
      ))}
    </div>
  );
}
```

---

## 12. Comparison & Best Practices

### Rendering Methods Comparison

| Method  | When Rendered    | Data Freshness      | Performance      | SEO        | Use Case           |
| ------- | ---------------- | ------------------- | ---------------- | ---------- | ------------------ |
| **SSG** | Build time       | Stale until rebuild | ⚡⚡⚡ Excellent | ✅ Perfect | Marketing, blogs   |
| **ISR** | Build + periodic | Periodic updates    | ⚡⚡ Great       | ✅ Perfect | E-commerce, news   |
| **SSR** | Every request    | Always fresh        | ⚡ Good          | ✅ Perfect | Dashboards, auth   |
| **CSR** | In browser       | Real-time           | ⚡ Variable      | ❌ Poor    | User-specific data |

### Decision Tree

```
Need SEO?
│
├─ Yes → Pre-render (SSG/ISR/SSR)
│   │
│   ├─ Data changes often?
│   │   ├─ Yes → SSR or ISR
│   │   └─ No → SSG
│   │
│   └─ Large number of pages?
│       ├─ Yes → ISR with fallback
│       └─ No → SSG
│
└─ No → CSR
    │
    └─ Real-time updates?
        ├─ Yes → CSR with polling/WebSocket
        └─ No → Hybrid (SSR + CSR)
```

### Best Practices

#### 1. **Choose the Right Method**

```tsx
// ✅ Good: Use appropriate method for each page
pages/
├── index.tsx              // SSG (marketing)
├── blog/
│   └── [slug].tsx        // ISR (blog posts)
├── dashboard/
│   └── page.tsx          // SSR (personalized)
└── cart/
    └── page.tsx          // CSR (real-time)

// ❌ Bad: Using SSR for everything
pages/
├── index.tsx             // SSR (unnecessary)
├── about.tsx             // SSR (should be SSG)
└── blog/[slug].tsx       // SSR (should be ISR)
```

#### 2. **Optimize Data Fetching**

```tsx
// ✅ Good: Fetch in parallel
export async function getServerSideProps() {
  const [user, posts, comments] = await Promise.all([fetchUser(), fetchPosts(), fetchComments()]);

  return { props: { user, posts, comments } };
}

// ❌ Bad: Sequential fetching
export async function getServerSideProps() {
  const user = await fetchUser(); // Wait
  const posts = await fetchPosts(); // Wait
  const comments = await fetchComments(); // Wait

  return { props: { user, posts, comments } };
}
```

#### 3. **Handle Errors Gracefully**

```tsx
export async function getStaticProps() {
  try {
    const data = await fetchData();

    return {
      props: { data },
      revalidate: 60,
    };
  } catch (error) {
    console.error("Error fetching data:", error);

    // Return fallback data
    return {
      props: {
        data: [],
        error: "Failed to load data",
      },
      revalidate: 10, // Retry sooner
    };
  }
}
```

#### 4. **Use ISR for Large Sites**

```tsx
// ✅ Good: Generate popular pages, use fallback
export async function getStaticPaths() {
  const popularPosts = await fetchPopularPosts(100);

  return {
    paths: popularPosts.map((p) => ({ params: { id: p.id } })),
    fallback: "blocking",
  };
}

// ❌ Bad: Generate all pages (slow builds)
export async function getStaticPaths() {
  const allPosts = await fetchAllPosts(); // 10,000 posts

  return {
    paths: allPosts.map((p) => ({ params: { id: p.id } })),
    fallback: false,
  };
}
```

#### 5. **Combine Methods**

```tsx
// Product page: SSR for product + CSR for reviews
export default function Product({ product }) {
  const [reviews, setReviews] = useState([]);

  useEffect(() => {
    // Fetch reviews client-side
    fetchReviews(product.id).then(setReviews);
  }, [product.id]);

  return (
    <div>
      {/* SSR: Product details (SEO) */}
      <h1>{product.name}</h1>
      <p>{product.description}</p>

      {/* CSR: Reviews (not critical for SEO) */}
      <div>{reviews.length} reviews</div>
    </div>
  );
}

export async function getServerSideProps({ params }) {
  const product = await fetchProduct(params.id);
  return { props: { product } };
}
```

### Performance Tips

```tsx
// 1. Cache API responses
const res = await fetch(url, {
  next: { revalidate: 3600 }, // Cache for 1 hour
});

// 2. Use Suspense for progressive loading
<Suspense fallback={<Loading />}>
  <SlowComponent />
</Suspense>

// 3. Prefetch links
<Link href="/dashboard" prefetch={true}>
  Dashboard
</Link>

// 4. Optimize images
<Image
  src="/product.jpg"
  width={500}
  height={300}
  alt="Product"
  priority // Load immediately
/>

// 5. Use incremental adoption
// Mix App Router and Pages Router
my-app/
├── app/          // New pages
└── pages/        // Existing pages
```

---

## Summary

### Quick Reference

| Goal               | Method | Code                                    |
| ------------------ | ------ | --------------------------------------- |
| Static page        | SSG    | `getStaticProps()`                      |
| Dynamic static     | SSG    | `getStaticPaths()` + `getStaticProps()` |
| Periodic updates   | ISR    | `getStaticProps()` with `revalidate`    |
| Fresh on request   | SSR    | `getServerSideProps()`                  |
| Client-side        | CSR    | `useEffect()` or React Query            |
| App Router static  | SSG    | `fetch()` with `cache: 'force-cache'`   |
| App Router dynamic | SSR    | `fetch()` with `cache: 'no-store'`      |
| App Router ISR     | ISR    | `fetch()` with `next: { revalidate }`   |

### Key Takeaways

✅ **SSG** - Best performance, use for static content
✅ **ISR** - Balance of performance and freshness
✅ **SSR** - Always fresh, use for personalized content
✅ **CSR** - User-specific, real-time data
✅ **Combine methods** - Use multiple strategies in one app
✅ **App Router** - Modern approach with better DX

---

## Resources

- **Official Docs:** [nextjs.org/docs/basic-features/data-fetching](https://nextjs.org/docs/basic-features/data-fetching)
- **React Query:** [tanstack.com/query/latest](https://tanstack.com/query/latest)
- **App Router:** [nextjs.org/docs/app](https://nextjs.org/docs/app)
- **Examples:** [github.com/vercel/next.js/tree/canary/examples](https://github.com/vercel/next.js/tree/canary/examples)
