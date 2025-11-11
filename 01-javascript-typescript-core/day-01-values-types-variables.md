# Day 1 — Values, Types, Variables

> **Learning Objective:** Master JavaScript's type system, understand coercion mechanics, and write modern variable declarations with proper scoping.

---

## 📚 Part 1: Primitive Types (Deep Dive)

### 1.1 The Seven Primitives

JavaScript has **7 primitive types**. Primitives are:

- **Immutable** — cannot be changed, only replaced
- **Stored by value** — copied when assigned
- **Compared by value** — equality checks the actual content

#### `undefined`

**Definition:** A variable that has been declared but not assigned a value.

```js
let name;
console.log(name); // undefined
console.log(typeof name); // 'undefined'

// Function with no return
function noReturn() {}
console.log(noReturn()); // undefined

// Missing object property
const user = { age: 25 };
console.log(user.name); // undefined

// Missing array element
const arr = [1, , 3];
console.log(arr[1]); // undefined
```

**Common Patterns:**

```js
// Check if variable is undefined
if (value === undefined) {
}
if (typeof value === "undefined") {
} // Safer, won't throw if not declared

// Default parameter (pre-ES6)
function greet(name) {
  name = name === undefined ? "Guest" : name;
}

// Modern default parameter
function greet(name = "Guest") {}
```

---

#### `null`

**Definition:** Intentional absence of any object value. Represents "nothing", "empty", or "value unknown".

```js
let emptyValue = null;
console.log(typeof null); // 'object' ⚠️ JavaScript bug from 1995

// Compare with undefined
console.log(null == undefined); // true (loose equality)
console.log(null === undefined); // false (different types)

// Common uses
let user = null; // No user logged in
const result = findUser("id"); // Returns null if not found
document.getElementById("fake"); // Returns null if element doesn't exist
```

**Best Practices:**

```js
// ✅ Explicit null check
if (value === null) {
}

// ✅ Check for null or undefined
if (value == null) {
} // Catches both null and undefined

// ❌ Avoid typeof for null
if (typeof value === "null") {
} // Never true!
```

---

#### `boolean`

**Definition:** Logical entity with two values: `true` or `false`.

```js
const isActive = true;
const isDeleted = false;

// Boolean() constructor
Boolean(1); // true
Boolean(0); // false
Boolean("hello"); // true
Boolean(""); // false

// Logical operators return booleans
console.log(5 > 3); // true
console.log("a" === "b"); // false
console.log(!false); // true
```

**Falsy Values (8 total):**

```js
Boolean(false); // false
Boolean(0); // false
Boolean(-0); // false
Boolean(0n); // false (BigInt zero)
Boolean(""); // false (empty string)
Boolean(null); // false
Boolean(undefined); // false
Boolean(NaN); // false

// Everything else is truthy!
Boolean([]); // true
Boolean({}); // true
Boolean("0"); // true ⚠️ (string with zero)
Boolean("false"); // true ⚠️ (string)
Boolean(new Date()); // true
Boolean(-1); // true
Boolean(Infinity); // true
```

**Double Negation (!! operator):**

```js
// Convert any value to boolean
!!1; // true
!!0; // false
!!""; // false
!!"hello"; // true

// Use in conditions
const hasData = !!data.length;
```

---

#### `number`

**Definition:** 64-bit IEEE 754 double-precision floating-point format. Represents integers and decimals.

```js
const integer = 42;
const float = 3.14159;
const negative = -273.15;
const scientific = 3e8; // 300000000 (3 × 10^8)

// Special numeric values
const infinite = Infinity;
const negInfinite = -Infinity;
const notANumber = NaN; // "Not a Number"

// Number properties
console.log(Number.MAX_VALUE); // 1.7976931348623157e+308
console.log(Number.MIN_VALUE); // 5e-324
console.log(Number.MAX_SAFE_INTEGER); // 9007199254740991 (2^53 - 1)
console.log(Number.MIN_SAFE_INTEGER); // -9007199254740991
console.log(Number.POSITIVE_INFINITY); // Infinity
console.log(Number.NEGATIVE_INFINITY); // -Infinity
console.log(Number.NaN); // NaN
```

**NaN (Not-a-Number) Quirks:**

```js
// Operations that produce NaN
console.log(0 / 0); // NaN
console.log(Math.sqrt(-1)); // NaN
console.log(parseInt("abc")); // NaN
console.log("hello" * 2); // NaN

// NaN is the only value not equal to itself!
console.log(NaN === NaN); // false ⚠️
console.log(NaN == NaN); // false ⚠️

// How to check for NaN
console.log(isNaN(NaN)); // true
console.log(Number.isNaN(NaN)); // true (better)
console.log(Number.isNaN("hello")); // false (doesn't coerce)
console.log(isNaN("hello")); // true ⚠️ (coerces, then checks)
```

**Floating-Point Precision Issues:**

```js
console.log(0.1 + 0.2); // 0.30000000000000004 ⚠️
console.log(0.1 + 0.2 === 0.3); // false ⚠️

// Solution: use epsilon comparison
const epsilon = Number.EPSILON; // 2.220446049250313e-16
const a = 0.1 + 0.2;
const b = 0.3;
console.log(Math.abs(a - b) < epsilon); // true

// Or use integer arithmetic
const cents1 = 10; // $0.10
const cents2 = 20; // $0.20
const total = (cents1 + cents2) / 100; // $0.30 ✅
```

**Number Conversion:**

```js
// Explicit conversion
Number("123"); // 123
Number("12.5"); // 12.5
Number("12px"); // NaN
Number(true); // 1
Number(false); // 0
Number(null); // 0
Number(undefined); // NaN

// Unary plus (+)
+"123"; // 123
+true; // 1
+[]; // 0
+{}; // NaN

// parseInt and parseFloat
parseInt("123px"); // 123 (stops at non-digit)
parseFloat("12.5em"); // 12.5
parseInt("0x10"); // 16 (hex)
parseInt("08", 10); // 8 (always specify radix!)
```

---

#### `bigint`

**Definition:** Represents integers larger than 2^53 - 1 (Number.MAX_SAFE_INTEGER).

```js
// Creating BigInts
const big1 = 1234567890123456789012345678901234567890n; // n suffix
const big2 = BigInt("1234567890123456789012345678901234567890");
const big3 = BigInt(9007199254740991);

// Operations
console.log(10n + 20n); // 30n
console.log(50n - 30n); // 20n
console.log(5n * 2n); // 10n
console.log(10n / 3n); // 3n (rounds toward zero)
console.log(10n % 3n); // 1n
console.log(2n ** 100n); // Very large number

// Cannot mix with Number
console.log(10n + 5); // TypeError ⚠️
console.log(10n + BigInt(5)); // 15n ✅

// Comparisons work
console.log(10n > 5); // true
console.log(10n === 10); // false (different types)
console.log(10n == 10); // true (coercion)

// typeof
console.log(typeof 42n); // 'bigint'
```

**Use Cases:**

```js
// Cryptography (large keys)
const key =
  179769313486231590772930519078902473361797697894230657273430081157732675805500963132708477322407536021120113879871393357658789768814416622492847430639474124377767893424865485276302219601246094119453082952085005768838150682342462881473913110540827237163350510684586298239947245938479716304835356329624224137859n;

// Financial calculations (avoid floating-point)
const dollars = 123456789012345n; // $1,234,567,890,123.45
const cents = dollars * 100n;

// Timestamps beyond 2038 problem
const futureTime = BigInt(Date.now()) + 100000000000n;
```

---

#### `string`

**Definition:** Sequence of characters (UTF-16 encoded text).

```js
// Three ways to create strings
const single = "Hello";
const double = "World";
const template = `Hello ${name}`;

// String properties
console.log("hello".length); // 5

// Immutability
let str = "hello";
str[0] = "H"; // Does nothing (silent fail)
console.log(str); // 'hello'
str = "Hello"; // Reassignment creates new string ✅

// Common methods
"Hello".toUpperCase(); // 'HELLO'
"Hello".toLowerCase(); // 'hello'
"Hello World".includes("World"); // true
"Hello World".indexOf("World"); // 6
"Hello World".slice(0, 5); // 'Hello'
"Hello World".split(" "); // ['Hello', 'World']
"  trim me  ".trim(); // 'trim me'
"repeat".repeat(3); // 'repeatrepeatrepeat'
"abc".charAt(1); // 'b'
"abc".charCodeAt(1); // 98 (UTF-16 code)
```

**Template Literals (Backticks):**

```js
const name = "Alice";
const age = 30;

// String interpolation
const greeting = `Hello, ${name}!`;
const message = `You are ${age} years old`;

// Multi-line strings
const multiline = `
  Line 1
  Line 2
  Line 3
`;

// Expression evaluation
const calc = `2 + 2 = ${2 + 2}`; // '2 + 2 = 4'
const nested = `${name.toUpperCase()}`; // 'ALICE'

// Tagged templates (advanced)
function tag(strings, ...values) {
  console.log(strings); // Array of string literals
  console.log(values); // Array of interpolated values
}
tag`Hello ${name}, you are ${age}!`;
```

**String Coercion:**

```js
// Implicit (concatenation with +)
"The answer is " + 42; // 'The answer is 42'
"5" + 3; // '53' ⚠️
"10" + "20"; // '1020' ⚠️

// Explicit
String(42); // '42'
String(true); // 'true'
String(null); // 'null'
String(undefined); // 'undefined'
String([1, 2, 3]); // '1,2,3'
String({ a: 1 }); // '[object Object]'

// toString() method
(42).toString(); // '42'
true.toString(); // 'true'
[1, 2].toString(); // '1,2'
```

**Escape Sequences:**

```js
const quote = 'He said, "Hello"'; // Mix quotes
const escaped = 'He said, "Hello"'; // Escape
const newline = "Line 1\nLine 2"; // \n = newline
const tab = "Col1\tCol2"; // \t = tab
const backslash = "C:\\Users\\file"; // \\ = backslash
const unicode = "\u0048\u0065\u006C\u006C\u006F"; // 'Hello'
```

---

#### `symbol`

**Definition:** Unique, immutable primitive used as object property keys.

```js
// Creating symbols
const sym1 = Symbol();
const sym2 = Symbol("description"); // Optional description
const sym3 = Symbol("description");

// Symbols are always unique
console.log(sym2 === sym3); // false ⚠️

// Use as object keys
const obj = {
  [sym2]: "value",
  normalKey: "normal",
};
console.log(obj[sym2]); // 'value'
console.log(obj.sym2); // undefined ⚠️

// Symbols are hidden from iteration
console.log(Object.keys(obj)); // ['normalKey']
console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(description)]

// Well-known symbols (built-in)
Symbol.iterator; // Used in for...of
Symbol.toStringTag; // Customize [object Type]
Symbol.hasInstance; // instanceof behavior
```

**Global Symbol Registry:**

```js
// Create global symbols
const globalSym1 = Symbol.for("app.id");
const globalSym2 = Symbol.for("app.id");

console.log(globalSym1 === globalSym2); // true ✅ (same key)

// Retrieve key
console.log(Symbol.keyFor(globalSym1)); // 'app.id'
console.log(Symbol.keyFor(sym2)); // undefined (local symbol)
```

**Use Cases:**

```js
// Private-like properties
const _privateMethod = Symbol("private");
class MyClass {
  [_privateMethod]() {
    return "Hidden behavior";
  }
}

// Avoid naming collisions
const STATUS = {
  LOADING: Symbol("loading"),
  SUCCESS: Symbol("success"),
  ERROR: Symbol("error"),
};

// Custom iterators
const range = {
  from: 1,
  to: 5,
  [Symbol.iterator]() {
    let current = this.from;
    const last = this.to;
    return {
      next() {
        if (current <= last) {
          return { done: false, value: current++ };
        } else {
          return { done: true };
        }
      },
    };
  },
};
console.log([...range]); // [1, 2, 3, 4, 5]
```

---

## 🔄 Part 2: Type Coercion (The Complete Guide)

### 2.1 Implicit vs Explicit Coercion

**Explicit Coercion:** You manually convert types.

```js
String(123); // '123'
Number("456"); // 456
Boolean(0); // false
```

**Implicit Coercion:** JavaScript converts types automatically.

```js
"5" + 3; // '53' (number → string)
"10" - 5; // 5 (string → number)
if ("hello") {
} // truthy check (string → boolean)
```

---

### 2.2 String Coercion

**Rule:** When `+` operator has a string operand, everything converts to string.

```js
// Number to string
"Hello" + 42; // 'Hello42'
42 + " bottles"; // '42 bottles'
"5" + 3; // '53' ⚠️
"10" + "20"; // '1020' ⚠️

// Boolean to string
"The answer is " + true; // 'The answer is true'
false + " statement"; // 'false statement'

// Null/undefined to string
"Value: " + null; // 'Value: null'
"Value: " + undefined; // 'Value: undefined'

// Object to string (ToPrimitive)
"Array: " + [1, 2, 3]; // 'Array: 1,2,3'
"Object: " + { a: 1 }; // 'Object: [object Object]'
"Object: " +
  {
    toString() {
      return "custom";
    },
  }; // 'Object: custom'
```

**ToPrimitive Algorithm for Objects:**

```js
const obj = {
  valueOf() {
    return 42;
  },
  toString() {
    return "forty-two";
  },
};

console.log(String(obj)); // 'forty-two' (toString called)
console.log(Number(obj)); // 42 (valueOf called)
console.log(obj + ""); // 'forty-two' (toString for string context)
console.log(obj + 0); // 42 (valueOf for numeric context)
```

---

### 2.3 Number Coercion

**Rule:** Arithmetic operators (except `+`) convert operands to numbers.

```js
// String to number
"10" - 5; // 5
"10" * "2"; // 20
"20" / "4"; // 5
"10" % 3; // 1

// Unary plus
+"42"; // 42
+"3.14"; // 3.14
+"-99"; // -99
+"hello"; // NaN ⚠️

// Boolean to number
true + 1; // 2 (true → 1)
false + 1; // 1 (false → 0)
Number(true); // 1
Number(false); // 0

// Null to number
null + 5; // 5 (null → 0)
Number(null); // 0

// Undefined to number
undefined + 5; // NaN (undefined → NaN)
Number(undefined); // NaN

// Array to number
+[]; // 0 (empty array)
+[5]; // 5 (single element)
+[1, 2]; // NaN (multiple elements)

// Object to number
+{}; // NaN
+{
  valueOf() {
    return 100;
  },
}; // 100
```

---

### 2.4 Boolean Coercion

**Rule:** Falsy values become `false`, everything else becomes `true`.

**The 8 Falsy Values:**

```js
Boolean(false); // false
Boolean(0); // false
Boolean(-0); // false
Boolean(0n); // false
Boolean(""); // false
Boolean(null); // false
Boolean(undefined); // false
Boolean(NaN); // false
```

**Everything Else is Truthy:**

```js
Boolean(true); // true
Boolean(1); // true
Boolean(-1); // true
Boolean(" "); // true (space is not empty)
Boolean("0"); // true ⚠️ (string)
Boolean("false"); // true ⚠️ (string)
Boolean([]); // true ⚠️ (empty array)
Boolean({}); // true ⚠️ (empty object)
Boolean(function () {}); // true
Boolean(Infinity); // true
```

**Logical Operators:**

```js
// && returns first falsy value or last value
console.log(0 && "hello"); // 0
console.log("hello" && "world"); // 'world'
console.log("a" && "b" && "c"); // 'c'
console.log("a" && false && "c"); // false

// || returns first truthy value or last value
console.log(0 || "hello"); // 'hello'
console.log("" || "default"); // 'default'
console.log("first" || "second"); // 'first'
console.log(null || undefined || 0); // 0 (last value)

// Nullish coalescing (??) - ES2020
console.log(null ?? "default"); // 'default'
console.log(undefined ?? "default"); // 'default'
console.log(0 ?? "default"); // 0 ⚠️ (0 is not nullish)
console.log("" ?? "default"); // '' ⚠️ ('' is not nullish)
```

---

### 2.5 Equality Coercion (== vs ===)

#### Strict Equality (===)

**Rule:** No type conversion. Both type and value must be identical.

```js
// Same type and value
5 === 5; // true
"hello" === "hello"; // true
true === true; // true

// Different types
5 === "5"; // false
true === 1; // false
null === undefined; // false

// Special cases
NaN === NaN; // false ⚠️ (use Number.isNaN)
+0 === -0; // true ⚠️ (use Object.is for distinction)

// Object comparison (by reference)
const obj1 = { a: 1 };
const obj2 = { a: 1 };
const obj3 = obj1;

console.log(obj1 === obj2); // false (different objects)
console.log(obj1 === obj3); // true (same reference)
```

#### Abstract Equality (==)

**Rule:** Performs type coercion before comparison.

**Coercion Algorithm:**

1. If types are the same, use `===`
2. If `null == undefined`, return `true`
3. If comparing string and number, convert string to number
4. If comparing boolean, convert to number
5. If comparing object and primitive, convert object to primitive

```js
// Number and string
5 == "5"; // true ('5' → 5)
0 == ""; // true ('' → 0)
0 == "0"; // true ('0' → 0)

// Boolean coercion
true == 1; // true (true → 1)
false == 0; // true (false → 0)
"1" == true; // true ('1' → 1, true → 1)
"0" == false; // true ('0' → 0, false → 0)

// Null and undefined
null == undefined; // true (special case)
null == 0; // false (null only equals undefined)
undefined == 0; // false

// Object coercion
[1] == 1; // true ([1] → '1' → 1)
[1, 2] == "1,2"; // true ([1,2] → '1,2')
[] == 0; // true ([] → '' → 0)
[] == false; // true ([] → '' → 0, false → 0)

// Mind-bending
[] == ![]; // true
// Step 1: ![] → false (array is truthy)
// Step 2: [] == false
// Step 3: [] → '' (ToPrimitive)
// Step 4: '' == false
// Step 5: '' → 0, false → 0
// Step 6: 0 == 0 → true
```

**Equality Comparison Table:**

| Comparison          | ==      | ===     |
| ------------------- | ------- | ------- |
| `5 == '5'`          | `true`  | `false` |
| `0 == false`        | `true`  | `false` |
| `null == undefined` | `true`  | `false` |
| `'' == 0`           | `true`  | `false` |
| `[] == false`       | `true`  | `false` |
| `'0' == false`      | `true`  | `false` |
| `NaN == NaN`        | `false` | `false` |

**Best Practice:**

```js
// ✅ Always use === unless you specifically need coercion
if (value === 42) {
}

// ✅ Exception: null/undefined check
if (value == null) {
} // Catches both null and undefined

// ❌ Avoid == in most cases
if (value == 42) {
} // Unpredictable with coercion
```

---

### 2.6 Advanced Coercion Patterns

#### Array and Object Coercion

```js
// Arrays
String([]); // ''
String([1, 2, 3]); // '1,2,3'
Number([]); // 0
Number([5]); // 5
Number([1, 2]); // NaN

// Objects
String({}); // '[object Object]'
Number({}); // NaN
Boolean({}); // true

// Custom toString/valueOf
const custom = {
  toString() {
    return "42";
  },
  valueOf() {
    return 100;
  },
};

console.log(String(custom)); // '42' (toString)
console.log(Number(custom)); // 100 (valueOf)
console.log(custom + 0); // 100 (valueOf for numeric)
console.log(custom + ""); // '42' (toString for string)
```

#### Comparison Operators

```js
// Relational comparison (<, >, <=, >=)
'2' > 1;                 // true ('2' → 2)
'10' < '9';              // true ⚠️ (lexicographic: '1' < '9')
'10' < 9;                // false ('10' → 10)

// String comparison
'apple' < 'banana';      // true (lexicographic)
'Apple' < 'apple';       // true (uppercase comes first)

// Multiple types
[5] > 4;                 // true ([5] → '5' → 5)
{ valueOf() { return 10; } } > 5;  // true (custom valueOf)
```

---

## 🎯 Part 3: Variable Declarations (var, let, const)

### 3.1 The Problems with `var`

**Problem 1: Function Scope (No Block Scope)**

```js
// var ignores block scope
if (true) {
  var x = 10;
}
console.log(x); // 10 ⚠️ (accessible outside block)

for (var i = 0; i < 3; i++) {
  // Loop code
}
console.log(i); // 3 ⚠️ (i leaked outside loop)

// Compare with let
if (true) {
  let y = 20;
}
console.log(y); // ReferenceError: y is not defined ✅
```

**Problem 2: Re-declaration Allowed**

```js
var count = 5;
var count = 10; // No error ⚠️
console.log(count); // 10

// Compare with let
let count2 = 5;
let count2 = 10; // SyntaxError: Identifier 'count2' has already been declared ✅
```

**Problem 3: Hoisting with Initialization**

```js
console.log(name); // undefined (not ReferenceError) ⚠️
var name = "Alice";

// Equivalent to:
var name; // Hoisted to top
console.log(name); // undefined
name = "Alice";
```

**Problem 4: Global Object Property**

```js
var globalVar = "hello";
console.log(window.globalVar); // 'hello' (browser) ⚠️

let globalLet = "world";
console.log(window.globalLet); // undefined ✅
```

**Problem 5: The Classic Loop Closure Bug**

```js
// Using var
for (var i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i); // Prints: 3, 3, 3 ⚠️
  }, 100);
}
// All callbacks share the same 'i' (function-scoped)

// Using let
for (let i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i); // Prints: 0, 1, 2 ✅
  }, 100);
}
// Each iteration has its own 'i' (block-scoped)

// Old solution with var (IIFE)
for (var i = 0; i < 3; i++) {
  (function (index) {
    setTimeout(function () {
      console.log(index); // Prints: 0, 1, 2
    }, 100);
  })(i);
}
```

---

### 3.2 Modern Declarations: `let` and `const`

#### `let` — Block-Scoped, Reassignable

```js
// Block scope
{
  let x = 10;
  console.log(x); // 10
}
console.log(x); // ReferenceError ✅

// Reassignable
let count = 0;
count = 1; // ✅ Works
count++; // ✅ Works

// No re-declaration
let name = "Alice";
let name = "Bob"; // SyntaxError ✅

// Loop scope
for (let i = 0; i < 3; i++) {
  console.log(i); // 0, 1, 2
}
console.log(i); // ReferenceError ✅

// Temporal Dead Zone
console.log(value); // ReferenceError (TDZ)
let value = 5;
```

**When to Use `let`:**

- Loop counters
- Values that change over time
- Accumulator variables
- Conditional reassignment

```js
// Good use cases for let
for (let i = 0; i < 10; i++) {}

let total = 0;
for (const item of items) {
  total += item.price;
}

let status;
if (condition) {
  status = "active";
} else {
  status = "inactive";
}
```

---

#### `const` — Block-Scoped, Immutable Binding

```js
// Must be initialized
const PI = 3.14159;

// Cannot reassign
PI = 3.14; // TypeError: Assignment to constant variable ✅

// Block scoped
{
  const temp = 100;
}
console.log(temp); // ReferenceError ✅

// No re-declaration
const name = "Alice";
const name = "Bob"; // SyntaxError ✅
```

**Important: Immutable Binding, NOT Immutable Value**

```js
// Primitives are immutable
const num = 42;
num = 43; // TypeError ✅

// Objects: binding is immutable, content is mutable
const person = {
  name: "Alice",
  age: 30,
};

person.age = 31; // ✅ Works (mutating property)
person.city = "NYC"; // ✅ Works (adding property)
person = { name: "Bob" }; // ❌ TypeError (reassigning)

// Arrays: binding is immutable, content is mutable
const numbers = [1, 2, 3];

numbers.push(4); // ✅ Works [1, 2, 3, 4]
numbers[0] = 99; // ✅ Works [99, 2, 3, 4]
numbers = [5, 6]; // ❌ TypeError (reassigning)

// To make object truly immutable
const frozen = Object.freeze({ name: "Alice" });
frozen.name = "Bob"; // Silent fail (strict mode: TypeError)
console.log(frozen.name); // 'Alice'

// Deep freeze (nested objects)
function deepFreeze(obj) {
  Object.freeze(obj);
  Object.values(obj).forEach((value) => {
    if (typeof value === "object" && value !== null) {
      deepFreeze(value);
    }
  });
  return obj;
}
```

**When to Use `const`:**

- Default choice for all variables
- Function declarations (arrow functions)
- Import statements
- Configuration values
- Object/array references that won't be reassigned

```js
// Good use cases for const
const MAX_USERS = 100;
const API_URL = "https://api.example.com";

const fetchData = async () => {};

const config = {
  timeout: 5000,
  retries: 3,
};

import { Component } from "react"; // Always const internally
```

---

### 3.3 Scope Comparison Table

| Feature             | `var`               | `let`            | `const`          |
| ------------------- | ------------------- | ---------------- | ---------------- |
| **Scope**           | Function            | Block            | Block            |
| **Reassignment**    | ✅ Yes              | ✅ Yes           | ❌ No            |
| **Re-declaration**  | ✅ Yes              | ❌ No            | ❌ No            |
| **Hoisting**        | ✅ (as `undefined`) | ✅ (TDZ)         | ✅ (TDZ)         |
| **TDZ**             | ❌ No               | ✅ Yes           | ✅ Yes           |
| **Global property** | ✅ Yes              | ❌ No            | ❌ No            |
| **Loop scope**      | ❌ Shared           | ✅ Per iteration | ✅ Per iteration |
| **Must initialize** | ❌ No               | ❌ No            | ✅ Yes           |

---

### 3.4 Scoping Examples

#### Block Scope

```js
// Block scope with let/const
function blockScopeExample() {
  if (true) {
    let blockVar = "I am block-scoped";
    const blockConst = "Me too";
    var funcVar = "I am function-scoped";
  }

  console.log(funcVar); // 'I am function-scoped'
  console.log(blockVar); // ReferenceError
  console.log(blockConst); // ReferenceError
}

// Nested blocks
{
  const outer = "outer";
  {
    const inner = "inner";
    console.log(outer); // 'outer' (accessible)
    console.log(inner); // 'inner'
  }
  console.log(inner); // ReferenceError
}

// Switch statements
switch (value) {
  case "a":
    let x = 1; // Block-scoped to case
    break;
  case "b":
    let x = 2; // SyntaxError: x already declared ⚠️
    break;
}

// Fix: wrap cases in blocks
switch (value) {
  case "a": {
    let x = 1;
    break;
  }
  case "b": {
    let x = 2; // ✅ Different scope
    break;
  }
}
```

#### Function Scope

```js
// var is function-scoped
function functionScope() {
  if (true) {
    var x = 10;
  }
  console.log(x); // 10 (accessible)
}

// Nested functions
function outer() {
  var outerVar = "outer";

  function inner() {
    var innerVar = "inner";
    console.log(outerVar); // 'outer' (closure)
    console.log(innerVar); // 'inner'
  }

  inner();
  console.log(innerVar); // ReferenceError
}

// Function parameters are function-scoped
function example(param) {
  console.log(param); // Accessible throughout function
  if (true) {
    console.log(param); // Still accessible
  }
}
```

#### Global Scope

```js
// Global variables (avoid!)
var globalVar = "I am global";
let globalLet = "Me too";
const globalConst = "And me";

console.log(window.globalVar); // 'I am global' (browser)
console.log(window.globalLet); // undefined
console.log(window.globalConst); // undefined

// Implicit global (no declaration keyword)
function createGlobal() {
  implicitGlobal = "Oops!"; // ⚠️ Creates global variable
}
createGlobal();
console.log(implicitGlobal); // 'Oops!'

// Prevent with strict mode
("use strict");
function strictMode() {
  undeclared = 5; // ReferenceError ✅
}
```

---

## 🔝 Part 4: Hoisting (Deep Dive)

### 4.1 What is Hoisting?

**Definition:** JavaScript moves all declarations to the top of their scope during the compilation phase (before execution).

**Important:** Only **declarations** are hoisted, not **initializations**.

---

### 4.2 Function Hoisting

**Function Declarations: Fully Hoisted**

```js
// This works!
greet(); // 'Hello, World!'

function greet() {
  console.log("Hello, World!");
}

// Equivalent to:
function greet() {
  console.log("Hello, World!");
}
greet(); // 'Hello, World!'
```

**Function Expressions: NOT Hoisted**

```js
// This fails!
greet(); // TypeError: greet is not a function

var greet = function () {
  console.log("Hello, World!");
};

// Equivalent to:
var greet; // Declaration hoisted
greet(); // TypeError (greet is undefined)
greet = function () {
  // Assignment stays
  console.log("Hello, World!");
};
```

**Arrow Functions: NOT Hoisted**

```js
greet(); // ReferenceError (TDZ if const/let)

const greet = () => {
  console.log("Hello, World!");
};
```

**Comparison:**

```js
// Function declaration
console.log(typeof declaredFunc); // 'function' ✅
function declaredFunc() {}

// Function expression
console.log(typeof expressionFunc); // 'undefined'
var expressionFunc = function () {};

// Arrow function
console.log(typeof arrowFunc); // ReferenceError (TDZ)
const arrowFunc = () => {};
```

---

### 4.3 Variable Hoisting

**`var`: Hoisted and Initialized as `undefined`**

```js
console.log(myVar); // undefined (not ReferenceError)
var myVar = 5;
console.log(myVar); // 5

// Equivalent to:
var myVar; // Hoisted to top
console.log(myVar); // undefined
myVar = 5; // Assignment stays
console.log(myVar); // 5
```

**`let` and `const`: Hoisted but Uninitialized (TDZ)**

```js
console.log(myLet); // ReferenceError: Cannot access 'myLet' before initialization
let myLet = 5;

console.log(myConst); // ReferenceError
const myConst = 10;

// They ARE hoisted, but in Temporal Dead Zone
{
  // TDZ starts here for 'value'
  console.log(value); // ReferenceError

  let value = 10; // TDZ ends here
  console.log(value); // 10
}
```

---

### 4.4 Temporal Dead Zone (TDZ)

**Definition:** The period between entering a scope and the variable's declaration where accessing the variable throws a ReferenceError.

```js
// TDZ Example 1: Basic
{
  // TDZ starts
  console.log(x); // ReferenceError
  console.log(y); // ReferenceError

  let x = 1; // TDZ ends for x
  const y = 2; // TDZ ends for y
}

// TDZ Example 2: typeof
console.log(typeof undeclaredVar); // 'undefined' (no error)
console.log(typeof declaredLater); // ReferenceError (TDZ)
var undeclaredVar;
let declaredLater;

// TDZ Example 3: Class
const myClass = new MyClass(); // ReferenceError (TDZ)
class MyClass {}

// TDZ Example 4: Closure
function outer() {
  return function inner() {
    console.log(x); // ReferenceError (x in TDZ)
  };
}
let x = 10;
outer()();
```

**Why TDZ Exists:**

- Catches errors (accessing uninitialized variables)
- Makes `const` behavior consistent
- Prevents confusion with `undefined`

---

### 4.5 Hoisting Priority

When multiple declarations conflict, this is the priority order:

1. **Function declarations** (highest)
2. **Variable declarations**
3. **Assignments** (not hoisted)

```js
// Example 1: Function wins over var
console.log(typeof foo); // 'function'

var foo = "I am a string";
function foo() {
  return "I am a function";
}

console.log(typeof foo); // 'string' (assignment executed)

// Equivalent to:
function foo() {
  return "I am a function";
}
var foo; // Ignored (function already declared)
console.log(typeof foo); // 'function'
foo = "I am a string";
console.log(typeof foo); // 'string'

// Example 2: Last function declaration wins
console.log(foo()); // 'second'

function foo() {
  return "first";
}

function foo() {
  return "second";
}
```

---

### 4.6 Practical Hoisting Scenarios

**Scenario 1: Variable Shadowing**

```js
var name = "Global";

function test() {
  console.log(name); // undefined ⚠️ (not 'Global')
  var name = "Local";
  console.log(name); // 'Local'
}

// Equivalent to:
var name = "Global";

function test() {
  var name; // Hoisted
  console.log(name); // undefined
  name = "Local";
  console.log(name); // 'Local'
}
```

**Scenario 2: Loop Variable in Closure**

```js
// Problem: var is hoisted to function scope
function createFunctions() {
  var functions = [];
  for (var i = 0; i < 3; i++) {
    functions.push(function () {
      console.log(i);
    });
  }
  return functions;
}

const fns = createFunctions();
fns[0](); // 3 ⚠️
fns[1](); // 3 ⚠️
fns[2](); // 3 ⚠️

// Solution 1: Use let
function createFunctionsFixed() {
  var functions = [];
  for (let i = 0; i < 3; i++) {
    // let creates new binding per iteration
    functions.push(function () {
      console.log(i);
    });
  }
  return functions;
}

// Solution 2: IIFE with var
function createFunctionsIIFE() {
  var functions = [];
  for (var i = 0; i < 3; i++) {
    (function (index) {
      functions.push(function () {
        console.log(index);
      });
    })(i);
  }
  return functions;
}
```

---

## 🛠️ Part 5: Hands-on Exercises

### 5.1 Type Coercion Challenge

**Predict the output (no peeking!):**

```js
// Exercise 1: String Coercion
console.log("5" + 3);
console.log("5" + +"3");
console.log("5" - "3");
console.log("5" * "2");
console.log("10" / "2");

// Exercise 2: Boolean Coercion
console.log(Boolean(""));
console.log(Boolean("0"));
console.log(Boolean([]));
console.log(Boolean({}));
console.log(Boolean(null));

// Exercise 3: Equality
console.log(0 == "0");
console.log(0 === "0");
console.log(null == undefined);
console.log(null === undefined);
console.log("" == 0);

// Exercise 4: Array Coercion
console.log([] + []);
console.log([] + {});
console.log({} + []);
console.log([1, 2] + [3, 4]);
console.log([1] == 1);

// Exercise 5: Unary Plus
console.log(+"42");
console.log(+true);
console.log(+false);
console.log(+null);
console.log(+undefined);
console.log(+[]);
console.log(+[5]);
console.log(+[1, 2]);

// Exercise 6: Logical Operators
console.log("hello" && "world");
console.log(0 && "hello");
console.log("" || "default");
console.log(null || undefined || 0);
console.log(5 && 10 && 15);

// Exercise 7: Mind Benders
console.log([] == ![]);
console.log("" == []);
console.log(0 == []);
console.log("0" == []);
console.log(false == []);

// Exercise 8: Comparisons
console.log("2" > "12");
console.log("2" > 12);
console.log([5] > 4);
console.log([10] < [9]);
```

<details>
<summary>💡 Show Answers</summary>

```js
// Exercise 1: String Coercion
"53"; // String concatenation
"53"; // '5' + (+'3') → '5' + 3 → '53'
2; // Numeric subtraction
10; // Numeric multiplication
5; // Numeric division

// Exercise 2: Boolean Coercion
false; // Empty string is falsy
true; // Non-empty string is truthy
true; // Empty array is truthy
true; // Empty object is truthy
false; // null is falsy

// Exercise 3: Equality
true; // '0' coerced to 0
false; // Different types
true; // Special case
false; // Different types
true; // '' coerced to 0

// Exercise 4: Array Coercion
(""); // [] → '', '' + '' → ''
("[object Object]"); // [] → '', {} → '[object Object]'
("[object Object]"); // or 0 (depends on context)
("1,23,4"); // [1,2] → '1,2', [3,4] → '3,4'
true; // [1] → '1' → 1

// Exercise 5: Unary Plus
42; // String to number
1; // true → 1
0; // false → 0
0; // null → 0
NaN; // undefined → NaN
0; // [] → '' → 0
5; // [5] → '5' → 5
NaN; // [1,2] → '1,2' → NaN

// Exercise 6: Logical Operators
("world"); // Last truthy value
0; // First falsy value
("default"); // First truthy value
0; // Last value (all falsy)
15; // Last truthy value

// Exercise 7: Mind Benders
true; // [] → '' → 0, ![] → false → 0
true; // [] → '', '' == ''
true; // [] → '' → 0
false; // [] → '', '0' != ''
true; // [] → '' → 0, false → 0

// Exercise 8: Comparisons
true; // Lexicographic: '2' > '1'
false; // '2' → 2, 2 > 12 is false
true; // [5] → '5' → 5, 5 > 4
false; // Lexicographic: '10' < '9'
```

</details>

---

### 5.2 Variable Scope Challenge

**Predict the output:**

```js
// Challenge 1
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

// Challenge 2
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

// Challenge 3
const arr = [1, 2, 3];
arr.push(4);
console.log(arr);
arr = [5, 6, 7]; // What happens?

// Challenge 4
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 0);
}

// Challenge 5
console.log(typeof myFunc);
console.log(typeof myVar);
console.log(typeof myLet);

function myFunc() {}
var myVar = "value";
let myLet = "value";
```

<details>
<summary>💡 Show Answers</summary>

```js
// Challenge 1
3; // innermost x
2; // middle x
1; // global x

// Challenge 2
30; // innermost block
20; // middle block
(10)[ // outer scope
  // Challenge 3
  (1, 2, 3, 4)
]; // push works
TypeError; // Cannot reassign const

// Challenge 4
3, 3, 3; // var shares i across iterations
0, 1, 2; // let creates new j per iteration

// Challenge 5
("function"); // Function declaration hoisted
("undefined"); // var hoisted but not initialized
ReferenceError; // let in TDZ
```

</details>

---

### 5.3 Real-World Refactoring

**Refactor this legacy code:**

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

**Your refactored version should use:**

- `const` for values that don't change
- `let` for values that do change
- Template literals for string concatenation
- Arrow functions where appropriate
- Fix the closure bug in the loop

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

</details>

---

## 📋 Part 6: Quick Reference Cheatsheet

### Type Checking Methods

```js
// typeof operator
typeof 42; // 'number'
typeof "hello"; // 'string'
typeof true; // 'boolean'
typeof undefined; // 'undefined'
typeof null; // 'object' ⚠️
typeof Symbol(); // 'symbol'
typeof 123n; // 'bigint'
typeof {}; // 'object'
typeof []; // 'object' ⚠️
typeof function () {}; // 'function'

// Better type checking
Array.isArray([]); // true
Number.isNaN(NaN); // true
Number.isFinite(42); // true
Number.isInteger(42); // true
Object.prototype.toString.call([]); // '[object Array]'
value === null; // Check for null
value === undefined; // Check for undefined
value == null; // Check for null OR undefined
```

### Conversion Functions

```js
// To String
String(value);
value.toString();
value + "";

// To Number
Number(value) + value;
parseInt(value, 10);
parseFloat(value);

// To Boolean
Boolean(value);
!!value;

// To Integer
Math.floor(value);
Math.ceil(value);
Math.round(value);
Math.trunc(value);
parseInt(value, 10);
```

### Falsy Values (Memorize These 8!)

```js
false;
0 - 0;
0n;
("");
null;
undefined;
NaN;
```

---

## ✅ Part 7: Best Practices Summary

### ✅ Do's

1. **Always use `const` by default** — only use `let` when reassignment is needed
2. **Use `===` for equality** — avoid `==` unless you understand coercion
3. **Check for null explicitly** — `if (value === null)` or `if (value == null)`
4. **Use `Number.isNaN()`** — not global `isNaN()` which coerces
5. **Use `Array.isArray()`** — to check for arrays (`typeof []` returns `'object'`)
6. **Specify radix in parseInt** — `parseInt('08', 10)` to avoid octal interpretation
7. **Use strict mode** — `'use strict';` at top of files
8. **Initialize variables** — avoid `undefined` states when possible
9. **Use template literals** — `` `Hello ${name}` `` instead of `'Hello ' + name`
10. **Block-scope variables** — keep variables in smallest scope possible

### ❌ Don'ts

1. **Don't use `var`** — use `let` or `const` instead
2. **Don't use `==`** — use `===` unless you specifically need coercion
3. **Don't rely on `typeof null`** — it returns `'object'` (bug)
4. **Don't assume NaN equals itself** — use `Number.isNaN()`
5. **Don't modify `const` objects thinking they're immutable** — binding is immutable, not content
6. **Don't declare variables without keywords** — implicit globals are bad
7. **Don't compare floating-point directly** — use epsilon for precision issues
8. **Don't access variables in TDZ** — declare before use
9. **Don't shadow variables unnecessarily** — causes confusion
10. **Don't use global variables** — pollutes global namespace

---

## 🔗 Additional Resources

### Official Documentation

- [MDN: JavaScript Data Types](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures)
- [MDN: Hoisting](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting)

### Deep Dives

- [You Don't Know JS: Types & Grammar](https://github.com/getify/You-Dont-Know-JS/tree/2nd-ed/types-grammar)
- [You Don't Know JS: Scope & Closures](https://github.com/getify/You-Dont-Know-JS/tree/2nd-ed/scope-closures)
- [JavaScript.info: Variables](https://javascript.info/variables)
- [JavaScript.info: Type Conversions](https://javascript.info/type-conversions)
