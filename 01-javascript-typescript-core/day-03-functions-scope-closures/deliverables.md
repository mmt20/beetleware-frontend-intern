# Closure-Based Helpers Library

> **Deliverable:** Implement closure-based utility functions demonstrating practical, production-ready closure patterns.

---

## counter()

Creates a counter with private state.

```javascript
function counter(initialValue = 0) {
  let count = initialValue;

  return {
    increment() {
      return ++count;
    },
    decrement() {
      return --count;
    },
    get() {
      return count;
    },
    reset() {
      return (count = 0);
    },
  };
}
```

**Usage**:

```javascript
const c = counter();
c.increment(); // 1
c.increment(); // 2
c.decrement(); // 1
```

---

## memoize()

Caches function results based on arguments.

```javascript
function memoize(fn) {
  if (typeof fn !== "function") throw new TypeError("Function required");

  const cache = new Map();

  return function memoized(...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      return cache.get(key);
    }

    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}
```

**Usage**:

```javascript
const add = memoize((a, b) => {
  console.log("Computing...");
  return a + b;
});

add(2, 3); // Logs "Computing..." → 5
add(2, 3); // Returns 5 (no log)
```

---

## debounce(fn, delay)

Delays execution, cancels previous calls within the time window.

```javascript
function debounce(fn, delay) {
  if (typeof fn !== "function") throw new TypeError("Function required");
  if (typeof delay !== "number") throw new TypeError("Delay required");

  let timeoutId = null;
  let lastArgs = null;
  let lastThis = null;

  function debounced(...args) {
    lastArgs = args;
    lastThis = this;

    if (timeoutId) clearTimeout(timeoutId);

    timeoutId = setTimeout(() => {
      fn.apply(lastThis, lastArgs);
      timeoutId = null;
    }, delay);
  }

  debounced.cancel = () => {
    if (timeoutId) clearTimeout(timeoutId);
    timeoutId = null;
  };

  return debounced;
}
```

**Usage**:

```javascript
const search = debounce((query) => {
  console.log("Searching:", query);
}, 300);

search("a");
search("ab");
search("abc");
// Only logs "Searching: abc" after 300ms
```

---

## Export

```javascript
export { counter, memoize, debounce };
```
