# Chapter 14 — Data Fetching on the Client and Performance

> **Learning Objective:** Master efficient data fetching strategies in React, understand browser limitations, identify and eliminate request waterfalls, and optimize initial page load performance.

---

## 🎯 Part 1: The Problem — Data Fetching Complexity

### The Landscape

The modern data fetching ecosystem is overwhelming:

- Endless data management libraries
- GraphQL vs REST debates
- `useEffect` criticized for causing waterfalls
- Suspense still not officially ready for data fetching (at time of publication)
- Confusing patterns: fetch-on-render, fetch-then-render, render-as-you-fetch

**The Core Question:** What is the actual "right way" to fetch data in React?

---

## 📚 Part 2: Types of Data Fetching

### 2.1 Two Main Categories

**1. Data on Demand**

**Definition:** Data fetched after user interaction to update their experience.

**Examples:**

- Autocompletes
- Dynamic forms
- Search experiences

**Implementation:** Usually triggered in callbacks.

**2. Initial Data**

**Definition:** Data expected to be visible immediately when a page opens.

**Characteristics:**

- Needs to be fetched before component renders
- Critical for user's first impression
- Usually happens in `useEffect` (or `componentDidMount` for classes)
- Most crucial for performance perception ("slow as hell" vs "blazing fast")

**Key Insight:** Although these categories seem different, the core principles and fundamental patterns are exactly the same for both.

---

## 🔧 Part 3: Do You Need an External Library?

### 3.1 Short Answer: It Depends

**Simple Use Case — NO:**

If you just need to fetch data once and forget about it, a simple `fetch` in `useEffect` works fine:

```js
const Component = () => {
  const [data, setData] = useState();

  useEffect(() => {
    const dataFetch = async () => {
      const data = await (await fetch("https://api.example.com/data")).json();

      setData(data);
    };

    dataFetch();
  }, []);

  return <>...</>;
};
```

### 3.2 Complex Use Cases — YES (or Build It Yourself)

**Challenges Beyond Simple Fetching:**

- Error handling
- Multiple components fetching from the same endpoint
- Caching (how long?)
- Race conditions
- Request cancellation
- Memory leaks

**Two Paths:**

1. **Reinvent the wheel** — Write code to solve all these problems
2. **Use existing libraries** — Leverage years of battle-tested solutions

### 3.3 Library Examples

**Axios:**

- Abstracts concerns like request cancellation
- No opinion on React-specific API
- Handles low-level HTTP complexities

**SWR:**

- Handles everything including caching
- React-specific hooks API
- Built-in error handling and revalidation

**Critical Principle:**

> No library or Suspense can improve your app's performance just by itself. You must understand the fundamentals of data fetching and orchestration patterns.

---

## 📊 Part 4: What Is a "Performant" App?

### 4.1 The Issue Tracker Example

**Scenario:** Building an issue tracker with:

- Sidebar navigation (left)
- Main issue information (center)
- Comments section (bottom)

**Three Different Implementations:**

**App 1: All at Once**

- Shows loading until all data loaded
- Renders everything together
- Takes ~3 seconds total
- Fastest overall time
- Longest time showing nothing

**App 2: Sidebar First**

- Shows loading, then sidebar first
- Keeps loading for main content
- Sidebar: ~1 second
- Everything: ~4 seconds total
- Shows something quickly but main content delayed

**App 3: Main Content First**

- Shows main issue first
- Then sidebar
- Finally comments
- Main: ~2 seconds
- Sidebar: +1 second later
- Comments: +2 seconds after sidebar
- Total: ~5 seconds
- Most "junky" experience (violates left-to-right flow)

### 4.2 The Verdict

**Which is most performant?**

**Answer:** None of them. Or all of them. Or any of them. **It depends.**

**Key Considerations:**

1. **Pure Speed:** App 1 is fastest (3s total)
2. **Show Something Fast:** App 2 wins (1s to first content)
3. **Show Main Content First:** App 3 prioritizes main issue
4. **Natural Flow:** App 3 violates reading patterns (top-left to bottom-right)

### 4.3 The Storytelling Principle

> Think of yourself as a storyteller, and the app is your story.

**Questions to Ask:**

- What is the most important piece of the story?
- What is second?
- Does your story have a flow?
- Can you tell it in pieces?
- Should users see the full story immediately?

**When to Optimize:**

Only after you know what your story should look like.

**True Power Comes From Understanding:**

1. When is it okay to start fetching data?
2. What can we do while data fetching is in progress?
3. What should we do when data fetching is done?

---

## 🔄 Part 5: React Lifecycle and Data Fetching

### 5.1 Conditional Rendering Review

**Example 1: Conditional Component**

```js
const Child = () => {
  useEffect(() => {
    // fetch data for Child
  }, []);

  return <div>Some child</div>;
};

const Parent = () => {
  const [isLoading, setIsLoading] = useState(true);

  if (isLoading) return "loading";

  return <Child />;
};
```

**Question:** Will Child's `useEffect` trigger?

**Answer:** NO — not until `isLoading` becomes `false`.

**Example 2: Element Before Return**

```js
const Parent = () => {
  const [isLoading, setIsLoading] = useState(true);

  // Child element created here!
  const child = <Child />;

  if (isLoading) return "loading";

  return child;
};
```

**Question:** Will Child's `useEffect` trigger now?

**Answer:** Still NO!

**Critical Understanding:**

```js
const child = <Child />;
```

This doesn't "render" the component. `<Child />` is syntax sugar for a function that creates a **description** of a future element.

**It is only rendered when:**

- This description ends up in the actual visible render tree
- i.e., returned from the component

Until then, it just sits there as an object and does nothing.

---

## 🌐 Part 6: Browser Limitations

### 6.1 The Parallel Request Limit

**Critical Browser Constraint:**

Browsers have a **limit on parallel requests to the same host**.

**HTTP/1 (70% of internet):**

- Chrome: **6 requests** in parallel
- If you fire more, the rest **queue and wait**

### 6.2 Why This Matters

**Simple Issue Tracker:**

- Already has 3 requests
- Just 3 more requests away from the limit
- A slow analytics request can block everything

**Example of the Problem:**

```js
const App = () => {
  const { data } = useData("/fetch-some-data");

  if (!data) return "loading...";

  return <div>I'm an app</div>;
};
```

**Scenario:**

- The fetch request is super fast (~50ms)
- But if 6 requests taking 10 seconds are fired before it
- The entire app load will take **10 seconds**

**Code Example:**

```js
// No waiting, no resolving, just fire and forget
fetch("https://some-url.com/url1");
fetch("https://some-url.com/url2");
fetch("https://some-url.com/url3");
fetch("https://some-url.com/url4");
fetch("https://some-url.com/url5");
fetch("https://some-url.com/url6");

const App = () => {
  // This will be blocked!
  const { data } = useData("/fetch-some-data");
  // ...
};
```

**Result:** Even though the app's request is fast, it waits for an available "slot."

---

## 🌊 Part 7: Request Waterfalls — How They Appear

### 7.1 Initial Implementation

**Component Structure:**

```js
const App = () => {
  return (
    <>
      <Sidebar />
      <Issue />
    </>
  );
};

const Sidebar = () => {
  return; // some sidebar links
};

const Issue = () => {
  return (
    <>
      {/* some issue data */}
      <Comments />
    </>
  );
};

const Comments = () => {
  return; // some issue comments
};
```

### 7.2 Custom Hook for Data Fetching

**Extracted Hook:**

```js
export const useData = (url) => {
  const [state, setState] = useState();

  useEffect(() => {
    const dataFetch = async () => {
      const data = await (await fetch(url)).json();
      setState(data);
    };

    dataFetch();
  }, [url]);

  return { data: state };
};
```

### 7.3 Natural (But Wrong) Implementation

**Comments Component:**

```js
const Comments = () => {
  const { data } = useData("/get-comments");

  if (!data) return "loading";

  return data.map((comment) => <div>{comment.title}</div>);
};
```

**Issue Component:**

```js
const Issue = () => {
  const { data } = useData("/get-issue");

  if (!data) return "loading";

  return (
    <div>
      <h3>{data.title}</h3>
      <p>{data.description}</p>
      <Comments />
    </div>
  );
};
```

**App Component:**

```js
const App = () => {
  const { data } = useData("/get-sidebar");

  if (!data) return "loading";

  return (
    <>
      <Sidebar data={data} />
      <Issue />
    </>
  );
};
```

### 7.4 The Problem: Classic Waterfall

**What Happens:**

1. App fetches sidebar data → shows "loading"
2. When sidebar data loads → App re-renders
3. Issue component mounts → fetches issue data → shows "loading"
4. When issue data loads → Issue re-renders
5. Comments component mounts → fetches comments → shows "loading"
6. When comments load → Comments renders

**Sequence Diagram:**

```
App fetch (1s) → Issue fetch (2s) → Comments fetch (3s)
Total: 1s + 2s + 3s = 6 seconds
```

**Why It's Slow:**

Every component returns "loading" while waiting for data. Only when data loads does it switch to the next component in the render tree, triggering its own data fetching.

**The cycle repeats** → Classic waterfall!

---

## ✅ Part 8: Solving Request Waterfalls

### 8.1 Solution 1: Promise.all

**Principle:** Pull all data-fetching requests as high in the render tree as possible.

**Wrong Approach (Still a Waterfall):**

```js
useEffect(async () => {
  const sidebar = await fetch("/get-sidebar");
  const issue = await fetch("/get-issue");
  const comments = await fetch("/get-comments");
}, []);
```

**Problem:** Sequential awaits

- Time = 1s + 2s + 3s = **6 seconds**

**Correct Approach: Promise.all**

```js
useEffect(async () => {
  const [sidebar, issue, comments] = await Promise.all([
    fetch("/get-sidebar"),
    fetch("/get-issue"),
    fetch("/get-comments"),
  ]);
}, []);
```

**Benefit:** Parallel requests

- Time = max(1s, 2s, 3s) = **3 seconds**
- **50% performance improvement!**

**Full Implementation:**

```js
const useAllData = () => {
  const [sidebar, setSidebar] = useState();
  const [comments, setComments] = useState();
  const [issue, setIssue] = useState();

  useEffect(() => {
    const dataFetch = async () => {
      // Fire all requests in parallel
      const result = (await Promise.all([fetch(sidebarUrl), fetch(issueUrl), fetch(commentsUrl)])).map((r) => r.json());

      // Wait for all JSON parsing
      const [sidebarResult, issueResult, commentsResult] = await Promise.all(result);

      // Save to state
      setSidebar(sidebarResult);
      setIssue(issueResult);
      setComments(commentsResult);
    };

    dataFetch();
  }, []);

  return { sidebar, comments, issue };
};

const App = () => {
  const { sidebar, comments, issue } = useAllData();

  // Wait for ALL data
  if (!sidebar || !comments || !issue) return "loading";

  return (
    <>
      <Sidebar data={sidebar} />
      <Issue comments={comments} issue={issue} />
    </>
  );
};
```

**This is how App 1 from the test is implemented.**

---

### 8.2 Solution 2: Parallel Promises (Independent Resolution)

**Problem with Promise.all:**

- Waits for ALL requests to complete
- Slowest request blocks everything
- Comments are slowest and least important

**Better Approach: Independent Resolution**

```js
fetch("/get-sidebar")
  .then((data) => data.json())
  .then((data) => setSidebar(data));

fetch("/get-issue")
  .then((data) => data.json())
  .then((data) => setIssue(data));

fetch("/get-comments")
  .then((data) => data.json())
  .then((data) => setComments(data));
```

**Benefits:**

- All requests fired **in parallel**
- Each resolved **independently**
- Fast data renders immediately
- Slow data doesn't block fast data

**App Implementation:**

```js
const App = () => {
  const { sidebar, issue, comments } = useAllData();

  // Wait only for sidebar
  if (!sidebar) return "loading";

  return (
    <>
      <Sidebar data={sidebar} />

      {/* Render issue as soon as available */}
      {issue ? <Issue comments={comments} issue={issue} /> : "loading"}
    </>
  );
};
```

**Result:**

- Sidebar appears in ~1s
- Issue appears in ~2s
- Comments appear in ~3s
- Same behavior as waterfall but **50% faster** (3s vs 6s)

**Trade-off:**

Three independent state updates = three re-renders of the parent component.

**Impact:** Depends on:

- Component order
- Component size
- App architecture

---

### 8.3 Solution 3: Data Providers (Best Architecture)

**Problem with Previous Solutions:**

- Lifting data up is good for performance
- But terrible for architecture and readability
- Massive props drilling
- Co-location lost

**Solution: Data Providers**

**Concept:** Abstraction that lets you:

- Fetch data in one place
- Access data in another
- Bypass all components in between
- Essentially a mini-caching layer per request

**Implementation with Context:**

```js
const Context = React.createContext();

export const CommentsDataProvider = ({ children }) => {
  const [comments, setComments] = useState();

  useEffect(async () => {
    fetch("/get-comments")
      .then((data) => data.json())
      .then((data) => setComments(data));
  }, []);

  return <Context.Provider value={comments}>{children}</Context.Provider>;
};

export const useComments = () => useContext(Context);
```

**Same for all three requests** (Sidebar, Issue, Comments).

**App Component (Now Simple):**

```js
const App = () => {
  const sidebar = useSidebar();
  const issue = useIssue();

  if (!sidebar) return "loading";

  return (
    <>
      <Sidebar />
      {issue ? <Issue /> : "loading"}
    </>
  );
};
```

**No props drilling!**

**Root Setup:**

```js
export const VeryRootApp = () => {
  return (
    <SidebarDataProvider>
      <IssueDataProvider>
        <CommentsDataProvider>
          <App />
        </CommentsDataProvider>
      </IssueDataProvider>
    </SidebarDataProvider>
  );
};
```

**Providers fire requests in parallel as soon as mounted.**

**Deep Component Usage:**

```js
const Comments = () => {
  // No props drilling!
  const comments = useComments();

  // render comments
};
```

**Works with Any State Management:**

Not a fan of Context? Use Redux, Zustand, Jotai, etc. — same concept applies.

---

### 8.4 Solution 4: Pre-fetching (Dangerous but Powerful)

**The Trick: Fetch Before React**

**Original Component:**

```js
const Comments = () => {
  const [data, setData] = useState();

  useEffect(() => {
    const dataFetch = async () => {
      const data = await (await fetch("/get-comments")).json();

      setData(data);
    };

    dataFetch();
  }, [url]);

  if (!data) return "loading";

  return data.map((comment) => <div>{comment.title}</div>);
};
```

**Observation:**

Line 6: `fetch('/get-comments')` is just a promise that:

- Doesn't depend on React
- No props, state, or variable dependencies
- Can exist outside the component

**The Hack:**

```js
// Move fetch OUTSIDE the component
const commentsPromise = fetch("/get-comments");

const Comments = () => {
  useEffect(() => {
    const dataFetch = async () => {
      // Just await the existing promise
      const data = await (await commentsPromise).json();
      setState(data);
    };

    dataFetch();
  }, [url]);
};
```

**What Happens:**

1. Fetch call **escapes** React lifecycle
2. Fires as soon as JavaScript loads
3. Before any `useEffect` anywhere
4. Before root App's first request
5. Data sits there until someone resolves it
6. `useEffect` in Comments just awaits the existing promise

**Performance Impact:**

**Before:**

```
App → Issue → Comments (waterfall)
```

**After:**

```
Comments fetches immediately (parallel with everything)
```

---

### 8.5 Why Pre-fetching Is Dangerous

**Remember Browser Limitations:**

- Only 6 parallel requests (HTTP/1)
- Pre-fetches fire **immediately**
- Completely **uncontrollable**

**The Problem:**

A component that:

- Fetches heavy data
- Renders rarely
- In traditional waterfall: harmless until rendered
- With pre-fetch: **steals critical milliseconds** from initial load

**Debugging Nightmare:**

Component in a corner of codebase, never rendered, slows down the entire app. Good luck figuring that out!

---

### 8.6 Legitimate Use Cases for Pre-fetching

**Only Two Acceptable Scenarios:**

**1. Critical Resources at Router Level**

- You **know** data is critical
- Required immediately
- Part of the core flow

**2. Lazy-Loaded Components**

- JavaScript downloaded only when component enters render tree
- By definition, after critical data fetched
- Safe because it won't block initial load

---

## 📚 Part 9: Data Fetching Libraries

### 9.1 Library-Independent Patterns

**Key Principle:**

All patterns shown are **fundamental** and **library-independent**.

Regardless of your library choice:

- Waterfalls still exist
- React lifecycle still matters
- Browser limitations still apply
- Fetching timing still critical

### 9.2 React-Independent Libraries

**Example: Axios**

- Abstracts `fetch` complexities
- Handles request cancellation, timeouts, etc.
- No React-specific API

**Usage:**

Replace all `fetch` with `axios.get` — patterns remain the same.

### 9.3 React-Integrated Libraries

**Example: SWR**

Abstracts away:

- `useCallback`, `useState`, `useEffect`
- Error handling
- Caching
- Revalidation

**Before (Raw Fetch):**

```js
const Comments = () => {
  const [data, setData] = useState();

  useEffect(() => {
    const dataFetch = async () => {
      const data = await (await fetch("/get-comments")).json();

      setState(data);
    };

    dataFetch();
  }, [url]);

  // rest of comments code
};
```

**After (SWR):**

```js
const Comments = () => {
  const { data } = useSWR("/get-comments", fetcher);

  // rest of comments code
};
```

**Underneath:**

All libraries still use `useEffect` or equivalent to fetch data and state to trigger re-renders.

---

## 🎭 Part 10: What About Suspense?

### 10.1 Current State (At Time of Publication)

**Reality Check:**

- Suspense for data fetching is **undocumented**
- Not officially supported or recommended by React
- Exception: Opinionated frameworks (e.g., Next.js)

**If Using Framework:**

Read their specific documentation on Suspense for data fetching.

### 10.2 Future Potential

**If Suspense Becomes Available:**

Will it fundamentally solve data fetching? **No.**

**What Suspense Actually Does:**

Replaces manual loading state management.

**Before:**

```js
const Comments = ({ comments }) => {
  if (!comments) return "loading";

  // render comments
};
```

**After (Suspense):**

```js
const Issue = () => {
  return (
    <>
      {/* issue data */}
      <Suspense fallback="loading">
        <Comments />
      </Suspense>
    </>
  );
};
```

**What Stays the Same:**

- Browser limitations
- React lifecycle
- Request waterfalls
- Orchestration patterns

**Suspense is a loading state abstraction, not a data fetching solution.**

---

## 📝 Key Takeaways

**1. Two Categories of Data Fetching**

- **Initial data:** Fetched before page renders
- **On demand:** Fetched after user interaction
- Same principles apply to both

**2. Library Decision**

- Simple use case: `fetch` in `useEffect` is fine
- Complex needs: Use battle-tested libraries (Axios, SWR, React Query)
- No library solves fundamental patterns for you

**3. Performance Is Subjective**

- Depends on the story you're telling
- Consider what's most important to show first
- Natural flow matters (top-left to bottom-right for LTR languages)

**4. React Lifecycle Matters**

- Components must be in the render tree to trigger `useEffect`
- `<Child />` is just a description until returned
- Conditional rendering affects when fetches fire

**5. Browser Limitations Are Real**

- HTTP/1: Only 6 parallel requests to same host
- Excess requests queue
- Pre-fetches can steal critical slots

**6. Request Waterfalls**

- Appear when fetches are sequential or conditional
- Caused by components fetching data only after mounting
- Total time = sum of individual times

**7. Waterfall Solutions**

- **Promise.all:** Parallel fetches, wait for all
- **Parallel promises:** Parallel fetches, independent resolution
- **Data providers:** Best architecture, maintains co-location
- **Pre-fetching:** Powerful but dangerous, use sparingly

**8. Library Integration**

- React-independent libraries (Axios): Abstract HTTP
- React-integrated libraries (SWR): Abstract state management
- All still use `useEffect` and state underneath

**9. Suspense Reality**

- Not ready for general use (at publication time)
- Abstracts loading states, not data orchestration
- Doesn't eliminate waterfalls or browser limits

**10. Optimization Principles**

- Fire requests as early as safe
- Fetch in parallel when possible
- Render as soon as data available
- Consider request priority
- Respect browser limits

---

## ⚠️ Common Pitfalls

1. **Sequential awaits in useEffect** — Creates unnecessary waterfalls
2. **Lifting state without data providers** — Props drilling nightmare
3. **Ignoring browser limits** — Queuing destroys parallelism
4. **Pre-fetching everything** — Steals critical request slots
5. **Assuming libraries solve patterns** — They don't; you must understand fundamentals
6. **Waiting for Suspense** — Use proven patterns now
7. **Forgetting React lifecycle** — Conditional components don't mount/fetch
8. **Not considering user story** — Technical speed ≠ perceived speed

---

## 🎓 Advanced Insights

**Performance Measurement:**

"Performant" means different things:

- **Time to first byte:** Network speed
- **Time to first content:** User sees something
- **Time to interactive:** User can act
- **Time to complete:** All data loaded
- **Perceived performance:** How fast it feels

**Prioritization Strategy:**

1. Identify critical path
2. Fetch critical data first (or in parallel)
3. Show critical UI immediately
4. Lazy load secondary content
5. Prefetch next likely interaction

**State Update Trade-offs:**

Independent promises cause multiple re-renders:

- **Pro:** Progressive rendering
- **Con:** Multiple parent re-renders
- **Solution:** Data providers with Context or state management

**HTTP/2 Consideration:**

HTTP/2 removes the 6-request limit:

- Multiplexing allows many parallel requests
- Still consider server load
- CDNs often use HTTP/2

**Caching Strategies:**

Implement at multiple levels:

- Browser cache (HTTP headers)
- Service Worker cache
- In-memory cache (libraries like SWR)
- Data provider cache (manual Context)

---

## 🔜 Next Chapter Preview

In Chapter 15, we'll dive into **race conditions** in data fetching:

- What are race conditions and why they occur
- How to detect them in your app
- Strategies to prevent them
- Request cancellation with AbortController
- Managing cleanup in useEffect
