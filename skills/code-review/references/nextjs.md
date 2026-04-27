# Next.js Best Practices

## Server vs Client Components

```tsx
// ❌ Bad: entire page marked 'use client' because one button needs onClick
'use client';

export default function DashboardPage() {
  const data = await fetchData(); // ← can't even do this in a client component
  return (
    <div>
      <h1>Dashboard</h1>
      <DataTable data={data} />
      <RefreshButton />  {/* only this needs interactivity */}
    </div>
  );
}

// ✅ Good: push 'use client' to the leaf
// app/dashboard/page.tsx (server component)
export default async function DashboardPage() {
  const data = await fetchData();
  return (
    <div>
      <h1>Dashboard</h1>
      <DataTable data={data} />
      <RefreshButton />  {/* this component has 'use client' */}
    </div>
  );
}

// components/refresh-button.tsx
'use client';
export function RefreshButton() {
  return <button onClick={() => window.location.reload()}>Refresh</button>;
}
```

## Data Fetching

```tsx
// ❌ Bad: API route for internal data fetching
// app/api/users/route.ts
export async function GET() {
  const users = await db.query('SELECT * FROM users');
  return Response.json(users);
}
// app/users/page.tsx
'use client';
const [users, setUsers] = useState([]);
useEffect(() => {
  fetch('/api/users').then(r => r.json()).then(setUsers);
}, []);

// ✅ Good: fetch directly in server component
// app/users/page.tsx
export default async function UsersPage() {
  const users = await db.query('SELECT * FROM users');
  return <UserList users={users} />;
}
```

```tsx
// ❌ Bad: no streaming, entire page blocks
export default async function Page() {
  const slowData = await fetchSlowData(); // blocks everything
  const fastData = await fetchFastData();
  return <div>{/* render both */}</div>;
}

// ✅ Good: stream with Suspense
import { Suspense } from 'react';

export default async function Page() {
  const fastData = await fetchFastData();
  return (
    <div>
      <FastSection data={fastData} />
      <Suspense fallback={<Skeleton />}>
        <SlowSection />  {/* fetches its own data, streams in */}
      </Suspense>
    </div>
  );
}
```

## Server Actions

```tsx
// ❌ Bad: no input validation in server action
'use server';
export async function createUser(formData: FormData) {
  const name = formData.get('name') as string;
  await db.insert({ name }); // trusting raw input
}

// ✅ Good: validate with zod
'use server';
import { z } from 'zod';

const schema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
});

export async function createUser(formData: FormData) {
  const parsed = schema.safeParse({
    name: formData.get('name'),
    email: formData.get('email'),
  });
  if (!parsed.success) return { error: parsed.error.flatten() };

  await db.insert(parsed.data);
  revalidatePath('/users');
}
```

## Images & Fonts

```tsx
// ❌ Bad: raw img tag
<img src="/hero.jpg" />

// ✅ Good: next/image with dimensions
import Image from 'next/image';
<Image src="/hero.jpg" width={1200} height={630} alt="Hero" />
```

```tsx
// ❌ Bad: external font CDN
<link href="https://fonts.googleapis.com/css2?family=Inter" rel="stylesheet" />

// ✅ Good: next/font
import { Inter } from 'next/font/google';
const inter = Inter({ subsets: ['latin'] });
```

## Route Structure

```
app/
├── layout.tsx          # root layout (nav, providers)
├── loading.tsx         # root loading state
├── error.tsx           # root error boundary
├── (auth)/
│   ├── login/page.tsx
│   └── register/page.tsx
├── (dashboard)/
│   ├── layout.tsx      # dashboard shell
│   ├── loading.tsx     # dashboard loading
│   ├── page.tsx        # dashboard home
│   └── settings/
│       ├── page.tsx
│       └── loading.tsx
```

## Environment Variables

```typescript
// ❌ Bad: server secret exposed to client
// .env
NEXT_PUBLIC_SUPABASE_SERVICE_ROLE_KEY=eyJ...  // CRITICAL: never do this

// ✅ Good: proper separation
// .env
SUPABASE_SERVICE_ROLE_KEY=eyJ...              // server only
NEXT_PUBLIC_SUPABASE_URL=https://...          // safe for client
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...          // safe for client
```

## Metadata

```tsx
// ❌ Bad: manual head manipulation
export default function Page() {
  return (
    <>
      <head><title>My Page</title></head>
      <main>Content</main>
    </>
  );
}

// ✅ Good: metadata export
export const metadata = {
  title: 'My Page',
  description: 'Page description for SEO',
};

// ✅ Good: dynamic metadata
export async function generateMetadata({ params }) {
  const product = await getProduct(params.id);
  return { title: product.name };
}
```

## Severity

- Server secrets exposed to client (`NEXT_PUBLIC_` prefix on secrets) → `[CRITICAL]`
- Entire page `'use client'`; API routes for internal fetching → `[MAJOR]`
- Missing metadata, raw `<img>`, missing `loading.tsx` → `[MINOR]`
- Convention deviations → `[NIT]`
