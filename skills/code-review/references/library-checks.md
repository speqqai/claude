# Custom Code Audit

## The Rule

**Do not build anything that your stack already provides. This list is not exhaustive — it's examples. If a framework, library, or package in your stack offers a feature, use it. Check before you build.**

The burden of proof is on custom code: **you must prove no alternative exists**, not prove the custom code works.

---

## How To Check (for anything, listed or not)

For every new function, component, hook, or utility:

1. **Check the framework.** Does React, Next.js, or Supabase have a built-in for this?
2. **Check the UI library.** Does shadcn/ui or Radix have a component for this?
3. **Check `package.json`.** Does an installed package already do this?
4. **Check hooks libraries.** Does `usehooks-ts` or SWR have a hook for this?
5. **Check platform APIs.** Does the browser or Node have a native API for this?
6. **Check npm.** Does a package with >1K weekly downloads do this?
7. **If nothing exists** — state: "Removing this cuts [feature] because [reason]. No built-in or library handles [specific thing]."

If you skip any of these checks, the audit fails.

---

## React — don't rebuild anything React provides

Including but not limited to:

- [ ] `useState`, `useReducer` for component state
- [ ] `React.createContext` for shared state
- [ ] `React.createPortal` for portals
- [ ] `React.memo`, `useMemo`, `useCallback` for memoization
- [ ] `React.forwardRef` / `ref` prop for ref forwarding
- [ ] `React.lazy` + `Suspense` for lazy loading
- [ ] `useId` for unique IDs
- [ ] `useTransition`, `useDeferredValue` for transitions
- [ ] `startTransition` for non-urgent updates

**If React has a hook or API for it, don't write your own version.**

## Next.js — don't rebuild anything Next.js provides

Including but not limited to:

- [ ] `next/navigation` (useRouter, usePathname, useSearchParams) for routing
- [ ] `next/link` for navigation
- [ ] `next/image` for images
- [ ] `next/font` for fonts
- [ ] `next/script` for third-party scripts
- [ ] `metadata` / `generateMetadata` for SEO
- [ ] `loading.tsx`, `error.tsx`, `not-found.tsx` file conventions
- [ ] `middleware.ts` for redirects, auth, rewrites
- [ ] Server Actions (`'use server'`) for mutations
- [ ] `revalidatePath` / `revalidateTag` for cache invalidation
- [ ] `generateStaticParams` for static generation
- [ ] `Suspense` boundaries for streaming
- [ ] Parallel routes for split views
- [ ] Route groups `(group)` for organization
- [ ] `next.config.js` redirects, rewrites, headers

**If Next.js has a convention, API, or file-based solution for it, don't write a custom version.**

## Supabase — don't rebuild anything Supabase provides

Including but not limited to:

- [ ] `supabase.auth` for login, signup, reset, OAuth, session
- [ ] `@supabase/ssr` for session middleware in Next.js
- [ ] `supabase.auth.getUser()` instead of custom JWT decoding
- [ ] RLS policies instead of app-level authorization filters
- [ ] Supabase Realtime for pub/sub and live queries
- [ ] Supabase Storage for file uploads
- [ ] `supabase gen types typescript` for database types
- [ ] `.from().select().eq()` chain instead of custom query builders
- [ ] `.textSearch()` for full-text search
- [ ] Database Webhooks / `pg_net` for DB-triggered events
- [ ] Edge Functions for serverless compute

**If Supabase has a client method, service, or feature for it, don't build a custom alternative.**

## shadcn/ui + Radix — don't rebuild any component they provide

Including but not limited to:

- [ ] Button, Input, Textarea, Label
- [ ] Select, Checkbox, RadioGroup, Switch, Slider
- [ ] Dialog, Sheet, Popover, HoverCard, Tooltip
- [ ] DropdownMenu, ContextMenu, NavigationMenu, Menubar
- [ ] Tabs, Accordion, Collapsible
- [ ] Card, Alert, Badge, Avatar
- [ ] Skeleton, Progress, Separator
- [ ] Toast (sonner), Command (cmdk), Calendar
- [ ] Table + `@tanstack/react-table` for data tables
- [ ] Form + `react-hook-form` + Zod for forms
- [ ] ScrollArea, AspectRatio, Breadcrumb

**If shadcn or Radix has a component for it — even close to it — use it and customize via props/variants. Don't rebuild from scratch.**

## SWR — don't rebuild any data pattern SWR provides

Including but not limited to:

- [ ] `useSWR` instead of custom fetch hooks
- [ ] `mutate` with `optimisticData` instead of custom optimistic updates
- [ ] `errorRetryCount` instead of custom retry logic
- [ ] `refreshInterval` instead of custom polling
- [ ] Built-in request deduplication — don't build your own
- [ ] Built-in stale-while-revalidate — that's literally what SWR does

**If SWR has a config option or pattern for it, use it.**

## usehooks-ts — don't rebuild any hook they provide

Including but not limited to:

- [ ] useDebounce / useDebouncedCallback
- [ ] useLocalStorage
- [ ] useMediaQuery
- [ ] useOnClickOutside
- [ ] useWindowSize
- [ ] useCopyToClipboard
- [ ] useIntersectionObserver
- [ ] useEventListener
- [ ] useInterval / useTimeout
- [ ] useToggle
- [ ] useHover
- [ ] useScrollLock
- [ ] useIsClient
- [ ] usePrevious
- [ ] useCountdown

**Before writing any `use*` hook, check the usehooks-ts catalog. If they have it, use theirs.**

## Utilities — don't rebuild these

Including but not limited to:

- [ ] Date/time → `date-fns` or `Intl.DateTimeFormat` (built-in)
- [ ] Validation → `zod` (already in stack)
- [ ] Deep clone → `structuredClone` (built-in)
- [ ] Deep merge → `deepmerge` or `lodash/merge`
- [ ] IDs → `crypto.randomUUID()` (built-in) or `nanoid`
- [ ] Debounce/throttle → `lodash/debounce` or `usehooks-ts`
- [ ] Retry → `p-retry`
- [ ] Slugify → `slugify`
- [ ] Crypto/hashing → platform crypto (built-in) — **never custom**
- [ ] Array groupBy → `Object.groupBy` (built-in)
- [ ] Query strings → `URLSearchParams` (built-in)
- [ ] URL construction → `new URL()` (built-in)
- [ ] String case conversion → `change-case`
- [ ] Markdown parsing → `react-markdown`, `marked`, `remark`
- [ ] HTML sanitization → `dompurify`, `sanitize-html`
- [ ] CSV parsing → `papaparse`
- [ ] YAML parsing → `js-yaml`
- [ ] Phone validation → `libphonenumber-js`
- [ ] Env var validation → `zod` or `t3-env`

**Check platform built-ins first, then `package.json`, then npm.**

---

## Required Output

For every custom function, component, hook, or utility in the diff:

```
| Custom Code | What It Does | Built-in / Library | Verdict |
|-------------|-------------|-------------------|---------| 
| useDebounce() | Debounces a value | usehooks-ts useDebouncedValue | ❌ REPLACE |
| <LoadingDots/> | Animated loading | shadcn Skeleton | ❌ REPLACE |
| fetchWithRetry() | Fetch + 3 retries | SWR errorRetryCount | ❌ REPLACE |
| calculatePricing() | Speqq pricing tiers | None — domain logic | ✅ KEEP — cuts pricing |
```

- **❌ REPLACE** = something in the stack handles this → use it
- **✅ KEEP** = nothing handles this → must state what feature gets cut without it
- **⚠️ SIMPLIFY** = partially covered → use the library for the covered part

**Any ❌ REPLACE = blocker. Fix before submitting.**
