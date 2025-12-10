# Next.js 16 - Server and Client Components Pattern Guide

## Table of Contents

1. [Introduction](#introduction)
2. [Understanding Component Types](#understanding-component-types)
3. [The Wrapping Pattern](#the-wrapping-pattern)
4. [How It Works](#how-it-works)
5. [Common Use Cases](#common-use-cases)
6. [Advanced Patterns & Tricks](#advanced-patterns--tricks)
7. [Best Practices](#best-practices)
8. [Common Mistakes](#common-mistakes)
9. [Real-World Examples](#real-world-examples)
10. [Performance Optimization](#performance-optimization)

---

## Introduction

One of the most powerful patterns in Next.js 16's App Router is the ability to **wrap Server Components inside Client Components** using the `children` prop. This pattern allows you to maintain the benefits of Server Components while adding client-side interactivity where needed.

### Key Concept

> [!IMPORTANT]
> You **cannot** import a Server Component directly into a Client Component, but you **can** pass a Server Component as `children` or props to a Client Component.

---

## Understanding Component Types

### Server Components (Default)

Server Components run only on the server and are the default in Next.js 16 App Router.

```tsx
// app/components/ServerComponent.tsx
// This is a Server Component by default (no 'use client')

export default async function ServerComponent() {
  // Can fetch data directly
  const data = await fetch('https://api.example.com/data');
  const result = await data.json();

  return (
    <div>
      <h2>Server Component</h2>
      <p>Data: {result.message}</p>
    </div>
  );
}
```

**Benefits:**
- Direct database access
- Secure API calls (keys stay on server)
- Smaller JavaScript bundle
- Better SEO

### Client Components

Client Components run on both server (for initial render) and client (for interactivity).

```tsx
// app/components/ClientComponent.tsx
'use client';

import { useState } from 'react';

export default function ClientComponent() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <h2>Client Component</h2>
      <button onClick={() => setCount(count + 1)}>
        Count: {count}
      </button>
    </div>
  );
}
```

**Benefits:**
- Interactive UI (onClick, onChange, etc.)
- React hooks (useState, useEffect, etc.)
- Browser APIs (localStorage, window, etc.)

---

## The Wrapping Pattern

### ❌ What Doesn't Work

You **cannot** import a Server Component directly into a Client Component:

```tsx
// ❌ This will NOT work!
'use client';

import ServerComponent from './ServerComponent'; // Error!

export default function ClientWrapper() {
  return (
    <div>
      <ServerComponent /> {/* This breaks! */}
    </div>
  );
}
```

**Why?** Once you mark a component with `'use client'`, everything it imports becomes part of the client bundle, including Server Components.

### ✅ What Works - The Children Pattern

Pass Server Components as `children` or props:

```tsx
// app/components/ClientWrapper.tsx
'use client';

export default function ClientWrapper({ children }: { children: React.ReactNode }) {
  return (
    <div className="wrapper">
      <h1>Client Wrapper</h1>
      {children}
    </div>
  );
}
```

```tsx
// app/page.tsx (Server Component)
import ClientWrapper from './components/ClientWrapper';
import ServerComponent from './components/ServerComponent';

export default function Page() {
  return (
    <ClientWrapper>
      <ServerComponent /> {/* ✅ This works! */}
    </ClientWrapper>
  );
}
```

---

## How It Works

### The React Element Tree

When you pass a Server Component as `children`, React creates the component tree like this:

```
1. Server renders ServerComponent → Creates React Element
2. Passes that element to ClientWrapper as children prop
3. ClientWrapper receives pre-rendered element (not the component itself)
4. Client hydrates the wrapper, but children stay server-rendered
```

### Visual Representation

```tsx
// Server Side:
const serverElement = <ServerComponent />; // Rendered on server

// Passed to Client:
<ClientWrapper>
  {serverElement} // Already rendered, just passed through
</ClientWrapper>
```

### What Gets Sent to Client

```javascript
// Simplified representation
{
  type: ClientWrapper,
  props: {
    children: {
      // Pre-rendered Server Component output
      type: 'div',
      props: {
        children: ['Server Component', 'Data: ...']
      }
    }
  }
}
```

---

## Common Use Cases

### 1. Interactive Wrapper Around Server Data

```tsx
// app/components/CollapsibleSection.tsx
'use client';

import { useState } from 'react';

export default function CollapsibleSection({ 
  title, 
  children 
}: { 
  title: string; 
  children: React.ReactNode 
}) {
  const [isOpen, setIsOpen] = useState(true);

  return (
    <div className="collapsible">
      <button onClick={() => setIsOpen(!isOpen)}>
        {isOpen ? '▼' : '▶'} {title}
      </button>
      {isOpen && <div className="content">{children}</div>}
    </div>
  );
}
```

```tsx
// app/page.tsx
import CollapsibleSection from './components/CollapsibleSection';

async function UserList() {
  const users = await fetch('https://api.example.com/users');
  const data = await users.json();

  return (
    <ul>
      {data.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}

export default function Page() {
  return (
    <CollapsibleSection title="Users">
      <UserList /> {/* Server Component with data fetching */}
    </CollapsibleSection>
  );
}
```

### 2. Tabs with Server-Rendered Content

```tsx
// app/components/Tabs.tsx
'use client';

import { useState } from 'react';

export default function Tabs({ 
  tabs 
}: { 
  tabs: { label: string; content: React.ReactNode }[] 
}) {
  const [activeTab, setActiveTab] = useState(0);

  return (
    <div>
      <div className="tab-buttons">
        {tabs.map((tab, index) => (
          <button
            key={index}
            onClick={() => setActiveTab(index)}
            className={activeTab === index ? 'active' : ''}
          >
            {tab.label}
          </button>
        ))}
      </div>
      <div className="tab-content">
        {tabs[activeTab].content}
      </div>
    </div>
  );
}
```

```tsx
// app/page.tsx
import Tabs from './components/Tabs';

async function ProfileTab() {
  const profile = await fetch('https://api.example.com/profile');
  const data = await profile.json();
  return <div>Profile: {data.name}</div>;
}

async function SettingsTab() {
  const settings = await fetch('https://api.example.com/settings');
  const data = await settings.json();
  return <div>Settings: {JSON.stringify(data)}</div>;
}

export default function Page() {
  return (
    <Tabs
      tabs={[
        { label: 'Profile', content: <ProfileTab /> },
        { label: 'Settings', content: <SettingsTab /> },
      ]}
    />
  );
}
```

### 3. Modal with Server Content

```tsx
// app/components/Modal.tsx
'use client';

import { useState } from 'react';

export default function Modal({ 
  trigger, 
  children 
}: { 
  trigger: React.ReactNode; 
  children: React.ReactNode 
}) {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <div onClick={() => setIsOpen(true)}>{trigger}</div>
      {isOpen && (
        <div className="modal-overlay" onClick={() => setIsOpen(false)}>
          <div className="modal-content" onClick={(e) => e.stopPropagation()}>
            {children}
            <button onClick={() => setIsOpen(false)}>Close</button>
          </div>
        </div>
      )}
    </>
  );
}
```

```tsx
// app/page.tsx
import Modal from './components/Modal';

async function ProductDetails({ id }: { id: string }) {
  const product = await fetch(`https://api.example.com/products/${id}`);
  const data = await product.json();
  
  return (
    <div>
      <h2>{data.name}</h2>
      <p>{data.description}</p>
      <p>Price: ${data.price}</p>
    </div>
  );
}

export default function Page() {
  return (
    <Modal trigger={<button>View Product</button>}>
      <ProductDetails id="123" />
    </Modal>
  );
}
```

---

## Advanced Patterns & Tricks

### Trick 1: Context Provider Pattern

Wrap Server Components with Client Context providers without making them client components.

```tsx
// app/components/ThemeProvider.tsx
'use client';

import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext({ theme: 'light', toggleTheme: () => {} });

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => setTheme(theme === 'light' ? 'dark' : 'light');

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      <div data-theme={theme}>{children}</div>
    </ThemeContext.Provider>
  );
}

export const useTheme = () => useContext(ThemeContext);
```

```tsx
// app/layout.tsx (Server Component)
import { ThemeProvider } from './components/ThemeProvider';
import ServerContent from './components/ServerContent';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <ThemeProvider>
          {/* All children remain Server Components */}
          {children}
          <ServerContent />
        </ThemeProvider>
      </body>
    </html>
  );
}
```

**Why it works:** The provider is a Client Component, but `children` are passed from the parent Server Component, so they remain server-rendered.

---

### Trick 2: Conditional Rendering with Server Components

Use client-side state to conditionally show Server Components.

```tsx
// app/components/ConditionalContent.tsx
'use client';

import { useState } from 'react';

export default function ConditionalContent({
  serverContent,
  fallback
}: {
  serverContent: React.ReactNode;
  fallback: React.ReactNode;
}) {
  const [showContent, setShowContent] = useState(false);

  return (
    <div>
      <button onClick={() => setShowContent(!showContent)}>
        {showContent ? 'Hide' : 'Show'} Content
      </button>
      {showContent ? serverContent : fallback}
    </div>
  );
}
```

```tsx
// app/page.tsx
import ConditionalContent from './components/ConditionalContent';

async function ExpensiveServerComponent() {
  const data = await fetch('https://api.example.com/expensive-data');
  const result = await data.json();
  return <div>Expensive Data: {JSON.stringify(result)}</div>;
}

export default function Page() {
  return (
    <ConditionalContent
      serverContent={<ExpensiveServerComponent />}
      fallback={<div>Click to load data</div>}
    />
  );
}
```

---

### Trick 3: Slot Pattern for Complex Layouts

Create flexible layouts with multiple named slots.

```tsx
// app/components/DashboardLayout.tsx
'use client';

import { useState } from 'react';

export default function DashboardLayout({
  header,
  sidebar,
  main,
  footer,
  modal
}: {
  header: React.ReactNode;
  sidebar: React.ReactNode;
  main: React.ReactNode;
  footer: React.ReactNode;
  modal?: React.ReactNode;
}) {
  const [sidebarOpen, setSidebarOpen] = useState(true);

  return (
    <div className="dashboard-layout">
      <header className="header">{header}</header>
      <div className="content">
        {sidebarOpen && <aside className="sidebar">{sidebar}</aside>}
        <main className="main">{main}</main>
      </div>
      <footer className="footer">{footer}</footer>
      {modal}
      <button onClick={() => setSidebarOpen(!sidebarOpen)}>
        Toggle Sidebar
      </button>
    </div>
  );
}
```

```tsx
// app/dashboard/page.tsx
import DashboardLayout from '../components/DashboardLayout';

async function DashboardHeader() {
  const user = await getCurrentUser();
  return <div>Welcome, {user.name}</div>;
}

async function DashboardSidebar() {
  const nav = await getNavigation();
  return <nav>{/* nav items */}</nav>;
}

async function DashboardMain() {
  const stats = await getStats();
  return <div>{/* stats */}</div>;
}

export default function Dashboard() {
  return (
    <DashboardLayout
      header={<DashboardHeader />}
      sidebar={<DashboardSidebar />}
      main={<DashboardMain />}
      footer={<div>© 2024</div>}
    />
  );
}
```

---

### Trick 4: Render Props Pattern

Pass Server Components through render props for maximum flexibility.

```tsx
// app/components/DataRenderer.tsx
'use client';

import { useState } from 'react';

export default function DataRenderer({
  renderData,
  renderLoading,
  renderError
}: {
  renderData: (props: { refresh: () => void }) => React.ReactNode;
  renderLoading: React.ReactNode;
  renderError: React.ReactNode;
}) {
  const [key, setKey] = useState(0);
  const [status, setStatus] = useState<'loading' | 'success' | 'error'>('success');

  const refresh = () => {
    setStatus('loading');
    setKey(prev => prev + 1);
    setTimeout(() => setStatus('success'), 1000);
  };

  if (status === 'loading') return <>{renderLoading}</>;
  if (status === 'error') return <>{renderError}</>;

  return <div key={key}>{renderData({ refresh })}</div>;
}
```

```tsx
// app/page.tsx
import DataRenderer from './components/DataRenderer';

async function DataContent({ refresh }: { refresh: () => void }) {
  const data = await fetch('https://api.example.com/data');
  const result = await data.json();
  
  return (
    <div>
      <p>{result.message}</p>
      <button onClick={refresh}>Refresh</button>
    </div>
  );
}

export default function Page() {
  return (
    <DataRenderer
      renderData={(props) => <DataContent {...props} />}
      renderLoading={<div>Loading...</div>}
      renderError={<div>Error occurred</div>}
    />
  );
}
```

---

### Trick 5: HOC Pattern for Server Components

Create Higher-Order Components that wrap Server Components.

```tsx
// app/components/withErrorBoundary.tsx
'use client';

import { Component, ReactNode } from 'react';

class ErrorBoundary extends Component<
  { children: ReactNode; fallback: ReactNode },
  { hasError: boolean }
> {
  constructor(props: any) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback;
    }
    return this.props.children;
  }
}

export function withErrorBoundary(
  children: ReactNode,
  fallback: ReactNode = <div>Something went wrong</div>
) {
  return <ErrorBoundary fallback={fallback}>{children}</ErrorBoundary>;
}
```

```tsx
// app/page.tsx
import { withErrorBoundary } from './components/withErrorBoundary';

async function RiskyServerComponent() {
  const data = await fetch('https://api.example.com/risky');
  const result = await data.json();
  return <div>{result.data}</div>;
}

export default function Page() {
  return withErrorBoundary(
    <RiskyServerComponent />,
    <div>Failed to load data</div>
  );
}
```

---

### Trick 6: Intersection Observer Pattern

Lazy load Server Components when they enter viewport.

```tsx
// app/components/LazyLoad.tsx
'use client';

import { useEffect, useRef, useState } from 'react';

export default function LazyLoad({
  children,
  placeholder = <div>Loading...</div>
}: {
  children: React.ReactNode;
  placeholder?: React.ReactNode;
}) {
  const [isVisible, setIsVisible] = useState(false);
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsVisible(true);
          observer.disconnect();
        }
      },
      { threshold: 0.1 }
    );

    if (ref.current) {
      observer.observe(ref.current);
    }

    return () => observer.disconnect();
  }, []);

  return <div ref={ref}>{isVisible ? children : placeholder}</div>;
}
```

```tsx
// app/page.tsx
import LazyLoad from './components/LazyLoad';

async function HeavyServerComponent() {
  const data = await fetch('https://api.example.com/heavy-data');
  const result = await data.json();
  return <div>{/* Heavy content */}</div>;
}

export default function Page() {
  return (
    <div>
      <div style={{ height: '100vh' }}>Scroll down...</div>
      <LazyLoad placeholder={<div>Scroll to load...</div>}>
        <HeavyServerComponent />
      </LazyLoad>
    </div>
  );
}
```

---

### Trick 7: Streaming with Suspense Boundaries

Combine Client Components with Suspense for progressive rendering.

```tsx
// app/components/StreamingContainer.tsx
'use client';

import { Suspense } from 'react';

export default function StreamingContainer({
  sections
}: {
  sections: { title: string; content: React.ReactNode; fallback: React.ReactNode }[];
}) {
  return (
    <div className="streaming-container">
      {sections.map((section, index) => (
        <div key={index} className="section">
          <h2>{section.title}</h2>
          <Suspense fallback={section.fallback}>
            {section.content}
          </Suspense>
        </div>
      ))}
    </div>
  );
}
```

```tsx
// app/page.tsx
import StreamingContainer from './components/StreamingContainer';

async function FastContent() {
  const data = await fetch('https://api.example.com/fast');
  const result = await data.json();
  return <div>Fast: {result.data}</div>;
}

async function SlowContent() {
  const data = await fetch('https://api.example.com/slow');
  const result = await data.json();
  return <div>Slow: {result.data}</div>;
}

export default function Page() {
  return (
    <StreamingContainer
      sections={[
        {
          title: 'Fast Section',
          content: <FastContent />,
          fallback: <div>Loading fast content...</div>
        },
        {
          title: 'Slow Section',
          content: <SlowContent />,
          fallback: <div>Loading slow content...</div>
        }
      ]}
    />
  );
}
```

---

### Trick 8: Form with Server Actions

Combine Client Component forms with Server Actions.

```tsx
// app/components/InteractiveForm.tsx
'use client';

import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : 'Submit'}
    </button>
  );
}

export default function InteractiveForm({
  action,
  children
}: {
  action: (formData: FormData) => Promise<void>;
  children: React.ReactNode;
}) {
  return (
    <form action={action}>
      {children}
      <SubmitButton />
    </form>
  );
}
```

```tsx
// app/page.tsx
import InteractiveForm from './components/InteractiveForm';
import { createUser } from './actions';

async function FormFields() {
  const countries = await fetch('https://api.example.com/countries');
  const data = await countries.json();
  
  return (
    <>
      <input name="name" placeholder="Name" required />
      <input name="email" type="email" placeholder="Email" required />
      <select name="country">
        {data.map((country: any) => (
          <option key={country.code} value={country.code}>
            {country.name}
          </option>
        ))}
      </select>
    </>
  );
}

export default function Page() {
  return (
    <InteractiveForm action={createUser}>
      <FormFields />
    </InteractiveForm>
  );
}
```

---

### Trick 9: Virtualized List with Server Items

Create virtualized lists where items are Server Components.

```tsx
// app/components/VirtualList.tsx
'use client';

import { useState } from 'react';

export default function VirtualList({
  items,
  itemHeight = 100
}: {
  items: React.ReactNode[];
  itemHeight?: number;
}) {
  const [scrollTop, setScrollTop] = useState(0);
  const containerHeight = 600;
  
  const startIndex = Math.floor(scrollTop / itemHeight);
  const endIndex = Math.min(
    startIndex + Math.ceil(containerHeight / itemHeight) + 1,
    items.length
  );
  
  const visibleItems = items.slice(startIndex, endIndex);
  const offsetY = startIndex * itemHeight;

  return (
    <div
      style={{ height: containerHeight, overflow: 'auto' }}
      onScroll={(e) => setScroll Top(e.currentTarget.scrollTop)}
    >
      <div style={{ height: items.length * itemHeight, position: 'relative' }}>
        <div style={{ transform: `translateY(${offsetY}px)` }}>
          {visibleItems}
        </div>
      </div>
    </div>
  );
}
```

```tsx
// app/page.tsx
import VirtualList from './components/VirtualList';

async function UserItem({ id }: { id: number }) {
  const user = await fetch(`https://api.example.com/users/${id}`);
  const data = await user.json();
  
  return (
    <div style={{ height: 100, padding: 10, borderBottom: '1px solid #ccc' }}>
      <h3>{data.name}</h3>
      <p>{data.email}</p>
    </div>
  );
}

export default async function Page() {
  const users = await fetch('https://api.example.com/users');
  const data = await users.json();
  
  return (
    <VirtualList
      items={data.map((user: any) => (
        <UserItem key={user.id} id={user.id} />
      ))}
      itemHeight={100}
    />
  );
}
```

---

### Trick 10: Drag and Drop with Server Components

Create drag-and-drop interfaces with Server Component items.

```tsx
// app/components/DraggableList.tsx
'use client';

import { useState } from 'react';

export default function DraggableList({
  items
}: {
  items: { id: string; content: React.ReactNode }[];
}) {
  const [orderedItems, setOrderedItems] = useState(items);
  const [draggedId, setDraggedId] = useState<string | null>(null);

  const handleDragStart = (id: string) => setDraggedId(id);
  
  const handleDragOver = (e: React.DragEvent, targetId: string) => {
    e.preventDefault();
    if (!draggedId || draggedId === targetId) return;

    const draggedIndex = orderedItems.findIndex(item => item.id === draggedId);
    const targetIndex = orderedItems.findIndex(item => item.id === targetId);
    
    const newItems = [...orderedItems];
    const [removed] = newItems.splice(draggedIndex, 1);
    newItems.splice(targetIndex, 0, removed);
    
    setOrderedItems(newItems);
  };

  return (
    <div>
      {orderedItems.map((item) => (
        <div
          key={item.id}
          draggable
          onDragStart={() => handleDragStart(item.id)}
          onDragOver={(e) => handleDragOver(e, item.id)}
          style={{ padding: 10, margin: 5, border: '1px solid #ccc', cursor: 'move' }}
        >
          {item.content}
        </div>
      ))}
    </div>
  );
}
```

```tsx
// app/page.tsx
import DraggableList from './components/DraggableList';

async function TaskItem({ id }: { id: string }) {
  const task = await fetch(`https://api.example.com/tasks/${id}`);
  const data = await task.json();
  
  return (
    <div>
      <h3>{data.title}</h3>
      <p>{data.description}</p>
    </div>
  );
}

export default async function Page() {
  const tasks = await fetch('https://api.example.com/tasks');
  const data = await tasks.json();
  
  return (
    <DraggableList
      items={data.map((task: any) => ({
        id: task.id,
        content: <TaskItem id={task.id} />
      }))}
    />
  );
}
```

---

### Trick 11: Search with Server-Rendered Results

Client-side search UI with server-rendered results.

```tsx
// app/components/SearchableList.tsx
'use client';

import { useState, useTransition } from 'react';
import { useRouter, useSearchParams } from 'next/navigation';

export default function SearchableList({ children }: { children: React.ReactNode }) {
  const router = useRouter();
  const searchParams = useSearchParams();
  const [isPending, startTransition] = useTransition();
  const [search, setSearch] = useState(searchParams.get('q') || '');

  const handleSearch = (value: string) => {
    setSearch(value);
    startTransition(() => {
      const params = new URLSearchParams(searchParams);
      if (value) {
        params.set('q', value);
      } else {
        params.delete('q');
      }
      router.push(`?${params.toString()}`);
    });
  };

  return (
    <div>
      <input
        type="search"
        value={search}
        onChange={(e) => handleSearch(e.target.value)}
        placeholder="Search..."
        className={isPending ? 'loading' : ''}
      />
      {isPending && <div>Searching...</div>}
      <div>{children}</div>
    </div>
  );
}
```

```tsx
// app/search/page.tsx
import SearchableList from '../components/SearchableList';

async function SearchResults({ query }: { query: string }) {
  const results = await fetch(`https://api.example.com/search?q=${query}`);
  const data = await results.json();
  
  return (
    <ul>
      {data.map((item: any) => (
        <li key={item.id}>{item.title}</li>
      ))}
    </ul>
  );
}

export default function SearchPage({
  searchParams
}: {
  searchParams: { q?: string }
}) {
  return (
    <SearchableList>
      {searchParams.q ? (
        <SearchResults query={searchParams.q} />
      ) : (
        <div>Enter a search query</div>
      )}
    </SearchableList>
  );
}
```

---

### Trick 12: Infinite Scroll with Server Components

Implement infinite scrolling with server-rendered items.

```tsx
// app/components/InfiniteScroll.tsx
'use client';

import { useEffect, useRef } from 'react';

export default function InfiniteScroll({
  children,
  onLoadMore,
  hasMore
}: {
  children: React.ReactNode;
  onLoadMore: () => void;
  hasMore: boolean;
}) {
  const sentinelRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting && hasMore) {
          onLoadMore();
        }
      },
      { threshold: 1.0 }
    );

    if (sentinelRef.current) {
      observer.observe(sentinelRef.current);
    }

    return () => observer.disconnect();
  }, [hasMore, onLoadMore]);

  return (
    <div>
      {children}
      {hasMore && <div ref={sentinelRef}>Loading more...</div>}
    </div>
  );
}
```

```tsx
// app/posts/page.tsx
import InfiniteScroll from '../components/InfiniteScroll';

async function PostList({ page }: { page: number }) {
  const posts = await fetch(`https://api.example.com/posts?page=${page}`);
  const data = await posts.json();
  
  return (
    <>
      {data.map((post: any) => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </>
  );
}

export default function PostsPage() {
  // Note: This is a simplified example
  // In production, use URL search params or state management
  return (
    <InfiniteScroll
      onLoadMore={() => {/* Load next page */}}
      hasMore={true}
    >
      <PostList page={1} />
    </InfiniteScroll>
  );
}
```

---

### Trick 13: Optimistic UI Updates

Show immediate feedback while Server Actions complete.

```tsx
// app/components/OptimisticList.tsx
'use client';

import { useOptimistic } from 'react';

type Item = { id: string; text: string; pending?: boolean };

export default function OptimisticList({
  items,
  addItem
}: {
  items: Item[];
  addItem: (text: string) => Promise<void>;
}) {
  const [optimisticItems, addOptimisticItem] = useOptimistic(
    items,
    (state, newItem: string) => [
      ...state,
      { id: crypto.randomUUID(), text: newItem, pending: true }
    ]
  );

  const handleSubmit = async (formData: FormData) => {
    const text = formData.get('text') as string;
    addOptimisticItem(text);
    await addItem(text);
  };

  return (
    <div>
      <form action={handleSubmit}>
        <input name="text" placeholder="Add item..." required />
        <button type="submit">Add</button>
      </form>
      <ul>
        {optimisticItems.map((item) => (
          <li key={item.id} style={{ opacity: item.pending ? 0.5 : 1 }}>
            {item.text}
            {item.pending && ' (Saving...)'}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

### Trick 14: Portal Pattern

Render Server Components in different DOM locations.

```tsx
// app/components/PortalWrapper.tsx
'use client';

import { useEffect, useState } from 'react';
import { createPortal } from 'react-dom';

export default function PortalWrapper({
  children,
  targetId
}: {
  children: React.ReactNode;
  targetId: string;
}) {
  const [mounted, setMounted] = useState(false);

  useEffect(() => {
    setMounted(true);
  }, []);

  if (!mounted) return null;

  const target = document.getElementById(targetId);
  if (!target) return null;

  return createPortal(children, target);
}
```

```tsx
// app/page.tsx
import PortalWrapper from './components/PortalWrapper';

async function ServerNotification() {
  const notifications = await fetch('https://api.example.com/notifications');
  const data = await notifications.json();
  
  return (
    <div>
      {data.map((notif: any) => (
        <div key={notif.id}>{notif.message}</div>
      ))}
    </div>
  );
}

export default function Page() {
  return (
    <>
      <div id="notification-portal"></div>
      <main>
        <h1>Main Content</h1>
        <PortalWrapper targetId="notification-portal">
          <ServerNotification />
        </PortalWrapper>
      </main>
    </>
  );
}
```

---

### Trick 15: Compound Components Pattern

Create flexible component APIs with Server Components.

```tsx
// app/components/Card.tsx
'use client';

import { createContext, useContext, useState } from 'react';

const CardContext = createContext({ isExpanded: false, toggle: () => {} });

export function Card({ children }: { children: React.ReactNode }) {
  const [isExpanded, setIsExpanded] = useState(false);
  
  return (
    <CardContext.Provider value={{ isExpanded, toggle: () => setIsExpanded(!isExpanded) }}>
      <div className="card">{children}</div>
    </CardContext.Provider>
  );
}

export function CardHeader({ children }: { children: React.ReactNode }) {
  const { toggle } = useContext(CardContext);
  return <div className="card-header" onClick={toggle}>{children}</div>;
}

export function CardBody({ children }: { children: React.ReactNode }) {
  const { isExpanded } = useContext(CardContext);
  return isExpanded ? <div className="card-body">{children}</div> : null;
}
```

```tsx
// app/page.tsx
import { Card, CardHeader, CardBody } from './components/Card';

async function UserDetails({ id }: { id: string }) {
  const user = await fetch(`https://api.example.com/users/${id}`);
  const data = await user.json();
  
  return (
    <div>
      <p>Email: {data.email}</p>
      <p>Phone: {data.phone}</p>
      <p>Address: {data.address}</p>
    </div>
  );
}

export default function Page() {
  return (
    <Card>
      <CardHeader>
        <h2>User Information</h2>
      </CardHeader>
      <CardBody>
        <UserDetails id="123" />
      </CardBody>
    </Card>
  );
}
```

---

## Best Practices



### 1. Keep Client Components Minimal

```tsx
// ✅ Good - Small client wrapper
'use client';

export default function InteractiveWrapper({ children }: { children: React.ReactNode }) {
  const [isVisible, setIsVisible] = useState(true);
  
  return isVisible ? <div>{children}</div> : null;
}
```

```tsx
// ❌ Bad - Entire page is client component
'use client';

export default function Page() {
  const [isVisible, setIsVisible] = useState(true);
  
  // Lots of server-side logic here...
  const data = await fetch(...); // Can't do this in client component!
  
  return <div>...</div>;
}
```

### 2. Use Composition Over Nesting

```tsx
// ✅ Good - Compose components
export default function Page() {
  return (
    <ClientWrapper>
      <ServerComponent1 />
      <ServerComponent2 />
    </ClientWrapper>
  );
}
```

```tsx
// ❌ Bad - Deep nesting
export default function Page() {
  return (
    <ClientWrapper1>
      <ClientWrapper2>
        <ClientWrapper3>
          <ServerComponent />
        </ClientWrapper3>
      </ClientWrapper2>
    </ClientWrapper1>
  );
}
```

### 3. Pass Multiple Server Components

```tsx
// ✅ Good - Multiple slots
'use client';

export default function Layout({ 
  header, 
  sidebar, 
  content 
}: { 
  header: React.ReactNode;
  sidebar: React.ReactNode;
  content: React.ReactNode;
}) {
  return (
    <div className="layout">
      <header>{header}</header>
      <aside>{sidebar}</aside>
      <main>{content}</main>
    </div>
  );
}
```

```tsx
// Usage
<Layout
  header={<ServerHeader />}
  sidebar={<ServerSidebar />}
  content={<ServerContent />}
/>
```

### 4. Separate Concerns

```tsx
// ✅ Good - Clear separation
// ClientInteractivity.tsx
'use client';
export function ClientInteractivity({ children }: { children: React.ReactNode }) {
  const [state, setState] = useState();
  // Only client logic here
  return <div onClick={...}>{children}</div>;
}

// ServerData.tsx (default Server Component)
export async function ServerData() {
  const data = await fetchData();
  // Only server logic here
  return <div>{data}</div>;
}

// page.tsx
export default function Page() {
  return (
    <ClientInteractivity>
      <ServerData />
    </ClientInteractivity>
  );
}
```

---

## Common Mistakes

### Mistake 1: Importing Server Component in Client Component

```tsx
// ❌ Wrong
'use client';

import ServerComponent from './ServerComponent'; // Error!

export default function ClientComponent() {
  return <ServerComponent />;
}
```

```tsx
// ✅ Correct
'use client';

export default function ClientComponent({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>;
}

// In parent (Server Component):
<ClientComponent>
  <ServerComponent />
</ClientComponent>
```

### Mistake 2: Using Async in Client Components

```tsx
// ❌ Wrong
'use client';

export default async function ClientComponent() { // Can't use async!
  const data = await fetch(...);
  return <div>{data}</div>;
}
```

```tsx
// ✅ Correct - Use Server Component for data fetching
async function ServerData() {
  const data = await fetch(...);
  return <div>{data}</div>;
}

'use client';
function ClientWrapper({ children }: { children: React.ReactNode }) {
  return <div className="wrapper">{children}</div>;
}

// In page:
<ClientWrapper>
  <ServerData />
</ClientWrapper>
```

### Mistake 3: Forgetting 'use client' Directive

```tsx
// ❌ Wrong - Missing 'use client'
import { useState } from 'react';

export default function Component() {
  const [count, setCount] = useState(0); // Error! Can't use hooks in Server Component
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

```tsx
// ✅ Correct
'use client';

import { useState } from 'react';

export default function Component() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

---

## Real-World Examples

### Example 1: Dashboard with Interactive Filters

```tsx
// app/components/FilterableList.tsx
'use client';

import { useState } from 'react';

export default function FilterableList({ children }: { children: React.ReactNode }) {
  const [filter, setFilter] = useState('');

  return (
    <div>
      <input
        type="text"
        placeholder="Filter..."
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
      />
      <div data-filter={filter}>{children}</div>
    </div>
  );
}
```

```tsx
// app/dashboard/page.tsx
import FilterableList from '../components/FilterableList';

async function UserList() {
  const users = await fetch('https://api.example.com/users', {
    cache: 'no-store'
  });
  const data = await users.json();

  return (
    <ul>
      {data.map(user => (
        <li key={user.id} data-name={user.name.toLowerCase()}>
          {user.name} - {user.email}
        </li>
      ))}
    </ul>
  );
}

export default function DashboardPage() {
  return (
    <div>
      <h1>User Dashboard</h1>
      <FilterableList>
        <UserList />
      </FilterableList>
    </div>
  );
}
```

### Example 2: Animated Container with Server Content

```tsx
// app/components/AnimatedContainer.tsx
'use client';

import { motion } from 'framer-motion';

export default function AnimatedContainer({ children }: { children: React.ReactNode }) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.5 }}
    >
      {children}
    </motion.div>
  );
}
```

```tsx
// app/page.tsx
import AnimatedContainer from './components/AnimatedContainer';

async function BlogPosts() {
  const posts = await fetch('https://api.example.com/posts');
  const data = await posts.json();

  return (
    <div>
      {data.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  );
}

export default function Page() {
  return (
    <AnimatedContainer>
      <BlogPosts />
    </AnimatedContainer>
  );
}
```

### Example 3: Accordion with Database Content

```tsx
// app/components/Accordion.tsx
'use client';

import { useState } from 'react';

export default function Accordion({ 
  items 
}: { 
  items: { title: string; content: React.ReactNode }[] 
}) {
  const [openIndex, setOpenIndex] = useState<number | null>(null);

  return (
    <div className="accordion">
      {items.map((item, index) => (
        <div key={index} className="accordion-item">
          <button
            className="accordion-title"
            onClick={() => setOpenIndex(openIndex === index ? null : index)}
          >
            {item.title}
            <span>{openIndex === index ? '−' : '+'}</span>
          </button>
          {openIndex === index && (
            <div className="accordion-content">{item.content}</div>
          )}
        </div>
      ))}
    </div>
  );
}
```

```tsx
// app/faq/page.tsx
import Accordion from '../components/Accordion';
import { db } from '@/lib/db';

async function FAQContent({ id }: { id: string }) {
  const faq = await db.faq.findUnique({ where: { id } });
  
  return (
    <div>
      <p>{faq.answer}</p>
      <small>Last updated: {faq.updatedAt.toLocaleDateString()}</small>
    </div>
  );
}

export default async function FAQPage() {
  const faqs = await db.faq.findMany();

  return (
    <div>
      <h1>Frequently Asked Questions</h1>
      <Accordion
        items={faqs.map(faq => ({
          title: faq.question,
          content: <FAQContent id={faq.id} />
        }))}
      />
    </div>
  );
}
```

---

## Summary

### Key Takeaways

✅ **Server Components** are the default in Next.js 16 App Router  
✅ **Client Components** need `'use client'` directive  
✅ **Cannot import** Server Components into Client Components  
✅ **Can pass** Server Components as `children` or props  
✅ **Use composition** to combine Server and Client Components  
✅ **Keep Client Components minimal** for better performance

### Quick Reference

| Pattern | Works? | Example |
|---------|--------|---------|
| Server → Client (import) | ❌ No | `import Server from './server'` in Client Component |
| Server → Client (children) | ✅ Yes | `<Client><Server /></Client>` |
| Server → Client (props) | ✅ Yes | `<Client content={<Server />} />` |
| Client → Server | ✅ Yes | `import Client from './client'` in Server Component |
| Client → Client | ✅ Yes | Normal React composition |
| Server → Server | ✅ Yes | Normal React composition |

### Decision Tree

```
Need interactivity (hooks, events)?
├─ Yes → Use Client Component
│   └─ Need server data inside?
│       ├─ Yes → Pass Server Component as children
│       └─ No → Regular Client Component
└─ No → Use Server Component (default)
    └─ Can fetch data directly!
```

---

## Related Documentation

- [Next.js 16 App Router Routing Guide](file:///c:/Users/mmt20/Desktop/beetleware-frontend-intern/nexjs/app-router-routing-guide.md) - Comprehensive routing patterns
- [Next.js 16 Folder Structure](file:///c:/Users/mmt20/Desktop/beetleware-frontend-intern/nexjs/next16-folder-structure.md) - Project organization
- [Rendering & Data Fetching](file:///c:/Users/mmt20/Desktop/beetleware-frontend-intern/nexjs/rendering-and-data-fetching.md) - SSR, SSG, ISR strategies
- [Next.js Basics](file:///c:/Users/mmt20/Desktop/beetleware-frontend-intern/nexjs/nextjs-basics.md) - Getting started guide

---

## Resources

- **Official Docs:** [nextjs.org/docs/app/building-your-application/rendering/composition-patterns](https://nextjs.org/docs/app/building-your-application/rendering/composition-patterns)
- **Server Components:** [nextjs.org/docs/app/building-your-application/rendering/server-components](https://nextjs.org/docs/app/building-your-application/rendering/server-components)
- **Client Components:** [nextjs.org/docs/app/building-your-application/rendering/client-components](https://nextjs.org/docs/app/building-your-application/rendering/client-components)
- **Original Article:** [Wrapping Server Components in Client Components](https://rishibakshi.hashnode.dev/wrapping-server-components-in-client-components-what-happens)

---

**Next.js 16** - Master the Server and Client Component composition pattern for optimal performance and developer experience.
