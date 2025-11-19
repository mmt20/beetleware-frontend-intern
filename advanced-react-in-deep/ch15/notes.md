# Chapter 15 — Data Fetching and Race Conditions

> **Learning Objective:** Master race conditions in React data fetching, understand Promise behavior in async operations, and implement multiple strategies to prevent data inconsistencies.

---

## 🎯 Part 1: The Problem — Race Conditions

### What Are Race Conditions?

Race conditions are relatively rare in normal development and it's possible to build complicated apps without encountering them. However, when they appear, investigating and fixing them can be a real challenge.

**Core Concept:**

Since `fetch` or any async operation in JavaScript is essentially a Promise, understanding Promises is fundamental to understanding race conditions.

### Chapter Focus

- What Promises are and how innocent code can create race conditions
- Reasons for race conditions to appear
- How to fix them in at least four different ways

---

## 📚 Part 2: Understanding Promises

### 2.1 What Is a Promise?

**Definition:** A Promise is literally a promise. It's a way to execute something asynchronously.

**Synchronous vs Asynchronous:**

- **Synchronous:** JavaScript executes code step by step
- **Promise:** Allows triggering a task and moving on immediately without waiting
- The task "promises" to notify us when completed (and it does!)

### 2.2 Promise Behavior

**Most Common Use Case:** Data fetching

Whether using:

- Native `fetch`
- Abstraction like Axios
- Any other library

The Promise behavior remains the same.

**Code Example:**

```js
console.log("first step"); // will log FIRST

fetch("/some-url") // create promise here
  .then(() => {
    // wait for Promise to be done
    // log stuff after the promise is done
    console.log("second step"); // will log THIRD (if successful)
  })
  .catch(() => {
    console.log("something bad happened"); // will log THIRD (if error)
  });

console.log("third step"); // will log SECOND
```

**Flow:**

1. Create a promise: `fetch('/some-url')`
2. Do something when result is available: `.then`
3. Handle errors: `.catch`
4. Life continues while waiting

### 2.3 Execution Order

**Output:**

```
first step
third step
second step
```

**Why?**

JavaScript doesn't wait for the promise to complete. It moves on immediately and comes back when the promise resolves.

---

## 🐛 Part 3: Race Conditions with Promises

### 3.1 The Scenario

**Simple App Implementation:**

- Tabs column on the left
- Navigating between tabs sends a fetch request
- Data from request renders on the right

**Problem:**

When navigating between tabs quickly:

- Content blinks
- Data appears seemingly at random
- Sometimes content of first tab appears, then quickly replaced by second tab
- Sometimes creates a carousel effect
- Whole thing behaves weird

### 3.2 The Code

**App Component:**

```js
const App = () => {
  const [page, setPage] = useState("1");

  return (
    <>
      {/* left column buttons */}
      <button onClick={() => setPage("1")}>Issue 1</button>
      <button onClick={() => setPage("2")}>Issue 2</button>

      {/* the actual content */}
      <Page id={page} />
    </>
  );
};
```

**Page Component:**

```js
const Page = ({ id }: { id: string }) => {
  const [data, setData] = useState({});

  // pass id to fetch relevant data
  const url = `/some-url/${id}`;

  useEffect(() => {
    fetch(url)
      .then((r) => r.json())
      .then((r) => {
        // save data from fetch request to state
        setData(r);
      });
  }, [url]);

  // render data
  return (
    <>
      <h2>{data.title}</h2>
      <p>{data.description}</p>
    </>
  );
};
```

**Looks innocent, right?** But the app is broken.

---

## 🔍 Part 4: Race Condition Causes

### 4.1 Two Root Causes

**1. Nature of Promises**

Asynchronous operations that don't wait

**2. React Lifecycle**

How components mount, update, and manage state

### 4.2 Initial Flow (Normal Case)

**Step by Step:**

1. `App` component mounts
2. `Page` component mounts with default prop value "1"
3. `useEffect` in `Page` kicks in for the first time
4. `fetch` within `useEffect` is triggered (a Promise)
5. React moves on without waiting (~2 seconds pass)
6. Request completes → `.then` callback fires
7. `setData` is called → state updates
8. `Page` component re-renders with new data
9. We see it on the screen

**This works fine.**

### 4.3 Navigation Flow (Still Fine)

**When clicking navigation button:**

1. `App` changes state to another page
2. State change triggers re-render of `App`
3. `Page` component re-renders as well
4. `useEffect` has dependency on `id`
5. `id` has changed → `useEffect` triggers again
6. New `fetch` fires with new `id`
7. ~2 seconds later, `setData` called again
8. `Page` updates with new data
9. We see new data on screen

**Still works fine.**

### 4.4 The Race Condition (The Problem!)

**What if `id` changes WHILE first fetch is still in progress?**

**The Sequence:**

1. `App` triggers re-render of `Page`
2. `useEffect` triggers again (`id` changed)
3. Second `fetch` fires
4. React continues as usual
5. **FIRST fetch finishes**
   - Still has reference to `setData` of the same `Page` component
   - `setData` is triggered
   - `Page` updates with data from **first fetch** (old data!)
6. **SECOND fetch finishes**
   - Was hanging in the background
   - Also has reference to same `setData`
   - Gets triggered
   - `Page` updates again with data from **second fetch** (new data!)

**Result:**

**Boom, race condition!** Flash of content: old content renders, then replaced by new content.

### 4.5 Even Worse Scenario

**If second fetch finishes BEFORE first fetch:**

1. Correct content of new page appears first
2. Then replaced by incorrect content of previous page
3. User sees wrong data!

**Evil Problem:**

Code looks innocent, but app is broken.

---

## ✅ Part 5: Solution 1 — Force Re-mounting

### 5.1 Why This Sometimes Works

Race conditions don't happen as often during regular page navigation. Why?

**Different Implementation:**

```js
const App = () => {
  const [page, setPage] = useState("issue");

  return (
    <>
      {page === "issue" && <Issue />}
      {page === "about" && <About />}
    </>
  );
};
```

**Key Differences:**

- No props passed down
- `Issue` and `About` are separate components
- Each has its own unique URL
- Data fetching happens in `useEffect` (same as before)

**About Component Example:**

```js
const About = () => {
  const [about, setAbout] = useState();

  useEffect(() => {
    fetch("/some-url-for-about-page")
      .then((r) => r.json())
      .then((r) => setAbout(r));
  }, []);

  // render...
};
```

**Result:** No race condition! Navigate as many times and as fast as you want — behaves normally.

### 5.2 Why No Race Condition?

**Critical Difference:**

Components are **re-mounted**, not **re-rendered**.

**What Happens:**

1. `App` renders, mounts `Issue` component
2. Data fetching kicks in
3. Navigate to next page while fetch still in progress
4. `App` **unmounts** `Issue` component
5. `App` **mounts** `About` component instead
6. `About` kicks off its own data fetching

**When Component Unmounts:**

- Gone completely
- Disappears from screen
- No one has access to it
- Everything including state is lost

**Compare with:** `<Page id={page} />`

This `Page` component was never unmounted — just reused with its state.

### 5.3 What Happens to Pending Fetch?

**When `Issue`'s fetch finishes while on `About` page:**

1. `.then` callback tries to call `setIssue` state
2. But component is gone!
3. From React's perspective, it doesn't exist
4. Promise dies out
5. Data disappears into the void

### 5.4 The Famous Warning (Removed)

**"Can't perform a React state update on an unmounted component"**

- Used to appear in exactly these situations
- Async operation finishes after component is gone
- Recently removed from React

### 5.5 Applying to Original Problem

**Force Re-mount with Key:**

```js
<Page id={page} key={page} />
```

**How It Works:**

- Changing "key" forces React to remove the old element
- Mounts new one with new key
- Even if they're the same component type

### 5.6 ⚠️ Not Recommended as General Solution

**Too many caveats:**

- Performance might suffer
- Unexpected bugs with focus
- Unexpected bugs with state
- Unexpected triggering of `useEffect` down the render tree
- More like sweeping problem under the rug

**Better ways exist** (see below).

**Can be a tool in your arsenal in certain cases if used carefully.**

---

## ✅ Part 6: Solution 2 — Drop Incorrect Result

### 6.1 The Concept

Much gentler than nuking the entire component from existence.

**Strategy:** Make sure the result coming in the `.then` callback matches the `id` currently "active."

### 6.2 Implementation with Refs

**The Trick:**

Escape the React lifecycle and locally scoped data. Get access to the "latest" `id` inside all iterations of `useEffect`, even "stale" ones.

**Use Case for Refs:** Discussed in Chapter 9 — Refs: from storing data to imperative API.

**Code:**

```js
const Page = ({ id }) => {
  // create ref
  const ref = useRef(id);

  useEffect(() => {
    // update ref value with the latest id
    ref.current = id;

    fetch(`/some-data-url/${id}`)
      .then((r) => r.json())
      .then((r) => {
        // compare the latest id with the result
        // only update state if the result actually belongs to that id
        if (ref.current === r.id) {
          setData(r);
        }
      });
  }, [id]);
};
```

**How It Works:**

1. Create ref with initial `id`
2. On each `useEffect` run, update ref to latest `id`
3. When fetch completes, check if `ref.current` matches result's `id`
4. Only update state if they match
5. Otherwise, ignore the result

### 6.3 Alternative: Compare URL

**If results don't return reliable identifier:**

```js
const Page = ({ id }) => {
  // create ref
  const ref = useRef(id);

  useEffect(() => {
    // update ref value with the latest url
    ref.current = url;

    fetch(`/some-data-url/${id}`).then((result) => {
      // compare the latest url with the result's url
      // only update state if the result actually belongs to that url
      if (result.url === ref.current) {
        result.json().then((r) => {
          setData(r);
        });
      }
    });
  }, [url]);
};
```

**Flexibility:** Compare whatever uniquely identifies the request.

---

## ✅ Part 7: Solution 3 — Drop All Previous Results

### 7.1 The Concept

**Alternative Approach:** Use `useEffect` cleanup function.

**Cleanup Function Purpose:**

Clean up stuff like:

- Subscriptions
- Active fetch requests (our case)

### 7.2 Cleanup Function Syntax

```js
// normal useEffect
useEffect(() => {
  // effect code here

  // "cleanup" function - returned in useEffect
  return () => {
    // clean something up here
  };
}, [url]); // dependency - triggers when url changes
```

**When Cleanup Runs:**

1. After component unmounts
2. Before every re-render with changed dependencies

### 7.3 Order of Operations During Re-render

1. `url` changes
2. Cleanup function is triggered
3. Actual content of `useEffect` is triggered

### 7.4 JavaScript Closures Magic

**Leveraging closures:**

```js
useEffect(() => {
  // local variable for useEffect's run
  let isActive = true;

  // do fetch here

  return () => {
    // local variable from above
    isActive = false;
  };
}, [url]);
```

**How It Works:**

- Function in `useEffect` is re-created on every re-render
- `isActive` for latest run always resets to `true`
- **BUT** cleanup function runs before it
- Cleanup still has access to scope of **previous** function
- Sets it to `false`
- This is how JavaScript closures work

### 7.5 Full Implementation

```js
useEffect(() => {
  // set this closure to "active"
  let isActive = true;

  fetch(`/some-data-url/${id}`)
    .then((r) => r.json())
    .then((r) => {
      // if the closure is active - update state
      if (isActive) {
        setData(r);
      }
    });

  return () => {
    // set this closure to not active before next re-render
    isActive = false;
  };
}, [id]);
```

**Result:**

- Only the latest run (not cleaned up yet) has `isActive = true`
- All previous runs have `isActive = false`
- Only latest fetch updates state
- Data from old fetches disappears into the void

**Elegant solution using JavaScript fundamentals!**

---

## ✅ Part 8: Solution 4 — Cancel All Previous Requests

### 8.1 The Concept

**Most Direct Approach:**

Cancel all previous requests. If they never finish, state update with obsolete data never happens.

**Tool:** `AbortController` interface

### 8.2 AbortController Basics

**Simple as:**

1. Create `AbortController` in `useEffect`
2. Call `.abort()` in cleanup function

### 8.3 Implementation

```js
useEffect(() => {
  // create controller here
  const controller = new AbortController();

  // pass controller as signal to fetch
  fetch(url, { signal: controller.signal })
    .then((r) => r.json())
    .then((r) => {
      setData(r);
    });

  return () => {
    // abort the request here
    controller.abort();
  };
}, [url]);
```

**Result:**

On every re-render:

- Request in progress is cancelled
- New one becomes the only one allowed to resolve and set state

### 8.4 Error Handling

**Important:** Aborting causes promise to reject.

**You want to catch errors** to avoid scary console warnings.

**AbortController Rejection:**

Gives a specific error type, easy to exclude from regular error handling.

```js
fetch(url, { signal: controller.signal })
  .then((r) => r.json())
  .then((r) => {
    setData(r);
  })
  .catch((error) => {
    // error because of AbortController
    if (error.name === "AbortError") {
      // do nothing
    } else {
      // do something, it's a real error!
    }
  });
```

**Best Practice:** Handle Promise rejections properly regardless of `AbortController`.

---

## 🔄 Part 9: Async/Await and Race Conditions

### 9.1 Does Async/Await Change Anything?

**Short answer:** Nope, not really.

**What Async/Await Is:**

A nicer way to write **exactly the same promises**.

- Turns them into "synchronous" functions from execution flow perspective
- Doesn't change their asynchronous nature

### 9.2 Syntax Comparison

**Traditional Promises:**

```js
fetch("/some-url")
  .then((r) => r.json())
  .then((r) => setData(r));
```

**Async/Await:**

```js
const response = await fetch("/some-url");
const result = await response.json();
setData(result);
```

### 9.3 Race Conditions Persist

**Same app with async/await:**

Will have **exactly the same race condition**.

**All solutions from above apply:**

- Same reasons
- Same fixes
- Just slightly different syntax

---

## 📝 Key Takeaways

**1. What Causes Race Conditions**

Race conditions happen when:

- We update state multiple times after a promise resolves
- In the same React component
- Requests complete out of order

**Vulnerable Code:**

```js
useEffect(() => {
  fetch(url)
    .then((r) => r.json())
    .then((r) => {
      // this is vulnerable to race conditions
      setData(r);
    });
}, [url]);
```

**2. Four Solutions**

**Solution 1: Force Re-mount**

- Use `key` attribute to force component remount
- Not recommended as general solution
- Too many caveats

**Solution 2: Drop Incorrect Result**

- Compare returned result with current variable
- Use Refs to track latest value
- Don't set state if they don't match

**Solution 3: Drop All Previous Results**

- Use cleanup function in `useEffect`
- Track active closure with boolean flag
- Only latest fetch updates state

**Solution 4: Cancel Previous Requests**

- Use `AbortController`
- Cancel all previous requests in cleanup
- Most direct approach
- Handle abort errors properly

**3. Promises Fundamentals**

- Promises execute asynchronously
- JavaScript doesn't wait for them
- Multiple promises can be in flight simultaneously
- They resolve in unpredictable order

**4. React Lifecycle Matters**

- `useEffect` runs after render
- Cleanup runs before next effect or unmount
- State updates trigger re-renders
- Components can be unmounted mid-fetch

**5. Async/Await Doesn't Solve It**

- Just syntactic sugar for promises
- Race conditions still occur
- Same solutions apply

**6. Choosing a Solution**

**Use AbortController (Solution 4) when:**

- You want clean, straightforward code
- You're okay with slightly more boilerplate
- You want to actually stop network requests

**Use Closure Tracking (Solution 3) when:**

- You prefer pure JavaScript solutions
- You understand closures well
- You don't mind slightly complex logic

**Use Ref Comparison (Solution 2) when:**

- Results have reliable identifiers
- You want explicit control
- You prefer simple boolean checks

**Avoid Force Re-mount (Solution 1) unless:**

- You have very specific use case
- You understand all the caveats
- Other solutions genuinely won't work

---

## ⚠️ Common Pitfalls

1. **Ignoring race conditions** — "It's rare, won't happen to me"
2. **Not handling Promise rejections** — AbortController causes rejections
3. **Assuming async/await prevents race conditions** — It doesn't
4. **Over-using force re-mount** — Creates more problems than it solves
5. **Not understanding closures** — Critical for Solution 3
6. **Forgetting cleanup functions** — Key to Solutions 3 and 4
7. **Not considering unmounting** — Component can disappear mid-fetch
8. **Testing only slow interactions** — Race conditions appear with fast navigation

---

## 🎓 Advanced Insights

**When Race Conditions Appear:**

Most common scenarios:

- Fast navigation between routes/tabs
- Rapid filter changes in search interfaces
- Quick successive button clicks
- Auto-complete with fast typing
- Pagination with quick clicks

**Performance Considerations:**

**AbortController Benefits:**

- Actually stops network requests
- Saves bandwidth
- Reduces server load
- Cleaner browser network tab

**Closure Tracking Benefits:**

- No extra network logic
- Pure JavaScript solution
- Slightly better performance (no abort overhead)

**Testing Strategies:**

**Simulate Race Conditions:**

- Add artificial delays to responses
- Use browser throttling
- Click rapidly during development
- Test with slow network conditions

**Debugging Tips:**

- Log request IDs in `.then` callbacks
- Use React DevTools to track state changes
- Monitor Network tab for abort signals
- Add console logs with timestamps

**Library Support:**

Many data fetching libraries handle this automatically:

- React Query cancels on unmount
- SWR has built-in deduplication
- Apollo Client has request policies

**But understanding fundamentals is still crucial!**

**Cleanup Function Details:**

Runs at two times:

1. **Before every re-render** with changed dependencies
2. **On component unmount**

Order matters:

1. Dependencies change
2. Cleanup runs
3. New effect runs

**Closure Scope:**

Each `useEffect` run creates new scope:

- New variables
- New promises
- New closures

Cleanup has access to its own run's scope:

- Can modify its variables
- Those variables accessible in async callbacks
- Later runs can't access earlier scopes

**Memory Leaks Prevention:**

Always clean up:

- Cancel pending requests
- Clear timers
- Remove event listeners
- Unsubscribe from subscriptions

Race condition solutions help prevent:

- Stale data in state
- Memory leaks from pending operations
- Unexpected state updates

**AbortController Browser Support:**

- Widely supported (modern browsers)
- Polyfills available for older browsers
- Check compatibility if supporting IE11

---

## 🔜 Next Chapter Preview

In the final chapter, we'll close the conversation about advanced React patterns with: **"What to do if something goes terribly wrong?"**

Topics will include:

- Error boundaries
- Error handling strategies
- Recovery mechanisms
- User experience during errors
- Logging and monitoring
