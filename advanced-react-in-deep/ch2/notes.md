# Chapter 2 — Elements, Children as Props, and Re-renders

> **Learning Objective:** Understand the fundamental difference between Components and Elements, master the components-as-props pattern, and learn how React's reconciliation algorithm determines re-renders.

---

## 📚 What You'll Learn

- How passing components as props can improve performance
- How exactly React triggers re-renders
- Why components as props are not affected by re-renders
- What an Element is and how it differs from a Component
- The basics of React reconciliation and diffing
- The "children as props" pattern and how it prevents re-renders

---

## 🎯 Part 1: The Problem — State in a Scrollable Area

### 1.1 The Scenario

You're working on a large, performance-sensitive app with a scrollable content area. The layout has:

- A sticky header
- A collapsible sidebar on the left
- Main scrollable functionality in the middle

**Current Code:**

```js
const App = () => {
  return (
    <div className="scrollable-block">
      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </div>
  );
};
```

The scrollable area uses `overflow: auto` in CSS.

---

### 1.2 New Requirement

Implement a floating block that:

- Appears at the bottom when user scrolls
- Slowly moves to the top as user scrolls down
- Slowly moves down and disappears if user scrolls up
- Must be smooth and lag-free

**Naive Implementation:**

```js
const MainScrollableArea = () => {
  const [position, setPosition] = useState(300);

  const onScroll = (e) => {
    // Calculate position based on scrolled value
    const calculated = getPosition(e.target.scrollTop);
    // Save to state
    setPosition(calculated);
  };

  return (
    <div className="scrollable-block" onScroll={onScroll}>
      {/* Pass position to the movable component */}
      <MovingBlock position={position} />

      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </div>
  );
};
```

**Problem with This Approach:**

Every scroll event triggers a state update. As we know from Chapter 1:

- State update triggers re-render of the component
- All nested components also re-render
- All the very slow components re-render on every scroll
- Scrolling becomes slow and laggy

**Challenge:**

We can't easily extract the state into a component like in Chapter 1 because:

- The `setPosition` is used in `onScroll`
- `onScroll` is attached to the div that wraps everything
- The state and its usage are intertwined with the wrapper structure

---

## ✅ Part 2: The Solution — Components as Props

### 2.1 The Pattern

We CAN still extract state and everything needed into a component, but we pass the slow components as props instead.

**Step 1: Extract state into a new component**

```js
const ScrollableWithMovingBlock = () => {
  const [position, setPosition] = useState(300);

  const onScroll = (e) => {
    const calculated = getPosition(e.target.scrollTop);
    setPosition(calculated);
  };

  return (
    <div className="scrollable-block" onScroll={onScroll}>
      <MovingBlock position={position} />
      {/* Slow components used to be here */}
    </div>
  );
};
```

**Step 2: Pass slow components as props from parent**

```js
const App = () => {
  const slowComponents = (
    <>
      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </>
  );

  return <ScrollableWithMovingBlock content={slowComponents} />;
};
```

**Step 3: Accept and render the prop**

```js
const ScrollableWithMovingBlock = ({ content }) => {
  const [position, setPosition] = useState(0);

  const onScroll = () => {
    /* same as before */
  };

  return (
    <div className="scrollable-block" onScroll={onScroll}>
      <MovingBlock position={position} />
      {content}
    </div>
  );
};
```

---

### 2.2 Why This Works

When state updates in `ScrollableWithMovingBlock`:

1. The component re-renders
2. It's just a div with a moving block
3. The slow components are passed through props
4. They are OUTSIDE this component in the render tree
5. They belong to the PARENT (App)
6. React never goes UP the tree during re-renders
7. Slow components don't re-render
8. Scrolling is smooth and lag-free

---

## 🔍 Part 3: Elements vs Components (The Fundamental Difference)

### 3.1 What is a Component?

**Definition:** A Component is just a function that returns Elements, which React converts into DOM elements.

**Here's the simplest component:**

```js
const Parent = () => {
  return <Child />;
};
```

If it has props, those are just the first argument:

```js
const Parent = (props) => {
  return <Child />;
};
```

---

### 3.2 What is an Element?

**Definition:** When we write `<Child />`, we create an Element. The HTML-like syntax (JSX) is syntax sugar for the `React.createElement` function.

**These are equivalent:**

```js
<Child />;
// Is the same as:
React.createElement(Child, null, null);
```

**An Element is an Object:**

The object definition for `<Child />` looks like:

```js
{
  type: Child,
  props: {},
  // ... lots of other internal React stuff
}
```

This object tells React:

- The Parent component wants to render the Child component
- With no props

**Elements can be DOM elements or components:**

For components:

```js
const Child = () => {
  return <h1>Some title</h1>;
};
```

The Child returns an Element with type as string:

```js
{
  type: "h1",
  // ... props and internal React stuff
}
```

---

### 3.3 React's Rendering Process

**When React renders (calls those functions):**

1. If type is a STRING: generate the HTML element of that type
2. If type is a FUNCTION: call it and iterate over its returned tree
3. Continue until the entire tree of DOM nodes is ready

**Example tree:**

```js
const Component = () => {
  return (
    <div>
      <Input placeholder="Text1" id="1" />
      <Input placeholder="Text2" id="2" />
    </div>
  );
};
```

**Object representation:**

```js
{
  type: 'div',
  props: {
    children: [
      {
        type: Input,
        props: { id: "1", placeholder: "Text1" }
      },
      {
        type: Input,
        props: { id: "2", placeholder: "Text2" }
      }
    ]
  }
}
```

**Resolves to HTML:**

```html
<div>
  <input placeholder="Text1" id="1" />
  <input placeholder="Text2" id="2" />
</div>
```

Finally, React appends these DOM elements to the document with JavaScript's `appendChild`.

---

## 🔄 Part 4: How Re-renders Actually Work

### 4.1 The Reconciliation Process

**After initial mount, when state updates:**

1. React needs to update all elements with new data
2. It begins its journey through the tree from where state updated
3. For each element, it compares "before" and "after" the state update

**Example:**

```js
const Component = () => {
  return <Input />;
};
```

React understands that Component returns:

```js
{
  type: Input,
  // ... other internal stuff
}
```

**During re-render, React compares:**

- Type field of object from "before"
- Type field of object from "after"

**If type is the SAME:**

- Input component is marked as "needs update"
- Its re-render is triggered

**If type CHANGES:**

- React unmounts the "previous" component
- Mounts the "next" component

Type is just a reference to a function, and that reference doesn't change between re-renders.

---

### 4.2 Conditional Rendering Example

```js
const Component = () => {
  if (isCompany) return <Input />;
  return <TextPlaceholder />;
};
```

If `isCompany` flips from true to false:

**Before:**

```js
{
  type: Input,
  // ...
}
```

**After:**

```js
{
  type: TextPlaceholder,
  // ...
}
```

**Result:** Type changed: Input is unmounted, TextPlaceholder is mounted. Everything associated with Input (including its state) is destroyed.

---

## 💡 Part 5: Why Components as Props Work

### 5.1 The Mechanism

**Consider this example:**

```js
const Parent = ({ child }) => {
  const [state, setState] = useState();
  return child;
};

// Usage:
<Parent child={<Child />} />;
```

**When state updates in Parent:**

1. React compares what Parent returns before and after
2. Before: reference to child object
3. After: reference to child object (same object!)
4. The child object was created OUTSIDE Parent's scope
5. It doesn't change when Parent is called
6. Comparison returns TRUE
7. React skips re-render of Child

---

### 5.2 Visual Representation

**Without props pattern:**

```
Parent re-renders
├─ Child re-renders (defined inside Parent)
└─ All descendants re-render
```

**With props pattern:**

```
Parent re-renders
└─ child (from props, created outside, doesn't re-render)
```

**This is exactly what happened in our scrollable example:**

```js
const ScrollableWithMovingBlock = ({ content }) => {
  const [position, setPosition] = useState(300);

  const onScroll = () => {
    /* ... */
  };

  return (
    <div className="scrollable-block" onScroll={onScroll}>
      <MovingBlock position={position} />
      {content} {/* Created outside, won't re-render */}
    </div>
  );
};
```

**When setPosition triggers re-render:**

- React compares object definitions
- content object is exactly the same (created outside)
- Comparison returns TRUE
- React skips re-render of content
- MovingBlock WILL re-render (created inside)

---

## 👶 Part 6: Children as Props (The Special Case)

### 6.1 Props are Just an Object

**Definition:** Props are the first argument to a component function. Everything we extract from it is a prop. Including children.

**These are the same:**

```js
const Parent = ({ child }) => {
  return child;
};

const Parent = ({ children }) => {
  return children;
};

// Both work:
<Parent child={<Child />} />
<Parent children={<Child />} />
```

---

### 6.2 The Special JSX Syntax

For children prop, we have special nesting syntax:

```jsx
<Parent>
  <Child />
</Parent>

// Exactly the same as:
<Parent children={<Child />} />
```

**Object representation:**

```js
{
  type: Parent,
  props: {
    children: {
      type: Child,
      // ...
    }
  }
}
```

---

### 6.3 Performance Benefits

Children passed through props have the SAME performance benefits as any other props pattern. They won't be affected by state changes in the component that receives them.

**Improved Scrollable Example:**

Before (using content prop):

```js
const App = () => {
  const slowComponents = (
    <>
      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </>
  );

  return <ScrollableWithMovingBlock content={slowComponents} />;
};
```

After (using children):

```js
const App = () => {
  return (
    <ScrollableWithMovingBlock>
      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </ScrollableWithMovingBlock>
  );
};
```

**Implementation change:**

```js
const ScrollableWithMovingBlock = ({ children }) => {
  const [position, setPosition] = useState(0);

  const onScroll = () => {
    /* ... */
  };

  return (
    <div className="scrollable-block" onScroll={onScroll}>
      <MovingBlock position={position} />
      {children}
    </div>
  );
};
```

**Result:** This is cleaner, more readable, and maintains the same performance benefits.

---

## 🎓 Part 7: Deeper Dive — Reconciliation and Diffing

### 7.1 What is the Fiber Tree?

**Definition:** When React calls component functions and executes code, it builds a tree of Element objects. This is called the Fiber Tree (sometimes Virtual DOM).

Actually, it's TWO trees:

- One before re-render
- One after re-render

---

### 7.2 The Diffing Algorithm

React compares these trees to extract information for the browser:

- Which DOM elements need to be updated
- Which need to be removed
- Which need to be added

This is the "reconciliation" algorithm.

---

### 7.3 The Crucial Part for Performance

If an Element object before and after re-render is EXACTLY THE SAME object:

- React skips re-render of that Component
- All nested components are also skipped
- React moves on to the next Element

**"Exactly the same" means:**

```js
Object.is(ElementBeforeRerender, ElementAfterRerender) === true;
```

React does NOT perform deep comparison of objects.

**When Comparison Returns False:**

If comparison returns false:

1. React looks at the type property
2. If type is the SAME: re-render the component
3. If type CHANGES: remove old component, mount new one

---

### 7.4 Parent/Child Example with State

```js
const Parent = (props) => {
  const [state, setState] = useState();
  return <Child />;
};
```

**When setState is called:**

1. React re-renders Parent
2. Parent function is called
3. Returns `<Child />` object
4. This object is defined LOCALLY to Parent function
5. On every re-render, object is RE-CREATED
6. Object.is on "before" and "after" returns FALSE
7. Child re-renders

**With Props:**

```js
const Parent = ({ child }) => {
  const [state, setState] = useState();
  return child;
};

// Usage:
<Parent child={<Child />} />;
```

**When state updates:**

1. React re-renders Parent
2. Parent returns child reference
3. child object created OUTSIDE Parent scope
4. Object doesn't change when Parent is called
5. Object.is returns TRUE
6. Child does NOT re-render

---

## 📝 Key Takeaways

**1. Components as Props Pattern**

- Pass heavy components as props to isolate them from state changes
- Components created outside a component's scope won't re-render when that component's state changes
- This works because React compares object references, not content

**2. Elements vs Components**

- A Component is a function that returns Elements
- An Element is an object with a type and props
- JSX syntax is sugar for React.createElement()
- Elements can have string types (DOM) or function types (components)

**3. React's Reconciliation**

- React compares Element objects before and after re-render
- If Object.is() returns true, React skips re-rendering
- If type changes, React unmounts old component and mounts new one
- React does NOT perform deep comparison of objects

**4. Children as Props**

- children is just a special prop name
- Can use nesting syntax: `<Parent><Child /></Parent>`
- Has the same performance benefits as any props pattern
- Makes code cleaner and more readable

**5. Performance Strategy**

- Extract state to smallest possible component
- Pass slow components as props (or children) when possible
- Understand that props pattern works due to object reference stability
- Use children pattern for better readability when appropriate

---

## ⚠️ Common Pitfalls

1. Creating components inside other components (re-creates on every render)
2. Not understanding the difference between Elements and Components
3. Thinking props are "special" when they're just function arguments
4. Not leveraging children pattern for cleaner component composition
5. Forgetting that React compares by reference, not by value

---

## 🔜 Next Chapter Preview

In Chapter 3, we'll explore React.memo and when to use it, understanding its limitations and proper use cases for preventing unnecessary re-renders through memoization.
