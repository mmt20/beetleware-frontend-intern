# SASS — Hands-on Exercises and Challenges

> **Practice Goal:** Build muscle memory for SASS features through practical exercises, real-world projects, and debugging challenges.

---

## 🎯 Exercise 1: Variables and Nesting Challenge

### Part A: Create a Theme System

**Instructions:** Create a complete theme system using SASS variables.

```scss
// TODO: Define your theme variables here
// Include: colors, typography, spacing, borders, shadows

// Example structure:
$primary-color: ?;
$secondary-color: ?;
$font-family-base: ?;
// ... add more
```

<details>
<summary>💡 Show Solution</summary>

```scss
// Theme Variables
$primary-color: #3498db;
$secondary-color: #2ecc71;
$danger-color: #e74c3c;
$warning-color: #f39c12;
$info-color: #3498db;

// Typography
$font-family-base: 'Helvetica Neue', Arial, sans-serif;
$font-family-heading: 'Georgia', serif;
$font-size-base: 16px;
$line-height-base: 1.5;

// Spacing
$spacing-unit: 8px;
$spacing-xs: $spacing-unit;
$spacing-sm: $spacing-unit * 2;
$spacing-md: $spacing-unit * 3;
$spacing-lg: $spacing-unit * 4;
$spacing-xl: $spacing-unit * 6;

// Borders
$border-radius-sm: 4px;
$border-radius-md: 8px;
$border-radius-lg: 16px;
$border-width: 1px;
$border-color: #ddd;

// Shadows
$shadow-sm: 0 2px 4px rgba(0,0,0,0.1);
$shadow-md: 0 4px 8px rgba(0,0,0,0.15);
$shadow-lg: 0 8px 16px rgba(0,0,0,0.2);

// Breakpoints
$breakpoint-sm: 576px;
$breakpoint-md: 768px;
$breakpoint-lg: 992px;
$breakpoint-xl: 1200px;
```

</details>

---

### Part B: Nested Navigation

**Instructions:** Create a navigation component using nesting and the parent selector.

```html
<nav class="nav">
  <ul class="nav__list">
    <li class="nav__item nav__item--active">
      <a href="#" class="nav__link">Home</a>
    </li>
    <li class="nav__item">
      <a href="#" class="nav__link">About</a>
    </li>
  </ul>
</nav>
```

**Requirements:**
- Use BEM naming convention
- Use parent selector (`&`) for modifiers and pseudo-classes
- Keep nesting under 3 levels
- Add hover and active states

<details>
<summary>💡 Show Solution</summary>

```scss
.nav {
  background: $primary-color;
  padding: $spacing-md 0;
  
  &__list {
    list-style: none;
    margin: 0;
    padding: 0;
    display: flex;
    gap: $spacing-md;
  }
  
  &__item {
    position: relative;
    
    &--active {
      .nav__link {
        background: darken($primary-color, 10%);
        font-weight: bold;
      }
    }
  }
  
  &__link {
    color: white;
    text-decoration: none;
    padding: $spacing-sm $spacing-md;
    display: block;
    border-radius: $border-radius-sm;
    transition: background 0.3s ease;
    
    &:hover {
      background: darken($primary-color, 5%);
    }
    
    &:active {
      background: darken($primary-color, 15%);
    }
    
    &:focus {
      outline: 2px solid white;
      outline-offset: 2px;
    }
  }
}
```

</details>

---

### Part C: Predict the Output

**What will this SASS compile to?**

```scss
.card {
  $padding: 20px;
  padding: $padding;
  
  &__header {
    padding: $padding / 2;
    
    &--large {
      padding: $padding;
    }
  }
}
```

<details>
<summary>💡 Show Answer</summary>

```css
.card {
  padding: 20px;
}

.card__header {
  padding: 10px;
}

.card__header--large {
  padding: 20px;
}
```

</details>

---

## 🎨 Exercise 2: Mixins and Functions

### Part A: Reusable Button Mixin

**Instructions:** Create a button mixin that accepts parameters for customization.

**Requirements:**
- Accept background color, text color, and padding
- Include hover and active states
- Add focus styles for accessibility
- Support disabled state

<details>
<summary>💡 Show Solution</summary>

```scss
@mixin button(
  $bg-color: $primary-color,
  $text-color: white,
  $padding: $spacing-sm $spacing-md,
  $hover-darken: 10%
) {
  background: $bg-color;
  color: $text-color;
  padding: $padding;
  border: none;
  border-radius: $border-radius-sm;
  cursor: pointer;
  font-size: $font-size-base;
  font-family: $font-family-base;
  transition: all 0.3s ease;
  
  &:hover:not(:disabled) {
    background: darken($bg-color, $hover-darken);
    transform: translateY(-2px);
    box-shadow: $shadow-md;
  }
  
  &:active:not(:disabled) {
    background: darken($bg-color, $hover-darken * 1.5);
    transform: translateY(0);
    box-shadow: $shadow-sm;
  }
  
  &:focus {
    outline: 2px solid $bg-color;
    outline-offset: 2px;
  }
  
  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
}

// Usage
.btn-primary {
  @include button($primary-color);
}

.btn-secondary {
  @include button($secondary-color);
}

.btn-danger {
  @include button($danger-color);
}
```

</details>

---

### Part B: Utility Functions

**Instructions:** Create these utility functions:

1. `rem($px)` - Convert pixels to rem
2. `em($px, $base)` - Convert pixels to em
3. `strip-unit($number)` - Remove unit from number
4. `color-yiq($color)` - Return black or white based on background lightness

<details>
<summary>💡 Show Solution</summary>

```scss
// Convert px to rem
@function rem($px, $base: 16px) {
  @return ($px / $base) * 1rem;
}

// Convert px to em
@function em($px, $base: 16px) {
  @return ($px / $base) * 1em;
}

// Strip unit from number
@function strip-unit($number) {
  @if type-of($number) == 'number' and not unitless($number) {
    @return $number / ($number * 0 + 1);
  }
  @return $number;
}

// YIQ color contrast
@function color-yiq($color) {
  $r: red($color);
  $g: green($color);
  $b: blue($color);
  
  $yiq: (($r * 299) + ($g * 587) + ($b * 114)) / 1000;
  
  @if ($yiq >= 128) {
    @return #000;
  } @else {
    @return #fff;
  }
}

// Usage
.text {
  font-size: rem(24px);        // 1.5rem
  margin: em(16px, 14px);      // 1.142857em
}

.button {
  background: $primary-color;
  color: color-yiq($primary-color);  // white or black
}
```

</details>

---

## 🔀 Exercise 3: Control Flow Challenges

### Challenge 1: Theme Switcher

**Instructions:** Create a mixin that generates different button styles based on a theme parameter.

```scss
@mixin button-theme($theme) {
  // TODO: Use @if/@else to set styles based on $theme
  // Themes: 'light', 'dark', 'colorful'
}
```

<details>
<summary>💡 Show Solution</summary>

```scss
@mixin button-theme($theme) {
  @if $theme == 'light' {
    background: white;
    color: $primary-color;
    border: 2px solid $primary-color;
    
    &:hover {
      background: $primary-color;
      color: white;
    }
  } @else if $theme == 'dark' {
    background: #1a1a1a;
    color: white;
    border: 2px solid #333;
    
    &:hover {
      background: #333;
    }
  } @else if $theme == 'colorful' {
    background: linear-gradient(45deg, $primary-color, $secondary-color);
    color: white;
    border: none;
    
    &:hover {
      background: linear-gradient(45deg, darken($primary-color, 10%), darken($secondary-color, 10%));
    }
  } @else {
    @warn "Unknown theme: #{$theme}. Using default.";
    background: $primary-color;
    color: white;
  }
  
  padding: $spacing-sm $spacing-md;
  border-radius: $border-radius-sm;
  cursor: pointer;
  transition: all 0.3s ease;
}

// Usage
.btn-light {
  @include button-theme('light');
}

.btn-dark {
  @include button-theme('dark');
}

.btn-colorful {
  @include button-theme('colorful');
}
```

</details>

---

### Challenge 2: Dynamic Spacing Utilities

**Instructions:** Create spacing utilities that use control flow to determine the multiplier.

<details>
<summary>💡 Show Solution</summary>

```scss
$spacing-base: 8px;

@mixin spacing($size, $property: 'margin') {
  $multiplier: 1;
  
  @if $size == 'xs' {
    $multiplier: 0.5;
  } @else if $size == 'sm' {
    $multiplier: 1;
  } @else if $size == 'md' {
    $multiplier: 2;
  } @else if $size == 'lg' {
    $multiplier: 3;
  } @else if $size == 'xl' {
    $multiplier: 4;
  } @else if type-of($size) == 'number' {
    $multiplier: $size;
  } @else {
    @error "Invalid size: #{$size}";
  }
  
  #{$property}: $spacing-base * $multiplier;
}

// Usage
.card {
  @include spacing('md', 'padding');  // padding: 16px
  @include spacing('lg', 'margin');   // margin: 24px
  @include spacing(5, 'padding');     // padding: 40px
}
```

</details>

---

## 🔁 Exercise 4: Loop Mastery

### Challenge 1: Generate Color Palette

**Instructions:** Create a color palette generator using `@each` and a map.

```scss
$colors: (
  'blue': #3498db,
  'green': #2ecc71,
  'red': #e74c3c,
  'yellow': #f39c12,
  'purple': #9b59b6
);

// TODO: Generate .bg-{color}, .text-{color}, .border-{color} classes
// TODO: Also generate lighter and darker variants
```

<details>
<summary>💡 Show Solution</summary>

```scss
$colors: (
  'blue': #3498db,
  'green': #2ecc71,
  'red': #e74c3c,
  'yellow': #f39c12,
  'purple': #9b59b6
);

@each $name, $color in $colors {
  // Base color
  .bg-#{$name} {
    background-color: $color;
  }
  
  .text-#{$name} {
    color: $color;
  }
  
  .border-#{$name} {
    border-color: $color;
  }
  
  // Light variant
  .bg-#{$name}-light {
    background-color: lighten($color, 20%);
  }
  
  .text-#{$name}-light {
    color: lighten($color, 20%);
  }
  
  // Dark variant
  .bg-#{$name}-dark {
    background-color: darken($color, 20%);
  }
  
  .text-#{$name}-dark {
    color: darken($color, 20%);
  }
}
```

</details>

---

### Challenge 2: Responsive Grid System

**Instructions:** Create a 12-column grid system with responsive breakpoints.

<details>
<summary>💡 Show Solution</summary>

```scss
$grid-columns: 12;
$breakpoints: (
  'sm': 576px,
  'md': 768px,
  'lg': 992px,
  'xl': 1200px
);

// Base columns (mobile-first)
@for $i from 1 through $grid-columns {
  .col-#{$i} {
    flex: 0 0 percentage($i / $grid-columns);
    max-width: percentage($i / $grid-columns);
  }
}

// Responsive columns
@each $breakpoint, $width in $breakpoints {
  @media (min-width: $width) {
    @for $i from 1 through $grid-columns {
      .col-#{$breakpoint}-#{$i} {
        flex: 0 0 percentage($i / $grid-columns);
        max-width: percentage($i / $grid-columns);
      }
    }
  }
}
```

</details>

---

### Challenge 3: Animation Keyframes Generator

**Instructions:** Create a loop that generates fade-in animations with different delays.

<details>
<summary>💡 Show Solution</summary>

```scss
@for $i from 1 through 10 {
  .fade-in-#{$i} {
    animation: fadeIn 0.5s ease-in-out #{$i * 0.1}s both;
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

// Usage:
// <div class="fade-in-1">Appears first</div>
// <div class="fade-in-2">Appears second</div>
// <div class="fade-in-3">Appears third</div>
```

</details>

---

## 🏗️ Exercise 5: Real-World Projects

### Project 1: Complete Card Component

**Instructions:** Create a complete card component with:
- Header, body, and footer sections
- Multiple variants (default, featured, compact)
- Hover effects
- Responsive design

<details>
<summary>💡 Show Solution</summary>

```scss
.card {
  background: white;
  border: $border-width solid $border-color;
  border-radius: $border-radius-md;
  overflow: hidden;
  transition: all 0.3s ease;
  
  &:hover {
    box-shadow: $shadow-lg;
    transform: translateY(-4px);
  }
  
  &__header {
    padding: $spacing-md;
    border-bottom: $border-width solid $border-color;
    background: lighten($primary-color, 45%);
  }
  
  &__title {
    margin: 0;
    font-size: rem(24px);
    font-family: $font-family-heading;
    color: $primary-color;
  }
  
  &__subtitle {
    margin: $spacing-xs 0 0;
    font-size: rem(14px);
    color: #666;
  }
  
  &__body {
    padding: $spacing-md;
  }
  
  &__footer {
    padding: $spacing-md;
    border-top: $border-width solid $border-color;
    background: #f9f9f9;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  // Variants
  &--featured {
    border-color: $primary-color;
    border-width: 2px;
    
    .card__header {
      background: $primary-color;
    }
    
    .card__title {
      color: white;
    }
  }
  
  &--compact {
    .card__header,
    .card__body,
    .card__footer {
      padding: $spacing-sm;
    }
  }
  
  // Responsive
  @media (max-width: $breakpoint-md) {
    &__footer {
      flex-direction: column;
      gap: $spacing-sm;
    }
  }
}
```

</details>

---

### Project 2: Form System

**Instructions:** Create a complete form system with validation styles.

<details>
<summary>💡 Show Solution</summary>

```scss
.form {
  &__group {
    margin-bottom: $spacing-md;
  }
  
  &__label {
    display: block;
    margin-bottom: $spacing-xs;
    font-weight: 600;
    color: #333;
    
    &--required::after {
      content: ' *';
      color: $danger-color;
    }
  }
  
  &__input,
  &__textarea,
  &__select {
    width: 100%;
    padding: $spacing-sm;
    border: $border-width solid $border-color;
    border-radius: $border-radius-sm;
    font-family: $font-family-base;
    font-size: $font-size-base;
    transition: all 0.3s ease;
    
    &:focus {
      outline: none;
      border-color: $primary-color;
      box-shadow: 0 0 0 3px rgba($primary-color, 0.1);
    }
    
    &:disabled {
      background: #f5f5f5;
      cursor: not-allowed;
    }
    
    // Validation states
    &.is-valid {
      border-color: $secondary-color;
      
      &:focus {
        box-shadow: 0 0 0 3px rgba($secondary-color, 0.1);
      }
    }
    
    &.is-invalid {
      border-color: $danger-color;
      
      &:focus {
        box-shadow: 0 0 0 3px rgba($danger-color, 0.1);
      }
    }
  }
  
  &__textarea {
    resize: vertical;
    min-height: 100px;
  }
  
  &__help {
    display: block;
    margin-top: $spacing-xs;
    font-size: rem(14px);
    color: #666;
  }
  
  &__error {
    display: block;
    margin-top: $spacing-xs;
    font-size: rem(14px);
    color: $danger-color;
  }
  
  &__success {
    display: block;
    margin-top: $spacing-xs;
    font-size: rem(14px);
    color: $secondary-color;
  }
}
```

</details>

---

## 🐛 Exercise 6: Debug the SASS

### Bug 1: Compilation Error

**What's wrong with this code?**

```scss
$colors: (
  primary: #3498db,
  secondary: #2ecc71
);

.button {
  background: $colors.primary;  // Error!
}
```

<details>
<summary>💡 Show Fix</summary>

```scss
// Problem: Can't use dot notation with maps
// Solution: Use map-get()

.button {
  background: map-get($colors, primary);  // ✅ Correct
}
```

</details>

---

### Bug 2: Nesting Too Deep

**Refactor this code to be more maintainable:**

```scss
.nav {
  ul {
    li {
      a {
        span {
          i {
            color: red;  // 6 levels deep!
          }
        }
      }
    }
  }
}
```

<details>
<summary>💡 Show Fix</summary>

```scss
// Use BEM instead of deep nesting
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
}

.nav__link-text {
  // Element styles
}

.nav__icon {
  color: red;  // ✅ Flat structure
}
```

</details>

---

### Bug 3: Mixin vs Function Confusion

**Fix this code:**

```scss
@function button-styles($color) {
  background: $color;
  color: white;
  padding: 10px 20px;
}

.button {
  @include button-styles(blue);  // Error!
}
```

<details>
<summary>💡 Show Fix</summary>

```scss
// Problem: Functions return values, mixins output CSS
// Solution: Use @mixin instead

@mixin button-styles($color) {
  background: $color;
  color: white;
  padding: 10px 20px;
}

.button {
  @include button-styles(blue);  // ✅ Correct
}
```

</details>

---

## 🏆 Final Challenge: Build a Design System

**Instructions:** Create a complete mini design system with:

1. **Variables** - Colors, typography, spacing, breakpoints
2. **Mixins** - Buttons, cards, forms, media queries
3. **Functions** - Unit conversion, color utilities
4. **Components** - At least 5 reusable components
5. **Utilities** - Spacing, colors, typography classes
6. **Grid System** - Responsive 12-column grid

**Bonus:**
- Use 7-1 architecture
- Include dark theme support
- Add animation utilities
- Create comprehensive documentation

---

## 💪 Practice Drills

### Rapid Fire: True or False

1. SASS variables start with `$`
2. `@import` is the modern way to include files
3. Mixins can accept parameters
4. Functions output CSS code
5. `@extend` creates grouped selectors
6. The parent selector is `&`
7. `@for` loop is inclusive with `through`
8. Maps are accessed with dot notation
9. `!default` sets a variable only if undefined
10. SCSS syntax requires semicolons

<details>
<summary>💡 Show Answers</summary>

1. ✅ True
2. ❌ False (`@use` is modern)
3. ✅ True
4. ❌ False (functions return values)
5. ✅ True
6. ✅ True
7. ✅ True
8. ❌ False (use `map-get()`)
9. ✅ True
10. ✅ True

</details>

---

**Keep practicing! The more you code, the better you'll get!** 🚀
