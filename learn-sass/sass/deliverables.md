# SASS — Quick Reference Cheatsheet

> **Quick Reference:** Your go-to guide for SASS syntax, features, and best practices.

---

## 📊 SASS vs SCSS Syntax

| Feature | SASS (Indented) | SCSS (Recommended) ✅ |
|---------|-----------------|----------------------|
| **Braces** | No | Yes `{}` |
| **Semicolons** | No | Yes `;` |
| **File Extension** | `.sass` | `.scss` |
| **Example** | `color: red` | `color: red;` |

**Use SCSS** - It's CSS-compatible and industry standard.

---

## 🎨 Variables

```scss
// Declaration
$variable-name: value;

// Types
$color: #3498db;              // Color
$font-size: 16px;             // Number
$font-family: 'Helvetica';    // String
$is-active: true;             // Boolean
$value: null;                 // Null
$list: 10px 20px 30px;        // List
$map: (key: value);           // Map

// Scope
$global: blue;                // Global scope

.selector {
  $local: red;                // Local scope
}

// Default values
$primary: blue !default;      // Set only if undefined

// Interpolation
.#{$name} {                   // In selectors
  #{$property}: value;        // In properties
  content: "Value: #{$var}";  // In strings
}
```

---

## 🎯 Nesting

```scss
// Basic nesting
.parent {
  .child {
    color: red;
  }
}

// Parent selector (&)
.button {
  &:hover { }                 // Pseudo-class
  &::before { }               // Pseudo-element
  &--modifier { }             // BEM modifier
  &.is-active { }             // Compound selector
  .sidebar & { }              // Parent context
}

// Property nesting
.box {
  font: {
    family: 'Helvetica';
    size: 16px;
    weight: bold;
  }
}

// Media query nesting
.sidebar {
  width: 300px;
  
  @media (max-width: 768px) {
    width: 100%;
  }
}
```

**Best Practice:** Keep nesting under 3-4 levels.

---

## 🔧 Mixins

```scss
// Basic mixin
@mixin mixin-name {
  property: value;
}

.selector {
  @include mixin-name;
}

// With parameters
@mixin button($bg, $color: white) {
  background: $bg;
  color: $color;
}

.btn {
  @include button(blue);
  @include button(red, black);
  @include button($color: yellow, $bg: purple);  // Named
}

// Variable arguments
@mixin box-shadow($shadows...) {
  box-shadow: $shadows;
}

// With @content
@mixin media($width) {
  @media (min-width: $width) {
    @content;
  }
}

.container {
  @include media(768px) {
    width: 750px;
  }
}
```

---

## 🔢 Functions

```scss
// Custom function
@function function-name($param) {
  @return value;
}

// Example
@function rem($px, $base: 16px) {
  @return ($px / $base) * 1rem;
}

.text {
  font-size: rem(24px);  // 1.5rem
}
```

**Built-in Functions:**

```scss
// Color
darken($color, 10%)
lighten($color, 10%)
saturate($color, 20%)
desaturate($color, 20%)
adjust-hue($color, 45deg)
rgba($color, 0.5)
mix($color1, $color2, 50%)
complement($color)
invert($color)

// String
quote($string)
unquote($string)
str-length($string)
to-upper-case($string)
to-lower-case($string)

// Number
percentage(0.5)          // 50%
round(3.7)               // 4
ceil(3.2)                // 4
floor(3.7)               // 3
abs(-10)                 // 10
min(1px, 2px, 3px)       // 1px
max(1px, 2px, 3px)       // 3px

// List
length($list)
nth($list, 1)            // 1-indexed
append($list, value)
join($list1, $list2)
index($list, value)

// Map
map-get($map, key)
map-has-key($map, key)
map-keys($map)
map-values($map)
map-merge($map1, $map2)
map-remove($map, key)
```

---

## 🎭 Placeholders and @extend

```scss
// Placeholder (doesn't compile alone)
%placeholder {
  property: value;
}

.selector {
  @extend %placeholder;
}

// Compiled (grouped selectors)
.selector1, .selector2 {
  property: value;
}
```

**Placeholder vs Mixin:**

| Feature | Placeholder | Mixin |
|---------|-------------|-------|
| Output | Grouped selectors | Duplicated styles |
| Parameters | ❌ No | ✅ Yes |
| Use Case | Static shared styles | Dynamic styles |

---

## 🔀 Control Flow

```scss
// @if / @else
@if condition {
  // Styles
} @else if condition {
  // Styles
} @else {
  // Styles
}

// Operators
==  !=  <  <=  >  >=
and  or  not

// Example
@if $theme == dark {
  background: black;
} @else {
  background: white;
}
```

---

## 🔁 Loops

```scss
// @for (inclusive)
@for $i from 1 through 5 {
  .col-#{$i} {
    width: percentage($i / 12);
  }
}

// @for (exclusive)
@for $i from 1 to 5 {
  // $i = 1, 2, 3, 4
}

// @each (list)
$colors: red, green, blue;

@each $color in $colors {
  .text-#{$color} {
    color: $color;
  }
}

// @each (map)
$theme-colors: (
  primary: #3498db,
  secondary: #2ecc71
);

@each $name, $color in $theme-colors {
  .btn-#{$name} {
    background: $color;
  }
}

// @while
$i: 1;

@while $i <= 5 {
  .item-#{$i} {
    width: $i * 20px;
  }
  $i: $i + 1;
}
```

---

## 📦 @use and @forward

```scss
// Modern way (recommended) ✅
@use 'variables';              // Namespaced
@use 'mixins' as mx;           // Custom namespace
@use 'functions' as *;         // No namespace

.button {
  color: variables.$primary;
  @include mx.button-style;
}

// Forward (barrel file)
// _index.scss
@forward 'variables';
@forward 'mixins';
@forward 'functions';

// main.scss
@use 'index' as *;
```

**Old way (deprecated) ❌:**

```scss
@import 'variables';  // Don't use!
```

---

## 📱 Media Queries

```scss
// Basic
@media (min-width: 768px) {
  .container {
    width: 750px;
  }
}

// Mixin pattern
$breakpoints: (
  sm: 576px,
  md: 768px,
  lg: 992px,
  xl: 1200px
);

@mixin respond-to($breakpoint) {
  @media (min-width: map-get($breakpoints, $breakpoint)) {
    @content;
  }
}

// Usage
.container {
  width: 100%;
  
  @include respond-to(md) {
    width: 750px;
  }
}
```

---

## 🏗️ Grid System Pattern

```scss
$grid-columns: 12;
$grid-gutter: 30px;

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 $grid-gutter / 2;
}

.row {
  display: flex;
  flex-wrap: wrap;
  margin: 0 -($grid-gutter / 2);
}

@for $i from 1 through $grid-columns {
  .col-#{$i} {
    flex: 0 0 percentage($i / $grid-columns);
    max-width: percentage($i / $grid-columns);
    padding: 0 $grid-gutter / 2;
  }
}
```

---

## 💬 Comments

```scss
// Silent comment (not in CSS output)

/* Loud comment (in CSS output) */

/*! Important comment (always in output) */

/// SassDoc comment
/// @param {Color} $color - The color value
```

---

## 🎯 BEM with SASS

```scss
.block {
  // Block styles
  
  &__element {
    // Element styles
  }
  
  &--modifier {
    // Modifier styles
  }
  
  &__element--modifier {
    // Element with modifier
  }
}

// Compiled:
.block { }
.block__element { }
.block--modifier { }
.block__element--modifier { }
```

---

## 📁 7-1 Architecture Pattern

```
scss/
├── abstracts/
│   ├── _variables.scss
│   ├── _mixins.scss
│   ├── _functions.scss
│   └── _index.scss
├── base/
│   ├── _reset.scss
│   ├── _typography.scss
│   └── _index.scss
├── components/
│   ├── _buttons.scss
│   ├── _cards.scss
│   └── _index.scss
├── layout/
│   ├── _header.scss
│   ├── _footer.scss
│   └── _index.scss
├── pages/
│   ├── _home.scss
│   └── _index.scss
├── themes/
│   ├── _dark.scss
│   └── _index.scss
├── vendors/
│   └── _normalize.scss
└── main.scss
```

---

## ⚙️ Compilation

```bash
# Install Dart Sass
npm install -g sass

# Compile
sass input.scss output.css

# Watch mode
sass --watch scss:css

# Compressed output
sass --style=compressed input.scss output.css

# Source maps
sass --source-map input.scss output.css
```

---

## ✅ Best Practices

**Do's ✅**

1. Use `@use` instead of `@import`
2. Use `const` for SCSS (use `let`/`const` in JS)
3. Keep nesting under 3-4 levels
4. Use variables for repeated values
5. Use mixins for reusable patterns
6. Use functions for calculations
7. Use placeholders for static shared styles
8. Follow BEM or similar naming convention
9. Organize with 7-1 pattern
10. Use meaningful variable names

**Don'ts ❌**

1. Don't nest too deeply
2. Don't overuse `@extend`
3. Don't use `@import` (deprecated)
4. Don't create unnecessary classes with loops
5. Don't ignore compilation warnings
6. Don't mix SASS and SCSS syntax
7. Don't use magic numbers
8. Don't duplicate code
9. Don't forget source maps
10. Don't ignore file size

---

## 🎯 Quick Decision Trees

### Which to Use?

```
Need to output CSS?
├─ Yes → Use @mixin
└─ No → Use @function

Need parameters?
├─ Yes → Use @mixin or @function
└─ No → Use placeholder

Static shared styles?
├─ Yes → Use placeholder
└─ No → Use mixin

Need to include file?
├─ Modern → @use
└─ Legacy → @import (deprecated)
```

---

## 🔧 Common Patterns

**Button Mixin:**

```scss
@mixin button($bg, $color: white) {
  background: $bg;
  color: $color;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  
  &:hover {
    background: darken($bg, 10%);
  }
}
```

**Responsive Mixin:**

```scss
@mixin respond-to($breakpoint) {
  @media (min-width: $breakpoint) {
    @content;
  }
}
```

**Flexbox Center:**

```scss
@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
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
```

---

## ⚠️ Common Gotchas

```scss
// ❌ Division deprecated
width: 100px / 2;

// ✅ Use math.div()
@use 'sass:math';
width: math.div(100px, 2);

// ❌ Can't darken black
background: darken(#000, 10%);

// ✅ Use lighten
background: lighten(#000, 10%);

// ❌ Map access with dot
color: $colors.primary;

// ✅ Use map-get()
color: map-get($colors, primary);

// ❌ Variable scope issue
.button {
  $padding: 10px;
}
.card {
  padding: $padding;  // Error!
}

// ✅ Use global scope
$padding: 10px;

.button {
  padding: $padding;
}
```

---

## 📚 Resources

- [Official SASS Documentation](https://sass-lang.com/)
- [SASS Guidelines](https://sass-guidelin.es/)
- [7-1 Pattern](https://sass-guidelin.es/#the-7-1-pattern)
- [BEM Methodology](http://getbem.com/)

---

**Print this cheatsheet and keep it handy!** 📄
