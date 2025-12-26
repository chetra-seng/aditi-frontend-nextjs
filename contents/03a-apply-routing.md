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

# Project Preview

```
TaskFlow/
├── Dashboard         → Overview of all tasks
├── Tasks/
│   ├── All Tasks     → List view with filters
│   ├── Task #1       → Dynamic task page
│   └── New Task      → Create task form
└── Projects/
    ├── All Projects  → Project list
    └── Project Page  → Tasks grouped by project
```

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

# Step 1: Project Setup

```bash
# Create a new Next.js app
npx create-next-app@latest taskflow

# Options:
✔ Would you like to use TypeScript? Yes
✔ Would you like to use ESLint? Yes
✔ Would you like to use Tailwind CSS? Yes
✔ Would you like to use `src/` directory? No
✔ Would you like to use App Router? Yes
✔ Would you like to customize the default import alias? No

# Navigate to project
cd taskflow

# Initialize shadcn
npx shadcn@latest init

# Add components we'll need
npx shadcn@latest add button card input badge checkbox separator
```

---

# Step 2: Create the Folder Structure

```bash
# Create your route folders
mkdir -p app/tasks/[id]
mkdir -p app/tasks/new
mkdir -p app/projects/[slug]
mkdir -p app/\(dashboard\)

# Create components folder
mkdir -p components/tasks
mkdir -p components/layout
```

Your structure should look like:

```
app/
├── (dashboard)/
├── tasks/
│   ├── [id]/
│   └── new/
├── projects/
│   └── [slug]/
├── layout.tsx
└── page.tsx
```

---

# Step 3: Create the Root Layout

```tsx
// app/layout.tsx
import type { Metadata } from "next";
import { Inter } from "next/font/google";
import "./globals.css";

const inter = Inter({ subsets: ["latin"] });

export const metadata: Metadata = {
  title: "TaskFlow - Task Manager",
  description: "A simple task management app built with Next.js",
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <div className="min-h-screen bg-gray-50">
          {children}
        </div>
      </body>
    </html>
  );
}
```

---

# Step 4: Create a Navigation Component

```tsx
// components/layout/navbar.tsx
import Link from "next/link";
import { Button } from "@/components/ui/button";

export function Navbar() {
  return (
    <nav className="border-b bg-white">
      <div className="container mx-auto px-4 py-3 flex items-center justify-between">
        <Link href="/" className="text-xl font-bold">
          TaskFlow
        </Link>

        <div className="flex items-center gap-4">
          <Link href="/tasks">
            <Button variant="ghost">Tasks</Button>
          </Link>
          <Link href="/projects">
            <Button variant="ghost">Projects</Button>
          </Link>
          <Link href="/tasks/new">
            <Button>New Task</Button>
          </Link>
        </div>
      </div>
    </nav>
  );
}
```

---

# Step 5: Create the Dashboard Page

```tsx
// app/page.tsx
import Link from "next/link";
import { Button } from "@/components/ui/button";
import { Card, CardHeader, CardTitle, CardContent } from "@/components/ui/card";
import { Navbar } from "@/components/layout/navbar";

export default function DashboardPage() {
  return (
    <>
      <Navbar />
      <main className="container mx-auto px-4 py-8">
        <h1 className="text-3xl font-bold mb-8">Dashboard</h1>

        <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
          <Card>
            <CardHeader>
              <CardTitle>Total Tasks</CardTitle>
            </CardHeader>
            <CardContent>
              <p className="text-4xl font-bold">12</p>
            </CardContent>
          </Card>

          <Card>
            <CardHeader>
              <CardTitle>In Progress</CardTitle>
            </CardHeader>
            <CardContent>
              <p className="text-4xl font-bold text-yellow-600">5</p>
            </CardContent>
          </Card>

          <Card>
            <CardHeader>
              <CardTitle>Completed</CardTitle>
            </CardHeader>
            <CardContent>
              <p className="text-4xl font-bold text-green-600">7</p>
            </CardContent>
          </Card>
        </div>

        <div className="mt-8">
          <Link href="/tasks">
            <Button>View All Tasks</Button>
          </Link>
        </div>
      </main>
    </>
  );
}
```

---

# Step 6: Create the Tasks List Page

```tsx
// app/tasks/page.tsx
import Link from "next/link";
import { Button } from "@/components/ui/button";
import { Card, CardHeader, CardTitle, CardContent } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Navbar } from "@/components/layout/navbar";

// Mock data - later we'll fetch this from an API
const tasks = [
  { id: "1", title: "Learn Next.js Routing", status: "completed", priority: "high" },
  { id: "2", title: "Build TaskFlow App", status: "in-progress", priority: "high" },
  { id: "3", title: "Add Authentication", status: "todo", priority: "medium" },
  { id: "4", title: "Deploy to Vercel", status: "todo", priority: "low" },
];

export default function TasksPage() {
  return (
    <>
      <Navbar />
      <main className="container mx-auto px-4 py-8">
        <div className="flex justify-between items-center mb-8">
          <h1 className="text-3xl font-bold">All Tasks</h1>
          <Link href="/tasks/new">
            <Button>New Task</Button>
          </Link>
        </div>

        <div className="space-y-4">
          {tasks.map((task) => (
            <Link key={task.id} href={`/tasks/${task.id}`}>
              <Card className="hover:shadow-md transition-shadow cursor-pointer">
                <CardHeader>
                  <CardTitle className="flex items-center justify-between">
                    <span>{task.title}</span>
                    <div className="flex gap-2">
                      <Badge variant={
                        task.status === "completed" ? "default" :
                        task.status === "in-progress" ? "secondary" : "outline"
                      }>
                        {task.status}
                      </Badge>
                      <Badge variant={
                        task.priority === "high" ? "destructive" :
                        task.priority === "medium" ? "secondary" : "outline"
                      }>
                        {task.priority}
                      </Badge>
                    </div>
                  </CardTitle>
                </CardHeader>
              </Card>
            </Link>
          ))}
        </div>
      </main>
    </>
  );
}
```

---

# Step 7: Create Dynamic Task Page

```tsx
// app/tasks/[id]/page.tsx
import Link from "next/link";
import { Button } from "@/components/ui/button";
import { Card, CardHeader, CardTitle, CardContent } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Checkbox } from "@/components/ui/checkbox";
import { Navbar } from "@/components/layout/navbar";

// Mock function - later we'll fetch from database
function getTask(id: string) {
  const tasks: Record<string, any> = {
    "1": {
      id: "1",
      title: "Learn Next.js Routing",
      description: "Master file-based routing, dynamic routes, and navigation in Next.js",
      status: "completed",
      priority: "high",
      subtasks: [
        { id: "1-1", title: "Read documentation", completed: true },
        { id: "1-2", title: "Build practice project", completed: true },
        { id: "1-3", title: "Create dynamic routes", completed: false },
      ],
    },
    "2": {
      id: "2",
      title: "Build TaskFlow App",
      description: "Create a full task management application with Next.js",
      status: "in-progress",
      priority: "high",
      subtasks: [
        { id: "2-1", title: "Setup project", completed: true },
        { id: "2-2", title: "Create routes", completed: false },
        { id: "2-3", title: "Add styling", completed: false },
      ],
    },
  };
  return tasks[id] || null;
}

export default async function TaskPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  const task = getTask(id);

  if (!task) {
    return (
      <>
        <Navbar />
        <main className="container mx-auto px-4 py-8">
          <h1 className="text-2xl font-bold">Task not found</h1>
          <Link href="/tasks">
            <Button className="mt-4">Back to Tasks</Button>
          </Link>
        </main>
      </>
    );
  }

  return (
    <>
      <Navbar />
      <main className="container mx-auto px-4 py-8">
        <Link href="/tasks">
          <Button variant="ghost" className="mb-4">
            ← Back to Tasks
          </Button>
        </Link>

        <Card>
          <CardHeader>
            <CardTitle className="flex items-center justify-between">
              <span className="text-2xl">{task.title}</span>
              <div className="flex gap-2">
                <Badge>{task.status}</Badge>
                <Badge variant="outline">{task.priority}</Badge>
              </div>
            </CardTitle>
          </CardHeader>
          <CardContent>
            <p className="text-gray-600 mb-6">{task.description}</p>

            <h3 className="font-semibold mb-4">Subtasks</h3>
            <div className="space-y-3">
              {task.subtasks.map((subtask: any) => (
                <div key={subtask.id} className="flex items-center gap-3">
                  <Checkbox checked={subtask.completed} />
                  <span className={subtask.completed ? "line-through text-gray-400" : ""}>
                    {subtask.title}
                  </span>
                </div>
              ))}
            </div>
          </CardContent>
        </Card>
      </main>
    </>
  );
}
```

---

# Step 8: Create New Task Page

```tsx
// app/tasks/new/page.tsx
import Link from "next/link";
import { Button } from "@/components/ui/button";
import { Card, CardHeader, CardTitle, CardContent } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Navbar } from "@/components/layout/navbar";

export default function NewTaskPage() {
  return (
    <>
      <Navbar />
      <main className="container mx-auto px-4 py-8 max-w-2xl">
        <Link href="/tasks">
          <Button variant="ghost" className="mb-4">
            ← Back to Tasks
          </Button>
        </Link>

        <Card>
          <CardHeader>
            <CardTitle>Create New Task</CardTitle>
          </CardHeader>
          <CardContent>
            <form className="space-y-4">
              <div>
                <label className="block text-sm font-medium mb-2">
                  Task Title
                </label>
                <Input
                  type="text"
                  placeholder="Enter task title..."
                  name="title"
                />
              </div>

              <div>
                <label className="block text-sm font-medium mb-2">
                  Description
                </label>
                <textarea
                  className="w-full min-h-[100px] px-3 py-2 border rounded-md"
                  placeholder="Describe your task..."
                  name="description"
                />
              </div>

              <div className="grid grid-cols-2 gap-4">
                <div>
                  <label className="block text-sm font-medium mb-2">
                    Priority
                  </label>
                  <select
                    name="priority"
                    className="w-full px-3 py-2 border rounded-md"
                  >
                    <option value="low">Low</option>
                    <option value="medium">Medium</option>
                    <option value="high">High</option>
                  </select>
                </div>

                <div>
                  <label className="block text-sm font-medium mb-2">
                    Status
                  </label>
                  <select
                    name="status"
                    className="w-full px-3 py-2 border rounded-md"
                  >
                    <option value="todo">To Do</option>
                    <option value="in-progress">In Progress</option>
                    <option value="completed">Completed</option>
                  </select>
                </div>
              </div>

              <div className="flex gap-4 pt-4">
                <Button type="submit">Create Task</Button>
                <Link href="/tasks">
                  <Button variant="outline">Cancel</Button>
                </Link>
              </div>
            </form>
          </CardContent>
        </Card>
      </main>
    </>
  );
}
```

---

# Step 9: Add Loading State

```tsx
// app/tasks/[id]/loading.tsx
import { Card, CardHeader, CardContent } from "@/components/ui/card";
import { Navbar } from "@/components/layout/navbar";

export default function TaskLoading() {
  return (
    <>
      <Navbar />
      <main className="container mx-auto px-4 py-8">
        <div className="h-10 w-32 bg-gray-200 rounded animate-pulse mb-4" />

        <Card>
          <CardHeader>
            <div className="h-8 w-3/4 bg-gray-200 rounded animate-pulse" />
          </CardHeader>
          <CardContent>
            <div className="space-y-4">
              <div className="h-4 w-full bg-gray-200 rounded animate-pulse" />
              <div className="h-4 w-2/3 bg-gray-200 rounded animate-pulse" />
              <div className="h-4 w-1/2 bg-gray-200 rounded animate-pulse" />
            </div>
          </CardContent>
        </Card>
      </main>
    </>
  );
}
```

---

# Step 10: Add Error Handling

```tsx
// app/tasks/[id]/error.tsx
"use client";

import { Button } from "@/components/ui/button";
import { Card, CardHeader, CardTitle, CardContent } from "@/components/ui/card";
import { Navbar } from "@/components/layout/navbar";

export default function TaskError({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <>
      <Navbar />
      <main className="container mx-auto px-4 py-8">
        <Card className="border-red-200 bg-red-50">
          <CardHeader>
            <CardTitle className="text-red-600">
              Something went wrong!
            </CardTitle>
          </CardHeader>
          <CardContent>
            <p className="text-gray-600 mb-4">{error.message}</p>
            <Button onClick={reset} variant="outline">
              Try again
            </Button>
          </CardContent>
        </Card>
      </main>
    </>
  );
}
```

---

# Step 11: Add Not Found Page

```tsx
// app/tasks/[id]/not-found.tsx
import Link from "next/link";
import { Button } from "@/components/ui/button";
import { Card, CardHeader, CardTitle, CardContent } from "@/components/ui/card";
import { Navbar } from "@/components/layout/navbar";

export default function TaskNotFound() {
  return (
    <>
      <Navbar />
      <main className="container mx-auto px-4 py-8">
        <Card>
          <CardHeader>
            <CardTitle>Task Not Found</CardTitle>
          </CardHeader>
          <CardContent>
            <p className="text-gray-600 mb-4">
              The task you're looking for doesn't exist or has been deleted.
            </p>
            <Link href="/tasks">
              <Button>Back to Tasks</Button>
            </Link>
          </CardContent>
        </Card>
      </main>
    </>
  );
}
```

---

# Your Final Structure

```
app/
├── layout.tsx              # Root layout
├── page.tsx                # Dashboard (/)
├── tasks/
│   ├── page.tsx            # Tasks list (/tasks)
│   ├── new/
│   │   └── page.tsx        # New task form (/tasks/new)
│   └── [id]/
│       ├── page.tsx        # Dynamic task page (/tasks/:id)
│       ├── loading.tsx     # Loading state
│       ├── error.tsx       # Error handling
│       └── not-found.tsx   # 404 for task
├── projects/
│   ├── page.tsx            # Projects list
│   └── [slug]/
│       └── page.tsx        # Project detail
components/
├── layout/
│   └── navbar.tsx          # Navigation component
└── ui/                     # shadcn components
    ├── button.tsx
    ├── card.tsx
    ├── input.tsx
    ├── badge.tsx
    └── checkbox.tsx
```

---

# Challenge: Extend TaskFlow!

<v-clicks>

## Try adding these features:

1. **Projects Page** - Create `/projects` and `/projects/[slug]` routes

2. **Task Layout** - Add a shared layout for all `/tasks/*` routes with a sidebar

3. **Search with Query Params** - Add `/tasks?status=completed&priority=high`

4. **Route Groups** - Organize with `(dashboard)` and `(auth)` groups

5. **Parallel Routes** - Show task list and task detail side by side

</v-clicks>

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

---
layout: center
---

# Great Job!

You've built the foundation of TaskFlow!

Next up: **Data Fetching** - We'll connect to a real database!
