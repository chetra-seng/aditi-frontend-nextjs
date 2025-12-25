# Introduction to Tailwind CSS

**Utility-First CSS Framework**

---

# What is Tailwind CSS?

<v-clicks>

- **Utility-first CSS framework** - pre-built CSS classes
- **No custom CSS needed** for most styling
- **Highly customizable** via config
- **Works great with React/Next.js**
- Created by Adam Wathan in 2017

</v-clicks>

---

# Traditional CSS vs Tailwind

<div class="grid grid-cols-2 gap-4">

<div>

## Traditional CSS

```css
/* styles.css */
.card {
  background-color: white;
  border-radius: 8px;
  padding: 16px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.card-title {
  font-size: 24px;
  font-weight: bold;
  color: #1a1a1a;
}
```

```jsx
<div className="card">
  <h2 className="card-title">Hello</h2>
</div>
```

</div>

<div>

## Tailwind CSS

```jsx
// No separate CSS file needed!

<div className="bg-white rounded-lg p-4 shadow">
  <h2 className="text-2xl font-bold text-gray-900">
    Hello
  </h2>
</div>
```

<v-clicks>

- Style directly in your components
- No context switching
- No naming things

</v-clicks>

</div>

</div>

---

# The Utility-First Concept

Instead of writing custom CSS, you compose styles using utility classes:

```jsx
<button className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">
  Click Me
</button>
```

Breaking it down:

| Class | What it does |
|-------|-------------|
| `bg-blue-500` | Background color (blue, shade 500) |
| `hover:bg-blue-700` | Darker blue on hover |
| `text-white` | White text color |
| `font-bold` | Bold font weight |
| `py-2` | Padding top & bottom (0.5rem) |
| `px-4` | Padding left & right (1rem) |
| `rounded` | Border radius |

---

# Tailwind's Design System

## Spacing Scale

| Class | Value | Pixels |
|-------|-------|--------|
| `p-1` | 0.25rem | 4px |
| `p-2` | 0.5rem | 8px |
| `p-4` | 1rem | 16px |
| `p-6` | 1.5rem | 24px |
| `p-8` | 2rem | 32px |

Works with: `m` (margin), `p` (padding), `w` (width), `h` (height), `gap`, etc.

---

# Color System

Tailwind provides a consistent color palette:

<div class="grid grid-cols-2 gap-4">

<div>

## Format: `{property}-{color}-{shade}`

```jsx
// Text colors
<p className="text-red-500">Error</p>
<p className="text-green-600">Success</p>
<p className="text-gray-700">Normal</p>

// Background colors
<div className="bg-blue-100">Light blue</div>
<div className="bg-blue-500">Medium blue</div>
<div className="bg-blue-900">Dark blue</div>
```

</div>

<div>

## Shade Scale (50-950)

- `50` - Lightest
- `100-200` - Very light
- `300-400` - Light
- `500` - Base color
- `600-700` - Dark
- `800-900` - Very dark
- `950` - Darkest

</div>

</div>

---

# Essential Layout Classes

<div class="grid grid-cols-2 gap-4">

<div>

## Flexbox

```jsx
// Horizontal center
<div className="flex justify-center">

// Vertical center
<div className="flex items-center">

// Both centered
<div className="flex justify-center items-center">

// Space between
<div className="flex justify-between">

// Column direction
<div className="flex flex-col">

// Gap between items
<div className="flex gap-4">
```

</div>

<div>

## Grid

```jsx
// Two columns
<div className="grid grid-cols-2 gap-4">

// Three columns
<div className="grid grid-cols-3 gap-6">

// Responsive grid
<div className="grid grid-cols-1 md:grid-cols-2
                lg:grid-cols-3 gap-4">
```

</div>

</div>

---

# Responsive Design

Tailwind uses **mobile-first** breakpoints:

| Prefix | Min Width | Typical Device |
|--------|-----------|----------------|
| (none) | 0px | Mobile (default) |
| `sm:` | 640px | Large phones |
| `md:` | 768px | Tablets |
| `lg:` | 1024px | Laptops |
| `xl:` | 1280px | Desktops |
| `2xl:` | 1536px | Large screens |

```jsx
<div className="text-sm md:text-base lg:text-lg">
  Responsive text size
</div>

<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4">
  Responsive grid
</div>
```

---

# State Modifiers

Apply styles on hover, focus, and other states:

```jsx
// Hover state
<button className="bg-blue-500 hover:bg-blue-700">
  Hover me
</button>

// Focus state
<input className="border focus:border-blue-500 focus:ring-2">

// Active state
<button className="bg-gray-200 active:bg-gray-300">

// Disabled state
<button className="disabled:opacity-50 disabled:cursor-not-allowed">

// Group hover (parent hover affects child)
<div className="group">
  <p className="group-hover:text-blue-500">I change when parent is hovered</p>
</div>
```

---

# Common Patterns You'll Use

```jsx
// Card
<div className="bg-white rounded-lg shadow-md p-6">

// Container with max width
<div className="max-w-4xl mx-auto px-4">

// Full-height centered content
<div className="min-h-screen flex items-center justify-center">

// Form input
<input className="w-full border border-gray-300 rounded-md px-3 py-2
                  focus:outline-none focus:ring-2 focus:ring-blue-500">

// Primary button
<button className="bg-blue-500 hover:bg-blue-600 text-white
                   font-medium py-2 px-4 rounded-md transition">
```

---

# Typography Classes

```jsx
// Font sizes
<p className="text-xs">Extra small (12px)</p>
<p className="text-sm">Small (14px)</p>
<p className="text-base">Base (16px)</p>
<p className="text-lg">Large (18px)</p>
<p className="text-xl">Extra large (20px)</p>
<p className="text-2xl">2XL (24px)</p>
<p className="text-4xl">4XL (36px)</p>

// Font weight
<p className="font-normal">Normal (400)</p>
<p className="font-medium">Medium (500)</p>
<p className="font-semibold">Semibold (600)</p>
<p className="font-bold">Bold (700)</p>

// Text alignment
<p className="text-left">Left</p>
<p className="text-center">Center</p>
<p className="text-right">Right</p>
```

---

# Why Tailwind + React/Next.js?

<v-clicks>

## Perfect Match

1. **Component-based** - Styles stay with components
2. **No CSS conflicts** - No global stylesheet issues
3. **Easy refactoring** - Delete component, styles go with it
4. **Fast development** - No switching between files
5. **Built-in with Next.js** - Zero configuration needed

</v-clicks>

---

# Quick Reference Card

<div class="grid grid-cols-2 gap-4 text-sm">

<div>

**Spacing**
- `p-{n}` / `m-{n}` - padding/margin
- `px-` / `py-` - horizontal/vertical
- `pt-` / `pb-` / `pl-` / `pr-` - single side

**Sizing**
- `w-full` / `h-full` - 100%
- `w-screen` / `h-screen` - viewport
- `max-w-{size}` - max width
- `min-h-screen` - minimum full height

**Display**
- `flex` / `grid` / `block` / `hidden`
- `items-center` / `justify-center`

</div>

<div>

**Colors**
- `text-{color}-{shade}`
- `bg-{color}-{shade}`
- `border-{color}-{shade}`

**Borders**
- `border` / `border-2`
- `rounded` / `rounded-lg` / `rounded-full`

**Effects**
- `shadow` / `shadow-md` / `shadow-lg`
- `opacity-{n}`
- `transition` / `duration-{ms}`

</div>

</div>

---

# Pro Tips

<v-clicks>

1. **Use VS Code extension** - "Tailwind CSS IntelliSense" for autocomplete

2. **Check the docs** - tailwindcss.com/docs has everything

3. **Don't memorize** - Look up what you need, you'll learn naturally

4. **Start simple** - Master flex, padding, margin, colors first

5. **Long class names are OK** - It looks verbose but it's explicit and maintainable

</v-clicks>

---

# Ready for Next.js!

Now when you see this during setup:

```bash
npx create-next-app@latest my-app

✔ Would you like to use Tailwind CSS? Yes  ← Say Yes!
```

You'll know exactly what you're getting!

**Next: Setting up your first Next.js project**
