# SASS — Complete Concepts Guide

> **Learning Objective:** Master SASS/SCSS preprocessor features including variables, nesting, mixins, functions, loops, and advanced architecture patterns to write maintainable and scalable CSS.

---

## 📚 Part 1: Introduction and Fundamentals

### 1.1 What Is SASS?

**SASS (Syntactically Awesome Style Sheets)** is a CSS preprocessor that extends CSS with programming features. It compiles to standard CSS that browsers can understand.

**Key Benefits:**

- **Variables** — Store reusable values (colors, fonts, sizes)
- **Nesting** — Write hierarchical CSS that mirrors HTML structure
- **Mixins** — Create reusable chunks of CSS
- **Functions** — Perform calculations and transformations
- **Partials** — Split CSS into smaller, manageable files
- **Inheritance** — Share styles between selectors
- **Operators** — Perform math and logic operations

---

#### SASS vs SCSS Syntax

SASS has **two syntaxes**:

**1. SASS (Indented Syntax) — `.sass`**

```sass
// No braces or semicolons
$primary-color: #3498db

.button
  background: $primary-color
  padding: 10px 20px
  &:hover
    background: darken($primary-color, 10%)
```

**2. SCSS (Sassy CSS) — `.scss` ✅ RECOMMENDED**

```scss
// CSS-like syntax with braces and semicolons
$primary-color: #3498db;

.button {
  background: $primary-color;
  padding: 10px 20px;
  
  &:hover {
    background: darken($primary-color, 10%);
  }
}
```

**Why SCSS?**

- ✅ Easier to learn (valid CSS is valid SCSS)
- ✅ Better tooling support
- ✅ More familiar to CSS developers
- ✅ Industry standard

---

#### SASS vs Other Preprocessors

| Feature | SASS/SCSS | LESS | Stylus |
|---------|-----------|------|--------|
| **Syntax** | SCSS (CSS-like) or SASS (indented) | CSS-like | Flexible (optional braces) |
| **Variables** | `$variable` | `@variable` | `variable` |
| **Mixins** | `@mixin` / `@include` | `.mixin()` | `mixin()` |
| **Functions** | `@function` | Built-in only | Yes |
| **Loops** | `@for`, `@each`, `@while` | Limited | Yes |
| **Compiler** | Dart Sass (official) | Less.js | Stylus |
| **Popularity** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |

**SASS is the most feature-rich and widely adopted.**

---

### 1.2 SASS Compilation Tools

SASS code must be **compiled to CSS** before browsers can use it.

#### Installation Methods

**1. Dart Sass (Official, Recommended) ✅**

```bash
# Install globally via npm
npm install -g sass

# Install as dev dependency
npm install --save-dev sass
```

**2. Node-sass (Deprecated) ❌**

```bash
# LibSass-based (no longer maintained)
npm install node-sass  # Don't use this!
```

> [!WARNING]
> **Node-sass is deprecated!** Use Dart Sass instead. LibSass (the C++ implementation) is no longer maintained.

---

#### Compilation Commands

**Basic Compilation:**

```bash
# Compile single file
sass input.scss output.css

# Compile with source maps
sass --source-map input.scss output.css

# Compressed output (production)
sass --style=compressed input.scss output.css
```

**Watch Mode (Auto-compile on save):**

```bash
# Watch single file
sass --watch input.scss:output.css

# Watch entire directory
sass --watch scss:css

# Watch with source maps
sass --watch --source-map scss:css
```

**Output Styles:**

```bash
sass --style=expanded input.scss output.css    # Default, readable
sass --style=compressed input.scss output.css  # Minified, production
```

---

#### Build Tool Integration

**Vite (Modern, Recommended) ✅**

```bash
# Install
npm install --save-dev sass

# Vite automatically compiles .scss files
# Just import in your JS/TS:
import './styles/main.scss'
```

**Webpack:**

```bash
# Install loaders
npm install --save-dev sass sass-loader css-loader style-loader

# webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.scss$/,
        use: ['style-loader', 'css-loader', 'sass-loader']
      }
    ]
  }
}
```

**Package.json Scripts:**

```json
{
  "scripts": {
    "sass": "sass scss:css",
    "sass:watch": "sass --watch scss:css",
    "sass:build": "sass --style=compressed scss:css"
  }
}
```

---

#### Source Maps

Source maps help debug compiled CSS by mapping it back to original SASS files.

```bash
# Generate source maps
sass --source-map scss/main.scss css/main.css

# Embed source maps in CSS file
sass --embed-source-map scss/main.scss css/main.css
```

**In Browser DevTools:**

```
Instead of:  main.css:45
You see:     _buttons.scss:12  ✅
```

---

### 1.3 Import, Use, and Advanced Architecture

#### The Old Way: `@import` (Deprecated) ❌

```scss
// _variables.scss
$primary: #3498db;

// main.scss
@import 'variables';  // ❌ Deprecated

.button {
  background: $primary;
}
```

**Problems with `@import`:**

- ❌ Global namespace pollution
- ❌ No encapsulation
- ❌ Multiple imports create duplicate CSS
- ❌ No control over what gets imported
- ❌ Slow compilation

---

#### The Modern Way: `@use` (Recommended) ✅

```scss
// _variables.scss
$primary: #3498db;
$secondary: #2ecc71;

// main.scss
@use 'variables';  // ✅ Modern

.button {
  background: variables.$primary;  // Namespaced!
}
```

**Benefits of `@use`:**

- ✅ Namespaced by default (no conflicts)
- ✅ Loaded once (no duplicates)
- ✅ Explicit imports
- ✅ Better performance
- ✅ Encapsulation

---

#### Namespace Control

**Default Namespace (filename):**

```scss
@use 'variables';  // Namespace: variables

.button {
  color: variables.$primary;
}
```

**Custom Namespace:**

```scss
@use 'variables' as vars;  // Namespace: vars

.button {
  color: vars.$primary;
}
```

**No Namespace (use with caution):**

```scss
@use 'variables' as *;  // No namespace

.button {
  color: $primary;  // Direct access
}
```

> [!CAUTION]
> Using `as *` defeats the purpose of namespacing. Only use for utility files.

---

#### `@forward` Directive

`@forward` creates a "barrel file" that re-exports multiple modules.

```scss
// _index.scss (barrel file)
@forward 'variables';
@forward 'mixins';
@forward 'functions';

// main.scss
@use 'index' as *;  // Import everything

.button {
  color: $primary;           // from variables
  @include button-style;     // from mixins
  width: calculate-width();  // from functions
}
```

**Selective Forwarding:**

```scss
// Only forward specific members
@forward 'variables' show $primary, $secondary;
@forward 'variables' hide $internal-var;

// Add prefix to forwarded members
@forward 'theme' as theme-*;
// $primary becomes $theme-primary
```

---

#### Advanced Architecture: 7-1 Pattern

The **7-1 Pattern** organizes SASS into 7 folders and 1 main file.

```
scss/
│
├── abstracts/          # Variables, mixins, functions
│   ├── _variables.scss
│   ├── _mixins.scss
│   ├── _functions.scss
│   └── _index.scss
│
├── base/               # Reset, typography, base styles
│   ├── _reset.scss
│   ├── _typography.scss
│   └── _index.scss
│
├── components/         # Buttons, cards, forms, etc.
│   ├── _buttons.scss
│   ├── _cards.scss
│   ├── _forms.scss
│   └── _index.scss
│
├── layout/             # Header, footer, grid, sidebar
│   ├── _header.scss
│   ├── _footer.scss
│   ├── _grid.scss
│   └── _index.scss
│
├── pages/              # Page-specific styles
│   ├── _home.scss
│   ├── _about.scss
│   └── _index.scss
│
├── themes/             # Theme variations
│   ├── _dark.scss
│   ├── _light.scss
│   └── _index.scss
│
├── vendors/            # Third-party CSS
│   ├── _normalize.scss
│   └── _index.scss
│
└── main.scss           # Main file that imports everything
```

**main.scss:**

```scss
// Configuration and helpers
@use 'abstracts' as *;

// Base styles
@use 'base';

// Layout
@use 'layout';

// Components
@use 'components';

// Pages
@use 'pages';

// Themes
@use 'themes';

// Vendors (if needed)
@use 'vendors';
```

---

#### Partials and File Naming

**Partials** are SASS files that start with an underscore (`_`).

```
_variables.scss  ✅ Partial (won't compile to CSS)
variables.scss   ❌ Will compile to variables.css
```

**Naming Conventions:**

```scss
// Import without underscore or extension
@use 'variables';        // Loads _variables.scss
@use 'mixins/buttons';   // Loads mixins/_buttons.scss
```

---

## 📦 Part 2: Variables

### 2.1 Variable Declaration and Syntax

Variables store reusable values.

**Syntax:**

```scss
$variable-name: value;
```

**Examples:**

```scss
// Colors
$primary-color: #3498db;
$secondary-color: #2ecc71;
$text-color: #333;

// Typography
$font-family: 'Helvetica Neue', sans-serif;
$font-size-base: 16px;
$line-height: 1.5;

// Spacing
$spacing-unit: 8px;
$padding-small: $spacing-unit;
$padding-medium: $spacing-unit * 2;
$padding-large: $spacing-unit * 3;

// Breakpoints
$breakpoint-mobile: 480px;
$breakpoint-tablet: 768px;
$breakpoint-desktop: 1024px;

// Usage
.button {
  background: $primary-color;
  font-family: $font-family;
  padding: $padding-medium;
}
```

---

### 2.2 Variable Scope

Variables can be **global** or **local**.

**Global Variables:**

```scss
// Top-level variables are global
$primary: #3498db;

.button {
  background: $primary;  // ✅ Works
}

.card {
  border-color: $primary;  // ✅ Works
}
```

**Local Variables:**

```scss
.button {
  $button-padding: 10px;  // Local to .button
  padding: $button-padding;
}

.card {
  padding: $button-padding;  // ❌ Error: Undefined variable
}
```

**Shadowing (Local overrides global):**

```scss
$color: blue;  // Global

.button {
  $color: red;  // Local (shadows global)
  background: $color;  // red
}

.card {
  background: $color;  // blue (global)
}
```

---

### 2.3 Default Values with `!default`

`!default` sets a variable **only if it's not already defined**.

```scss
// _variables.scss
$primary: #3498db !default;
$secondary: #2ecc71 !default;

// main.scss
$primary: #e74c3c;  // Override before import

@use 'variables';

.button {
  background: variables.$primary;  // #e74c3c (overridden)
  color: variables.$secondary;     // #2ecc71 (default)
}
```

**Use Case: Library Configuration**

```scss
// _library.scss (your library)
$lib-primary: #3498db !default;
$lib-spacing: 8px !default;

// user.scss (user's code)
$lib-primary: #e74c3c;  // Customize

@use 'library';
// Library uses user's $lib-primary value
```

---

### 2.4 Variable Types

SASS supports multiple data types:

**Numbers:**

```scss
$width: 100px;
$opacity: 0.8;
$z-index: 10;
$percentage: 50%;
$unitless: 1.5;
```

**Strings:**

```scss
$font-family: 'Helvetica Neue';
$font-family-unquoted: Helvetica;
$url: 'images/bg.jpg';
```

**Colors:**

```scss
$color-hex: #3498db;
$color-rgb: rgb(52, 152, 219);
$color-rgba: rgba(52, 152, 219, 0.8);
$color-hsl: hsl(204, 70%, 53%);
$color-name: blue;
```

**Booleans:**

```scss
$is-rounded: true;
$has-shadow: false;
```

**Null:**

```scss
$border: null;  // Means "no value"

.button {
  border: $border;  // Property not rendered
}
```

**Lists (Arrays):**

```scss
$sizes: 10px 20px 30px;
$fonts: 'Helvetica', 'Arial', sans-serif;
$mixed: 1px solid red;

// Access with nth()
.button {
  padding: nth($sizes, 1);  // 10px
  font-family: $fonts;
}
```

**Maps (Objects):**

```scss
$colors: (
  primary: #3498db,
  secondary: #2ecc71,
  danger: #e74c3c
);

$breakpoints: (
  sm: 576px,
  md: 768px,
  lg: 992px,
  xl: 1200px
);

// Access with map-get()
.button {
  background: map-get($colors, primary);
}
```

---

### 2.5 CSS Custom Properties vs SASS Variables

**SASS Variables (Compile-time):**

```scss
$primary: #3498db;

.button {
  background: $primary;  // Replaced at compile time
}

// Compiled CSS:
.button {
  background: #3498db;
}
```

**CSS Custom Properties (Runtime):**

```scss
:root {
  --primary: #3498db;
}

.button {
  background: var(--primary);  // Evaluated at runtime
}

// Can be changed with JavaScript!
document.documentElement.style.setProperty('--primary', '#e74c3c');
```

**When to Use Each:**

| Use SASS Variables | Use CSS Custom Properties |
|-------------------|---------------------------|
| Static values | Dynamic values (JS changes) |
| Calculations | Theme switching |
| Build-time logic | Browser-level cascading |
| Better performance | Better for theming |

**Best Practice: Use Both!**

```scss
// SASS variables for configuration
$primary: #3498db;
$secondary: #2ecc71;

// CSS custom properties for runtime
:root {
  --color-primary: #{$primary};
  --color-secondary: #{$secondary};
}

.button {
  background: var(--color-primary);
}
```

---

## 🎯 Part 3: Nesting and Parent Selector

### 3.1 Selector Nesting

Nesting mirrors HTML structure.

**Without Nesting (Plain CSS):**

```css
.nav {
  background: #333;
}

.nav ul {
  list-style: none;
}

.nav ul li {
  display: inline-block;
}

.nav ul li a {
  color: white;
  text-decoration: none;
}
```

**With Nesting (SCSS):**

```scss
.nav {
  background: #333;
  
  ul {
    list-style: none;
    
    li {
      display: inline-block;
      
      a {
        color: white;
        text-decoration: none;
      }
    }
  }
}
```

**Compiled Output:**

```css
.nav {
  background: #333;
}
.nav ul {
  list-style: none;
}
.nav ul li {
  display: inline-block;
}
.nav ul li a {
  color: white;
  text-decoration: none;
}
```

---

### 3.2 Parent Selector (`&`)

The `&` references the parent selector.

**Pseudo-classes:**

```scss
.button {
  background: blue;
  
  &:hover {
    background: darkblue;
  }
  
  &:active {
    background: navy;
  }
  
  &:disabled {
    opacity: 0.5;
  }
}

// Compiled:
.button { background: blue; }
.button:hover { background: darkblue; }
.button:active { background: navy; }
.button:disabled { opacity: 0.5; }
```

**Pseudo-elements:**

```scss
.button {
  position: relative;
  
  &::before {
    content: '';
    position: absolute;
  }
  
  &::after {
    content: '';
    position: absolute;
  }
}
```

**Modifiers (BEM):**

```scss
.button {
  padding: 10px;
  
  &--large {
    padding: 20px;
  }
  
  &--small {
    padding: 5px;
  }
}

// Compiled:
.button { padding: 10px; }
.button--large { padding: 20px; }
.button--small { padding: 5px; }
```

**Compound Selectors:**

```scss
.button {
  background: blue;
  
  &.is-active {
    background: green;
  }
  
  &.is-disabled {
    opacity: 0.5;
  }
}

// Compiled:
.button { background: blue; }
.button.is-active { background: green; }
.button.is-disabled { opacity: 0.5; }
```

**Parent Context:**

```scss
.button {
  background: blue;
  
  .sidebar & {
    background: gray;
  }
  
  .dark-theme & {
    background: black;
  }
}

// Compiled:
.button { background: blue; }
.sidebar .button { background: gray; }
.dark-theme .button { background: black; }
```

---

### 3.3 BEM Methodology with SASS

**BEM (Block Element Modifier)** is a naming convention.

**Structure:**

```
.block {}
.block__element {}
.block--modifier {}
.block__element--modifier {}
```

**SASS Implementation:**

```scss
.card {
  // Block
  background: white;
  border: 1px solid #ddd;
  
  &__header {
    // Element: .card__header
    padding: 20px;
    border-bottom: 1px solid #ddd;
  }
  
  &__title {
    // Element: .card__title
    font-size: 24px;
    margin: 0;
  }
  
  &__body {
    // Element: .card__body
    padding: 20px;
  }
  
  &__footer {
    // Element: .card__footer
    padding: 20px;
    border-top: 1px solid #ddd;
  }
  
  &--featured {
    // Modifier: .card--featured
    border-color: gold;
    box-shadow: 0 4px 8px rgba(0,0,0,0.1);
  }
  
  &--compact {
    // Modifier: .card--compact
    
    .card__header,
    .card__body,
    .card__footer {
      padding: 10px;
    }
  }
}
```

**Compiled CSS:**

```css
.card { background: white; border: 1px solid #ddd; }
.card__header { padding: 20px; border-bottom: 1px solid #ddd; }
.card__title { font-size: 24px; margin: 0; }
.card__body { padding: 20px; }
.card__footer { padding: 20px; border-top: 1px solid #ddd; }
.card--featured { border-color: gold; box-shadow: 0 4px 8px rgba(0,0,0,0.1); }
.card--compact .card__header,
.card--compact .card__body,
.card--compact .card__footer { padding: 10px; }
```

---

### 3.4 Media Query Nesting

Nest media queries inside selectors.

```scss
.sidebar {
  width: 300px;
  
  @media (max-width: 768px) {
    width: 100%;
  }
  
  @media (max-width: 480px) {
    display: none;
  }
}

// Compiled:
.sidebar {
  width: 300px;
}
@media (max-width: 768px) {
  .sidebar {
    width: 100%;
  }
}
@media (max-width: 480px) {
  .sidebar {
    display: none;
  }
}
```

---

### 3.5 Nesting Best Practices

> [!WARNING]
> **Avoid deep nesting!** Maximum 3-4 levels.

**❌ Bad (Too deep):**

```scss
.nav {
  ul {
    li {
      a {
        span {
          i {
            // 6 levels deep! ❌
          }
        }
      }
    }
  }
}
```

**✅ Good (Flat structure):**

```scss
.nav {
  // Block styles
}

.nav__list {
  // Element styles
}

.nav__item {
  // Element styles
}

.nav__link {
  // Element styles
  
  &:hover {
    // Only nest pseudo-classes
  }
}
```

**Best Practices:**

1. ✅ Nest pseudo-classes and pseudo-elements
2. ✅ Nest media queries
3. ✅ Nest modifiers with `&`
4. ❌ Avoid nesting beyond 3-4 levels
5. ❌ Don't nest just because you can

---

## 🎨 Part 4: Property Declarations and Placeholders

### 4.1 Property Nesting

Group related properties with a common namespace.

**Font Properties:**

```scss
.text {
  font: {
    family: 'Helvetica';
    size: 16px;
    weight: bold;
    style: italic;
  }
}

// Compiled:
.text {
  font-family: 'Helvetica';
  font-size: 16px;
  font-weight: bold;
  font-style: italic;
}
```

**Border Properties:**

```scss
.box {
  border: {
    top: 1px solid red;
    bottom: 2px dashed blue;
    left: {
      width: 3px;
      style: dotted;
      color: green;
    }
  }
}

// Compiled:
.box {
  border-top: 1px solid red;
  border-bottom: 2px dashed blue;
  border-left-width: 3px;
  border-left-style: dotted;
  border-left-color: green;
}
```

**Background Properties:**

```scss
.hero {
  background: {
    image: url('hero.jpg');
    size: cover;
    position: center;
    repeat: no-repeat;
  }
}
```

> [!TIP]
> Property nesting is rarely used in practice. Regular properties are more readable.

---

### 4.2 Placeholder Selectors (`%`)

Placeholders are **reusable style templates** that don't generate CSS until extended.

**Syntax:**

```scss
%placeholder-name {
  // Styles
}

.selector {
  @extend %placeholder-name;
}
```

**Example:**

```scss
// Define placeholder
%button-base {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

// Extend placeholder
.button-primary {
  @extend %button-base;
  background: blue;
  color: white;
}

.button-secondary {
  @extend %button-base;
  background: gray;
  color: white;
}

.button-danger {
  @extend %button-base;
  background: red;
  color: white;
}
```

**Compiled CSS:**

```css
.button-primary, .button-secondary, .button-danger {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

.button-primary {
  background: blue;
  color: white;
}

.button-secondary {
  background: gray;
  color: white;
}

.button-danger {
  background: red;
  color: white;
}
```

**Notice:** Shared styles are grouped together! ✅

---

### 4.3 `@extend` Directive

`@extend` shares styles between selectors.

**Basic Usage:**

```scss
.error {
  border: 1px solid red;
  background: #ffe6e6;
  color: red;
}

.critical-error {
  @extend .error;
  font-weight: bold;
  font-size: 18px;
}

// Compiled:
.error, .critical-error {
  border: 1px solid red;
  background: #ffe6e6;
  color: red;
}

.critical-error {
  font-weight: bold;
  font-size: 18px;
}
```

---

### 4.4 Placeholder vs Mixin

| Feature | Placeholder (`%`) | Mixin (`@mixin`) |
|---------|------------------|------------------|
| **Output** | Grouped selectors | Duplicated styles |
| **Parameters** | ❌ No | ✅ Yes |
| **File Size** | Smaller | Larger |
| **Use Case** | Static shared styles | Dynamic styles |

**Placeholder Example:**

```scss
%card {
  border: 1px solid #ddd;
  padding: 20px;
}

.card-1 { @extend %card; }
.card-2 { @extend %card; }

// Compiled (grouped):
.card-1, .card-2 {
  border: 1px solid #ddd;
  padding: 20px;
}
```

**Mixin Example:**

```scss
@mixin card($padding) {
  border: 1px solid #ddd;
  padding: $padding;
}

.card-1 { @include card(20px); }
.card-2 { @include card(30px); }

// Compiled (duplicated):
.card-1 {
  border: 1px solid #ddd;
  padding: 20px;
}
.card-2 {
  border: 1px solid #ddd;
  padding: 30px;
}
```

**When to Use:**

- ✅ Use **placeholders** for static shared styles
- ✅ Use **mixins** for dynamic styles with parameters

---

### 4.5 Performance Implications

**`@extend` Can Cause Issues:**

```scss
.button { padding: 10px; }
.icon { font-size: 16px; }

.button-icon {
  @extend .button;
  @extend .icon;
}

// Compiled (can create unexpected selectors):
.button, .button-icon { padding: 10px; }
.icon, .button-icon { font-size: 16px; }
```

**Complex Selectors:**

```scss
.nav .button { padding: 10px; }

.sidebar-button {
  @extend .button;  // Creates: .nav .sidebar-button
}
```

> [!CAUTION]
> `@extend` can create bloated CSS with complex selectors. Use sparingly!

**Best Practice:**

```scss
// ✅ Good: Extend placeholders only
%button-base { padding: 10px; }

.button { @extend %button-base; }
.link-button { @extend %button-base; }

// ❌ Avoid: Extending classes
.button { padding: 10px; }
.link-button { @extend .button; }  // Can cause issues
```

---

## 🔀 Part 5: Control Flow

### 5.1 `@if` Directive

Conditionally include styles.

**Syntax:**

```scss
@if condition {
  // Styles
}
```

**Example:**

```scss
$theme: dark;

.button {
  @if $theme == dark {
    background: #333;
    color: white;
  }
}

// Compiled:
.button {
  background: #333;
  color: white;
}
```

---

### 5.2 `@else if` and `@else`

**Syntax:**

```scss
@if condition1 {
  // Styles
} @else if condition2 {
  // Styles
} @else {
  // Styles
}
```

**Example:**

```scss
$size: large;

.button {
  @if $size == small {
    padding: 5px 10px;
    font-size: 12px;
  } @else if $size == medium {
    padding: 10px 20px;
    font-size: 16px;
  } @else if $size == large {
    padding: 15px 30px;
    font-size: 20px;
  } @else {
    padding: 10px 20px;
    font-size: 16px;
  }
}

// Compiled:
.button {
  padding: 15px 30px;
  font-size: 20px;
}
```

---

### 5.3 Comparison Operators

```scss
==   // Equal
!=   // Not equal
<    // Less than
<=   // Less than or equal
>    // Greater than
>=   // Greater than or equal
```

**Examples:**

```scss
$width: 100px;

@if $width > 50px {
  // True
}

@if $width == 100px {
  // True
}

@if $width != 200px {
  // True
}
```

---

### 5.4 Logical Operators

```scss
and   // Both conditions true
or    // At least one condition true
not   // Negate condition
```

**Examples:**

```scss
$width: 100px;
$height: 200px;

@if $width > 50px and $height > 100px {
  // Both true
}

@if $width > 50px or $height < 100px {
  // At least one true
}

@if not ($width == 50px) {
  // Negation
}
```

---

### 5.5 Truthy and Falsy Values

**Falsy Values:**

```scss
false
null
```

**Truthy Values (everything else):**

```scss
true
0          // ⚠️ Truthy in SASS!
""         // ⚠️ Truthy in SASS!
()         // Empty list is truthy
```

**Examples:**

```scss
@if 0 {
  // This runs! 0 is truthy in SASS
}

@if "" {
  // This runs! Empty string is truthy
}

@if null {
  // This doesn't run
}

@if false {
  // This doesn't run
}
```

---

### 5.6 Practical Use Cases

**Theme Switching:**

```scss
$theme: dark;

.app {
  @if $theme == dark {
    background: #1a1a1a;
    color: #ffffff;
  } @else {
    background: #ffffff;
    color: #000000;
  }
}
```

**Responsive Font Sizes:**

```scss
@mixin font-size($size) {
  @if $size == small {
    font-size: 12px;
  } @else if $size == medium {
    font-size: 16px;
  } @else if $size == large {
    font-size: 20px;
  } @else {
    font-size: $size;  // Custom size
  }
}

.text {
  @include font-size(large);
}
```

**Feature Flags:**

```scss
$enable-shadows: true;
$enable-gradients: false;

.card {
  background: white;
  
  @if $enable-shadows {
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  }
  
  @if $enable-gradients {
    background: linear-gradient(to bottom, white, #f5f5f5);
  }
}
```

---

### 5.7 Create Triangle with If and Else

**Triangle Mixin:**

```scss
@mixin triangle($direction, $size, $color) {
  width: 0;
  height: 0;
  border-style: solid;
  
  @if $direction == up {
    border-width: 0 ($size / 2) $size ($size / 2);
    border-color: transparent transparent $color transparent;
  } @else if $direction == down {
    border-width: $size ($size / 2) 0 ($size / 2);
    border-color: $color transparent transparent transparent;
  } @else if $direction == left {
    border-width: ($size / 2) $size ($size / 2) 0;
    border-color: transparent $color transparent transparent;
  } @else if $direction == right {
    border-width: ($size / 2) 0 ($size / 2) $size;
    border-color: transparent transparent transparent $color;
  } @else {
    @error "Direction must be up, down, left, or right";
  }
}

// Usage
.arrow-up {
  @include triangle(up, 20px, red);
}

.arrow-down {
  @include triangle(down, 20px, blue);
}

.arrow-left {
  @include triangle(left, 20px, green);
}

.arrow-right {
  @include triangle(right, 20px, orange);
}
```

**Compiled CSS:**

```css
.arrow-up {
  width: 0;
  height: 0;
  border-style: solid;
  border-width: 0 10px 20px 10px;
  border-color: transparent transparent red transparent;
}

.arrow-down {
  width: 0;
  height: 0;
  border-style: solid;
  border-width: 20px 10px 0 10px;
  border-color: blue transparent transparent transparent;
}

/* ... etc */
```

---

## 🔤 Part 6: Interpolation

### 6.1 Interpolation Syntax

**Syntax:** `#{expression}`

Interpolation evaluates a SASS expression and inserts the result as a string.

---

### 6.2 Interpolation in Selectors

```scss
$name: button;

.#{$name} {
  padding: 10px;
}

.#{$name}-primary {
  background: blue;
}

.#{$name}-secondary {
  background: gray;
}

// Compiled:
.button { padding: 10px; }
.button-primary { background: blue; }
.button-secondary { background: gray; }
```

**Dynamic Class Generation:**

```scss
$sizes: small, medium, large;

@each $size in $sizes {
  .button-#{$size} {
    @if $size == small {
      padding: 5px 10px;
    } @else if $size == medium {
      padding: 10px 20px;
    } @else {
      padding: 15px 30px;
    }
  }
}

// Compiled:
.button-small { padding: 5px 10px; }
.button-medium { padding: 10px 20px; }
.button-large { padding: 15px 30px; }
```

---

### 6.3 Interpolation in Property Names

```scss
$side: left;

.box {
  margin-#{$side}: 20px;
  padding-#{$side}: 10px;
}

// Compiled:
.box {
  margin-left: 20px;
  padding-left: 10px;
}
```

**Generate Spacing Utilities:**

```scss
$sides: top, right, bottom, left;

@each $side in $sides {
  .m-#{$side} {
    margin-#{$side}: 10px;
  }
  
  .p-#{$side} {
    padding-#{$side}: 10px;
  }
}

// Compiled:
.m-top { margin-top: 10px; }
.p-top { padding-top: 10px; }
.m-right { margin-right: 10px; }
.p-right { padding-right: 10px; }
/* ... etc */
```

---

### 6.4 Interpolation in Strings

```scss
$version: '1.0.0';

.app::before {
  content: 'Version: #{$version}';
}

// Compiled:
.app::before {
  content: 'Version: 1.0.0';
}
```

**URL Interpolation:**

```scss
$image-path: 'images/';
$image-name: 'hero.jpg';

.hero {
  background-image: url('#{$image-path}#{$image-name}');
}

// Compiled:
.hero {
  background-image: url('images/hero.jpg');
}
```

---

### 6.5 Interpolation with CSS Custom Properties

```scss
$primary: #3498db;

:root {
  --color-primary: #{$primary};
  --spacing-unit: #{8px};
}

.button {
  background: var(--color-primary);
  padding: var(--spacing-unit);
}
```

> [!IMPORTANT]
> Use `#{}` to convert SASS variables to CSS custom property values.

---

### 6.6 Use Cases and Examples

**Breakpoint Mixin:**

```scss
$breakpoints: (
  sm: 576px,
  md: 768px,
  lg: 992px,
  xl: 1200px
);

@mixin respond-to($breakpoint) {
  @media (min-width: #{map-get($breakpoints, $breakpoint)}) {
    @content;
  }
}

// Usage
.container {
  width: 100%;
  
  @include respond-to(md) {
    width: 750px;
  }
  
  @include respond-to(lg) {
    width: 970px;
  }
}
```

**Grid System:**

```scss
@for $i from 1 through 12 {
  .col-#{$i} {
    width: percentage($i / 12);
  }
}

// Compiled:
.col-1 { width: 8.3333333333%; }
.col-2 { width: 16.6666666667%; }
/* ... */
.col-12 { width: 100%; }
```

---

## 💬 Part 7: Comments and Documenting

### 7.1 Silent Comments (`//`)

**Not included in compiled CSS.**

```scss
// This is a silent comment
// It won't appear in the CSS output

.button {
  padding: 10px;  // Inline comment
}
```

**Compiled CSS:**

```css
.button {
  padding: 10px;
}
```

---

### 7.2 Loud Comments (`/* */`)

**Included in compiled CSS** (except in compressed mode).

```scss
/* This is a loud comment */
/* It will appear in the CSS output */

.button {
  padding: 10px;  /* Inline loud comment */
}
```

**Compiled CSS:**

```css
/* This is a loud comment */
/* It will appear in the CSS output */
.button {
  padding: 10px;  /* Inline loud comment */
}
```

**Compressed Mode:**

```bash
sass --style=compressed input.scss output.css
```

```css
.button{padding:10px}
```

---

### 7.3 Documentation Comments

**SassDoc Syntax:**

```scss
/// Button mixin for creating styled buttons
/// @param {Color} $bg-color - Background color
/// @param {Color} $text-color - Text color
/// @param {Number} $padding - Padding value
/// @example scss - Basic usage
///   .my-button {
///     @include button(blue, white, 10px);
///   }
@mixin button($bg-color, $text-color, $padding) {
  background: $bg-color;
  color: $text-color;
  padding: $padding;
  border: none;
  border-radius: 4px;
}
```

---

### 7.4 Best Practices for Documentation

**File Headers:**

```scss
//
// Buttons
// --------------------------------------------------
// Button styles and variations
//

$button-padding: 10px 20px;
$button-radius: 4px;

.button {
  // Base button styles
}
```

**Section Comments:**

```scss
// ==========================================================================
// Variables
// ==========================================================================

$primary: #3498db;
$secondary: #2ecc71;

// ==========================================================================
// Mixins
// ==========================================================================

@mixin button-variant($color) {
  // ...
}
```

**Inline Explanations:**

```scss
.button {
  // Use flexbox for centering
  display: flex;
  align-items: center;
  justify-content: center;
  
  // Prevent text selection
  user-select: none;
}
```

---

## 🎨 Part 8: Mixins and Include

### 8.1 `@mixin` Directive

Mixins are **reusable blocks of styles** that can accept parameters.

**Basic Syntax:**

```scss
@mixin mixin-name {
  // Styles
}

.selector {
  @include mixin-name;
}
```

**Example:**

```scss
@mixin reset-list {
  margin: 0;
  padding: 0;
  list-style: none;
}

.nav-list {
  @include reset-list;
}

.sidebar-list {
  @include reset-list;
}

// Compiled:
.nav-list {
  margin: 0;
  padding: 0;
  list-style: none;
}

.sidebar-list {
  margin: 0;
  padding: 0;
  list-style: none;
}
```

---

### 8.2 Mixin Parameters

**Single Parameter:**

```scss
@mixin border-radius($radius) {
  -webkit-border-radius: $radius;
  -moz-border-radius: $radius;
  border-radius: $radius;
}

.button {
  @include border-radius(10px);
}

// Compiled:
.button {
  -webkit-border-radius: 10px;
  -moz-border-radius: 10px;
  border-radius: 10px;
}
```

**Multiple Parameters:**

```scss
@mixin box($width, $height, $bg-color) {
  width: $width;
  height: $height;
  background: $bg-color;
}

.square {
  @include box(100px, 100px, red);
}

.rectangle {
  @include box(200px, 100px, blue);
}
```

---

### 8.3 Default Parameter Values

```scss
@mixin button($bg-color: blue, $text-color: white, $padding: 10px 20px) {
  background: $bg-color;
  color: $text-color;
  padding: $padding;
  border: none;
  border-radius: 4px;
}

// Use defaults
.button-default {
  @include button;
}

// Override some parameters
.button-custom {
  @include button(red);
}

// Override all parameters
.button-full {
  @include button(green, black, 15px 30px);
}

// Named parameters
.button-named {
  @include button($text-color: yellow, $bg-color: purple);
}
```

---

### 8.4 Variable Arguments (`...`)

Accept unlimited arguments.

```scss
@mixin box-shadow($shadows...) {
  -webkit-box-shadow: $shadows;
  -moz-box-shadow: $shadows;
  box-shadow: $shadows;
}

.card {
  @include box-shadow(
    0 2px 4px rgba(0,0,0,0.1),
    0 4px 8px rgba(0,0,0,0.1),
    0 8px 16px rgba(0,0,0,0.1)
  );
}

// Compiled:
.card {
  -webkit-box-shadow: 0 2px 4px rgba(0,0,0,0.1), 0 4px 8px rgba(0,0,0,0.1), 0 8px 16px rgba(0,0,0,0.1);
  -moz-box-shadow: 0 2px 4px rgba(0,0,0,0.1), 0 4px 8px rgba(0,0,0,0.1), 0 8px 16px rgba(0,0,0,0.1);
  box-shadow: 0 2px 4px rgba(0,0,0,0.1), 0 4px 8px rgba(0,0,0,0.1), 0 8px 16px rgba(0,0,0,0.1);
}
```

---

### 8.5 Content Blocks (`@content`)

Pass content to mixins.

```scss
@mixin media($breakpoint) {
  @media (min-width: $breakpoint) {
    @content;
  }
}

.container {
  width: 100%;
  
  @include media(768px) {
    width: 750px;
  }
  
  @include media(992px) {
    width: 970px;
  }
}

// Compiled:
.container {
  width: 100%;
}
@media (min-width: 768px) {
  .container {
    width: 750px;
  }
}
@media (min-width: 992px) {
  .container {
    width: 970px;
  }
}
```

---

### 8.6 Practical Mixin Examples

**Flexbox Centering:**

```scss
@mixin flex-center($direction: row) {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: $direction;
}

.modal {
  @include flex-center;
}

.sidebar {
  @include flex-center(column);
}
```

**Truncate Text:**

```scss
@mixin truncate($width: 100%) {
  width: $width;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.title {
  @include truncate(200px);
}
```

**Aspect Ratio:**

```scss
@mixin aspect-ratio($width, $height) {
  position: relative;
  
  &::before {
    content: '';
    display: block;
    padding-top: percentage($height / $width);
  }
}

.video-container {
  @include aspect-ratio(16, 9);
}
```

---

## 🔁 Part 9: Loops

### 9.1 `@for` Loop

Repeat styles with a counter.

**Syntax:**

```scss
// from...through (inclusive)
@for $i from 1 through 5 {
  // $i = 1, 2, 3, 4, 5
}

// from...to (exclusive)
@for $i from 1 to 5 {
  // $i = 1, 2, 3, 4
}
```

**Example: Spacing Utilities**

```scss
@for $i from 1 through 5 {
  .m-#{$i} {
    margin: #{$i * 10}px;
  }
  
  .p-#{$i} {
    padding: #{$i * 10}px;
  }
}

// Compiled:
.m-1 { margin: 10px; }
.p-1 { padding: 10px; }
.m-2 { margin: 20px; }
.p-2 { padding: 20px; }
.m-3 { margin: 30px; }
.p-3 { padding: 30px; }
.m-4 { margin: 40px; }
.p-4 { padding: 40px; }
.m-5 { margin: 50px; }
.p-5 { padding: 50px; }
```

**Example: Grid Columns**

```scss
@for $i from 1 through 12 {
  .col-#{$i} {
    width: percentage($i / 12);
  }
}

// Compiled:
.col-1 { width: 8.3333333333%; }
.col-2 { width: 16.6666666667%; }
.col-3 { width: 25%; }
// ... up to .col-12
```

**Example: Z-Index Layers**

```scss
@for $i from 1 through 10 {
  .z-#{$i} {
    z-index: $i * 10;
  }
}

// Compiled:
.z-1 { z-index: 10; }
.z-2 { z-index: 20; }
// ... up to .z-10 { z-index: 100; }
```

---

### 9.2 `@each` Loop

Iterate over lists or maps.

**Syntax (List):**

```scss
@each $item in $list {
  // Use $item
}
```

**Example: Color Utilities**

```scss
$colors: red, green, blue, yellow;

@each $color in $colors {
  .text-#{$color} {
    color: $color;
  }
  
  .bg-#{$color} {
    background: $color;
  }
}

// Compiled:
.text-red { color: red; }
.bg-red { background: red; }
.text-green { color: green; }
.bg-green { background: green; }
// ... etc
```

---

### 9.3 `@each` with Maps

**Syntax (Map):**

```scss
@each $key, $value in $map {
  // Use $key and $value
}
```

**Example: Theme Colors**

```scss
$theme-colors: (
  primary: #3498db,
  secondary: #2ecc71,
  danger: #e74c3c,
  warning: #f39c12,
  info: #3498db,
  success: #2ecc71
);

@each $name, $color in $theme-colors {
  .btn-#{$name} {
    background: $color;
    color: white;
  }
  
  .text-#{$name} {
    color: $color;
  }
  
  .border-#{$name} {
    border-color: $color;
  }
}

// Compiled:
.btn-primary { background: #3498db; color: white; }
.text-primary { color: #3498db; }
.border-primary { border-color: #3498db; }
// ... etc
```

**Example: Breakpoints**

```scss
$breakpoints: (
  sm: 576px,
  md: 768px,
  lg: 992px,
  xl: 1200px,
  xxl: 1400px
);

@each $name, $width in $breakpoints {
  .container-#{$name} {
    @media (min-width: $width) {
      max-width: $width;
    }
  }
}
```

---

### 9.4 Destructuring in Loops

**Multiple Values per Item:**

```scss
$sizes: (small, 10px, 12px), (medium, 16px, 18px), (large, 20px, 24px);

@each $name, $font-size, $line-height in $sizes {
  .text-#{$name} {
    font-size: $font-size;
    line-height: $line-height;
  }
}

// Compiled:
.text-small { font-size: 10px; line-height: 12px; }
.text-medium { font-size: 16px; line-height: 18px; }
.text-large { font-size: 20px; line-height: 24px; }
```

---

### 9.5 Multi-Dimensional Maps

```scss
$social-colors: (
  facebook: (
    base: #3b5998,
    hover: darken(#3b5998, 10%)
  ),
  twitter: (
    base: #1da1f2,
    hover: darken(#1da1f2, 10%)
  ),
  instagram: (
    base: #e4405f,
    hover: darken(#e4405f, 10%)
  )
);

@each $network, $colors in $social-colors {
  .btn-#{$network} {
    background: map-get($colors, base);
    
    &:hover {
      background: map-get($colors, hover);
    }
  }
}
```

---

### 9.6 `@while` Loop

Loop while a condition is true.

**Syntax:**

```scss
@while condition {
  // Styles
}
```

**Example: Exponential Scale**

```scss
$i: 1;

@while $i <= 5 {
  .font-#{$i} {
    font-size: #{$i * $i}px;
  }
  
  $i: $i + 1;
}

// Compiled:
.font-1 { font-size: 1px; }
.font-2 { font-size: 4px; }
.font-3 { font-size: 9px; }
.font-4 { font-size: 16px; }
.font-5 { font-size: 25px; }
```

**Example: Fibonacci Sequence**

```scss
$fibonacci: 1 1;
$i: 1;

@while $i <= 8 {
  $prev: nth($fibonacci, length($fibonacci) - 1);
  $current: nth($fibonacci, length($fibonacci));
  $next: $prev + $current;
  $fibonacci: append($fibonacci, $next);
  $i: $i + 1;
}

// Use Fibonacci values
@for $i from 1 through length($fibonacci) {
  .spacing-#{$i} {
    margin: #{nth($fibonacci, $i)}px;
  }
}
```

---

### 9.7 Loop Performance Considerations

> [!WARNING]
> Loops generate CSS at compile time. Large loops can create bloated CSS files.

**❌ Bad (Generates 1000 classes):**

```scss
@for $i from 1 through 1000 {
  .width-#{$i} {
    width: #{$i}px;
  }
}
```

**✅ Good (Generate only what you need):**

```scss
$widths: 25, 50, 75, 100, 200, 300, 400, 500;

@each $width in $widths {
  .w-#{$width} {
    width: #{$width}px;
  }
}
```

---

## 🏗️ Part 10: Create Bootstrap Grid System

### 10.1 Grid System Architecture

**Requirements:**

- 12-column grid
- Responsive breakpoints
- Container, row, and column classes
- Flexbox-based
- Gutter spacing

---

### 10.2 Configuration Variables

```scss
// Grid settings
$grid-columns: 12;
$grid-gutter-width: 30px;

// Breakpoints
$breakpoints: (
  xs: 0,
  sm: 576px,
  md: 768px,
  lg: 992px,
  xl: 1200px,
  xxl: 1400px
);

// Container max widths
$container-max-widths: (
  sm: 540px,
  md: 720px,
  lg: 960px,
  xl: 1140px,
  xxl: 1320px
);
```

---

### 10.3 Container Classes

```scss
.container {
  width: 100%;
  padding-right: $grid-gutter-width / 2;
  padding-left: $grid-gutter-width / 2;
  margin-right: auto;
  margin-left: auto;
  
  // Responsive max-widths
  @each $breakpoint, $max-width in $container-max-widths {
    @media (min-width: map-get($breakpoints, $breakpoint)) {
      max-width: $max-width;
    }
  }
}

.container-fluid {
  width: 100%;
  padding-right: $grid-gutter-width / 2;
  padding-left: $grid-gutter-width / 2;
  margin-right: auto;
  margin-left: auto;
}
```

---

### 10.4 Row Class

```scss
.row {
  display: flex;
  flex-wrap: wrap;
  margin-right: -($grid-gutter-width / 2);
  margin-left: -($grid-gutter-width / 2);
}
```

---

### 10.5 Column Classes

```scss
// Base column styles
%col-base {
  position: relative;
  width: 100%;
  padding-right: $grid-gutter-width / 2;
  padding-left: $grid-gutter-width / 2;
}

// Generate column classes for each breakpoint
@each $breakpoint, $min-width in $breakpoints {
  @if $breakpoint == xs {
    // Extra small (no media query)
    @for $i from 1 through $grid-columns {
      .col-#{$i} {
        @extend %col-base;
        flex: 0 0 percentage($i / $grid-columns);
        max-width: percentage($i / $grid-columns);
      }
    }
    
    .col {
      @extend %col-base;
      flex-basis: 0;
      flex-grow: 1;
      max-width: 100%;
    }
  } @else {
    // Other breakpoints
    @media (min-width: $min-width) {
      @for $i from 1 through $grid-columns {
        .col-#{$breakpoint}-#{$i} {
          @extend %col-base;
          flex: 0 0 percentage($i / $grid-columns);
          max-width: percentage($i / $grid-columns);
        }
      }
      
      .col-#{$breakpoint} {
        @extend %col-base;
        flex-basis: 0;
        flex-grow: 1;
        max-width: 100%;
      }
    }
  }
}
```

---

### 10.6 Offset Classes

```scss
@each $breakpoint, $min-width in $breakpoints {
  @if $breakpoint == xs {
    @for $i from 0 through ($grid-columns - 1) {
      .offset-#{$i} {
        margin-left: percentage($i / $grid-columns);
      }
    }
  } @else {
    @media (min-width: $min-width) {
      @for $i from 0 through ($grid-columns - 1) {
        .offset-#{$breakpoint}-#{$i} {
          margin-left: percentage($i / $grid-columns);
        }
      }
    }
  }
}
```

---

### 10.7 Complete Grid System Example

```scss
// Grid System
// ============================================

// Configuration
$grid-columns: 12;
$grid-gutter-width: 30px;

$breakpoints: (
  xs: 0,
  sm: 576px,
  md: 768px,
  lg: 992px,
  xl: 1200px
);

$container-max-widths: (
  sm: 540px,
  md: 720px,
  lg: 960px,
  xl: 1140px
);

// Container
.container {
  width: 100%;
  padding-right: $grid-gutter-width / 2;
  padding-left: $grid-gutter-width / 2;
  margin-right: auto;
  margin-left: auto;
  
  @each $breakpoint, $max-width in $container-max-widths {
    @media (min-width: map-get($breakpoints, $breakpoint)) {
      max-width: $max-width;
    }
  }
}

// Row
.row {
  display: flex;
  flex-wrap: wrap;
  margin-right: -($grid-gutter-width / 2);
  margin-left: -($grid-gutter-width / 2);
}

// Columns
%col-base {
  position: relative;
  width: 100%;
  padding-right: $grid-gutter-width / 2;
  padding-left: $grid-gutter-width / 2;
}

@each $breakpoint, $min-width in $breakpoints {
  $infix: if($breakpoint == xs, '', '-#{$breakpoint}');
  
  @if $min-width == 0 {
    @for $i from 1 through $grid-columns {
      .col#{$infix}-#{$i} {
        @extend %col-base;
        flex: 0 0 percentage($i / $grid-columns);
        max-width: percentage($i / $grid-columns);
      }
    }
  } @else {
    @media (min-width: $min-width) {
      @for $i from 1 through $grid-columns {
        .col#{$infix}-#{$i} {
          @extend %col-base;
          flex: 0 0 percentage($i / $grid-columns);
          max-width: percentage($i / $grid-columns);
        }
      }
    }
  }
}
```

**Usage:**

```html
<div class="container">
  <div class="row">
    <div class="col-12 col-md-6 col-lg-4">Column 1</div>
    <div class="col-12 col-md-6 col-lg-4">Column 2</div>
    <div class="col-12 col-md-12 col-lg-4">Column 3</div>
  </div>
</div>
```

---

## 🔧 Part 11: Functions

### 11.1 `@function` Directive

Functions return values (unlike mixins which output CSS).

**Syntax:**

```scss
@function function-name($param1, $param2) {
  @return value;
}
```

**Example:**

```scss
@function calculate-rem($px, $base: 16px) {
  @return ($px / $base) * 1rem;
}

.text {
  font-size: calculate-rem(24px);  // 1.5rem
  padding: calculate-rem(32px);    // 2rem
}
```

---

### 11.2 Built-in Functions

**Color Functions:**

```scss
// Lighten/Darken
$primary: #3498db;

.button {
  background: $primary;
  
  &:hover {
    background: darken($primary, 10%);
  }
  
  &:active {
    background: darken($primary, 20%);
  }
}

.light-bg {
  background: lighten($primary, 40%);
}

// Saturate/Desaturate
.saturated {
  color: saturate($primary, 20%);
}

.desaturated {
  color: desaturate($primary, 20%);
}

// Adjust hue
.shifted {
  color: adjust-hue($primary, 45deg);
}

// Transparency
.transparent {
  background: rgba($primary, 0.5);
  background: transparentize($primary, 0.5);  // Same
}

.opaque {
  background: opacify(rgba($primary, 0.5), 0.3);
}

// Mix colors
.mixed {
  background: mix(red, blue, 50%);  // Purple
}

// Complement
.complement {
  color: complement($primary);
}

// Invert
.inverted {
  color: invert($primary);
}
```

**String Functions:**

```scss
// Quote/Unquote
$font: quote(Helvetica);        // "Helvetica"
$unquoted: unquote("Helvetica"); // Helvetica

// String length
$length: str-length("Hello");   // 5

// String insert
$result: str-insert("Hello", " World", 6);  // "Hello World"

// String index
$index: str-index("Hello World", "World");  // 7

// String slice
$slice: str-slice("Hello World", 1, 5);     // "Hello"

// To upper/lower case
$upper: to-upper-case("hello");  // "HELLO"
$lower: to-lower-case("HELLO");  // "hello"
```

**Number Functions:**

```scss
// Percentage
$percent: percentage(0.5);      // 50%

// Round
$rounded: round(3.7);           // 4
$rounded: round(3.2);           // 3

// Ceil/Floor
$ceil: ceil(3.2);               // 4
$floor: floor(3.7);             // 3

// Abs
$abs: abs(-10);                 // 10

// Min/Max
$min: min(1px, 2px, 3px);       // 1px
$max: max(1px, 2px, 3px);       // 3px

// Random
$random: random(100);           // Random number 1-100
```

**List Functions:**

```scss
$list: 10px 20px 30px;

// Length
$length: length($list);         // 3

// Nth (1-indexed)
$first: nth($list, 1);          // 10px
$last: nth($list, -1);          // 30px

// Append
$new-list: append($list, 40px); // 10px 20px 30px 40px

// Join
$list1: 10px 20px;
$list2: 30px 40px;
$joined: join($list1, $list2);  // 10px 20px 30px 40px

// Index
$index: index($list, 20px);     // 2
```

**Map Functions:**

```scss
$colors: (
  primary: #3498db,
  secondary: #2ecc71
);

// Get value
$primary: map-get($colors, primary);  // #3498db

// Has key
$has-key: map-has-key($colors, primary);  // true

// Keys
$keys: map-keys($colors);  // primary, secondary

// Values
$values: map-values($colors);  // #3498db, #2ecc71

// Merge
$new-colors: (danger: #e74c3c);
$merged: map-merge($colors, $new-colors);

// Remove
$removed: map-remove($colors, secondary);
```

---

### 11.3 Custom Function Examples

**Strip Units:**

```scss
@function strip-unit($number) {
  @if type-of($number) == 'number' and not unitless($number) {
    @return $number / ($number * 0 + 1);
  }
  
  @return $number;
}

$value: strip-unit(16px);  // 16
```

**Fluid Typography:**

```scss
@function fluid-type($min-size, $max-size, $min-width: 320px, $max-width: 1200px) {
  $slope: ($max-size - $min-size) / ($max-width - $min-width);
  $y-axis-intersection: -$min-width * $slope + $min-size;
  
  @return clamp(
    #{$min-size},
    #{$y-axis-intersection} + #{$slope * 100}vw,
    #{$max-size}
  );
}

.title {
  font-size: fluid-type(24px, 48px);
}
```

**Color Contrast:**

```scss
@function color-contrast($color) {
  $lightness: lightness($color);
  
  @if $lightness > 50% {
    @return #000;  // Dark text on light background
  } @else {
    @return #fff;  // Light text on dark background
  }
}

.button {
  background: #3498db;
  color: color-contrast(#3498db);  // white
}
```

**Spacing Scale:**

```scss
@function spacing($multiplier) {
  $base: 8px;
  @return $base * $multiplier;
}

.card {
  padding: spacing(2);    // 16px
  margin: spacing(4);     // 32px
}
```

---

### 11.4 Function vs Mixin

| Feature | Function | Mixin |
|---------|----------|-------|
| **Returns** | Value | CSS code |
| **Usage** | In expressions | `@include` |
| **Parameters** | ✅ Yes | ✅ Yes |
| **@content** | ❌ No | ✅ Yes |

**Use Function:**

```scss
@function double($value) {
  @return $value * 2;
}

.box {
  width: double(50px);  // 100px
}
```

**Use Mixin:**

```scss
@mixin double-size($value) {
  width: $value * 2;
  height: $value * 2;
}

.box {
  @include double-size(50px);
}
```

---

## 📱 Part 12: Media Queries and Responsive Design

### 12.1 Media Query Mixin with Content

```scss
$breakpoints: (
  xs: 0,
  sm: 576px,
  md: 768px,
  lg: 992px,
  xl: 1200px
);

@mixin respond-to($breakpoint) {
  @if map-has-key($breakpoints, $breakpoint) {
    @media (min-width: map-get($breakpoints, $breakpoint)) {
      @content;
    }
  } @else {
    @warn "Breakpoint `#{$breakpoint}` not found in $breakpoints map.";
  }
}

// Usage
.container {
  width: 100%;
  
  @include respond-to(sm) {
    width: 540px;
  }
  
  @include respond-to(md) {
    width: 720px;
  }
  
  @include respond-to(lg) {
    width: 960px;
  }
}
```

---

### 12.2 Mobile-First vs Desktop-First

**Mobile-First (min-width):**

```scss
@mixin mq-up($breakpoint) {
  @media (min-width: $breakpoint) {
    @content;
  }
}

.text {
  font-size: 14px;  // Mobile default
  
  @include mq-up(768px) {
    font-size: 16px;  // Tablet and up
  }
  
  @include mq-up(1024px) {
    font-size: 18px;  // Desktop and up
  }
}
```

**Desktop-First (max-width):**

```scss
@mixin mq-down($breakpoint) {
  @media (max-width: $breakpoint - 1px) {
    @content;
  }
}

.text {
  font-size: 18px;  // Desktop default
  
  @include mq-down(1024px) {
    font-size: 16px;  // Tablet and down
  }
  
  @include mq-down(768px) {
    font-size: 14px;  // Mobile and down
  }
}
```

---

### 12.3 Range-Based Media Queries

```scss
@mixin mq-between($min, $max) {
  @media (min-width: $min) and (max-width: $max - 1px) {
    @content;
  }
}

.sidebar {
  display: none;
  
  @include mq-between(768px, 1024px) {
    display: block;
    width: 200px;
  }
  
  @include mq-between(1024px, 9999px) {
    display: block;
    width: 300px;
  }
}
```

---

### 12.4 Complete Media Query System

```scss
// Breakpoints
$breakpoints: (
  xs: 0,
  sm: 576px,
  md: 768px,
  lg: 992px,
  xl: 1200px,
  xxl: 1400px
);

// Min-width (mobile-first)
@mixin mq-up($breakpoint) {
  @if map-has-key($breakpoints, $breakpoint) {
    $min: map-get($breakpoints, $breakpoint);
    @media (min-width: $min) {
      @content;
    }
  } @else {
    @media (min-width: $breakpoint) {
      @content;
    }
  }
}

// Max-width (desktop-first)
@mixin mq-down($breakpoint) {
  @if map-has-key($breakpoints, $breakpoint) {
    $max: map-get($breakpoints, $breakpoint) - 1px;
    @media (max-width: $max) {
      @content;
    }
  } @else {
    @media (max-width: $breakpoint - 1px) {
      @content;
    }
  }
}

// Between two breakpoints
@mixin mq-between($min-breakpoint, $max-breakpoint) {
  $min: if(map-has-key($breakpoints, $min-breakpoint), 
           map-get($breakpoints, $min-breakpoint), 
           $min-breakpoint);
  $max: if(map-has-key($breakpoints, $max-breakpoint), 
           map-get($breakpoints, $max-breakpoint) - 1px, 
           $max-breakpoint - 1px);
  
  @media (min-width: $min) and (max-width: $max) {
    @content;
  }
}

// Only specific breakpoint
@mixin mq-only($breakpoint) {
  $breakpoint-keys: map-keys($breakpoints);
  $index: index($breakpoint-keys, $breakpoint);
  
  @if $index {
    $min: map-get($breakpoints, $breakpoint);
    
    @if $index < length($breakpoint-keys) {
      $next-breakpoint: nth($breakpoint-keys, $index + 1);
      $max: map-get($breakpoints, $next-breakpoint) - 1px;
      
      @media (min-width: $min) and (max-width: $max) {
        @content;
      }
    } @else {
      @media (min-width: $min) {
        @content;
      }
    }
  }
}

// Print media
@mixin mq-print {
  @media print {
    @content;
  }
}

// Retina displays
@mixin mq-retina {
  @media (-webkit-min-device-pixel-ratio: 2), (min-resolution: 192dpi) {
    @content;
  }
}
```

**Usage Examples:**

```scss
.container {
  padding: 10px;
  
  @include mq-up(md) {
    padding: 20px;
  }
  
  @include mq-down(sm) {
    padding: 5px;
  }
  
  @include mq-between(md, lg) {
    background: lightblue;
  }
  
  @include mq-only(md) {
    border: 1px solid red;
  }
  
  @include mq-print {
    display: none;
  }
  
  @include mq-retina {
    background-image: url('logo@2x.png');
  }
}
```

---

## 🎓 Part 13: Mastering SASS

### 13.1 Advanced Techniques

**Dynamic Property Generation:**

```scss
$properties: (
  margin: m,
  padding: p
);

$sides: (
  top: t,
  right: r,
  bottom: b,
  left: l
);

$sizes: (0, 1, 2, 3, 4, 5);

@each $prop-name, $prop-abbr in $properties {
  @each $side-name, $side-abbr in $sides {
    @each $size in $sizes {
      .#{$prop-abbr}#{$side-abbr}-#{$size} {
        #{$prop-name}-#{$side-name}: #{$size * 0.25}rem;
      }
    }
  }
}

// Generates: .mt-0, .mt-1, .mr-0, .pr-1, etc.
```

**Theming System:**

```scss
$themes: (
  light: (
    bg: #ffffff,
    text: #000000,
    primary: #3498db
  ),
  dark: (
    bg: #1a1a1a,
    text: #ffffff,
    primary: #5dade2
  )
);

@mixin themed($property, $key) {
  @each $theme-name, $theme-map in $themes {
    .theme-#{$theme-name} & {
      #{$property}: map-get($theme-map, $key);
    }
  }
}

.card {
  @include themed(background, bg);
  @include themed(color, text);
}

.button {
  @include themed(background, primary);
}
```

---

### 13.2 Performance Optimization

**1. Avoid Deep Nesting:**

```scss
// ❌ Bad (creates .nav ul li a span)
.nav {
  ul {
    li {
      a {
        span {
          color: red;
        }
      }
    }
  }
}

// ✅ Good (flat selectors)
.nav__link-text {
  color: red;
}
```

**2. Use Placeholders for Shared Styles:**

```scss
// ✅ Good (grouped selectors)
%button-base {
  padding: 10px 20px;
  border: none;
}

.btn-primary { @extend %button-base; }
.btn-secondary { @extend %button-base; }
```

**3. Minimize `@extend` Usage:**

```scss
// ❌ Can create bloated CSS
.button { padding: 10px; }
.link { @extend .button; }

// ✅ Better: use mixin or placeholder
%button-base { padding: 10px; }
.button { @extend %button-base; }
.link { @extend %button-base; }
```

---

### 13.3 Common Pitfalls

**1. Color Manipulation Limits:**

```scss
$color: #000;

// ❌ Can't darken black
.dark {
  background: darken($color, 10%);  // Still #000
}

// ✅ Use lighten or adjust-color
.lighter {
  background: lighten($color, 10%);
}
```

**2. Division Deprecation:**

```scss
// ❌ Deprecated (will be removed)
.box {
  width: 100px / 2;
}

// ✅ Use math.div()
@use 'sass:math';

.box {
  width: math.div(100px, 2);
}
```

**3. Variable Scope Issues:**

```scss
.button {
  $padding: 10px;  // Local scope
}

.card {
  padding: $padding;  // ❌ Error: undefined variable
}
```

---

### 13.4 Best Practices Summary

**Do's ✅**

1. Use `@use` instead of `@import`
2. Organize with 7-1 pattern
3. Use variables for repeated values
4. Keep nesting shallow (max 3-4 levels)
5. Use mixins for reusable patterns
6. Use functions for calculations
7. Use placeholders for static shared styles
8. Comment your code
9. Use meaningful variable names
10. Follow BEM or similar naming convention

**Don'ts ❌**

1. Don't nest too deeply
2. Don't overuse `@extend`
3. Don't create unnecessary classes with loops
4. Don't use `@import` (deprecated)
5. Don't ignore compilation warnings
6. Don't mix SASS and SCSS syntax
7. Don't use magic numbers (use variables)
8. Don't duplicate code (use mixins/functions)
9. Don't forget source maps
10. Don't ignore file size

---

### 13.5 Next Steps

**Continue Learning:**

1. **Practice Projects:**
   - Build a component library
   - Create a design system
   - Implement a responsive framework

2. **Advanced Topics:**
   - CSS architecture (ITCSS, SMACSS)
   - Design tokens
   - CSS-in-JS alternatives
   - PostCSS integration

3. **Tools and Workflow:**
   - Stylelint for linting
   - Prettier for formatting
   - Storybook for component development
   - CI/CD integration

4. **Performance:**
   - Critical CSS
   - CSS purging (PurgeCSS)
   - Bundle optimization
   - Lazy loading styles

---

## 📚 Summary

You've learned:

✅ **Week 1:**
- SASS fundamentals and compilation
- Modern `@use` and `@forward`
- Variables and scoping
- Nesting and parent selector
- Placeholders and `@extend`
- Control flow (`@if/@else`)
- Interpolation
- Comments and documentation

✅ **Week 2:**
- Mixins with parameters and `@content`
- Loops (`@for`, `@each`, `@while`)
- Functions and built-in functions
- Bootstrap grid system
- Media query mixins
- Advanced techniques
- Performance optimization
- Best practices

**You're now ready to write professional, maintainable SASS code!** 🎉

---

*For hands-on practice, see `hands-on.md`. For quick reference, see `deliverables.md`.*
