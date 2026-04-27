# TypeScript Strictness

## Critical Checks

### No `any` without justification

```typescript
// ❌ Bad: lazy any
function processData(data: any) {
  return data.items.map((item: any) => item.name);
}

// ✅ Good: proper typing
interface DataResponse {
  items: Array<{ name: string; id: string }>;
}

function processData(data: DataResponse) {
  return data.items.map((item) => item.name);
}

// ✅ Acceptable: justified any with comment
// eslint-disable-next-line @typescript-eslint/no-explicit-any
// any required: third-party lib types are incorrect (see #1234)
function wrapLegacyLib(input: any): SafeOutput {
  return sanitize(input);
}
```

### No unsafe `as` casts

```typescript
// ❌ Bad: casting API response as trusted
const user = (await response.json()) as User;

// ✅ Good: validate at runtime
const parsed = userSchema.safeParse(await response.json());
if (!parsed.success) throw new ValidationError(parsed.error);
const user = parsed.data;
```

### Discriminated unions over optional fields

```typescript
// ❌ Bad: ambiguous state
type ApiResult = {
  data?: User;
  error?: string;
  loading?: boolean;
};

// ✅ Good: impossible states are impossible
type ApiResult =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: User }
  | { status: 'error'; error: string };
```

### No non-null assertions

```typescript
// ❌ Bad: trusting that it exists
const user = users.find((u) => u.id === id)!;

// ✅ Good: handle the possibility
const user = users.find((u) => u.id === id);
if (!user) throw new NotFoundError(`User ${id} not found`);
```

### Types from schemas, not parallel definitions

```typescript
// ❌ Bad: types defined separately from validation
interface CreateUserInput {
  email: string;
  name: string;
}

const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1),
});

// ✅ Good: single source of truth
const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1),
});
type CreateUserInput = z.infer<typeof createUserSchema>;
```

### Database types generated

```typescript
// ❌ Bad: manually typing database rows
interface UserRow {
  id: string;
  email: string;
  created_at: string;
}

// ✅ Good: generated from Supabase
import { Database } from '@/types/supabase';
type UserRow = Database['public']['Tables']['users']['Row'];
```

### Enums vs const objects

```typescript
// ❌ Avoid: TypeScript enum (runtime overhead, poor tree-shaking)
enum Status {
  Active = 'active',
  Inactive = 'inactive',
}

// ✅ Prefer: const object or string union
const STATUS = {
  Active: 'active',
  Inactive: 'inactive',
} as const;
type Status = (typeof STATUS)[keyof typeof STATUS];

// ✅ Also fine: simple string union
type Status = 'active' | 'inactive';
```

## Common Anti-Patterns to Flag

| Pattern | Problem | Fix |
|---------|---------|-----|
| `Record<string, any>` | No type safety | Define specific shape |
| `Function` type | No signature info | Use specific `(args) => return` |
| `event: any` on handlers | Skips React event types | `React.ChangeEvent<HTMLInputElement>` |
| Inline types repeated | DRY violation | Extract shared type |
| `as unknown as Type` | Double cast = red flag | Fix the actual type mismatch |
| `@ts-ignore` without comment | Unknown suppression | Add comment + issue link |

## Severity

- `strict: false` in tsconfig → `[CRITICAL]`
- `any` without justification; unsafe `as` on external data → `[MAJOR]`
- Missing return types on exports; unexplained `@ts-ignore` → `[MINOR]`
- Style preferences (interface vs type) → `[NIT]`
