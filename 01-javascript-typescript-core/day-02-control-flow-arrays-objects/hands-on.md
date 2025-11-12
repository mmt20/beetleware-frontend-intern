# Day 2 — Hands-on: Control Flow & Data Structures Mastery

> **Practice Goal:** Build muscle memory for control flow decisions, array transformations, and object manipulations through practical, performance-oriented exercises.

---

## 🎯 Part 1: Control Flow Challenges

### Exercise 1: FizzBuzz Evolution

**Instructions:** Implement FizzBuzz three different ways and compare their characteristics.

#### Variant 1: Classic If-Else

```js
function fizzBuzzIfElse(n) {
  for (let i = 1; i <= n; i++) {
    if (i % 3 === 0 && i % 5 === 0) {
      console.log("FizzBuzz");
    } else if (i % 3 === 0) {
      console.log("Fizz");
    } else if (i % 5 === 0) {
      console.log("Buzz");
    } else {
      console.log(i);
    }
  }
}

fizzBuzzIfElse(15);
```

<details>
<summary>💡 Characteristics</summary>

**Pros:**

- Most readable for beginners
- Clear logical flow
- Easy to debug

**Cons:**

- Multiple condition checks
- Verbose
- Hard to extend with new rules

</details>

---

#### Variant 2: Switch Statement

```js
function fizzBuzzSwitch(n) {
  for (let i = 1; i <= n; i++) {
    const divisibleBy3 = i % 3 === 0;
    const divisibleBy5 = i % 5 === 0;
    const key = `${divisibleBy3}-${divisibleBy5}`;

    switch (key) {
      case "true-true":
        console.log("FizzBuzz");
        break;
      case "true-false":
        console.log("Fizz");
        break;
      case "false-true":
        console.log("Buzz");
        break;
      default:
        console.log(i);
    }
  }
}

fizzBuzzSwitch(15);
```

<details>
<summary>💡 Characteristics</summary>

**Pros:**

- Clear case handling
- Easy to add new cases
- Single condition evaluation

**Cons:**

- More complex setup
- Requires string key creation
- Not as intuitive for this use case

</details>

---

#### Variant 3: Ternary & String Building

```js
function fizzBuzzTernary(n) {
  for (let i = 1; i <= n; i++) {
    const fizz = i % 3 === 0 ? "Fizz" : "";
    const buzz = i % 5 === 0 ? "Buzz" : "";
    console.log(fizz + buzz || i);
  }
}

fizzBuzzTernary(15);
```

<details>
<summary>💡 Characteristics</summary>

**Pros:**

- Most concise
- Highly extensible (add more words easily)
- Smart use of truthy/falsy values
- Best performance (fewer branches)

**Cons:**

- Less obvious for beginners
- Requires understanding of `||` operator

**Winner:** Ternary approach for production code! ✅

</details>

---

#### 🏆 Challenge: FizzBuzzPop

Extend FizzBuzz with a new rule:

- Divisible by 7: "Pop"
- Divisible by 3 and 7: "FizzPop"
- Divisible by 5 and 7: "BuzzPop"
- Divisible by 3, 5, and 7: "FizzBuzzPop"

<details>
<summary>💡 Show Solution</summary>

```js
function fizzBuzzPop(n) {
  for (let i = 1; i <= n; i++) {
    const fizz = i % 3 === 0 ? "Fizz" : "";
    const buzz = i % 5 === 0 ? "Buzz" : "";
    const pop = i % 7 === 0 ? "Pop" : "";
    const result = fizz + buzz + pop;
    console.log(result || i);
  }
}

fizzBuzzPop(21);
// 1, 2, Fizz, 4, Buzz, Fizz, Pop, 8, Fizz, Buzz, 11, Fizz, 13, Pop, FizzBuzz, 16, 17, Fizz, 19, Buzz, FizzPop
```

**Why this approach wins:**

- Adding new rules is just one line
- No complex conditional logic
- Easy to maintain

</details>

---

### Exercise 3: Grade Calculator with Multiple Conditions

```js
function calculateGrade(score) {
  // Validate input
  if (typeof score !== "number" || score < 0 || score > 100) {
    return "Invalid score";
  }

  // Method 1: If-Else Chain
  if (score >= 90) return "A";
  else if (score >= 80) return "B";
  else if (score >= 70) return "C";
  else if (score >= 60) return "D";
  else return "F";
}

function calculateGradeSwitch(score) {
  // Validate input
  if (typeof score !== "number" || score < 0 || score > 100) {
    return "Invalid score";
  }

  // Method 2: Switch with Math.floor
  const grade = Math.floor(score / 10);

  switch (grade) {
    case 10:
    case 9:
      return "A";
    case 8:
      return "B";
    case 7:
      return "C";
    case 6:
      return "D";
    default:
      return "F";
  }
}

// Test cases
console.log(calculateGrade(95)); // A
console.log(calculateGrade(85)); // B
console.log(calculateGrade(75)); // C
console.log(calculateGrade(65)); // D
console.log(calculateGrade(55)); // F
console.log(calculateGrade(105)); // Invalid score
console.log(calculateGrade("85")); // Invalid score
```

---

## 🔄 Part 2: Array Transformations

### Exercise 4: Array to Object Conversion

**Scenario:** Convert an array of users into a lookup map by ID for O(1) access.

#### Challenge: Build a User Lookup Map

```js
const users = [
  { id: 1, name: "Alice", role: "admin" },
  { id: 2, name: "Bob", role: "user" },
  { id: 3, name: "Charlie", role: "moderator" },
  { id: 4, name: "Diana", role: "user" },
];

// Method 1: Using reduce
const userMapReduce = users.reduce((acc, user) => {
  acc[user.id] = user;
  return acc;
}, {});

console.log(userMapReduce);
// {
//   1: { id: 1, name: 'Alice', role: 'admin' },
//   2: { id: 2, name: 'Bob', role: 'user' },
//   ...
// }

// Method 2: Using Object.fromEntries (Modern ✨)
const userMapModern = Object.fromEntries(users.map((user) => [user.id, user]));

console.log(userMapModern);

// Usage: O(1) lookup
console.log(userMapModern[2]); // { id: 2, name: 'Bob', role: 'user' }
```

<details>
<summary>💡 Performance Comparison</summary>

**Array.find() - O(n):**

```js
const bob = users.find((user) => user.id === 2); // Slow for large arrays
```

**Object lookup - O(1):**

```js
const bob = userMapModern[2]; // Fast! ✅
```

**Winner:** Object lookup for frequent access! 🏆

</details>

---

#### 🏆 Challenge: Group By Property

Group users by their role.

<details>
<summary>💡 Show Solution</summary>

```js
const users = [
  { id: 1, name: "Alice", role: "admin" },
  { id: 2, name: "Bob", role: "user" },
  { id: 3, name: "Charlie", role: "moderator" },
  { id: 4, name: "Diana", role: "user" },
  { id: 5, name: "Eve", role: "user" },
];

const groupByRole = users.reduce((acc, user) => {
  if (!acc[user.role]) {
    acc[user.role] = [];
  }
  acc[user.role].push(user);
  return acc;
}, {});

console.log(groupByRole);
// {
//   admin: [{ id: 1, name: 'Alice', role: 'admin' }],
//   user: [
//     { id: 2, name: 'Bob', role: 'user' },
//     { id: 4, name: 'Diana', role: 'user' },
//     { id: 5, name: 'Eve', role: 'user' }
//   ],
//   moderator: [{ id: 3, name: 'Charlie', role: 'moderator' }]
// }

// Modern approach with Object.groupBy (ES2024) 🔥
// const groupByRole = Object.groupBy(users, user => user.role);
```

</details>

---

### Exercise 5: Filter → Map → Reduce Chain

**Scenario:** Calculate the total price of items in a shopping cart with discounts.

#### Challenge: Cart Total Calculator

```js
const cartItems = [
  { id: 1, name: "Laptop", price: 1200, quantity: 1, inStock: true },
  { id: 2, name: "Mouse", price: 25, quantity: 2, inStock: true },
  { id: 3, name: "Keyboard", price: 75, quantity: 1, inStock: false },
  { id: 4, name: "Monitor", price: 300, quantity: 2, inStock: true },
  { id: 5, name: "USB Cable", price: 10, quantity: 3, inStock: true },
];

// Step 1: Filter only in-stock items
// Step 2: Map to calculate subtotal (price * quantity)
// Step 3: Reduce to calculate total
const total = cartItems
  .filter((item) => item.inStock)
  .map((item) => item.price * item.quantity)
  .reduce((sum, subtotal) => sum + subtotal, 0);

console.log(`Total: $${total}`); // Total: $1880

// With detailed logging
const totalWithLogging = cartItems
  .filter((item) => {
    console.log(`Checking ${item.name}: ${item.inStock ? "✅" : "❌"}`);
    return item.inStock;
  })
  .map((item) => {
    const subtotal = item.price * item.quantity;
    console.log(`${item.name}: $${item.price} × ${item.quantity} = $${subtotal}`);
    return subtotal;
  })
  .reduce((sum, subtotal) => {
    console.log(`Running total: $${sum} + $${subtotal} = $${sum + subtotal}`);
    return sum + subtotal;
  }, 0);

console.log(`Final Total: $${totalWithLogging}`);
```

<details>
<summary>💡 Show Expected Output</summary>

```
Checking Laptop: ✅
Checking Mouse: ✅
Checking Keyboard: ❌
Checking Monitor: ✅
Checking USB Cable: ✅
Laptop: $1200 × 1 = $1200
Mouse: $25 × 2 = $50
Monitor: $300 × 2 = $600
USB Cable: $10 × 3 = $30
Running total: $0 + $1200 = $1200
Running total: $1200 + $50 = $1250
Running total: $1250 + $600 = $1850
Running total: $1850 + $30 = $1880
Final Total: $1880
```

</details>

---

#### 🏆 Challenge: Apply Discount

Apply a 10% discount to items over $100, then calculate the total.

<details>
<summary>💡 Show Solution</summary>

```js
const cartItems = [
  { id: 1, name: "Laptop", price: 1200, quantity: 1, inStock: true },
  { id: 2, name: "Mouse", price: 25, quantity: 2, inStock: true },
  { id: 3, name: "Keyboard", price: 75, quantity: 1, inStock: false },
  { id: 4, name: "Monitor", price: 300, quantity: 2, inStock: true },
  { id: 5, name: "USB Cable", price: 10, quantity: 3, inStock: true },
];

const totalWithDiscount = cartItems
  .filter((item) => item.inStock)
  .map((item) => {
    const subtotal = item.price * item.quantity;
    // Apply 10% discount if subtotal > $100
    const discount = subtotal > 100 ? subtotal * 0.1 : 0;
    const finalPrice = subtotal - discount;

    console.log(`${item.name}: $${subtotal.toFixed(2)} - $${discount.toFixed(2)} = $${finalPrice.toFixed(2)}`);

    return finalPrice;
  })
  .reduce((sum, subtotal) => sum + subtotal, 0);

console.log(`\nTotal with discounts: $${totalWithDiscount.toFixed(2)}`);
// Total with discounts: $1602.00
```

**Output:**

```
Laptop: $1200.00 - $120.00 = $1080.00
Mouse: $50.00 - $0.00 = $50.00
Monitor: $600.00 - $60.00 = $540.00
USB Cable: $30.00 - $0.00 = $30.00

Total with discounts: $1700.00
```

</details>

---

### Exercise 6: Safe Sum Calculator

**Scenario:** Create a robust sum calculator that handles various input types.

```js
function safeSum(...values) {
  // Validate all inputs are numbers
  const numbers = values.filter((val) => {
    const isValid = typeof val === "number" && !isNaN(val);
    if (!isValid) {
      console.warn(`Ignoring invalid value: ${val}`);
    }
    return isValid;
  });

  if (numbers.length === 0) {
    throw new Error("No valid numbers provided");
  }

  return numbers.reduce((sum, num) => sum + num, 0);
}

// Test cases
console.log(safeSum(1, 2, 3)); // 6
console.log(safeSum(1, "2", 3)); // 4 (ignores '2')
console.log(safeSum(1, null, 3)); // 4 (ignores null)
console.log(safeSum(1, undefined, 3)); // 4 (ignores undefined)
console.log(safeSum(1, NaN, 3)); // 4 (ignores NaN)

try {
  console.log(safeSum("a", "b", "c")); // Error
} catch (e) {
  console.error(e.message);
}
```

#### 🏆 Challenge: Enhanced Calculator

Add support for string numbers (like "5") by attempting conversion.

<details>
<summary>💡 Show Solution</summary>

```js
function safeSumEnhanced(...values) {
  const numbers = values
    .map((val) => {
      // Try to convert to number
      if (typeof val === "string") {
        const num = Number(val);
        if (!isNaN(num)) {
          console.log(`Converted "${val}" to ${num}`);
          return num;
        }
      }
      return val;
    })
    .filter((val) => {
      const isValid = typeof val === "number" && !isNaN(val);
      if (!isValid) {
        console.warn(`Ignoring invalid value: ${val}`);
      }
      return isValid;
    });

  if (numbers.length === 0) {
    throw new Error("No valid numbers provided");
  }

  return numbers.reduce((sum, num) => sum + num, 0);
}

// Test cases
console.log(safeSumEnhanced(1, "2", 3)); // 6 (converts "2")
console.log(safeSumEnhanced("10", "20", "30")); // 60
console.log(safeSumEnhanced(1, "abc", 3)); // 4 (ignores "abc")
```

</details>

---

## 🐛 Part 3: Debugging Practice

### Exercise 7: Identify Logic Errors

#### Bug 1: Filter Gone Wrong

```js
const numbers = [1, 2, 3, 4, 5];

// Intent: Keep only even numbers
const evens = numbers.filter((num) => {
  return num % 2 === 0;
  num; // ❌ This line does nothing!
});

console.log(evens);
// What does this print?
```

<details>
<summary>💡 Show Issue & Fix</summary>

**Issue:** The return statement returns before reaching `num`, so it works correctly by accident. However, the extra `num;` line is dead code.

**Output:** `[2, 4]` ✅ (works but has dead code)

**Fixed:**

```js
const evens = numbers.filter((num) => num % 2 === 0);
```

</details>

---

#### Bug 2: Reduce Mishap

```js
const prices = [10, 20, 30, 40];

const total = prices.reduce((sum, price) => {
  sum + price; // ❌ Missing return!
});

console.log(total);
// What does this print?
```

<details>
<summary>💡 Show Issue & Fix</summary>

**Issue:** Missing return statement causes `undefined` accumulation.

**Output:** `undefined`

**Fixed:**

```js
const total = prices.reduce((sum, price) => sum + price, 0);
// Or with explicit return:
const total = prices.reduce((sum, price) => {
  return sum + price;
}, 0);
```

</details>

---

#### Bug 3: Array Reference Issue

```js
const original = [1, 2, 3];
const modified = original;

modified.push(4);

console.log(original);
// What does this print?
```

<details>
<summary>💡 Show Issue & Fix</summary>

**Issue:** Arrays are reference types. `modified` points to the same array as `original`.

**Output:** `[1, 2, 3, 4]` (original is modified!)

**Fixed:**

```js
const modified = [...original]; // Create a copy
modified.push(4);
console.log(original); // [1, 2, 3] ✅
console.log(modified); // [1, 2, 3, 4] ✅
```

</details>

---

#### Bug 4: Object Mutation

```js
const user = { name: "Alice", age: 25 };

function updateAge(userObj, newAge) {
  userObj.age = newAge;
  return userObj;
}

const updatedUser = updateAge(user, 26);
console.log(user.age); // What prints?
console.log(updatedUser.age); // What prints?
```

<details>
<summary>💡 Show Issue & Fix</summary>

**Issue:** Function mutates the original object.

**Output:** Both print `26` (original is mutated!)

**Fixed (Immutable approach):**

```js
function updateAge(userObj, newAge) {
  return { ...userObj, age: newAge };
}

const updatedUser = updateAge(user, 26);
console.log(user.age); // 25 ✅
console.log(updatedUser.age); // 26 ✅
```

</details>

---

### Exercise 8: Debugging with Modern Tools

#### Using console.table for Arrays of Objects

```js
const students = [
  { id: 1, name: "Alice", grade: 85, passed: true },
  { id: 2, name: "Bob", grade: 72, passed: true },
  { id: 3, name: "Charlie", grade: 55, passed: false },
  { id: 4, name: "Diana", grade: 90, passed: true },
];

// ❌ Hard to read
console.log(students);

// ✅ Beautiful table format
console.table(students);

// ✅ Show only specific columns
console.table(students, ["name", "grade"]);
```

---

#### Using Array.isArray for Type Checking

```js
function processData(data) {
  // ❌ Wrong: typeof [] === 'object'
  if (typeof data === "array") {
    // Never executes!
    return data.map((x) => x * 2);
  }

  // ✅ Correct: Use Array.isArray
  if (Array.isArray(data)) {
    return data.map((x) => x * 2);
  }

  return "Not an array";
}

console.log(processData([1, 2, 3])); // [2, 4, 6] ✅
console.log(processData("hello")); // 'Not an array' ✅
```

---

#### Using Destructuring for Debugging

```js
const response = {
  status: 200,
  data: {
    user: {
      id: 1,
      name: "Alice",
      email: "alice@example.com",
    },
    token: "abc123",
  },
  timestamp: Date.now(),
};

// ❌ Verbose debugging
console.log("Status:", response.status);
console.log("User:", response.data.user);
console.log("Token:", response.data.token);

// ✅ Destructuring for cleaner logs
const {
  status,
  data: { user, token },
} = response;

console.log({ status, user, token });

// ✅ With renaming
const {
  data: { user: userData },
} = response;
console.log("User data:", userData);

// ✅ Quick object inspection
const debug = { status, userName: user.name, token };
console.table(debug);
```

---

### Exercise 12: Real-World Debugging Scenario

```js
// Bug-ridden shopping cart implementation
const cart = [];

function addItem(item) {
  cart.push(item);
}

function removeItem(itemId) {
  for (var i = 0; i < cart.length; i++) {
    if (cart[i].id == itemId) {
      // ❌ Using == instead of ===
      cart.splice(i, 1);
    }
  }
}

function getTotal() {
  let total = 0;
  for (var i = 0; i < cart.length; i++) {
    total = total + cart[i].price * cart[i].quantity; // ❌ What if price is a string?
  }
  return total;
}

// Test the buggy code
addItem({ id: 1, name: "Laptop", price: "1200", quantity: 1 });
addItem({ id: 2, name: "Mouse", price: "25", quantity: 2 });
addItem({ id: "2", name: "Keyboard", price: 75, quantity: 1 }); // String id!

console.log("Cart:", cart);
console.log("Total:", getTotal());

removeItem("1"); // String instead of number
console.log("After removal:", cart);
console.log("New total:", getTotal());
```

<details>
<summary>🐛 Issues Found</summary>

1. **Type coercion with `==`**: `removeItem` will match "1" === 1
2. **String arithmetic**: `"1200" * 1` works but is fragile
3. **No input validation**: Accepts string prices
4. **Global cart state**: Hard to test and reason about
5. **Using `var`**: Should use `const`/`let`

</details>

<details>
<summary>💡 Show Fixed Version</summary>

```js
// Fixed shopping cart implementation
class ShoppingCart {
  constructor() {
    this.items = [];
  }

  addItem(item) {
    // Validate input
    if (!item || typeof item.id === "undefined") {
      throw new Error("Item must have an id");
    }

    const { id, name, price, quantity } = item;

    // Validate types
    if (typeof price !== "number" || price < 0) {
      throw new Error("Price must be a positive number");
    }

    if (typeof quantity !== "number" || quantity < 1) {
      throw new Error("Quantity must be a positive number");
    }

    this.items.push({ id, name, price, quantity });
  }

  removeItem(itemId) {
    const index = this.items.findIndex((item) => item.id === itemId);

    if (index === -1) {
      console.warn(`Item with id ${itemId} not found`);
      return false;
    }

    this.items.splice(index, 1);
    return true;
  }

  getTotal() {
    return this.items.reduce((total, item) => {
      return total + item.price * item.quantity;
    }, 0);
  }

  getItems() {
    return [...this.items]; // Return copy to prevent mutation
  }

  debug() {
    console.table(this.items);
    console.log(`Total: $${this.getTotal().toFixed(2)}`);
  }
}

// Test the fixed version
const cart = new ShoppingCart();

try {
  cart.addItem({ id: 1, name: "Laptop", price: 1200, quantity: 1 });
  cart.addItem({ id: 2, name: "Mouse", price: 25, quantity: 2 });
  cart.addItem({ id: 3, name: "Keyboard", price: 75, quantity: 1 });

  cart.debug();

  cart.removeItem(1);
  console.log("\nAfter removing item 1:");
  cart.debug();

  // Try invalid operations
  cart.addItem({ id: 4, name: "Invalid", price: "100", quantity: 1 });
} catch (error) {
  console.error("Error:", error.message);
}
```

</details>

---

## 🎯 Rapid Fire Challenges

Predict the output (10 seconds each!):

```js
1. [1, 2, 3].map(x => x * 2)
2. [1, 2, 3].filter(x => x > 2)
3. [1, 2, 3].reduce((a, b) => a + b)
4. [1, 2, 3].reduce((a, b) => a + b, 10)
5. [1, 2, 3].includes(2)
6. [1, 2, 3].find(x => x > 1)
7. [1, 2, 3].findIndex(x => x > 1)
8. [1, 2, 3].some(x => x > 2)
9. [1, 2, 3].every(x => x > 0)
10. [...[1, 2], ...[3, 4]]
11. { a: 1, ...{ b: 2 } }
12. const { a, ...rest } = { a: 1, b: 2, c: 3 }
13. [1, 2, 3][10]
14. Array(3).fill(0)
15. Array.from({ length: 3 }, (_, i) => i)
16. [1, [2, [3]]].flat()
17. [1, [2, [3]]].flat(2)
18. [1, 2, 3].flatMap(x => [x, x * 2])
19. Object.keys({ a: 1, b: 2 })
20. Object.values({ a: 1, b: 2 })
```

<details>
<summary>💡 Show All Answers</summary>

```js
1. [2, 4, 6]
2. [3]
3. 6
4. 16
5. true
6. 2
7. 1
8. true
9. true
10. [1, 2, 3, 4]
11. { a: 1, b: 2 }
12. rest = { b: 2, c: 3 }
13. undefined
14. [0, 0, 0]
15. [0, 1, 2]
16. [1, 2, [3]]
17. [1, 2, 3]
18. [1, 2, 2, 4, 3, 6]
19. ['a', 'b']
20. [1, 2]
```

</details>
