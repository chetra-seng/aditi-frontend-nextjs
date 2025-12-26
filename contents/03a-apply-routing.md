# Practice: Build TaskFlow

Let's put your routing knowledge into practice!

---
layout: center
---

# What We're Building

## TaskFlow - A Task Manager App

Think **ClickUp** or **Todoist**, but simpler!

---

# TaskFlow Features

<div class="grid grid-cols-2 gap-8">

<div>

## Pages We'll Create

- `/` - Dashboard (task overview)
- `/tasks` - All tasks list
- `/tasks/[id]` - Individual task page
- `/tasks/new` - Create new task
- `/projects` - Projects list
- `/projects/[slug]` - Project details

</div>

<div>

## What You'll Practice

- File-based routing
- Dynamic routes `[id]`
- Route groups `(group)`
- Layouts
- Loading & Error states
- Navigation with `Link`

</div>

</div>

---
layout: center
---

# Setting Up shadcn/ui

Beautiful, accessible components for our app

---

# What is shadcn/ui?

<v-clicks>

- **NOT a component library** - it's a collection of reusable components
- Components are **copied into your project** (you own the code!)
- Built on **Radix UI** (accessible primitives)
- Styled with **Tailwind CSS**
- Fully **customizable**
- Used by Vercel, Supabase, and many more

</v-clicks>

---

# Installing shadcn/ui

```bash
# Make sure you have a Next.js project with Tailwind CSS
# (create-next-app already sets this up)

# Run the shadcn init command
npx shadcn@latest init
```

You'll be prompted with questions:

```bash
✔ Which style would you like to use? › New York
✔ Which color would you like to use as base color? › Neutral
✔ Would you like to use CSS variables for colors? › yes
```

---

# What shadcn Creates

```
my-next-app/
├── components/
│   └── ui/              # shadcn components go here
├── lib/
│   └── utils.ts         # cn() helper function
├── components.json      # shadcn configuration
└── app/
    └── globals.css      # Updated with CSS variables
```

---

# The cn() Helper

```tsx
// lib/utils.ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

**What it does:**
- Merges Tailwind classes intelligently
- Handles conditional classes
- Prevents class conflicts

```tsx
// Example usage
cn("px-4 py-2", isActive && "bg-blue-500", className)
```

---

# Adding Components

```bash
# Add individual components as needed
npx shadcn@latest add button
npx shadcn@latest add card
npx shadcn@latest add input
npx shadcn@latest add badge
npx shadcn@latest add checkbox

# Or add multiple at once
npx shadcn@latest add button card input badge checkbox
```

Components are added to `components/ui/`

---

# Using shadcn Components

```tsx
import { Button } from "@/components/ui/button";
import { Card, CardHeader, CardTitle, CardContent } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";

export default function TaskCard() {
  return (
    <Card>
      <CardHeader>
        <CardTitle className="flex items-center gap-2">
          Build TaskFlow App
          <Badge variant="secondary">In Progress</Badge>
        </CardTitle>
      </CardHeader>
      <CardContent>
        <p>Learn Next.js routing by building a real app!</p>
        <Button className="mt-4">View Task</Button>
      </CardContent>
    </Card>
  );
}
```

---
layout: center
---

# Let's Build TaskFlow!

Follow along step by step

---


# What You Learned

<div class="grid grid-cols-2 gap-8">

<div>

## Routing Concepts

- File-based routing
- Dynamic routes `[id]`
- Navigation with `Link`
- Loading states
- Error boundaries
- Not found pages

</div>

<div>

## Tools & Libraries

- shadcn/ui setup
- Component installation
- The `cn()` helper
- Tailwind CSS styling
- TypeScript in Next.js

</div>

</div>


