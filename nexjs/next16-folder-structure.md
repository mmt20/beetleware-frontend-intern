# Next.js 16 Folder Structure - Complete Guide

## Table of Contents

1. [Introduction](#introduction)
2. [App Router Structure](#app-router-structure)
3. [Pages Router Structure](#pages-router-structure)
4. [Project Root Files](#project-root-files)
5. [Public Directory](#public-directory)
6. [Components Organization](#components-organization)
7. [Lib and Utils](#lib-and-utils)
8. [Styles Organization](#styles-organization)
9. [API Routes Structure](#api-routes-structure)
10. [TypeScript Configuration](#typescript-configuration)
11. [Best Practices](#best-practices)
12. [Real-World Examples](#real-world-examples)

---

## Introduction

Next.js 16 supports both the modern **App Router** and the legacy **Pages Router**. Understanding the proper folder structure is crucial for building scalable applications.

### Key Concepts

- **App Router** (`app/`) - Modern routing with Server Components (Recommended)
- **Pages Router** (`pages/`) - Traditional routing system (Legacy)
- **File-based routing** - Folders and files define routes
- **Colocation** - Keep related files together
- **Convention over configuration** - Special file names have specific purposes

---

## App Router Structure

The `app` directory is the modern way to structure Next.js applications.

### Basic App Router Structure

```
my-nextjs-app/
├── app/
│   ├── layout.tsx              # Root layout (required)
│   ├── page.tsx                # Home page (/)
│   ├── loading.tsx             # Loading UI for home
│   ├── error.tsx               # Error UI for home
│   ├── not-found.tsx           # 404 page
│   ├── global.css              # Global styles
│   │
│   ├── (auth)/                 # Route group (doesn't affect URL)
│   │   ├── layout.tsx          # Auth layout
│   │   ├── login/
│   │   │   ├── page.tsx        # /login
│   │   │   └── loading.tsx
│   │   └── register/
│   │       └── page.tsx        # /register
│   │
│   ├── (marketing)/            # Route group
│   │   ├── layout.tsx          # Marketing layout
│   │   ├── about/
│   │   │   └── page.tsx        # /about
│   │   ├── contact/
│   │   │   └── page.tsx        # /contact
│   │   └── pricing/
│   │       └── page.tsx        # /pricing
│   │
│   ├── blog/
│   │   ├── layout.tsx          # Blog layout
│   │   ├── page.tsx            # /blog
│   │   ├── loading.tsx
│   │   ├── error.tsx
│   │   ├── [slug]/
│   │   │   ├── page.tsx        # /blog/[slug]
│   │   │   ├── loading.tsx
│   │   │   └── not-found.tsx
│   │   └── categories/
│   │       └── [category]/
│   │           └── page.tsx    # /blog/categories/[category]
│   │
│   ├── dashboard/
│   │   ├── layout.tsx          # Dashboard layout
│   │   ├── page.tsx            # /dashboard
│   │   ├── loading.tsx
│   │   ├── error.tsx
│   │   ├── settings/
│   │   │   └── page.tsx        # /dashboard/settings
│   │   ├── analytics/
│   │   │   └── page.tsx        # /dashboard/analytics
│   │   └── profile/
│   │       └── page.tsx        # /dashboard/profile
│   │
│   ├── api/                    # API routes
│   │   ├── auth/
│   │   │   └── [...nextauth]/
│   │   │       └── route.ts    # /api/auth/[...nextauth]
│   │   ├── users/
│   │   │   ├── route.ts        # /api/users
│   │   │   └── [id]/
│   │   │       └── route.ts    # /api/users/[id]
│   │   └── posts/
│   │       └── route.ts        # /api/posts
│   │
│   ├── _components/            # Private folder (not routed)
│   │   ├── Header.tsx
│   │   ├── Footer.tsx
│   │   └── Sidebar.tsx
│   │
│   └── _lib/                   # Private folder
│       ├── auth.ts
│       └── utils.ts
│
├── components/                 # Shared components
│   ├── ui/
│   │   ├── Button.tsx
│   │   ├── Card.tsx
│   │   ├── Input.tsx
│   │   └── Modal.tsx
│   ├── forms/
│   │   ├── LoginForm.tsx
│   │   └── ContactForm.tsx
│   └── layout/
│       ├── Container.tsx
│       └── Section.tsx
│
├── lib/                        # Utility functions
│   ├── api.ts
│   ├── auth.ts
│   ├── db.ts
│   └── utils.ts
│
├── hooks/                      # Custom React hooks
│   ├── useAuth.ts
│   ├── useUser.ts
│   └── useDebounce.ts
│
├── types/                      # TypeScript types
│   ├── index.ts
│   ├── user.ts
│   └── post.ts
│
├── config/                     # Configuration files
│   ├── site.ts
│   └── navigation.ts
│
├── styles/                     # Additional styles
│   └── globals.css
│
├── public/                     # Static assets
│   ├── images/
│   ├── fonts/
│   └── favicon.ico
│
├── prisma/                     # Database schema (if using Prisma)
│   └── schema.prisma
│
├── .env.local                  # Environment variables
├── .eslintrc.json              # ESLint config
├── .gitignore
├── next.config.js              # Next.js config
├── package.json
├── tsconfig.json               # TypeScript config
├── tailwind.config.ts          # Tailwind CSS config (if using)
└── README.md
```

### App Router Special Files

| File            | Purpose                        | Required |
| --------------- | ------------------------------ | -------- |
| `layout.tsx`    | Shared UI for segment          | Yes\*    |
| `page.tsx`      | Route UI                       | Yes      |
| `loading.tsx`   | Loading UI (Suspense boundary) | No       |
| `error.tsx`     | Error UI (Error boundary)      | No       |
| `not-found.tsx` | Not found UI                   | No       |
| `template.tsx`  | Re-rendered layout             | No       |
| `default.tsx`   | Parallel route fallback        | No       |
| `route.ts`      | API endpoint                   | No       |

\*Required at root level

### App Router File Naming Conventions

```
app/
├── layout.tsx          # Layout component
├── page.tsx            # Page component
├── loading.tsx         # Loading component
├── error.tsx           # Error component
├── not-found.tsx       # Not found component
├── template.tsx        # Template component
├── default.tsx         # Default component
├── route.ts            # API route handler
├── middleware.ts       # Middleware
├── instrumentation.ts  # Instrumentation
├── global.css          # Global styles
└── icon.png            # App icon
```

---

## Pages Router Structure

The traditional `pages` directory structure (still supported).

### Basic Pages Router Structure

```
my-nextjs-app/
├── pages/
│   ├── _app.tsx                # Custom App component
│   ├── _document.tsx           # Custom Document
│   ├── _error.tsx              # Custom Error page
│   ├── index.tsx               # Home page (/)
│   ├── 404.tsx                 # Custom 404 page
│   ├── 500.tsx                 # Custom 500 page
│   │
│   ├── about.tsx               # /about
│   ├── contact.tsx             # /contact
│   │
│   ├── blog/
│   │   ├── index.tsx           # /blog
│   │   ├── [slug].tsx          # /blog/[slug]
│   │   └── categories/
│   │       └── [category].tsx  # /blog/categories/[category]
│   │
│   ├── dashboard/
│   │   ├── index.tsx           # /dashboard
│   │   ├── settings.tsx        # /dashboard/settings
│   │   └── profile.tsx         # /dashboard/profile
│   │
│   └── api/                    # API routes
│       ├── hello.ts            # /api/hello
│       ├── auth/
│       │   └── [...nextauth].ts
│       ├── users/
│       │   ├── index.ts        # /api/users
│       │   └── [id].ts         # /api/users/[id]
│       └── posts.ts            # /api/posts
│
├── components/                 # Shared components
│   ├── Layout.tsx
│   ├── Header.tsx
│   ├── Footer.tsx
│   └── ui/
│       ├── Button.tsx
│       └── Card.tsx
│
├── lib/                        # Utility functions
│   ├── api.ts
│   └── utils.ts
│
├── hooks/                      # Custom hooks
│   └── useAuth.ts
│
├── styles/                     # Styles
│   ├── globals.css
│   └── Home.module.css
│
├── public/                     # Static files
│   ├── images/
│   └── favicon.ico
│
├── types/                      # TypeScript types
│   └── index.ts
│
├── .env.local
├── next.config.js
├── package.json
└── tsconfig.json
```

### Pages Router Special Files

| File            | Purpose              | Required |
| --------------- | -------------------- | -------- |
| `_app.tsx`      | Custom App wrapper   | No       |
| `_document.tsx` | Custom HTML document | No       |
| `_error.tsx`    | Custom error page    | No       |
| `404.tsx`       | Custom 404 page      | No       |
| `500.tsx`       | Custom 500 page      | No       |
| `index.tsx`     | Route index page     | Yes\*    |

---

## Project Root Files

### Essential Configuration Files

```
my-nextjs-app/
├── .env.local                  # Local environment variables
├── .env.production             # Production environment variables
├── .env.development            # Development environment variables
├── .eslintrc.json              # ESLint configuration
├── .gitignore                  # Git ignore rules
├── .prettierrc                 # Prettier configuration
├── next.config.js              # Next.js configuration
├── next-env.d.ts               # Next.js TypeScript declarations
├── package.json                # Dependencies and scripts
├── package-lock.json           # Lock file
├── tsconfig.json               # TypeScript configuration
├── tailwind.config.ts          # Tailwind CSS config
├── postcss.config.js           # PostCSS config
├── middleware.ts               # Global middleware
└── README.md                   # Project documentation
```

### Example next.config.js

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "example.com",
      },
    ],
  },
  experimental: {
    serverActions: true,
  },
};

module.exports = nextConfig;
```

### Example tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./*"],
      "@/components/*": ["./components/*"],
      "@/lib/*": ["./lib/*"],
      "@/styles/*": ["./styles/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

---

## Public Directory

Static assets that are served from the root URL.

### Public Folder Structure

```
public/
├── images/                     # Image assets
│   ├── hero.jpg
│   ├── logo.png
│   └── avatars/
│       ├── user-1.jpg
│       └── user-2.jpg
│
├── fonts/                      # Custom fonts
│   ├── custom-font.woff2
│   └── custom-font.woff
│
├── icons/                      # Icon files
│   ├── icon-16x16.png
│   ├── icon-32x32.png
│   └── icon-192x192.png
│
├── favicon.ico                 # Favicon
├── apple-touch-icon.png        # Apple touch icon
├── manifest.json               # PWA manifest
├── robots.txt                  # Robots.txt for SEO
├── sitemap.xml                 # Sitemap for SEO
└── sw.js                       # Service worker (PWA)
```

### Usage Example

```tsx
import Image from "next/image";

export default function Hero() {
  return (
    <div>
      {/* Access files directly from public */}
      <Image src="/images/hero.jpg" alt="Hero" width={1200} height={600} />

      {/* Favicon is automatically loaded from public */}
    </div>
  );
}
```

---

## Components Organization

Best practices for organizing React components.

### Component Structure Options

#### Option 1: Flat Structure

```
components/
├── Button.tsx
├── Card.tsx
├── Header.tsx
├── Footer.tsx
├── Modal.tsx
└── Input.tsx
```

#### Option 2: Feature-based Structure (Recommended)

```
components/
├── ui/                         # Reusable UI components
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx
│   │   ├── Button.module.css
│   │   └── index.ts
│   ├── Card/
│   │   ├── Card.tsx
│   │   └── index.ts
│   ├── Input/
│   │   ├── Input.tsx
│   │   └── index.ts
│   └── Modal/
│       ├── Modal.tsx
│       └── index.ts
│
├── forms/                      # Form components
│   ├── LoginForm.tsx
│   ├── RegisterForm.tsx
│   ├── ContactForm.tsx
│   └── SearchForm.tsx
│
├── layout/                     # Layout components
│   ├── Header.tsx
│   ├── Footer.tsx
│   ├── Sidebar.tsx
│   ├── Container.tsx
│   └── Section.tsx
│
├── features/                   # Feature-specific components
│   ├── blog/
│   │   ├── BlogCard.tsx
│   │   ├── BlogList.tsx
│   │   └── BlogFilters.tsx
│   ├── dashboard/
│   │   ├── DashboardStats.tsx
│   │   └── DashboardChart.tsx
│   └── user/
│       ├── UserProfile.tsx
│       └── UserAvatar.tsx
│
└── shared/                     # Shared components
    ├── Loading.tsx
    ├── ErrorBoundary.tsx
    └── Pagination.tsx
```

#### Option 3: Atomic Design Structure

```
components/
├── atoms/                      # Basic building blocks
│   ├── Button.tsx
│   ├── Input.tsx
│   ├── Label.tsx
│   └── Icon.tsx
│
├── molecules/                  # Simple combinations
│   ├── FormField.tsx
│   ├── SearchBar.tsx
│   └── NavItem.tsx
│
├── organisms/                  # Complex components
│   ├── Header.tsx
│   ├── Footer.tsx
│   ├── LoginForm.tsx
│   └── ProductCard.tsx
│
├── templates/                  # Page templates
│   ├── MainLayout.tsx
│   ├── DashboardLayout.tsx
│   └── AuthLayout.tsx
│
└── pages/                      # Page-specific components
    ├── HomePage.tsx
    └── AboutPage.tsx
```

### Component Example with Colocation

```
components/
└── ui/
    └── Button/
        ├── Button.tsx              # Component
        ├── Button.test.tsx         # Tests
        ├── Button.stories.tsx      # Storybook stories
        ├── Button.module.css       # Styles
        ├── Button.types.ts         # TypeScript types
        ├── useButton.ts            # Custom hook
        └── index.ts                # Barrel export
```

```tsx
// components/ui/Button/Button.tsx
import styles from "./Button.module.css";
import { ButtonProps } from "./Button.types";

export function Button({ children, variant = "primary", ...props }: ButtonProps) {
  return (
    <button className={`${styles.button} ${styles[variant]}`} {...props}>
      {children}
    </button>
  );
}
```

```ts
// components/ui/Button/index.ts
export { Button } from "./Button";
export type { ButtonProps } from "./Button.types";
```

---

## Lib and Utils

Utility functions and shared logic.

### Lib Folder Structure

```
lib/
├── api/                        # API functions
│   ├── client.ts               # API client setup
│   ├── users.ts                # User API calls
│   ├── posts.ts                # Post API calls
│   └── auth.ts                 # Auth API calls
│
├── db/                         # Database utilities
│   ├── prisma.ts               # Prisma client
│   ├── queries.ts              # Database queries
│   └── migrations/
│
├── auth/                       # Authentication
│   ├── config.ts               # Auth config
│   ├── session.ts              # Session management
│   └── middleware.ts           # Auth middleware
│
├── validations/                # Validation schemas
│   ├── user.ts                 # User validation
│   ├── post.ts                 # Post validation
│   └── auth.ts                 # Auth validation
│
├── constants.ts                # App constants
├── utils.ts                    # Utility functions
├── hooks.ts                    # Custom hooks
└── types.ts                    # Shared types
```

### Example Utils File

```ts
// lib/utils.ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

export function formatDate(date: Date): string {
  return new Intl.DateTimeFormat("en-US", {
    month: "long",
    day: "numeric",
    year: "numeric",
  }).format(date);
}

export function slugify(text: string): string {
  return text
    .toLowerCase()
    .replace(/[^\w\s-]/g, "")
    .replace(/[\s_-]+/g, "-")
    .replace(/^-+|-+$/g, "");
}
```

### Example API Client

```ts
// lib/api/client.ts
const API_BASE_URL = process.env.NEXT_PUBLIC_API_URL || "http://localhost:3000/api";

export async function fetcher<T>(endpoint: string, options?: RequestInit): Promise<T> {
  const response = await fetch(`${API_BASE_URL}${endpoint}`, {
    headers: {
      "Content-Type": "application/json",
      ...options?.headers,
    },
    ...options,
  });

  if (!response.ok) {
    throw new Error(`API Error: ${response.statusText}`);
  }

  return response.json();
}
```

---

## Styles Organization

Different approaches to organizing styles in Next.js.

### CSS Modules Structure

```
app/
├── page.tsx
├── page.module.css
├── blog/
│   ├── page.tsx
│   ├── page.module.css
│   └── [slug]/
│       ├── page.tsx
│       └── page.module.css
│
styles/
├── globals.css                 # Global styles
├── variables.css               # CSS variables
├── typography.css              # Typography
└── utilities.css               # Utility classes
```

### Tailwind CSS Structure

```
app/
├── globals.css                 # Tailwind imports
└── page.tsx

styles/
├── globals.css
└── tailwind/
    ├── base.css
    ├── components.css
    └── utilities.css

tailwind.config.ts
```

```css
/* app/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  h1 {
    @apply text-4xl font-bold;
  }
}

@layer components {
  .btn-primary {
    @apply bg-blue-500 text-white px-4 py-2 rounded;
  }
}
```

### CSS-in-JS Structure (Styled Components)

```
components/
└── ui/
    └── Button/
        ├── Button.tsx
        ├── Button.styles.ts
        └── index.ts
```

```tsx
// components/ui/Button/Button.styles.ts
import styled from "styled-components";

export const StyledButton = styled.button`
  padding: 0.5rem 1rem;
  background-color: ${(props) => props.theme.colors.primary};
  color: white;
  border-radius: 0.25rem;

  &:hover {
    opacity: 0.9;
  }
`;
```

---

## API Routes Structure

Organizing API routes in Next.js.

### App Router API Structure

```
app/
└── api/
    ├── auth/
    │   ├── login/
    │   │   └── route.ts            # POST /api/auth/login
    │   ├── logout/
    │   │   └── route.ts            # POST /api/auth/logout
    │   └── [...nextauth]/
    │       └── route.ts            # NextAuth endpoints
    │
    ├── users/
    │   ├── route.ts                # GET, POST /api/users
    │   ├── [id]/
    │   │   └── route.ts            # GET, PUT, DELETE /api/users/[id]
    │   └── me/
    │       └── route.ts            # GET /api/users/me
    │
    ├── posts/
    │   ├── route.ts                # GET, POST /api/posts
    │   ├── [id]/
    │   │   ├── route.ts            # GET, PUT, DELETE /api/posts/[id]
    │   │   └── comments/
    │   │       └── route.ts        # GET, POST /api/posts/[id]/comments
    │   └── search/
    │       └── route.ts            # GET /api/posts/search
    │
    ├── upload/
    │   └── route.ts                # POST /api/upload
    │
    └── webhook/
        ├── stripe/
        │   └── route.ts            # POST /api/webhook/stripe
        └── github/
            └── route.ts            # POST /api/webhook/github
```

### API Route Example

```ts
// app/api/users/[id]/route.ts
import { NextRequest, NextResponse } from "next/server";

// GET /api/users/[id]
export async function GET(request: NextRequest, { params }: { params: { id: string } }) {
  try {
    const user = await getUserById(params.id);

    if (!user) {
      return NextResponse.json({ error: "User not found" }, { status: 404 });
    }

    return NextResponse.json(user);
  } catch (error) {
    return NextResponse.json({ error: "Internal server error" }, { status: 500 });
  }
}

// PUT /api/users/[id]
export async function PUT(request: NextRequest, { params }: { params: { id: string } }) {
  try {
    const body = await request.json();
    const updatedUser = await updateUser(params.id, body);

    return NextResponse.json(updatedUser);
  } catch (error) {
    return NextResponse.json({ error: "Failed to update user" }, { status: 500 });
  }
}

// DELETE /api/users/[id]
export async function DELETE(request: NextRequest, { params }: { params: { id: string } }) {
  try {
    await deleteUser(params.id);

    return NextResponse.json({ message: "User deleted successfully" }, { status: 200 });
  } catch (error) {
    return NextResponse.json({ error: "Failed to delete user" }, { status: 500 });
  }
}
```

---

## TypeScript Configuration

TypeScript organization and configuration.

### Types Folder Structure

```
types/
├── index.ts                    # Main types export
├── api.ts                      # API types
├── database.ts                 # Database types
├── user.ts                     # User types
├── post.ts                     # Post types
├── auth.ts                     # Auth types
└── utils.ts                    # Utility types
```

### Example Type Definitions

```ts
// types/user.ts
export interface User {
  id: string;
  email: string;
  name: string;
  avatar?: string;
  role: UserRole;
  createdAt: Date;
  updatedAt: Date;
}

export type UserRole = "admin" | "user" | "guest";

export interface CreateUserInput {
  email: string;
  name: string;
  password: string;
}

export interface UpdateUserInput {
  name?: string;
  avatar?: string;
}
```

```ts
// types/api.ts
export interface ApiResponse<T> {
  data: T;
  message?: string;
  success: boolean;
}

export interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}

export interface ApiError {
  error: string;
  statusCode: number;
  details?: unknown;
}
```

---

## Best Practices

### 1. **Use Route Groups for Organization**

```
app/
├── (marketing)/
│   ├── layout.tsx
│   ├── page.tsx
│   └── about/
│       └── page.tsx
└── (app)/
    ├── layout.tsx
    └── dashboard/
        └── page.tsx
```

### 2. **Colocate Related Files**

```
app/
└── dashboard/
    ├── page.tsx
    ├── loading.tsx
    ├── error.tsx
    ├── _components/
    │   ├── StatsCard.tsx
    │   └── Chart.tsx
    └── _lib/
        └── utils.ts
```

### 3. **Use Private Folders**

Prefix folders with `_` to exclude from routing:

```
app/
├── _components/               # Not a route
├── _lib/                      # Not a route
└── page.tsx                   # Route
```

### 4. **Organize by Feature**

```
app/
├── (blog)/
│   ├── blog/
│   │   ├── page.tsx
│   │   ├── _components/
│   │   └── _lib/
│   └── authors/
│       ├── page.tsx
│       └── [id]/
│           └── page.tsx
```

### 5. **Path Aliases**

Use path aliases in `tsconfig.json`:

```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./*"],
      "@/components/*": ["./components/*"],
      "@/lib/*": ["./lib/*"],
      "@/app/*": ["./app/*"]
    }
  }
}
```

```tsx
// Instead of
import { Button } from "../../../components/ui/Button";

// Use
import { Button } from "@/components/ui/Button";
```

### 6. **Consistent Naming Conventions**

- **Components**: PascalCase (`UserProfile.tsx`)
- **Utils/Hooks**: camelCase (`useAuth.ts`, `formatDate.ts`)
- **Constants**: UPPER_SNAKE_CASE (`API_BASE_URL`)
- **Folders**: kebab-case or camelCase (`user-profile/` or `userProfile/`)

### 7. **Separate Server and Client Code**

```
lib/
├── server/                    # Server-only code
│   ├── db.ts
│   └── auth.ts
└── client/                    # Client-only code
    ├── api.ts
    └── hooks.ts
```

### 8. **Environment Variables**

```
.env.local                     # Local development
.env.development               # Development
.env.production                # Production
.env.test                      # Testing
```

```
# Database
DATABASE_URL=postgresql://...

# Auth
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your-secret-key

# API Keys
NEXT_PUBLIC_API_URL=https://api.example.com
```

### 9. **Testing Structure**

```
app/
└── dashboard/
    ├── page.tsx
    └── page.test.tsx

components/
└── ui/
    └── Button/
        ├── Button.tsx
        └── Button.test.tsx

__tests__/
├── integration/
│   └── dashboard.test.tsx
└── e2e/
    └── login.spec.ts
```

### 10. **Documentation**

```
docs/
├── api/
│   ├── authentication.md
│   └── endpoints.md
├── components/
│   └── button.md
├── architecture.md
└── deployment.md
```

---

## Real-World Examples

### E-commerce Application

```
my-ecommerce-app/
├── app/
│   ├── (shop)/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── products/
│   │   │   ├── page.tsx
│   │   │   ├── [id]/
│   │   │   │   └── page.tsx
│   │   │   └── category/
│   │   │       └── [slug]/
│   │   │           └── page.tsx
│   │   ├── cart/
│   │   │   └── page.tsx
│   │   └── checkout/
│   │       ├── page.tsx
│   │       ├── shipping/
│   │       │   └── page.tsx
│   │       └── payment/
│   │           └── page.tsx
│   │
│   ├── (account)/
│   │   ├── layout.tsx
│   │   ├── login/
│   │   │   └── page.tsx
│   │   ├── register/
│   │   │   └── page.tsx
│   │   └── profile/
│   │       ├── page.tsx
│   │       ├── orders/
│   │       │   └── page.tsx
│   │       └── settings/
│   │           └── page.tsx
│   │
│   ├── (admin)/
│   │   ├── layout.tsx
│   │   └── admin/
│   │       ├── page.tsx
│   │       ├── products/
│   │       │   └── page.tsx
│   │       ├── orders/
│   │       │   └── page.tsx
│   │       └── users/
│   │           └── page.tsx
│   │
│   └── api/
│       ├── products/
│       │   └── route.ts
│       ├── cart/
│       │   └── route.ts
│       ├── orders/
│       │   └── route.ts
│       └── payment/
│           └── webhook/
│               └── route.ts
│
├── components/
│   ├── products/
│   │   ├── ProductCard.tsx
│   │   ├── ProductList.tsx
│   │   └── ProductFilters.tsx
│   ├── cart/
│   │   ├── CartItem.tsx
│   │   └── CartSummary.tsx
│   └── ui/
│       ├── Button.tsx
│       └── Modal.tsx
│
├── lib/
│   ├── stripe.ts
│   ├── prisma.ts
│   └── auth.ts
│
└── types/
    ├── product.ts
    ├── order.ts
    └── user.ts
```

### SaaS Dashboard Application

```
my-saas-app/
├── app/
│   ├── (marketing)/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── pricing/
│   │   │   └── page.tsx
│   │   └── about/
│   │       └── page.tsx
│   │
│   ├── (auth)/
│   │   ├── login/
│   │   │   └── page.tsx
│   │   └── register/
│   │       └── page.tsx
│   │
│   ├── (dashboard)/
│   │   ├── layout.tsx
│   │   └── dashboard/
│   │       ├── page.tsx
│   │       ├── projects/
│   │       │   ├── page.tsx
│   │       │   └── [id]/
│   │       │       └── page.tsx
│   │       ├── analytics/
│   │       │   └── page.tsx
│   │       ├── settings/
│   │       │   ├── page.tsx
│   │       │   ├── profile/
│   │       │   │   └── page.tsx
│   │       │   └── billing/
│   │       │       └── page.tsx
│   │       └── team/
│   │           └── page.tsx
│   │
│   └── api/
│       ├── auth/
│       │   └── [...nextauth]/
│       │       └── route.ts
│       ├── projects/
│       │   └── route.ts
│       ├── analytics/
│       │   └── route.ts
│       └── stripe/
│           └── webhook/
│               └── route.ts
│
├── components/
│   ├── dashboard/
│   │   ├── Sidebar.tsx
│   │   ├── StatsCard.tsx
│   │   └── Chart.tsx
│   ├── projects/
│   │   ├── ProjectCard.tsx
│   │   └── ProjectList.tsx
│   └── ui/
│       └── ...
│
├── lib/
│   ├── auth.ts
│   ├── stripe.ts
│   └── analytics.ts
│
└── types/
    ├── project.ts
    ├── analytics.ts
    └── user.ts
```

### Blog/Content Platform

```
my-blog-app/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   │
│   ├── blog/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── [slug]/
│   │   │   ├── page.tsx
│   │   │   └── not-found.tsx
│   │   ├── category/
│   │   │   └── [slug]/
│   │   │       └── page.tsx
│   │   ├── tag/
│   │   │   └── [slug]/
│   │   │       └── page.tsx
│   │   └── author/
│   │       └── [slug]/
│   │           └── page.tsx
│   │
│   ├── about/
│   │   └── page.tsx
│   │
│   └── api/
│       ├── posts/
│       │   └── route.ts
│       └── comments/
│           └── route.ts
│
├── components/
│   ├── blog/
│   │   ├── PostCard.tsx
│   │   ├── PostList.tsx
│   │   ├── PostContent.tsx
│   │   └── CommentSection.tsx
│   └── ui/
│       └── ...
│
├── lib/
│   ├── mdx.ts
│   ├── posts.ts
│   └── markdown.ts
│
├── content/
│   ├── posts/
│   │   ├── post-1.mdx
│   │   └── post-2.mdx
│   └── authors/
│       └── john-doe.json
│
└── types/
    ├── post.ts
    └── author.ts
```

---

## Summary

### Key Principles

✅ **File-based Routing**

- Folders define routes
- Special files (`page.tsx`, `layout.tsx`) have specific purposes
- Route groups `()` organize without affecting URLs

✅ **Colocation**

- Keep related files together
- Use private folders (`_folder`) for non-route files
- Organize by feature when possible

✅ **Separation of Concerns**

- Components for UI
- Lib for utilities and business logic
- Types for TypeScript definitions
- API routes for backend logic

✅ **Scalability**

- Use consistent naming conventions
- Implement path aliases
- Organize by feature for large apps
- Keep configuration files at root

### Quick Reference

| Directory       | Purpose                      | Example                 |
| --------------- | ---------------------------- | ----------------------- |
| `app/`          | App Router pages and layouts | `app/page.tsx`          |
| `pages/`        | Pages Router (legacy)        | `pages/index.tsx`       |
| `components/`   | Reusable UI components       | `components/Button.tsx` |
| `lib/`          | Utility functions and logic  | `lib/utils.ts`          |
| `types/`        | TypeScript type definitions  | `types/user.ts`         |
| `public/`       | Static assets                | `public/logo.png`       |
| `styles/`       | Global and shared styles     | `styles/globals.css`    |
| `hooks/`        | Custom React hooks           | `hooks/useAuth.ts`      |
| `config/`       | Configuration files          | `config/site.ts`        |
| `middleware.ts` | Global middleware            | Root level              |

---

## Resources

- **Official Docs:** [nextjs.org/docs/getting-started/project-structure](https://nextjs.org/docs/getting-started/project-structure)
- **App Router:** [nextjs.org/docs/app](https://nextjs.org/docs/app)
- **Pages Router:** [nextjs.org/docs/pages](https://nextjs.org/docs/pages)
- **Examples:** [github.com/vercel/next.js/tree/canary/examples](https://github.com/vercel/next.js/tree/canary/examples)
