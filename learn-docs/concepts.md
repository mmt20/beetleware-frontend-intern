# Web Development — Complete Concepts Guide

> **Learning Objective:** Master modern web development fundamentals including HTML5, CSS3, JavaScript ES6+, Git, performance optimization, and accessibility standards.

---

## 📚 Table of Contents

1. [HTML5 & Semantic Markup](#part-1-html5--semantic-markup)
2. [CSS3 & Modern Layouts](#part-2-css3--modern-layouts)
3. [JavaScript ES6+](#part-3-javascript-es6)
4. [Git & Version Control](#part-4-git--version-control)
5. [Web Performance](#part-5-web-performance)
6. [Web Accessibility](#part-6-web-accessibility)

---

## 📦 Part 1: HTML5 & Semantic Markup

### 1.1 What Is Semantic HTML?

**Semantic HTML** uses meaningful tags that describe the content's purpose, not just its appearance.

**Key Benefits:**

- **SEO** — Search engines understand content structure
- **Accessibility** — Screen readers navigate better
- **Maintainability** — Code is self-documenting
- **Consistency** — Standardized meaning across browsers

---

#### Semantic vs Non-Semantic Elements

**Non-Semantic (Avoid):**

```html
<div class="header">
  <div class="nav">
    <div class="nav-item">Home</div>
  </div>
</div>
<div class="main-content">
  <div class="article">...</div>
</div>
<div class="footer">...</div>
```

**Semantic (Recommended) ✅:**

```html
<header>
  <nav>
    <a href="/">Home</a>
  </nav>
</header>
<main>
  <article>...</article>
</main>
<footer>...</footer>
```

---

### 1.2 HTML5 Structural Elements

**Document Structure:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Page Title</title>
</head>
<body>
  <header>
    <!-- Site header, logo, navigation -->
  </header>
  
  <nav>
    <!-- Main navigation links -->
  </nav>
  
  <main>
    <!-- Primary content -->
    <article>
      <!-- Self-contained content -->
      <header>
        <h1>Article Title</h1>
      </header>
      <section>
        <!-- Thematic grouping -->
      </section>
    </article>
    
    <aside>
      <!-- Sidebar, related content -->
    </aside>
  </main>
  
  <footer>
    <!-- Site footer -->
  </footer>
</body>
</html>
```

**Element Descriptions:**

| Element | Purpose | Example Use |
|---------|---------|-------------|
| `<header>` | Introductory content | Site header, article header |
| `<nav>` | Navigation links | Main menu, breadcrumbs |
| `<main>` | Primary content | Main page content (one per page) |
| `<article>` | Self-contained content | Blog post, news article |
| `<section>` | Thematic grouping | Chapter, tab panel |
| `<aside>` | Tangential content | Sidebar, callout box |
| `<footer>` | Footer content | Copyright, links |

---

### 1.3 Forms and Input Types

**Modern Form Structure:**

```html
<form action="/submit" method="POST">
  <!-- Text Input -->
  <label for="name">Name:</label>
  <input type="text" id="name" name="name" required>
  
  <!-- Email with Validation -->
  <label for="email">Email:</label>
  <input type="email" id="email" name="email" required>
  
  <!-- Number with Range -->
  <label for="age">Age:</label>
  <input type="number" id="age" name="age" min="18" max="100">
  
  <!-- Date Picker -->
  <label for="birthday">Birthday:</label>
  <input type="date" id="birthday" name="birthday">
  
  <!-- Select Dropdown -->
  <label for="country">Country:</label>
  <select id="country" name="country">
    <option value="">Select...</option>
    <option value="us">United States</option>
    <option value="uk">United Kingdom</option>
  </select>
  
  <!-- Textarea -->
  <label for="message">Message:</label>
  <textarea id="message" name="message" rows="4"></textarea>
  
  <!-- Checkbox -->
  <label>
    <input type="checkbox" name="subscribe" value="yes">
    Subscribe to newsletter
  </label>
  
  <!-- Radio Buttons -->
  <fieldset>
    <legend>Gender:</legend>
    <label>
      <input type="radio" name="gender" value="male"> Male
    </label>
    <label>
      <input type="radio" name="gender" value="female"> Female
    </label>
  </fieldset>
  
  <button type="submit">Submit</button>
</form>
```

**HTML5 Input Types:**

- `text`, `email`, `password`, `tel`, `url`
- `number`, `range`, `date`, `time`, `datetime-local`
- `color`, `file`, `search`

**Validation Attributes:**

- `required` — Field must be filled
- `pattern="[A-Za-z]+"` — Regex validation
- `min`, `max` — Number/date ranges
- `minlength`, `maxlength` — String length

---

### 1.4 Media Elements

**Images:**

```html
<!-- Basic Image -->
<img src="image.jpg" alt="Description" width="800" height="600">

<!-- Responsive Images -->
<img 
  src="image-800.jpg" 
  srcset="image-400.jpg 400w, image-800.jpg 800w, image-1200.jpg 1200w"
  sizes="(max-width: 600px) 400px, (max-width: 1000px) 800px, 1200px"
  alt="Responsive image">

<!-- Picture Element (Art Direction) -->
<picture>
  <source media="(min-width: 800px)" srcset="large.jpg">
  <source media="(min-width: 400px)" srcset="medium.jpg">
  <img src="small.jpg" alt="Fallback">
</picture>
```

**Video:**

```html
<video controls width="640" height="360" poster="thumbnail.jpg">
  <source src="video.mp4" type="video/mp4">
  <source src="video.webm" type="video/webm">
  <track kind="subtitles" src="subtitles.vtt" srclang="en" label="English">
  Your browser doesn't support video.
</video>
```

**Audio:**

```html
<audio controls>
  <source src="audio.mp3" type="audio/mpeg">
  <source src="audio.ogg" type="audio/ogg">
  Your browser doesn't support audio.
</audio>
```

---

## 🎨 Part 2: CSS3 & Modern Layouts

### 2.1 Flexbox Layout

**Flexbox** is a one-dimensional layout system for rows or columns.

**Basic Flexbox:**

```css
.container {
  display: flex;
  flex-direction: row; /* row | column */
  justify-content: space-between; /* Horizontal alignment */
  align-items: center; /* Vertical alignment */
  gap: 20px; /* Space between items */
}

.item {
  flex: 1; /* Grow to fill space */
  flex-basis: 200px; /* Base width */
}
```

**Flex Properties:**

**Container Properties:**
- `display: flex` — Enable flexbox
- `flex-direction` — row | column | row-reverse | column-reverse
- `justify-content` — flex-start | center | space-between | space-around
- `align-items` — flex-start | center | flex-end | stretch
- `flex-wrap` — nowrap | wrap | wrap-reverse
- `gap` — Space between items

**Item Properties:**
- `flex-grow` — Growth factor (default: 0)
- `flex-shrink` — Shrink factor (default: 1)
- `flex-basis` — Base size
- `flex` — Shorthand (grow shrink basis)
- `align-self` — Override container alignment

**Common Patterns:**

```css
/* Center Content */
.center {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

/* Equal Columns */
.columns {
  display: flex;
  gap: 20px;
}
.columns > * {
  flex: 1;
}

/* Responsive Navigation */
.nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
```

---

### 2.2 CSS Grid Layout

**Grid** is a two-dimensional layout system for rows and columns.

**Basic Grid:**

```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr); /* 3 equal columns */
  grid-template-rows: auto;
  gap: 20px;
}

.item {
  grid-column: span 2; /* Span 2 columns */
  grid-row: 1 / 3; /* From row 1 to 3 */
}
```

**Grid Properties:**

**Container:**
- `display: grid`
- `grid-template-columns` — Column sizes
- `grid-template-rows` — Row sizes
- `gap` — Space between cells
- `grid-template-areas` — Named areas

**Item:**
- `grid-column` — Column placement
- `grid-row` — Row placement
- `grid-area` — Named area

**Common Patterns:**

```css
/* 12-Column Grid */
.grid {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: 20px;
}

/* Responsive Grid */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}

/* Named Areas */
.layout {
  display: grid;
  grid-template-areas:
    "header header header"
    "sidebar main main"
    "footer footer footer";
  grid-template-columns: 200px 1fr 1fr;
}

.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.footer { grid-area: footer; }
```

---

### 2.3 Responsive Design

**Mobile-First Approach:**

```css
/* Base styles (mobile) */
.container {
  padding: 10px;
}

/* Tablet */
@media (min-width: 768px) {
  .container {
    padding: 20px;
  }
}

/* Desktop */
@media (min-width: 1024px) {
  .container {
    padding: 40px;
    max-width: 1200px;
    margin: 0 auto;
  }
}
```

**Common Breakpoints:**

```css
/* Mobile: 320px - 767px (default) */
/* Tablet: 768px - 1023px */
@media (min-width: 768px) { }

/* Desktop: 1024px+ */
@media (min-width: 1024px) { }

/* Large Desktop: 1440px+ */
@media (min-width: 1440px) { }
```

---

### 2.4 CSS Custom Properties (Variables)

**Defining Variables:**

```css
:root {
  /* Colors */
  --color-primary: #3498db;
  --color-secondary: #2ecc71;
  --color-danger: #e74c3c;
  
  /* Typography */
  --font-family: 'Helvetica Neue', sans-serif;
  --font-size-base: 16px;
  --line-height: 1.5;
  
  /* Spacing */
  --spacing-unit: 8px;
  --spacing-sm: calc(var(--spacing-unit) * 1);
  --spacing-md: calc(var(--spacing-unit) * 2);
  --spacing-lg: calc(var(--spacing-unit) * 4);
}

/* Usage */
.button {
  background: var(--color-primary);
  padding: var(--spacing-md);
  font-family: var(--font-family);
}
```

**Dynamic Theming:**

```css
/* Light Theme (default) */
:root {
  --bg-color: #ffffff;
  --text-color: #333333;
}

/* Dark Theme */
[data-theme="dark"] {
  --bg-color: #1a1a1a;
  --text-color: #ffffff;
}

body {
  background: var(--bg-color);
  color: var(--text-color);
}
```

---

## ⚡ Part 3: JavaScript ES6+

### 3.1 Modern Syntax

**Variables:**

```javascript
// const - Cannot reassign (use by default)
const PI = 3.14159;

// let - Can reassign (use when needed)
let count = 0;
count++;

// var - Avoid (function-scoped, hoisted)
```

**Arrow Functions:**

```javascript
// Traditional function
function add(a, b) {
  return a + b;
}

// Arrow function
const add = (a, b) => a + b;

// With block
const multiply = (a, b) => {
  const result = a * b;
  return result;
};

// Single parameter (no parentheses)
const square = x => x * x;
```

**Template Literals:**

```javascript
const name = 'John';
const age = 30;

// String interpolation
const message = `Hello, ${name}! You are ${age} years old.`;

// Multi-line strings
const html = `
  <div>
    <h1>${name}</h1>
    <p>Age: ${age}</p>
  </div>
`;
```

**Destructuring:**

```javascript
// Array destructuring
const [first, second, ...rest] = [1, 2, 3, 4, 5];
// first = 1, second = 2, rest = [3, 4, 5]

// Object destructuring
const user = { name: 'John', age: 30, city: 'NYC' };
const { name, age } = user;

// Renaming
const { name: userName, age: userAge } = user;

// Default values
const { country = 'USA' } = user;
```

**Spread Operator:**

```javascript
// Array spread
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2]; // [1, 2, 3, 4, 5, 6]

// Object spread
const user = { name: 'John', age: 30 };
const updatedUser = { ...user, city: 'NYC' };
// { name: 'John', age: 30, city: 'NYC' }
```

---

### 3.2 Async JavaScript

**Promises:**

```javascript
// Creating a promise
const fetchData = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const data = { id: 1, name: 'John' };
      resolve(data); // Success
      // reject(new Error('Failed')); // Error
    }, 1000);
  });
};

// Using promises
fetchData()
  .then(data => console.log(data))
  .catch(error => console.error(error))
  .finally(() => console.log('Done'));
```

**Async/Await:**

```javascript
// Async function
async function getUserData() {
  try {
    const response = await fetch('/api/user');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Error:', error);
    throw error;
  }
}

// Using async function
getUserData()
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

**Parallel Requests:**

```javascript
// Sequential (slow)
const user = await fetchUser();
const posts = await fetchPosts();

// Parallel (fast) ✅
const [user, posts] = await Promise.all([
  fetchUser(),
  fetchPosts()
]);
```

---

### 3.3 DOM Manipulation

**Selecting Elements:**

```javascript
// Single element
const element = document.querySelector('.class');
const elementById = document.getElementById('id');

// Multiple elements
const elements = document.querySelectorAll('.class');
const elementsByTag = document.getElementsByTagName('div');
```

**Modifying Elements:**

```javascript
// Change content
element.textContent = 'New text';
element.innerHTML = '<strong>Bold text</strong>';

// Change attributes
element.setAttribute('data-id', '123');
element.id = 'new-id';
element.className = 'new-class';

// Change styles
element.style.color = 'red';
element.style.backgroundColor = 'blue';

// Add/remove classes
element.classList.add('active');
element.classList.remove('inactive');
element.classList.toggle('visible');
```

**Creating Elements:**

```javascript
// Create element
const div = document.createElement('div');
div.textContent = 'Hello';
div.className = 'box';

// Append to DOM
document.body.appendChild(div);

// Insert before
parent.insertBefore(div, referenceElement);

// Remove element
element.remove();
```

**Event Handling:**

```javascript
// Add event listener
button.addEventListener('click', (event) => {
  console.log('Clicked!', event.target);
});

// Remove event listener
const handler = () => console.log('Clicked');
button.addEventListener('click', handler);
button.removeEventListener('click', handler);

// Event delegation
document.addEventListener('click', (event) => {
  if (event.target.matches('.button')) {
    console.log('Button clicked');
  }
});
```

---

## 🔧 Part 4: Git & Version Control

### 4.1 Git Fundamentals

**Basic Commands:**

```bash
# Initialize repository
git init

# Clone repository
git clone https://github.com/user/repo.git

# Check status
git status

# Add files
git add file.txt          # Single file
git add .                 # All files
git add *.js              # Pattern

# Commit changes
git commit -m "Add feature"

# Push to remote
git push origin main

# Pull from remote
git pull origin main
```

**Viewing History:**

```bash
# View commit history
git log
git log --oneline
git log --graph --all

# View changes
git diff                  # Unstaged changes
git diff --staged         # Staged changes
git diff commit1 commit2  # Between commits
```

---

### 4.2 Branching Strategies

**Branch Commands:**

```bash
# Create branch
git branch feature-login

# Switch branch
git checkout feature-login
# Or (modern)
git switch feature-login

# Create and switch
git checkout -b feature-login
# Or
git switch -c feature-login

# List branches
git branch                # Local
git branch -r             # Remote
git branch -a             # All

# Delete branch
git branch -d feature-login
git branch -D feature-login  # Force delete
```

**Git Flow:**

```bash
# Main branches
main (production)
develop (integration)

# Supporting branches
feature/feature-name
release/v1.0.0
hotfix/bug-fix

# Workflow
git checkout develop
git checkout -b feature/login
# ... work on feature ...
git checkout develop
git merge feature/login
git branch -d feature/login
```

---

### 4.3 Collaboration Workflows

**Pull Request Workflow:**

```bash
# 1. Create feature branch
git checkout -b feature/new-feature

# 2. Make changes and commit
git add .
git commit -m "Add new feature"

# 3. Push to remote
git push origin feature/new-feature

# 4. Create Pull Request on GitHub
# 5. Review and merge
# 6. Delete branch
git checkout main
git pull origin main
git branch -d feature/new-feature
```

**Resolving Conflicts:**

```bash
# Pull latest changes
git pull origin main

# If conflicts occur
# 1. Open conflicted files
# 2. Resolve conflicts manually
# 3. Mark as resolved
git add conflicted-file.txt
git commit -m "Resolve merge conflict"
```

---

## 🚀 Part 5: Web Performance

### 5.1 Core Web Vitals

**Key Metrics:**

1. **LCP (Largest Contentful Paint)** — Loading performance
   - **Good:** < 2.5s
   - **Needs Improvement:** 2.5s - 4s
   - **Poor:** > 4s

2. **FID (First Input Delay)** — Interactivity
   - **Good:** < 100ms
   - **Needs Improvement:** 100ms - 300ms
   - **Poor:** > 300ms

3. **CLS (Cumulative Layout Shift)** — Visual stability
   - **Good:** < 0.1
   - **Needs Improvement:** 0.1 - 0.25
   - **Poor:** > 0.25

---

### 5.2 Loading Optimization

**Image Optimization:**

```html
<!-- Use modern formats -->
<picture>
  <source srcset="image.avif" type="image/avif">
  <source srcset="image.webp" type="image/webp">
  <img src="image.jpg" alt="Fallback">
</picture>

<!-- Lazy loading -->
<img src="image.jpg" loading="lazy" alt="Description">

<!-- Responsive images -->
<img 
  srcset="small.jpg 400w, medium.jpg 800w, large.jpg 1200w"
  sizes="(max-width: 600px) 400px, 800px"
  src="medium.jpg" 
  alt="Responsive">
```

**Resource Hints:**

```html
<!-- Preconnect to external domains -->
<link rel="preconnect" href="https://fonts.googleapis.com">

<!-- Prefetch resources -->
<link rel="prefetch" href="/next-page.html">

<!-- Preload critical resources -->
<link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>
```

**Code Splitting:**

```javascript
// Dynamic imports
const module = await import('./module.js');

// React lazy loading
const Component = React.lazy(() => import('./Component'));
```

---

### 5.3 Caching Strategies

**HTTP Caching:**

```
# Cache static assets
Cache-Control: public, max-age=31536000, immutable

# Cache HTML (revalidate)
Cache-Control: public, max-age=0, must-revalidate

# No cache
Cache-Control: no-store
```

**Service Worker Caching:**

```javascript
// Cache-first strategy
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request)
      .then(response => response || fetch(event.request))
  );
});
```

---

## ♿ Part 6: Web Accessibility

### 6.1 WCAG Guidelines

**Four Principles (POUR):**

1. **Perceivable** — Information must be presentable
2. **Operable** — UI components must be operable
3. **Understandable** — Information must be understandable
4. **Robust** — Content must work with assistive technologies

**Conformance Levels:**

- **Level A** — Minimum (basic)
- **Level AA** — Mid-range (recommended) ✅
- **Level AAA** — Highest (enhanced)

---

### 6.2 Semantic HTML for Accessibility

**Proper Headings:**

```html
<!-- Correct hierarchy -->
<h1>Page Title</h1>
  <h2>Section 1</h2>
    <h3>Subsection 1.1</h3>
  <h2>Section 2</h2>

<!-- Incorrect (skip levels) ❌ -->
<h1>Title</h1>
<h3>Subsection</h3>
```

**Form Labels:**

```html
<!-- Explicit label (recommended) ✅ -->
<label for="email">Email:</label>
<input type="email" id="email" name="email">

<!-- Implicit label -->
<label>
  Email:
  <input type="email" name="email">
</label>

<!-- No label (inaccessible) ❌ -->
<input type="email" placeholder="Email">
```

---

### 6.3 ARIA Attributes

**ARIA Roles:**

```html
<!-- Navigation -->
<nav role="navigation">
  <ul role="menubar">
    <li role="menuitem"><a href="/">Home</a></li>
  </ul>
</nav>

<!-- Button -->
<div role="button" tabindex="0">Click me</div>

<!-- Alert -->
<div role="alert">Error: Please try again</div>
```

**ARIA States:**

```html
<!-- Expanded/Collapsed -->
<button aria-expanded="false" aria-controls="menu">
  Menu
</button>
<div id="menu" aria-hidden="true">...</div>

<!-- Selected -->
<button aria-pressed="true">Bold</button>

<!-- Disabled -->
<button aria-disabled="true">Submit</button>
```

**ARIA Labels:**

```html
<!-- Label -->
<button aria-label="Close dialog">×</button>

<!-- Described by -->
<input 
  type="password" 
  aria-describedby="password-help">
<span id="password-help">
  Must be at least 8 characters
</span>
```

---

### 6.4 Keyboard Navigation

**Tab Order:**

```html
<!-- Natural tab order -->
<input type="text" tabindex="0">
<button tabindex="0">Submit</button>

<!-- Custom tab order (avoid) -->
<input type="text" tabindex="2">
<button tabindex="1">

<!-- Remove from tab order -->
<div tabindex="-1">Not focusable</div>
```

**Keyboard Events:**

```javascript
// Handle keyboard navigation
element.addEventListener('keydown', (event) => {
  if (event.key === 'Enter' || event.key === ' ') {
    // Activate element
    event.preventDefault();
    element.click();
  }
  
  if (event.key === 'Escape') {
    // Close dialog
    closeDialog();
  }
});
```

---

## 🎓 Summary

This comprehensive guide covers:

✅ **HTML5** — Semantic markup, forms, media elements  
✅ **CSS3** — Flexbox, Grid, responsive design, variables  
✅ **JavaScript ES6+** — Modern syntax, async programming, DOM manipulation  
✅ **Git** — Version control, branching, collaboration  
✅ **Performance** — Core Web Vitals, optimization techniques  
✅ **Accessibility** — WCAG guidelines, ARIA, keyboard navigation  

**Next Steps:**
1. Practice with hands-on exercises
2. Build real projects
3. Review quick reference notes
4. Explore additional resources

---

**Last Updated:** December 2025
