# Supabase Best Practices

## Row-Level Security (RLS)

### Every table must have RLS enabled

```sql
-- ❌ Bad: RLS disabled
ALTER TABLE public.documents DISABLE ROW LEVEL SECURITY;

-- ✅ Good: RLS enabled with policies
ALTER TABLE public.documents ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own documents"
  ON public.documents FOR SELECT
  USING (user_id = auth.uid());

CREATE POLICY "Users can insert own documents"
  ON public.documents FOR INSERT
  WITH CHECK (user_id = auth.uid());

CREATE POLICY "Users can update own documents"
  ON public.documents FOR UPDATE
  USING (user_id = auth.uid())
  WITH CHECK (user_id = auth.uid());

CREATE POLICY "Users can delete own documents"
  ON public.documents FOR DELETE
  USING (user_id = auth.uid());
```

### Never use `raw_user_meta_data` for authorization

```sql
-- ❌ CRITICAL: user can modify their own JWT claims
CREATE POLICY "Admin access"
  ON public.admin_data FOR SELECT
  USING (
    (auth.jwt() -> 'user_metadata' ->> 'role') = 'admin'
  );

-- ✅ Good: check against a database table
CREATE POLICY "Admin access"
  ON public.admin_data FOR SELECT
  USING (
    auth.uid() IN (SELECT user_id FROM public.user_roles WHERE role = 'admin')
  );
```

### RLS performance — filter by auth.uid() first

```sql
-- ❌ Slow: checks row column against subquery result per row
CREATE POLICY "Team access"
  ON public.projects FOR SELECT
  USING (
    auth.uid() IN (
      SELECT user_id FROM team_members WHERE team_members.team_id = projects.team_id
    )
  );

-- ✅ Fast: gets user's teams first, then filters
CREATE POLICY "Team access"
  ON public.projects FOR SELECT
  USING (
    team_id IN (
      SELECT team_id FROM team_members WHERE user_id = auth.uid()
    )
  );

-- ✅ Even better: security definer function for complex lookups
CREATE OR REPLACE FUNCTION user_team_ids()
RETURNS SETOF uuid
LANGUAGE sql
SECURITY DEFINER
SET search_path = public
AS $$
  SELECT team_id FROM team_members WHERE user_id = auth.uid()
$$;

CREATE POLICY "Team access"
  ON public.projects FOR SELECT
  USING (team_id IN (SELECT user_team_ids()));
```

## Client Usage

### Correct client per context

```typescript
// ❌ Bad: browser client in a server component
import { createClient } from '@supabase/supabase-js';
// This creates a client-side instance — wrong in server context

// ✅ Good: use the right client

// Client component
import { createBrowserClient } from '@supabase/ssr';
const supabase = createBrowserClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
);

// Server component / server action
import { createServerClient } from '@supabase/ssr';
import { cookies } from 'next/headers';
// ... create with cookie handlers

// Route handler
import { createRouteHandlerClient } from '@supabase/auth-helpers-nextjs';
```

### Never expose service_role key to client

```typescript
// ❌ CRITICAL: service_role bypasses ALL RLS
const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_SERVICE_ROLE_KEY!, // ← NEVER in client code
);

// ✅ Good: service_role only in trusted server code (webhooks, admin scripts)
// And only when you specifically need to bypass RLS
```

### Always handle errors

```typescript
// ❌ Bad: ignoring the error
const { data } = await supabase.from('users').select('*');
// If this fails, data is null and you get a cryptic runtime error

// ✅ Good: check error
const { data, error } = await supabase.from('users').select('*');
if (error) {
  console.error('Failed to fetch users:', error.message);
  throw new DatabaseError('Could not load users');
}
```

### Select only needed columns

```typescript
// ❌ Bad on wide tables: pulling everything
const { data } = await supabase.from('users').select('*');

// ✅ Good: explicit columns
const { data } = await supabase.from('users').select('id, name, email, avatar_url');
```

## Realtime

### Always clean up subscriptions

```typescript
// ❌ Bad: subscription never unsubscribed
useEffect(() => {
  const channel = supabase
    .channel('changes')
    .on('postgres_changes', { event: '*', schema: 'public', table: 'messages' }, handleChange)
    .subscribe();
  // Missing cleanup — memory leak
}, []);

// ✅ Good: unsubscribe on unmount
useEffect(() => {
  const channel = supabase
    .channel('changes')
    .on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'messages',
      filter: `room_id=eq.${roomId}` }, handleChange)
    .subscribe();

  return () => {
    supabase.removeChannel(channel);
  };
}, [roomId]);
```

## Migrations

```
-- ❌ Bad: schema changes without migration files
-- Just clicking around in the Supabase dashboard

-- ✅ Good: migration file
-- supabase/migrations/20240101000000_create_documents.sql
CREATE TABLE public.documents (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id uuid REFERENCES auth.users(id) NOT NULL,
  title text NOT NULL,
  content jsonb DEFAULT '{}'::jsonb,
  created_at timestamptz DEFAULT now()
);

ALTER TABLE public.documents ENABLE ROW LEVEL SECURITY;

-- Policies ship with the table
CREATE POLICY "Users can CRUD own documents"
  ON public.documents FOR ALL
  USING (user_id = auth.uid())
  WITH CHECK (user_id = auth.uid());
```

## Severity

- RLS disabled or missing policies → `[CRITICAL]`
- `service_role` key in client bundle → `[CRITICAL]`
- Auth checks using `raw_user_meta_data` → `[CRITICAL]`
- Wrong client for context; ignored errors → `[MAJOR]`
- Missing realtime cleanup; `select('*')` on wide tables → `[MINOR]`
