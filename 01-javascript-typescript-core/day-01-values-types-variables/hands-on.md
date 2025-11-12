# Day 1 — Hands-on: Quick Coercion Drills & Refactoring

> **Practice Goal:** Build muscle memory for type coercion rules and modern variable declarations through practical exercises.

---

## 🎯 Exercise 1: Type Coercion Challenge

### Predict the Output (No Peeking!)

**Instructions:** Write down your predictions for each console.log before running the code.

#### Round 1: String Coercion

```js
console.log("5" + 3);
console.log("5" + +"3");
console.log("5" - "3");
console.log("5" * "2");
console.log("10" / "2");
```

<details>
<summary>💡 Show Answers</summary>

```js
"53"; // String concatenation
"53"; // '5' + (+'3') → '5' + 3 → '53'
2; // Numeric subtraction
10; // Numeric multiplication
5; // Numeric division
```

</details>

---

#### Round 2: Boolean Coercion

```js
console.log(Boolean(""));
console.log(Boolean("0"));
console.log(Boolean([]));
console.log(Boolean({}));
console.log(Boolean(null));
```

<details>
<summary>💡 Show Answers</summary>

```js
false; // Empty string is falsy
true; // Non-empty string is truthy
true; // Empty array is truthy
true; // Empty object is truthy
false; // null is falsy
```

</details>

---

#### Round 3: Equality

```js
console.log(0 == "0");
console.log(0 === "0");
console.log(null == undefined);
console.log(null === undefined);
console.log("" == 0);
```

<details>
<summary>💡 Show Answers</summary>

```js
true; // '0' coerced to 0
false; // Different types
true; // Special case
false; // Different types
true; // '' coerced to 0
```

</details>

---

#### Round 4: Array Coercion

```js
console.log([] + []);
console.log([] + {});
console.log({} + []);
console.log([1, 2] + [3, 4]);
console.log([1] == 1);
```

<details>
<summary>💡 Show Answers</summary>

```js
""; // [] → '', '' + '' → ''
"[object Object]"; // [] → '', {} → '[object Object]'
"[object Object]"; // or 0 (depends on context)
"1,23,4"; // [1,2] → '1,2', [3,4] → '3,4'
true; // [1] → '1' → 1
```

</details>

---

#### Round 5: Unary Plus

```js
console.log(+"42");
console.log(+true);
console.log(+false);
console.log(+null);
console.log(+undefined);
console.log(+[]);
console.log(+[5]);
console.log(+[1, 2]);
```

<details>
<summary>💡 Show Answers</summary>

```js
42; // String to number
1; // true → 1
0; // false → 0
0; // null → 0
NaN; // undefined → NaN
0; // [] → '' → 0
5; // [5] → '5' → 5
NaN; // [1,2] → '1,2' → NaN
```

</details>

---

#### Round 6: Logical Operators

```js
console.log("hello" && "world");
console.log(0 && "hello");
console.log("" || "default");
console.log(null || undefined || 0);
console.log(5 && 10 && 15);
```

<details>
<summary>💡 Show Answers</summary>

```js
"world"; // Last truthy value
0; // First falsy value
("default"); // First truthy value
0; // Last value (all falsy)
15; // Last truthy value
```

</details>

---

#### Round 7: Mind Benders 🤯

```js
console.log([] == ![]);
console.log("" == []);
console.log(0 == []);
console.log("0" == []);
console.log(false == []);
```

<details>
<summary>💡 Show Answers</summary>

```js
true; // [] → '' → 0, ![] → false → 0
true; // [] → '', '' == ''
true; // [] → '' → 0
false; // [] → '', '0' != ''
true; // [] → '' → 0, false → 0
```

**Explanation for `[] == ![]`:**

```js
// Step 1: ![] → false (array is truthy)
// Step 2: [] == false
// Step 3: [] → '' (ToPrimitive)
// Step 4: '' == false
// Step 5: '' → 0, false → 0
// Step 6: 0 == 0 → true
```

</details>

---

#### Round 8: Comparisons

```js
console.log("2" > "12");
console.log("2" > 12);
console.log([5] > 4);
console.log([10] < [9]);
```

<details>
<summary>💡 Show Answers</summary>

```js
true; // Lexicographic: '2' > '1'
false; // '2' → 2, 2 > 12 is false
true; // [5] → '5' → 5, 5 > 4
false; // Lexicographic: '10' < '9'
```

</details>

---

## 🔧 Exercise 2: Variable Scope Challenge

### Predict the Output

#### Challenge 1: Nested Scopes

```js
var x = 1;
function outer() {
  var x = 2;
  function inner() {
    var x = 3;
    console.log(x);
  }
  inner();
  console.log(x);
}
outer();
console.log(x);
```

<details>
<summary>💡 Show Answer</summary>

```js
3; // innermost x
2; // middle x
1; // global x
```

</details>

---

#### Challenge 2: Block Scope

```js
let a = 10;
{
  let a = 20;
  {
    let a = 30;
    console.log(a);
  }
  console.log(a);
}
console.log(a);
```

<details>
<summary>💡 Show Answer</summary>

```js
30; // innermost block
20; // middle block
10; // outer scope
```

</details>

---

#### Challenge 3: Const Mutability

```js
const arr = [1, 2, 3];
arr.push(4);
console.log(arr);
arr = [5, 6, 7]; // What happens?
```

<details>
<summary>💡 Show Answer</summary>

```js
[1, 2, 3, 4]; // push works
TypeError; // Cannot reassign const
```

</details>

---

#### Challenge 4: Loop Closures

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}

for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 0);
}
```

<details>
<summary>💡 Show Answer</summary>

```js
3, 3, 3; // var shares i across iterations
0, 1, 2; // let creates new j per iteration
```

</details>

---

#### Challenge 5: Hoisting

```js
console.log(typeof myFunc);
console.log(typeof myVar);
console.log(typeof myLet);

function myFunc() {}
var myVar = "value";
let myLet = "value";
```

<details>
<summary>💡 Show Answer</summary>

```js
"function"; // Function declaration hoisted
"undefined"; // var hoisted but not initialized
ReferenceError; // let in TDZ
```

</details>

---

## 🛠️ Exercise 3: Real-World Refactoring

### Task: Modernize Legacy Code

**Original Code (Legacy):**

```js
// Legacy code with var
var userName = "Guest";
var userAge = 0;
var isLoggedIn = false;

function login(name, age) {
  if (name) {
    var userName = name; // Shadows outer variable
    var userAge = age;
    var isLoggedIn = true;

    if (userAge >= 18) {
      var accessLevel = "adult";
    } else {
      var accessLevel = "minor";
    }

    console.log("Welcome " + userName);
    console.log("Access: " + accessLevel);
  }

  return isLoggedIn;
}

var users = [];
for (var i = 0; i < 3; i++) {
  users.push({
    id: i,
    getName: function () {
      return "User " + i;
    },
  });
}

console.log(users[0].getName()); // What does this print?
```

**Requirements for Refactoring:**

1. Replace `var` with `const` or `let` appropriately
2. Use template literals for string concatenation
3. Use arrow functions where appropriate
4. Fix the closure bug in the loop
5. Fix variable shadowing issues

<details>
<summary>💡 Show Refactored Solution</summary>

```js
// Modern code with let/const
let userName = "Guest";
let userAge = 0;
let isLoggedIn = false;

function login(name, age) {
  if (name) {
    userName = name; // Update outer variable
    userAge = age;
    isLoggedIn = true;

    const accessLevel = userAge >= 18 ? "adult" : "minor";

    console.log(`Welcome ${userName}`);
    console.log(`Access: ${accessLevel}`);
  }

  return isLoggedIn;
}

const users = [];
for (let i = 0; i < 3; i++) {
  users.push({
    id: i,
    getName: () => `User ${i}`, // Arrow function captures correct i
  });
}

console.log(users[0].getName()); // 'User 0' ✅
console.log(users[1].getName()); // 'User 1' ✅
console.log(users[2].getName()); // 'User 2' ✅
```

**Key Changes:**

- Used `let` for variables that change (`userName`, `userAge`, `isLoggedIn`)
- Used `const` for the `users` array and `accessLevel`
- Replaced string concatenation with template literals
- Used arrow function to capture loop variable correctly
- Used ternary operator for cleaner conditional logic
- Fixed variable shadowing by updating outer variables instead of creating new ones

</details>

---

## 🎓 Exercise 4: Debug the Bugs

### Find and Fix the Errors

#### Bug 1: Type Confusion

```js
function calculateTotal(price, quantity) {
  return price * quantity;
}

const item1 = calculateTotal("10", 2);
const item2 = calculateTotal("5", "3");
const item3 = calculateTotal("$20", 2);

console.log(item1); // What prints?
console.log(item2); // What prints?
console.log(item3); // What prints?
```

<details>
<summary>💡 Show Answer & Fix</summary>

**Output:**

```js
20; // '10' * 2 → 10 * 2 (works by accident)
15; // '5' * '3' → 5 * 3 (works by accident)
NaN; // '$20' * 2 → NaN (fails!)
```

**Fixed Version:**

```js
function calculateTotal(price, quantity) {
  // Validate and convert inputs
  const numPrice = Number(price);
  const numQuantity = Number(quantity);

  if (isNaN(numPrice) || isNaN(numQuantity)) {
    throw new Error("Invalid price or quantity");
  }

  return numPrice * numQuantity;
}

// Or use stricter typing
function calculateTotal(price, quantity) {
  if (typeof price !== "number" || typeof quantity !== "number") {
    throw new TypeError("Price and quantity must be numbers");
  }
  return price * quantity;
}
```

</details>

---

#### Bug 2: Loop Variable Leak

```js
function processItems(items) {
  for (var i = 0; i < items.length; i++) {
    setTimeout(function () {
      console.log(`Processing item ${i}: ${items[i]}`);
    }, i * 1000);
  }
  console.log(`Total items: ${i}`);
}

processItems(["A", "B", "C"]);
```

<details>
<summary>💡 Show Answer & Fix</summary>

**Problem:**

- `i` leaks outside the loop
- All timeouts reference the same `i` (which becomes 3)
- Prints "Processing item 3: undefined" three times

**Fixed Version:**

```js
function processItems(items) {
  for (let i = 0; i < items.length; i++) {
    setTimeout(function () {
      console.log(`Processing item ${i}: ${items[i]}`);
    }, i * 1000);
  }
  console.log(`Total items: ${items.length}`);
}

processItems(["A", "B", "C"]);
// Output:
// Total items: 3
// Processing item 0: A
// Processing item 1: B
// Processing item 2: C
```

**Alternative with forEach:**

```js
function processItems(items) {
  items.forEach((item, i) => {
    setTimeout(() => {
      console.log(`Processing item ${i}: ${item}`);
    }, i * 1000);
  });
  console.log(`Total items: ${items.length}`);
}
```

</details>

---

#### Bug 3: Premature Access

```js
function greet(name) {
  console.log(`Hello, ${fullName}`);

  if (name) {
    let fullName = `${name} Smith`;
  }

  console.log(`Goodbye, ${fullName}`);
}

greet("John");
```

<details>
<summary>💡 Show Answer & Fix</summary>

**Problem:**

- `fullName` is block-scoped to the if statement
- Both console.logs are outside that scope
- Both will throw ReferenceError

**Fixed Version:**

```js
function greet(name) {
  let fullName = name ? `${name} Smith` : "Guest";

  console.log(`Hello, ${fullName}`);
  console.log(`Goodbye, ${fullName}`);
}

greet("John");
// Output:
// Hello, John Smith
// Goodbye, John Smith
```

</details>

---

## 💪 Exercise 5: Build Your Own Examples

### Create Examples That Demonstrate:

1. **A situation where `==` gives unexpected results**

   ```js
   // Your example here
   ```

2. **A closure bug with `var` and how `let` fixes it**

   ```js
   // Your example here
   ```

3. **Type coercion causing a calculation error**

   ```js
   // Your example here
   ```

4. **Temporal Dead Zone (TDZ) error**

   ```js
   // Your example here
   ```

5. **The difference between immutable binding and immutable value**
   ```js
   // Your example here
   ```

---

## 🎯 Rapid Fire Drill (10 seconds each!)

Answer quickly without running the code:

```js
1. typeof null
2. typeof []
3. 0 == false
4. 0 === false
5. '' == 0
6. '' === 0
7. Boolean([])
8. Boolean({})
9. [] == []
10. NaN === NaN
11. undefined == null
12. undefined === null
13. '5' + 3
14. '5' - 3
15. [] + {}
16. let x; console.log(x);
17. const y; console.log(y);
18. +'42'
19. !!'0'
20. 2 ** 3
```

<details>
<summary>💡 Show All Answers</summary>

```js
1. 'object' ⚠️
2. 'object'
3. true
4. false
5. true
6. false
7. true
8. true
9. false (different references)
10. false
11. true
12. false
13. '53'
14. 2
15. '[object Object]'
16. undefined
17. SyntaxError (const must be initialized)
18. 42
19. true
20. 8
```

</details>

---

## 🏆 Final Challenge: Code Review

Review this code and list all the issues:

```js
var API_KEY = "secret123";

function fetchUserData(userId) {
  var url = "https://api.example.com/users/" + userId;

  if (userId) {
    var isValid = true;
  }

  if (isValid) {
    var data = fetch(url);
    return data;
  }
}

var users = [];
for (var i = 0; i < 5; i++) {
  users.push({
    id: i,
    fetch: function () {
      return fetchUserData(i);
    },
  });
}

console.log(users[0].fetch());
```

<details>
<summary>💡 Show Issues & Fixed Version</summary>

**Issues Found:**

1. `API_KEY` should be `const` (won't be reassigned)
2. Using `var` instead of `let`/`const`
3. String concatenation instead of template literals
4. `isValid` can leak outside if block
5. Closure bug: all `fetch` functions will use `i = 5`
6. Missing error handling for `fetch`
7. Not returning the promise properly

**Fixed Version:**

```js
const API_KEY = "secret123";

async function fetchUserData(userId) {
  if (!userId) {
    throw new Error("User ID is required");
  }

  const url = `https://api.example.com/users/${userId}`;

  try {
    const response = await fetch(url);
    return await response.json();
  } catch (error) {
    console.error("Failed to fetch user:", error);
    throw error;
  }
}

const users = [];
for (let i = 0; i < 5; i++) {
  users.push({
    id: i,
    fetch: () => fetchUserData(i), // Arrow function captures correct i
  });
}

// Usage
users[0].fetch().then((data) => console.log(data));
```

</details>
