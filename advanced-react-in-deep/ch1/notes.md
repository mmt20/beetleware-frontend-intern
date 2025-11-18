# Chapter 1 — Introduction to Re-renders

> **Learning Objective:** Master React's re-rendering mechanism, understand state propagation, and optimize component performance through composition patterns.

---

## 🎯 Part 1: The Problem — Performance and UI Lag

### Scenario

You inherit a large, performance-sensitive application. Your first task is to add a simple modal dialog that opens via a button click.

### Initial Implementation

**The approach seems trivial:**

1. Add a state variable `isOpen` (boolean)
2. Add a button that sets `isOpen` to `true`
3. Conditionally render the `ModalDialog` based on `isOpen`
4. Render the rest of the application's heavy components alongside the modal

**Code Example:**

```js
const App = () => {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div className="layout">
      <Button onClick={() => setIsOpen(true)}>Open dialog</Button>

      {isOpen ? <ModalDialog onClose={() => setIsOpen(false)} /> : null}

      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </div>
  );
};
```

**Observation:**

When the button is clicked, there is a significant delay (approximately 1 second) before the modal appears. The interface feels sluggish.

---

## 🔍 Part 2: Root Cause Analysis

### 2.1 The React Component Lifecycle

To understand the lag, we must analyze React's lifecycle with its three most important stages:

#### Stage 1: Mounting

**Definition:** When a component first appears on the screen, this is called mounting.

**What Happens During Mounting:**

- React creates the component's instance for the first time
- Initializes its state
- Runs its hooks
- Appends DOM elements to the actual browser DOM
- Result: we see whatever we render in this component on the screen

#### Stage 2: Unmounting

**Definition:** This is when React detects that a component is not needed anymore.

**What Happens During Unmounting:**

- React does the final cleanup
- Destroys the component's instance and everything associated with it
- Removes the component's state from memory
- Removes the DOM element associated with it from the browser

#### Stage 3: Re-rendering

**Definition:** This is when React updates an already existing component with new information.

**What Happens During Re-rendering:**

Compared to mounting, re-rendering is lightweight:

- React re-uses the already existing component instance
- Runs the hooks again
- Does all the necessary calculations
- Updates the existing DOM element with new attributes

**Critical Concept:**

Re-rendering is one of the most important things to understand in React. Without re-renders, there would be no data updates and no interactivity. The app would be completely static.

---

### 2.2 The Chain of Re-renders

Every re-render starts with a **state update**. When we use hooks like `useState`, `useReducer`, or external state management libraries like Redux, we add interactivity to a component. The component now has a piece of data that is preserved throughout its lifecycle.

**Key Principle:**

> State update is the initial source of ALL re-renders in React apps.

**In our example:**

```js
const App = () => {
  const [isOpen, setIsOpen] = useState(false);

  return <Button onClick={() => setIsOpen(true)}>Open dialog</Button>;
};
```

**When we click the Button:**

1. We trigger the setIsOpen setter function
2. We update the isOpen state from false to true
3. As a result, the App component that holds that state re-renders itself

**Propagation Mechanism:**

After the state is updated and the App component re-renders, the new data needs to be delivered to other components that depend on it. React does this automatically:

1. React grabs all components that the initial component renders inside
2. Re-renders those components
3. Then re-renders components nested inside of them
4. Continues until it reaches the end of the chain of components

**Tree Visualization:**

If you imagine a typical React app as a tree:

- Everything DOWN from where the state update was initiated will be re-rendered
- React NEVER goes "up" the render tree when it re-renders components

In our app example:

```js
const App = () => {
  const [isOpen, setIsOpen] = useState(false);

  // Everything returned here will be re-rendered when state updates
  return (
    <div className="layout">
      <Button onClick={() => setIsOpen(true)}>Open dialog</Button>

      {isOpen ? <ModalDialog onClose={() => setIsOpen(false)} /> : null}

      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </div>
  );
};
```

Result: ALL very slow components re-render when the state changes, causing the nearly 1-second delay before the dialog appears on screen.

**Important Rule:** The only way for components at the "bottom" to affect components at the "top" of the hierarchy is for them to either explicitly call a state update in the "top" components or to pass components as functions.

---

## 🚫 Part 3: The Big Re-renders Myth

### 3.1 The Misconception

**MYTH:** "A component re-renders when its props change."

This is one of the most common misconceptions in React. Everyone believes it, no one doubts it, and it's just not true.

**REALITY:** Normal React behavior is that if a state update is triggered, React will re-render all the nested components REGARDLESS OF THEIR PROPS.

If a state update is NOT triggered, then changing props will be "swallowed" - React doesn't monitor props for changes.

**Example of Why This Doesn't Work:**

```js
const App = () => {
  // Using a local variable instead of state
  let isOpen = false;

  return (
    <div className="layout">
      {/* This won't work - no state update triggered */}
      <Button onClick={() => (isOpen = true)}>Open dialog</Button>

      {/* Will never show up */}
      {isOpen ? <ModalDialog onClose={() => (isOpen = false)} /> : null}
    </div>
  );
};
```

When the Button is clicked:

- The local isOpen variable changes
- BUT the React lifecycle is not triggered
- The render output is never updated
- The ModalDialog will never show up

---

### 3.2 When Props Actually Matter

In the context of re-renders, whether props have changed matters in ONLY ONE CASE: if the component is wrapped in the React.memo higher-order component.

**React.memo Behavior:**

- React stops its natural chain of re-renders
- Checks the props first
- If NONE of the props change: re-renders stop there
- If even ONE SINGLE prop changes: re-renders continue as usual

Note: Preventing re-renders with memoization is complicated and has several caveats, covered in detail in Chapter 5.

---

## ✅ Part 4: Moving State Down (The Solution)

### 4.1 Analyzing State Usage

Now that we understand how React re-renders components, we can fix the original problem. Let's examine where we use the modal dialog state:

```js
const App = () => {
  // State declared here
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div className="layout">
      {/* State used here */}
      <Button onClick={() => setIsOpen(true)}>Open dialog</Button>

      {/* State used here */}
      {isOpen ? <ModalDialog onClose={() => setIsOpen(false)} /> : null}

      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </div>
  );
};
```

**OBSERVATION:** The state is relatively isolated - we use it only on the Button component and in ModalDialog itself. The rest of the code (all those very slow components) doesn't depend on it and therefore doesn't actually need to re-render when this state changes.

This is a classic example of an "unnecessary re-render."

---

### 4.2 The Fix

**Step 1: Extract components that depend on the state into a smaller component**

```js
const ButtonWithModalDialog = () => {
  const [isOpen, setIsOpen] = useState(false);

  // Render only Button and ModalDialog here
  return (
    <>
      <Button onClick={() => setIsOpen(true)}>Open dialog</Button>

      {isOpen ? <ModalDialog onClose={() => setIsOpen(false)} /> : null}
    </>
  );
};
```

**Step 2: Use this new component in the original App**

```js
const App = () => {
  return (
    <div className="layout">
      {/_ Component with state inside _/}
      <ButtonWithModalDialog />

      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </div>
  );
};
```

---

### 4.3 What Happens Now

When the Button is clicked:

1. The state update is still triggered
2. Some components re-render because of it
3. BUT it only happens with components INSIDE ButtonWithModalDialog
4. It's just a tiny button and the dialog that should be rendered anyway
5. The rest of the app is safe

**Conceptual Diagram:**

```su
Before (state in App):
App (state here)
├─ Button (triggers re-render of everything below App)
├─ ModalDialog
├─ VerySlowComponent (re-renders unnecessarily)
├─ BunchOfStuff (re-renders unnecessarily)
└─ OtherStuffAlsoComplicated (re-renders unnecessarily)
```

After (state moved down):

```su
App
├─ ButtonWithModalDialog (state here)
│ ├─ Button (triggers re-render only within this branch)
│ └─ ModalDialog
├─ VerySlowComponent (NOT re-rendered)
├─ BunchOfStuff (NOT re-rendered)
└─ OtherStuffAlsoComplicated (NOT re-rendered)
```

We essentially created a new sub-branch inside our render tree and moved our state down to it.

RESULT: The modal dialog appears instantly. We fixed a big performance problem with a simple composition technique.

---

## ⚠️ Part 5: The Danger of Custom Hooks

### 5.1 Understanding Custom Hooks and Re-renders

Another very important concept when dealing with state, re-renders, and performance is custom hooks.

Custom hooks were introduced to abstract away stateful logic. It's common to see logic like our dialog implementation extracted into a custom hook:

```js
const useModalDialog = () => {
  const [isOpen, setIsOpen] = useState(false);

  return {
    isOpen,
    open: () => setIsOpen(true),
    close: () => setIsOpen(false),
  };
};
```

Usage:

```js
const App = () => {
  // State is in the hook now
  const { isOpen, open, close } = useModalDialog();

  return (
    <div className="layout">
      {/* Use methods from the hook */}
      <Button onClick={open}>Open dialog</Button>

      {isOpen ? <ModalDialog onClose={close} /> : null}

      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </div>
  );
};
```

---

### 5.2 Why This Is Dangerous

The hook **HIDES** the fact that we have state in the app. But the state is still there! Every time it changes, it will still trigger a re-render of the component that uses this hook.

It doesn't even matter whether:

- This state is used in the App directly
- The hook returns anything at all

**Extreme Example:**

```js
const useModalDialog = () => {
  const [width, setWidth] = useState(0);

  useEffect(() => {
    const listener = () => {
      setWidth(window.innerWidth);
    };

    window.addEventListener("resize", listener);

    return () => window.removeEventListener("resize", listener);
  }, []);

  // Return is the same as before
  return {
    isOpen,
    open: () => setIsOpen(true),
    close: () => setIsOpen(false),
  };
};
```

The entire App component will re-render on every window resize, even though the width value is not even returned from the hook!

**ANALOGY:**

Hooks are essentially just pockets in your trousers. If you have a 10-kilogram dumbbell:

- Carrying it in your hands: hard to run
- Putting it in your pocket: still hard to run (10kg is still on your person)
- Putting it in a self-driving trolley: you can run freely

Components for state are that trolley.

---

### 5.3 Chain of Hooks

The same logic applies to hooks that use other hooks. Anything that can trigger a re-render, however deep in the chain of hooks, will trigger a re-render in the component that uses that very first hook.

Example:

```js
const useResizeDetector = () => {
  const [width, setWidth] = useState(0);

  useEffect(() => {
    const listener = () => {
      setWidth(window.innerWidth);
    };

    window.addEventListener("resize", listener);
    return () => window.removeEventListener("resize", listener);
  }, []);

  return null; // Returns nothing!
};

const useModalDialog = () => {
  // Doesn't even use the return value
  useResizeDetector();

  return {
    isOpen,
    open: () => setIsOpen(true),
    close: () => setIsOpen(false),
  };
};

const App = () => {
  // This hook uses useResizeDetector underneath
  // The entire App will re-render on every resize!
  const { isOpen, open, close } = useModalDialog();

  return /* same return */;
};
```

---

### 5.4 The Fix for Hooks

You still need to extract the button, dialog, and the custom hook into a component:

```js
const ButtonWithModalDialog = () => {
  const { isOpen, open, close } = useModalDialog();

  return (
    <>
      <Button onClick={open}>Open dialog</Button>
      {isOpen ? <ModalDialog onClose={close} /> : null}
    </>
  );
};

const App = () => {
  return (
    <div className="layout">
      <ButtonWithModalDialog />

      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </div>
  );
};
```

**BEST PRACTICE:** Where you put state is very important. To avoid future performance problems, isolate state as much as possible to the smallest and lightest components possible.

---

## 📝 Key Takeaways

**1. Re-rendering Fundamentals**

- Re-rendering is how React updates components with new data
- Without re-renders, there would be no interactivity in our apps
- Re-render is lightweight compared to mounting/unmounting
- React re-uses the existing instance during re-renders

**2. State Is the Source**

- State update is the initial source of ALL re-renders
- Every re-render starts with state
- No state update = no re-render

**3. Re-render Propagation**

- If a component's re-render is triggered, ALL nested components inside will be re-rendered
- React propagates re-renders DOWN the tree, never UP
- This happens regardless of props

**4. The Props Myth**

- Components re-render when their PARENT re-renders
- Props changing does NOT automatically cause re-renders
- During normal React re-renders (without memoization), props change doesn't matter
- Components will re-render even if they don't have any props
- Exception: Components wrapped in React.memo

**5. Moving State Down Pattern**

- Extract state and components that use it into a smaller component
- This isolates re-renders to a smaller part of the tree
- Protects heavy components from unnecessary re-renders
- Simple and effective performance optimization

**6. Custom Hooks Warning**

- Hooks are "pockets" for state within a component
- State update in a hook triggers re-render of the component using it
- This happens even if the state itself is not used
- Even if the state is not returned from the hook
- In chains of hooks, any state update anywhere in the chain triggers re-render
- Always consider where to place your custom hooks carefully

**7. Architectural Principle**

- Ideally, isolate state as much as possible
- Place state in the smallest, lightest components possible
- This prevents future performance problems
- Makes the app easier to reason about and maintain

---

## ⚠️ Common Pitfalls

1. Assuming props changes cause re-renders
2. Not considering where to place state in the component tree
3. Hiding state in custom hooks without considering re-render implications
4. Not extracting state when components grow larger
5. Reaching for React.memo before trying compositional solutions

---

## 🎓 Advanced Insights

**Performance Implications:**

The "moving state down" technique is often more effective than memoization because:

- It's simpler and has fewer edge cases
- No need to worry about memoization dependencies
- No risk of breaking memoization accidentally
- More maintainable code
- Better separation of concerns

**Re-render Cost:**

Not all re-renders are expensive. A re-render becomes a performance problem only when:

- The component is complex and takes time to execute
- The component has many children that also re-render
- Re-renders happen very frequently (e.g., on every keystroke)
- The cumulative effect of multiple re-renders causes noticeable lag

**When to Optimize:**

Don't prematurely optimize. Move state down when:

- You observe actual performance problems
- Components are provably slow (use React DevTools Profiler)
- The state is logically isolated to a subset of the UI
- The optimization makes the code clearer, not more complex

---

## 🔜 Next Chapter Preview

In Chapter 2, we'll explore another pattern that helps with isolating re-renders: passing components as props, particularly the "children as props" pattern. We'll also dig deeper into how React triggers re-renders and what an Element is versus a Component.
