# Next.js 16 Optimization Guide - Complete Guide

## Table of Contents

1. [Introduction](#introduction)
2. [Image Optimization](#image-optimization)
3. [Font Optimization](#font-optimization)
4. [Script Optimization](#script-optimization)
5. [Head Tag & Metadata](#head-tag--metadata)
6. [Document.js File](#documentjs-file)
7. [Code Splitting & Bundling](#code-splitting--bundling)
8. [Performance Monitoring](#performance-monitoring)
9. [Caching Strategies](#caching-strategies)
10. [Build Optimization](#build-optimization)
11. [Best Practices](#best-practices)

---

## Introduction

> [!NOTE]
> This guide covers optimization techniques for Next.js 16, including both the modern App Router and legacy Pages Router approaches.

Next.js provides built-in optimizations to improve performance, SEO, and user experience. This guide covers all optimization techniques, tools, and best practices.

### Key Optimization Areas

- **Images** - Automatic image optimization and lazy loading
- **Fonts** - Self-host fonts with automatic optimization
- **Scripts** - Efficient third-party script loading
- **Metadata** - SEO optimization with Head and Metadata API
- **Code Splitting** - Automatic route-based splitting
- **Caching** - Intelligent caching strategies

---

## Image Optimization

Next.js provides the `Image` component with automatic optimization.

### Basic Image Component

```tsx
import Image from "next/image";

export default function ProfilePage() {
  return (
    <div>
      <Image src="/profile.jpg" alt="Profile picture" width={500} height={500} />
    </div>
  );
}
```

### Remote Images

```tsx
import Image from "next/image";

export default function Avatar() {
  return <Image src="https://example.com/avatar.jpg" alt="User avatar" width={200} height={200} />;
}
```

**Configure remote domains:**

```js
// next.config.js
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "example.com",
        port: "",
        pathname: "/images/**",
      },
      {
        protocol: "https",
        hostname: "cdn.example.com",
      },
    ],
  },
};
```

### Image Sizes and Quality

```tsx
import Image from "next/image";

export default function Hero() {
  return (
    <div>
      {/* Responsive image with sizes */}
      <Image
        src="/hero.jpg"
        alt="Hero image"
        width={1200}
        height={600}
        sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
        quality={85}
        priority // Disable lazy loading for above-fold images
      />
    </div>
  );
}
```

### Fill Container

```tsx
import Image from "next/image";

export default function Card() {
  return (
    <div style={{ position: "relative", width: "300px", height: "400px" }}>
      <Image src="/card-image.jpg" alt="Card" fill style={{ objectFit: "cover" }} />
    </div>
  );
}
```

### Image Loading Strategies

```tsx
import Image from "next/image";

export default function Gallery() {
  return (
    <>
      {/* Priority loading for above-fold images */}
      <Image src="/hero.jpg" alt="Hero" width={1200} height={600} priority />

      {/* Lazy loading (default) */}
      <Image src="/gallery-1.jpg" alt="Gallery image" width={400} height={300} loading="lazy" />

      {/* Eager loading */}
      <Image src="/important.jpg" alt="Important" width={400} height={300} loading="eager" />
    </>
  );
}
```

### Placeholder & Blur

```tsx
import Image from "next/image";

export default function BlurImage() {
  return (
    <div>
      {/* Blur placeholder */}
      <Image
        src="/photo.jpg"
        alt="Photo"
        width={800}
        height={600}
        placeholder="blur"
        blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRg..." // Base64 blur
      />

      {/* Automatic blur for static imports */}
      <Image
        src={require("./photo.jpg")}
        alt="Photo"
        placeholder="blur" // Automatically generates blur
      />
    </div>
  );
}
```

### Image Configuration

```js
// next.config.js
module.exports = {
  images: {
    formats: ["image/avif", "image/webp"], // Preferred formats
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
    minimumCacheTTL: 60, // Cache time in seconds
    dangerouslyAllowSVG: true,
    contentSecurityPolicy: "default-src 'self'; script-src 'none'; sandbox;",
  },
};
```

---

## Font Optimization

Next.js 13+ provides `next/font` for automatic font optimization.

### Google Fonts

```tsx
// app/layout.tsx
import { Inter, Roboto_Mono } from "next/font/google";

const inter = Inter({
  subsets: ["latin"],
  display: "swap",
});

const robotoMono = Roboto_Mono({
  subsets: ["latin"],
  weight: ["400", "700"],
  style: ["normal", "italic"],
  display: "swap",
  variable: "--font-roboto-mono",
});

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={`${inter.className} ${robotoMono.variable}`}>
      <body>{children}</body>
    </html>
  );
}
```

### Local Fonts

```tsx
// app/layout.tsx
import localFont from "next/font/local";

const myFont = localFont({
  src: "./fonts/my-font.woff2",
  display: "swap",
});

const myFontMultiple = localFont({
  src: [
    {
      path: "./fonts/my-font-regular.woff2",
      weight: "400",
      style: "normal",
    },
    {
      path: "./fonts/my-font-bold.woff2",
      weight: "700",
      style: "normal",
    },
  ],
  variable: "--font-custom",
});

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={myFont.className}>
      <body>{children}</body>
    </html>
  );
}
```

### Font Variables (Tailwind CSS)

```tsx
// app/layout.tsx
import { Inter, Roboto_Mono } from "next/font/google";

const inter = Inter({
  subsets: ["latin"],
  variable: "--font-inter",
});

const robotoMono = Roboto_Mono({
  subsets: ["latin"],
  variable: "--font-roboto-mono",
});

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={`${inter.variable} ${robotoMono.variable}`}>
      <body>{children}</body>
    </html>
  );
}
```

```css
/* globals.css */
@import "tailwindcss";

body {
  font-family: var(--font-inter), sans-serif;
}

code {
  font-family: var(--font-roboto-mono), monospace;
}
```

### Preload Fonts

```tsx
import { Inter } from "next/font/google";

const inter = Inter({
  subsets: ["latin"],
  preload: true, // Preload font (default: true)
});
```

---

## Script Optimization

The `Script` component optimizes third-party script loading.

### Script Loading Strategies

```tsx
import Script from "next/script";

export default function Page() {
  return (
    <>
      {/* beforeInteractive: Load before page is interactive */}
      <Script src="https://example.com/critical-script.js" strategy="beforeInteractive" />

      {/* afterInteractive: Load after page is interactive (default) */}
      <Script src="https://www.google-analytics.com/analytics.js" strategy="afterInteractive" />

      {/* lazyOnload: Load during idle time */}
      <Script src="https://example.com/non-critical.js" strategy="lazyOnload" />

      {/* worker: Load in web worker (experimental) */}
      <Script src="https://example.com/partytown-script.js" strategy="worker" />
    </>
  );
}
```

### Inline Scripts

```tsx
import Script from "next/script";

export default function Page() {
  return (
    <Script id="inline-script" strategy="afterInteractive">
      {`
        console.log('Inline script executed');
        window.dataLayer = window.dataLayer || [];
      `}
    </Script>
  );
}
```

### Script Events

```tsx
"use client";

import Script from "next/script";

export default function Analytics() {
  return (
    <Script
      src="https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID"
      strategy="afterInteractive"
      onLoad={() => {
        console.log("Script loaded successfully");
      }}
      onError={(e) => {
        console.error("Script failed to load", e);
      }}
      onReady={() => {
        console.log("Script ready");
      }}
    />
  );
}
```

### Google Analytics Example

```tsx
// app/layout.tsx
import Script from "next/script";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        {children}

        {/* Google Analytics */}
        <Script src="https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID" strategy="afterInteractive" />
        <Script id="google-analytics" strategy="afterInteractive">
          {`
            window.dataLayer = window.dataLayer || [];
            function gtag(){dataLayer.push(arguments);}
            gtag('js', new Date());
            gtag('config', 'GA_MEASUREMENT_ID');
          `}
        </Script>
      </body>
    </html>
  );
}
```

---

## Head Tag & Metadata

### App Router - Metadata API (Next.js 13+)

The modern way to manage metadata using the Metadata API.

#### Static Metadata

```tsx
// app/page.tsx
import { Metadata } from "next";

export const metadata: Metadata = {
  title: "Home Page",
  description: "Welcome to our website",
  keywords: ["nextjs", "react", "web development"],
  authors: [{ name: "John Doe", url: "https://example.com" }],
  creator: "Your Company",
  publisher: "Your Company",
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      "max-video-preview": -1,
      "max-image-preview": "large",
      "max-snippet": -1,
    },
  },
  openGraph: {
    title: "Home Page",
    description: "Welcome to our website",
    url: "https://example.com",
    siteName: "My Website",
    images: [
      {
        url: "https://example.com/og-image.jpg",
        width: 1200,
        height: 630,
        alt: "Site preview",
      },
    ],
    locale: "en_US",
    type: "website",
  },
  twitter: {
    card: "summary_large_image",
    title: "Home Page",
    description: "Welcome to our website",
    creator: "@yourusername",
    images: ["https://example.com/twitter-image.jpg"],
  },
  viewport: {
    width: "device-width",
    initialScale: 1,
    maximumScale: 1,
  },
  verification: {
    google: "google-site-verification-code",
    yandex: "yandex-verification-code",
  },
  alternates: {
    canonical: "https://example.com",
    languages: {
      "en-US": "https://example.com/en-US",
      "es-ES": "https://example.com/es-ES",
    },
  },
  icons: {
    icon: "/favicon.ico",
    shortcut: "/favicon-16x16.png",
    apple: "/apple-touch-icon.png",
  },
};

export default function HomePage() {
  return <h1>Home Page</h1>;
}
```

#### Dynamic Metadata

```tsx
// app/blog/[slug]/page.tsx
import { Metadata } from "next";

type Props = {
  params: { slug: string };
  searchParams: { [key: string]: string | string[] | undefined };
};

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  // Fetch data
  const post = await getPost(params.slug);

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [
        {
          url: post.coverImage,
          width: 1200,
          height: 630,
        },
      ],
      publishedTime: post.publishedAt,
      authors: [post.author.name],
    },
    twitter: {
      card: "summary_large_image",
      title: post.title,
      description: post.excerpt,
      images: [post.coverImage],
    },
    alternates: {
      canonical: `https://example.com/blog/${params.slug}`,
    },
  };
}

export default function BlogPost({ params }: Props) {
  return <article>...</article>;
}
```

#### Metadata Inheritance

```tsx
// app/layout.tsx - Root metadata
import { Metadata } from "next";

export const metadata: Metadata = {
  title: {
    template: "%s | My Website",
    default: "My Website",
  },
  description: "Default description",
  metadataBase: new URL("https://example.com"),
};

export default function RootLayout({ children }) {
  return (
    <html>
      <body>{children}</body>
    </html>
  );
}
```

```tsx
// app/blog/page.tsx - Inherits and overrides
import { Metadata } from "next";

export const metadata: Metadata = {
  title: "Blog", // Will become "Blog | My Website"
  description: "Read our latest articles",
};

export default function BlogPage() {
  return <h1>Blog</h1>;
}
```

#### JSON-LD for Structured Data

```tsx
// app/blog/[slug]/page.tsx
export default function BlogPost({ post }) {
  const jsonLd = {
    "@context": "https://schema.org",
    "@type": "BlogPosting",
    headline: post.title,
    image: post.coverImage,
    datePublished: post.publishedAt,
    dateModified: post.updatedAt,
    author: {
      "@type": "Person",
      name: post.author.name,
    },
  };

  return (
    <>
      <script type="application/ld+json" dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }} />
      <article>...</article>
    </>
  );
}
```

### Pages Router - Head Component (Legacy)

For Pages Router, use the `Head` component from `next/head`.

```tsx
// pages/index.tsx
import Head from "next/head";

export default function HomePage() {
  return (
    <>
      <Head>
        <title>Home Page | My Website</title>
        <meta name="description" content="Welcome to our website" />
        <meta name="keywords" content="nextjs, react, web development" />
        <meta name="author" content="John Doe" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />

        {/* Open Graph */}
        <meta property="og:title" content="Home Page" />
        <meta property="og:description" content="Welcome to our website" />
        <meta property="og:image" content="https://example.com/og-image.jpg" />
        <meta property="og:url" content="https://example.com" />
        <meta property="og:type" content="website" />

        {/* Twitter Card */}
        <meta name="twitter:card" content="summary_large_image" />
        <meta name="twitter:title" content="Home Page" />
        <meta name="twitter:description" content="Welcome to our website" />
        <meta name="twitter:image" content="https://example.com/twitter-image.jpg" />

        {/* Favicon */}
        <link rel="icon" href="/favicon.ico" />
        <link rel="apple-touch-icon" href="/apple-touch-icon.png" />

        {/* Canonical */}
        <link rel="canonical" href="https://example.com" />
      </Head>

      <h1>Home Page</h1>
    </>
  );
}
```

#### Dynamic Head Content

```tsx
// pages/blog/[slug].tsx
import Head from "next/head";

export default function BlogPost({ post }) {
  return (
    <>
      <Head>
        <title>{post.title} | My Blog</title>
        <meta name="description" content={post.excerpt} />
        <meta property="og:title" content={post.title} />
        <meta property="og:description" content={post.excerpt} />
        <meta property="og:image" content={post.coverImage} />
        <meta property="og:type" content="article" />
        <meta property="article:published_time" content={post.publishedAt} />
        <meta property="article:author" content={post.author.name} />
        <link rel="canonical" href={`https://example.com/blog/${post.slug}`} />
      </Head>

      <article>
        <h1>{post.title}</h1>
        <div dangerouslySetInnerHTML={{ __html: post.content }} />
      </article>
    </>
  );
}

export async function getStaticProps({ params }) {
  const post = await getPost(params.slug);
  return { props: { post } };
}

export async function getStaticPaths() {
  const posts = await getAllPosts();
  return {
    paths: posts.map((post) => ({ params: { slug: post.slug } })),
    fallback: false,
  };
}
```

---

## Document.js File

The `_document.js` file (Pages Router) customizes the HTML document structure. It's only used in the Pages Router, not in the App Router.

### Basic Document Structure

```tsx
// pages/_document.tsx
import { Html, Head, Main, NextScript } from "next/document";

export default function Document() {
  return (
    <Html lang="en">
      <Head />
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  );
}
```

### Custom Document with Metadata

```tsx
// pages/_document.tsx
import { Html, Head, Main, NextScript } from "next/document";

export default function Document() {
  return (
    <Html lang="en">
      <Head>
        {/* Fonts */}
        <link rel="preconnect" href="https://fonts.googleapis.com" />
        <link rel="preconnect" href="https://fonts.gstatic.com" crossOrigin="anonymous" />
        <link
          href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap"
          rel="stylesheet"
        />

        {/* Favicon */}
        <link rel="icon" href="/favicon.ico" />
        <link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png" />
        <link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png" />
        <link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png" />
        <link rel="manifest" href="/site.webmanifest" />

        {/* Theme Color */}
        <meta name="theme-color" content="#ffffff" />

        {/* PWA */}
        <meta name="application-name" content="My App" />
        <meta name="apple-mobile-web-app-capable" content="yes" />
        <meta name="apple-mobile-web-app-status-bar-style" content="default" />
        <meta name="apple-mobile-web-app-title" content="My App" />
        <meta name="mobile-web-app-capable" content="yes" />
      </Head>
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  );
}
```

### Document with Custom Scripts

```tsx
// pages/_document.tsx
import { Html, Head, Main, NextScript } from "next/document";

export default function Document() {
  return (
    <Html lang="en">
      <Head>
        {/* Google Tag Manager */}
        <script
          dangerouslySetInnerHTML={{
            __html: `
              (function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
              new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
              j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
              'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
              })(window,document,'script','dataLayer','GTM-XXXXXXX');
            `,
          }}
        />
      </Head>
      <body>
        {/* Google Tag Manager (noscript) */}
        <noscript>
          <iframe
            src="https://www.googletagmanager.com/ns.html?id=GTM-XXXXXXX"
            height="0"
            width="0"
            style={{ display: "none", visibility: "hidden" }}
          />
        </noscript>

        <Main />
        <NextScript />
      </body>
    </Html>
  );
}
```

### Document with Dark Mode Support

```tsx
// pages/_document.tsx
import { Html, Head, Main, NextScript } from "next/document";

export default function Document() {
  return (
    <Html>
      <Head />
      <body>
        {/* Prevent flash of unstyled content */}
        <script
          dangerouslySetInnerHTML={{
            __html: `
              (function() {
                const theme = localStorage.getItem('theme');
                if (theme === 'dark' || (!theme && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
                  document.documentElement.classList.add('dark');
                }
              })();
            `,
          }}
        />
        <Main />
        <NextScript />
      </body>
    </Html>
  );
}
```

### Document with Server-Side Data

```tsx
// pages/_document.tsx
import Document, { Html, Head, Main, NextScript, DocumentContext } from "next/document";

class MyDocument extends Document {
  static async getInitialProps(ctx: DocumentContext) {
    const initialProps = await Document.getInitialProps(ctx);

    // Add custom data
    return {
      ...initialProps,
      customData: "Some server-side data",
    };
  }

  render() {
    return (
      <Html lang="en">
        <Head>
          <meta name="custom" content={this.props.customData} />
        </Head>
        <body>
          <Main />
          <NextScript />
        </body>
      </Html>
    );
  }
}

export default MyDocument;
```

### Important Notes about \_document.js

**What you CAN do:**

- Customize `<html>` and `<body>` tags
- Add persistent meta tags across all pages
- Add web fonts
- Add third-party scripts
- Add CSS-in-JS server-side rendering

**What you CANNOT do:**

- Use React hooks
- Fetch data in `getStaticProps` or `getServerSideProps`
- Add application logic
- Use Next.js components like `<Image>` or `<Link>`

**Differences from \_app.js:**

- `_document` renders on the server only
- `_app` wraps all page components
- `_document` is for document structure
- `_app` is for application logic

---

## Code Splitting & Bundling

### Automatic Code Splitting

Next.js automatically splits code by route:

```tsx
// Each page is a separate bundle
// app/page.tsx -> main bundle
// app/about/page.tsx -> about bundle
// app/blog/[slug]/page.tsx -> blog-slug bundle
```

### Dynamic Imports

```tsx
import dynamic from "next/dynamic";

// Dynamic import with loading state
const DynamicComponent = dynamic(() => import("@/components/HeavyComponent"), {
  loading: () => <p>Loading...</p>,
  ssr: false, // Disable server-side rendering
});

export default function Page() {
  return (
    <div>
      <h1>Page Content</h1>
      <DynamicComponent />
    </div>
  );
}
```

### Named Exports

```tsx
import dynamic from "next/dynamic";

const DynamicComponent = dynamic(() => import("@/components/MyComponents").then((mod) => mod.SpecificComponent));
```

### Dynamic Import with No SSR

```tsx
import dynamic from "next/dynamic";

// Perfect for components that use browser-only APIs
const MapComponent = dynamic(() => import("@/components/Map"), {
  ssr: false,
  loading: () => <div>Loading map...</div>,
});

export default function LocationPage() {
  return (
    <div>
      <h1>Our Location</h1>
      <MapComponent />
    </div>
  );
}
```

### Conditional Dynamic Imports

```tsx
"use client";

import { useState } from "react";
import dynamic from "next/dynamic";

const AdminPanel = dynamic(() => import("@/components/AdminPanel"));

export default function Dashboard() {
  const [showAdmin, setShowAdmin] = useState(false);

  return (
    <div>
      <button onClick={() => setShowAdmin(true)}>Load Admin Panel</button>

      {showAdmin && <AdminPanel />}
    </div>
  );
}
```

### Bundle Analyzer

Analyze your bundle size:

```bash
npm install @next/bundle-analyzer
```

```js
// next.config.js
const withBundleAnalyzer = require("@next/bundle-analyzer")({
  enabled: process.env.ANALYZE === "true",
});

module.exports = withBundleAnalyzer({
  // Your Next.js config
});
```

```json
// package.json
{
  "scripts": {
    "analyze": "ANALYZE=true next build"
  }
}
```

---

## Performance Monitoring

### Web Vitals

Next.js provides built-in Web Vitals reporting.

```tsx
// app/layout.tsx or pages/_app.tsx
export function reportWebVitals(metric) {
  console.log(metric);

  // Send to analytics
  if (metric.label === "web-vital") {
    gtag("event", metric.name, {
      value: Math.round(metric.name === "CLS" ? metric.value * 1000 : metric.value),
      event_category: "Web Vitals",
      event_label: metric.id,
      non_interaction: true,
    });
  }
}
```

**Metrics reported:**

- **FCP** - First Contentful Paint
- **LCP** - Largest Contentful Paint
- **CLS** - Cumulative Layout Shift
- **FID** - First Input Delay
- **TTFB** - Time to First Byte
- **INP** - Interaction to Next Paint

### Custom Metrics

```tsx
"use client";

import { useEffect } from "react";

export default function PerformanceTracker() {
  useEffect(() => {
    // Measure component mount time
    const start = performance.now();

    return () => {
      const end = performance.now();
      console.log(`Component rendered in ${end - start}ms`);
    };
  }, []);

  return null;
}
```

### Performance API

```tsx
"use client";

import { useEffect } from "react";

export default function Page() {
  useEffect(() => {
    // Get navigation timing
    const navigation = performance.getEntriesByType("navigation")[0];
    console.log("Page load time:", navigation.loadEventEnd - navigation.fetchStart);

    // Get resource timing
    const resources = performance.getEntriesByType("resource");
    resources.forEach((resource) => {
      console.log(`${resource.name}: ${resource.duration}ms`);
    });
  }, []);

  return <div>Performance tracked</div>;
}
```

---

## Caching Strategies

### Static Generation with Revalidation

```tsx
// app/page.tsx
export const revalidate = 3600; // Revalidate every hour

export default async function Page() {
  const data = await fetch("https://api.example.com/data");
  return <div>{JSON.stringify(data)}</div>;
}
```

### On-Demand Revalidation

```tsx
// app/api/revalidate/route.ts
import { revalidatePath } from "next/cache";
import { NextRequest } from "next/server";

export async function POST(request: NextRequest) {
  const path = request.nextUrl.searchParams.get("path");

  if (path) {
    revalidatePath(path);
    return Response.json({ revalidated: true, now: Date.now() });
  }

  return Response.json({ revalidated: false, now: Date.now() });
}
```

### Fetch Caching

```tsx
// Cache forever (default)
fetch("https://api.example.com/data");

// No caching
fetch("https://api.example.com/data", { cache: "no-store" });

// Revalidate after 60 seconds
fetch("https://api.example.com/data", { next: { revalidate: 60 } });

// Tag-based revalidation
fetch("https://api.example.com/data", { next: { tags: ["posts"] } });
```

### Route Segment Config

```tsx
// app/page.tsx
export const dynamic = "force-dynamic"; // Always dynamic
export const dynamicParams = true; // Generate dynamic params on-demand
export const revalidate = false; // Never revalidate
export const fetchCache = "force-cache"; // Force cache all fetches
export const runtime = "edge"; // Use Edge Runtime
export const preferredRegion = "auto"; // Deploy to specific regions

export default function Page() {
  return <div>Configured page</div>;
}
```

---

## Build Optimization

### Next.js Config Optimizations

```js
// next.config.js
module.exports = {
  // Strict mode for better error handling
  reactStrictMode: true,

  // Enable SWC minification (faster than Terser)
  swcMinify: true,

  // Compress assets
  compress: true,

  // Production source maps (optional, for debugging)
  productionBrowserSourceMaps: false,

  // Optimize packages
  experimental: {
    optimizePackageImports: ["lodash", "date-fns"],
  },

  // Compiler options
  compiler: {
    removeConsole: process.env.NODE_ENV === "production",
    styledComponents: true,
  },

  // Output config
  output: "standalone", // For Docker deployment

  // Ignore ESLint errors during build (not recommended)
  eslint: {
    ignoreDuringBuilds: false,
  },

  // Ignore TypeScript errors during build (not recommended)
  typescript: {
    ignoreBuildErrors: false,
  },
};
```

### Tree Shaking

Ensure proper tree shaking:

```tsx
// ❌ Bad - imports entire library
import _ from "lodash";

// ✅ Good - imports only what's needed
import debounce from "lodash/debounce";
import throttle from "lodash/throttle";
```

### Module Federation

```js
// next.config.js
const { NextFederationPlugin } = require("@module-federation/nextjs-mf");

module.exports = {
  webpack(config, options) {
    config.plugins.push(
      new NextFederationPlugin({
        name: "app",
        remotes: {
          remote: "remote@http://localhost:3001/_next/static/chunks/remoteEntry.js",
        },
        shared: {},
      })
    );
    return config;
  },
};
```

---

## Best Practices

### 1. **Use Image Component**

Always use `next/image` for automatic optimization:

```tsx
// ❌ Bad
<img src="/photo.jpg" alt="Photo" />

// ✅ Good
<Image src="/photo.jpg" alt="Photo" width={800} height={600} />
```

### 2. **Optimize Fonts**

Use `next/font` instead of external font links:

```tsx
// ❌ Bad
<link href="https://fonts.googleapis.com/css2?family=Inter" rel="stylesheet" />;

// ✅ Good
import { Inter } from "next/font/google";
const inter = Inter({ subsets: ["latin"] });
```

### 3. **Lazy Load Heavy Components**

```tsx
// ✅ Good for heavy components
const Chart = dynamic(() => import("./Chart"), { ssr: false });
const Editor = dynamic(() => import("./Editor"), { loading: () => <Skeleton /> });
```

### 4. **Minimize Client-Side JavaScript**

```tsx
// Use Server Components by default (App Router)
export default async function Page() {
  const data = await fetchData(); // Server-side
  return <ServerComponent data={data} />;
}

// Only mark as Client Component when needed
'use client';
export default function InteractiveComponent() {
  const [state, setState] = useState();
  return <button onClick={() => setState(...)}>Click</button>;
}
```

### 5. **Optimize Third-Party Scripts**

```tsx
// ✅ Use Script component with appropriate strategy
<Script src="https://example.com/script.js" strategy="lazyOnload" />
```

### 6. **Implement Proper Caching**

```tsx
// ✅ Cache API responses appropriately
fetch("https://api.example.com/data", {
  next: { revalidate: 3600 }, // 1 hour
});
```

### 7. **Use Metadata API**

```tsx
// ✅ Use Metadata API for SEO
export const metadata = {
  title: "Page Title",
  description: "Page description",
};
```

### 8. **Optimize Bundle Size**

- Use dynamic imports for large components
- Analyze bundle with `@next/bundle-analyzer`
- Remove unused dependencies
- Use tree-shakeable imports

### 9. **Implement Error Boundaries**

```tsx
// app/error.tsx
"use client";

export default function Error({ error, reset }) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

### 10. **Monitor Performance**

```tsx
// Implement Web Vitals tracking
export function reportWebVitals(metric) {
  // Send to analytics
  analytics.track(metric);
}
```

---

## Summary

### Key Optimization Techniques

✅ **Built-in Optimizations**

- Automatic code splitting
- Image optimization with `next/image`
- Font optimization with `next/font`
- Script optimization with `next/script`

✅ **Performance Features**

- Server Components
- Streaming with Suspense
- Incremental Static Regeneration
- Edge Runtime

✅ **Best Practices**

- Use Metadata API for SEO
- Implement proper caching strategies
- Lazy load heavy components
- Monitor Web Vitals

### Quick Reference

| Optimization   | Tool/API          | Use Case                      |
| -------------- | ----------------- | ----------------------------- |
| Images         | `next/image`      | Automatic image optimization  |
| Fonts          | `next/font`       | Self-hosted font optimization |
| Scripts        | `next/script`     | Third-party script loading    |
| Metadata       | Metadata API      | SEO and social sharing        |
| Document       | `_document.js`    | HTML document customization   |
| Code Splitting | `dynamic()`       | Lazy load components          |
| Caching        | `revalidate`      | ISR and caching control       |
| Performance    | `reportWebVitals` | Track Core Web Vitals         |

---

## Related Documentation

- [Next.js 16 App Router Routing Guide](file:///c:/Users/mmt20/Desktop/beetleware-frontend-intern/nexjs/app-router-routing-guide.md) - Comprehensive routing patterns
- [Next.js 16 Folder Structure](file:///c:/Users/mmt20/Desktop/beetleware-frontend-intern/nexjs/next16-folder-structure.md) - Project organization
- [Rendering & Data Fetching](file:///c:/Users/mmt20/Desktop/beetleware-frontend-intern/nexjs/rendering-and-data-fetching.md) - SSR, SSG, ISR strategies
- [Next.js Basics](file:///c:/Users/mmt20/Desktop/beetleware-frontend-intern/nexjs/nextjs-basics.md) - Getting started guide

---

## Resources

- **Official Docs:** [nextjs.org/docs/app/building-your-application/optimizing](https://nextjs.org/docs/app/building-your-application/optimizing)
- **Image Optimization:** [nextjs.org/docs/app/building-your-application/optimizing/images](https://nextjs.org/docs/app/building-your-application/optimizing/images)
- **Font Optimization:** [nextjs.org/docs/app/building-your-application/optimizing/fonts](https://nextjs.org/docs/app/building-your-application/optimizing/fonts)
- **Metadata API:** [nextjs.org/docs/app/building-your-application/optimizing/metadata](https://nextjs.org/docs/app/building-your-application/optimizing/metadata)
- **Web Vitals:** [web.dev/vitals](https://web.dev/vitals)
