---
name: Billing code standards
description: No try-catch, no inline math, all constants in lib/, all types from supabase-generated.ts
type: feedback
---

No try-catch blocks in route handlers. Let errors propagate naturally or use conditional checks.
No inline math equations (e.g., `* 1000` for timestamp conversion) — use date-fns or named functions.
All constants go in `lib/constants/` files, never defined inline in route files.
All database types come from `lib/types/supabase-generated.ts` — no hand-written type aliases for DB rows.

**Why:** Keep route handlers clean and declarative. Constants in lib/ are discoverable. Generated types are the single source of truth. Inline math is opaque.

**How to apply:** Before writing a route handler, check if any constant or type is being defined inline. If so, move it to the appropriate lib/ file. Use date-fns for all date/time operations.
