# Day 3 — Values, Types, Variables

> **Learning Objective:** Master JavaScript's function behavior, scope rules, and closures to write efficient, predictable, and maintainable code.

---

## Part 1: Function Fundamentals

### Function Declaration vs. Expression

The distinction between function declarations and expressions goes beyond syntax—it affects hoisting, scope, and how functions integrate into your code's execution model.

**Function Declaration** is a statement that creates a named function:

```javascript
function greet(name) {
  return `Hello, ${name}`;
}
```

Function declarations are **hoisted** to the top of their scope, meaning they're fully available before the code executes. This happens because the JavaScript engine parses the entire scope first, creating function bindings for all declarations.

**Function Expression** assigns a function to a variable:

```javascript
const greet = function (name) {
  return `Hello, ${name}`;
};

const add = (a, b) => a + b;
```

Function expressions are hoisted differently. With `const` or `let`, the variable name is hoisted but remains uninitialized (Temporal Dead Zone), causing a ReferenceError if accessed before assignment. This provides a safety net against accidentally using a function before it's defined.

**Why it matters**: Function declarations can lead to subtle bugs in large codebases where functions are accidentally called before definition. Using function expressions with `const` enforces intentional ordering and makes dependencies explicit. This is why modern linters recommend preferring function expressions.

### Arrow Functions vs. Regular Functions: `this`, `arguments`, and Implicit Return

Arrow functions introduce a fundamentally different behavior for context binding—they do not have their own `this`, `arguments`, or `prototype`. This is not merely a syntactic convenience; it's a semantic distinction that changes how functions interact with their environment.

**The `this` Binding Difference**:

Regular functions get their `this` value at call-time, depending on how they're invoked:

```javascript
const person = {
  name: "Alice",
  greet: function () {
    console.log(this.name); // Determined by call site
  },
  arrowGreet: () => {
    console.log(this.name); // Lexically bound at definition
  },
};

person.greet(); // "Alice" — `this` is person
person.arrowGreet(); // undefined or global object
```

Arrow functions capture `this` lexically from their enclosing scope—they inherit it from the context in which they're defined, not where they're called. This makes them ideal for callbacks:

```javascript
function User(name) {
  this.name = name;
  this.friends = [];

  // Arrow function captures `this` from User constructor
  setTimeout(() => {
    console.log(`${this.name} logged in`); // Works correctly
  }, 1000);

  // Regular function loses context
  setTimeout(function () {
    console.log(`${this.name} logged in`); // `this` is undefined/global
  }, 1000);
}
```

**The `arguments` Object**:

Regular functions receive an implicit `arguments` object—an array-like collection of all passed arguments. Arrow functions do not:

```javascript
function logArgs(a, b) {
  console.log(arguments); // [1, 2, 3]
}

const arrowLogArgs = (a, b) => {
  console.log(arguments); // ReferenceError (no arguments object)
};

logArgs(1, 2, 3);
arrowLogArgs(1, 2, 3); // Error
```

Modern code uses rest parameters (`...args`) instead, which work with both function types and provide an actual array rather than an array-like object.

**Implicit Return**:

Arrow functions with a single expression implicitly return that expression's value:

```javascript
const square = (x) => x * x;
const getUser = (id) => ({ id, name: "Alice" }); // Parentheses required for object

// Equivalent to:
function square(x) {
  return x * x;
}
```

This reduces boilerplate but can reduce clarity in complex functions. developers balance brevity with readability, using implicit return for simple operations and explicit returns for complex logic.

### Default Parameters, Rest Parameters, and Parameter Destructuring

Modern JavaScript provides powerful parameter patterns that replace legacy workarounds.

**Default Parameters** eliminate the need for conditional checks at function start:

```javascript
// Legacy approach (before ES2015)
function greet(name) {
  name = name || "Guest"; // Falsy values become "Guest"
  return `Hello, ${name}`;
}

// Modern approach
function greet(name = "Guest") {
  return `Hello, ${name}`;
}

greet(undefined); // "Hello, Guest"
greet(null); // "Hello, null" — null is not undefined
```

Default values are evaluated at call-time, not at function definition-time. This enables dynamic defaults:

```javascript
function createUser(name = "User", timestamp = Date.now()) {
  return { name, timestamp };
}

createUser("Alice"); // Fresh timestamp each call
```

**Rest Parameters** collect remaining arguments into an actual array:

```javascript
function sum(first, ...rest) {
  return first + rest.reduce((a, b) => a + b, 0);
}

sum(1, 2, 3, 4); // 10
```

Rest parameters can only appear as the last parameter and create a true array (unlike the legacy `arguments` object).

**Parameter Destructuring** extracts values from objects or arrays at function entry:

```javascript
// Object destructuring with defaults
function createConfig({ host = "localhost", port = 3000, ssl = false } = {}) {
  return { host, port, ssl };
}

createConfig({ host: "example.com" }); // Uses defaults for port, ssl

// Array destructuring
function swapCoordinates([x, y]) {
  return [y, x];
}

swapCoordinates([10, 20]); // [20, 10]
```

Destructuring in parameters enables clear, self-documenting APIs. Combined with defaults, it eliminates verbose "options object" pattern boilerplate.

### Pure vs. Impure Functions and Functional Patterns

A **pure function** returns the same output for the same input and causes no side effects. It doesn't modify external state, doesn't depend on external state (beyond its parameters), and doesn't perform I/O operations.

```javascript
// Pure function
function add(a, b) {
  return a + b; // Deterministic, no side effects
}

// Impure function
let multiplier = 2;
function multiply(x) {
  return x * multiplier; // Depends on external state
}

// Impure function
const users = [];
function addUser(name) {
  users.push(name); // Modifies external state
  return users; // Side effect
}
```

Pure functions are testable, composable, and easier to reason about. However, real applications require side effects—fetching data, updating the DOM, writing to a database. The functional paradigm advocates _isolating_ impurity, keeping the core logic pure.

**Functional Patterns** that developers use:

**Function Composition**: Combine pure functions to build more complex operations:

```javascript
const compose =
  (...fns) =>
  (x) =>
    fns.reduceRight((val, fn) => fn(val), x);

const trim = (str) => str.trim();
const toLowerCase = (str) => str.toLowerCase();
const reverse = (str) => str.split("").reverse().join("");

const processString = compose(reverse, toLowerCase, trim);
processString("  HELLO  "); // "olleh"
```

**Higher-Order Functions**: Functions that accept or return functions:

```javascript
function withCache(fn) {
  const cache = new Map();
  return (...args) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

const expensiveCompute = withCache((n) => {
  console.log("Computing...");
  return n * n;
});

expensiveCompute(5); // Logs "Computing..." and returns 25
expensiveCompute(5); // Returns 25 without logging
```

Understanding when to apply pure functions versus accepting impurity is a mark of seniority. Over-engineering functional patterns can obscure intent; pragmatic use of impurity when necessary is often the right choice.

---

## Part 2: The Scope System

### Global, Function, and Block Scope

JavaScript has multiple scope levels, each creating a new binding environment:

**Global Scope** exists at the top level of your program:

```javascript
// In browsers
var globalVar = "I'm global";
console.log(window.globalVar); // "I'm global"

// In Node.js
var globalVar = "I'm in global scope";
console.log(global.globalVar); // undefined (vars at module level are module-scoped)
```

Global scope is a source of bugs and pollution. Avoid creating global variables intentionally.

**Function Scope** is created by function declarations and expressions:

```javascript
function outer() {
  let functionScoped = "I'm inside outer";

  function inner() {
    console.log(functionScoped); // Can access
  }

  inner();
  console.log(functionScoped); // Still accessible
}

console.log(functionScoped); // ReferenceError
```

Each function creates a new scope, and inner scopes can access outer scopes (lexical scoping).

**Block Scope** is created by code blocks (if, for, while, try/catch):

```javascript
if (true) {
  let blockScoped = "I'm in a block";
  const alsoBlockScoped = "Me too";
  var varInBlock = "I escape"; // Hoisted out of block
}

console.log(blockScoped); // ReferenceError
console.log(varInBlock); // "I escape"
```

Block scope is a critical feature introduced by ES2015. It enables more precise variable scoping and prevents accidental value overwriting in loops.

### `var`, `let`, and `const` Behavior Within Scopes

**`var`** has function scope and is hoisted to the top of its function with a value of `undefined`:

```javascript
function example() {
  console.log(x); // undefined (hoisted)
  var x = 5;
  console.log(x); // 5
}

// Equivalent to:
function example() {
  var x;
  console.log(x); // undefined
  x = 5;
  console.log(x); // 5
}
```

`var` in a block scope is hoisted to the enclosing function scope, a common source of bugs:

```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Logs: 3, 3, 3 (because `i` is function-scoped, not block-scoped)
```

**`let` and `const`** have block scope and remain uninitialized during their Temporal Dead Zone (TDZ)—the period between the start of their scope and their declaration:

```javascript
function example() {
  console.log(x); // ReferenceError: Cannot access 'x' before initialization
  let x = 5;
}

// Block scope example
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Logs: 0, 1, 2 (each iteration gets its own `i` binding)
```

**`const`** cannot be reassigned but can have its properties mutated if it holds an object:

```javascript
const user = { name: "Alice" };
user.name = "Bob"; // Valid—mutating object properties
user = {}; // Error—cannot reassign const

const num = 5;
num = 6; // Error
```

developers prefer `const` by default, use `let` when reassignment is necessary, and avoid `var` entirely. This pattern leads to more intentional code and catches accidental reassignments.

### Lexical vs. Dynamic Scope

**Lexical Scope** (also called static scope) means a variable's availability is determined by its position in the source code. Where a function is _defined_ matters, not where it's _called_:

```javascript
let value = "global";

function outer() {
  let value = "outer";
  inner(); // Calls inner
}

function inner() {
  console.log(value); // "global" — uses value from definition site
}

outer(); // Logs "global"
```

`inner` looks up `value` in the scope where it was _defined_ (global scope), not where it was _called_ (inside `outer`). This is JavaScript's standard behavior.

**Dynamic Scope** would look up variables based on the call stack—where a function is _called_, not where it's defined. JavaScript uses lexical scope exclusively, but understanding dynamic scope helps clarify how lexical scope works.

If JavaScript used dynamic scope:

```javascript
// Hypothetical: if JavaScript used dynamic scope
// Suppose JavaScript had dynamic (not lexical) scope
function outer() {
  let value = "outer";
  inner(); // Would find value in call stack
}

function inner() {
  console.log(value); // Would log "outer" under dynamic scoping
}

outer(); // Would log "outer" (not the actual behavior)
```

Lexical scoping is a feature, not a bug. It makes code predictable: you know exactly where a variable comes from by reading the source code, not by tracing execution paths.

### How Closures Preserve Lexical Environments

A closure is a function that retains access to variables from its lexical scope, even after that scope has "finished executing." This isn't magic—it's a result of how JavaScript manages environments.

When a function is created, it maintains an internal `[[Environment]]` reference to the scope in which it was defined. This reference persists throughout the function's lifetime:

```javascript
function makeCounter() {
  let count = 0; // Captured in closure

  return function () {
    count++;
    return count;
  };
}

const counter1 = makeCounter();
const counter2 = makeCounter();

console.log(counter1()); // 1
console.log(counter1()); // 2
console.log(counter2()); // 1 — separate closure, separate `count`
```

Each call to `makeCounter()` creates a new environment with its own `count` variable. The returned function captures that environment. Later calls to `counter1()` execute in that captured environment, accessing and modifying the original `count`.

This preservation of lexical environments is fundamental to understanding closures. The garbage collector recognizes that the captured variables are still referenced and keeps them alive.

---

## Part 3: Closures Deep Dive

### Definition and Mental Model

A **closure** is a combination of a function and the variables it has access to from its lexical environment. Every function in JavaScript is a closure because every function retains references to its outer scopes.

The mental model: imagine a function as carrying a backpack containing all the variables from scopes it was defined in. That backpack never empties. Even if the outer function finishes executing, the variables in the backpack remain available:

```javascript
function createBackpack() {
  const backpack = { items: [] }; // In the backpack

  return {
    add: (item) => backpack.items.push(item),
    get: () => backpack.items,
    clear: () => (backpack.items.length = 0),
  };
}

const myBackpack = createBackpack();
myBackpack.add("book");
myBackpack.add("pen");
console.log(myBackpack.get()); // ["book", "pen"]
```

The three methods share access to the same `backpack` object because they're all defined in the same scope. Changes through one method are visible to others because they modify the same captured variable.

### Real-World Use Cases

**Data Privacy and Encapsulation**:

Closures enable true data privacy before JavaScript had classes:

```javascript
function createBankAccount(initialBalance) {
  let balance = initialBalance; // Private variable

  return {
    deposit: (amount) => {
      if (amount > 0) balance += amount;
      return balance;
    },
    withdraw: (amount) => {
      if (amount > 0 && amount <= balance) balance -= amount;
      return balance;
    },
    getBalance: () => balance,
  };
}

const account = createBankAccount(1000);
account.deposit(500); // 1500
account.balance = 99999; // Doesn't affect internal balance
console.log(account.getBalance()); // 1500 (still protected)
```

The `balance` variable is truly private—there's no way to access or modify it directly from outside the returned object. This is more secure than class fields with a convention of privacy (like `_balance`).

**Factory Functions**:

Closures make factory functions powerful:

```javascript
function createUser(name, email) {
  let verified = false;
  const createdAt = new Date();

  return {
    name,
    email,
    verify: () => {
      verified = true;
    },
    isVerified: () => verified,
    getCreatedDate: () => createdAt,
  };
}

const user1 = createUser("Alice", "alice@example.com");
const user2 = createUser("Bob", "bob@example.com");

user1.verify();
console.log(user1.isVerified()); // true
console.log(user2.isVerified()); // false — separate closure
```

Each user gets its own closure with private state. Factory functions are cleaner and more memory-efficient than creating a new class instance for every object.

**Event Handlers and Callbacks**:

Closures are essential for event handling and callbacks. They allow handlers to remember context:

```javascript
function setupEventListeners() {
  const users = ["Alice", "Bob", "Charlie"];

  users.forEach((user, index) => {
    const button = document.querySelector(`#user-${index}`);
    button.addEventListener("click", () => {
      console.log(`Clicked: ${user}`); // Closure captures current user
      alert(`Hello, ${user}!`);
    });
  });
}
```

Without closures (using `var` in the loop):

```javascript
// Bad: var is function-scoped
for (var i = 0; i < users.length; i++) {
  const button = document.querySelector(`#user-${i}`);
  button.addEventListener("click", () => {
    console.log(`Clicked: ${users[i]}`); // Always uses final `i` value
  });
}
```

**Caching and Memoization**:

Closures store computed results:

```javascript
function memoize(fn) {
  const cache = new Map();

  return function (...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      console.log("Returning cached result");
      return cache.get(key);
    }

    console.log("Computing result");
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

const fibonacci = memoize((n) => {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
});

console.time("First call");
console.log(fibonacci(10)); // Computes
console.timeEnd("First call");

console.time("Second call");
console.log(fibonacci(10)); // Returns cached
console.timeEnd("Second call");
```

**Module Pattern**:

Closures enable the module pattern, encapsulating private and public interfaces:

```javascript
const Calculator = (function () {
  let lastResult = 0; // Private state

  function calculate(operation, a, b) {
    switch (operation) {
      case "add":
        lastResult = a + b;
        break;
      case "multiply":
        lastResult = a * b;
        break;
    }
    return lastResult;
  }

  return {
    // Public API
    add: (a, b) => calculate("add", a, b),
    multiply: (a, b) => calculate("multiply", a, b),
    getLast: () => lastResult,
  };
})();

console.log(Calculator.add(5, 3)); // 8
console.log(Calculator.multiply(4, 2)); // 8
console.log(Calculator.lastResult); // undefined (private)
console.log(Calculator.getLast()); // 8
```

**Loop Variable Capture**:

A classic gotcha when using closures in loops:

```javascript
// Wrong: captures the variable, not the value
const handlers = [];
for (var i = 0; i < 3; i++) {
  handlers.push(() => console.log(i));
}
handlers[0](); // 3 (final value of i)
handlers[1](); // 3
handlers[2](); // 3

// Correct: use let for block scope
const handlers2 = [];
for (let i = 0; i < 3; i++) {
  handlers2.push(() => console.log(i));
}
handlers2[0](); // 0
handlers2[1](); // 1
handlers2[2](); // 2

// Correct: IIFE creates a new scope per iteration
const handlers3 = [];
for (var i = 0; i < 3; i++) {
  handlers3.push(
    (function (j) {
      return () => console.log(j);
    })(i)
  );
}
handlers3[0](); // 0
handlers3[1](); // 1
handlers3[2](); // 2
```

**Unintended Global Leaks**:

Forgetting `const`, `let`, or `var` creates a global variable:

```javascript
function leaky() {
  globalLeak = "I pollute global scope"; // Missing declaration
}

leaky();
console.log(window.globalLeak); // "I pollute global scope"
```

Enable strict mode to prevent this:

```javascript
"use strict";
function notLeaky() {
  globalLeak = "This throws an error"; // ReferenceError
}
```

---

## Part 4: Advanced Concepts

### Immediately Invoked Function Expressions (IIFE)

An IIFE is a function that's defined and immediately executed:

```javascript
(function () {
  console.log("I run immediately!");
})();

// With parameters
(function (name) {
  console.log(`Hello, ${name}!`);
})("Alice");

// Arrow function IIFE
(() => {
  console.log("Arrow IIFE");
})();

// Returning values
const result = (function () {
  return 42;
})();
console.log(result); // 42
```

**Historical Context**: IIFEs were essential before ES2015 for creating a new scope and avoiding global namespace pollution. They're less necessary now that we have block scope with `let` and `const`, but they're still useful.

**Modern Use Cases**:

Creating an initialization scope:

```javascript
// Initialize with privacy
(function setupApp() {
  const config = { apiUrl: "https://api.example.com" };
  const cache = new Map();

  window.App = {
    fetch: (url) => {
      if (cache.has(url)) return cache.get(url);
      // Fetch and cache
    },
  };
})();

// config and cache are private
```

Handling async initialization:

```javascript
(async function initializeDatabase() {
  const db = await openDatabase();
  window.db = db;
})();
```

### Module Pattern Using Closures

Before ES6 modules, the module pattern used closures and IIFEs to create encapsulated code:

```javascript
const TodoList = (function () {
  // Private variables
  let todos = [];
  let id = 0;

  // Private functions
  function generateId() {
    return ++id;
  }

  function findTodo(todoId) {
    return todos.find((todo) => todo.id === todoId);
  }

  // Public API
  return {
    add: function (title) {
      const todo = { id: generateId(), title, completed: false };
      todos.push(todo);
      return todo;
    },

    complete: function (todoId) {
      const todo = findTodo(todoId);
      if (todo) {
        todo.completed = true;
      }
      return todo;
    },

    getAll: function () {
      return [...todos]; // Return copy to prevent external mutation
    },

    clear: function () {
      todos = [];
      id = 0;
    },
  };
})();

TodoList.add("Learn closures");
console.log(TodoList.getAll());
console.log(TodoList.todos); // undefined (private)
```

**Compared to ES6 Modules**:

```javascript
// todos.js
let todos = [];
let id = 0;

function generateId() {
  return ++id;
}

function findTodo(todoId) {
  return todos.find((todo) => todo.id === todoId);
}

export const add = (title) => {
  const todo = { id: generateId(), title, completed: false };
  todos.push(todo);
  return todo;
};

export const complete = (todoId) => {
  const todo = findTodo(todoId);
  if (todo) todo.completed = true;
  return todo;
};

export const getAll = () => [...todos];

// app.js
import { add, complete, getAll } from "./todos.js";
```

ES6 modules are now the standard, but understanding the closure-based module pattern helps explain how ES6 modules achieve encapsulation.

### Scope Chain and Execution Context Visualization

When JavaScript resolves a variable, it searches through a **scope chain**—a series of nested scopes:

```javascript
let global = "global scope";

function outer() {
  let outerVar = "outer scope";

  function middle() {
    let middleVar = "middle scope";

    function inner() {
      let innerVar = "inner scope";

      console.log(innerVar); // Found in inner's scope
      console.log(middleVar); // Found in middle's scope
      console.log(outerVar); // Found in outer's scope
      console.log(global); // Found in global scope
    }

    inner();
  }

  middle();
}

outer();
```

The scope chain: `inner` → `middle` → `outer` → `global`

JavaScript searches left-to-right. If a variable isn't found, it looks in the parent scope, then the parent's parent, and so on. If not found anywhere, a ReferenceError is thrown.

**Shadowing** occurs when an inner scope redefines a name from an outer scope:

```javascript
let name = "global";

function outer() {
  let name = "outer";

  function inner() {
    let name = "inner";
    console.log(name); // "inner" — uses the innermost binding
  }

  inner();
  console.log(name); // "outer"
}

outer();
console.log(name); // "global"
```

The innermost `name` shadows outer definitions. This is often unintentional and can cause bugs. Linters warn about shadowing to prevent confusion.

**Execution Context** is a broader concept that includes the scope chain, `this` value, and variable environment:

When a function executes, JavaScript creates an execution context with:

1. **Lexical Environment**: Variable bindings and scope chain
2. **This Binding**: The `this` value for the function
3. **Outer Environment Reference**: Reference to parent scope chain

```javascript
const user = {
  name: "Alice",
  greet: function () {
    // Execution context for greet():
    // - Lexical Environment: { name: "Alice", greet: function }
    // - this: user
    // - Outer: global scope

    const message = `Hello, ${this.name}`;

    const delayed = () => {
      // Execution context for delayed():
      // - Lexical Environment: { message }
      // - this: user (captured from greet's context)
      // - Outer: greet's context

      console.log(message);
      console.log(this.name);
    };

    setTimeout(delayed, 1000);
  },
};

user.greet();
```

Understanding execution context helps explain why arrow functions work differently for `this`, how closures preserve their environment, and why certain code patterns succeed or fail.
