# Day 2: Control Flow + Arrays & Objects

> **Learning Objective:** Master JavaScript's control flow, array, and object systems to write performant, maintainable, and production-ready code using modern ES6+ patterns.

---

## 1. Control Flow Fundamentals

### If/Else Statements

Decision-making based on conditions. Understanding truthiness and proper branching strategy is crucial for clean, maintainable code.

#### Basic Conditional Logic

```javascript
// Basic if/else
const age = 25;
if (age >= 18) {
  console.log("Adult");
} else {
  console.log("Minor");
}

// If/else if/else chain
const score = 85;
if (score >= 90) {
  console.log("A");
} else if (score >= 80) {
  console.log("B");
} else if (score >= 70) {
  console.log("C");
} else {
  console.log("F");
}

// Ternary operator (inline if/else)
const status = age >= 18 ? "adult" : "minor";
const message = user ? `Hello, ${user.name}` : "Hello, guest";
```

#### Truthy and Falsy Values

Understanding JavaScript's type coercion in conditionals:

```javascript
// Falsy values: false, 0, -0, 0n, "", null, undefined, NaN
if (0) {
  // Never executes
}
if ("") {
  // Never executes
}
if (null) {
  // Never executes
}

// Everything else is truthy
if ([]) {
  // Executes! Empty arrays are truthy
}
if ({}) {
  // Executes! Empty objects are truthy
}
if ("0") {
  // Executes! Non-empty strings are truthy
}

// Common patterns
const username = input || "Anonymous"; // ⚠️ Fails for "" but you want empty string
const username = input ?? "Anonymous"; // ✅ Only defaults for null/undefined

// Explicit checks are clearer
if (array.length > 0) {
  // Better than if (array.length)
}
if (user !== null && user !== undefined) {
  // More explicit than if (user)
}
```

#### Short-Circuit Evaluation

Logical operators return values, not just booleans:

```javascript
// AND (&&) - returns first falsy or last value
const result1 = true && "hello"; // 'hello'
const result2 = false && "hello"; // false
const result3 = null && "hello"; // null

// OR (||) - returns first truthy or last value
const result4 = false || "hello"; // 'hello'
const result5 = true || "hello"; // true
const result6 = null || undefined; // undefined

// Practical uses
// 1. Conditional execution
isLoggedIn && redirectToDashboard(); // Only calls if true

// 2. Default values
const port = config.port || 3000; // ⚠️ Fails if port = 0
const port = config.port ?? 3000; // ✅ Only for null/undefined

// 3. Conditional object properties
const user = {
  name: "Alice",
  ...(isAdmin && { role: "admin" }), // Only adds if isAdmin is true
  ...(email && { email }), // Only adds if email exists
};

// 4. Guard clauses
function processUser(user) {
  user && user.isActive && doSomething(user);
  // More readable:
  if (!user || !user.isActive) return;
  doSomething(user);
}
```

**Best Practices:**

- Use ternary for simple assignments only
- Avoid nested ternaries (hard to read)
- Early returns improve readability and reduce nesting
- Prefer nullish coalescing (`??`) over OR (`||`) for defaults

```javascript
// ❌ Avoid deep nesting
function processUser(user) {
  if (user) {
    if (user.isActive) {
      if (user.hasPermission) {
        return "Allowed";
      }
    }
  }
  return "Denied";
}

// ✅ Use early returns (guard clauses)
function processUser(user) {
  if (!user) return "Denied";
  if (!user.isActive) return "Denied";
  if (!user.hasPermission) return "Denied";
  return "Allowed";
}

// ✅ Even better: combine conditions
function processUser(user) {
  if (!user?.isActive || !user?.hasPermission) return "Denied";
  return "Allowed";
}
```

### Switch Statements

Elegant multi-way branching for discrete values. Understanding fallthrough and performance characteristics is key.

#### Basic Switch Syntax

```javascript
// Basic switch
const day = "Monday";
switch (day) {
  case "Monday":
    console.log("Start of work week");
    break;
  case "Friday":
    console.log("TGIF!");
    break;
  case "Saturday":
  case "Sunday":
    console.log("Weekend!");
    break;
  default:
    console.log("Midweek");
}
```

#### Fallthrough Control

```javascript
// Intentional fallthrough (grouping cases)
function getQuarter(month) {
  switch (month) {
    case 1:
    case 2:
    case 3:
      return "Q1";
    case 4:
    case 5:
    case 6:
      return "Q2";
    case 7:
    case 8:
    case 9:
      return "Q3";
    case 10:
    case 11:
    case 12:
      return "Q4";
    default:
      throw new Error("Invalid month");
  }
}

// Partial fallthrough with shared logic
function processAction(action) {
  let result;
  switch (action) {
    case "start":
      result = "Starting...";
    // Falls through
    case "continue":
      console.log("Process continuing");
      break;
    case "stop":
      result = "Stopped";
      break;
  }
  return result;
}
```

#### Modern Alternatives: Object Lookup

Often cleaner and more maintainable than switch:

```javascript
// Basic object lookup
const dayMessages = {
  Monday: "Start of work week",
  Friday: "TGIF!",
  Saturday: "Weekend!",
  Sunday: "Weekend!",
};
const message = dayMessages[day] || "Midweek";

// Function map for actions
const actions = {
  create: () => createUser(),
  update: () => updateUser(),
  delete: () => deleteUser(),
  list: () => listUsers(),
};

// Safe execution with optional chaining
const result = actions[action]?.();

// With parameters
const operations = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
  multiply: (a, b) => a * b,
  divide: (a, b) => (b !== 0 ? a / b : NaN),
};
const result = operations[op]?.(x, y) ?? "Invalid operation";

// Map for complex return values
const statusConfig = {
  success: { color: "green", icon: "✓", retry: false },
  error: { color: "red", icon: "✗", retry: true },
  pending: { color: "yellow", icon: "⏳", retry: false },
};
const config = statusConfig[status] || statusConfig.error;
```

#### Performance Notes

```javascript
// Switch vs Object Lookup Performance:
// - Switch: O(1) for small sets, optimized by JS engines with jump tables
// - Object lookup: O(1) average, but has property access overhead
// - Switch is slightly faster for < 10 cases
// - Object lookup is more maintainable and flexible

// ✅ Use switch when:
// - Multiple cases need same logic (fallthrough)
// - Working with primitive values (strings, numbers)
// - Performance-critical hot paths with < 10 branches
// - Complex conditions beyond simple equality

// ✅ Use object lookup when:
// - Simple value mapping
// - Many cases (> 10)
// - Need dynamic runtime configuration
// - Want to pass around the mapping
```

**When to use Switch vs Object Lookup:**

- **Switch**: Multiple cases need same logic, complex conditions, or < 10 branches in hot paths
- **Object lookup**: Simple value mapping, many cases, need flexibility/testability

---

## 2. Loops & Iteration

Understanding when to use each loop type and the difference between iterables and enumerables is crucial for writing efficient JavaScript.

### For Loops

Traditional iteration with full control over the iteration process.

```javascript
// Classic for loop - full control
for (let i = 0; i < 5; i++) {
  console.log(i); // 0, 1, 2, 3, 4
}

// Multiple variables
for (let i = 0, j = 10; i < 5; i++, j--) {
  console.log(i, j); // 0 10, 1 9, 2 8, 3 7, 4 6
}

// Iterate backwards
for (let i = arr.length - 1; i >= 0; i--) {
  console.log(arr[i]);
}

// Step by 2
for (let i = 0; i < 10; i += 2) {
  console.log(i); // 0, 2, 4, 6, 8
}
```

### For...of (Iterables)

Modern way to iterate over **values** of iterables (Arrays, Strings, Maps, Sets, NodeLists, etc.)

```javascript
// Arrays
const colors = ["red", "green", "blue"];
for (const color of colors) {
  console.log(color); // red, green, blue
}

// Strings
for (const char of "hello") {
  console.log(char); // h, e, l, l, o
}

// With index using entries()
for (const [index, value] of colors.entries()) {
  console.log(`${index}: ${value}`);
}

// Maps
const map = new Map([
  ["a", 1],
  ["b", 2],
]);
for (const [key, value] of map) {
  console.log(key, value);
}

// Sets
const set = new Set([1, 2, 3]);
for (const value of set) {
  console.log(value);
}

// ⚠️ Does NOT work with plain objects (not iterable)
const obj = { a: 1, b: 2 };
// for (const item of obj) {} // ❌ TypeError: obj is not iterable
```

### For...in (Enumerables)

Iterates over **enumerable property keys** (mainly for objects, but works on arrays too).

```javascript
// Objects - primary use case
const user = { name: "Alice", age: 30, city: "NYC" };
for (const key in user) {
  console.log(`${key}: ${user[key]}`);
  // name: Alice
  // age: 30
  // city: NYC
}

// ⚠️ Works on arrays but NOT recommended
const arr = ["a", "b", "c"];
for (const index in arr) {
  console.log(index); // "0", "1", "2" (strings, not numbers!)
}

// ❌ Problem: includes inherited properties
function Parent() {
  this.parentProp = "parent";
}
function Child() {
  this.childProp = "child";
}
Child.prototype = new Parent();
const child = new Child();

for (const key in child) {
  console.log(key); // childProp, parentProp (includes inherited!)
}

// ✅ Solution: Use hasOwnProperty
for (const key in child) {
  if (child.hasOwnProperty(key)) {
    console.log(key); // Only childProp
  }
}

// ✅ Better: Use Object.keys/values/entries
for (const key of Object.keys(user)) {
  console.log(key); // Only own properties
}
```

### Iterables vs. Enumerables

Understanding the difference is essential:

```javascript
// ITERABLE: Has [Symbol.iterator] method
// - Arrays, Strings, Maps, Sets, TypedArrays, NodeLists
// - Can use for...of
// - Can use spread operator

const arr = [1, 2, 3];
console.log(arr[Symbol.iterator]); // ƒ values() { [native code] }
const spread = [...arr]; // Works ✅

// ENUMERABLE: Has enumerable properties
// - Plain objects, arrays (indexes are enumerable)
// - Can use for...in
// - Object.keys() returns enumerable properties

const obj = { a: 1, b: 2 };
for (const key in obj) {
} // Works ✅
// for (const val of obj) {} // ❌ Not iterable

// Arrays are both iterable AND enumerable
for (const val of arr) {
} // Works (iterable)
for (const key in arr) {
} // Works (enumerable) but avoid!

// Making objects iterable
const iterableObj = {
  a: 1,
  b: 2,
  c: 3,
  [Symbol.iterator]() {
    const keys = Object.keys(this);
    let index = 0;
    return {
      next: () => {
        if (index < keys.length) {
          const key = keys[index++];
          return { value: this[key], done: false };
        }
        return { done: true };
      },
    };
  },
};

for (const value of iterableObj) {
  console.log(value); // 1, 2, 3
}
```

### While Loops

Condition-based iteration when iteration count is unknown.

```javascript
// While loop
let count = 0;
while (count < 5) {
  console.log(count);
  count++;
}

// Reading until condition
let input = "";
while (input !== "exit") {
  input = prompt('Enter "exit" to quit');
}

// ⚠️ Infinite loop if condition never becomes false
while (true) {
  if (shouldExit) break;
}
```

### Do...While Loops

Executes at least once, then checks condition.

```javascript
// Do...while (executes at least once)
let input;
do {
  input = prompt('Enter "exit" to quit');
} while (input !== "exit");

// Useful when you need to execute before checking
do {
  const result = fetchData();
  if (result.success) break;
} while (retryCount++ < 3);
```

### Loop Control Statements

```javascript
// Break: exit loop entirely
for (let i = 0; i < 10; i++) {
  if (i === 5) break;
  console.log(i); // 0, 1, 2, 3, 4
}

// Continue: skip to next iteration
for (let i = 0; i < 5; i++) {
  if (i === 2) continue;
  console.log(i); // 0, 1, 3, 4
}

// Labeled statements (rarely used, but powerful)
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (i === 1 && j === 1) break outer; // Breaks outer loop
    console.log(i, j);
  }
}
// Output: 0 0, 0 1, 0 2, 1 0

// Continue with labels
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (j === 1) continue outer; // Continues outer loop
    console.log(i, j);
  }
}
```

### When to Replace Loops with Array Methods

Modern JavaScript favors declarative array methods over imperative loops:

```javascript
// ❌ Imperative (how to do it)
const doubled = [];
for (let i = 0; i < numbers.length; i++) {
  doubled.push(numbers[i] * 2);
}

// ✅ Declarative (what to do)
const doubled = numbers.map((n) => n * 2);

// ❌ Filtering with loop
const evens = [];
for (const num of numbers) {
  if (num % 2 === 0) {
    evens.push(num);
  }
}

// ✅ Filter method
const evens = numbers.filter((n) => n % 2 === 0);

// When to use loops vs array methods:
// Use traditional loops when:
// - Need to break early (performance optimization)
// - Complex iteration logic with multiple exit points
// - Working with very large datasets (avoid creating intermediate arrays)

// Use array methods when:
// - Transforming data (map, filter, reduce)
// - Code readability is priority
// - Chaining operations
// - Working with standard data transformations
```

**Performance Considerations:**

```javascript
// For performance-critical code with early exit
function findUser(users, id) {
  // ✅ Loop with break - stops immediately
  for (const user of users) {
    if (user.id === id) return user;
  }

  // ❌ find() must check every element internally (but still better than custom loop for readability)
  return users.find((u) => u.id === id);
}

// For most cases, readability > micro-optimizations
// Modern JS engines optimize array methods heavily
```

---

## 3. Arrays: The Workhorse Data Structure

Understanding array internals and methods is critical for modern JavaScript development. Arrays are the most commonly manipulated data structure in front-end applications.

### Array Internals: Dense vs Sparse Arrays

JavaScript arrays are actually objects with special handling for numeric indexes:

```javascript
// Dense array - continuous elements
const dense = [1, 2, 3, 4, 5];
console.log(dense.length); // 5
console.log(Object.keys(dense)); // ['0', '1', '2', '3', '4']

// Sparse array - holes in indexes
const sparse = [1, , , 4, 5]; // Note the empty slots
console.log(sparse.length); // 5
console.log(Object.keys(sparse)); // ['0', '3', '4'] - missing '1' and '2'
console.log(sparse[1]); // undefined
console.log(1 in sparse); // false (no property at index 1)

// Creating sparse arrays
const arr1 = new Array(3); // [empty × 3] - sparse!
const arr2 = Array(3); // [empty × 3] - sparse!
const arr3 = [undefined, undefined, undefined]; // Dense array with undefined values

// Array methods handle sparse arrays differently
sparse.forEach((v) => console.log(v)); // Skips holes: 1, 4, 5
sparse.map((v) => v * 2); // [2, empty × 2, 8, 10] - preserves holes
[...sparse]; // [1, undefined, undefined, 4, 5] - fills holes with undefined

// Performance: Dense arrays are faster

// ⚠️ Operations that cause "holes" hurt performance
const fast = [1, 2, 3, 4, 5];
delete fast[2]; // Creates hole: [1, 2, empty, 4, 5] - now slower!

// ✅ Keep arrays dense for performance
const better = [1, 2, 3, 4, 5];
better.splice(2, 1); // [1, 2, 4, 5] - stays dense
```

### Array Basics

```javascript
// Creation
const numbers = [1, 2, 3, 4, 5];
const mixed = [1, "hello", true, null, { key: "value" }];
const empty = [];
const fromString = Array.from("hello"); // ['h', 'e', 'l', 'l', 'o']
const filled = Array(5).fill(0); // [0, 0, 0, 0, 0]
const range = Array.from({ length: 5 }, (_, i) => i); // [0, 1, 2, 3, 4]

// Access
console.log(numbers[0]); // 1 (first element)
console.log(numbers[numbers.length - 1]); // 5 (last element)
console.log(numbers.at(-1)); // 5 (modern way to get last)
console.log(numbers.at(-2)); // 4 (second to last)

// Modification (mutating methods)
numbers[1] = 10; // [1, 10, 3, 4, 5]
numbers.push(6); // Add to end: [1, 10, 3, 4, 5, 6]
numbers.pop(); // Remove from end: [1, 10, 3, 4, 5]
numbers.unshift(0); // Add to start: [0, 1, 10, 3, 4, 5]
numbers.shift(); // Remove from start: [1, 10, 3, 4, 5]
```

### Array Methods: Mutation vs Immutability

Understanding which methods mutate the original array is critical:

```javascript
// ⚠️ MUTATING METHODS (modify original)
const arr = [1, 2, 3];

arr.push(4); // Returns new length: 4, arr = [1, 2, 3, 4]
arr.pop(); // Returns removed element: 4, arr = [1, 2, 3]
arr.unshift(0); // Returns new length: 4, arr = [0, 1, 2, 3]
arr.shift(); // Returns removed element: 0, arr = [1, 2, 3]

arr.splice(1, 1, 10); // Returns removed elements: [2], arr = [1, 10, 3]
arr.reverse(); // Returns reversed array: [3, 10, 1], arr = [3, 10, 1]
arr.sort(); // Returns sorted array, arr = [1, 10, 3] (⚠️ string sort!)
arr.sort((a, b) => a - b); // Numeric sort: [1, 3, 10]

arr.fill(0); // arr = [0, 0, 0]
arr.copyWithin(1, 0); // Copies elements within same array

// ✅ NON-MUTATING METHODS (return new array/value)
const arr2 = [1, 2, 3, 4, 5];

arr2.slice(1, 3); // [2, 3], arr2 unchanged
arr2.concat([6, 7]); // [1, 2, 3, 4, 5, 6, 7], arr2 unchanged
[...arr2, 6, 7]; // Same as concat, arr2 unchanged

arr2.map((n) => n * 2); // [2, 4, 6, 8, 10], arr2 unchanged
arr2.filter((n) => n > 2); // [3, 4, 5], arr2 unchanged
arr2.reduce((sum, n) => sum + n, 0); // 15, arr2 unchanged

arr2.find((n) => n > 2); // 3, arr2 unchanged
arr2.findIndex((n) => n > 2); // 2, arr2 unchanged
arr2.indexOf(3); // 2, arr2 unchanged
arr2.includes(3); // true, arr2 unchanged

// Best practice: Prefer immutable operations
// ❌ Mutation can cause bugs in React/Redux
const state = [1, 2, 3];
state.push(4); // Mutates - React won't detect change!

// ✅ Create new array
const newState = [...state, 4]; // New reference - React detects change
```

### Essential Array Methods (Functional Iteration)

Modern JavaScript development is heavily functional. Mastering these methods is non-negotiable.

#### **map()** - Transform each element

```javascript
const numbers = [1, 2, 3, 4];
const doubled = numbers.map((num) => num * 2);
// [2, 4, 6, 8]

const users = [
  { name: "Alice", age: 25 },
  { name: "Bob", age: 30 },
];
const names = users.map((user) => user.name);
// ['Alice', 'Bob']

// With index and array parameters
const indexed = numbers.map((num, index, arr) => {
  return `${num} at index ${index} of ${arr.length}`;
});

// ⚠️ map() always returns new array with same length
// Don't use map if you're not using the returned array
numbers.map((n) => console.log(n)); // ❌ Use forEach instead
numbers.forEach((n) => console.log(n)); // ✅

// Common pattern: Extracting properties
const ids = users.map((u) => u.id);
const fullNames = users.map((u) => `${u.firstName} ${u.lastName}`);

// Chaining transformations
const result = numbers
  .map((n) => n * 2)
  .map((n) => n + 1)
  .map((n) => `Number: ${n}`);
```

#### **filter()** - Keep elements that pass test

```javascript
const numbers = [1, 2, 3, 4, 5, 6];
const evens = numbers.filter((num) => num % 2 === 0);
// [2, 4, 6]

const users = [
  { name: "Alice", age: 25, active: true },
  { name: "Bob", age: 30, active: false },
  { name: "Charlie", age: 35, active: true },
];

const activeUsers = users.filter((user) => user.active);
// [{ name: 'Alice', ... }, { name: 'Charlie', ... }]

// Complex filtering
const filtered = users.filter((u) => {
  return u.active && u.age > 25 && u.name.startsWith("C");
});

// Remove falsy values
const mixed = [0, 1, false, 2, "", 3, null, undefined, 4];
const truthy = mixed.filter(Boolean);
// [1, 2, 3, 4]

// Remove duplicates (with filter)
const withDupes = [1, 2, 2, 3, 3, 3, 4];
const unique = withDupes.filter((val, index, arr) => arr.indexOf(val) === index);
// [1, 2, 3, 4]

// Better way to remove duplicates
const unique2 = [...new Set(withDupes)];
```

#### **reduce()** - Reduce array to single value (most powerful)

```javascript
// Sum numbers
const numbers = [1, 2, 3, 4];
const sum = numbers.reduce((accumulator, current) => accumulator + current, 0);
// 10

// Understanding reduce parameters
const result = arr.reduce((acc, current, index, array) => {
  // acc: accumulated value (starts as initialValue)
  // current: current element
  // index: current index
  // array: original array
  return newAccValue;
}, initialValue);

// Build object from array
const users = ["Alice", "Bob", "Charlie"];
const userObj = users.reduce((acc, name, index) => {
  acc[index] = name;
  return acc;
}, {});
// { 0: 'Alice', 1: 'Bob', 2: 'Charlie' }

// Group by property (VERY common pattern)
const items = [
  { type: "fruit", name: "apple" },
  { type: "veggie", name: "carrot" },
  { type: "fruit", name: "banana" },
];
const grouped = items.reduce((acc, item) => {
  if (!acc[item.type]) acc[item.type] = [];
  acc[item.type].push(item.name);
  return acc;
}, {});
// { fruit: ['apple', 'banana'], veggie: ['carrot'] }

// Count occurrences
const fruits = ["apple", "banana", "apple", "orange", "banana", "apple"];
const counts = fruits.reduce((acc, fruit) => {
  acc[fruit] = (acc[fruit] || 0) + 1;
  return acc;
}, {});
// { apple: 3, banana: 2, orange: 1 }

// Flatten array
const nested = [
  [1, 2],
  [3, 4],
  [5, 6],
];
const flat = nested.reduce((acc, arr) => acc.concat(arr), []);
// [1, 2, 3, 4, 5, 6]
// Better: nested.flat() or nested.flatMap()

// Build complex data structures
const transactions = [
  { id: 1, amount: 100, type: "credit" },
  { id: 2, amount: 50, type: "debit" },
  { id: 3, amount: 75, type: "credit" },
];

const summary = transactions.reduce(
  (acc, t) => {
    acc.total += t.type === "credit" ? t.amount : -t.amount;
    acc.count++;
    acc[t.type] = (acc[t.type] || 0) + 1;
    return acc;
  },
  { total: 0, count: 0 }
);
// { total: 125, count: 3, credit: 2, debit: 1 }

// Async reduce (sequential promises)
const urls = ["url1", "url2", "url3"];
const results = await urls.reduce(async (accPromise, url) => {
  const acc = await accPromise;
  const data = await fetch(url).then((r) => r.json());
  acc.push(data);
  return acc;
}, Promise.resolve([]));
```

#### **find()** / **findIndex()** - First element/index that matches

```javascript
const users = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" },
  { id: 3, name: "Charlie" },
];

// find() - returns element or undefined
const user = users.find((u) => u.id === 2);
// { id: 2, name: 'Bob' }

const notFound = users.find((u) => u.id === 999);
// undefined

// findIndex() - returns index or -1
const index = users.findIndex((u) => u.id === 2);
// 1

const notFoundIndex = users.findIndex((u) => u.id === 999);
// -1

// findLast() and findLastIndex() (ES2023)
const lastActive = users.findLast((u) => u.active);
const lastActiveIndex = users.findLastIndex((u) => u.active);

// Common pattern: find then update immutably
const userId = 2;
const updated = users.map((u) => (u.id === userId ? { ...u, name: "Robert" } : u));
```

#### **some()** / **every()** - Test conditions

```javascript
const numbers = [1, 2, 3, 4, 5];

// some() - true if ANY element passes test
const hasEven = numbers.some((n) => n % 2 === 0);
// true (2 and 4 are even)

const hasLarge = numbers.some((n) => n > 10);
// false

// every() - true if ALL elements pass test
const allPositive = numbers.every((n) => n > 0);
// true

const allEven = numbers.every((n) => n % 2 === 0);
// false

// Real-world examples
const users = [
  { name: "Alice", age: 25, verified: true },
  { name: "Bob", age: 17, verified: false },
];

const hasMinor = users.some((u) => u.age < 18);
// true - need to show age warning

const allVerified = users.every((u) => u.verified);
// false - can't proceed with unverified users

// Validation
const formData = {
  name: "Alice",
  email: "alice@example.com",
  age: 25,
};
const requiredFields = ["name", "email", "age"];
const isValid = requiredFields.every((field) => formData[field]);
// true - all required fields present
```

#### **flatMap()** - Map then flatten

```javascript
// Regular map with nested arrays
const sentences = ["Hello world", "How are you"];
const words = sentences.map((s) => s.split(" "));
// [['Hello', 'world'], ['How', 'are', 'you']]

// flatMap - map + flat(1)
const allWords = sentences.flatMap((s) => s.split(" "));
// ['Hello', 'world', 'How', 'are', 'you']

// Equivalent to:
const allWords2 = sentences.map((s) => s.split(" ")).flat();

// Use case: Expanding items
const users = [
  { name: "Alice", skills: ["JS", "React"] },
  { name: "Bob", skills: ["Python", "Django"] },
];

const allSkills = users.flatMap((u) => u.skills);
// ['JS', 'React', 'Python', 'Django']

// Filtering and mapping in one pass
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.flatMap((n) => (n % 2 === 0 ? [n * 2] : []));
// [4, 8] - only even numbers doubled
```

#### **Other Critical Methods**

```javascript
// includes() - check if value exists
const numbers = [1, 2, 3];
numbers.includes(2); // true
numbers.includes(5); // false
numbers.includes(2, 2); // false (start searching from index 2)

// indexOf() / lastIndexOf() - get index of value
numbers.indexOf(2); // 1
numbers.indexOf(5); // -1 (not found)
numbers.lastIndexOf(2); // 1 (searches from end)

// slice() - extract portion (non-mutating)
const slice = numbers.slice(1, 3); // [2, 3] (from index 1 to 3, not including 3)
const last3 = numbers.slice(-3); // Last 3 elements
const copy = numbers.slice(); // Shallow copy

// splice() - remove/add elements (MUTATING!)
const arr = [1, 2, 3, 4, 5];
const removed = arr.splice(2, 2, "a", "b");
// removed: [3, 4], arr: [1, 2, 'a', 'b', 5]

// concat() - merge arrays (non-mutating)
const arr1 = [1, 2];
const arr2 = [3, 4];
const merged = arr1.concat(arr2); // [1, 2, 3, 4]
const merged2 = arr1.concat(arr2, [5, 6]); // Can take multiple arrays

// join() - array to string
const words = ["Hello", "world"];
words.join(" "); // 'Hello world'
words.join("-"); // 'Hello-world'
words.join(""); // 'Helloworld'

// sort() - sort in place (MUTATING!)
const nums = [3, 1, 4, 1, 5, 9, 2, 6];
nums.sort(); // [1, 1, 2, 3, 4, 5, 6, 9] (⚠️ string sort by default!)

const mixed = [1, 10, 2, 20];
mixed.sort(); // [1, 10, 2, 20] - wrong! String comparison
mixed.sort((a, b) => a - b); // [1, 2, 10, 20] - correct numeric sort
mixed.sort((a, b) => b - a); // [20, 10, 2, 1] - descending

// Complex sorting
const users = [
  { name: "Charlie", age: 35 },
  { name: "Alice", age: 25 },
  { name: "Bob", age: 30 },
];
users.sort((a, b) => a.name.localeCompare(b.name)); // Sort by name
users.sort((a, b) => a.age - b.age); // Sort by age

// reverse() - reverse in place (MUTATING!)
const arr = [1, 2, 3];
arr.reverse(); // [3, 2, 1], arr is now reversed

// Non-mutating reverse
const reversed = [...arr].reverse(); // arr unchanged

// flat() - flatten nested arrays
const nested = [1, [2, 3], [4, [5, 6]]];
nested.flat(); // [1, 2, 3, 4, [5, 6]] (default depth: 1)
nested.flat(2); // [1, 2, 3, 4, 5, 6] (depth: 2)
nested.flat(Infinity); // Flatten all levels

// fill() - fill with static value (MUTATING!)
const arr = new Array(5).fill(0); // [0, 0, 0, 0, 0]
arr.fill(1, 2, 4); // [0, 0, 1, 1, 0] (fill from index 2 to 4)

// ⚠️ Watch out with objects!
const arr2 = new Array(3).fill({}); // [{}, {}, {}]
arr2[0].x = 1; // [{x: 1}, {x: 1}, {x: 1}] - same object reference!

// ✅ Use Array.from for unique objects
const arr3 = Array.from({ length: 3 }, () => ({})); // 3 unique objects
```

---

## 4. Objects: Key-Value Data Mastery

Objects are JavaScript's fundamental data structure. Understanding property descriptors, enumeration, and efficient patterns is essential.

### Object Basics

```javascript
// Creation
const user = {
  name: "Alice",
  age: 25,
  email: "alice@example.com",
  "favorite-color": "blue", // Keys with special chars must be strings
  123: "numeric key", // Numeric keys are coerced to strings
};

// Access
console.log(user.name); // Dot notation (preferred when possible)
console.log(user["age"]); // Bracket notation
console.log(user["favorite-color"]); // Required for non-standard keys
console.log(user[123]); // "numeric key"
console.log(user["123"]); // Same as above

// Modification
user.age = 26; // Update property
user.city = "New York"; // Add new property
delete user.email; // Remove property (⚠️ slow operation!)

// Better than delete: set to undefined/null
user.email = undefined; // Keeps property shape (better for V8 optimization)

// Computed property names (ES6+)
const key = "dynamicKey";
const obj = {
  [key]: "value",
  [`${key}2`]: "value2",
  [Symbol("id")]: "symbol key",
};
// { dynamicKey: 'value', dynamicKey2: 'value2', [Symbol(id)]: 'symbol key' }
```

### Property Descriptors

Objects aren't just key-value pairs—each property has metadata:

```javascript
const user = {
  name: "Alice",
  age: 25,
};

// Get property descriptor
const descriptor = Object.getOwnPropertyDescriptor(user, "name");
console.log(descriptor);
// {
//   value: 'Alice',
//   writable: true,       // Can be changed
//   enumerable: true,     // Shows up in for...in, Object.keys()
//   configurable: true    // Can be deleted or reconfigured
// }

// Define property with custom descriptor
Object.defineProperty(user, "id", {
  value: 123,
  writable: false, // Cannot be changed
  enumerable: false, // Won't show in Object.keys()
  configurable: false, // Cannot be deleted or reconfigured
});

user.id = 456; // Silently fails (throws in strict mode)
console.log(user.id); // 123
Object.keys(user); // ['name', 'age'] - id is not enumerable
delete user.id; // Silently fails (throws in strict mode)

// Define multiple properties at once
Object.defineProperties(user, {
  firstName: {
    value: "Alice",
    writable: true,
    enumerable: true,
  },
  lastName: {
    value: "Smith",
    writable: true,
    enumerable: true,
  },
});

// Getters and setters
const person = {
  firstName: "Alice",
  lastName: "Smith",
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  },
  set fullName(value) {
    [this.firstName, this.lastName] = value.split(" ");
  },
};

console.log(person.fullName); // "Alice Smith" (getter)
person.fullName = "Bob Jones"; // Calls setter
console.log(person.firstName); // "Bob"
```

### Enumeration Order (Important!)

JavaScript object property enumeration follows a specific order (since ES2015):

```javascript
const obj = {
  2: "numeric 2",
  b: "string b",
  1: "numeric 1",
  a: "string a",
  [Symbol("sym")]: "symbol",
};

// Order: numeric keys (ascending) → string keys (insertion) → symbols (insertion)
Object.keys(obj); // ['1', '2', 'a', 'b']
Object.getOwnPropertyNames(obj); // ['1', '2', 'a', 'b']
Object.getOwnPropertySymbols(obj); // [Symbol(sym)]

// for...in follows same order
for (const key in obj) {
  console.log(key); // 1, 2, a, b
}
```

### Object Methods

```javascript
const user = { name: "Alice", age: 25, city: "NYC" };

// Object.keys() - array of enumerable keys
Object.keys(user); // ['name', 'age', 'city']

// Object.values() - array of enumerable values
Object.values(user); // ['Alice', 25, 'NYC']

// Object.entries() - array of [key, value] pairs
Object.entries(user);
// [['name', 'Alice'], ['age', 25], ['city', 'NYC']]

// Iterate over object (modern way)
for (const [key, value] of Object.entries(user)) {
  console.log(`${key}: ${value}`);
}

// Object.fromEntries() - convert entries to object (inverse of entries)
const entries = [
  ["name", "Alice"],
  ["age", 25],
];
Object.fromEntries(entries); // { name: 'Alice', age: 25 }

// Common pattern: Transform object values
const prices = { apple: 1.2, banana: 0.5, orange: 0.8 };
const rounded = Object.fromEntries(Object.entries(prices).map(([key, value]) => [key, Math.round(value)]));
// { apple: 1, banana: 1, orange: 1 }

// Object.assign() - merge objects (shallow)
const defaults = { theme: "light", fontSize: 14 };
const userPrefs = { fontSize: 16 };
const settings = Object.assign({}, defaults, userPrefs);
// { theme: 'light', fontSize: 16 }

// ⚠️ First argument is mutated!
const target = { a: 1 };
Object.assign(target, { b: 2 }); // target is now { a: 1, b: 2 }

// ✅ Use empty object as target to avoid mutation
const result = Object.assign({}, obj1, obj2);

// Object.create() - create object with specific prototype
const parent = { greet: () => console.log("Hello") };
const child = Object.create(parent);
child.name = "Alice";
child.greet(); // "Hello" (inherited from parent)

// Object.freeze() - prevent all modifications
const frozen = Object.freeze({ name: "Alice" });
frozen.name = "Bob"; // Silently fails (throws in strict mode)
frozen.age = 25; // Silently fails
delete frozen.name; // Silently fails
// ⚠️ Shallow freeze only!
const obj = Object.freeze({ nested: { value: 1 } });
obj.nested.value = 2; // Works! nested object not frozen

// Object.seal() - prevent adding/removing properties
const sealed = Object.seal({ name: "Alice" });
sealed.name = "Bob"; // ✅ Works
sealed.age = 25; // ❌ Fails
delete sealed.name; // ❌ Fails

// Object.preventExtensions() - prevent adding properties only
const obj = Object.preventExtensions({ name: "Alice" });
obj.name = "Bob"; // ✅ Works
obj.age = 25; // ❌ Fails
delete obj.name; // ✅ Works

// Check object state
Object.isFrozen(frozen); // true
Object.isSealed(sealed); // true
Object.isExtensible(obj); // false

// Object.hasOwn() - modern way to check own property (ES2022)
const obj = { name: "Alice" };
Object.hasOwn(obj, "name"); // true
Object.hasOwn(obj, "toString"); // false (inherited)

// Old way (still works but verbose)
obj.hasOwnProperty("name"); // true

// Object.is() - strict equality check
Object.is(NaN, NaN); // true (unlike NaN === NaN which is false)
Object.is(+0, -0); // false (unlike +0 === -0 which is true)
Object.is(5, 5); // true
```

### Copying and Merging Patterns

Understanding shallow vs. deep copies is critical for avoiding bugs:

#### Shallow Copy

```javascript
// Spread operator (most common)
const original = { name: "Alice", age: 25 };
const copy = { ...original };
copy.name = "Bob"; // original unchanged

// Object.assign()
const copy2 = Object.assign({}, original);

// ⚠️ PROBLEM: Shallow copy only copies first level
const user = {
  name: "Alice",
  address: { city: "NYC", zip: "10001" },
};

const shallow = { ...user };
shallow.name = "Bob"; // ✅ user.name still "Alice"
shallow.address.city = "LA"; // ❌ user.address.city now "LA" too!

console.log(user.address.city); // "LA" - MUTATED!

// Why? address is a reference
console.log(user.address === shallow.address); // true - same object!
```

#### Deep Copy

```javascript
// Method 1: JSON parse/stringify (simple but limited)
const user = {
  name: "Alice",
  address: { city: "NYC", zip: "10001" },
  hobbies: ["reading", "gaming"],
};

const deepCopy = JSON.parse(JSON.stringify(user));
deepCopy.address.city = "LA"; // ✅ Original unchanged
console.log(user.address.city); // "NYC"

// ⚠️ JSON limitations:
// - Loses functions
// - Loses undefined values
// - Loses Symbol keys
// - Converts Date to string
// - Fails on circular references

const obj = {
  fn: () => {},
  undef: undefined,
  date: new Date(),
  [Symbol("id")]: 123,
};
const copy = JSON.parse(JSON.stringify(obj));
console.log(copy);
// { date: "2024-..." } - fn, undef, and symbol are gone!

// Method 2: structuredClone() (Modern - Node 17+, Modern browsers)
const deepCopy2 = structuredClone(user);
deepCopy2.address.city = "SF"; // ✅ Original unchanged

// ✅ Handles:
// - Nested objects and arrays
// - Date, RegExp, Map, Set
// - ArrayBuffer, TypedArrays
// - Circular references

// ❌ Does NOT handle:
// - Functions
// - DOM nodes
// - Symbols

// Method 3: Manual recursive deep clone
function deepClone(obj, seen = new WeakMap()) {
  // Handle primitives and null
  if (obj === null || typeof obj !== "object") return obj;

  // Handle circular references
  if (seen.has(obj)) return seen.get(obj);

  // Handle Date
  if (obj instanceof Date) return new Date(obj);

  // Handle Array
  if (Array.isArray(obj)) {
    const arrCopy = [];
    seen.set(obj, arrCopy);
    obj.forEach((item, index) => {
      arrCopy[index] = deepClone(item, seen);
    });
    return arrCopy;
  }

  // Handle Object
  const objCopy = {};
  seen.set(obj, objCopy);
  Object.keys(obj).forEach((key) => {
    objCopy[key] = deepClone(obj[key], seen);
  });

  return objCopy;
}

// Method 4: Libraries (Lodash)
import _ from "lodash";
const deepCopy3 = _.cloneDeep(user);
```

#### Merging Objects

```javascript
// Shallow merge with spread
const defaults = { theme: "light", fontSize: 14, features: { darkMode: false } };
const userPrefs = { fontSize: 16, features: { notifications: true } };

const merged = { ...defaults, ...userPrefs };
console.log(merged);
// {
//   theme: 'light',
//   fontSize: 16,
//   features: { notifications: true }  // ⚠️ Lost darkMode!
// }

// Deep merge (manual)
function deepMerge(target, source) {
  const result = { ...target };

  for (const key in source) {
    if (source[key] instanceof Object && key in target) {
      result[key] = deepMerge(target[key], source[key]);
    } else {
      result[key] = source[key];
    }
  }

  return result;
}

const deepMerged = deepMerge(defaults, userPrefs);
console.log(deepMerged);
// {
//   theme: 'light',
//   fontSize: 16,
//   features: { darkMode: false, notifications: true }  // ✅ Both preserved
// }

// Lodash merge
import _ from "lodash";
const merged2 = _.merge({}, defaults, userPrefs);

// Conditional merging
const config = {
  ...baseConfig,
  ...(isDev && { debug: true }), // Only add if isDev
  ...(apiKey && { apiKey }), // Only add if apiKey exists
};
```

---

## 5. Destructuring, Spread & Rest Operators

These ES6+ features are essential for modern JavaScript development. Understanding their nuances prevents bugs and improves code readability.

### Array Spreading (Spread Operator)

The spread operator (`...`) expands iterables into individual elements.

```javascript
// Copy array (shallow)
const original = [1, 2, 3];
const copy = [...original]; // [1, 2, 3] (new array reference)
copy.push(4); // original unchanged

// Merge arrays
const arr1 = [1, 2];
const arr2 = [3, 4];
const merged = [...arr1, ...arr2]; // [1, 2, 3, 4]

// Add elements
const withNew = [0, ...original, 4]; // [0, 1, 2, 3, 4]

// Clone with modifications
const numbers = [1, 2, 3];
const doubled = [...numbers].map((n) => n * 2); // Don't mutate original

// Convert iterable to array
const divs = [...document.querySelectorAll("div")]; // NodeList → Array
const chars = [..."hello"]; // ['h', 'e', 'l', 'l', 'o']
const set = new Set([1, 2, 3]);
const arr = [...set]; // [1, 2, 3]

// Function arguments (spreading)
const numbers = [1, 5, 3, 9, 2];
Math.max(...numbers); // 9 (instead of Math.max(1, 5, 3, 9, 2))
console.log(...numbers); // 1 5 3 9 2

// Combining with destructuring
const [first, ...rest] = [1, 2, 3, 4];
// first = 1, rest = [2, 3, 4]

// Removing duplicates
const withDupes = [1, 2, 2, 3, 3, 3];
const unique = [...new Set(withDupes)]; // [1, 2, 3]
```

### Object Spreading

```javascript
// Copy object (shallow)
const user = { name: "Alice", age: 25 };
const userCopy = { ...user };
userCopy.name = "Bob"; // user.name unchanged

// Merge objects (right overwrites left)
const defaults = { theme: "light", fontSize: 14 };
const userPrefs = { fontSize: 16, color: "blue" };
const settings = { ...defaults, ...userPrefs };
// { theme: 'light', fontSize: 16, color: 'blue' }

// Add/override properties
const updated = { ...user, age: 26, city: "NYC" };
// { name: 'Alice', age: 26, city: 'NYC' }

// Remove property (by omission)
const { age, ...withoutAge } = user;
// withoutAge = { name: 'Alice' }

// Conditional properties
const config = {
  ...baseConfig,
  ...(isDev && { debug: true }), // Only add if isDev is truthy
  ...(apiKey && { apiKey }), // Only add if apiKey exists
};

// Common pattern: Update nested property immutably (React/Redux)
const state = {
  user: {
    name: "Alice",
    address: { city: "NYC", zip: "10001" },
  },
};

// ❌ Doesn't work - shallow spread
const newState = {
  ...state,
  user: { ...state.user, address: { city: "LA" } }, // ❌ Lost zip!
};

// ✅ Must spread each level
const newState = {
  ...state,
  user: {
    ...state.user,
    address: {
      ...state.user.address,
      city: "LA", // ✅ zip preserved
    },
  },
};
```

### Rest Parameters (Gathering)

Rest syntax (`...`) collects remaining elements into an array. It looks like spread but does the opposite.

```javascript
// Function rest parameters - must be last parameter
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3, 4); // 10
sum(1); // 1
sum(); // 0

// With other parameters
function greet(greeting, ...names) {
  return `${greeting}, ${names.join(" and ")}!`;
}
greet("Hello", "Alice", "Bob", "Charlie");
// "Hello, Alice and Bob and Charlie!"

// Array destructuring with rest
const [first, second, ...others] = [1, 2, 3, 4, 5];
// first = 1, second = 2, others = [3, 4, 5]

const [head, ...tail] = [1];
// head = 1, tail = []

// Skip elements
const [, , ...fromThird] = [1, 2, 3, 4];
// fromThird = [3, 4]

// Object destructuring with rest
const user = { name: "Alice", age: 25, city: "NYC", country: "USA" };
const { name, ...rest } = user;
// name = 'Alice', rest = { age: 25, city: 'NYC', country: 'USA' }

// Common pattern: Extract and exclude
function updateUser({ id, ...updates }) {
  // id is available separately, updates has everything else
  return database.update(id, updates);
}

// Forwarding props (React pattern)
function Button({ variant, size, ...otherProps }) {
  return <button className={`btn-${variant} btn-${size}`} {...otherProps} />;
}
```

### Array Destructuring

```javascript
// Basic
const colors = ["red", "green", "blue"];
const [first, second] = colors;
// first = 'red', second = 'green'

// Skip elements
const [, , third] = colors; // third = 'blue'

// Rest elements (must be last)
const numbers = [1, 2, 3, 4, 5];
const [head, ...tail] = numbers;
// head = 1, tail = [2, 3, 4, 5]

// Default values
const [a, b, c = "default"] = [1, 2];
// a = 1, b = 2, c = 'default'

const [x = 10, y = 20] = [undefined, null];
// x = 10 (undefined triggers default), y = null (doesn't trigger default)

// Swapping variables (elegant!)
let x = 1,
  y = 2;
[x, y] = [y, x]; // x = 2, y = 1

// Parsing function return values
function getCoords() {
  return [10, 20];
}
const [x, y] = getCoords();

// Nested destructuring
const nested = [1, [2, 3], 4];
const [first, [second, third], fourth] = nested;
// first = 1, second = 2, third = 3, fourth = 4

// Ignoring return values
const [, second] = getCoords(); // Only need second value

// With iterables
const [first, ...rest] = "hello"; // first = 'h', rest = ['e', 'l', 'l', 'o']
const [a, b] = new Set([1, 2, 3]); // a = 1, b = 2
```

### Object Destructuring

```javascript
// Basic
const user = { name: "Alice", age: 25, city: "NYC" };
const { name, age } = user;
// name = 'Alice', age = 25

// Rename variables (aliasing)
const { name: userName, age: userAge } = user;
// userName = 'Alice', userAge = 25

// Default values
const { name, country = "USA" } = user;
// country = 'USA' (not in original object)

const { name, missing = "N/A" } = user;
// missing = 'N/A'

// Both renaming and defaults
const { name: fullName = "Anonymous" } = {};
// fullName = 'Anonymous'

// Rest properties (must be last)
const { name, ...rest } = user;
// name = 'Alice', rest = { age: 25, city: 'NYC' }

// Nested destructuring
const data = {
  user: {
    name: "Alice",
    address: { city: "NYC", zip: "10001" },
  },
};

const {
  user: {
    name,
    address: { city },
  },
} = data;
// name = 'Alice', city = 'NYC'

// ⚠️ Be careful: intermediate levels aren't assigned
const {
  user: {
    address: { city },
  },
} = data;
// city = 'NYC', but user and address are NOT assigned

// If you need intermediate levels too:
const { user } = data; // Get user
const { address } = user; // Get address
const { city } = address; // Get city

// Function parameters (VERY common pattern)
function greet({ name, age = 0 }) {
  console.log(`Hello ${name}, age ${age}`);
}
greet(user); // Hello Alice, age 25
greet({ name: "Bob" }); // Hello Bob, age 0

// With defaults for entire parameter
function config({ host = "localhost", port = 3000 } = {}) {
  console.log(`${host}:${port}`);
}
config({ port: 8080 }); // localhost:8080
config(); // localhost:3000 (empty object uses defaults)

// Extract multiple properties in function
function renderUser({ name, email, avatar = "default.png" }) {
  return `<div>${name} - ${email} - <img src="${avatar}"></div>`;
}

// React component pattern
function UserCard({ user: { name, email, age }, theme = "light", ...otherProps }) {
  // Destructure nested user AND get other props
}

// Computed property names
const key = "dynamicKey";
const obj = { [key]: "value" };
const { [key]: extractedValue } = obj;
// extractedValue = 'value'
```

### Avoiding Reference Leaks with Destructuring

```javascript
// Problem: Accidental mutation through references
const state = {
  user: { name: "Alice", settings: { theme: "dark" } },
};

const { user } = state;
user.name = "Bob"; // ❌ Mutates state.user!

// Solution 1: Destructure values, not objects
const {
  user: { name },
} = state;
// Now 'name' is just a string, can't mutate state

// Solution 2: Spread when extracting
const user = { ...state.user };
user.name = "Bob"; // ✅ state.user unchanged

// Solution 3: Deep clone if needed
const user = structuredClone(state.user);
```

### Combining Destructuring with Defaults and Renaming

```javascript
// Real-world example: API response handling
function processResponse({
  data: results = [],
  status: responseStatus = 200,
  error: errorMessage = null,
  meta: { page = 1, total = 0 } = {},
} = {}) {
  console.log({ results, responseStatus, errorMessage, page, total });
}

// Handles all these cases:
processResponse(); // All defaults
processResponse({ data: [1, 2, 3] }); // Some values
processResponse({ meta: { page: 2 } }); // Nested values

// React component with complex props
function DataTable({
  data = [],
  columns = [],
  options: { sortable = true, filterable = false, pageSize = 10, theme = "light" } = {},
  onRowClick = () => {},
  className = "",
  ...otherProps
}) {
  // Clean access to all configuration
}
```

---

## 6. Modern Patterns & Operators

ES2020+ introduced powerful operators that eliminate common bugs and improve code readability.

### Optional Chaining (?.)

Safely access nested properties without explicit null/undefined checks.

```javascript
// Problem: Traditional null checks
const user = getUser();
let city;
if (user && user.profile && user.profile.address) {
  city = user.profile.address.city;
}

// Solution: Optional chaining
const city = user?.profile?.address?.city; // undefined if any level is null/undefined

// Works with arrays
const firstFriend = user?.friends?.[0]?.name;

// Works with function calls
const result = obj.method?.(); // Only calls if method exists

// Real-world example: API response
const apiResponse = await fetch("/api/user");
const userName = apiResponse?.data?.user?.name ?? "Guest";

// ⚠️ Only checks for null/undefined, not other falsy values
const obj = { value: 0 };
obj?.value; // 0 (not undefined!)
obj?.missing; // undefined
```

### Nullish Coalescing (??)

Provide default values only for `null` or `undefined` (not other falsy values like `0`, `""`, `false`).

```javascript
// Problem: OR operator treats all falsy values the same
const port = config.port || 3000;
// If port = 0, uses 3000 (probably not what you want!)

const count = data.count || 10;
// If count = 0, uses 10 (wrong!)

// Solution: Nullish coalescing
const port = config.port ?? 3000; // Only 3000 if port is null/undefined
const count = data.count ?? 10; // 0 is valid, only defaults for null/undefined

// More examples
const user = {
  name: "",
  age: 0,
  score: null,
};

user.name || "Anonymous"; // "Anonymous" (empty string is falsy)
user.name ?? "Anonymous"; // "" (empty string is not null/undefined)

user.age || 18; // 18 (0 is falsy)
user.age ?? 18; // 0 (0 is not null/undefined)

user.score || 100; // 100 (null is falsy)
user.score ?? 100; // 100 (null triggers default)

// Combined with optional chaining (VERY common pattern)
const theme = user?.settings?.theme ?? "light";
const pageSize = config?.table?.pageSize ?? 10;
```

### Logical Assignment Operators (ES2021)

Combine logical operators with assignment for concise updates.

```javascript
// Logical OR assignment (||=)
let x = 0;
x ||= 10; // x = 10 (0 is falsy)

let y = 5;
y ||= 10; // y = 5 (5 is truthy)

// Equivalent to:
x = x || 10;

// Use case: Lazy initialization
config.port ||= 3000; // Only set if falsy
cache[key] ||= expensiveComputation();

// Logical AND assignment (&&=)
let obj = { value: 5 };
obj.value &&= obj.value * 2; // obj.value = 10 (value exists)

let obj2 = {};
obj2.value &&= obj2.value * 2; // obj2.value = undefined (unchanged)

// Equivalent to:
obj.value = obj.value && obj.value * 2;

// Use case: Conditional transformation
user.email &&= user.email.toLowerCase(); // Only lowercase if email exists

// Nullish coalescing assignment (??=)
let port;
port ??= 3000; // port = 3000 (undefined triggers assignment)

port = 0;
port ??= 3000; // port = 0 (0 doesn't trigger assignment)

// Equivalent to:
port = port ?? 3000;

// Use case: Setting defaults only for null/undefined
options.timeout ??= 5000;
config.retries ??= 3;

// Real-world pattern: Caching
const cache = {};
function getData(key) {
  cache[key] ??= fetchFromAPI(key); // Only fetch if not cached
  return cache[key];
}
```

### Performance and Readability Trade-offs

```javascript
// Readability: Optional chaining is clearer
// ❌ Traditional null checks
if (user && user.profile && user.profile.address && user.profile.address.city) {
  console.log(user.profile.address.city);
}

// ✅ Optional chaining
if (user?.profile?.address?.city) {
  console.log(user?.profile?.address?.city);
}

// Performance: Optional chaining has negligible overhead in modern engines
// Hot path optimization: If you check the same path millions of times per second
const city = user?.profile?.address?.city; // Slightly slower
const city = user && user.profile && user.profile.address && user.profile.address.city; // Slightly faster

// But readability > micro-optimizations in 99.9% of cases

// Method chaining with optional chaining
const result = obj?.method1()?.method2()?.method3(); // Stops at first undefined/null

// Traditional try-catch
let result;
try {
  result = obj.method1().method2().method3();
} catch (e) {
  result = undefined;
}
```

---

## 7. Practical Patterns & Best Practices

### Immutable Updates (React/Redux Pattern)

Critical for state management in React, Redux, and other reactive frameworks.

```javascript
// ❌ Mutating (bad for React state)
const user = { name: "Alice", age: 25 };
user.age = 26; // Mutates original - React won't detect change!

// ✅ Immutable (create new object)
const updatedUser = { ...user, age: 26 }; // New reference - React detects change

// Array immutability
const todos = [
  { id: 1, text: "Learn JS", done: false },
  { id: 2, text: "Build app", done: false },
];

// ❌ Mutating array
todos.push({ id: 3, text: "Deploy", done: false }); // React won't detect!
todos[0].done = true; // React won't detect!

// ✅ Add item immutably
const withNew = [...todos, { id: 3, text: "Deploy", done: false }];

// ✅ Update item immutably
const updated = todos.map(
  (todo) =>
    todo.id === 1
      ? { ...todo, done: true } // Create new object for matching item
      : todo // Keep original object for others
);

// ✅ Remove item immutably
const filtered = todos.filter((todo) => todo.id !== 1);

// ✅ Update nested property immutably
const state = {
  user: {
    name: "Alice",
    address: { city: "NYC", zip: "10001" },
  },
};

const newState = {
  ...state,
  user: {
    ...state.user,
    address: {
      ...state.user.address,
      city: "LA", // Only city changes, zip preserved
    },
  },
};

// Helper function for nested updates (Lodash-style)
function updateNested(obj, path, value) {
  const keys = path.split(".");
  if (keys.length === 1) {
    return { ...obj, [keys[0]]: value };
  }

  const [first, ...rest] = keys;
  return {
    ...obj,
    [first]: updateNested(obj[first], rest.join("."), value),
  };
}

const updated = updateNested(state, "user.address.city", "LA");
```

### Method Chaining

Combine array methods for powerful, declarative transformations.

```javascript
// Combine array methods for powerful transformations
const users = [
  { name: "Alice", age: 25, active: true, role: "admin" },
  { name: "Bob", age: 30, active: false, role: "user" },
  { name: "Charlie", age: 35, active: true, role: "user" },
  { name: "Diana", age: 28, active: true, role: "admin" },
];

// Chain: filter → map → sort → join
const result = users
  .filter((user) => user.active) // Keep active users
  .map((user) => user.name) // Extract names
  .sort() // Sort alphabetically
  .join(", "); // Join into string
// 'Alice, Charlie, Diana'

// Complex transformation with reduce at the end
const stats = users
  .filter((u) => u.active)
  .reduce(
    (acc, u) => {
      acc.count++;
      acc.totalAge += u.age;
      acc.avgAge = acc.totalAge / acc.count;
      acc.roles[u.role] = (acc.roles[u.role] || 0) + 1;
      return acc;
    },
    { count: 0, totalAge: 0, avgAge: 0, roles: {} }
  );
// { count: 3, totalAge: 88, avgAge: 29.33, roles: { admin: 2, user: 1 } }

// Chaining with early exit consideration
// ⚠️ This processes ALL users even though we only need first 3
const first3Active = users.filter((u) => u.active).slice(0, 3);

// ✅ Better: Stop after finding 3
const first3ActiveOptimized = [];
for (const user of users) {
  if (user.active) {
    first3ActiveOptimized.push(user);
    if (first3ActiveOptimized.length === 3) break;
  }
}

// Real-world example: Processing API data
const apiData = [
  { id: 1, name: "Product A", price: 29.99, category: "electronics", inStock: true },
  { id: 2, name: "Product B", price: 49.99, category: "clothing", inStock: false },
  { id: 3, name: "Product C", price: 19.99, category: "electronics", inStock: true },
  { id: 4, name: "Product D", price: 99.99, category: "electronics", inStock: true },
];

const result = apiData
  .filter((p) => p.inStock && p.category === "electronics")
  .map((p) => ({
    ...p,
    price: p.price * 0.9, // 10% discount
    discounted: true,
  }))
  .sort((a, b) => a.price - b.price)
  .map((p) => ({
    id: p.id,
    name: p.name,
    finalPrice: p.price.toFixed(2),
  }));
```

### Safe Merging Patterns

```javascript
// Safe array merge (remove duplicates)
const arr1 = [1, 2, 3];
const arr2 = [2, 3, 4];
const merged = [...new Set([...arr1, ...arr2])]; // [1, 2, 3, 4]

// Merge arrays of objects by unique key
function mergeByKey(arr1, arr2, key) {
  const map = new Map();
  [...arr1, ...arr2].forEach((item) => map.set(item[key], item));
  return Array.from(map.values());
}

const users1 = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" },
];
const users2 = [
  { id: 2, name: "Robert" },
  { id: 3, name: "Charlie" },
];
mergeByKey(users1, users2, "id");
// [{ id: 1, name: 'Alice' }, { id: 2, name: 'Robert' }, { id: 3, name: 'Charlie' }]

// Deep merge with validation
function deepMerge(target, source) {
  if (!source || typeof source !== "object") return target;
  if (!target || typeof target !== "object") return source;

  const result = { ...target };

  for (const key in source) {
    if (source[key] instanceof Object && !Array.isArray(source[key])) {
      result[key] = deepMerge(target[key], source[key]);
    } else {
      result[key] = source[key];
    }
  }

  return result;
}
```

---

## 8. Common Pitfalls & How to Avoid Them

### Pitfall 1: Mutating Data Directly

```javascript
// ❌ Bad: Mutates original
const arr = [1, 2, 3];
arr.push(4); // arr is now [1, 2, 3, 4]
arr.sort(); // arr is mutated
delete arr[0]; // Creates hole - sparse array

const obj = { a: 1 };
obj.b = 2; // Mutates

// ✅ Good: Create new references
const newArr = [...arr, 4]; // Original unchanged
const sorted = [...arr].sort(); // Original unchanged
const { a, ...rest } = obj; // Remove property immutably

const newObj = { ...obj, b: 2 }; // Original unchanged
```

### Pitfall 2: Shallow Copy Trap

```javascript
// ❌ Bad: Shallow copy doesn't protect nested objects
const user = {
  name: "Alice",
  address: { city: "NYC" },
};
const copy = { ...user };
copy.address.city = "LA"; // ❌ Also changes user.address.city!

// ✅ Good: Deep copy when needed
const deepCopy = structuredClone(user); // Modern way
const deepCopy2 = JSON.parse(JSON.stringify(user)); // Old way (limitations)

// Or spread each level
const safeCopy = {
  ...user,
  address: { ...user.address },
};
```

### Pitfall 3: Using for...in on Arrays

```javascript
// ❌ Bad: for...in iterates over keys (and inherited properties!)
const arr = [10, 20, 30];
arr.customProp = "value";

for (const i in arr) {
  console.log(i); // "0", "1", "2", "customProp" (includes non-index!)
}

// ✅ Good: Use for...of for values
for (const value of arr) {
  console.log(value); // 10, 20, 30 (only array elements)
}

// ✅ Or use array methods
arr.forEach((value) => console.log(value));
```

### Pitfall 4: Array.sort() String Comparison

```javascript
// ❌ Bad: Default sort is lexicographic (string comparison)
const nums = [1, 10, 2, 20];
nums.sort(); // [1, 10, 2, 20] - Wrong!

// ✅ Good: Provide comparator for numbers
nums.sort((a, b) => a - b); // [1, 2, 10, 20] - Correct!
nums.sort((a, b) => b - a); // [20, 10, 2, 1] - Descending
```

### Pitfall 5: Falsy vs Nullish Confusion

```javascript
// ❌ Bad: OR operator fails for valid falsy values
const port = config.port || 3000; // If port = 0, uses 3000 (wrong!)
const count = data.count || 10; // If count = 0, uses 10 (wrong!)
const name = user.name || "Guest"; // If name = "", uses "Guest" (maybe wrong!)

// ✅ Good: Use nullish coalescing for defaults
const port = config.port ?? 3000; // 0 is valid
const count = data.count ?? 10; // 0 is valid
const name = user.name ?? "Guest"; // "" might still need ||
```

### Pitfall 6: Reference Sharing in Arrays

```javascript
// ❌ Bad: Array.fill() with objects creates shared references
const arr = new Array(3).fill({});
arr[0].x = 1; // [{x: 1}, {x: 1}, {x: 1}] - Same object!

// ✅ Good: Create unique objects
const arr = Array.from({ length: 3 }, () => ({}));
arr[0].x = 1; // [{x: 1}, {}, {}] - Different objects
```

---

## Key Takeaways

1. **Control Flow**: Use early returns, guard clauses, and avoid deep nesting for maintainability
2. **Truthiness**: Understand falsy values and prefer explicit checks when clarity matters
3. **Loops**: Use `for...of` for values, `for...in` for object keys (with `hasOwnProperty`), array methods for transformations
4. **Arrays**: Master `map`, `filter`, `reduce` - they're essential for modern JavaScript
5. **Immutability**: Always create new arrays/objects instead of mutating (critical for React/Redux)
6. **Destructuring**: Extract values elegantly in assignments, function params, and imports
7. **Spread/Rest**: Use spread for copying/merging, rest for gathering remaining elements
8. **Optional Chaining**: Prevent errors when accessing nested properties (`?.`)
9. **Nullish Coalescing**: Provide defaults only for `null/undefined`, not other falsy values (`??`)
10. **Method Chaining**: Combine operations for readable, declarative transformations
11. **Shallow vs Deep**: Understand copy depth and use appropriate strategy
12. **Performance**: Prefer readability over micro-optimizations unless profiling shows hotspots

---
