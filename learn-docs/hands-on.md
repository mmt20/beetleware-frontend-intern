# Web Development — Hands-On Exercises

> **Practice makes perfect!** Complete these exercises to reinforce your understanding of web development concepts.

---

## 📚 Table of Contents

1. [HTML5 Exercises](#html5-exercises)
2. [CSS3 Exercises](#css3-exercises)
3. [JavaScript Exercises](#javascript-exercises)
4. [Git Exercises](#git-exercises)
5. [Performance Exercises](#performance-exercises)
6. [Accessibility Exercises](#accessibility-exercises)

---

## 📦 HTML5 Exercises

### Exercise 1: Semantic Blog Layout

**Objective:** Create a semantic HTML structure for a blog post.

**Requirements:**
- Use `<header>`, `<nav>`, `<main>`, `<article>`, `<aside>`, `<footer>`
- Include proper heading hierarchy (h1-h6)
- Add meta tags for SEO

**Starter Code:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My Blog Post</title>
</head>
<body>
  <!-- Your code here -->
</body>
</html>
```

**Expected Output:**
- Semantic structure with all required elements
- Proper nesting and hierarchy
- Valid HTML5

---

### Exercise 2: Advanced Form with Validation

**Objective:** Build a registration form with HTML5 validation.

**Requirements:**
- Email input with validation
- Password with minimum length (8 characters)
- Number input for age (18-100)
- Date picker for birthday
- Select dropdown for country
- Checkbox for terms acceptance
- Submit button

**Challenge:**
- Add custom validation messages
- Use `pattern` attribute for phone number
- Implement `required` and `aria-*` attributes

```html
<form action="/register" method="POST">
  <!-- Your form fields here -->
</form>
```

---

### Exercise 3: Responsive Images

**Objective:** Implement responsive images with multiple sources.

**Requirements:**
- Use `<picture>` element
- Provide 3 image sizes (400px, 800px, 1200px)
- Add WebP and AVIF formats with fallback
- Implement lazy loading

```html
<picture>
  <!-- Your sources here -->
</picture>
```

---

## 🎨 CSS3 Exercises

### Exercise 4: Flexbox Navigation

**Objective:** Create a responsive navigation bar using Flexbox.

**Requirements:**
- Logo on the left
- Navigation links on the right
- Centered on mobile (stacked)
- Horizontal on desktop

**HTML:**

```html
<nav class="navbar">
  <div class="logo">MyBrand</div>
  <ul class="nav-links">
    <li><a href="/">Home</a></li>
    <li><a href="/about">About</a></li>
    <li><a href="/contact">Contact</a></li>
  </ul>
</nav>
```

**CSS Challenge:**

```css
.navbar {
  /* Your flexbox styles here */
}

@media (max-width: 768px) {
  /* Mobile styles */
}
```

---

### Exercise 5: CSS Grid Layout

**Objective:** Build a responsive card grid layout.

**Requirements:**
- 3 columns on desktop
- 2 columns on tablet
- 1 column on mobile
- Equal height cards
- 20px gap between cards

**HTML:**

```html
<div class="grid">
  <div class="card">Card 1</div>
  <div class="card">Card 2</div>
  <div class="card">Card 3</div>
  <div class="card">Card 4</div>
  <div class="card">Card 5</div>
  <div class="card">Card 6</div>
</div>
```

**CSS Challenge:**

```css
.grid {
  display: grid;
  /* Your grid styles here */
}

@media (max-width: 1024px) {
  /* Tablet: 2 columns */
}

@media (max-width: 768px) {
  /* Mobile: 1 column */
}
```

---

### Exercise 6: CSS Variables Theme Switcher

**Objective:** Create a light/dark theme using CSS custom properties.

**Requirements:**
- Define color variables in `:root`
- Create dark theme with `[data-theme="dark"]`
- Smooth transitions between themes

**CSS:**

```css
:root {
  --bg-color: #ffffff;
  --text-color: #333333;
  --primary-color: #3498db;
}

[data-theme="dark"] {
  /* Dark theme variables */
}

body {
  background: var(--bg-color);
  color: var(--text-color);
  transition: background 0.3s, color 0.3s;
}
```

**JavaScript:**

```javascript
// Toggle theme
function toggleTheme() {
  const currentTheme = document.documentElement.getAttribute('data-theme');
  const newTheme = currentTheme === 'dark' ? 'light' : 'dark';
  document.documentElement.setAttribute('data-theme', newTheme);
}
```

---

## ⚡ JavaScript Exercises

### Exercise 7: Array Methods Practice

**Objective:** Master array methods (map, filter, reduce).

**Data:**

```javascript
const users = [
  { id: 1, name: 'John', age: 25, active: true },
  { id: 2, name: 'Jane', age: 30, active: false },
  { id: 3, name: 'Bob', age: 35, active: true },
  { id: 4, name: 'Alice', age: 28, active: true }
];
```

**Tasks:**

```javascript
// 1. Get names of all users
const names = users.map(/* Your code */);

// 2. Filter active users
const activeUsers = users.filter(/* Your code */);

// 3. Calculate average age
const averageAge = users.reduce(/* Your code */, 0);

// 4. Get active users over 25
const result = users
  .filter(/* Your code */)
  .map(/* Your code */);
```

---

### Exercise 8: Async/Await API Calls

**Objective:** Fetch data from an API using async/await.

**Requirements:**
- Fetch users from JSONPlaceholder API
- Handle errors with try/catch
- Display loading state
- Show error messages

**Challenge:**

```javascript
async function fetchUsers() {
  try {
    // 1. Set loading state
    
    // 2. Fetch data
    const response = await fetch('https://jsonplaceholder.typicode.com/users');
    
    // 3. Check response
    if (!response.ok) {
      throw new Error('Failed to fetch');
    }
    
    // 4. Parse JSON
    const users = await response.json();
    
    // 5. Display users
    displayUsers(users);
    
  } catch (error) {
    // 6. Handle errors
    console.error('Error:', error);
  } finally {
    // 7. Clear loading state
  }
}

function displayUsers(users) {
  // Your code to display users
}
```

---

### Exercise 9: DOM Manipulation Todo List

**Objective:** Build a simple todo list with DOM manipulation.

**Requirements:**
- Add new todos
- Mark todos as complete
- Delete todos
- Filter todos (all, active, completed)

**HTML:**

```html
<div class="todo-app">
  <input type="text" id="todo-input" placeholder="Add todo...">
  <button id="add-btn">Add</button>
  
  <div class="filters">
    <button data-filter="all">All</button>
    <button data-filter="active">Active</button>
    <button data-filter="completed">Completed</button>
  </div>
  
  <ul id="todo-list"></ul>
</div>
```

**JavaScript Challenge:**

```javascript
// State
let todos = [];

// Add todo
function addTodo(text) {
  const todo = {
    id: Date.now(),
    text: text,
    completed: false
  };
  todos.push(todo);
  renderTodos();
}

// Toggle complete
function toggleTodo(id) {
  // Your code
}

// Delete todo
function deleteTodo(id) {
  // Your code
}

// Render todos
function renderTodos(filter = 'all') {
  // Your code
}

// Event listeners
document.getElementById('add-btn').addEventListener('click', () => {
  const input = document.getElementById('todo-input');
  if (input.value.trim()) {
    addTodo(input.value);
    input.value = '';
  }
});
```

---

## 🔧 Git Exercises

### Exercise 10: Basic Git Workflow

**Objective:** Practice basic Git commands.

**Tasks:**

```bash
# 1. Initialize a new repository
git init my-project
cd my-project

# 2. Create a file
echo "# My Project" > README.md

# 3. Stage and commit
git add README.md
git commit -m "Initial commit"

# 4. Create a new branch
git checkout -b feature/add-content

# 5. Make changes
echo "## Description" >> README.md

# 6. Commit changes
git add README.md
git commit -m "Add description section"

# 7. Switch back to main
git checkout main

# 8. Merge feature branch
git merge feature/add-content

# 9. Delete feature branch
git branch -d feature/add-content
```

---

### Exercise 11: Collaboration Workflow

**Objective:** Simulate a team collaboration workflow.

**Scenario:**
You're working on a team project. Practice the pull request workflow.

**Tasks:**

```bash
# 1. Clone repository (use any public repo)
git clone https://github.com/user/repo.git
cd repo

# 2. Create feature branch
git checkout -b feature/my-feature

# 3. Make changes and commit
# ... edit files ...
git add .
git commit -m "Add my feature"

# 4. Push to remote
git push origin feature/my-feature

# 5. Create Pull Request on GitHub
# (Use GitHub UI)

# 6. After merge, update local main
git checkout main
git pull origin main

# 7. Delete local feature branch
git branch -d feature/my-feature
```

---

### Exercise 12: Resolve Merge Conflicts

**Objective:** Practice resolving merge conflicts.

**Setup:**

```bash
# Create test repository
git init conflict-practice
cd conflict-practice

# Create file
echo "Line 1" > file.txt
git add file.txt
git commit -m "Initial commit"

# Create branch 1
git checkout -b branch1
echo "Line 2 from branch1" >> file.txt
git commit -am "Update from branch1"

# Create branch 2
git checkout main
git checkout -b branch2
echo "Line 2 from branch2" >> file.txt
git commit -am "Update from branch2"

# Merge branch1 into main
git checkout main
git merge branch1

# Try to merge branch2 (conflict!)
git merge branch2
```

**Resolve:**

```bash
# 1. Open file.txt and resolve conflict
# 2. Remove conflict markers (<<<<, ====, >>>>)
# 3. Keep desired changes
# 4. Stage resolved file
git add file.txt

# 5. Complete merge
git commit -m "Resolve merge conflict"
```

---

## 🚀 Performance Exercises

### Exercise 13: Image Optimization

**Objective:** Optimize images for web performance.

**Tasks:**

1. **Convert images to modern formats:**
   - Use online tools or CLI to convert to WebP/AVIF
   - Compare file sizes

2. **Implement responsive images:**

```html
<picture>
  <source 
    srcset="image-400.avif 400w, image-800.avif 800w" 
    type="image/avif">
  <source 
    srcset="image-400.webp 400w, image-800.webp 800w" 
    type="image/webp">
  <img 
    src="image-800.jpg" 
    alt="Description"
    loading="lazy">
</picture>
```

3. **Measure impact:**
   - Use Chrome DevTools Network tab
   - Compare before/after file sizes

---

### Exercise 14: Code Splitting

**Objective:** Implement code splitting to improve load time.

**Vanilla JavaScript:**

```javascript
// Dynamic import
button.addEventListener('click', async () => {
  const module = await import('./heavy-module.js');
  module.initialize();
});
```

**React Example:**

```javascript
import React, { lazy, Suspense } from 'react';

// Lazy load component
const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <HeavyComponent />
    </Suspense>
  );
}
```

---

### Exercise 15: Performance Audit

**Objective:** Audit a website's performance using Lighthouse.

**Tasks:**

1. **Run Lighthouse audit:**
   - Open Chrome DevTools
   - Go to Lighthouse tab
   - Run audit for Performance

2. **Analyze results:**
   - Review Core Web Vitals
   - Identify opportunities
   - Note diagnostics

3. **Implement improvements:**
   - Fix identified issues
   - Re-run audit
   - Compare scores

---

## ♿ Accessibility Exercises

### Exercise 16: Accessible Form

**Objective:** Create a fully accessible form.

**Requirements:**
- Proper labels for all inputs
- ARIA attributes where needed
- Keyboard navigation support
- Error messages

```html
<form>
  <div class="form-group">
    <label for="name">Name:</label>
    <input 
      type="text" 
      id="name" 
      name="name" 
      required
      aria-required="true"
      aria-describedby="name-error">
    <span id="name-error" class="error" role="alert"></span>
  </div>
  
  <div class="form-group">
    <label for="email">Email:</label>
    <input 
      type="email" 
      id="email" 
      name="email" 
      required
      aria-required="true"
      aria-describedby="email-help email-error">
    <span id="email-help" class="help-text">
      We'll never share your email
    </span>
    <span id="email-error" class="error" role="alert"></span>
  </div>
  
  <button type="submit">Submit</button>
</form>
```

---

### Exercise 17: Keyboard Navigation

**Objective:** Implement keyboard navigation for a custom dropdown.

**Requirements:**
- Tab to focus dropdown
- Enter/Space to open
- Arrow keys to navigate options
- Escape to close
- Enter to select

```javascript
class AccessibleDropdown {
  constructor(element) {
    this.dropdown = element;
    this.button = element.querySelector('[role="button"]');
    this.menu = element.querySelector('[role="menu"]');
    this.options = element.querySelectorAll('[role="menuitem"]');
    this.currentIndex = -1;
    
    this.init();
  }
  
  init() {
    // Button click
    this.button.addEventListener('click', () => this.toggle());
    
    // Keyboard navigation
    this.button.addEventListener('keydown', (e) => {
      if (e.key === 'Enter' || e.key === ' ') {
        e.preventDefault();
        this.toggle();
      }
    });
    
    this.menu.addEventListener('keydown', (e) => {
      switch(e.key) {
        case 'ArrowDown':
          e.preventDefault();
          this.focusNext();
          break;
        case 'ArrowUp':
          e.preventDefault();
          this.focusPrevious();
          break;
        case 'Escape':
          this.close();
          break;
        case 'Enter':
          this.selectCurrent();
          break;
      }
    });
  }
  
  toggle() {
    const isOpen = this.menu.getAttribute('aria-hidden') === 'false';
    if (isOpen) {
      this.close();
    } else {
      this.open();
    }
  }
  
  open() {
    this.menu.setAttribute('aria-hidden', 'false');
    this.button.setAttribute('aria-expanded', 'true');
    this.options[0].focus();
    this.currentIndex = 0;
  }
  
  close() {
    this.menu.setAttribute('aria-hidden', 'true');
    this.button.setAttribute('aria-expanded', 'false');
    this.button.focus();
    this.currentIndex = -1;
  }
  
  focusNext() {
    this.currentIndex = (this.currentIndex + 1) % this.options.length;
    this.options[this.currentIndex].focus();
  }
  
  focusPrevious() {
    this.currentIndex = this.currentIndex <= 0 
      ? this.options.length - 1 
      : this.currentIndex - 1;
    this.options[this.currentIndex].focus();
  }
  
  selectCurrent() {
    // Handle selection
    this.close();
  }
}
```

---

### Exercise 18: Screen Reader Testing

**Objective:** Test your website with a screen reader.

**Tasks:**

1. **Install screen reader:**
   - Windows: NVDA (free)
   - Mac: VoiceOver (built-in)
   - Linux: Orca

2. **Test navigation:**
   - Navigate with Tab key
   - Listen to announcements
   - Test form inputs
   - Verify heading structure

3. **Common issues to check:**
   - Missing alt text on images
   - Unlabeled form inputs
   - Poor heading hierarchy
   - Missing ARIA labels

4. **Fix issues:**
   - Add missing labels
   - Improve semantic structure
   - Add ARIA attributes where needed

---

## 🎯 Challenge Projects

### Challenge 1: Accessible Modal Dialog

Build a fully accessible modal dialog with:
- Keyboard navigation
- Focus trap
- ARIA attributes
- Escape to close

### Challenge 2: Performance-Optimized Gallery

Create an image gallery with:
- Lazy loading
- Responsive images
- Modern image formats
- Smooth animations

### Challenge 3: Git Workflow Automation

Create a script to automate:
- Branch creation
- Commit message formatting
- Pre-commit hooks
- Automated testing

---

## 📊 Self-Assessment

After completing exercises, rate your understanding:

- [ ] HTML5 Semantic Markup
- [ ] CSS3 Layouts (Flexbox/Grid)
- [ ] JavaScript ES6+ Features
- [ ] Async JavaScript
- [ ] Git Workflows
- [ ] Performance Optimization
- [ ] Web Accessibility

---

**Next Steps:**
1. Complete all exercises
2. Build deliverable projects
3. Review solutions
4. Practice regularly

---

**Last Updated:** December 2025
