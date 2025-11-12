# Day 1 — Deliverable: Types & Coercion Cheatsheet

> **Quick Reference:** Your go-to guide for JavaScript types, coercion, and variable declarations.

---

## 📊 The 7 Primitive Types

| Type        | Example                                | typeof        | Notes                         |
| ----------- | -------------------------------------- | ------------- | ----------------------------- |
| `undefined` | `undefined`                            | `'undefined'` | Uninitialized variables       |
| `null`      | `null`                                 | `'object'` ⚠️ | Intentional absence (JS bug!) |
| `boolean`   | `true`, `false`                        | `'boolean'`   | Logical values                |
| `number`    | `42`, `3.14`, `NaN`, `Infinity`        | `'number'`    | 64-bit floating point         |
| `bigint`    | `123n`                                 | `'bigint'`    | Arbitrary precision integers  |
| `string`    | `'hello'`, `"world"`, `` `template` `` | `'string'`    | UTF-16 text                   |
| `symbol`    | `Symbol('id')`                         | `'symbol'`    | Unique identifiers            |

---

## 🎯 The 8 Falsy Values (Memorize!)

```js
false;
0 - 0;
0n; // BigInt zero
(""); // Empty string
null;
undefined;
NaN;
```

**Everything else is truthy!** Including:

- `'0'`, `'false'` (non-empty strings)
- `[]`, `{}` (empty arrays/objects)
- `function() {}` (functions)
- All other numbers (including `-1`, `Infinity`)

---

## 🔄 Type Coercion Rules

### String Coercion (+ with string)

```js
'5' + 3           → '53'
42 + ' items'     → '42 items'
'Hello' + true    → 'Hellotrue'
'Value: ' + null  → 'Value: null'
[] + []           → ''
[] + {}           → '[object Object]'
[1,2] + [3,4]     → '1,23,4'
```

**Rule:** With `+`, if either operand is a string, both convert to string.

---

### Number Coercion (-, \*, /, %)

```js
'10' - 5          → 5
'10' * '2'        → 20
'20' / '4'        → 5
'10' % 3          → 1
+'42'             → 42
+true             → 1
+false            → 0
+null             → 0
+undefined        → NaN
+[]               → 0
+[5]              → 5
+[1,2]            → NaN
```

**Rule:** Arithmetic operators (except `+`) convert operands to numbers.

---

### Boolean Coercion

```js
Boolean(0)         → false
Boolean('')        → false
Boolean('0')       → true  ⚠️
Boolean([])        → true  ⚠️
Boolean({})        → true  ⚠️

// Double negation
!!0                → false
!!''               → false
!!'hello'          → true
!![]               → true
```

**Rule:** Only the 8 falsy values become `false`, everything else is `true`.

---

### Logical Operators (Short-Circuit Evaluation)

```js
// && returns first falsy OR last value
0 && 'hello'           → 0
'hello' && 'world'     → 'world'
'a' && 'b' && 'c'      → 'c'
'a' && false && 'c'    → false

// || returns first truthy OR last value
0 || 'hello'           → 'hello'
'' || 'default'        → 'default'
'first' || 'second'    → 'first'
null || undefined || 0 → 0

// ?? returns first defined value (ES2020)
null ?? 'default'      → 'default'
undefined ?? 'default' → 'default'
0 ?? 'default'         → 0  (0 is defined!)
'' ?? 'default'        → '' ('' is defined!)
```

---

## ⚖️ Equality: == vs ===

### Strict Equality (===) - PREFERRED

```js
5 === 5              → true
5 === '5'            → false  (different types)
true === 1           → false  (different types)
null === undefined   → false  (different types)
NaN === NaN          → false  ⚠️ (use Number.isNaN)
[] === []            → false  (different references)
```

**Rule:** No coercion. Both type AND value must match.

---

### Abstract Equality (==) - AVOID

```js
5 == '5'             → true   ('5' → 5)
0 == false           → true   (false → 0)
'' == 0              → true   ('' → 0)
null == undefined    → true   (special case)
[] == 0              → true   ([] → '' → 0)
[] == false          → true   (both → 0)
[] == ![]            → true   🤯
```

**Coercion Algorithm:**

1. Same type? Use `===`
2. `null == undefined`? Return `true`
3. String vs Number? Convert string to number
4. Boolean? Convert to number
5. Object vs Primitive? Convert object

**Exception:** `value == null` catches both `null` and `undefined`:

```js
if (value == null) {
} // Catches both null and undefined ✅
```

---

## 📦 Variable Declarations

### var vs let vs const

| Feature             | `var`               | `let`            | `const`          |
| ------------------- | ------------------- | ---------------- | ---------------- |
| **Scope**           | Function            | Block            | Block            |
| **Reassignment**    | ✅ Yes              | ✅ Yes           | ❌ No            |
| **Re-declaration**  | ✅ Yes              | ❌ No            | ❌ No            |
| **Hoisting**        | ✅ (as `undefined`) | ✅ (TDZ)         | ✅ (TDZ)         |
| **Must Initialize** | ❌ No               | ❌ No            | ✅ Yes           |
| **Global Property** | ✅ Yes (browser)    | ❌ No            | ❌ No            |
| **Loop Scope**      | ❌ Shared           | ✅ Per iteration | ✅ Per iteration |

---

### When to Use What

```js
// ✅ DEFAULT: Use const
const API_URL = "https://api.com";
const user = { name: "Alice" };
const fetchData = async () => {};

// ✅ USE let: When reassignment is needed
let count = 0;
let status = "pending";
for (let i = 0; i < 10; i++) {}

// ❌ AVOID var: Use let/const instead
var oldStyle = "no!"; // Don't do this
```

**Rule of Thumb:**

1. Start with `const`
2. Change to `let` only if you need reassignment
3. Never use `var`

---

### const is NOT Immutable!

```js
// ❌ WRONG: const makes value immutable
const arr = [1, 2, 3];
arr.push(4); // ✅ Works! [1, 2, 3, 4]
arr[0] = 99; // ✅ Works! [99, 2, 3, 4]
arr = [5, 6]; // ❌ TypeError

const obj = { x: 1 };
obj.x = 2; // ✅ Works! { x: 2 }
obj.y = 3; // ✅ Works! { x: 2, y: 3 }
obj = { x: 4 }; // ❌ TypeError

// ✅ RIGHT: const makes binding immutable
// To freeze object content:
const frozen = Object.freeze({ name: "Alice" });
frozen.name = "Bob"; // ❌ Fails silently (strict mode: TypeError)
```

---

## 🔝 Hoisting Quick Reference

### What Gets Hoisted?

```js
// ✅ Function Declarations: Fully hoisted
greet(); // ✅ Works!
function greet() {
  return "Hello";
}

// ❌ Function Expressions: NOT hoisted
greet(); // ❌ TypeError
var greet = function () {
  return "Hello";
};

// ❌ Arrow Functions: NOT hoisted
greet(); // ❌ ReferenceError (TDZ)
const greet = () => "Hello";

// ⚠️ var: Hoisted as undefined
console.log(x); // undefined (not error)
var x = 5;

// ⚠️ let/const: Hoisted but in TDZ
console.log(y); // ❌ ReferenceError
let y = 10;
```

---

### Temporal Dead Zone (TDZ)

```js
{
  // TDZ starts here for x
  console.log(x); // ❌ ReferenceError

  let x = 10; // TDZ ends here
  console.log(x); // ✅ 10
}

// TDZ catches errors:
typeof undeclared; // 'undefined' ✅
typeof declaredLater; // ReferenceError (TDZ) ⚠️
let declaredLater;
```

---

## 🧪 Common Type Checking

```js
// ✅ Recommended Methods
typeof value === "string";
typeof value === "number";
typeof value === "boolean";
typeof value === "function";
typeof value === "undefined";
typeof value === "symbol";
typeof value === "bigint";

Array.isArray(value); // Check for array
Number.isNaN(value); // Check for NaN (better than isNaN)
Number.isFinite(value); // Check for finite number
Number.isInteger(value); // Check for integer
value === null; // Check for null
value === undefined; // Check for undefined
value == null; // Check for null OR undefined

// ❌ Avoid
typeof value === "object"; // Too broad (includes null, arrays)
typeof null === "object"; // True (JS bug!)
isNaN(value); // Coerces value first
```

---

## 🔧 Type Conversion Functions

### To String

```js
String(42)           → '42'
String(true)         → 'true'
String(null)         → 'null'
String(undefined)    → 'undefined'
String([1, 2])       → '1,2'
(42).toString()      → '42'
value + ''           → implicit conversion
```

---

### To Number

```js
Number('42')         → 42
Number('3.14')       → 3.14
Number('42px')       → NaN
Number(true)         → 1
Number(false)        → 0
Number(null)         → 0
Number(undefined)    → NaN
Number([])           → 0
Number([5])          → 5
+'42'                → 42 (unary plus)
parseInt('42px', 10) → 42 (stops at non-digit)
parseFloat('3.14em') → 3.14
```

**Always specify radix in parseInt:**

```js
parseInt('08', 10)   → 8  ✅
parseInt('08')       → 8  (modern)
parseInt('08', 8)    → 0  (octal in older browsers)
```

---

### To Boolean

```js
Boolean(value); // Explicit
!!value; // Double negation
if (value) {
} // Implicit in conditions
```

---

### To Integer

```js
Math.floor(3.7)      → 3   (rounds down)
Math.ceil(3.2)       → 4   (rounds up)
Math.round(3.5)      → 4   (rounds to nearest)
Math.trunc(3.7)      → 3   (removes decimals)
parseInt('3.7', 10)  → 3
~~3.7                → 3   (bitwise NOT NOT)
3.7 | 0              → 3   (bitwise OR)
```

---

## ⚠️ Common Gotchas

### NaN Quirks

```js
NaN === NaN          → false  ⚠️
NaN == NaN           → false  ⚠️

// ✅ Check for NaN:
Number.isNaN(NaN)    → true
Number.isNaN('hello') → false (doesn't coerce)

// ❌ Avoid:
isNaN(NaN)           → true
isNaN('hello')       → true  ⚠️ (coerces then checks)
```

---

### Floating Point Precision

```js
0.1 + 0.2            → 0.30000000000000004  ⚠️
0.1 + 0.2 === 0.3    → false  ⚠️

// ✅ Solutions:
// 1. Use epsilon comparison
Math.abs((0.1 + 0.2) - 0.3) < Number.EPSILON  → true

// 2. Use integer arithmetic
const cents1 = 10;  // $0.10
const cents2 = 20;  // $0.20
(cents1 + cents2) / 100  → 0.3  ✅
```

---

### Array/Object typeof

```js
typeof []            → 'object'  ⚠️
typeof {}            → 'object'  ✅
typeof null          → 'object'  ⚠️ (JS bug)

// ✅ Better checks:
Array.isArray([])    → true
value === null       → true (for null)
```

---

### Loop Variable Closure

```js
// ❌ WRONG (var):
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// Prints: 3, 3, 3

// ✅ RIGHT (let):
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// Prints: 0, 1, 2
```

---

## ✅ Best Practices Checklist

### Do's ✅

- Use `const` by default, `let` when reassignment needed
- Use `===` for equality (not `==`)
- Use `Number.isNaN()` to check for NaN
- Use `Array.isArray()` to check for arrays
- Specify radix in `parseInt('08', 10)`
- Use template literals: `` `Hello ${name}` ``
- Check null/undefined: `value == null` or `value === null`
- Use strict mode: `'use strict';`
- Initialize variables when declaring
- Keep variables in smallest scope

---

### Don'ts ❌

- Don't use `var` (use `let`/`const`)
- Don't use `==` (use `===`)
- Don't rely on `typeof null` returning `'null'`
- Don't assume `NaN === NaN` is true
- Don't think `const` makes objects immutable
- Don't create implicit globals (always use `let`/`const`/`var`)
- Don't compare floats directly (`0.1 + 0.2 === 0.3`)
- Don't access variables before declaration (TDZ)
- Don't use global variables unnecessarily
- Don't use `isNaN()` (use `Number.isNaN()`)

---

## 🎯 Quick Decision Trees

### Which Variable Declaration?

```
Does it need reassignment?
├─ No  → const ✅
└─ Yes → let ✅
(Never var ❌)
```

---

### Which Equality Operator?

```
Do you need type coercion?
├─ No  → === ✅
└─ Yes → Check for null/undefined only?
         ├─ Yes → value == null ✅
         └─ No  → Rethink design (coercion is risky) ⚠️
```

---

### How to Convert Types?

```
Target type?
├─ String   → String(value) or `${value}` ✅
├─ Number   → Number(value) or +value ✅
├─ Boolean  → Boolean(value) or !!value ✅
└─ Integer  → parseInt(value, 10) or Math.floor(value) ✅
```
