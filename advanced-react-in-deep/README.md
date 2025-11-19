# Advanced React — Learning Notes

> My personal notes and learnings from the book **"Advanced React"** by Nadia Makarevich

---

## 📖 Study Resources

**Book:** [Advanced React — What is Happening Under the Hood?](https://www.advanced-react.com/)  
**Video Series:** [Advanced React In Deep - ازاي اسبق ال AI بخطوه](https://www.youtube.com/playlist?list=PLDQ11FgmbqQNwfzL9zKuJp8BkD3mXnNeW)

---

## 📝 Chapter Notes

### Chapter 1 — Introduction to Re-renders

[📖 Read Notes](./ch1/notes.md)

**Key Learnings:**

- What a re-render is and why it happens
- State propagation through component trees
- Props myth about re-renders
- Moving state down technique
- Custom hooks behavior with state

---

### Chapter 2 — Elements, Children as Props, and Re-renders

[📖 Read Notes](./ch2/notes.md)

**Key Learnings:**

- Difference between Elements and Components
- Components as props pattern
- React's reconciliation algorithm
- Children as props optimization
- Why these patterns improve performance

---

### Chapter 3 — Configuration Concerns with Elements as Props

[📖 Read Notes](./ch3/notes.md)

**Key Learnings:**

- Elements as props for configuration
- Using React.cloneElement safely
- When this pattern breaks down
- Real-world examples from UI libraries
- Comparison with render props

---

### Chapter 14 — Data Fetching on the Client and Performance

[📖 Read Notes](./ch14/notes.md)

**Key Learnings:**

- Types of data fetching (initial vs on-demand)
- When to use external libraries vs native fetch
- What makes an app "performant"
- Browser limitations (6 parallel requests)
- Request waterfalls and how they appear
- Four solutions: Promise.all, parallel promises, data providers, pre-fetching
- Suspense reality check

---

### Chapter 15 — Data Fetching and Race Conditions

[📖 Read Notes](./ch15/notes.md)

**Key Learnings:**

- Understanding Promises and async operations
- How race conditions appear in React
- Four complete solutions:
  - Force re-mounting (not recommended)
  - Drop incorrect results (using Refs)
  - Drop all previous results (cleanup functions)
  - Cancel previous requests (AbortController)
- Async/await and race conditions
- Testing and debugging strategies

---

## 📊 Study Progress

| Chapter                                       | Status         |
| --------------------------------------------- | -------------- |
| Chapter 1: Introduction to Re-renders         | ✅ Complete    |
| Chapter 2: Elements, Children as Props        | ✅ Complete    |
| Chapter 3: Configuration Concerns             | ✅ Complete    |
| Chapter 4: Render Props                       | 🔄 In Progress |
| Chapter 14: Data Fetching and Performance     | ✅ Complete    |
| Chapter 15: Data Fetching and Race Conditions | ✅ Complete    |
