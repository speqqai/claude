# Common Anti-Patterns

Code examples of frequently flagged issues.

## God Components

```tsx
// ❌ Bad: one component doing everything
export function UserDashboard() {
  const [users, setUsers] = useState([]);
  const [search, setSearch] = useState('');
  const [sort, setSort] = useState('name');
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [selectedUser, setSelectedUser] = useState(null);
  const [isEditing, setIsEditing] = useState(false);
  const [formData, setFormData] = useState({});
  // ... 200 more lines of mixed concerns

  // Fetching + filtering + sorting + rendering + editing all in one file
}

// ✅ Good: separated concerns
// hooks/use-users.ts — data fetching
// components/user-filters.tsx — search and sort controls
// components/user-list.tsx — list rendering
// components/user-edit-dialog.tsx — edit form
// app/users/page.tsx — composes the above
```

## Magic Numbers

```typescript
// ❌ Bad
if (password.length < 8) throw new Error('Too short');
if (items.length > 100) paginate();
setTimeout(retry, 3000);

// ✅ Good
const MIN_PASSWORD_LENGTH = 8;
const MAX_PAGE_SIZE = 100;
const RETRY_DELAY_MS = 3000;

if (password.length < MIN_PASSWORD_LENGTH) throw new Error('Too short');
if (items.length > MAX_PAGE_SIZE) paginate();
setTimeout(retry, RETRY_DELAY_MS);
```

## Deep Nesting

```typescript
// ❌ Bad
async function processOrder(order: Order) {
  if (order) {
    if (order.items.length > 0) {
      if (order.status === 'pending') {
        const user = await getUser(order.userId);
        if (user) {
          if (user.isActive) {
            // actual logic buried 5 levels deep
          }
        }
      }
    }
  }
}

// ✅ Good: early returns
async function processOrder(order: Order) {
  if (!order) return;
  if (order.items.length === 0) return;
  if (order.status !== 'pending') return;

  const user = await getUser(order.userId);
  if (!user?.isActive) return;

  // actual logic at top level
}
```

## Silent Error Swallowing

```typescript
// ❌ Bad: error disappears
try {
  await saveUser(data);
} catch (e) {
  // silently ignored
}

// ❌ Also bad: catch-and-log without handling
try {
  await saveUser(data);
} catch (e) {
  console.log(e); // logged but nothing happens, user sees no feedback
}

// ✅ Good: handle meaningfully
try {
  await saveUser(data);
} catch (error) {
  logger.error('Failed to save user', { error, userId: data.id });
  throw new AppError('Could not save your changes. Please try again.');
}
```

## N+1 Queries

```typescript
// ❌ Bad: N+1 — one query per user
const users = await supabase.from('users').select('id, name');
for (const user of users.data!) {
  const { data: posts } = await supabase
    .from('posts')
    .select('*')
    .eq('user_id', user.id);
  user.posts = posts;
}

// ✅ Good: single query with join
const { data } = await supabase
  .from('users')
  .select('id, name, posts(id, title, created_at)');
```

## Hardcoded Secrets

```typescript
// ❌ CRITICAL: secrets in source code
const STRIPE_KEY = 'sk_live_1234567890abcdef';
const DB_PASSWORD = 'super_secret_password';

// ✅ Good: from environment
const STRIPE_KEY = process.env.STRIPE_SECRET_KEY;
if (!STRIPE_KEY) throw new Error('STRIPE_SECRET_KEY not configured');
```

## SQL Injection

```typescript
// ❌ Bad: string interpolation in query
const { data } = await supabase
  .rpc('search_users', { search_term: `%${userInput}%` });
// If the RPC function uses the input in raw SQL, this is injectable

// ✅ Good: parameterized via Supabase query builder
const { data } = await supabase
  .from('users')
  .select('*')
  .ilike('name', `%${userInput}%`);
// Query builder handles parameterization
```

## Unnecessary Re-renders

```tsx
// ❌ Bad: new object/array reference every render
function UserList({ users }: { users: User[] }) {
  return (
    <ExpensiveComponent
      config={{ showAvatar: true, maxItems: 10 }}  // new object every render
      columns={['name', 'email', 'role']}           // new array every render
      onSelect={(user) => handleSelect(user)}        // new function every render
    />
  );
}

// ✅ Good: stable references
const CONFIG = { showAvatar: true, maxItems: 10 } as const;
const COLUMNS = ['name', 'email', 'role'] as const;

function UserList({ users }: { users: User[] }) {
  const handleSelect = useCallback((user: User) => {
    // handle selection
  }, []);

  return (
    <ExpensiveComponent
      config={CONFIG}
      columns={COLUMNS}
      onSelect={handleSelect}
    />
  );
}
```

## Missing Loading/Error/Empty States

```tsx
// ❌ Bad: only handles success
function UserList() {
  const { data } = useQuery(['users'], fetchUsers);
  return <ul>{data.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
  // Crashes if data is undefined; shows nothing on error; no loading indicator
}

// ✅ Good: all states handled
function UserList() {
  const { data, error, isLoading } = useQuery(['users'], fetchUsers);

  if (isLoading) return <Skeleton count={5} />;
  if (error) return <ErrorMessage message="Failed to load users" retry />;
  if (!data?.length) return <EmptyState message="No users found" />;

  return <ul>{data.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```
