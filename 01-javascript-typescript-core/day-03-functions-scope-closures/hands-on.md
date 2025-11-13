# Day 3 — Hands-on: Closure-Based Utilities

> **Practice Goal:** Master closures by building real-world utilities.

---

## 🎯 Exercise 1: Build a Counter

**Instructions:** Create a counter function that maintains a private count variable.

```js
function counter() {
  // Your implementation here
}

// Usage:
const count = counter();
console.log(count()); // 1
console.log(count()); // 2
console.log(count()); // 3
```

<details>
<summary>💡 Show Solution</summary>

```js
function counter() {
  let count = 0;
  return function () {
    count++;
    return count;
  };
}

// Each counter is independent
const count1 = counter();
const count2 = counter();
console.log(count1()); // 1
console.log(count1()); // 2
console.log(count2()); // 1
```

</details>

---

## 🔒 Exercise 2: Implement `once(fn)`

**Instructions:** Create a function that runs only once, no matter how many times it's called.

```js
function once(fn) {
  // Your implementation here
}

// Usage:
const initialize = once(() => {
  console.log("Initialized!");
  return "Done";
});

console.log(initialize()); // 'Initialized!' then 'Done'
console.log(initialize()); // 'Done' (no log)
```

<details>
<summary>💡 Show Solution</summary>

```js
function once(fn) {
  let hasRun = false;
  let result;

  return function (...args) {
    if (!hasRun) {
      result = fn.apply(this, args);
      hasRun = true;
    }
    return result;
  };
}

// Example: Payment processing
const processPayment = once((amount) => {
  console.log(`Processing $${amount}`);
  return { success: true };
});

console.log(processPayment(100)); // Processes
console.log(processPayment(100)); // Doesn't process again
```

</details>

---

## 🧠 Exercise 3: Build a Memoization Function

**Instructions:** Cache function results to avoid expensive recalculations.

```js
function memoize(fn) {
  // Your implementation here
}

// Usage:
const add = (a, b) => {
  console.log(`Computing ${a} + ${b}`);
  return a + b;
};

const memoizedAdd = memoize(add);
console.log(memoizedAdd(2, 3)); // 'Computing 2 + 3' then 5
console.log(memoizedAdd(2, 3)); // 5 (from cache, no log)
```

<details>
<summary>💡 Show Solution</summary>

```js
function memoize(fn) {
  const cache = {};

  return function (...args) {
    const key = JSON.stringify(args);

    if (key in cache) {
      return cache[key];
    }

    const result = fn.apply(this, args);
    cache[key] = result;
    return result;
  };
}

// Example: Fibonacci
const fibonacci = memoize((n) => {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
});

console.log(fibonacci(40)); // Fast!
```

</details>

---

## 🎫 Exercise 4: Create an ID Generator

**Instructions:** Generate unique IDs with a custom prefix.

```js
function createIdGenerator(prefix = "id") {
  // Your implementation here
}

// Usage:
const generateUserId = createIdGenerator("user");
console.log(generateUserId()); // 'user-1'
console.log(generateUserId()); // 'user-2'

const generateOrderId = createIdGenerator("order");
console.log(generateOrderId()); // 'order-1'
```

<details>
<summary>💡 Show Solution</summary>

```js
function createIdGenerator(prefix = "id") {
  let counter = 0;

  return function () {
    counter++;
    return `${prefix}-${counter}`;
  };
}

// Each generator is independent
const userId = createIdGenerator("usr");
const orderId = createIdGenerator("ord");

console.log(userId()); // 'usr-1'
console.log(orderId()); // 'ord-1'
console.log(userId()); // 'usr-2'
```

</details>

---

## 💪 Exercise 5: Build Your Own Utility

**Choose one to implement:**

### Option 1: `debounce(fn, delay)`

```js
function debounce(fn, delay) {
  // Your implementation
}

// Usage: Search input
const search = debounce((query) => {
  console.log(`Searching: ${query}`);
}, 500);

search("h"); // Won't call
search("he"); // Won't call
search("hello"); // Calls after 500ms
```

<details>
<summary>💡 Show Solution</summary>

```js
function debounce(fn, delay) {
  let timeoutId;

  return function (...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}
```

</details>

---

### Option 2: Simple Score Manager

**Requirements:**

- Private score variable
- Methods: add, deduct, get, reset

<details>
<summary>💡 Show Solution</summary>

```js
function createScoreManager() {
  let score = 0;
  let high = 0;

  return {
    add(points) {
      score += points;
      if (score > high) high = score;
      return score;
    },
    deduct(points) {
      score = Math.max(0, score - points);
      return score;
    },
    get() {
      return { current: score, high };
    },
    reset() {
      score = 0;
      return score;
    },
  };
}

// Usage
const game = createScoreManager();
console.log(game.add(100)); // 100
console.log(game.add(50)); // 150
console.log(game.deduct(30)); // 120
console.log(game.get()); // { current: 120, high: 150 }
```

</details>
