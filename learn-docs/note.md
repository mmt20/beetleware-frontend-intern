# Web Development — Quick Reference

> **Fast lookup for common patterns and syntax.** Bookmark this page!

---

## 📚 Table of Contents

1. [HTML5 Cheatsheet](#html5-cheatsheet)
2. [CSS3 Cheatsheet](#css3-cheatsheet)
3. [JavaScript Cheatsheet](#javascript-cheatsheet)
4. [Git Commands](#git-commands)
5. [Performance Tips](#performance-tips)
6. [Accessibility Checklist](#accessibility-checklist)

---

## 📦 HTML5 Cheatsheet

### Document Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Page Title</title>
</head>
<body>
  <header><!-- Site header --></header>
  <nav><!-- Navigation --></nav>
  <main>
    <article><!-- Self-contained content --></article>
    <aside><!-- Sidebar --></aside>
  </main>
  <footer><!-- Site footer --></footer>
</body>
</html>
```

### Common Elements

```html
<!-- Headings -->
<h1>Main Title</h1>
<h2>Section</h2>
<h3>Subsection</h3>

<!-- Text -->
<p>Paragraph</p>
<strong>Bold</strong>
<em>Italic</em>
<mark>Highlighted</mark>

<!-- Links -->
<a href="url">Link</a>
<a href="mailto:email">Email</a>
<a href="tel:+1234567890">Phone</a>

<!-- Lists -->
<ul><li>Unordered</li></ul>
<ol><li>Ordered</li></ol>

<!-- Images -->
<img src="image.jpg" alt="Description">

<!-- Forms -->
<form action="/submit" method="POST">
  <label for="name">Name:</label>
  <input type="text" id="name" name="name" required>
  <button type="submit">Submit</button>
</form>
```

### Input Types

```html
<input type="text">
<input type="email">
<input type="password">
<input type="number">
<input type="date">
<input type="checkbox">
<input type="radio">
<input type="file">
<input type="color">
<textarea></textarea>
<select><option>Option</option></select>
```

---

## 🎨 CSS3 Cheatsheet

### Selectors

```css
/* Element */
p { }

/* Class */
.class-name { }

/* ID */
#id-name { }

/* Attribute */
[type="text"] { }

/* Pseudo-class */
a:hover { }
li:first-child { }
li:nth-child(2) { }

/* Pseudo-element */
p::before { }
p::after { }

/* Combinators */
div p { }        /* Descendant */
div > p { }      /* Child */
div + p { }      /* Adjacent sibling */
div ~ p { }      /* General sibling */
```

### Flexbox

```css
/* Container */
.container {
  display: flex;
  flex-direction: row | column;
  justify-content: flex-start | center | space-between | space-around;
  align-items: flex-start | center | flex-end | stretch;
  flex-wrap: nowrap | wrap;
  gap: 20px;
}

/* Items */
.item {
  flex: 1;                    /* Grow */
  flex-basis: 200px;          /* Base size */
  align-self: flex-start;     /* Override alignment */
}
```

### Grid

```css
/* Container */
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: auto;
  gap: 20px;
  
  /* Named areas */
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
}

/* Items */
.item {
  grid-column: 1 / 3;         /* Span columns */
  grid-row: 1 / 2;            /* Span rows */
  grid-area: header;          /* Named area */
}
```

### Responsive Design

```css
/* Mobile-first */
.element {
  width: 100%;
}

/* Tablet */
@media (min-width: 768px) {
  .element {
    width: 50%;
  }
}

/* Desktop */
@media (min-width: 1024px) {
  .element {
    width: 33.333%;
  }
}
```

### Common Properties

```css
/* Box Model */
width: 100px;
height: 100px;
padding: 10px;
margin: 10px;
border: 1px solid #000;

/* Typography */
font-family: Arial, sans-serif;
font-size: 16px;
font-weight: bold;
line-height: 1.5;
text-align: center;
color: #333;

/* Background */
background-color: #fff;
background-image: url('image.jpg');
background-size: cover;
background-position: center;

/* Display */
display: block | inline | inline-block | flex | grid | none;
visibility: visible | hidden;
opacity: 0.5;

/* Position */
position: static | relative | absolute | fixed | sticky;
top: 0;
right: 0;
z-index: 10;

/* Transform */
transform: translate(10px, 20px);
transform: rotate(45deg);
transform: scale(1.5);

/* Transition */
transition: all 0.3s ease;

/* Animation */
animation: name 2s infinite;

@keyframes name {
  0% { opacity: 0; }
  100% { opacity: 1; }
}
```

### CSS Variables

```css
:root {
  --primary: #3498db;
  --spacing: 8px;
}

.element {
  color: var(--primary);
  padding: var(--spacing);
}
```

---

## ⚡ JavaScript Cheatsheet

### Variables

```javascript
const PI = 3.14;           // Cannot reassign
let count = 0;             // Can reassign
var old = 'avoid';         // Function-scoped (avoid)
```

### Data Types

```javascript
// Primitives
const str = 'string';
const num = 42;
const bool = true;
const nothing = null;
const undef = undefined;
const sym = Symbol('id');

// Objects
const obj = { key: 'value' };
const arr = [1, 2, 3];
const func = () => {};
```

### Functions

```javascript
// Function declaration
function add(a, b) {
  return a + b;
}

// Arrow function
const add = (a, b) => a + b;

// With block
const multiply = (a, b) => {
  return a * b;
};

// Default parameters
const greet = (name = 'Guest') => `Hello, ${name}`;
```

### Array Methods

```javascript
const arr = [1, 2, 3, 4, 5];

// Map - transform
arr.map(x => x * 2);              // [2, 4, 6, 8, 10]

// Filter - select
arr.filter(x => x > 2);           // [3, 4, 5]

// Reduce - accumulate
arr.reduce((sum, x) => sum + x, 0); // 15

// Find
arr.find(x => x > 2);             // 3
arr.findIndex(x => x > 2);        // 2

// Some/Every
arr.some(x => x > 3);             // true
arr.every(x => x > 0);            // true

// Sort
arr.sort((a, b) => a - b);        // Ascending

// Other
arr.push(6);                      // Add to end
arr.pop();                        // Remove from end
arr.unshift(0);                   // Add to start
arr.shift();                      // Remove from start
arr.slice(1, 3);                  // Extract [2, 3]
arr.splice(1, 2);                 // Remove 2 items at index 1
```

### Object Methods

```javascript
const obj = { a: 1, b: 2, c: 3 };

Object.keys(obj);                 // ['a', 'b', 'c']
Object.values(obj);               // [1, 2, 3]
Object.entries(obj);              // [['a', 1], ['b', 2], ['c', 3]]

// Destructuring
const { a, b } = obj;             // a = 1, b = 2

// Spread
const newObj = { ...obj, d: 4 };  // { a: 1, b: 2, c: 3, d: 4 }
```

### Async/Await

```javascript
// Promise
const promise = new Promise((resolve, reject) => {
  setTimeout(() => resolve('Done'), 1000);
});

promise
  .then(result => console.log(result))
  .catch(error => console.error(error));

// Async/Await
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error(error);
  }
}

// Parallel requests
const [user, posts] = await Promise.all([
  fetchUser(),
  fetchPosts()
]);
```

### DOM Manipulation

```javascript
// Select
const el = document.querySelector('.class');
const els = document.querySelectorAll('.class');

// Modify
el.textContent = 'Text';
el.innerHTML = '<strong>HTML</strong>';
el.style.color = 'red';
el.classList.add('active');
el.classList.remove('inactive');
el.classList.toggle('visible');

// Create
const div = document.createElement('div');
div.textContent = 'Hello';
document.body.appendChild(div);

// Events
el.addEventListener('click', (e) => {
  console.log('Clicked', e.target);
});
```

### Template Literals

```javascript
const name = 'John';
const age = 30;

const message = `Hello, ${name}! You are ${age} years old.`;

const html = `
  <div>
    <h1>${name}</h1>
    <p>Age: ${age}</p>
  </div>
`;
```

---

## 🔧 Git Commands

### Basic Commands

```bash
# Initialize
git init

# Clone
git clone <url>

# Status
git status

# Add files
git add .                    # All files
git add file.txt             # Specific file

# Commit
git commit -m "Message"
git commit -am "Message"     # Add + commit

# Push
git push origin main

# Pull
git pull origin main

# View history
git log
git log --oneline
git log --graph
```

### Branching

```bash
# Create branch
git branch feature-name

# Switch branch
git checkout feature-name
git switch feature-name      # Modern

# Create and switch
git checkout -b feature-name
git switch -c feature-name

# List branches
git branch                   # Local
git branch -r                # Remote
git branch -a                # All

# Merge
git checkout main
git merge feature-name

# Delete branch
git branch -d feature-name
git branch -D feature-name   # Force
```

### Undoing Changes

```bash
# Discard changes
git checkout -- file.txt

# Unstage
git reset HEAD file.txt

# Undo commit (keep changes)
git reset --soft HEAD~1

# Undo commit (discard changes)
git reset --hard HEAD~1

# Revert commit
git revert <commit-hash>
```

### Remote

```bash
# Add remote
git remote add origin <url>

# View remotes
git remote -v

# Fetch
git fetch origin

# Pull
git pull origin main

# Push
git push origin main
git push -u origin main      # Set upstream
```

---

## 🚀 Performance Tips

### Image Optimization

```html
<!-- Modern formats -->
<picture>
  <source srcset="image.avif" type="image/avif">
  <source srcset="image.webp" type="image/webp">
  <img src="image.jpg" alt="Fallback">
</picture>

<!-- Lazy loading -->
<img src="image.jpg" loading="lazy" alt="Description">

<!-- Responsive -->
<img 
  srcset="small.jpg 400w, large.jpg 800w"
  sizes="(max-width: 600px) 400px, 800px"
  src="large.jpg" 
  alt="Responsive">
```

### Resource Hints

```html
<!-- Preconnect -->
<link rel="preconnect" href="https://fonts.googleapis.com">

<!-- Prefetch -->
<link rel="prefetch" href="/next-page.html">

<!-- Preload -->
<link rel="preload" href="font.woff2" as="font" crossorigin>
```

### Code Splitting

```javascript
// Dynamic import
const module = await import('./module.js');

// React lazy
const Component = React.lazy(() => import('./Component'));
```

### Core Web Vitals Targets

- **LCP:** < 2.5s (Largest Contentful Paint)
- **FID:** < 100ms (First Input Delay)
- **CLS:** < 0.1 (Cumulative Layout Shift)

---

## ♿ Accessibility Checklist

### HTML

```html
<!-- Semantic structure -->
<header>, <nav>, <main>, <article>, <aside>, <footer>

<!-- Proper headings -->
<h1>Page Title</h1>
  <h2>Section</h2>
    <h3>Subsection</h3>

<!-- Form labels -->
<label for="email">Email:</label>
<input type="email" id="email" name="email">

<!-- Alt text -->
<img src="image.jpg" alt="Description">

<!-- Link text -->
<a href="/about">About Us</a>  ✅
<a href="/about">Click here</a> ❌
```

### ARIA

```html
<!-- Roles -->
<div role="button" tabindex="0">Click me</div>
<div role="alert">Error message</div>

<!-- States -->
<button aria-expanded="false">Menu</button>
<button aria-pressed="true">Bold</button>

<!-- Labels -->
<button aria-label="Close">×</button>
<input aria-describedby="help-text">
<span id="help-text">Help text</span>
```

### Keyboard Navigation

```javascript
// Handle keyboard events
element.addEventListener('keydown', (e) => {
  if (e.key === 'Enter' || e.key === ' ') {
    element.click();
  }
  if (e.key === 'Escape') {
    closeDialog();
  }
});

// Tab order
<input tabindex="0">     <!-- Natural order -->
<div tabindex="-1">      <!-- Not focusable -->
```

### Color Contrast

- **Normal text:** 4.5:1 minimum
- **Large text:** 3:1 minimum
- **UI components:** 3:1 minimum

### Quick Checks

- [ ] All images have alt text
- [ ] Forms have labels
- [ ] Proper heading hierarchy
- [ ] Keyboard navigation works
- [ ] Color contrast meets standards
- [ ] Focus indicators visible
- [ ] ARIA attributes used correctly
- [ ] Screen reader tested

---

## 🎯 Common Patterns

### Debounce

```javascript
function debounce(func, delay) {
  let timeout;
  return (...args) => {
    clearTimeout(timeout);
    timeout = setTimeout(() => func(...args), delay);
  };
}

// Usage
const search = debounce((query) => {
  console.log('Searching:', query);
}, 300);
```

### Throttle

```javascript
function throttle(func, limit) {
  let inThrottle;
  return (...args) => {
    if (!inThrottle) {
      func(...args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

// Usage
const handleScroll = throttle(() => {
  console.log('Scrolling');
}, 100);
```

### Local Storage

```javascript
// Save
localStorage.setItem('key', JSON.stringify(data));

// Get
const data = JSON.parse(localStorage.getItem('key'));

// Remove
localStorage.removeItem('key');

// Clear all
localStorage.clear();
```

---

## 📊 Browser DevTools

### Console

```javascript
console.log('Message');
console.error('Error');
console.warn('Warning');
console.table([{a: 1}, {a: 2}]);
console.time('label');
console.timeEnd('label');
```

### Network Tab

- View all requests
- Check file sizes
- Analyze load times
- Throttle connection

### Performance Tab

- Record performance
- Analyze Core Web Vitals
- Find bottlenecks
- Check frame rate

### Lighthouse

- Performance audit
- Accessibility audit
- Best practices
- SEO audit

---

**Last Updated:** December 2025
