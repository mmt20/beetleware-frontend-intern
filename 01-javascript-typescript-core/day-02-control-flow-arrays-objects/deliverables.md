# Day 2 — Deliverable: Tiny Collection Utilities

> **Implementation:** Pure, immutable, type-safe utility functions for arrays and objects.

---

## 📦 Implementation

```js
// Tiny Collection Utilities
// Pure, immutable, type-safe utility functions for arrays and objects

/**
 * Groups array elements by a key or function result
 * @param {Array} array - Input array
 * @param {string|Function} key - Property name or grouping function
 * @returns {Object} - Grouped object
 * Time: O(n), Space: O(n)
 */
function groupBy(array, key) {
  // Input validation
  if (!Array.isArray(array)) {
    return {};
  }

  return array.reduce((groups, item) => {
    // Determine group key
    const groupKey = typeof key === "function" ? key(item) : item?.[key];

    // Create new group if it doesn't exist, then add item
    return {
      ...groups,
      [groupKey]: [...(groups[groupKey] || []), item],
    };
  }, {});
}

/**
 * Returns unique array elements (primitive-safe, handles NaN)
 * @param {Array} array - Input array
 * @returns {Array} - Array with unique elements
 * Time: O(n), Space: O(n)
 */
function unique(array) {
  // Input validation
  if (!Array.isArray(array)) {
    return [];
  }

  // Set handles NaN correctly (unlike indexOf)
  return [...new Set(array)];
}

/**
 * Deep merge two objects immutably
 * @param {Object} obj1 - Target object
 * @param {Object} obj2 - Source object (overrides obj1)
 * @returns {Object} - New merged object
 * Time: O(n), Space: O(n) where n = total properties
 */
function deepMerge(obj1, obj2) {
  // Helper: Check if value is plain object
  const isObject = (val) => val !== null && typeof val === "object" && !Array.isArray(val);

  // Handle null/undefined inputs
  if (!isObject(obj1)) {
    return isObject(obj2) ? { ...obj2 } : obj2;
  }
  if (!isObject(obj2)) {
    return { ...obj1 };
  }

  // Start with copy of obj1
  const result = { ...obj1 };

  // Iterate over obj2 properties
  for (const key in obj2) {
    if (obj2.hasOwnProperty(key)) {
      // If both values are objects, merge recursively
      if (isObject(obj1[key]) && isObject(obj2[key])) {
        result[key] = deepMerge(obj1[key], obj2[key]);
      } else {
        // Otherwise, obj2 value overrides
        result[key] = obj2[key];
      }
    }
  }

  return result;
}

/**
 * Deep clone any value safely
 * @param {*} value - Value to clone
 * @returns {*} - Cloned value
 * Time: O(n), Space: O(n) where n = total elements/properties
 */
function deepClone(value) {
  // Primitives: return as-is (immutable)
  if (value === null || typeof value !== "object") {
    return value;
  }

  // Handle Date
  if (value instanceof Date) {
    return new Date(value.getTime());
  }

  // Handle RegExp
  if (value instanceof RegExp) {
    return new RegExp(value.source, value.flags);
  }

  // Handle Array
  if (Array.isArray(value)) {
    return value.map((item) => deepClone(item));
  }

  // Handle Set
  if (value instanceof Set) {
    const clonedSet = new Set();
    value.forEach((item) => clonedSet.add(deepClone(item)));
    return clonedSet;
  }

  // Handle Map
  if (value instanceof Map) {
    const clonedMap = new Map();
    value.forEach((val, key) => clonedMap.set(deepClone(key), deepClone(val)));
    return clonedMap;
  }

  // Handle plain Object
  const clonedObj = {};
  for (const key in value) {
    if (value.hasOwnProperty(key)) {
      clonedObj[key] = deepClone(value[key]);
    }
  }

  return clonedObj;
}

/**
 * Extract property values from array of objects
 * @param {Array} array - Array of objects
 * @param {string} key - Property key to extract
 * @returns {Array} - Array of extracted values
 * Time: O(n), Space: O(n)
 */
function pluck(array, key) {
  // Input validation
  if (!Array.isArray(array)) {
    return [];
  }

  // Extract property value from each element
  // Use optional chaining for safe access
  return array.map((item) => item?.[key]);
}

// Export functions (Node.js)
module.exports = { groupBy, unique, deepMerge, deepClone, pluck };

// For browser (uncomment if needed):
// if (typeof window !== 'undefined') {
//   window.TinyUtils = { groupBy, unique, deepMerge, deepClone, pluck };
// }
```

---

## 🚀 Usage Examples

### 1. groupBy

```js
const users = [
  { name: "Alice", role: "admin" },
  { name: "Bob", role: "user" },
  { name: "Charlie", role: "admin" },
];

groupBy(users, "role");
// → {
//     admin: [
//       { name: 'Alice', role: 'admin' },
//       { name: 'Charlie', role: 'admin' }
//     ],
//     user: [{ name: 'Bob', role: 'user' }]
//   }

groupBy(users, (user) => user.name.length);
// → {
//     3: [{ name: 'Bob', role: 'user' }],
//     5: [{ name: 'Alice', role: 'admin' }],
//     7: [{ name: 'Charlie', role: 'admin' }]
//   }
```

---

### 2. unique

```js
unique([1, 2, 2, 3, 1, 4]);
// → [1, 2, 3, 4]

unique(["a", "b", "a", "c"]);
// → ['a', 'b', 'c']

unique([1, "1", 1, true, "true"]);
// → [1, '1', true, 'true']

unique([NaN, NaN, 1, 2]);
// → [NaN, 1, 2]  (NaN handled correctly)
```

---

### 3. deepMerge

```js
const obj1 = {
  name: "Alice",
  settings: {
    theme: "dark",
    notifications: true,
  },
};

const obj2 = {
  age: 25,
  settings: {
    theme: "light",
    language: "en",
  },
};

deepMerge(obj1, obj2);
// → {
//     name: 'Alice',
//     age: 25,
//     settings: {
//       theme: 'light',      // overridden
//       notifications: true,  // preserved
//       language: 'en'       // added
//     }
//   }

// Original objects unchanged
console.log(obj1.settings.theme); // 'dark' ✅
```

---

### 4. deepClone

```js
const original = {
  name: "Alice",
  scores: [90, 85, 92],
  meta: {
    created: new Date(),
    tags: ["student", "active"],
  },
};

const cloned = deepClone(original);

// Deep clone verification
cloned.scores.push(88);
console.log(original.scores); // [90, 85, 92] ✅ unchanged

cloned.meta.tags.push("premium");
console.log(original.meta.tags); // ['student', 'active'] ✅ unchanged
```

---

### 5. pluck

```js
const users = [
  { name: "Alice", age: 25 },
  { name: "Bob", age: 30 },
  { name: "Charlie", age: 35 },
];

pluck(users, "name");
// → ['Alice', 'Bob', 'Charlie']

pluck(users, "age");
// → [25, 30, 35]

// Missing properties
const mixed = [{ name: "Alice" }, { age: 30 }, { name: "Bob", age: 25 }];
pluck(mixed, "name");
// → ['Alice', undefined, 'Bob']
```
