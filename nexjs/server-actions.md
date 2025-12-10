# Next.js Server Actions - Complete Guide

## What are Server Actions?

Server Actions are asynchronous functions that run on the server in Next.js applications. Introduced in Next.js 13.4 and stabilized in Next.js 14, they allow you to write server-side logic directly in your components without creating separate API routes. Server Actions are built on top of React Actions and provide a seamless way to handle form submissions, data mutations, and server-side operations.

## Why Use Server Actions?

### 1. **Simplified Data Mutations**

- Write server-side logic directly in components
- No need to create separate API routes for simple operations
- Automatic handling of POST requests and form submissions
- Type-safe data mutations with TypeScript

### 2. **Progressive Enhancement**

- Forms work without JavaScript enabled
- Automatic fallback to traditional form submission
- Enhanced user experience when JavaScript is available
- Built-in loading and error states

### 3. **Security**

- Server-side validation and authentication
- Sensitive operations never exposed to the client
- Automatic CSRF protection
- Secure by default

### 4. **Developer Experience**

- Colocation of server logic with UI components
- Reduced boilerplate code
- Better code organization
- Seamless integration with React Server Components

### 5. **Performance**

- Reduced client-side JavaScript bundle
- Efficient data revalidation
- Optimistic updates support
- Automatic request deduplication

## Key Concepts

### Server Actions vs API Routes

| Feature              | Server Actions                    | API Routes                  |
| -------------------- | --------------------------------- | --------------------------- |
| Location             | Inline or separate files          | `app/api/` directory        |
| Use case             | Form submissions, data mutations  | Complex APIs, webhooks      |
| Client integration   | Direct function calls             | Fetch requests              |
| Progressive enhance. | Built-in                          | Manual implementation       |
| Type safety          | End-to-end TypeScript             | Requires manual typing      |
| Boilerplate          | Minimal                           | More setup required         |
| Revalidation         | Built-in with `revalidatePath()`  | Manual cache invalidation   |

### Where to Define Server Actions

**1. Server Components (Inline)**

```tsx
// app/posts/page.tsx
export default function PostsPage() {
  async function createPost(formData: FormData) {
    'use server';
    
    const title = formData.get('title');
    // Server-side logic here
  }

  return (
    <form action={createPost}>
      <input name="title" />
      <button type="submit">Create</button>
    </form>
  );
}
```

**2. Separate Files (Recommended for Reusability)**

```tsx
// app/actions/posts.ts
'use server';

export async function createPost(formData: FormData) {
  const title = formData.get('title');
  // Server-side logic here
}

export async function deletePost(postId: string) {
  // Server-side logic here
}
```

**3. Client Components (Import Only)**

```tsx
// app/components/PostForm.tsx
'use client';

import { createPost } from '@/app/actions/posts';

export default function PostForm() {
  return (
    <form action={createPost}>
      <input name="title" />
      <button type="submit">Create</button>
    </form>
  );
}
```

## Basic Usage

### Simple Form Submission

```tsx
// app/actions/posts.ts
'use server';

import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;

  // Validate data
  if (!title || !content) {
    throw new Error('Title and content are required');
  }

  // Save to database
  await db.posts.create({
    data: { title, content },
  });

  // Revalidate the posts page cache
  revalidatePath('/posts');
  
  // Redirect to the posts page
  redirect('/posts');
}
```

```tsx
// app/posts/new/page.tsx
import { createPost } from '@/app/actions/posts';

export default function NewPostPage() {
  return (
    <form action={createPost}>
      <input type="text" name="title" placeholder="Title" required />
      <textarea name="content" placeholder="Content" required />
      <button type="submit">Create Post</button>
    </form>
  );
}
```

### Programmatic Invocation

Server Actions can be called programmatically from Client Components:

```tsx
// app/components/DeleteButton.tsx
'use client';

import { deletePost } from '@/app/actions/posts';
import { useTransition } from 'react';

export default function DeleteButton({ postId }: { postId: string }) {
  const [isPending, startTransition] = useTransition();

  const handleDelete = () => {
    startTransition(async () => {
      await deletePost(postId);
    });
  };

  return (
    <button onClick={handleDelete} disabled={isPending}>
      {isPending ? 'Deleting...' : 'Delete'}
    </button>
  );
}
```

## Advanced Patterns

### 1. **Type-Safe Server Actions with Zod**

```tsx
// app/actions/posts.ts
'use server';

import { z } from 'zod';
import { revalidatePath } from 'next/cache';

const PostSchema = z.object({
  title: z.string().min(3).max(100),
  content: z.string().min(10),
  published: z.boolean().default(false),
});

export async function createPost(formData: FormData) {
  // Parse and validate
  const validatedFields = PostSchema.safeParse({
    title: formData.get('title'),
    content: formData.get('content'),
    published: formData.get('published') === 'on',
  });

  // Return errors if validation fails
  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
      message: 'Validation failed',
    };
  }

  // Save to database
  try {
    await db.posts.create({
      data: validatedFields.data,
    });
  } catch (error) {
    return {
      message: 'Database error: Failed to create post',
    };
  }

  revalidatePath('/posts');
  return { message: 'Post created successfully' };
}
```

### 2. **Optimistic Updates**

```tsx
// app/components/TodoList.tsx
'use client';

import { useOptimistic } from 'react';
import { addTodo } from '@/app/actions/todos';

export default function TodoList({ todos }: { todos: Todo[] }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo: string) => [
      ...state,
      { id: Date.now(), text: newTodo, completed: false },
    ]
  );

  async function formAction(formData: FormData) {
    const text = formData.get('text') as string;
    addOptimisticTodo(text);
    await addTodo(text);
  }

  return (
    <div>
      <ul>
        {optimisticTodos.map((todo) => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
      <form action={formAction}>
        <input name="text" placeholder="Add todo" />
        <button type="submit">Add</button>
      </form>
    </div>
  );
}
```

### 3. **Server Actions with Authentication**

```tsx
// app/actions/posts.ts
'use server';

import { auth } from '@/lib/auth';
import { revalidatePath } from 'next/cache';

export async function createPost(formData: FormData) {
  // Check authentication
  const session = await auth();
  
  if (!session?.user) {
    throw new Error('Unauthorized');
  }

  const title = formData.get('title') as string;
  const content = formData.get('content') as string;

  // Create post with user ID
  await db.posts.create({
    data: {
      title,
      content,
      authorId: session.user.id,
    },
  });

  revalidatePath('/posts');
}
```

### 4. **Returning Data from Server Actions**

```tsx
// app/actions/search.ts
'use server';

export async function searchPosts(query: string) {
  const posts = await db.posts.findMany({
    where: {
      OR: [
        { title: { contains: query } },
        { content: { contains: query } },
      ],
    },
  });

  return posts;
}
```

```tsx
// app/components/SearchForm.tsx
'use client';

import { searchPosts } from '@/app/actions/search';
import { useState } from 'react';

export default function SearchForm() {
  const [results, setResults] = useState([]);

  async function handleSearch(formData: FormData) {
    const query = formData.get('query') as string;
    const posts = await searchPosts(query);
    setResults(posts);
  }

  return (
    <div>
      <form action={handleSearch}>
        <input name="query" placeholder="Search..." />
        <button type="submit">Search</button>
      </form>
      <ul>
        {results.map((post) => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

### 5. **Handling File Uploads**

```tsx
// app/actions/upload.ts
'use server';

import { writeFile } from 'fs/promises';
import { join } from 'path';

export async function uploadImage(formData: FormData) {
  const file = formData.get('image') as File;
  
  if (!file) {
    throw new Error('No file uploaded');
  }

  // Validate file type
  if (!file.type.startsWith('image/')) {
    throw new Error('Only images are allowed');
  }

  // Validate file size (5MB max)
  if (file.size > 5 * 1024 * 1024) {
    throw new Error('File size must be less than 5MB');
  }

  const bytes = await file.arrayBuffer();
  const buffer = Buffer.from(bytes);

  // Save file
  const path = join(process.cwd(), 'public', 'uploads', file.name);
  await writeFile(path, buffer);

  return { url: `/uploads/${file.name}` };
}
```

```tsx
// app/components/ImageUpload.tsx
'use client';

import { uploadImage } from '@/app/actions/upload';
import { useState } from 'react';

export default function ImageUpload() {
  const [imageUrl, setImageUrl] = useState('');

  async function handleUpload(formData: FormData) {
    const result = await uploadImage(formData);
    setImageUrl(result.url);
  }

  return (
    <div>
      <form action={handleUpload}>
        <input type="file" name="image" accept="image/*" />
        <button type="submit">Upload</button>
      </form>
      {imageUrl && <img src={imageUrl} alt="Uploaded" />}
    </div>
  );
}
```

## Form Validation Patterns

### Client-Side + Server-Side Validation

```tsx
// app/components/PostForm.tsx
'use client';

import { createPost } from '@/app/actions/posts';
import { useFormState, useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Creating...' : 'Create Post'}
    </button>
  );
}

export default function PostForm() {
  const [state, formAction] = useFormState(createPost, { errors: {} });

  return (
    <form action={formAction}>
      <div>
        <label htmlFor="title">Title</label>
        <input
          id="title"
          name="title"
          type="text"
          required
        />
        {state.errors?.title && (
          <p className="error">{state.errors.title[0]}</p>
        )}
      </div>

      <div>
        <label htmlFor="content">Content</label>
        <textarea
          id="content"
          name="content"
          required
        />
        {state.errors?.content && (
          <p className="error">{state.errors.content[0]}</p>
        )}
      </div>

      <SubmitButton />
      
      {state.message && (
        <p className={state.errors ? 'error' : 'success'}>
          {state.message}
        </p>
      )}
    </form>
  );
}
```

## Revalidation Strategies

### 1. **Revalidate Specific Paths**

```tsx
'use server';

import { revalidatePath } from 'next/cache';

export async function updatePost(postId: string, formData: FormData) {
  // Update post in database
  await db.posts.update({
    where: { id: postId },
    data: {
      title: formData.get('title'),
      content: formData.get('content'),
    },
  });

  // Revalidate specific paths
  revalidatePath('/posts');
  revalidatePath(`/posts/${postId}`);
}
```

### 2. **Revalidate by Tag**

```tsx
'use server';

import { revalidateTag } from 'next/cache';

export async function createPost(formData: FormData) {
  await db.posts.create({
    data: {
      title: formData.get('title'),
      content: formData.get('content'),
    },
  });

  // Revalidate all data tagged with 'posts'
  revalidateTag('posts');
}
```

```tsx
// Fetch with tag
async function getPosts() {
  const res = await fetch('https://api.example.com/posts', {
    next: { tags: ['posts'] },
  });
  return res.json();
}
```

### 3. **Revalidate Entire Route**

```tsx
'use server';

import { revalidatePath } from 'next/cache';

export async function updateSettings(formData: FormData) {
  await db.settings.update({
    data: { theme: formData.get('theme') },
  });

  // Revalidate all routes under /dashboard
  revalidatePath('/dashboard', 'layout');
}
```

## Error Handling

### 1. **Try-Catch Pattern**

```tsx
'use server';

export async function createPost(formData: FormData) {
  try {
    const title = formData.get('title') as string;
    const content = formData.get('content') as string;

    await db.posts.create({
      data: { title, content },
    });

    revalidatePath('/posts');
    return { success: true, message: 'Post created' };
  } catch (error) {
    console.error('Failed to create post:', error);
    return { success: false, message: 'Failed to create post' };
  }
}
```

### 2. **Error Boundaries**

```tsx
// app/posts/error.tsx
'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

### 3. **Custom Error Handling**

```tsx
'use server';

class ValidationError extends Error {
  constructor(public errors: Record<string, string[]>) {
    super('Validation failed');
    this.name = 'ValidationError';
  }
}

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  
  if (!title || title.length < 3) {
    throw new ValidationError({
      title: ['Title must be at least 3 characters'],
    });
  }

  // Continue with post creation
}
```

## Best Practices

### 1. **Always Validate on the Server**

```tsx
'use server';

import { z } from 'zod';

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

export async function signup(formData: FormData) {
  // Never trust client data - always validate
  const validated = schema.parse({
    email: formData.get('email'),
    password: formData.get('password'),
  });

  // Proceed with validated data
}
```

### 2. **Use Separate Files for Organization**

```
app/
├── actions/
│   ├── posts.ts          # Post-related actions
│   ├── users.ts          # User-related actions
│   └── comments.ts       # Comment-related actions
├── posts/
│   └── page.tsx
└── users/
    └── page.tsx
```

### 3. **Implement Rate Limiting**

```tsx
'use server';

import { ratelimit } from '@/lib/redis';

export async function createPost(formData: FormData) {
  const session = await auth();
  
  // Rate limit: 5 posts per hour
  const { success } = await ratelimit.limit(
    `posts:${session.user.id}`,
    5,
    3600
  );

  if (!success) {
    throw new Error('Rate limit exceeded');
  }

  // Continue with post creation
}
```

### 4. **Use TypeScript for Type Safety**

```tsx
'use server';

type ActionResult = {
  success: boolean;
  message: string;
  data?: any;
  errors?: Record<string, string[]>;
};

export async function createPost(
  formData: FormData
): Promise<ActionResult> {
  // Implementation with typed return
  return {
    success: true,
    message: 'Post created',
  };
}
```

### 5. **Handle Loading States**

```tsx
'use client';

import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? (
        <>
          <Spinner />
          <span>Submitting...</span>
        </>
      ) : (
        'Submit'
      )}
    </button>
  );
}
```

### 6. **Implement Logging**

```tsx
'use server';

import { logger } from '@/lib/logger';

export async function deletePost(postId: string) {
  const session = await auth();
  
  logger.info('Deleting post', {
    postId,
    userId: session.user.id,
    timestamp: new Date().toISOString(),
  });

  try {
    await db.posts.delete({ where: { id: postId } });
    logger.info('Post deleted successfully', { postId });
  } catch (error) {
    logger.error('Failed to delete post', { postId, error });
    throw error;
  }
}
```

## Common Patterns

### 1. **Multi-Step Forms**

```tsx
'use server';

import { cookies } from 'next/headers';

export async function saveStep1(formData: FormData) {
  const data = {
    name: formData.get('name'),
    email: formData.get('email'),
  };

  // Store in cookie or session
  (await cookies()).set('formStep1', JSON.stringify(data));
  
  return { success: true };
}

export async function saveStep2(formData: FormData) {
  // Retrieve step 1 data
  const step1Data = JSON.parse(
    (await cookies()).get('formStep1')?.value || '{}'
  );

  // Combine and save
  await db.users.create({
    data: {
      ...step1Data,
      address: formData.get('address'),
      phone: formData.get('phone'),
    },
  });

  // Clear cookie
  (await cookies()).delete('formStep1');
}
```

### 2. **Batch Operations**

```tsx
'use server';

export async function bulkDeletePosts(postIds: string[]) {
  const session = await auth();

  // Verify ownership
  const posts = await db.posts.findMany({
    where: {
      id: { in: postIds },
      authorId: session.user.id,
    },
  });

  if (posts.length !== postIds.length) {
    throw new Error('Unauthorized');
  }

  // Delete in transaction
  await db.$transaction(
    postIds.map((id) => db.posts.delete({ where: { id } }))
  );

  revalidatePath('/posts');
}
```

### 3. **Conditional Redirects**

```tsx
'use server';

import { redirect } from 'next/navigation';

export async function createPost(formData: FormData) {
  const shouldPublish = formData.get('publish') === 'true';

  const post = await db.posts.create({
    data: {
      title: formData.get('title'),
      content: formData.get('content'),
      published: shouldPublish,
    },
  });

  revalidatePath('/posts');

  // Redirect based on action
  if (shouldPublish) {
    redirect(`/posts/${post.id}`);
  } else {
    redirect('/posts/drafts');
  }
}
```

## Performance Optimization

### 1. **Debounce Server Actions**

```tsx
'use client';

import { useDebouncedCallback } from 'use-debounce';
import { searchPosts } from '@/app/actions/search';

export default function SearchInput() {
  const handleSearch = useDebouncedCallback(async (query: string) => {
    const results = await searchPosts(query);
    // Update UI with results
  }, 300);

  return (
    <input
      type="search"
      onChange={(e) => handleSearch(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

### 2. **Parallel Server Actions**

```tsx
'use server';

export async function updateProfile(formData: FormData) {
  // Execute multiple operations in parallel
  await Promise.all([
    db.users.update({
      where: { id: userId },
      data: { name: formData.get('name') },
    }),
    uploadAvatar(formData.get('avatar')),
    sendNotification(userId, 'Profile updated'),
  ]);

  revalidatePath('/profile');
}
```

### 3. **Selective Revalidation**

```tsx
'use server';

export async function updatePost(postId: string, formData: FormData) {
  const oldPost = await db.posts.findUnique({ where: { id: postId } });
  const newTitle = formData.get('title');

  await db.posts.update({
    where: { id: postId },
    data: { title: newTitle },
  });

  // Only revalidate if title changed
  if (oldPost.title !== newTitle) {
    revalidatePath('/posts');
    revalidatePath(`/posts/${postId}`);
  }
}
```

## Testing Server Actions

### Unit Testing

```tsx
// __tests__/actions/posts.test.ts
import { createPost } from '@/app/actions/posts';
import { db } from '@/lib/db';

jest.mock('@/lib/db');

describe('createPost', () => {
  it('should create a post with valid data', async () => {
    const formData = new FormData();
    formData.append('title', 'Test Post');
    formData.append('content', 'Test content');

    const mockCreate = jest.fn().mockResolvedValue({
      id: '1',
      title: 'Test Post',
      content: 'Test content',
    });

    (db.posts.create as jest.Mock) = mockCreate;

    const result = await createPost(formData);

    expect(mockCreate).toHaveBeenCalledWith({
      data: {
        title: 'Test Post',
        content: 'Test content',
      },
    });
    expect(result.success).toBe(true);
  });

  it('should return error for invalid data', async () => {
    const formData = new FormData();
    formData.append('title', 'ab'); // Too short

    const result = await createPost(formData);

    expect(result.success).toBe(false);
    expect(result.errors?.title).toBeDefined();
  });
});
```

## Migration from API Routes

### Before (API Route)

```tsx
// pages/api/posts.ts
export default async function handler(req, res) {
  if (req.method === 'POST') {
    const { title, content } = req.body;
    
    const post = await db.posts.create({
      data: { title, content },
    });

    res.status(200).json(post);
  }
}
```

```tsx
// Client component
async function handleSubmit(e) {
  e.preventDefault();
  const res = await fetch('/api/posts', {
    method: 'POST',
    body: JSON.stringify({ title, content }),
  });
  const post = await res.json();
}
```

### After (Server Action)

```tsx
// app/actions/posts.ts
'use server';

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;

  const post = await db.posts.create({
    data: { title, content },
  });

  revalidatePath('/posts');
  return post;
}
```

```tsx
// Client component
import { createPost } from '@/app/actions/posts';

<form action={createPost}>
  <input name="title" />
  <textarea name="content" />
  <button type="submit">Create</button>
</form>
```

## Common Pitfalls

### 1. **❌ Forgetting 'use server' Directive**

```tsx
// Wrong - Missing directive
export async function createPost(formData: FormData) {
  // This won't work as a Server Action
}

// Correct
'use server';

export async function createPost(formData: FormData) {
  // Now it's a Server Action
}
```

### 2. **❌ Passing Non-Serializable Data**

```tsx
// Wrong - Functions can't be serialized
export async function updatePost(callback: () => void) {
  // Error: Functions can't be passed to Server Actions
}

// Correct - Use serializable data
export async function updatePost(postId: string, data: PostData) {
  // Works with serializable data
}
```

### 3. **❌ Not Handling Errors**

```tsx
// Wrong - No error handling
export async function createPost(formData: FormData) {
  await db.posts.create({ data: { ... } });
  // What if this fails?
}

// Correct - Proper error handling
export async function createPost(formData: FormData) {
  try {
    await db.posts.create({ data: { ... } });
    return { success: true };
  } catch (error) {
    console.error(error);
    return { success: false, error: 'Failed to create post' };
  }
}
```

### 4. **❌ Skipping Server-Side Validation**

```tsx
// Wrong - Trusting client data
export async function createPost(formData: FormData) {
  // Directly using unvalidated data
  await db.posts.create({
    data: {
      title: formData.get('title'),
      content: formData.get('content'),
    },
  });
}

// Correct - Always validate
export async function createPost(formData: FormData) {
  const validated = schema.parse({
    title: formData.get('title'),
    content: formData.get('content'),
  });
  
  await db.posts.create({ data: validated });
}
```

## When to Use Server Actions vs API Routes

### Use Server Actions for:

✅ Form submissions and data mutations
✅ Simple CRUD operations
✅ Operations tightly coupled with UI
✅ Progressive enhancement requirements
✅ Type-safe client-server communication

### Use API Routes for:

✅ Webhooks from external services
✅ Complex API logic with multiple endpoints
✅ Public APIs for third-party consumption
✅ Non-form-based integrations
✅ Custom response headers or status codes

## Learning Resources

- **Official Documentation**: [nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations)
- **React Documentation**: [react.dev/reference/react/use-server](https://react.dev/reference/react/use-server)
- **Examples**: [github.com/vercel/next.js/tree/canary/examples](https://github.com/vercel/next.js/tree/canary/examples)

## Next Steps

1. Start with simple form submissions
2. Add validation with Zod or similar libraries
3. Implement authentication and authorization
4. Explore optimistic updates for better UX
5. Learn about revalidation strategies
6. Practice error handling patterns

---

## Related Documentation

- [Next.js 16 Complete Introduction](file:///c:/Users/mmt20/Desktop/beetleware-frontend-intern/nexjs/nextjs-basics.md) - Next.js fundamentals
- [Next.js 16 App Router Routing Guide](file:///c:/Users/mmt20/Desktop/beetleware-frontend-intern/nexjs/app-router-routing-guide.md) - Routing patterns
- [Rendering & Data Fetching](file:///c:/Users/mmt20/Desktop/beetleware-frontend-intern/nexjs/rendering-and-data-fetching.md) - Data fetching strategies

---

**Pro Tip**: Server Actions are perfect for most form submissions and data mutations. Start with Server Actions and only create API routes when you need features that Server Actions don't provide (like webhooks or public APIs).
