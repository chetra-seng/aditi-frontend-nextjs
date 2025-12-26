# TaskFlow Data Features

<div class="grid grid-cols-2 gap-8">

<div>

## Features We'll Add

- Fetch mocks data from JSON server
- Create new tasks with a form
- Update task status
- Delete tasks
- Form validation

</div>

<div>

## What You'll Practice

- Server Components for fetching
- Server Actions for mutations
- React Hook Form
- Zod validation
- Loading & error states
- Cache revalidation

</div>

</div>

---
layout: center
---

# Form Handling in React

The problem with forms...

---

# Forms Without Libraries

```tsx
'use client';

export default function CreateTaskForm() {
  const [title, setTitle] = useState('');
  const [description, setDescription] = useState('');
  const [errors, setErrors] = useState({});

  const handleSubmit = (e) => {
    e.preventDefault();
    // Manual validation for every field...
    // Error state management...
    // Submit logic...
  };

  return (/* lots of boilerplate... */);
}
```

---

# Problems with Manual Forms

<v-clicks>

- **Boilerplate**: useState for every field
- **Validation**: Manual checks, easy to miss cases
- **Error handling**: Managing error state is tedious
- **Performance**: Re-renders on every keystroke
- **Type safety**: No TypeScript integration
- **Scalability**: Gets messy with complex forms

</v-clicks>

---
layout: center
---

# React Hook Form

Performant, flexible forms with easy validation

---

# What is React Hook Form?

<v-clicks>

- **Minimal re-renders** - uses uncontrolled inputs
- **Easy validation** - built-in or custom rules
- **Small bundle** - ~9kb minified
- **TypeScript support** - full type inference
- **Flexible** - works with any UI library
- **Battle-tested** - 37k+ GitHub stars

</v-clicks>

---

# Installing React Hook Form

```bash
npm install react-hook-form
```

---

# Basic React Hook Form

```tsx
'use client';

import { useForm } from 'react-hook-form';

export default function CreateTaskForm() {
  const { register, handleSubmit, formState: { errors } } = useForm();

  const onSubmit = (data) => {
    console.log(data); // { title: '...', description: '...' }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('title', { required: 'Title is required' })} />
      {errors.title && <span>{errors.title.message}</span>}

      <textarea {...register('description', { required: true })} />
      {errors.description && <span>Description is required</span>}

      <button type="submit">Create Task</button>
    </form>
  );
}
```

---

# React Hook Form Validation

```tsx
const { register, handleSubmit, formState: { errors } } = useForm();

// Built-in validation rules
<input {...register('title', {
  required: 'Title is required',
  minLength: { value: 3, message: 'At least 3 characters' },
  maxLength: { value: 100, message: 'Maximum 100 characters' },
})} />

<input {...register('email', {
  required: 'Email is required',
  pattern: {
    value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
    message: 'Invalid email address'
  }
})} />

<input type="number" {...register('priority', {
  min: { value: 1, message: 'Minimum priority is 1' },
  max: { value: 5, message: 'Maximum priority is 5' },
})} />
```

---
layout: center
---

# Zod

TypeScript-first schema validation

---

# What is Zod?

<v-clicks>

- **Schema declaration** - define your data shape
- **Validation** - validate data against schemas
- **Type inference** - TypeScript types from schemas
- **Composable** - build complex schemas from simple ones
- **Zero dependencies** - lightweight and fast
- **Great errors** - detailed, customizable error messages

</v-clicks>

---

# Installing Zod

```bash
npm install zod
```

---

# Basic Zod Schemas

```tsx
import { z } from 'zod';

// Define a schema
const taskSchema = z.object({
  title: z.string().min(3, 'Title must be at least 3 characters'),
  description: z.string().min(10, 'Description must be at least 10 characters'),
  priority: z.number().min(1).max(5),
  dueDate: z.string().optional(),
  tags: z.array(z.string()).default([]),
});

// Infer TypeScript type from schema
type Task = z.infer<typeof taskSchema>;
// { title: string; description: string; priority: number; dueDate?: string; tags: string[] }

// Validate data
const result = taskSchema.safeParse(formData);
if (result.success) {
  console.log(result.data); // Typed as Task
} else {
  console.log(result.error.errors); // Validation errors
}
```

---

# Zod Validation Methods

```tsx
import { z } from 'zod';

// Strings
z.string().min(1).max(100)
z.string().email('Invalid email')
z.string().url('Invalid URL')
z.string().regex(/pattern/)

// Numbers
z.number().positive()
z.number().int()
z.number().min(0).max(100)

// Others
z.boolean()
z.date()
z.enum(['low', 'medium', 'high'])
z.literal('specific-value')

// Optional & nullable
z.string().optional()      // string | undefined
z.string().nullable()      // string | null
z.string().nullish()       // string | null | undefined
```

---
layout: center
---

# React Hook Form + Zod

The perfect combination

---

# Why Combine Them?

<div class="grid grid-cols-2 gap-8">

<div>

## React Hook Form

- Form state management
- Minimal re-renders
- Easy field registration
- Submit handling

</div>

<div>

## Zod

- Schema definition
- Type inference
- Detailed validation
- Reusable across app

</div>

</div>

<v-click>

**Together**: Type-safe forms with powerful validation and great DX!

</v-click>

---

# Installing the Resolver

```bash
npm install react-hook-form zod @hookform/resolvers
```

---

# React Hook Form + Zod Example

```tsx
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const taskSchema = z.object({
  title: z.string().min(3),
  description: z.string().min(10),
  priority: z.enum(['low', 'medium', 'high']),
});

type TaskFormData = z.infer<typeof taskSchema>;

export default function CreateTaskForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<TaskFormData>({
    resolver: zodResolver(taskSchema),  // Connect Zod to React Hook Form
  });

  return (/* form JSX */);
}
```

---

# Complete Form Example

```tsx
export default function CreateTaskForm() {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm<TaskFormData>({
    resolver: zodResolver(taskSchema),
    defaultValues: { priority: 'medium' },
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <Input {...register('title')} placeholder="Task title" />
        {errors.title && <p className="text-red-500 text-sm">{errors.title.message}</p>}
      </div>

      <div>
        <Textarea {...register('description')} placeholder="Description" />
        {errors.description && <p className="text-red-500 text-sm">{errors.description.message}</p>}
      </div>

      <Button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Creating...' : 'Create Task'}
      </Button>
    </form>
  );
}
```

---

# Form Evolution

````md magic-move
```tsx
// ❌ Without libraries
'use client';
import { useState } from 'react';

export default function CreateTaskForm() {
  const [title, setTitle] = useState('');
  const [errors, setErrors] = useState({});

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!title || title.length < 3) {
      setErrors({ title: 'Min 3 characters' });
      return;
    }
    // submit...
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={title} onChange={e => setTitle(e.target.value)} />
      {errors.title && <p>{errors.title}</p>}
      <button type="submit">Create</button>
    </form>
  );
}
```

```tsx
// ✅ With React Hook Form
'use client';
import { useForm } from 'react-hook-form';

export default function CreateTaskForm() {
  const { register, handleSubmit, formState: { errors } } = useForm();

  return (
    <form onSubmit={handleSubmit((data) => { /* submit */ })}>
      <input {...register('title', { required: true, minLength: 3 })} />
      {errors.title && <p>{errors.title.message}</p>}
      <button type="submit">Create</button>
    </form>
  );
}
```

```tsx
// ✅✅ With React Hook Form + Zod
'use client';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({ title: z.string().min(3) });

export default function CreateTaskForm() {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(schema),
  });

  return (
    <form onSubmit={handleSubmit((data) => { /* submit */ })}>
      <input {...register('title')} />
      {errors.title && <p>{errors.title.message}</p>}
      <button type="submit">Create</button>
    </form>
  );
}
```
````

---
layout: center
---

# Let's Build!

Follow along step by step

---

# What You Learned

<div class="grid grid-cols-2 gap-8">

<div>

## Data Fetching

- Server Components for data
- Server Actions for mutations
- Cache revalidation
- Loading & error states
- Form handling patterns

</div>

<div>

## Tools & Libraries

- React Hook Form setup
- Zod schema validation
- Form + Zod integration
- Type-safe forms
- Server-side validation

</div>

</div>

